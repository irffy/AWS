# AWS EC2 Cross-Account S3 Access Setup

## Part 1: Grant EC2 Read-Only Access to All Buckets in Account A

### Step 1: Create IAM Role for EC2 (Account A)
1. Go to **IAM > Roles > Create Role**
2. Choose **AWS service** → **EC2**
3. **Permissions**: Attach the **AmazonS3ReadOnlyAccess** policy
   ```json
   {
     "Effect": "Allow",
     "Action": [
       "s3:Get*",
       "s3:List*"
     ],
     "Resource": "*"
   }
   ```
4. Name the role: `EC2S3ReadOnlyAccessRole`
5. Create the role

### Step 2: Attach IAM Role to EC2 Instance
1. Go to **EC2 > Instances**
2. Select your instance
3. **Actions > Security > Modify IAM Role**
4. Attach `EC2S3ReadOnlyAccessRole`

---

## Part 2: Allow Read-Only Access to Buckets in Account B

### Step 3: Copy Account A's IAM Role ARN
Format:
```
arn:aws:iam::<AccountA-ID>:role/EC2S3ReadOnlyAccessRole
```
Example:
```
arn:aws:iam::111111111111:role/EC2S3ReadOnlyAccessRole
```

### Step 4: Update S3 Bucket Policies in Account B
For each bucket in Account B:
1. Go to **S3 > Bucket > Permissions > Bucket Policy**
2. Add this policy:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "AllowReadOnlyFromAccountA",
         "Effect": "Allow",
         "Principal": {
           "AWS": "arn:aws:iam::111111111111:role/EC2S3ReadOnlyAccessRole"
         },
         "Action": [
           "s3:GetObject",
           "s3:ListBucket"
         ],
         "Resource": [
           "arn:aws:s3:::bucket-in-account-b",
           "arn:aws:s3:::bucket-in-account-b/*"
         ]
       }
     ]
   }
   ```
   - Replace `111111111111` with **Account A's ID**
   - Replace `bucket-in-account-b` with the actual bucket name

---

## Part 3: Verify Access from EC2
SSH into the EC2 instance and run:
```bash
aws s3 ls                         # Lists buckets in Account A
aws s3 ls s3://bucket-in-account-a/
aws s3 ls s3://bucket-in-account-b/
aws s3 cp s3://bucket-in-account-b/file.txt .
```

### Expected Results:
✅ Can list/download objects  
❌ Cannot upload/delete anything

---

## Summary Table
| **What** | **How** |
|----------|---------|
| EC2 access to Account A S3 buckets | Attach `AmazonS3ReadOnlyAccess` policy to EC2's IAM role |
| EC2 access to Account B S3 buckets | Add read-only bucket policy to each bucket in Account B |
| **Role ARN for Account B policy** | `arn:aws:iam::<AccountA-ID>:role/EC2S3ReadOnlyAccessRole` |
