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
└── Other customer's VPCs
```

# Step 1: Create a VPC

When you create this

```text
CIDR : 10.0.0.0/16
```

you're basically saying

> "Give me a private network with 65,536 IP addresses."

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

Nothing can exist until there is a VPC.

Think of it like buying land before building houses.

# Step 2: Create Subnets

A subnet divides the VPC into smaller networks.

Example

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

Why?

Because every server doesn't need internet.

Example

```text
Internet

↓

Frontend Server

↓

Backend Server

↓

Database
```

Only frontend should be reachable by users.

Backend and DB should stay hidden.

# Step 3: Internet Gateway (IGW)

Internet Gateway is simply the door between your VPC and the Internet.

Without it

```text
Internet

❌

VPC
```

Nobody can enter.

With it

```text
Internet

↓

Internet Gateway

↓

VPC
```

Now traffic can enter.

# Step 4: Route Table

This is like Google Maps.

Every packet asks

> "Where should I go?"

Example

```text
Destination        Next Hop

0.0.0.0/0    ---> Internet Gateway
10.0.0.0/16 ---> Local
```

Meaning

If destination is internet

```text
Go to Internet Gateway
```

If destination is inside VPC

```text
Stay inside VPC
```

Without Route Table

The packet has no idea where to go.

# Step 5: Public Subnet

Suppose

```text
EC2

IP

10.0.1.25
```

Public subnet has

```text
Route

0.0.0.0/0

↓

Internet Gateway
```

Now

```text
Internet

↓

IGW

↓

Public Subnet

↓

EC2
```

Users can access it.

# Step 6: Private Subnet

Private subnet has

```text
No route

to Internet Gateway
```

So

```text
Internet

❌

Private EC2
```

Nobody can directly reach it.

Perfect for

- Backend
- Database
- Redis

# Step 7: NAT Gateway

Question

If backend is private...

How will it install npm packages?

How will Ubuntu run

```bash
sudo apt update
```

It still needs internet.

That's where NAT Gateway comes.

```text
Private EC2

↓

NAT Gateway

↓

Internet Gateway

↓

Internet
```

Notice

Internet cannot initiate a connection back to the private EC2.

Private EC2 can only make outbound requests.

# Step 8: Security Group

Security Group is like a security guard at the building entrance.

Example

```text
Allow

HTTP 80

HTTPS 443

SSH 22
```

Everything else

```text
Denied
```

Example

```text
Internet

↓

Port 80 ✅

↓

EC2
```

```text
Internet

↓

Port 3306 ❌

↓

Blocked
```

# Step 9: Network ACL

Think of it as security at the colony gate.

Security Group protects the server.

Network ACL protects the subnet.

```text
Internet

↓

NACL

↓

Subnet

↓

Security Group

↓

EC2
```

AWS checks both.

# Now let's build a Node.js application

Imagine your MERN backend.

```text
React

↓

Node.js API

↓

PostgreSQL
```

AWS architecture

```text
Internet

↓

Route53 (optional)

↓

CloudFront (optional)

↓

Application Load Balancer

↓

Public Subnet

↓

Node.js EC2

↓

Private Subnet

↓

PostgreSQL (RDS)
```

# What happens when a user sends a request?

Let's say the user visits

```text
https://api.myapp.com/users
```

Flow

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

Load Balancer

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

This is the complete request flow.
