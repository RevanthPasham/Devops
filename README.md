# AWS ECR + ECS + Application Load Balancer Complete Guide

> A complete beginner-to-production guide for deploying Dockerized applications on AWS using Amazon ECR, Amazon ECS (Fargate), and Application Load Balancer (ALB).

---

# Table of Contents

1. Introduction
2. Docker Overview
3. Amazon Elastic Container Registry (ECR)
4. Amazon ECS Cluster
5. ECS Task Definition
6. ECS Service
7. ECS Tasks
8. Networking (VPC, Subnets, Security Groups)
9. Target Groups
10. Application Load Balancer
11. Connecting ECS with ALB
12. Deployment Verification
13. Troubleshooting
14. Best Practices

---

# Goal

By the end of this guide you will know how to:

✔ Containerize your application

✔ Push Docker images to Amazon ECR

✔ Create an ECS Cluster

✔ Create a Task Definition

✔ Create an ECS Service

✔ Create an Application Load Balancer

✔ Create a Target Group

✔ Connect ECS with ALB

✔ Verify deployment

✔ Troubleshoot deployment issues

✔ Understand every AWS option used during deployment

---

# Overall Architecture

                        Developer
                            │
                            ▼
                    Build Docker Image
                            │
                            ▼
                    Push Image to ECR
                            │
                            ▼
                    Amazon ECR Repository
                            │
                            ▼
                  ECS Task Definition
                            │
                            ▼
                     ECS Service
                            │
                            ▼
                     ECS Cluster
                            │
                            ▼
                     Running Task
                            │
                            ▼
                     Target Group
                            │
                            ▼
                 Application Load Balancer
                            │
                            ▼
                         Internet
                            │
                            ▼
                           Users

---

# Technologies Used

- Docker
- Amazon ECR
- Amazon ECS
- AWS Fargate
- Application Load Balancer
- Target Groups
- VPC
- Subnets
- Security Groups
- IAM
- CloudWatch Logs
# Step 1: Create Amazon Elastic Container Registry (ECR)

## What is Amazon ECR?

Amazon Elastic Container Registry (ECR) is AWS's Docker Image Registry.

Think of it like GitHub, but instead of storing source code, it stores Docker Images.

Amazon ECS cannot run your local Docker image directly. It must first pull the image from a registry like Amazon ECR.

Architecture

Developer
    │
    ▼
Docker Build
    │
    ▼
Docker Image
    │
docker push
    │
    ▼
Amazon ECR
    │
docker pull
    │
    ▼
Amazon ECS

---

## Step 1.1 Open Amazon ECR

AWS Console

→ Search **Elastic Container Registry**

→ Click **Repositories**

→ Click **Create Repository**

---

# Repository Settings

---

## 1. Visibility Settings

Options

- Private ✅
- Public

### Private

Purpose

Only AWS IAM users or services can access the repository.

Use When

- Backend APIs
- Company Projects
- Production Applications
- Internal Applications

Advantages

- Secure
- IAM Protected
- Recommended for almost every project

### Public

Purpose

Anyone on the Internet can pull your Docker image.

Use When

- Open Source Projects
- Public Docker Images

Example

Node.js Sample Images

Python Sample Images

### Selected

✅ Private

Reason

Our backend should not be publicly downloadable.

---

## 2. Repository Name

Purpose

Unique name of your Docker Image Repository.

Example

backend

resume-api

payment-service

auth-service

### Selected

backend

Recommendation

Always use meaningful names.

Avoid

test

abc

repo1

backend123

---

## 3. Image Tag Mutability

Purpose

Controls whether Docker image tags can be overwritten.

Options

### Mutable ✅

Allows

latest

to be replaced.

Example

Push

backend:latest

Tomorrow

Push another

backend:latest

Old image gets replaced.

Use When

Development

Testing

Daily deployments

### Immutable

Docker tag cannot be overwritten.

Example

v1.0.0

Once pushed

Nobody can replace it.

Use When

Production

Release versions

Banking applications

Enterprise systems

### Selected

✅ Mutable

Reason

During development we continuously push new images.

Production Recommendation

Immutable

---

## 4. Scan on Push

Purpose

Automatically scans Docker Images for security vulnerabilities.

Options

Enabled

Disabled ✅

### Enabled

AWS checks

- Known vulnerabilities
- Security risks
- Outdated packages

Recommended

Production

Enterprise Applications

### Disabled

No automatic scanning.

Recommended

Development

Learning

### Selected

Disabled

Reason

Development environment.

---

## 5. Encryption

Purpose

Encrypt Docker Images stored inside Amazon ECR.

Options

### AES-256 ✅

AWS manages encryption automatically.

Recommended for

Almost everyone.

### AWS KMS

Uses your own encryption key.

Recommended for

Banks

Healthcare

Government

Enterprise companies with compliance requirements.

### Selected

AES-256

Reason

Simple and managed by AWS.

---

## 6. Tags

Purpose

Helps organize AWS Resources.

Example

Environment = Development

Project = Backend

Owner = DevOps Team

Department = Engineering

Required?

No

### Selected

Skipped

Reason

Not required for this deployment.

---

## Step 1.2 Create Repository

Click

Create Repository

Result

Repository successfully created.

Example URI

123456789012.dkr.ecr.us-west-1.amazonaws.com/backend

This URI will be required later while creating the ECS Task Definition.

---

# Step 1.3 Login Docker to AWS

Run

```bash
aws ecr get-login-password --region us-west-1 \
| docker login \
--username AWS \
--password-stdin 123456789012.dkr.ecr.us-west-1.amazonaws.com
```

Purpose

Authenticates Docker with Amazon ECR.

Without this command

Docker cannot push images.

---

# Step 1.4 Build Docker Image

Run

```bash
docker build -t backend .
```

Purpose

Creates a Docker Image.

Verify

```bash
docker images
```

Expected

backend

latest

---

# Step 1.5 Tag Docker Image

Run

```bash
docker tag backend:latest \
123456789012.dkr.ecr.us-west-1.amazonaws.com/backend:latest
```

Purpose

Associates your local Docker Image with the Amazon ECR Repository.

---

# Step 1.6 Push Docker Image

Run

```bash
docker push \
123456789012.dkr.ecr.us-west-1.amazonaws.com/backend:latest
```

Purpose

Uploads Docker Image to Amazon ECR.

---

# Step 1.7 Verify

Open

Amazon ECR

↓

Repositories

↓

backend

↓

Images

You should see

- latest
- Image Size
- Digest
- Push Time
- URI

Congratulations!

Your Docker Image is now stored in Amazon ECR.

Next Step

Create an Amazon ECS Cluster.

# Step 2: Create Amazon ECS Cluster

## What is Amazon ECS?

Amazon Elastic Container Service (ECS) is a fully managed container orchestration service.

Its job is to run and manage your Docker containers.

