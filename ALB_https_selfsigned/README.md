# ALB with https using self signed certificate

## Steps to Create a Self-Signed Certificate and Use HTTPS on ALB:

1. Create a Self-Signed Certificate
First, create a self-signed certificate using OpenSSL:

Generate a Private Key:

openssl genpkey -algorithm RSA -out private.key

Generate a Certificate Signing Request (CSR):

openssl req -new -key private.key -out certificate.csr

You will be prompted to provide some details like Country, State, Common Name (CN), etc. Ensure that the Common Name (CN) matches your domain or ALB DNS name (if you're using the ALB DNS directly).

Create the Self-Signed Certificate:
s
openssl x509 -req -days 365 -in certificate.csr -signkey private.key -out selfsigned.crt
This will create a certificate valid for 365 days.

2. Upload the Self-Signed Certificate to ACM
AWS ACM doesn't natively support creating self-signed certificates, but you can import one.

Open the ACM Console:

Go to AWS Certificate Manager in the AWS Management Console.
Click Import a certificate.
Import the Certificate:

Certificate body: Open selfsigned.crt in a text editor and copy the entire content (including the -----BEGIN CERTIFICATE----- and -----END CERTIFICATE----- tags) and paste it into the Certificate body field.
Certificate private key: Open private.key in a text editor and copy its content, then paste it into the Private key field.
Certificate chain: Leave this field empty since it's self-signed and doesn't require a chain.
Click Next and complete the process.

3. Modify the ALB to Add HTTPS Listener
Go to the EC2 Console and select Load Balancers from the Load Balancing section.
Select the ALB you created earlier, and go to the Listeners tab.
Click Add Listener, then:
Choose HTTPS as the protocol.
Set Port 443 (the default port for HTTPS).
Choose the ACM Certificate:
Under Default actions, select Forward to and choose the target group where your EC2 instances are registered.
Under the SSL certificate section, select the ACM certificate you uploaded earlier.
Security Policy: You can choose the default or customize the security policy based on your requirements.
Click Save to create the HTTPS listener.
4. Update Security Groups
Ensure that the security group for your ALB allows inbound traffic on port 443 (HTTPS):

Go to the Security Groups in the EC2 Console.
Find the security group attached to your ALB, and edit its inbound rules to allow traffic on port 443.
5. Test the HTTPS Endpoint
Copy the DNS name of your ALB.
Open a browser and try accessing https://<ALB-DNS-Name>. Since this is a self-signed certificate, the browser will likely warn that the connection is not trusted. You can proceed by adding an exception.

## Steps to redirect http requests to https on ALB:

To redirect HTTP requests to HTTPS on your ALB, you can configure a rule in the ALB that forwards all HTTP traffic (port 80) to HTTPS (port 443). Here's how to do it:

Steps to Redirect HTTP Requests to HTTPS on ALB
1. Open EC2 Console and Go to Load Balancers
Go to the EC2 Dashboard in AWS Management Console.
Under Load Balancing, select Load Balancers.
Choose your ALB from the list.
2. Modify HTTP Listener
In the Listeners tab of your ALB, find the HTTP (Port 80) listener.
Click on the View/edit rules link.
3. Create a Redirect Rule
In the Rules Editor, there should already be a default rule forwarding traffic to your target group.
Click the + button (Add rule) above the existing rule.
Here's how to create the redirect rule:

Condition: Leave it as default, meaning it will apply to all incoming HTTP requests.

Action: Click + Add action.

Choose Redirect to.
Configure the redirection settings:
Protocol: HTTPS
Port: 443
Host, Path, Query: You can leave these as #{host}, #{path}, and #{query} to ensure the request parameters remain the same.
Status code: HTTP 301 – Permanent Redirect
After configuring the rule, click Save.

4. Save and Activate the Rule
After saving, make sure the redirect rule is listed before the rule that forwards traffic to the target group.
The rule should now redirect all HTTP requests to HTTPS.
5. Test the Redirection
Open your browser and try accessing http://<ALB-DNS-Name>.
It should automatically redirect to https://<ALB-DNS-Name>.
This configuration ensures that any HTTP traffic (port 80) is automatically redirected to HTTPS (port 443), providing a secure connection for all requests.