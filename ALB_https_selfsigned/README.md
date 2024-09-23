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