Think of it this way:

Docker creates the container.

ECR stores the container image.

ECS runs the container.

---

## Why do we need ECS?

Without ECS

Developer
    │
    ▼
Docker Image
    │
Copy to Server
    │
SSH into Server
    │
docker run

You have to manage everything manually.

---

With ECS

Developer
      │
      ▼
Docker Image
      │
      ▼
Amazon ECR
      │
      ▼
Amazon ECS
      │
      ▼
Container Running

AWS automatically manages the container.

---

## What is a Cluster?

A Cluster is a logical group where ECS runs your applications.

Think of it as a workspace.

Example

Company

Cluster

backend-cluster

Inside the Cluster

- Backend API
- Authentication Service
- Payment Service

All can run inside one cluster.

---

## Step 2.1 Open ECS

AWS Console

↓

Search

Elastic Container Service

↓

Clusters

↓

Create Cluster

---

# Cluster Configuration

---

## 1. Cluster Name

Purpose

A unique name for your ECS Cluster.

Examples

backend-cluster

production-cluster

resume-cluster

payment-cluster

### Selected

backend-cluster

Recommendation

Choose meaningful names.

Avoid

cluster1

abc

test

---

## 2. Infrastructure

AWS provides three infrastructure options.

---

### Option 1: AWS Fargate ✅ (Selected)

Purpose

Serverless container hosting.

You don't manage servers.

AWS automatically

- Creates compute
- Runs containers
- Handles scaling
- Maintains infrastructure

Use When

- Node.js Applications
- React Backend
- REST APIs
- Small Projects
- Medium Projects
- Most Production Applications

Advantages

✔ No EC2 management

✔ Easy to deploy

✔ Pay only while containers run

✔ Recommended for beginners

Disadvantages

Slightly more expensive than EC2 for very large workloads.

### Selected

AWS Fargate

Reason

We don't want to manage EC2 servers.

---

### Option 2: Amazon EC2

Purpose

Runs containers on EC2 Instances.

You manage

- EC2
- Security
- Patching
- Scaling
- Instance Types

Use When

- High-performance workloads
- GPU workloads
- Large enterprise applications
- Custom operating systems

Advantages

More control

Lower cost for large workloads

Disadvantages

More maintenance.

---

### Option 3: Amazon ECS Managed Instances

Purpose

AWS launches and manages EC2 instances for you.

You still get EC2-based infrastructure, but AWS handles most management tasks.

Use When

- Need EC2 flexibility
- Don't want to manage instances manually

Advantages

Less management than EC2

More flexibility than Fargate

Disadvantages

More complex than Fargate.

---

### Selected

AWS Fargate

Reason

Simple, serverless, and ideal for our deployment.

---

## 3. Monitoring (Container Insights)

Purpose

Collects metrics about your ECS Cluster.

Options

Enabled

Disabled ✅

---

### Enabled

Collects

- CPU Usage
- Memory Usage
- Network Usage
- Task Metrics

Useful For

Production

Monitoring

Performance Analysis

---

### Disabled

No monitoring data.

Suitable for

Learning

Testing

Development

---

### Selected

Disabled

Reason

Not required during initial deployment.

Production Recommendation

Enable Container Insights.

---

## 4. Tags

Purpose

Helps organize AWS resources.

Examples

Environment = Development

Project = Backend

Owner = DevOps

Department = Engineering

Required?

No

---

### Selected

Skipped

Reason

Not required for learning.

---

## Step 2.2 Create Cluster

Click

Create

AWS creates the cluster.

Expected Result

Cluster Status

Active

Services

0

Tasks

0

Capacity Providers

AWS Fargate

This is expected because we haven't deployed an application yet.

---

# Cluster Architecture

Developer
      │
      ▼
Amazon ECR
      │
      ▼
ECS Cluster
      │
      ▼
(No Running Tasks Yet)

---

# Common Mistakes

❌ Choosing EC2 without understanding EC2 management.

Result

Need to create EC2 instances manually.

---

❌ Expecting the Cluster to start containers automatically.

Reality

A Cluster is only a workspace.

It does NOT run containers until you create:

- Task Definition
- ECS Service

---

❌ Thinking the Cluster stores Docker Images.

Reality

Docker Images are stored in Amazon ECR.

The Cluster only runs containers.

---

# Summary

After completing this step, you have:

✔ Created an ECS Cluster

✔ Selected AWS Fargate as the infrastructure

✔ Created a logical workspace where your applications will run

Current Architecture

Developer
      │
      ▼
Docker Image
      │
      ▼
Amazon ECR
      │
      ▼
Amazon ECS Cluster

Next Step

Create an ECS Task Definition, which tells ECS **what container to run and how to run it**.

# Step 3: Create ECS Task Definition

## What is a Task Definition?

A Task Definition is a **blueprint** that tells Amazon ECS **how to run your Docker container**.

Think of it as an instruction manual.

It answers questions like:

- Which Docker Image should I run?
- How much CPU should I use?
- How much Memory should I use?
- Which Port should I expose?
- Which Environment Variables should I use?
- Which IAM Role should I use?

Without a Task Definition, ECS does not know how to start your application.

---

## Architecture

Amazon ECR
      │
Docker Image
      │
      ▼
Task Definition
      │
      ▼
ECS Service
      │
      ▼
Running Task

---

## Step 3.1 Open Task Definitions

AWS Console

↓

Amazon ECS

↓

Task Definitions

↓

Create New Task Definition

---

# Task Definition Configuration

---

## 1. Task Definition Family

Purpose

The Family is simply the name of your Task Definition.

Every time you update your application, AWS creates a new Revision under the same Family.

Example

backend

↓

Revision 1

↓

Revision 2

↓

Revision 3

Instead of creating multiple Task Definitions, AWS versions them.

Examples

backend

resume-api

payment-service

auth-service

### Selected

backend

Recommendation

Choose meaningful names.

Avoid

test

abc

task1

---

## 2. Launch Type (Infrastructure Requirements)

Purpose

Specifies where this Task can run.

Options

### AWS Fargate ✅ (Selected)

Serverless.

AWS automatically provides compute resources.

Use When

- Node.js APIs
- React Backends
- Express Servers
- Most applications

Advantages

✔ No EC2 management

✔ Easy deployment

✔ Automatic infrastructure

### Amazon EC2

Runs containers on EC2 instances.

Use When

Need

- GPU
- Large workloads
- Custom operating system
- Complete server control

Requires EC2 management.

### ECS Managed Instances

AWS manages EC2 instances for you.

Used in larger organizations requiring EC2 flexibility.

### Selected

AWS Fargate

Reason

Simple and recommended for our application.

---

## 3. Operating System

Options

Linux ✅

Windows

### Linux

Most Docker applications use Linux.

Recommended for

- Node.js
- Java
- Python
- Go
- PHP

### Windows

Use only if your application requires Windows.

