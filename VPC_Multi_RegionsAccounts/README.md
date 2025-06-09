# Connecting EC2 Instances Across VPCs, Regions, and AWS Accounts

In AWS, networking plays a critical role in enabling secure, scalable, and cost-effective communication between your resources. One of the most common scenarios is allowing EC2 instances to talk to each other across different VPCs, regions, and even AWS accounts. This document will walk you through the different ways to achieve this, including both the theory and detailed implementation steps.

---

## Why This is Important

As your architecture grows, you might deploy EC2 instances in different:

* VPCs (to separate environments like dev, staging, and prod)
* Availability Zones (for high availability)
* Regions (for latency optimization or disaster recovery)
* AWS Accounts (for cost separation, security boundaries, or team isolation)

Understanding how to interconnect these EC2s is essential for building a truly distributed and scalable system.

---

## Understanding the Scope

| Scenario                                                 | Connectivity Possible? | Method                                            |
| -------------------------------------------------------- | ---------------------- | ------------------------------------------------- |
| EC2s in same VPC                                         | Yes                    | No setup required                                 |
| EC2s in different VPCs (same region, same account)       | Yes                    | VPC Peering                                       |
| EC2s in different VPCs (different regions, same account) | Yes                    | Cross-region VPC Peering or Site-to-Site VPN      |
| EC2s in different AWS accounts                           | Yes                    | VPC Peering (inter-account), Transit Gateway, VPN |
| EC2s via public IP                                       | Yes (not recommended)  | Public IP + SG rules                              |

---

## EC2 Instances with Public IPs (Across VPCs)

If you create EC2 instances in different VPCs but assign **public IPs** to them, you may notice that **you can SSH from one to another without any VPC peering**. Here's why:

### Why This Works

1. **Public IP Access**

   * EC2 instances with public IPs are reachable over the internet.
   * Your EC2 in VPC1 uses the public IP of EC2 in VPC2 to connect, just like any external system.

2. **Security Group Rules**

   * If the target instance (e.g., in VPC2) allows inbound SSH (port 22) from 0.0.0.0/0 or from the source instance's public IP, the connection is allowed.

3. **Internet Gateway Access**

   * Since both instances are in public subnets (with IGWs), they can initiate outbound and receive inbound traffic via their public IPs.

### Why It's Not Recommended

| Concern    | Reason                                                                                    |
| ---------- | ----------------------------------------------------------------------------------------- |
| Security   | Traffic travels over the public internet, even between instances in the same AWS account. |
| Latency    | Introduces unnecessary hops and latency.                                                  |
| Cost       | Data transfer charges may apply for egress traffic.                                       |
| Management | Harder to control, audit, and secure long-term.                                           |

### Better Alternative

* Set up **VPC Peering** between VPC1 and VPC2.
* Use **private IP addresses** for inter-EC2 communication.
* Lock down security groups using **private CIDR ranges** instead of public IPs.

---

## VPC Peering (Intra-Region, Same Account)

### What is It?

VPC peering connects two VPCs so that instances in either VPC can communicate using private IP addresses.

### Use Case

Two applications deployed in different VPCs (within the same region) need to communicate securely.

### Implementation Steps

1. **Go to the VPC console**
2. **Create a Peering Connection**

   * Requester VPC: VPC-A (e.g., 10.0.0.0/16)
   * Accepter VPC: VPC-B (e.g., 10.1.0.0/16)
3. **Accept the Peering Connection**
4. **Update Route Tables**
5. **Modify Security Groups** to allow traffic from the peer VPC

---

## Cross-Region VPC Peering (Same Account)

### What is It?

VPC peering across AWS regions to enable private IP communication.

### Use Case

You have EC2 instances in ap-south-1 and ap-northeast-1 and want them to talk privately.

### Implementation Steps

1. **Initiate VPC Peering (Cross-region)**
2. **Accept Peering in Region-B**
3. **Update Route Tables in Both Regions**
4. **Adjust Security Groups**

---

## Inter-Account VPC Peering

### What is It?

Peering VPCs that belong to **different AWS accounts**, either in the same or different regions.

### Use Case

You separate dev/staging/prod into different AWS accounts but still need network communication.

### Implementation Steps

#### (1) Create Peering from Account-A:

* Go to VPC > Peering > Create Peering
* Select VPC in Account-A
* For accepter, provide the VPC ID and AWS Account ID of Account-B

#### (2) Accept Peering in Account-B:

* Switch to Account-B > VPC > Peering > Accept

#### (3) Update Route Tables:

* In Account-A VPC: add route to B’s CIDR block via peering connection
* In Account-B VPC: add route to A’s CIDR block via peering connection

#### (4) Modify Security Groups:

* Allow inbound traffic from peer VPC’s CIDR block or instance IP

#### (5) Optional:

* Use **Resource Access Manager (RAM)** to share VPC resources across accounts

---

