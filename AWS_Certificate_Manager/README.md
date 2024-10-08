# AWS Certificate Manager

This section goes through how to create a Private Root CA, Intermediate(Sub-ordinate) CA and then request new private cert issued by Private CA:

Here's a step-by-step guide on how to create a Private Root CA, Intermediate CA (subordinate CA), and how to request a new private certificate using the Private Root CA hierarchy in AWS Certificate Manager Private Certificate Authority (ACM Private CA).

1. Create a Private Root CA:

Step 1: Open AWS Certificate Manager Private CA
Navigate to the AWS Management Console.
Go to Certificate Manager and choose Private CA.
Step 2: Create a Private Root CA
Click Create a CA.
Select Root CA and click Next.
Step 3: Configure CA Details
Certificate Authority Type: Select RSA 2048 (or another key size depending on your needs).

Subject Details: Enter information for the certificate. These are typically your organization’s details. For example:

Common Name (CN): awstraining-rootca.com
Organization (O): AlChemists
Country (C): India
State (ST): Karnataka
Locality (L): Bangalore
Click Next.

Step 4: Configure CA Permissions
In the CA permissions section, choose an IAM role that will have permission to manage this CA.
You can skip the Revocation configuration for now.
Step 5: Review and Create
Review your settings and click Create CA.
Once created, the Private Root CA will be in a pending state and requires activation by creating a certificate for it.
Step 6: Activate the Root CA
Select the Private Root CA you created and click Actions > Install CA Certificate.
Choose Self-signed Certificate and click Next.
Review and confirm the self-signed certificate creation.
Your Private Root CA is now active.

2. Create a Private Intermediate (Subordinate) CA:

Step 1: Create Subordinate CA
Go back to AWS Certificate Manager Private CA.
Click Create a CA.
This time, select Subordinate CA and click Next.
Step 2: Configure Subordinate CA Details
Certificate Authority Type: Select RSA 2048 (or another key size if desired).

Subject Details: Enter information for the subordinate CA:


Common Name (CN): awstraining-intermediateca.com
Organization (O): AlChemists
Country (C): India
State (ST): Karnataka
Locality (L): Bangalore
Click Next.

Step 3: Configure CA Permissions and Skip Revocation Configuration:

Same as with the root CA, choose an IAM role and skip revocation configuration.
Step 4: Review and Create Subordinate CA
Review all settings and click Create CA.
Step 5: Sign the Subordinate CA Certificate with Root CA
Once the subordinate CA is created, it will also be in a pending state and needs to be signed by your Root CA.
Step 6: Download the CSR for Subordinate CA
Select the newly created Subordinate CA.
Click Actions > Get CSR and download the CSR (Certificate Signing Request).
Step 7: Sign the CSR with the Root CA
Go back to the Private Root CA that you created earlier.
Select the Root CA, click Actions > Issue Certificate.
Choose CSR as the signing method.
Upload the CSR you downloaded from the subordinate CA and configure the validity period.
Once signed, download the signed certificate.
Step 8: Install Signed Certificate on Subordinate CA
Go back to the Subordinate CA you created.
Click Actions > Install CA Certificate.
Upload the signed certificate from the root CA to activate the subordinate CA.
Now the subordinate CA is active and ready to issue certificates.
3. Request a New Private Certificate Using the Intermediate CA
Step 1: Request a Private Certificate
Go to AWS Certificate Manager (ACM).
Click Request a certificate.
Select Request a private certificate and click Next.
Step 2: Specify the Intermediate CA
In the Certificate authority dropdown, select the Intermediate CA (the subordinate CA you created).
In the Fully qualified domain name (FQDN) field, enter the domain name (e.g., example.com).
Click Next.
Step 3: Review and Request
Review your request and click Request.
Step 4: Validate the Domain
If needed, validate the domain ownership using the provided methods (DNS validation is typically preferred).
Step 5: Use the Private Certificate
Once the certificate is issued, it will be available in the ACM Console. You can now use it with your services (such as Elastic Load Balancers, CloudFront distributions, or EC2 instances).