### Selected

Linux

Reason

Node.js application.

---

## 4. CPU Architecture

Options

X86_64 ✅

ARM64

---

### X86_64

Standard Intel/AMD processors.

Works with almost every Docker Image.

Recommended

Most projects.

---

### ARM64

Runs on AWS Graviton processors.

Advantages

Lower cost

Better performance

Use When

Docker image supports ARM.

### Selected

X86_64

Reason

Maximum compatibility.

---

## 5. Network Mode

For AWS Fargate

Network Mode is automatically set to

awsvpc

Purpose

Every running Task gets

- Private IP
- Optional Public IP
- Elastic Network Interface (ENI)
- Security Group

Why?

Each container behaves like an independent server.

### Selected

awsvpc (Automatic)

---

## 6. Task Size (CPU & Memory)

Purpose

Defines resources allocated to the Task.

---

### CPU

Common Options

0.25 vCPU

0.5 vCPU

1 vCPU

2 vCPU

4 vCPU

Selected

0.25 vCPU

Reason

Small backend application.

Production Recommendation

Choose based on application usage.

---

### Memory

Options depend on CPU selection.

Examples

512 MB

1 GB

2 GB

4 GB

Selected

512 MB

Reason

Enough for a small Express backend.

Production

Increase based on monitoring.

---

## 7. Task Execution Role

Purpose

Allows ECS to interact with AWS services **before** your application starts.

Examples

- Pull Docker Image from ECR
- Send Logs to CloudWatch

Without this role

Task cannot start.

Recommended Role

ecsTaskExecutionRole

### Selected

ecsTaskExecutionRole

---

## 8. Task Role

Purpose

Permissions used **inside your running application**.

Example

Your backend accesses

- S3
- DynamoDB
- SNS
- SQS
- Secrets Manager

Without Task Role

Application receives

Access Denied

Our Project

Application only connects to PostgreSQL.

### Selected

None

Reason

No AWS SDK calls.

Production

Create a custom IAM Role.

Never use AdministratorAccess.

---

# Container Configuration

Click

Add Container

---

## 9. Container Name

Purpose

Name of the container.

Examples

backend

node-api

resume-service

### Selected

backend

Recommendation

Keep it meaningful.

---

## 10. Image URI

Purpose

Specifies which Docker Image ECS should pull.

Example

123456789012.dkr.ecr.us-west-1.amazonaws.com/backend:latest

This comes from Amazon ECR.

Without this

Task cannot start.

### Selected

Our ECR Image URI

---

## 11. Essential Container

Options

Yes ✅

No

Purpose

Determines whether the Task should stop if this container stops.

### Yes

If container crashes

↓

Entire Task stops.

Recommended

Main backend container.

### No

Container can stop

Task continues running.

Use When

Sidecar containers

Examples

- Log collectors
- Monitoring agents

### Selected

Yes

---

## 12. Port Mapping

Purpose

Maps container ports.

Container Port

3000

Protocol

TCP

### Selected

3000

Reason

Express application listens on port 3000.

Important

Container Port must match

app.listen(PORT)

Example

```javascript
app.listen(3000, "0.0.0.0");
```

---

## 13. Environment Variables

Purpose

Pass configuration into the container.

Examples

PORT=3000

NODE_ENV=production

DATABASE_URL=...

JWT_SECRET=...

Selected

Added required variables for the application.

Never store sensitive values directly.

Production

Use AWS Secrets Manager or Systems Manager Parameter Store.

---

## 14. Logging

Purpose

Stores container logs.

Options

AWS CloudWatch Logs ✅

None

Selected

AWS CloudWatch Logs

Reason

Allows viewing logs without SSH.

Production Recommendation

Always enable CloudWatch Logs.

---

## 15. Health Check

Purpose

Checks whether the container is healthy.

Example

```bash
CMD-SHELL curl -f http://localhost:3000/ || exit 1
```

Use When

Production deployments.

Selected

Skipped during initial setup.

Reason

We later use ALB Health Checks.

---

## 16. Restart Policy

Purpose

Defines what ECS should do if the container crashes.

Selected

Default

AWS automatically replaces failed Tasks through the ECS Service.

---

## 17. Storage

Purpose

Attach storage to the container.

Options

Ephemeral Storage

EFS

Selected

Default Ephemeral Storage

Reason

Our application does not require persistent storage.

Use EFS only when files must survive container restarts.

---

## 18. Docker Labels

Purpose

Metadata for Docker.

Selected

Skipped

Use When

Monitoring or service discovery tools require labels.

---

## 19. Startup Dependency

Purpose

Controls container startup order.

Use When

Multiple containers exist in one Task.

Example

Database

↓

Backend

↓

Nginx

Selected

Skipped

Reason

Only one container.

---

## 20. Monitoring

Purpose

Additional monitoring configuration.

Selected

Default

---

## 21. Tags

Purpose

Organize AWS resources.

Examples

Environment=Development

Project=Backend

Owner=DevOps

Selected

Skipped

---

## Step 3.2 Create Task Definition

Click

Create

AWS creates

Task Definition

↓

Revision 1

Example

backend:1

Future updates become

backend:2

backend:3

backend:4

No need to create a new Task Definition each time.

AWS automatically creates new revisions.

---

# Common Mistakes

❌ Wrong Image URI

Result

CannotPullContainerError

---

❌ Wrong Container Port

Result

Application starts

ALB Health Checks fail.

---

❌ Missing Execution Role

Result

Cannot pull image from ECR.

---

❌ Forgetting Environment Variables

Result

Application crashes.

---

# Summary

After completing this step, you have:

✔ Created a Task Definition

✔ Linked your Docker Image from Amazon ECR

✔ Configured CPU and Memory

✔ Configured Port 3000

✔ Added Environment Variables

✔ Configured CloudWatch Logs

✔ Created Revision 1

Current Architecture

Developer
      │
      ▼
Docker Image
      │
      ▼
Amazon ECR
      │
      ▼
Task Definition
      │
      ▼
(Not Running Yet)

Next Step

Create an **ECS Service**, which uses this Task Definition to launch and maintain running Tasks.

# Step 4: Create Amazon ECS Service

## What is an ECS Service?

An ECS Service is responsible for **running and managing your Tasks**.

A Task Definition only describes **how** to run a container.

A Service actually **starts the Task**, keeps it running, and replaces it if it crashes.

Think of it like this:

Task Definition = Blueprint

Service = Manager

Task = Running Application

---

## Architecture

Amazon ECR
      │
      ▼
Task Definition
      │
      ▼
ECS Service
      │
      ▼
Running Task
      │
      ▼
Application Running

---

## Step 4.1 Open Your Cluster

AWS Console

↓

Amazon ECS

↓

Clusters

↓

backend-cluster

↓

Services

↓

Create

---

# Service Configuration

---

## 1. Compute Configuration

Purpose