## Transit Gateway (Multi-VPC, Multi-Account, Multi-Region)

### What is It?

A highly scalable hub-and-spoke model to connect multiple VPCs and VPNs across AWS accounts/regions.

### Use Case

* Enterprise-grade network design
* Simplify many-to-many VPC connections

### Implementation Steps (Multi-Account Example):

1. **Create a Transit Gateway in Account-A**
2. **Share Transit Gateway** using AWS RAM with Account-B
3. **Attach VPCs from both accounts** to the Transit Gateway
4. **Update Route Tables in both VPCs** to route via Transit Gateway
5. **Update Security Groups**

> Note: Transit Gateway supports cross-region attachments.

---

## Site-to-Site VPN (Hybrid or Inter-Account Secure Connectivity)

### What is It?

Create a VPN tunnel between VPCs across accounts or to on-prem data centers

### Use Case

Regulated workloads, encryption at all layers

### Implementation Steps

1. Create Virtual Private Gateway in one VPC
2. Create Customer Gateway in the other
3. Create a VPN Connection between them
4. Update routes and SGs

---

## Public IP Based Connectivity (Not Recommended)

### What is It?

Using public IPs for EC2 instances to connect over the internet

### Use Case

Quick testing or if VPC peering is not feasible

### Drawbacks

* Higher latency
* Costlier
* Less secure

### Steps

1. Assign public IP to EC2s
2. Modify security groups to allow traffic from each other's public IP
3. Ensure OS firewall allows it

---

## Summary Table

This table highlights key scenarios with their connectivity method, scope, and relative complexity:

| Scenario                                                         | Account Scope      | Region Scope | Connectivity Method                          | Private IP? | Complexity | Cost Impact                        |
| ---------------------------------------------------------------- | ------------------ | ------------ | -------------------------------------------- | ----------- | ---------- | ---------------------------------- |
| **1. EC2s in Same VPC**                                          | Same               | Same         | Native (no setup)                            | ✅           | Very Low   | Free                               |
| **2. EC2s in Different VPCs (Same Region, Same Account)**        | Same               | Same         | VPC Peering                                  | ✅           | Low        | Minimal (data transfer)            |
| **3. EC2s in Different VPCs (Cross-Region, Same Account)**       | Same               | Cross-Region | Cross-Region VPC Peering                     | ✅           | Medium     | Low to Medium (peering + transfer) |
| **4. EC2s in Different VPCs (Same Region, Different Accounts)**  | Different Accounts | Same         | Inter-Account VPC Peering or Transit Gateway | ✅           | Medium     | Low to Medium                      |
| **5. EC2s in Different VPCs (Cross-Region, Different Accounts)** | Different Accounts | Cross-Region | Transitive via Transit Gateway or VPN        | ✅           | High       | Medium to High                     |
| **6. EC2s via Public IP (Any Mix)**                              | Any                | Any          | Public Internet                              | ❌           | Very Low   | Higher (egress fees)               |

---

## Detailed Implementation Steps

Below are **easy to follow, AWS Console–based steps** for each primary scenario.

### 1. EC2s in Same VPC

*No configuration needed beyond default networking.*

1. **Launch EC2 Instances** in the same VPC and subnet or across subnets.
2. **Ensure Security Groups** allow required traffic (e.g., port 22 for SSH) within the group or VPC CIDR.
3. **Use Private IP** of the target instance to connect: `ssh ec2-user@10.x.x.x`.

> **Result:** Direct, low-latency communication using AWS backend.

---

### 2. EC2s in Different VPCs (Same Region, Same Account)

Use **VPC Peering** to connect instances in two VPCs in the same region.

#### Step-by-Step Implementation

1. **Open the VPC Console**

   * Sign in to the AWS Management Console.
   * In the Services menu, search for **VPC** and click **VPC**.

2. **Create a VPC Peering Connection**

   1. In the left-hand navigation pane, click **Peering Connections**.
   2. Click **Create peering connection**.
   3. For **Peering connection name tag**, enter `VPC-A-to-VPC-B`.
   4. Under **Requester VPC**, click the dropdown and select **VPC-A** (e.g., CIDR `10.0.0.0/16`).
   5. Under **Accepter VPC**, choose **Another VPC in this account** and select **VPC-B** (e.g., CIDR `10.1.0.0/16`).
   6. Leave **Region** as the current region (e.g., `ap-south-1`).
   7. Click **Create peering connection**.

3. **Accept the Peering Connection**

   1. Still in **Peering Connections**, find the new entry with status **Pending Acceptance**.
   2. Select the checkbox next to it, then click **Actions** → **Accept request**.
   3. Confirm by clicking **Accept**.

