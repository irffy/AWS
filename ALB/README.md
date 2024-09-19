# Elastic loadbalancer

## ALB using three EC2 instances as pool members:

To set up an Application Load Balancer (ALB) in AWS to distribute traffic among your three EC2 instances running an HTTPD web server, follow these steps:

Step-by-Step Guide to Set Up Application Load Balancer (ALB)
1. Create an Application Load Balancer
Navigate to the EC2 Dashboard:

Go to the AWS Management Console and open the EC2 Dashboard.
Select Load Balancers:

In the left navigation pane, scroll down to Load Balancing and click on Load Balancers.
Create Load Balancer:

Click on the Create Load Balancer button.
Select Application Load Balancer.
Configure the Load Balancer:

Name: Provide a name for your ALB.
Scheme: Choose whether it will be Internet-facing (public) or Internal (private).
IP address type: Select IPv4.
Listeners: The default listener will be on port 80 (HTTP). You can add additional listeners (e.g., HTTPS on port 443) later if needed.
Availability Zones: Select the VPC and the Availability Zones where your EC2 instances are running. Make sure to enable at least two subnets in different Availability Zones for high availability.
Configure Security Groups:

Choose an existing security group or create a new one that allows inbound HTTP (port 80) traffic. If you plan to use HTTPS, allow port 443 as well.
Configure Routing:

Target Group: Create a new target group for your instances.
Target Group Name: Provide a name.
Target Type: Choose Instances.
Protocol: Set to HTTP and the port (default is 80).
Health Check Protocol: Set to HTTP and specify a health check path (e.g., / or /health).
After configuring the target group settings, click on Next.
Register Targets:

Select your running EC2 instances that are running the HTTPD web server.
Click on Add to registered targets, then Next.
Review and Create:

Review all configurations and click on Create to create the Application Load Balancer.
2. Update Security Group of EC2 Instances
Modify Security Group:
Ensure that the security group attached to your EC2 instances allows inbound traffic from the ALB security group.
Go to the EC2 Dashboard and click on Security Groups.
Select the security group used by your EC2 instances.
Add an inbound rule allowing HTTP (port 80) traffic from the ALB security group.
3. Testing the Application Load Balancer
Obtain the ALB DNS Name:

Once your ALB is created, go back to the Load Balancers section in the EC2 dashboard.
Select your newly created ALB and find the DNS name listed in the description.
Access the Application:

Open a web browser and enter the ALB DNS name. This should distribute incoming requests to your registered EC2 instances.
Check if the HTTPD server is responding from all instances by refreshing the page multiple times to see if the load balancer is correctly routing traffic.