Specifies where your Tasks should run.

Options

### Launch Type

AWS Fargate ✅

Amazon EC2

Managed Instances

### Selected

AWS Fargate

Reason

We don't want to manage EC2 instances.

---

## 2. Task Definition Family

Purpose

Choose the Task Definition created earlier.

Example

backend

---

## 3. Revision

Purpose

Select which revision to deploy.

Options

Latest Revision ✅

Revision 1

Revision 2

Revision 3

...

### Selected

Latest

Reason

Deploys the newest version automatically.

Production

Sometimes companies deploy a specific revision for stability.

---

## 4. Service Name

Purpose

Name of the ECS Service.

Examples

backend-service

resume-api-service

payment-service

### Selected

backend-service

Recommendation

Always use meaningful names.

---

## 5. Service Type

Options

Replica ✅

Daemon

---

### Replica

Runs the number of Tasks you specify.

Example

Desired Tasks = 2

↓

ECS keeps exactly 2 Tasks running.

Recommended

Almost every web application.

---

### Daemon

Runs exactly one Task on every EC2 instance.

Only available for EC2 Launch Type.

Example

Monitoring Agent

Security Agent

Log Collector

Not available for Fargate.

---

### Selected

Replica

Reason

Backend API.

---

## 6. Desired Tasks

Purpose

Number of Task copies ECS should maintain.

Examples

1

2

3

5

10

### Selected

1

Reason

Development environment.

Production

Usually 2 or more Tasks for High Availability.

---

## 7. Availability Zone Rebalancing

Purpose

Keeps Tasks balanced across multiple Availability Zones.

Options

Enabled ✅

Disabled

---

### Enabled

If one Availability Zone fails

↓

ECS launches Tasks in another zone.

Recommended

Production

Development

---

### Selected

Enabled

Reason

Improves availability.

---

## 8. Health Check Grace Period

Purpose

Time ECS waits before checking Task health.

Example

Container starts

↓

Needs 30 seconds

↓

Application becomes ready

Without Grace Period

Health Check fails immediately.

Typical Values

30

60

120 seconds

### Selected

Default

Reason

Application starts quickly.

Increase if startup is slow.

---

# Deployment Configuration

---

## 9. Deployment Type

Options

Rolling Update ✅

Blue/Green

Linear

Canary

---

### Rolling Update

Default deployment strategy.

Old Tasks

↓

Gradually replaced

↓

New Tasks

Advantages

Simple

Zero downtime

Recommended

Most applications.

### Selected

Rolling Update

---

### Blue/Green

Creates

Old Environment

New Environment

Traffic switches only after verification.

Advantages

Very safe deployment.

Used by

Large production systems.

---

### Linear

Updates Tasks gradually.

Example

25%

↓

50%

↓

75%

↓

100%

Useful when deploying slowly.

---

### Canary

Deploys a very small percentage first.

Example

10%

↓

Observe

↓

100%

Useful for detecting production bugs.

---

### Selected

Rolling Update

Reason

Simple and sufficient for development.

---

## 10. Minimum Running Tasks

Purpose

Minimum number of Tasks kept running during deployment.

Example

Desired Tasks = 2

Minimum = 100%

↓

2 Tasks stay running.

Production

100%

Development

Default

### Selected

100%

---

## 11. Maximum Running Tasks

Purpose

Maximum number of Tasks allowed during deployment.

Example

Desired Tasks = 2

Maximum = 200%

↓

AWS may temporarily run 4 Tasks while deploying.

Purpose

Zero downtime deployment.

### Selected

200%

---

## 12. Deployment Circuit Breaker

Purpose

Automatically rolls back failed deployments.

Options

Enabled

Disabled ✅

---

### Enabled

Deployment fails

↓

AWS automatically restores previous version.

Recommended

Production

---

### Disabled

Manual rollback.

### Selected

Disabled

Reason

Development environment.

---

# Networking

---

## 13. VPC

Purpose

Virtual network where ECS Tasks run.

### Selected

Existing VPC

Reason

Task and ALB must use the same VPC.

---

## 14. Subnets

Purpose

Network where Tasks run.

Recommendation

Select at least two Public Subnets.

Reason

Provides High Availability.

### Selected

2 Public Subnets

---

## 15. Security Group

Purpose

Acts as a firewall for ECS Tasks.

Controls

Inbound Traffic

Outbound Traffic

### Selected

Default Security Group (during initial setup)

Production

Create a dedicated Security Group.

---

## 16. Public IP

Options

Enabled ✅

Disabled

---

### Enabled

Assigns a Public IP to the Task.

Useful when

No Load Balancer exists yet.

Development

Testing

---

### Disabled

Task only has a Private IP.

Used when

Application is behind an Application Load Balancer.

Production Recommendation

Disabled

---

### Selected

Enabled (during initial deployment)

Reason

Allowed direct testing before configuring the ALB.

---

# Load Balancer

Initially

Do NOT configure.

Reason

We'll create the Application Load Balancer in the next steps.

### Selected

None

---

# Service Discovery

Purpose

Allows services to find each other using DNS.

Example

payment-service

↓

auth-service

↓

notification-service

Selected

Skipped

Reason

Only one backend service.

---

# Service Connect

Purpose

Simplifies communication between ECS Services.

Selected

Skipped

Reason

Not required for a single service.

---

# VPC Lattice

Purpose

Connects services across VPCs and AWS accounts.

Selected

Skipped

Reason

Not required.

---

# Auto Scaling

Purpose

Automatically increases or decreases the number of Tasks.

Example

CPU > 70%

↓

Launch another Task

Selected

Disabled

Reason

Development environment.

Production

Recommended.

---

# Volumes

Purpose

Attach persistent storage.

Selected

Default

Reason

Application stores no persistent files.

---

# Tags

Purpose

Organize AWS resources.

Examples

Environment=Development

Project=Backend

Owner=DevOps

Selected

Skipped

---

## Step 4.2 Create Service

Click

Create

AWS automatically performs the following:

1. Reads the Task Definition

↓

2. Pulls Docker Image from Amazon ECR

↓

3. Creates a Task

↓

4. Starts the Container

↓

5. Monitors the Task

↓

6. Restarts it automatically if it crashes

---

# Verify

Go to

Cluster

↓

Services

↓

backend-service

You should see

Status

Active

Desired Tasks

1

Running Tasks

1

Pending Tasks

0

---

Click

Tasks

You should see

Task Status

RUNNING

Public IP

Private IP

Container Name

Health

---

# Common Mistakes

❌ Wrong VPC

Result

ALB cannot communicate with ECS.

---

❌ Wrong Security Group

Result

Application becomes inaccessible.

---

❌ Desired Tasks = 0

Result

No containers start.

---

❌ Missing Task Definition

Result

Service cannot be created.

---

# Summary

After completing this step, you have:

✔ Created an ECS Service

