# AWS Auto Scaling Apache Web Server Setup

## 🚀 What Is an Auto Scaling Group (ASG)?
An Auto Scaling Group automatically adjusts the number of EC2 instances based on demand. It helps:
- **Scale Out**: Launch new instances when CPU exceeds 70%.
- **Scale In**: Terminate instances when CPU drops below 30%.
- Maintain a minimum/maximum instance count.
- Use scaling policies (CPU, memory, or custom metrics).

---

## ✅ Step-by-Step Guide (AWS CLI)

### 🧩 Step 1: Create a Launch Template
**Purpose**: Define instance configuration for the ASG.

**CLI Command**:
```bash
aws ec2 create-launch-template \
  --launch-template-name my-webserver-template \
  --version-description "v1" \
  --launch-template-data '{
    "ImageId": "ami-0abcdef1234567890",  # Replace with your AMI ID
    "InstanceType": "t2.micro",
    "SecurityGroupIds": ["sg-0123456789abcdef0"],  # Replace with your security group
    "KeyName": "my-keypair",  # Replace with your key pair name
    "UserData": "IyEvYmluL2Jhc2gKc3VkbyB5dW0gaW5zdGFsbCAteSBodHRwZApzeXN0ZW1jdGwgc3RhcnQgaHR0cGQ="  # Base64-encoded script
  }'
```

**UserData Decoded**:
```bash
#!/bin/bash
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
echo "<h1>Hello from Auto Scaling Instance $(hostname)</h1>" > /var/www/html/index.html
```

---

### 🧩 Step 2: Create the Auto Scaling Group
**CLI Command**:
```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name my-webserver-asg \
  --launch-template "LaunchTemplateName=my-webserver-template,Version=1" \
  --min-size 1 \
  --max-size 3 \
  --desired-capacity 1 \
  --vpc-zone-identifier "subnet-12345678,subnet-87654321"  # Replace with your subnets
  --tags "Key=Name,Value=WebServer,PropagateAtLaunch=true"
```

---

### 🧩 Step 3: Attach a Target Tracking Scaling Policy
**CLI Command**:
```bash
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-webserver-asg \
  --policy-name cpu-scale-policy \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
      "PredefinedMetricSpecification": {
        "PredefinedMetricType": "ASGAverageCPUUtilization"
      },
      "TargetValue": 50.0,
      "DisableScaleIn": false
    }'
```

---

### 🧪 Step 4: Test Auto Scaling
1. **Simulate High CPU**:
   ```bash
   ssh -i my-keypair.pem ec2-user@<public-ip>
   sudo amazon-linux-extras install epel -y
   sudo yum install stress -y
   stress --cpu 2 --timeout 300
   ```
2. **Verify Scaling**:
   - Check CloudWatch → EC2 → Auto Scaling Groups.
   - Observe new instances launch up to 3 nodes.

---

### 🧹 Step 5: Clean Up Resources
```bash
aws autoscaling delete-auto-scaling-group --auto-scaling-group-name my-webserver-asg --force-delete
aws ec2 delete-launch-template --launch-template-name my-webserver-template
```

---

## 📌 Important Notes
- **Replace placeholders**: Update AMI ID, subnet IDs, and security groups with your AWS resources.
- **Integrate with ALB**: Add a Load Balancer for traffic distribution.
- **Advanced Policies**: Use CloudWatch alarms for step scaling policies.

---

## 🧠 Recap Table
| Step | Action | CLI Command |
|------|--------|-------------|
| 1️⃣  | Create launch template | `aws ec2 create-launch-template` |
| 2️⃣  | Define ASG | `aws autoscaling create-auto-scaling-group` |
| 3️⃣  | Set scaling policy | `aws autoscaling put-scaling-policy` |
| 4️⃣  | Test scaling | Stress test + monitor CloudWatch |
| 5️⃣  | Cleanup | Delete ASG and template |
