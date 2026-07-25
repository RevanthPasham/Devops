# Amazon S3 Complete Setup Guide (Beginner to Production)

This guide explains:

- What Amazon S3 is
- How to create an S3 bucket
- Why every option exists
- How to create IAM Access Keys
- How to connect Node.js with S3
- Common interview questions
- Security best practices

---

# What is Amazon S3?

Amazon S3 (Simple Storage Service) is AWS's object storage service.

It is used to store files such as:

- Images
- Videos
- PDF files
- User profile pictures
- Resumes
- Invoice PDFs
- Backup files
- Application assets

Unlike a database, S3 stores files instead of structured data.

Example

User uploads

profile.jpg

↓

Node.js Backend

↓

Amazon S3

↓

Stored safely

---

# What is a Bucket?

A Bucket is simply a container that stores files.

Think of it like:

Computer

Documents Folder

↓

resume.pdf

↓

photo.png

↓

invoice.pdf

In S3,

Bucket

↓

resume.pdf

↓

photo.png

↓

invoice.pdf

Everything is stored inside a bucket.

---

# Creating an S3 Bucket

AWS Console

↓

Search

↓

S3

↓

Create Bucket

---

# Step 1 - AWS Region

Example

ap-south-1 (Mumbai)

us-west-1 (California)

eu-west-1 (Ireland)

Question:

Why do we need a region?

Because AWS stores your files in physical data centers.

Example

India User

↓

Mumbai Bucket

Fast

Example

India User

↓

California Bucket

↓

Higher latency

Important

The bucket region CANNOT be changed after creation.

If you accidentally choose the wrong region,

Create a new bucket.

Delete the old one.

---

# What is a Global Service?

Many beginners think S3 should have a Global Region.

It does not.

Files must physically exist somewhere.

Some AWS services are Global because they don't store customer files in one location.

Examples

Global Services

- IAM
- Route53
- CloudFront
- AWS Organizations

Regional Services

- EC2
- S3
- RDS
- VPC
- Lambda

Interview Question

Q: Why isn't S3 Global?

Answer:

Because uploaded files must physically exist in a specific AWS region for performance, cost, and regulatory reasons.

---

# Bucket Namespace

AWS gives two options.

Option 1

Global Namespace

Bucket name

parcel

Must be unique across ALL AWS accounts.

If another AWS customer already owns

parcel

You cannot create it.

-----------------------

Option 2

Account Regional Namespace (Recommended)

AWS automatically creates

parcel-465573888554-us-west-1-an

Advantages

No need to think of unique names.

AWS automatically makes it unique.

Recommended for beginners.

---

# Bucket Name

Example

parcel-images

or

parcel

Bucket names cannot be changed later.

---

# Object Ownership

Option 1

ACL Disabled (Recommended)

Option 2

ACL Enabled

What is ACL?

ACL = Access Control List

Old AWS permission system.

Example

photo1

↓

John can Read

photo2

↓

Alice can Write

Managing thousands of files becomes difficult.

AWS now recommends IAM Policies.

Advantages of ACL Disabled

- Easier permission management
- Bucket owner owns every uploaded file
- Modern AWS recommendation

Use ACL Enabled only if you specifically need object-level ACLs for cross-account scenarios.

---

# Block Public Access

One of the most important settings.

Recommended

Enable

Block All Public Access

Question

Can users still upload files?

YES.

Example

User

↓

Node.js Backend

↓

AWS SDK

↓

Private S3 Bucket

Works perfectly.

Question

Can users directly open

https://bucket.s3.amazonaws.com/image.jpg

NO

They receive

Access Denied

Question

Then how does my website show images?

Answer

Node.js requests the image.

or

Node.js generates a Pre-Signed URL.

Then React loads the image.

Examples of Private Files

- User profile pictures
- Driver documents
- Parcel images
- Invoices
- Medical reports
- Resumes

Examples of Public Files

- Website logo
- CSS
- JavaScript
- Public product images

Never make private user documents public.

---

# Bucket Versioning

Disabled

Only latest version exists.

Enabled

Every version is stored.

Example

resume.pdf

↓

Version 1

↓

Version 2

