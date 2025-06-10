# Connecting EC2 Instances Across VPCs, Regions, and AWS Accounts

## Table of Contents

* [1. EC2s in the Same VPC](#1-ec2s-in-the-same-vpc)
* [2. EC2s in Different VPCs (Same Region, Same Account)](#2-ec2s-in-different-vpcs-same-region-same-account)
* [3. EC2s in Different VPCs (Cross-Region, Same Account)](#3-ec2s-in-different-vpcs-cross-region-same-account)
* [4. EC2s in Different VPCs (Same Region, Different Accounts)](#4-ec2s-in-different-vpcs-same-region-different-accounts)
* [5. Exposing a Service Privately with AWS PrivateLink](#5-exposing-a-service-privately-with-aws-privatelink)
* [6. EC2s in Different VPCs (Cross-Region & Cross-Account)](#6-ec2s-in-different-vpcs-cross-region--cross-account)
* [7. Connecting EC2s via Public IP (Least Secure)](#7-connecting-ec2s-via-public-ip-least-secure)
* [8. Summary Table](#8-summary-table)

In AWS, networking plays a critical role in enabling secure, scalable, and cost-effective communication between your resources. This guide blends conceptual explanations with **step-by-step, console-based walkthroughs** to provide a clear path for connecting EC2 instances in any scenario.

---

## 1. EC2s in the Same VPC](#1-ec2s-in-the-same-vpc)

**Concept:** Instances within the same Virtual Private Cloud (VPC) can communicate over AWS’s private network using their private IP addresses by default. This is the simplest and most common scenario.

**Implementation Steps:**

1.  **Launch EC2 instances** in the same VPC. They can be in the same or different subnets within that VPC.
2.  **Configure Security Groups (Best Practice):**
    *   Navigate to **EC2 → Security Groups**.
    *   Select the security group attached to your instances and click **Edit inbound rules**.
    *   Add a rule to allow traffic from other instances within the same group:
        *   **Type:** `All ICMP - IPv4` (for ping tests) or your specific application port (e.g., `SSH`, `HTTP`).
        *   **Source:** Instead of a hardcoded IP range, reference the security group itself. Start typing `sg-` in the source field and select the group's ID from the list.
    *   **Why?** Referencing the security group ID is more secure and dynamic than using the VPC CIDR. It automatically grants access to any instance that is part of the group, without exposing ports to the entire VPC.
3.  **Connect:** From one instance, use the private IP of the other instance to connect.
    ```bash
    # Example: Pinging another instance in the same security group
    ping <Private-IP-of-other-EC2>

    # Example: SSHing to another instance
    ssh -i key.pem ec2-user@<Private-IP-of-other-EC2>
**Result:** Secure, zero-cost (within the same Availability Zone), low-latency communication entirely on the AWS backbone.

---

## 2. EC2s in Different VPCs (Same Region, Same Account)](#2-ec2s-in-different-vpcs-same-region-same-account)

**Concept:** **VPC Peering** creates a private, point-to-point connection between two VPCs, allowing them to communicate as if they were in the same network. Traffic uses private IPs and stays on the AWS global network.

### A. Create and Accept the Peering Connection

1.  Navigate to the **VPC Console** → **Peering Connections** → **Create peering connection**.
2.  **Name tag:** `VPC-A-to-VPC-B`.
3.  **VPC (Requester):** Select **VPC-A** (e.g., with CIDR `172.31.0.0/16`).
4.  **VPC (Accepter):** Choose **Another VPC in this account** and select **VPC-B** (e.g., with CIDR `10.0.0.0/24`).
5.  Click **Create peering connection**.
6.  The connection is now `pending-acceptance`. Select it, then click **Actions** → **Accept request**.

### B. Configure Route Tables in Both VPCs

1.  In **VPC → Route Tables**, find the route table associated with the subnet of your instance in **VPC-A**.
2.  Select the route table → **Routes** tab → **Edit routes**.
3.  **Add route:**
    *   **Destination:** The CIDR of **VPC-B** (`10.0.0.0/24`).
    *   **Target:** Select **Peering Connection** and choose the `pcx-...` ID you just created.
4.  **Save routes**.
5.  **Repeat for VPC-B's route table**: Add a route with **Destination** `172.31.0.0/16` and the same peering connection as the **Target**.

### C. Update Security Groups

1.  In **EC2-A's security group**, add an inbound rule allowing traffic from **VPC-B's CIDR**:
    *   Type: SSH (or your port), Source: `10.0.0.0/24`.
2.  In **EC2-B's security group**, add an inbound rule allowing traffic from **VPC-A's CIDR**:
    *   Type: SSH (or your port), Source: `172.31.0.0/16`.

### D. Test the Connection

From an instance in VPC-A, connect to an instance in VPC-B using its private IP.
```bash
ssh -i key.pem ec2-user@<EC2-B-private-IP>
```

**Common Pitfalls:**
*   **Missing or Incorrect Routes:** Forgetting to update the route tables in *both* VPCs is the most common error.
*   **Security Group / NACL Rules:** Ensure SGs and Network ACLs permit traffic between the peered CIDR blocks.
*   **Overlapping CIDRs:** VPCs with overlapping IP ranges cannot be peered.
*   **DNS Resolution Failure:** By default, you can't use private DNS hostnames (e.g., `ip-10-0-1-12.ec2.internal`) across a peering connection. You must enable this setting on the peering connection itself (**Actions → Edit DNS Settings**).

---

## 3. EC2s in Different VPCs (Cross-Region, Same Account)](#3-ec2s-in-different-vpcs-cross-region-same-account)

**Concept:** The same VPC Peering mechanism can be used to connect VPCs across different AWS regions.

**Steps:**
1.  In the source region (**VPC Console → Peering Connections → Create**):
    *   Requester: VPC-A (e.g., `us-east-1`, CIDR `172.31.0.0/16`).
    *   Accepter: Select **Another VPC in another region**. Provide the Accepter VPC's region (`eu-west-1`) and VPC ID (`vpc-...`).
2.  **Accept in the Accepter Region:** Switch to the accepter region (`eu-west-1`) in the console, find the pending request, and accept it.
3.  **Update Route Tables:** In **both** VPCs, add a route for the peer’s CIDR, targeting the peering connection ID (`pcx-...`).
4.  **Update Security Groups:** In each VPC, allow inbound traffic from the other VPC's CIDR block.
5.  **Connect** using the private IP address.

> **Cost Alert:** All data transferred over a cross-region peering connection incurs charges. Monitor these costs closely.

---

## 4. EC2s in Different VPCs (Same Region, Different Accounts)](#4-ec2s-in-different-vpcs-same-region-different-accounts)

**Concept:** For connecting a few VPCs, **Inter-Account VPC Peering** is sufficient. For connecting many VPCs at scale, **AWS Transit Gateway** is the modern, recommended approach.

### A. Inter-Account VPC Peering

1.  **Account A (Requester):** Go to **VPC → Peering Connections → Create**.
    *   Requester VPC: Select your local VPC-A.
    *   Accepter VPC: Select **Another VPC in another account**. Enter the **Account ID** of Account B and the **VPC ID** of VPC-B.
2.  **Account B (Accepter):** Log in to Account B, navigate to Peering Connections, and **Accept** the request.
3.  **Update Route Tables and Security Groups** in both accounts, just as in the same-account peering scenario.

### B. AWS Transit Gateway (for many VPCs)

A Transit Gateway (TGW) acts as a central cloud router. Instead of creating many individual peering connections (which don't support transitive routing), you connect each VPC to the TGW.

1.  **Account A (Hub Account):** Create a **Transit Gateway**.
2.  **Share the TGW:** Use **AWS Resource Access Manager (RAM)** to share the Transit Gateway with Account B's organization or account ID.
3.  **Account B (Spoke Account):** Accept the RAM share.
4.  **Create Attachments:** In each account, create a **Transit Gateway Attachment** to connect its VPC to the shared TGW.
5.  **Update VPC Route Tables:**
    *   In **VPC-A's** route table, add a route for VPC-B's CIDR, with the **Target** set to the **Transit Gateway ID**.
    *   In **VPC-B's** route table, add a route for VPC-A's CIDR, with the **Target** set to the **Transit Gateway ID**.
6.  **Verify TGW Route Table:** The TGW has its own route table that automatically propagates routes from attached VPCs, enabling them to communicate.

> **Use TGW** when you need to connect more than two VPCs, as it simplifies management and enables a hub-and-spoke network topology.

---

## 5. Exposing a Service Privately with AWS PrivateLink](#5-exposing-a-service-privately-with-aws-privatelink)

**Concept:** Instead of connecting entire networks, **AWS PrivateLink** allows you to expose a specific service (e.g., an application running on EC2 behind a Network Load Balancer) from one VPC to consumers in another. Traffic is one-way and never leaves the AWS network. This is fundamentally more secure for service-oriented communication.

**When to Use PrivateLink:**
*   You want to provide **one-way access to a specific application/API**, not full network connectivity.
*   You want to **avoid CIDR conflicts**, route table management, and the security concerns of network peering.
*   You are a SaaS provider or internal platform team exposing a service to multiple consumer VPCs (across accounts and regions).

**High-Level Steps:**
1.  **Provider VPC:** Place your EC2 instances behind a **Network Load Balancer (NLB)**.
2.  **Endpoint Service:** In the Provider's account, create a **VPC Endpoint Service** that points to the NLB.
3.  **Consumer VPC:** In the Consumer's account, create an **Interface VPC Endpoint** to connect to the Endpoint Service. This creates an Elastic Network Interface (ENI) in the consumer's subnet.
4.  **Connect:** The consumer application can now send traffic to the service by using the private DNS name of the VPC Endpoint ENI.

---

## 6. EC2s in Different VPCs (Cross-Region & Cross-Account)](#6-ec2s-in-different-vpcs-cross-region--cross-account)

**Concept:** Connecting networks across both account and region boundaries is a common enterprise requirement. The recommended approach for scalability and manageability is Transit Gateway Peering.

### A. Transit Gateway Peering (Recommended for Scale)

This architecture uses a TGW in each region and connects them.

1.  **Create TGWs:** Provision a TGW in each region (e.g., TGW-A in Account A, `us-east-1`; TGW-B in Account B, `eu-west-1`).
2.  **Attach Local VPCs:** Attach the local VPC in each region to its respective TGW.
3.  **Create TGW Peering Attachment:** From TGW-A, initiate a peering request to TGW-B, specifying its TGW ID and region.
4.  **Accept Peering:** In Account B, accept the TGW peering request.
5.  **Update TGW Route Tables:** In each TGW's route table, add a static route for the remote VPC's CIDR, pointing the traffic to the **TGW peering attachment**.

### B. Cross-Account, Cross-Region VPC Peering (For Simple Setups)

For a simple point-to-point connection, you can use standard VPC peering. The steps are a combination of cross-region and cross-account peering.

1.  **Initiate Peering:** From Account A, create a peering connection, specifying the VPC in the current region and providing the **Account ID**, **VPC ID**, and **Region** for the VPC in Account B.
2.  **Accept Peering:** In Account B, switch to the correct region and accept the request.
3.  **Update Route Tables & Security Groups** in both VPCs to allow traffic to flow over the new peering connection.

---

## 7. Connecting EC2s via Public IP (Least Secure)](#7-connecting-ec2s-via-public-ip-least-secure)

**Concept:** This method uses the public internet to connect instances. While simple to set up, it is the least secure and should be avoided for private communication.

**Implementation:**
1.  Ensure both EC2 instances have an **Elastic IP** or Public IP address.
2.  Ensure their subnets have a route to an **Internet Gateway**.
3.  In the security group for each instance, add an inbound rule allowing traffic (e.g., SSH) from the **Public IP** of the other instance.
4.  Connect using the Public IP or DNS hostname.

**Drawbacks:** Public exposure (increased attack surface), higher data egress costs, potential for higher latency, and compliance risks.

---

## 8. Summary Table](#8-summary-table)

| Scenario                                          | Method                                       | Setup Complexity | Cost Impact (Data Transfer)        | Key Use Case                                      |
| ------------------------------------------------- | -------------------------------------------- | ---------------- | ---------------------------------- | ------------------------------------------------- |
| EC2s in Same VPC                                  | Native (Private IP)                          | Very Low         | Free (within same AZ)              | Standard application components.                  |
| Different VPCs (Same Region, Same Account)        | VPC Peering                                  | Low              | Low (in-region peering fees)       | Connecting two or three VPCs for full access.     |
| Different VPCs (Cross-Region, Same Account)       | Cross-Region VPC Peering                     | Low-Medium       | Medium (cross-region rates)        | Disaster recovery, geo-distributed apps.          |
| Different VPCs (Same Region, Different Accounts)  | Transit Gateway (recommended) or Peering   | Medium           | Low-Medium                         | Centralized networking, scalable multi-VPC env.   |
| **Service-to-Service Connectivity (Any)**         | **AWS PrivateLink**                          | Medium           | Medium (per-GB + hourly endpoint)  | **Securely exposing one service to consumers.**   |
| Different VPCs (Cross-Region & Cross-Account)     | TGW Peering (recommended) or VPC Peering   | High             | High (cross-region + TGW fees)     | Global enterprise networks.                       |
| EC2s via Public IP                                | Public Internet                              | Very Low         | High (standard internet egress)    | **Not recommended for private traffic.**          |