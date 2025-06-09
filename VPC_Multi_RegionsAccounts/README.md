# Connecting EC2 Instances Across VPCs, Regions, and AWS Accounts

![Flow Diagram of VPC and EC2 Connectivity](flow-diagram)

In AWS, networking plays a critical role in enabling secure, scalable, and cost-effective communication between your resources. This guide blends conceptual explanations with **step-by-step, console-based walkthroughs** placed directly in each section to ensure clarity from first glance.

In AWS, networking plays a critical role in enabling secure, scalable, and cost-effective communication between your resources. This guide blends conceptual explanations with **step-by-step, console-based walkthroughs** placed directly in each section to ensure clarity from first glance.

---

## 1. EC2s in Same VPC

**Concept:** Instances within the same VPC communicate over AWS’s private network using their private IPs—no additional setup required.

**Implementation Steps:**

1. **Launch EC2 instances** in the same VPC and subnet (or different subnets within that VPC).
2. **Security Groups:**

   * Navigate to **EC2 → Security Groups**.
   * Select the group attached to your instances, click **Edit inbound rules**.
   * Add a rule:

     * **Type:** SSH (or your application port)
     * **Source:** Custom → Enter the VPC’s CIDR (e.g., `10.0.0.0/16`) or the security group itself.
3. **Connect:** Use the private IP to SSH or call your service:

   ```bash
   ssh -i key.pem ec2-user@<Private-IP>
   ```

**Result:** Secure, low-latency communication entirely within AWS’s backbone.

---

## 2. EC2s in Different VPCs (Same Region, Same Account)

**Concept:** VPC Peering links two VPCs so traffic between them uses private IPs over AWS’s internal network.

**Step-by-Step Walkthrough:**

### A. Create and Accept Peering

1. **Open VPC Console** → **Peering Connections** → **Create peering connection**.
2. **Name tag:** `VPC-A-to-VPC-B`.
3. **Requester VPC:** Select **VPC-A** (CIDR `172.31.0.0/16`) .
4. **Accepter VPC:** Choose **Another VPC in this account**, select **VPC-B** (CIDR `10.0.0.0/24`) .
5. Click **Create peering connection**.
6. Select the pending connection → **Actions** → **Accept request**.

### B. Configure Route Tables

1. **Identify route tables**:

   * In **VPC → Route Tables**, locate the table associated with EC2-A’s subnet (check **Subnet Associations**).
2. **Add route in VPC-A’s table**:

   * **Edit routes** → **Add route**
   * **Destination:** `10.0.0.0/24`
   * **Target:** Select **Peering Connection**, pick the new ID `pcx-...` → **Save routes**.
3. **Repeat for VPC-B**:

   * Destination: `172.31.0.0/16`, Target: same peering ID.
4. **Verify Subnet Associations**: Ensure each table is linked to the correct subnet.

### C. Security Groups

1. **EC2-A’s SG** → **Edit inbound rules**:

   * Type: SSH, Source: `10.1.0.0/16` → **Save**.
2. **EC2-B’s SG** similarly allows `10.0.0.0/16`.

### D. Test Connection

```bash
ssh -i key.pem ec2-user@<EC2‑B‑private‑IP>
```

**Common Pitfalls:** Missing routes, wrong table/subnet associations, NACL blocking, SG misconfiguration, overlapping CIDRs.

---

## 3. EC2s in Different VPCs (Cross-Region, Same Account)

**Concept:** Cross-region peering extends VPC peering across AWS regions.

**Steps:**

1. **VPC Console → Peering Connections → Create**

   * Requester: VPC-A (`ap-south-1`, `10.0.0.0/16`)
   * Accepter: VPC-B (`ap-northeast-1`, `10.2.0.0/16`)
   * Enable **Cross-Region**.
2. **Accept** in the accepter region.
3. **Route Tables**:

   * In both VPCs, add routes for the peer’s CIDR via the peering ID.
4. **Security Groups**: Allow peer VPC CIDR inbound.
5. **Connect** via private IP.

> **Tip:** Monitor cross-region data charges.

---

## 4. EC2s in Different VPCs (Same Region, Different Accounts)

