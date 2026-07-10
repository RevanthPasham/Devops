# AWS VPC Explained (Simple Guide)

This guide explains how an AWS VPC works, what each networking component does, and how a request reaches a Node.js backend application.

---

# AWS Cloud Structure

```text
AWS Cloud
│
├── Your VPC (Private Network)
│      │
│      ├── Public Subnet
│      ├── Private Subnet
│      ├── Route Tables
│      ├── Internet Gateway
│      ├── NAT Gateway
│      ├── Security Groups
│      └── Network ACLs
│
└── Other Customer's VPCs
```

Think of a **VPC (Virtual Private Cloud)** as your own private data center inside AWS.

---

# Step 1: Create a VPC

When creating a VPC, you'll specify a CIDR block.

Example:

```text
CIDR: 10.0.0.0/16
```

This means:

> "Create a private network with 65,536 available IP addresses."

Example:

```text
VPC
10.0.0.0/16

Possible IPs

10.0.0.1
10.0.0.2
10.0.0.3
...
10.0.255.255
```

Without a VPC, none of your AWS resources can communicate privately.

Think of a VPC like buying land before building houses.

---

# Step 2: Create Subnets

A subnet divides the VPC into smaller networks.

Example:

```text
VPC
10.0.0.0/16

│
├── Public Subnet
│      10.0.1.0/24
│
├── Private Subnet
│      10.0.2.0/24
│
└── Database Subnet
       10.0.3.0/24
```

## Why create multiple subnets?

Not every server should be exposed to the internet.

Example architecture:

```text
Internet

↓

Frontend Server

↓

Backend Server

↓

Database
```

- Frontend should be accessible to users.
- Backend should stay private.
- Database should never be directly accessible from the internet.

---

# Step 3: Internet Gateway (IGW)

An Internet Gateway connects your VPC to the internet.

Without an Internet Gateway:

```text
Internet

❌

VPC
```

Nobody can reach your VPC.

With an Internet Gateway:

```text
Internet

↓

Internet Gateway

↓

VPC
```

Now internet traffic can enter and leave the VPC.

---

# Step 4: Route Table

A Route Table tells packets where they should go.

Think of it like Google Maps.

Example:

```text
Destination        Next Hop

0.0.0.0/0    ---> Internet Gateway
10.0.0.0/16 ---> Local
```

Meaning:

If destination is the internet:

```text
Go to Internet Gateway
```

If destination is inside the VPC:

```text
Stay inside the VPC
```

Without a Route Table, packets don't know where to go.

---

# Step 5: Public Subnet

Suppose you launch an EC2 instance.

```text
EC2

IP:
10.0.1.25
```

Its subnet contains this route:

```text
0.0.0.0/0

↓

Internet Gateway
```

Traffic flow:

```text
Internet

↓

Internet Gateway

↓

Public Subnet

↓

EC2
```

Users can reach this EC2 from the internet.

Typical resources placed here:

- Load Balancer
- Bastion Host
- Public EC2
- NAT Gateway

---

# Step 6: Private Subnet

Private subnets have **no direct route** to the Internet Gateway.

```text
Internet

❌

Private EC2
```

Resources inside cannot be directly accessed from the internet.

Perfect for:

- Backend APIs
- Databases
- Redis
- Internal services

---

# Step 7: NAT Gateway

Question:

If a backend server is private...

How can it run:

```bash
sudo apt update
```

or

```bash
npm install
```

It still needs internet access.

That's the job of a NAT Gateway.

```text
Private EC2

↓

NAT Gateway

↓

Internet Gateway

↓

Internet
```

Important:

- ✅ Private EC2 can access the internet.
- ❌ Internet cannot directly access the Private EC2.

The connection is one-way (outbound only).

---

# Step 8: Security Group

A Security Group acts as a firewall for an EC2 instance.

Example rules:

```text
Allow

HTTP   80
HTTPS  443
SSH    22
```

Everything else is blocked.

Example:

Allowed:

```text
Internet

↓

Port 80

↓

EC2
```

Blocked:

