# AWS Auto Scaling Apache Web Server Setup

## 🚀 What Is an Auto Scaling Group (ASG)?
An Auto Scaling Group automatically adjusts the number of EC2 instances based on demand. It helps:
- **Scale Out**: Launch new instances when CPU exceeds 70%.
- **Scale In**: Terminate instances when CPU drops below 30%.
- Maintain a minimum/maximum instance count.
- Use scaling policies (CPU, memory, or custom metrics).

---

## 🎯 Goal
Deploy a basic web server (Apache) that automatically scales out when CPU > 70% and scales in when CPU < 30%.

---

## ✅ Step-by-Step Guide

### 🧩 Step 1: Create a Launch Template
**Purpose**: Defines how new EC2 instances in the Auto Scaling Group will be launched.

**How to do it**:
1. Go to the **AWS Management Console**.
2. Navigate to **EC2 → Launch Templates**.
3. Click **Create launch template**.
4. Fill in:
   - **Launch template name**: `webserver-template`
   - **AMI**: Amazon Linux 2 (or region-specific default)
   - **Instance type**: `t2.micro` (free-tier eligible)
   - **Key pair**: Select an existing key pair for SSH access
   - **Security Group**: Allow ports **22 (SSH)** and **80 (HTTP)**
5. In **Advanced Details → User data**, paste this script:

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from Auto Scaling Instance $(hostname)</h1>" > /var/www/html/index.html
```

6. Click **Create launch template**.

---

### 🧩 Step 2: Create the Auto Scaling Group
**Purpose**: Define the Auto Scaling Group (ASG) using the launch template.

**How to do it**:
1. Go to **EC2 → Auto Scaling Groups**.
2. Click **Create Auto Scaling group**.
3. Configure:
   - **Name**: `webserver-asg`
   - **Launch template**: Select `webserver-template`
4. **Network settings**:
   - Choose your VPC and at least 2 subnets across different AZs
5. **Group size**:
   - **Desired capacity**: 1
   - **Minimum capacity**: 1
   - **Maximum capacity**: 3
6. **Scaling policies**:
   - Select **Target tracking scaling policy**
   - **Metric type**: Average CPU utilization
   - **Target value**: 50
7. Review and create the ASG.

---

### 🧩 Step 3: Confirm Instance Launch
1. Check **EC2 → Instances** for the new instance.
2. Use the **Public IPv4 address** to access the web server:
   ```
   http://<public-ip>
   ```
3. Verify the page displays:
   ```
   Hello from Auto Scaling Instance ip-xxx-xxx-xxx-xxx
   ```

---

### 🧪 Step 4: Test Auto Scaling (Scale Out)
**Simulate high CPU usage**:
1. SSH into the instance:
   ```bash
   ssh -i your-key.pem ec2-user@<instance-public-ip>
   ```
2. Install and run stress test:
   ```bash
   sudo amazon-linux-extras install epel -y
   sudo yum install stress -y
   stress --cpu 2 --timeout 300
   ```
3. Wait 5-10 minutes. The ASG will add instances up to the max capacity (3).

---

### 🧪 Step 5: Test Auto Scaling (Scale In)
1. Stop the stress test (Ctrl+C in SSH session).
2. Wait 5-10 minutes. The ASG will reduce instances back to the minimum (1).

---

### 🧹 Step 6: Clean Up Resources
1. Delete the **Auto Scaling Group** (EC2 → Auto Scaling Groups).
2. Delete the **Launch Template** (EC2 → Launch Templates).
3. Terminate unused instances (EC2 → Instances).

---

## 🧠 Recap Table
| Step | Action |
|------|--------|
| 1️⃣  | Create launch template with Apache setup script |
| 2️⃣  | Create ASG using the template with scaling policies |
| 3️⃣  | Verify instance and web server functionality |
| 4️⃣  | Test scaling out under high CPU load |
| 5️⃣  | Confirm scaling in when CPU drops |
| 6️⃣  | Clean up resources to avoid costs |