✔ Connected it to the Task Definition

✔ Started your Docker Container

✔ Running 1 Task on AWS Fargate

Current Architecture

Developer
      │
      ▼
Docker Image
      │
      ▼
Amazon ECR
      │
      ▼
Task Definition
      │
      ▼
ECS Service
      │
      ▼
Running Task

Next Step

Create a **Target Group**, which allows the Application Load Balancer to route traffic to your ECS Tasks.
# Step 5: Create Target Group

## What is a Target Group?

A Target Group is a collection of servers (targets) that receive traffic from an Application Load Balancer (ALB).

Think of it as a bridge between the Load Balancer and your ECS Tasks.

Without a Target Group, the Load Balancer does not know where to send incoming requests.

---

## Architecture

                Internet
                    │
                    ▼
      Application Load Balancer
                    │
                    ▼
             Target Group
                    │
                    ▼
             ECS Running Task

---

## Why do we need a Target Group?

The ALB only receives incoming traffic.

The Target Group decides:

- Which ECS Tasks should receive traffic
- Whether the Tasks are healthy
- Which port to forward traffic to

---

## Step 5.1 Open Target Groups

AWS Console

↓

EC2

↓

Target Groups

↓

Create Target Group

---

# Target Group Configuration

---

## 1. Target Type

Purpose

Specifies what resources will receive traffic.

Options

### Instances

Registers EC2 Instances.

Example

EC2 running Node.js

↓

ALB

↓

EC2

Use When

Using ECS with EC2 Launch Type.

---

### IP Addresses ✅ (Selected)

Registers IP Addresses.

Every Fargate Task gets its own private IP.

Since our application runs on AWS Fargate, ALB communicates directly using the Task's IP Address.

Use When

- AWS Fargate
- ECS Tasks
- Containers

Advantages

✔ Recommended for ECS Fargate

✔ Automatically registers Task IPs

---

### Lambda

Routes requests directly to AWS Lambda.

Use When

Serverless Lambda applications.

---

### Selected

✅ IP Addresses

Reason

Our backend runs on AWS Fargate.

---

## 2. Target Group Name

Purpose

Unique name of the Target Group.

Examples

backend-target-group

resume-api-tg

payment-tg

auth-tg

### Selected

backend-target-group

Recommendation

Use meaningful names.

Avoid

tg1

test

abc

---

## 3. Protocol

Purpose

Defines how the Load Balancer communicates with your application.

Options

HTTP ✅

HTTPS

TCP

UDP

### Selected

HTTP

Reason

Our backend application uses HTTP on port 3000.

Production

Usually

HTTPS

for external users.

---

## 4. Port

Purpose

Port on which your application listens.

Example

Express

```javascript
app.listen(3000);
```

Port should match

3000

### Selected

3000

Reason

Our Node.js backend listens on port 3000.

Common Mistake

Choosing Port 80 while the application runs on Port 3000.

Result

Health Checks fail.

---

## 5. IP Address Type

Options

IPv4 ✅

IPv6

### Selected

IPv4

Reason

Standard deployment.

Use IPv6 only if your infrastructure requires it.

---

## 6. VPC

Purpose

Specifies where the Target Group searches for ECS Tasks.

Important

Must be the **same VPC** used by:

- ECS Cluster
- ECS Service
- Application Load Balancer

### Selected

Existing VPC

Reason

All resources must communicate within the same VPC.

---

## 7. Protocol Version

Options

HTTP1 ✅

HTTP2

gRPC

---

### HTTP1

Recommended for

Node.js

Express

React Backend

REST APIs

---

### HTTP2

Supports multiplexing.

Useful for modern applications.

---

### gRPC

Used only for gRPC services.

---

### Selected

HTTP1

Reason

Our backend is an Express REST API.

---

# Health Check Configuration

Health Checks determine whether your application is working correctly.

If a Task becomes unhealthy,

ALB stops sending traffic to it.

---

## 8. Health Check Protocol

Options

HTTP ✅

HTTPS

### Selected

HTTP

Reason

Application serves HTTP.

---

## 9. Health Check Path

Purpose

URL used by ALB to check whether the application is alive.

Examples

/

/health

/api/health

### Selected

/

Reason

Our backend responds on the root path.

Production Recommendation

Create a dedicated endpoint

```text
/health
```

Example

```javascript
app.get("/health", (req, res) => {
    res.status(200).send("Healthy");
});
```

---

## 10. Advanced Health Check Settings

Purpose

Controls how often AWS checks your application.

Default values are usually sufficient.

Options include

Healthy Threshold

Number of successful checks before marking the Task healthy.

Unhealthy Threshold

Number of failed checks before marking the Task unhealthy.

Timeout

How long ALB waits for a response.

Interval

Time between health checks.

Success Codes

Expected HTTP response codes.

### Selected

Default values

Reason

Suitable for development.

Production

Tune these values based on application startup time and traffic.

---

## 11. Traffic Port

Options

Traffic Port ✅

Override Port

### Traffic Port

Uses the same port configured above (3000).

Recommended.

### Override Port

Uses a different port only for health checks.

Rarely used.

### Selected

Traffic Port

---

## 12. Success Codes

Purpose

HTTP status codes considered healthy.

Default

200

Can also be

200-299

### Selected

200

Reason

Our backend returns HTTP 200.

---

## Step 5.2 Register Targets

AWS asks whether you want to register targets manually.

For ECS Fargate

**Do NOT register anything manually.**

Leave the target list empty.

Click

Next

↓

Create Target Group

Why?

ECS automatically registers running Tasks when we attach this Target Group to the ECS Service.

---

## Why was "Targets = 0"?

After creating the Target Group, you may see:

Targets

0

This is **expected**.

At this point:

✔ Target Group exists

❌ It is not yet connected to the ECS Service

Once we update the ECS Service and attach this Target Group,

AWS automatically registers the running Task.

The Target list will then show something like:

Private IP

10.0.12.45

Status

Healthy

---

## Verify

Go to

EC2

↓

Target Groups

↓

backend-target-group

You should see

Protocol

HTTP

Port

3000

Target Type

IP

Targets

0 (Expected for now)

---

# Common Mistakes

❌ Selecting "Instances" instead of "IP Addresses"

Result

Fargate Tasks cannot register.

---

❌ Using the wrong Port

Result

Health Checks fail.

---

❌ Using a different VPC

Result

ALB cannot reach ECS Tasks.

---

❌ Registering IP addresses manually

Result

Not required.

ECS automatically handles registration.

---

# Summary

After completing this step, you have:

✔ Created a Target Group

✔ Configured Health Checks

✔ Selected IP Address Target Type

✔ Configured Port 3000

✔ Left Targets empty (this is correct)

Current Architecture

Internet
    │
    ▼
Application Load Balancer
        │
        ▼
Target Group
        │
        ▼
(No Targets Yet)