```text
Internet

↓

Port 3306

↓

Blocked
```

Security Groups are **stateful**, meaning response traffic is automatically allowed.

---

# Step 9: Network ACL (NACL)

A Network ACL protects the subnet.

Think of it as the security gate for the entire neighborhood.

```text
Internet

↓

Network ACL

↓

Subnet

↓

Security Group

↓

EC2
```

Difference:

- Network ACL protects the subnet.
- Security Group protects individual resources.

AWS checks both before allowing traffic.

---

# Building a Node.js Application on AWS

Example MERN backend:

```text
React

↓

Node.js API

↓

PostgreSQL
```

Typical AWS architecture:

```text
Internet

↓

Route 53 (DNS)

↓

CloudFront (Optional)

↓

Application Load Balancer

↓

Public Subnet

↓

Node.js EC2

↓

Private Subnet

↓

PostgreSQL (Amazon RDS)
```

---

# Complete Request Flow

Suppose a user visits:

```text
https://api.myapp.com/users
```

The request flows like this:

```text
Browser

↓

Internet

↓

AWS

↓

Internet Gateway

↓

Route Table

↓

Public Subnet

↓

Application Load Balancer

↓

Security Group

↓

Node.js Server

↓

Express

↓

Route

↓

Controller

↓

Service

↓

Database

↓

Response

↓

Browser
```

---

# Inside the Node.js Server

Example Express route:

```javascript
app.get("/users", getUsers);
```

Request flow:

```text
Browser

↓

GET /users

↓

Express Router

↓

usersController.js

↓

userService.js

↓

PostgreSQL

↓

JSON Response

↓

Browser
```

---

# AWS Route Table vs Express Router

Many beginners confuse these.

## AWS Route Table

Routes **network traffic**.

Example:

```text
Packet

↓

Where should I go?

↓

Internet Gateway

or

Local Network
```

It only decides the next network destination.

---

## Express Router

Routes **HTTP requests** inside your application.

Example:

```text
Browser

↓

GET /login

↓

Express Router

↓

loginController()

↓

Response
```

The Route Table never looks at `/login` or `/users`.

Express Router never decides where network packets go.

They solve completely different problems.

---

# Complete End-to-End Architecture

```text
User

↓

DNS (Route 53)

↓

Internet

↓

Internet Gateway

↓

Route Table

↓

Public Subnet

↓

Application Load Balancer

↓

Security Group

↓

EC2 (Node.js)

↓

Express Router

↓

Controller

↓

Business Logic

↓

Amazon RDS (PostgreSQL)

↓

Controller

↓

Express

↓

Load Balancer

↓

Internet

↓

User
```

---

# AWS Components Summary

| Component | Purpose | Example |
|------------|---------|----------|
| VPC | Private network | Entire application |
| Subnet | Divide network | Public / Private |
| Internet Gateway | Connect VPC to Internet | Public access |
| Route Table | Decide network path | Internet or Local |
| NAT Gateway | Internet for private resources | `apt update`, `npm install` |
| Security Group | Firewall for EC2 | Allow 80, 443, 22 |
| Network ACL | Firewall for subnet | Block or allow subnet traffic |
| EC2 | Virtual machine | Run Node.js |
| Amazon RDS | Managed database | PostgreSQL |
| Application Load Balancer | Distribute traffic | Multiple EC2 instances |
| Route 53 | DNS | `api.example.com` |
| CloudFront | CDN | Faster content delivery |

---

# Key Takeaways

- A VPC is your private network inside AWS.
- Subnets divide the network into public and private sections.
- Internet Gateway provides internet connectivity.
- Route Tables decide where network packets travel.
- Public Subnets are internet-accessible.
- Private Subnets hide sensitive resources.
- NAT Gateway allows private resources to access the internet safely.
- Security Groups protect individual resources.
- Network ACLs protect entire subnets.
- Express Router handles application routes like `/users`, while AWS Route Tables handle network routing.
- Together, these components provide a secure and scalable architecture for applications like Node.js APIs running on AWS.
