# AWS Auto Scaling with ALB and Target Tracking

## 🎯 Objective
Deploy an auto-scaling Apache web server using:
- **Application Load Balancer (ALB)** for traffic distribution
- **Target Tracking Scaling Policy** to maintain average CPU at 50%
- **Min: 1 | Desired: 1 | Max: 3** instances

---

## ✅ Step-by-Step Guide

### 🧩 Step 1: Create Launch Template
**Purpose**: Define instance configuration for the ASG.

**How to do it**:
1. Go to **EC2 → Launch Templates**.
2. Click **Create launch template**.
3. Configure:
   - **Name**: `tt-webserver-template`
   - **AMI**: Amazon Linux 2
   - **Instance Type**: `t2.micro`
   - **Key Pair**: Select an existing key pair
4. **Security Group**: Allow ports **22 (SSH)** and **80 (HTTP)**.
5. **User Data** (installs Apache and creates index.html):
   ```bash
   #!/bin/bash
   yum update -y
   yum install -y httpd
   systemctl start httpd
   systemctl enable httpd
   echo "<h1>Hello from $(hostname)</h1>" > /var/www/html/index.html
   ```
6. Click **Create launch template**.

---

### 🧩 Step 2: Create Target Group
1. Go to **EC2 → Target Groups**.
2. Click **Create Target Group**.
3. Configure:
   - **Name**: `tt-webserver-target-group`
   - **Protocol/Port**: HTTP/80
   - **VPC**: Select your default VPC
4. Set health checks to **HTTP:80**.
5. Click **Create target group**.

---

### 🧩 Step 3: Create Application Load Balancer (ALB)
1. Go to **EC2 → Load Balancers**.
2. Click **Create Load Balancer** → **Application Load Balancer**.
3. Configure:
   - **Name**: `tt-webserver-alb`
   - **Scheme**: Internet-facing
   - **Listeners**: HTTP:80
   - **Availability Zones**: Select 2 subnets across AZs
   - **Security Group**: Allow HTTP traffic (port 80)
4. Under **Routing**, select the **tt-webserver-target-group**.
5. Click **Create**.

---

### 🧩 Step 4: Create Auto Scaling Group (ASG)
1. Go to **EC2 → Auto Scaling Groups**.
2. Click **Create Auto Scaling Group**.
3. Configure:
   - **Name**: `tt-webserver-asg`
   - **Launch Template**: `tt-webserver-template` (Version 1)
   - **VPC**: Select same VPC + 2 subnets
4. **Load Balancing**:
   - Attach to **tt-webserver-alb**
   - Select **tt-webserver-target-group**
5. **Scaling Settings**:
   - **Desired**: 1
   - **Min**: 1
   - **Max**: 3
6. **Scaling Policy**:
   - Choose **Target tracking scaling policy**
   - **Metric**: Average CPU Utilization
   - **Target**: 50%
7. Click **Create Auto Scaling Group**.

---

### 🔍 Step 5: Test the Setup
1. **Get ALB DNS**:
   - Go to **EC2 → Load Balancers**.
   - Copy the **DNS name** of your ALB.
   - Visit in browser: `http://<alb-dns-name>`  
   Expected output: `Hello from ip-xxx-xxx-xxx-xxx`

2. **Simulate High CPU**:
   ```bash
   ssh -i your-key.pem ec2-user@<public-ip>
   sudo amazon-linux-extras install epel -y
   sudo yum install stress -y
   stress --cpu 2 --timeout 300
   ```

3. **Monitor Scaling**:
   - Go to **Auto Scaling Groups → Activity**.
   - Watch new instances launch when CPU exceeds 50%.

---

### 🧹 Step 6: Clean Up Resources
1. Delete the Auto Scaling Group (**EC2 → Auto Scaling Groups**).
2. Delete the Launch Template (**EC2 → Launch Templates**).
3. Delete the Target Group (**EC2 → Target Groups**).
4. Delete the ALB (**EC2 → Load Balancers**).
5. Terminate unused instances (**EC2 → Instances**).

---

## 🧠 Why Target Tracking Scaling?
| Benefit              | Description |
|----------------------|-------------|
| 🔄 Auto-Scaling      | Automatically adjusts instance count to maintain target CPU (50%). |
| ⚙️ No Manual Alarms  | AWS handles metric calculations internally. |
| 🔥 Bursty Workloads  | Ideal for web apps with fluctuating traffic. |

---

## 📌 Important Notes
- Replace placeholders (AMI IDs, subnet IDs, etc.) with your AWS resources.
- Ensure security groups allow inbound traffic on ports **22 (SSH)** and **80 (HTTP)**.
- For production, consider HTTPS and more robust health checks.
