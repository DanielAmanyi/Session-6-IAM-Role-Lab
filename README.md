# IAM Lab 4: Launch EC2, Attach IAM Role, Test from Console & Localhost

Here is the revised README.md with:

A definition of IAM Role

Common use cases

A realistic scenario to ground the lab in a practical context

No emojis. Plain, direct, and practical.

IAM Lab 4: Launch EC2, Attach IAM Role, Test from Console & Localhost
Introduction
This lab is designed to help you understand and practice the use of IAM roles with Amazon EC2 instances. You will launch a new EC2 instance, attach an IAM role that grants read-only access to Amazon S3, and validate access from both the EC2 console and your local environment using the AWS CLI. This hands-on exercise reinforces core IAM concepts such as role-based access, security best practices, and the principle of least privilege.

What is an IAM Role?
An IAM Role is an AWS identity with specific permissions that can be assumed by trusted entities such as EC2 instances, Lambda functions, or other AWS services. Unlike IAM users, roles are not associated with long-term credentials. Instead, they provide temporary credentials that are dynamically issued and automatically rotated.

Roles are commonly used when:
-- You want to delegate access without sharing credentials
-- An EC2 instance needs to interact with other AWS services (e.g., S3, DynamoDB, CloudWatch)
-- A Lambda function needs permission to read/write data in S3 or invoke other AWS services
-- You want to grant cross-account access between AWS environments

Scenario
You are part of a DevOps team tasked with setting up a temporary analysis server.
A data science team needs to pull log files stored in an Amazon S3 bucket to analyze performance metrics. However, they must not be able to upload or modify the contents of the bucket.

Your job is to:
-- Launch an EC2 instance
-- Attach an IAM Role that grants read-only access to S3
-- Ensure no credentials are hardcoded into the server
-- Confirm that the instance can list and download objects from S3, but cannot upload anything

This lab simulates that exact use case.  


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
