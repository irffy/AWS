AWS Three Tier Architecture – Web Tier, App Tier and DB Tier:

AWS services used:
* VPC – Subnet, Route table, Internet Gateway, Nat gateway, Security Group, NACL
* EC2 – ASG, Launch template, ALB, Target group
* RDS – MySQL engine  

Network design:
To create a three-tier network with two availability zones (AZs) using the VPC CIDR 172.0.0.0/20 (which provides up to 4096 IP addresses), you can divide this into subnets where each subnet has 256 IP addresses (/24 subnets). You want one public and two private subnets per AZ, which will result in two public subnets and four private subnets.
Here’s the recommended subnetting plan:
VPC CIDR: 172.0.0.0/20
* Total range: 172.0.0.0 to 172.0.15.255
Subnet CIDRs (each with 256 IPs)
Availability Zone 1 (AZ1)
1. Public Subnet 1 (AZ1): 172.0.0.0/24
o IP Range: 172.0.0.0 - 172.0.0.255
2. Private Subnet 1 (AZ1 - Tier 1): 172.0.1.0/24
o IP Range: 172.0.1.0 - 172.0.1.255
3. Private Subnet 2 (AZ1 - Tier 2): 172.0.2.0/24
o IP Range: 172.0.2.0 - 172.0.2.255
Availability Zone 2 (AZ2)
1. Public Subnet 2 (AZ2): 172.0.3.0/24
o IP Range: 172.0.3.0 - 172.0.3.255
2. Private Subnet 1 (AZ2 - Tier 1): 172.0.4.0/24
o IP Range: 172.0.4.0 - 172.0.4.255
3. Private Subnet 2 (AZ2 - Tier 2): 172.0.5.0/24
o IP Range: 172.0.5.0 - 172.0.5.255
Summary of the Plan:
* Two public subnets (one per AZ).
* Four private subnets (two tiers per AZ).
* Each subnet has 256 IP addresses.
* You still have additional unused IP ranges within the VPC CIDR for future subnet expansion

EC2 Userdata for Web Tier:

#!/bin/bash

#install apache
sudo yum -y install httpd

#enable and start apache
sudo systemctl enable httpd
sudo systemctl start httpd

sudo echo '<!DOCTYPE html>

<html lang="en">
	<head>
		<meta charset="utf-8" />
		<meta name="viewport" content="width=device-width, initial-scale=1" />

		<title>A Basic HTML5 Template</title>

		<link rel="preconnect" href="https://fonts.googleapis.com" />
		<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
		<link
			href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700;800&display=swap"
			rel="stylesheet"
		/>

		<link rel="stylesheet" href="css/styles.css?v=1.0" />
	</head>

	<body>
		<div class="wrapper">
			<div class="container">
				<h1>Welcome! An Apache web server has been started successfully.</h1>
				<p>Replace this with your own index.html file in /var/www/html.</p>
			</div>
		</div>
	</body>
</html>

<style>
	body {
		background-color: #34333d;
		display: flex;
		align-items: center;
		justify-content: center;
		font-family: Inter;
		padding-top: 128px;
	}

	.container {
		box-sizing: border-box;
		width: 741px;
		height: 449px;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: flex-start;
		padding: 48px 48px 48px 48px;
		box-shadow: 0px 1px 32px 11px rgba(38, 37, 44, 0.49);
		background-color: #5d5b6b;
		overflow: hidden;
		align-content: flex-start;
		flex-wrap: nowrap;
		gap: 24;
		border-radius: 24px;
	}

	.container h1 {
		flex-shrink: 0;
		width: 100%;
		height: auto; /* 144px */
		position: relative;
		color: #ffffff;
		line-height: 1.2;
		font-size: 40px;
	}
	.container p {
		position: relative;
		color: #ffffff;
		line-height: 1.2;
		font-size: 18px;
	}
</style>
' > /var/www/html/index.html



EC2 Userdata for App Tier:

#!/bin/bash
# Add MySQL repository for Amazon Linux 2023 (based on RHEL 9)
cat <<EOF > /etc/yum.repos.d/mysql.repo
[mysql80-community]
name=MySQL 8.0 Community Server
baseurl=https://repo.mysql.com/yum/mysql-8.0-community/el/9/x86_64/
enabled=1
gpgcheck=0
EOF

# Clean the yum cache
yum clean all

# Install MySQL client without GPG check
yum install mysql-community-client --nogpgcheck -y

# Verify installation
mysql --version
