# EC2 instance additional settings/actions

## EC2 instance Termination Protection:
EC2 instance termination protection is a feature that helps prevent accidental termination of your instance. When enabled, it ensures that the instance cannot be terminated via the AWS Management Console, AWS CLI, or API unless the protection is first disabled.

How to Enable Termination Protection:
1. Using the AWS Management Console:
 - Go to the EC2 Dashboard.
 - Select the instance you want to protect.
 - In the Actions dropdown, choose Instance Settings > Change Termination Protection.
 - Set Enable to protect the instance.
 - Click Save.

2. Using the AWS CLI:
Run the following command to enable termination protection:
aws ec2 modify-instance-attribute --instance-id <instance-id> --no-disable-api-termination

How to Disable Termination Protection:
Go to the Instance Settings again, and follow the same steps, but choose Disable termination protection.
Via CLI, you can use:
aws ec2 modify-instance-attribute --instance-id <instance-id> --disable-api-termination

Once enabled, any attempt to terminate the instance will result in an error until the termination protection is disabled.

## EC2 instance edit user data:
To edit user data on an EC2 instance, you typically modify the "user data" script associated with the instance. This script runs only once, when the instance is first launched, unless the instance is manually configured to run the script on every boot.

Here’s how you can edit or update user data for an EC2 instance:

Using the AWS Management Console:
	Go to the EC2 Dashboard.
	Select the instance for which you want to edit the user data.
	Click on Actions > Instance Settings > Edit User Data.
	Modify the script as required.
	Save the changes.

## EC2 instance connect to RDS database:
The "EC2 Actions → Networking → Connect to RDS database" option is a feature in the AWS Management Console designed to simplify the process of connecting an EC2 instance to an RDS database by automatically configuring necessary networking settings. However, this option doesn't directly establish the connection between your EC2 instance and RDS but helps to ensure that security groups and network configurations are set up correctly.

How to Use "Connect to RDS Database" in EC2 Console:
Navigate to the EC2 Dashboard:

In the AWS Management Console, go to the EC2 Dashboard.
Find and select the EC2 instance you want to connect to the RDS database.
Go to Networking → Connect to RDS Database:

Click on Actions (with the selected EC2 instance).
Under Networking, choose Connect to RDS Database.
Choose an RDS Database:

A list of your available RDS databases will appear. Select the RDS instance that you want the EC2 instance to connect to.
Verify Security Group Settings:

AWS will automatically check if the security groups of both the EC2 instance and the RDS instance allow communication.
If necessary, AWS will suggest modifications to the security group rules, such as allowing the correct port (e.g., 3306 for MySQL) in the RDS security group for the EC2 instance's IP address or security group.
Accept the Changes:

If the security groups need to be modified, accept the suggested changes. AWS will automatically update the security group rules to allow the EC2 instance to connect to the RDS instance.
Test the Connection:

Once the networking settings have been updated, you can test the connection using the method described earlier (e.g., using a MySQL or PostgreSQL client installed on the EC2 instance).
The "Connect to RDS Database" option ensures that the network traffic between the EC2 instance and the RDS instance is allowed, but you still need to initiate the actual database connection (e.g., via a database client or application).
After the Configuration:
Once the networking connection is configured, you can connect to the RDS database from your EC2 instance as follows:

For MySQL:
mysql -h <RDS_ENDPOINT> -u <DB_USERNAME> -p<DB_PASSWORD> <DB_NAME>
	
## EC2 instance Launch more like this
The "Launch More Like This" option in the AWS EC2 Management Console allows you to quickly launch additional EC2 instances with the same configuration as an existing instance. This option is helpful when you want to scale out or replicate an instance setup.

How to Use the "Launch More Like This" Feature:
Navigate to the EC2 Dashboard:

Go to the AWS Management Console.
Open the EC2 Dashboard.
Select the Running Instance:

In the list of EC2 instances, select the running instance you want to replicate.
Use the "Launch More Like This" Option:

With the instance selected, click on Actions (from the top).
Under the Instance Settings, choose Launch More Like This.
Review the Instance Configuration:

The configuration wizard will appear, pre-populating most of the settings based on the selected instance. This includes:
AMI (Amazon Machine Image)
Instance type (e.g., t2.micro, t3.medium)
VPC and Subnet
Security Group settings
Storage configuration
Key pair settings (for SSH access)
Review each step and adjust as needed. For example, you may want to change the number of instances or use a different key pair.
Adjust Any Custom Settings (Optional):

If you want to modify specific parameters like instance size, storage, or security groups, you can do so in this wizard before launching the instance.
Ensure the networking settings (such as subnets, security groups, etc.) are appropriate for the new instance.
Launch the Instance:

Once you’ve reviewed and adjusted the settings, click Launch.
You may be prompted to select an existing key pair or create a new one if SSH access is needed.
Monitor the New Instance:

After launching, go to the Instances section in the EC2 Dashboard to monitor the status of the newly launched instance(s).
Key Considerations:
Security Groups: If you need the same network permissions as the original instance, ensure that the security groups are configured identically.
Elastic IP: The newly launched instance will not have the same Elastic IP as the original unless you manually assign one.
Auto Scaling: If you’re replicating instances as part of a scaling operation, consider using Auto Scaling Groups (ASG) to automate this process.
Would you like help with setting up more instances or configuring an Auto Scaling Group?
	
## Create AMI from running EC2 instance:
To create an Amazon Machine Image (AMI) from a running EC2 instance, follow these steps. This process allows you to capture the current state of the instance, including installed software, configuration settings, and attached volumes, so that you can launch new instances with the same configuration.

Steps to Create an AMI from a Running EC2 Instance
Navigate to the EC2 Dashboard:

Go to the AWS Management Console.
Open the EC2 Dashboard.
Select the Running Instance:

In the list of EC2 instances, locate and select the running instance from which you want to create an image (AMI).
Create Image (AMI):

With the instance selected, click on Actions.
Under Image and templates, choose Create Image.
Configure the AMI Settings: In the "Create Image" window, configure the following settings:

Image Name: Provide a meaningful name for the image (e.g., MyApp-Image-September2024).
Image Description: Optionally, provide a description of the image (e.g., "Image of EC2 instance with PHP and MySQL installed").
No Reboot (optional): By default, AWS reboots the instance to ensure file system integrity before creating the image. You can select No Reboot if you don’t want to reboot the instance, but this can risk data integrity for in-memory processes.
Root Device Volume: You can configure the size of the root volume and add any additional EBS volumes you want to include in the AMI. Modify the volume settings if needed (e.g., size, type, or whether to delete the volume on termination).
Create the AMI:

After configuring the options, click Create Image.
Monitor Image Creation:

After you initiate the image creation process, go to the AMIs section in the EC2 dashboard (under Images > AMIs).
The AMI creation process may take a few minutes, depending on the size of the instance’s data. During this time, the status will be shown as Pending.
Once the image is created, the status will change to Available.
Launch New Instances from the AMI:

Once your AMI is ready, you can launch new EC2 instances based on this AMI.
Go to Images > AMIs, select your AMI, and click Launch.
