# S3 Bucket Access Control Guide

This document provides an overview of how access control works for Amazon S3 buckets, focusing on default behavior, IAM policies, and bucket policies.

## Key Points

- **By Default: S3 Buckets Are Private**
  - New S3 buckets are completely private by default.
  - Only the bucket owner (account root) has implicit full access.
  - Access must be explicitly granted via:
    - Bucket policies
    - IAM policies
    - ACLs (not commonly used anymore)

## Behavior Without a Bucket Policy

- **Same-Account Access**: IAM roles/users in the same account can access the bucket if their IAM policies allow it.
- **Cross-Account Access**: Denied by default, even if the IAM policy allows it. A bucket policy is required to explicitly trust cross-account roles.


## Summary
## Access Control Table

The following table summarizes the access control behavior for S3 buckets under different conditions:

| **Condition**                     | **Access**                              |
|------------------------------------|-----------------------------------------|
| No bucket policy (default)         | ❌ Denied for other accounts             |
| Same-account access with IAM role  | ✅ If IAM policy allows it              |
| Cross-account access               | ❌ Unless bucket policy explicitly allows it |
| Bucket policy + cross-account IAM role | ✅ Access works                     |

## Examples

### Without a Bucket Policy (Cross-Account)
If an EC2 role in Account A has `AmazonS3ReadOnlyAccess` and there's no bucket policy on a bucket in Account B:
- The EC2 role will receive an `AccessDenied` error.
- Reason: Account B’s bucket does not trust the role from Account A.

### With a Bucket Policy
Adding a bucket policy like the following in Account B allows cross-account access:

```json
{
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