In the next step, we'll create the **Application Load Balancer (ALB)** and then connect it to this Target Group and the ECS Service. Once connected, ECS will automatically register your running Task, and the Target Group status will change from **0 Targets** to **1 Healthy Target**.


# Step 6: Create an Application Load Balancer (ALB)

## What is an Application Load Balancer (ALB)?

An Application Load Balancer (ALB) distributes incoming HTTP/HTTPS traffic to one or more backend applications.

Instead of users accessing your ECS Task directly, they access the ALB.

The ALB then forwards requests to healthy ECS Tasks.

---

## Why do we need an ALB?

Without ALB

User
   │
   ▼
Public IP
   │
   ▼
ECS Task

Problems

- Single Point of Failure
- No Health Checks
- No Load Balancing
- No HTTPS Termination
- Public IP changes if Task restarts

---

With ALB

                Internet
                    │
                    ▼
      Application Load Balancer
                    │
          Health Checks
                    │
                    ▼
             Target Group
                    │
                    ▼
              ECS Service
                    │
                    ▼
              Running Task

Advantages

✔ One fixed DNS Name

✔ Health Checks

✔ Load Balancing

✔ SSL Support

✔ High Availability

✔ Automatic Failover

---

## Step 6.1 Open Load Balancer

AWS Console

↓

EC2

↓

Load Balancers

↓

Create Load Balancer

↓

Application Load Balancer

↓

Create

---

# Load Balancer Configuration

---

## 1. Load Balancer Name

Purpose

Unique name for the ALB.

Examples

backend-alb

resume-alb

payment-alb

auth-alb

### Selected

backend-alb

Recommendation

Always use meaningful names.

Avoid

alb1

test

abc

---

## 2. Scheme

Purpose

Determines whether the Load Balancer is accessible from the Internet.

Options

### Internet-facing ✅ (Selected)

Users on the Internet can access the Load Balancer.

Use When

- Public APIs
- Company Websites
- React Applications
- Node.js APIs

Example

Internet

↓

ALB

↓

Backend

Recommended

Most web applications.

---

### Internal

Only resources inside the VPC can access it.

Internet users cannot.

Use When

- Internal Microservices
- Internal Company Applications
- Private APIs

Example

Backend Service

↓

Internal ALB

↓

Database Service

---

### Selected

Internet-facing

Reason

Users should access our backend over the Internet.

---

## 3. IP Address Type

Purpose

Determines which IP version ALB supports.

Options

### IPv4 ✅

Supports IPv4 only.

Recommended for most projects.

---

### Dualstack

Supports both

- IPv4
- IPv6

Use When

Application must support IPv6 clients.

---

### Dualstack without Public IPv4

Uses IPv6 publicly while IPv4 remains private.

Rarely used.

---

### Selected

IPv4

Reason

Simple deployment.

---

## 4. VPC

Purpose

Network where ALB will be created.

Important

Must be the **same VPC** used by

- ECS Cluster
- ECS Service
- Target Group

### Selected

Existing VPC

Reason

All resources must communicate within one VPC.

---

## 5. Availability Zones

Purpose

Specifies where AWS creates the Load Balancer.

Recommendation

Select **at least two Availability Zones**.

Example

☑ us-west-1a

☑ us-west-1b

Why?

If one Availability Zone fails,

ALB continues serving traffic from the other zone.

### Selected

Both available subnets.

---

## 6. Security Group

Purpose

Acts as a firewall for the Load Balancer.

Controls

Inbound

Outbound

### Selected

Default Security Group (Development)

Production

Create a dedicated Security Group.

Example

Inbound

HTTP (80)

HTTPS (443)

Source

0.0.0.0/0

Outbound

All Traffic

---

## 7. Listener

Purpose

Specifies which port ALB listens on.

Options

HTTP : 80

HTTPS : 443

### Selected

HTTP : 80

Reason

We haven't configured SSL yet.

Production Recommendation

HTTPS : 443 using ACM Certificate.

---

## 8. Default Action

Purpose

Determines where traffic should go.

Options

Forward to Target Group

Redirect

Fixed Response

### Selected

Forward to Target Group

Reason

Traffic should go to our ECS application.

---

## 9. Target Group

Select

backend-target-group

Purpose

ALB forwards all incoming requests to this Target Group.

### Selected

backend-target-group

---

## 10. Listener Rules

Purpose

Allows routing based on

- Path
- Hostname
- Headers

Examples

