# AWS Security Tools

| Tool                         | Description                                                                 | Use Case                                                                                  |
|------------------------------|-----------------------------------------------------------------------------|-------------------------------------------------------------------------------------------|
| **IAM Access Analyzer**      | Identifies resources that are shared with an external entity.               | Identify if an IAM role grants public access to S3, KMS, or Lambda.                       |
| **AWS Detective** 🔍         | Investigates AWS security threats using AI and log analytics.               | Trace anomalous API requests from a hacked AWS account.                                   |
| **AWS Systems Manager (SSM)** 🚀 | Manages EC2 patching, remote access (without SSH), and automation.        | Use Session Manager to securely access EC2 instances without SSH keys.                    |
| **Amazon Cognito** 🔑        | Provides user authentication, MFA, and OAuth-based login.                   | Securely authenticate users in a mobile app with Google/Facebook sign-in.                 |