**Concept:** Inter-account peering or Transit Gateway for scalable multi-account setups.

### A. Inter-Account VPC Peering

1. **Account A → VPC → Peering Connections → Create**

   * Requester: VPC-A, Accepter: Provide VPC ID + Account B ID.
2. **Account B → Accept**.
3. **Route Tables**: Add peer routes in each VPC.
4. **SGs**: Allow peer CIDR inbound.

### B. Transit Gateway (for many VPCs)

1. **Account A → Create Transit Gateway**.
2. **Share via AWS RAM**: Send share with Account B.
3. **Attach VPCs** (A & B) to TGW.
4. **Route Tables** (TGW attachments): Add other VPC CIDRs.
5. **SGs**: Permit traffic from peer CIDRs.

> **Use TGW** when connecting more than two VPCs/accounts.

---

## 5. EC2s in Different VPCs (Cross-Region, Different Accounts)

**Concept:** You can use **VPC Peering** even across **regions** and **AWS accounts**, but there are important considerations and limitations.

### A. Cross-Account, Cross-Region VPC Peering

1. **Initiate Peering from Account A:**

   * Go to **VPC Console** → **Peering Connections** → **Create Peering Connection**.
   * Name: `A-to-B-crossregion-peering`
   * **Requester VPC:** Choose VPC in Account A (e.g., `172.31.0.0/16`, `ap-south-1`).
   * **Accepter VPC:** Enter **Account ID** and **VPC ID** of Account B (e.g., `10.0.0.0/24`, `ap-northeast-1`).
   * Enable **cross-region** checkbox.

2. **Account B Accepts the Peering Request:**

   * Switch to Account B.
   * Navigate to **VPC → Peering Connections** → **Accept** the pending request.

3. **Update Route Tables** in Both VPCs:

   * In each VPC’s route table, add a route:

     * Destination: peer VPC CIDR
     * Target: the peering connection ID

4. **Update Security Groups:**

   * Add inbound rules allowing traffic from the other VPC’s CIDR.

### B. When to Use Transit Gateway or VPN

While VPC peering works in this scenario, it may not scale well due to these limitations:

* No transitive routing
* Manual route/security management

So, if you're planning to connect **many VPCs across regions/accounts**, consider:

#### Option 1: **Transit Gateway (TGW) Cross-Region Attachments**

* Central hub to interconnect VPCs and accounts.
* Easier to scale and manage.

#### Option 2: **Site-to-Site VPN**

* Encrypted connection.
* Best suited when VPC peering isn’t possible or for hybrid setups.

> **Tip:** Always avoid overlapping CIDRs to ensure successful peering.

## 6. EC2s via Public IP (Any Mix)

**Concept:** Uses the internet to connect—simple but insecure.

**Implementation:**

1. Assign public IPs.
2. SGs: Allow inbound from the other instance’s public IP.
3. Connect via public DNS/IP.

**Drawbacks:** Public exposure, latency, costs, compliance risk.

---

## Summary Table

| Scenario                                          | Scope (Account/Region)   | Method                                   | Setup Complexity | Cost Impact     |
| ------------------------------------------------- | ------------------------ | ---------------------------------------- | ---------------- | --------------- |
| EC2s in Same VPC                                  | Same / Same              | Native (private IP)                      | Very Low         | Free            |
| Different VPCs (Same Region, Same Account)        | Same / Same              | VPC Peering                              | Low              | Minimal         |
| Different VPCs (Cross-Region, Same Account)       | Same / Cross-Region      | Cross-Region Peering                     | Medium           | Low–Medium      |
| Different VPCs (Same Region, Different Accounts)  | Different / Same         | Inter-Account Peering or Transit Gateway | Medium           | Low–Medium      |
| Different VPCs (Cross-Region, Different Accounts) | Different / Cross-Region | TGW Cross-Region or VPN                  | High             | Medium–High     |
| EC2s via Public IP                                | Any / Any                | Public Internet                          | Very Low         | Higher (egress) |

---

With these **in-section, detailed steps**, newcomers can immediately find both the rationale and precise console actions needed for each scenario. Let me know if any section needs further expansion or visual aids!