/api/*

↓

Backend API

/admin/*

↓

Admin Service

/images/*

↓

Image Service

Selected

Default Rule

Reason

Only one backend service.

---

## 11. IPAM Pools

Purpose

AWS IP Address Manager.

Allows centralized management of IP addresses.

Use When

Large organizations managing many VPCs.

### Selected

Skipped

Reason

Not required.

---

## 12. AWS WAF

Purpose

Protects applications against

- SQL Injection
- Cross Site Scripting (XSS)
- Bots
- Malicious Requests

Use When

Production

Enterprise

Public Applications

### Selected

Skipped

Reason

Can be added later.

---

## 13. CloudFront

Purpose

Global CDN that caches static content closer to users.

Use When

- React Applications
- Static Websites
- Images
- Videos

Our backend

Not required.

Selected

Skipped.

---

## 14. Global Accelerator

Purpose

Improves worldwide performance by routing users through AWS's global network.

Use When

Applications serving users from multiple countries.

Selected

Skipped.

---

## 15. Tags

Purpose

Organize AWS Resources.

Examples

Environment=Development

Project=Backend

Owner=DevOps

Selected

Skipped

---

## Step 6.2 Create Load Balancer

Click

Create Load Balancer

AWS automatically creates

- ALB
- DNS Name
- Listeners
- Security Group Association

---

## Verify

Go to

EC2

↓

Load Balancers

↓

backend-alb

Verify

State

Active

Scheme

Internet-facing

Listener

HTTP : 80

Target Group

backend-target-group

DNS Name

Example

backend-alb-123456789.us-west-1.elb.amazonaws.com

Do not use the ECS Task Public IP anymore.

Instead use the ALB DNS Name after connecting the ECS Service.

---

# Common Mistakes

❌ Selecting Internal instead of Internet-facing

Result

Application is not accessible from the Internet.

---

❌ Choosing the wrong VPC

Result

ALB cannot communicate with ECS Tasks.

---

❌ Selecting only one Availability Zone

Result

No High Availability.

---

❌ Forgetting to attach the Target Group

Result

ALB has nowhere to send traffic.

---

# Summary

After completing this step, you have

✔ Created an Application Load Balancer

✔ Configured an Internet-facing ALB

✔ Selected IPv4

✔ Configured HTTP Listener on Port 80

✔ Attached the backend Target Group

✔ Created an ALB DNS Name

Current Architecture

Internet
    │
    ▼
Application Load Balancer
        │
        ▼
Target Group
        │
        ▼
(No ECS Tasks Attached Yet)

Next Step

Update the ECS Service to attach the Load Balancer and Target Group. ECS will then automatically register the running Task, making the application accessible through the ALB DNS Name.

# Step 7: Connect ECS Service with the Application Load Balancer

## Why is this step required?

Until now, we have created:

✔ Amazon ECR Repository

✔ ECS Cluster

✔ Task Definition

✔ ECS Service

✔ Target Group

✔ Application Load Balancer

However,

the ECS Service **does not yet know** that it should use the Load Balancer.

Similarly,

the Target Group **does not know** which ECS Tasks should receive traffic.

This step connects everything together.

---

## Before Connection

Internet
    │
    ▼
Application Load Balancer

(No Backend Connected)

Target Group

(No Targets)

↓

ECS Service

↓

Running Task

Users cannot access the application.

---

## After Connection

Internet
      │
      ▼
Application Load Balancer
      │
HTTP : 80
      │
      ▼
backend-target-group
      │
      ▼
ECS Service
      │
      ▼
Running Task

Users can now access the application using the ALB DNS Name.

---

# Step 7.1

Open

AWS Console

↓

Amazon ECS

↓

Clusters

↓

backend-cluster

↓

Services

↓

backend-service

↓

Update

---

# Load Balancing Section

Scroll down until you see

Load Balancing

Enable

Use Load Balancing

---

## 1. Use Load Balancing

Purpose

Allows ECS Service to register Tasks with an Application Load Balancer.

Options

Enabled ✅

Disabled

### Selected

Enabled

Reason

We want users to access the application through ALB.

---

## 2. Load Balancer Type

Options

Application Load Balancer ✅

Network Load Balancer

Gateway Load Balancer

---

### Application Load Balancer

Routes HTTP and HTTPS requests.

Supports

- REST APIs
- Websites
- Path Routing
- Host Routing

Recommended

Node.js

React

Express

Spring Boot

Laravel

ASP.NET

---

### Network Load Balancer

Layer 4

Supports

TCP

UDP

TLS

Use When

Gaming

VoIP

Very High Performance

---

### Gateway Load Balancer

Used with Firewalls

Security Appliances

Rarely used.

---

### Selected

Application Load Balancer

Reason

Backend API uses HTTP.

---

## 3. Container

Purpose

Specifies which container receives traffic.

Options

Lists containers from the Task Definition.

Example

backend

3000:3000

### Selected

backend

Reason

This is our application container.

---

## 4. Container Port

Purpose

Specifies the port inside the container.

Must match

```javascript
app.listen(3000);
```

Selected

3000

Reason

Express listens on Port 3000.

---

## 5. Load Balancer

Purpose

Choose the ALB created earlier.

Selected

backend-alb

Reason

This is our Internet-facing Load Balancer.

---

## 6. Listener

Purpose

Specifies which listener receives incoming requests.

Options

HTTP : 80

HTTPS : 443

Selected

HTTP : 80

Reason

SSL has not been configured yet.

Production

HTTPS : 443

---

## 7. Target Group

Purpose

Determines where ALB forwards requests.

Selected

backend-target-group

Reason

Created in Step 5.

---

# Step 7.2

Click

Update

AWS now performs several operations automatically.

---

## What Happens Internally?

Step 1

ECS updates the Service.

↓

Step 2

New Deployment starts.

↓

Step 3

Task registers with Target Group.

↓

Step 4

Target Group starts Health Checks.

↓

Step 5

If Health Checks pass,

Status becomes

Healthy

↓

Step 6

ALB starts forwarding traffic.

---

## Deployment Process

Old Service

↓

Updating

↓

Deployment In Progress

↓

Running

↓

Completed

Wait until

Deployment Status

Completed

Do not test before deployment finishes.

---

# Verify ECS

Go to

Amazon ECS

↓

Clusters

↓

backend-cluster

↓

Services

↓

backend-service

Verify

Status

Active

Desired Tasks

1

Running Tasks

1

Deployment

Completed

---

# Verify Running Task

Go to

Tasks

↓

Running Task

Verify

Task Status

RUNNING

Health

Healthy

Container

Running

---

# Verify Target Group

Go to

EC2

↓

Target Groups

↓

backend-target-group

↓

Targets

You should now see

Private IP

10.x.x.x

Health Status

Healthy

Initially

Targets

0

After connecting ECS

↓

Targets

1 Healthy

This confirms ECS automatically registered the Task.

---

# Verify Load Balancer

Go to

EC2

↓

Load Balancers

↓

backend-alb

↓

Copy

DNS Name

Example

backend-alb-123456789.us-west-1.elb.amazonaws.com

Open

http://YOUR-ALB-DNS

If your application responds on `/`, you should see your backend response.

Example

```text
Server Running
```

or

```json
{
    "message":"API Running"
}
```

depending on your application.

---

# Common Problems

## Target Group shows 0 Targets

Reason

Service was not updated with the Target Group.

Solution

Update ECS Service again and select

- backend-alb
- HTTP : 80
- backend-target-group

---

## Target Status = Unhealthy

Possible Reasons

Wrong Port

Wrong Health Check Path

Application crashed

Security Group blocking traffic

Wrong VPC

---

## Browser Times Out

Possible Reasons

ALB Security Group

Port 80 not allowed

↓

ECS Security Group

Port 3000 blocked

↓

Application not listening on

0.0.0.0

Correct

```javascript
app.listen(PORT, "0.0.0.0");
```

---

## ECS Task Stops Immediately

Possible Reasons

Wrong Environment Variables

Database Connection Failed

Wrong Docker Image

Application Crash

Check

CloudWatch Logs

---

# Summary

After completing this step, you have successfully connected every AWS service.

Final Architecture

Internet
     │
     ▼
Application Load Balancer
     │
HTTP : 80
     │
     ▼
Target Group
     │
Health Checks
     │
     ▼
ECS Service
     │
     ▼
Running Task
     │
     ▼
Docker Container
     │
     ▼
Amazon ECR Image

Your application is now accessible through the **Application Load Balancer DNS Name** instead of the ECS Task's public IP.

# Step 8: Verify Deployment & Troubleshooting

After connecting the ECS Service with the Application Load Balancer, the final step is to verify that everything is working correctly.

Do **not** assume the deployment is successful just because the Service was created.

Always verify each AWS resource.

---

# Verification Flow

Docker Image
      │
      ▼
Amazon ECR
      │
      ▼
Task Definition
      │
      ▼
ECS Service
      │
      ▼
Running Task
      │
      ▼
Target Group
      │
      ▼
Application Load Balancer
      │
      ▼
Browser

If any step fails, the application will not be accessible.

---

# Step 8.1 Verify Amazon ECR

Open

AWS Console

↓

Amazon ECR

↓

Repositories

↓

backend

Verify

✔ Repository Exists

✔ Image Exists

✔ Image Tag

latest

✔ Image URI

Example

123456789012.dkr.ecr.us-west-1.amazonaws.com/backend:latest

If no image exists

Run

```bash
docker push <your-ecr-uri>:latest
```

---

# Step 8.2 Verify ECS Cluster

Open

Amazon ECS

↓

Clusters

↓

backend-cluster

Verify

Cluster Status

Active

Services

1

Running Tasks

1

Pending Tasks

0

If Running Tasks = 0

Problem exists in

- Task Definition
- ECS Service
- Docker Image

---

# Step 8.3 Verify ECS Service

Open

Clusters

↓

backend-cluster

↓

Services

↓

backend-service

Verify

Status

Active

Deployment

Completed

Desired Tasks

1

Running Tasks

1

Pending Tasks

0

If Deployment is still

In Progress

Wait until deployment finishes.

---

# Step 8.4 Verify Running Task

Open

Tasks

↓

Running Task

Verify

Task Status

RUNNING

Health

Healthy

Launch Type

Fargate

Container

Running

If Task Status

STOPPED

Click

Stopped Task

↓

Stopped Reason

Common Reasons

CannotPullContainerError

Task Execution Role missing

Wrong Image URI

Application Crash

Database Connection Failed

Missing Environment Variables

---

# Step 8.5 Verify Container Logs

Open

Running Task

↓

Logs

or

CloudWatch

↓

Log Groups

↓

ECS Log Group

Look for

Server started

Database Connected

Application Running

If you see

Error

Fix the application first.

Examples

Database connection refused

Port already in use

Environment Variable Missing

Cannot connect to Redis

---

# Step 8.6 Verify Target Group

Open

EC2

↓

Target Groups

↓

backend-target-group

↓

Targets

Expected

Private IP

↓

Healthy

Example

10.0.15.45

Healthy

---

If Status

Healthy

Everything is working.

---

If Status

Unhealthy

Click the Target

↓

View Health Check Details

Possible Reasons

Wrong Port

Wrong Health Check Path

Application Not Running

Security Group

Wrong VPC

---

If Targets

0

Reason

ECS Service is not attached to the Target Group.

Solution

Update ECS Service

↓

Attach

backend-alb

↓

backend-target-group

---

# Step 8.7 Verify Application Load Balancer

Open

EC2

↓

Load Balancers

↓

backend-alb

Verify

State

Active

Scheme

Internet-facing

Listener

HTTP : 80

Target Group

backend-target-group

DNS Name

Available

Example

backend-alb-123456789.us-west-1.elb.amazonaws.com

---

# Step 8.8 Test Application

Copy

ALB DNS Name

Open Browser

http://backend-alb-xxxxxxxx.us-west-1.elb.amazonaws.com

Expected

Your backend response.

Example

```text
API Running
```

or

```json
{
    "message":"Server Running"
}
```

Do NOT use

ECS Task Public IP

Always use

ALB DNS Name

---

# Step 8.9 Verify Health Checks

Target Group

↓

Health Checks

Verify

Protocol

HTTP

Port

3000

Path

/

Success Code

200

If your application has

/health

Use

/health

instead of

/

Production Recommendation

Always create

```text
/health
```

---

# Troubleshooting

## Problem 1

Cannot Pull Docker Image

Error

CannotPullContainerError

Reasons

Wrong Image URI

Image Not Pushed

Execution Role Missing

Wrong Region

Solution

Verify

ECR Repository

↓

Image Exists

↓

Execution Role

ecsTaskExecutionRole

---

## Problem 2

Task Stops Immediately

Reasons

Application Crash

Database Error

Wrong Environment Variables

Port Conflict

Solution

Check

CloudWatch Logs

---

## Problem 3

Browser Timeout

Reasons

ALB Security Group

↓

Port 80 blocked

or

Task Security Group

↓

Port 3000 blocked

or

Application listening on

127.0.0.1

Correct

```javascript
app.listen(PORT, "0.0.0.0");
```

---

## Problem 4

Target Unhealthy

Reasons

Wrong Port

Wrong Health Check Path

Application Crash

Wrong VPC

Wrong Security Group

Solution

Verify

Port

3000

↓

Path

/

↓

Task Running

↓

Security Group

---

## Problem 5

502 Bad Gateway

Reason

ALB reached the Target

Application failed.

Usually

Wrong Port

Application Crash

Container Not Running

---

## Problem 6

503 Service Unavailable

Reason

No Healthy Targets.

Solution

Verify

Target Group

↓

Targets

↓

Healthy

---

## Problem 7

403 Forbidden

Reason

Application authentication

or

Security configuration.

---

## Problem 8

404 Not Found

Reason

Requested route does not exist.

Verify

Application Routes.

---

## Problem 9

Deployment Never Completes

Reasons

Health Checks Failing

Container Crash

Target Never Healthy

Wrong Image

Solution

Check

CloudWatch Logs

↓

Target Group

↓

Task Status

---

# Deployment Checklist

Before considering deployment successful, verify:

☑ Docker Image pushed to Amazon ECR

☑ ECS Cluster Active

☑ Task Definition Created

☑ ECS Service Active

☑ Running Task = 1

☑ Task Status = RUNNING

☑ Target Group = Healthy

☑ ALB Status = Active

☑ ALB DNS Name Accessible

☑ Application Responding

☑ CloudWatch Logs Showing No Errors

---

# Best Practices

✔ Use meaningful resource names

✔ Use Private ECR repositories

✔ Use Immutable Image Tags in Production

✔ Enable Image Scan on Push

✔ Use AWS Fargate unless EC2 is required

✔ Enable CloudWatch Logs

✔ Create a dedicated `/health` endpoint

✔ Use HTTPS (ACM Certificate) in Production

✔ Disable Public IP when using ALB

✔ Use at least two Availability Zones

✔ Use Auto Scaling for Production

✔ Create dedicated Security Groups

✔ Store secrets in AWS Secrets Manager

✔ Never hardcode credentials in Task Definitions

✔ Monitor ECS Tasks using CloudWatch

✔ Keep Task Definitions versioned

✔ Test deployments after every update

---

# Final Architecture

                    Internet
                        │
                        ▼
          Application Load Balancer
                HTTP / HTTPS
                        │
                        ▼
              Target Group (Healthy)
                        │
                        ▼
                Amazon ECS Service
                        │
                        ▼
              Running ECS Task (Fargate)
                        │
                        ▼
               Docker Container (Node.js)
                        │
                        ▼
              Docker Image (Amazon ECR)

Congratulations! 🎉

You have successfully deployed a Dockerized application on AWS using:

- Amazon ECR
- Amazon ECS (Fargate)
- Target Groups
- Application Load Balancer (ALB)

Your application is now accessible through the **Application Load Balancer DNS Name**, which is the recommended approach for production deployments.