4. **Configure Route Tables for VPC-A**

   1. In the left pane, click **Route Tables**.
   2. Identify the route table associated with the subnet where EC2-A resides:

      * The **Name** column or **Subnet Associations** tab indicates which subnets are linked.
      * If you used the **Main route table**, it’ll show as **Main** under **Main** column.
   3. Select the correct route table and click on the **Routes** tab below.
   4. Click **Edit routes**, then **Add route**.
   5. In **Destination**, type `10.1.0.0/16` (the CIDR block of VPC-B).
   6. In **Target**, select **Peering connection** and choose the ID (e.g., `pcx-0abcd1234efgh5678`).
   7. Click **Save routes**.

5. **Configure Route Tables for VPC-B**

   1. Switch the **Region selector** (if needed) or stay in the same region, and click **Route Tables** again.
   2. Locate the table used by EC2-B’s subnet.
   3. Repeat the **Edit routes** process:

      * **Destination**: `10.0.0.0/16` (CIDR of VPC-A)
      * **Target**: the same peering connection ID
   4. Click **Save routes**.

6. **Verify Subnet Associations**

   * For each route table, select it and open the **Subnet Associations** tab.
   * Confirm that the subnets containing EC2-A and EC2-B are checked. If not, click **Edit subnet associations**, select the appropriate subnets, and save.

7. **Update Security Groups**

   1. Go to **EC2** → **Security Groups** in the left menu.
   2. Select the security group attached to EC2-A.
   3. Under the **Inbound rules** tab, click **Edit inbound rules** → **Add rule**.

      * **Type**: SSH
      * **Protocol**: TCP
      * **Port range**: 22
      * **Source**: Custom, enter `10.1.0.0/16` (VPC-B CIDR)
   4. Save rules. Repeat for EC2-B’s security group, allowing `10.0.0.0/16`.

8. **Test the Connection**

   * From EC2-A’s terminal, run:

     ```bash
     ssh -i /path/to/key.pem ec2-user@<EC2-B private IP>
     ```
   * If it hangs or fails, check each previous step:

     * Peering status is **Active**
     * Routes exist and are correct
     * Subnets correctly associated
     * Security group rules allow SSH

---

### 3. EC2s in Different VPCs (Cross-Region, Same Account)

Use **Cross-Region VPC Peering**.

1. **VPC Console → Peering Connections → Create Peering Connection**

   * **Requester VPC:** in ap-south-1 (CIDR 10.0.0.0/16)
   * **Accepter VPC:** in ap-northeast-1 (CIDR 10.2.0.0/16)
   * **Enable Cross-Region** option
2. **Accept Peering** in the acce­pted region’s console
3. **Update Route Tables** in both regions with peer CIDR and peering target
4. **Adjust Security Groups** to permit inbound from the peer CIDR blocks
5. **Connect** using private IP of the target in other region

> **Tip:** Consider data transfer costs for cross-region traffic.

---

### 4. EC2s in Different VPCs (Same Region, Different Accounts)

Use **Inter-Account VPC Peering** or **Transit Gateway** (for many VPCs).

#### A. VPC Peering (Best for 1:1)

1. **Account A**: VPC → Peering Connections → Create

   * Requester VPC in Account A
   * Provide Accepter VPC ID and AWS Account ID B
2. **Account B**: Accept the peering
3. **Route Tables**: add cross-account peer routes in both VPCs
4. **Security Groups**: allow peer VPC CIDRs

#### B. Transit Gateway (Best for Many)

1. **Account A**: Create Transit Gateway → Share via AWS RAM with Account B
2. **Both Accounts**: Attach each VPC to the shared Transit Gateway
3. **Route Tables**: in each VPC’s TGW attachment, route other VPC CIDRs
4. **Security Groups**: permit traffic from all peer VPC CIDRs

> **Recommendation:** Use TGW when connecting more than 2 VPCs/accounts.

---

### 5. EC2s in Different VPCs (Cross-Region, Different Accounts)

Prefer **Transit Gateway (cross-region attachments)** or **Site-to-Site VPN**.

#### A. Transit Gateway with Cross-Region Attachments

1. **Account A**: Create TGW in Region-1 → Share to Account B via RAM
2. **Account B**: Accept share → Attach its VPC in Region-2 as a cross-region attachment
3. **Route Tables**: configure to route between VPC CIDRs through TGW
4. **Security Groups**: allow peer VPC CIDR blocks

#### B. Site-to-Site VPN

1. **Account A**: Create Virtual Private Gateway → Attach to VPC-1
2. **Account B**: Create Customer Gateway representing VPC-2
3. **VPN Connection**: between VGW and CGW
4. **Routes and SGs**: add VPN routes and open required ports

> **Note:** VPN adds encryption but more maintenance.

---

## Final Thoughts

* Choose **Same VPC** or **Peering** for simplicity and performance.
* Use **Transit Gateway** for complex, multi-account, multi-region landscapes.
* Resort to **VPN** for regulated or encrypted traffic beyond AWS-managed connectivity.

With these detailed steps, you can confidently implement any EC2-to-EC2 connectivity scenario in your AWS environments.
