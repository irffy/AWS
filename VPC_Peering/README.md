# VPC Peering between two VPC in same AWS account:

VPC Peering is a networking connection between two Virtual Private Clouds (VPCs) that allows them to communicate as if they were part of the same network. This is particularly useful when you want to securely connect VPCs that are either in the same AWS account or across different AWS accounts and regions.

What is VPC Peering?
VPC Peering: A one-to-one network connection between two VPCs that allows resources (like EC2 instances) in different VPCs to communicate with each other privately.

Use Cases:
Connecting production and development environments.
Connecting VPCs across different AWS accounts or regions.
Sharing services (like databases) between VPCs.

How to Set Up VPC Peering (Practical Walkthrough)
Step 1: Create Two VPCs
Go to the VPC Dashboard in AWS.
Create VPC #1:
CIDR Block: 10.0.0.0/16
Create a public subnet: 10.0.1.0/24
Create VPC #2:
CIDR Block: 10.1.0.0/16
Create a public subnet: 10.1.1.0/24
Step 2: Set Up VPC Peering
In the VPC Dashboard, go to Peering Connections.
Click Create Peering Connection.
Requester VPC: Select VPC #1.
Accepter VPC: Select VPC #2.
Once created, go to Peering Connections and select the newly created peering connection.
Click Actions > Accept Request.
Step 3: Update Route Tables
To enable communication between the VPCs, you need to update the route tables in both VPCs.

In VPC #1 (10.0.0.0/16):
Go to Route Tables and select the route table for VPC #1.
Add a new route:
Destination: 10.1.0.0/16 (CIDR of VPC #2).
Target: The Peering Connection.
In VPC #2 (10.1.0.0/16):
Go to Route Tables and select the route table for VPC #2.
Add a new route:
Destination: 10.0.0.0/16 (CIDR of VPC #1).
Target: The Peering Connection.
Step 4: Update Security Groups
For instances to communicate across the VPCs, you need to modify the security groups to allow traffic.

Go to the EC2 Dashboard and select the security group attached to the instances in both VPCs.
In both security groups:
Add an Inbound Rule:
Type: Custom TCP Rule (or specific service like SSH, HTTP, etc.)
Source: The CIDR block of the other VPC (e.g., 10.1.0.0/16 for VPC #1, 10.0.0.0/16 for VPC #2).
Step 5: Test the VPC Peering
Launch EC2 Instances:
Launch one instance in VPC #1 in the 10.0.1.0/24 subnet.
Launch another instance in VPC #2 in the 10.1.1.0/24 subnet.
Test Connectivity:
SSH into the instance in VPC #1.
From that instance, ping the private IP address of the instance in VPC #2.
If the peering connection is working correctly, you should get a response from the ping.
bash
Copy code
ping <Private-IP-of-Instance-in-VPC2>
Similarly, SSH into the instance in VPC #2 and try to ping the private IP address of the instance in VPC #1.
