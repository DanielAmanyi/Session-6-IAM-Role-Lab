# IAM Lab 4: Launch EC2, Attach IAM Role, Test from Console & Localhost

## Introduction

In this lab, you will learn how to launch an EC2 instance, attach an IAM role, and validate AWS service access from both the AWS Console and your local machine. This exercise focuses on practical IAM usage, particularly how to grant temporary permissions to an EC2 instance without embedding long-term credentials.

By the end of this lab, you should be able to:
- Create and attach an IAM role to an EC2 instance
- Understand how instance metadata provides temporary credentials
- Use those credentials to access Amazon S3
- Differentiate between permissions available to EC2 vs your local IAM user

---

## What is an IAM Role?

An **IAM Role** is an AWS identity with specific permissions that can be assumed by trusted entities such as:
- AWS services (like EC2, Lambda, or ECS)
- IAM users
- Federated identities

Unlike IAM users, roles do **not** have permanent credentials. Instead, they provide temporary security credentials when assumed. This makes roles especially useful for granting time-limited access without hardcoding secrets.

### Common Use Cases:
- **EC2 access to S3** (as in this lab)
- Lambda function access to DynamoDB or SNS
- Cross-account access between AWS accounts
- Federated access via Identity Providers (e.g., Google Workspace or Okta)

---

## Lab Scenario

You are a DevOps engineer tasked with deploying an EC2 instance that will run a script needing read-only access to files stored in an Amazon S3 bucket. Your team policy disallows the use of long-term credentials on EC2. Instead, the instance must assume a role that grants minimal required permissions — in this case, AmazonS3ReadOnlyAccess.

This lab will guide you step-by-step through:
- Launching the EC2 instance
- Creating and attaching an IAM role
- Testing S3 access using both browser-based EC2 Instance Connect and local AWS CLI

---


## Step 1: Launch an EC2 Instance from AWS Console

1. Sign in to the AWS Console  
   Navigate to: https://console.aws.amazon.com/ec2  
   Ensure you're in the correct AWS region.

2. Launch a New Instance  
   - Click Launch instance  
   - Name: IAM-Test-Instance  
   - AMI: Amazon Linux 2 AMI  
   - Instance Type: t2.micro (Free-tier eligible)  
   - Key Pair (Login): Create or select an existing key pair (for SSH)  
   - Network Settings:  
     -- Use default VPC or your custom one  
     -- Enable Auto-assign Public IP  
     -- Security Group: Create/use one that allows SSH (port 22)  
   - Click Launch Instance

## Step 2: Check and Record Key EC2 Components

After the instance is running, click its Instance ID and inspect the following:

-- Public IP address – Needed for SSH  
-- Private IP address – Used internally within the VPC  
-- Subnet ID – Identifies the subnet  
-- VPC ID – Confirms network placement  
-- IAM Role – Should say None initially  
-- Security Group ID – Review inbound rules  
-- Key pair name – Required for SSH  
-- Instance state – Should show running

Tip: Use Actions > Monitor and troubleshoot > Get system log to view boot diagnostics.

## Step 3: Create an IAM Role for EC2 to Access S3

1. Go to IAM > Roles > Create role  
2. Trusted Entity: Choose AWS service, then select EC2  
3. Permissions Policy: Attach AmazonS3ReadOnlyAccess  
4. Role Name: EC2S3ReadRole  
5. Click Create role

Why: This allows EC2 to assume the role and access S3 securely via temporary credentials.

## Step 4: Attach the IAM Role to the EC2 Instance

1. Go to EC2 Dashboard > Instances  
2. Select your instance  
3. Click Actions > Security > Modify IAM Role  
4. Attach the EC2S3ReadRole  
5. Click Update IAM Role

## Step 5: Connect via Console (EC2 Instance Connect)

1. Select your instance  
2. Click Connect  
3. Choose EC2 Instance Connect tab  
4. Click Connect to open a browser-based terminal

Once inside the instance:

```
# Check IAM role metadata
curl http://169.254.169.254/latest/meta-data/iam/info

# List available S3 buckets
aws s3 ls

# Try downloading a file from a bucket
aws s3 cp s3://your-bucket-name/yourfile.txt .
```

These should succeed if the IAM role is attached correctly.

## Step 6: Enable Access from Localhost

### 6.1: Ensure AWS CLI is Installed

```
aws --version
```

If not installed, download from:  
https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html

### 6.2: Create Access Keys (Temporary Use Only)

1. Go to IAM > Users > YourUser > Security credentials  
2. Under Access Keys, click Create access key  
3. Save the Access key ID and Secret access key

### 6.3: Configure AWS CLI on Local Machine

```
aws configure
# Enter Access key ID
# Enter Secret access key
# Enter Default region (e.g., us-east-1)
# Choose output format (e.g., json)
```

## Step 7: Test IAM Permissions from Localhost

```
# List all S3 buckets
aws s3 ls

# Download a file from S3
aws s3 cp s3://your-bucket-name/yourfile.txt .

# Attempt to upload a file (should fail)
echo "test" > testfile.txt
aws s3 cp testfile.txt s3://your-bucket-name/
```

Upload should fail — the role only grants read permissions.

## Step 8: Clean Up (Recommended)

-- Terminate the EC2 instance  
-- Delete the IAM role  
-- Remove local access keys from your IAM user

## Summary

This lab walked through:

-- Launching and inspecting an EC2 instance  
-- Creating and attaching an IAM role  
-- Testing role-based access from EC2 and localhost  
-- Validating least-privilege access to S3