↓

Version 3

Advantages

Recover deleted files

Rollback accidental changes

Disadvantages

More storage cost.

---

# Tags

Tags help organize resources.

Example

Environment=Production

Project=Parcel

Owner=Revanth

Advantages

- Cost tracking
- Easy searching
- Resource organization

---

# Default Encryption

Options

SSE-S3

AWS manages encryption keys.

Recommended for most applications.

---------------------

SSE-KMS

AWS Key Management Service manages keys.

Use when

- Banks
- Healthcare
- Government
- Compliance requirements

Advantages

Audit logs

Access control

Key rotation

Disadvantage

Higher complexity.

---

# Bucket Key

Used with KMS.

Purpose

Reduce KMS API calls.

Result

Lower encryption cost.

Ignore this when using SSE-S3.

---

# Advanced Settings

Contains advanced features like

- Object Lock
- Transfer Acceleration
- Event Notifications

Beginners usually leave these as default.

---

# Can I Block Specific Countries?

Yes.

Method 1

CloudFront Geographic Restrictions

Allow

India

Singapore

Block

China

Russia

North Korea

Recommended for serving public content.

-----------------------

Method 2

AWS WAF

Create Geo Match Rules

Example

IF Country == China

↓

Block

Works with

- CloudFront
- Application Load Balancer
- API Gateway

-----------------------

Method 3

Bucket Policy

Bucket Policies cannot directly block countries.

They can block

- IP addresses
- AWS Accounts
- IAM Users
- VPC Endpoints

---

# Creating IAM User

Never use Root User Access Keys.

Always create an IAM User.

Steps

AWS Console

↓

IAM

↓

Users

↓

Create User

User Name

parcel-app

Click Next

---

# Permissions

Choose

Attach Policies Directly

Search

AmazonS3FullAccess

Select

AmazonS3FullAccess

Click Next

Create User

Note

For learning,

AmazonS3FullAccess is okay.

Production applications should use custom least-privilege policies.

---

# Creating Access Keys

Open

IAM

↓

Users

↓

parcel-app

↓

Security Credentials

↓

Access Keys

↓

Create Access Key

Choose

Application Running Outside AWS

Click Next

Create Access Key

AWS shows

Access Key ID

AKIAXXXXXXXXXXXX

Secret Access Key

XXXXXXXXXXXXXXXXXX

IMPORTANT

AWS shows the Secret Access Key ONLY ONCE.

Save it immediately.

---

# Environment Variables

.env

AWS_ACCESS_KEY_ID=AKIAxxxxxxxxxxxxxxxx

AWS_SECRET_ACCESS_KEY=xxxxxxxxxxxxxxxx

AWS_REGION=ap-south-1

AWS_BUCKET_NAME=parcel-images

Never commit this file to GitHub.

---

# Install AWS SDK

npm install @aws-sdk/client-s3

---

# Upload Flow

User

↓

React

↓

Node.js Backend

↓

AWS SDK

↓

Amazon S3

↓

File Stored

Never upload directly using your Access Keys from React.

The backend should communicate with AWS.

---

# Production Best Practices

Use

Private Bucket

Block Public Access

IAM User

Least Privilege Permissions

Server-side Encryption

Pre-Signed URLs

Never

Store AWS Keys in React

Commit .env to GitHub

Use Root User Keys

Make user documents public

---

# Common Interview Questions

Q. Why use S3 instead of storing images in PostgreSQL?

Answer

S3 is optimized for storing large files, is cheaper, highly durable, and scales automatically.

---

Q. Why is Block Public Access enabled?

Answer

To prevent anyone on the internet from directly accessing uploaded files.

---

Q. Why is S3 Regional instead of Global?

Answer

Because files must physically reside in a specific AWS region.

---

Q. Why use IAM User instead of Root User?

Answer

Root has unlimited privileges. IAM follows the principle of least privilege and improves security.

---

Q. Can users upload files when Block Public Access is enabled?

Answer

Yes.

The backend uploads using AWS credentials.

Only direct public access is blocked.

---

Q. Can I change the bucket region later?

Answer

No.

Create a new bucket in the desired region and migrate the objects if needed.
