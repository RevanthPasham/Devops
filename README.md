# AWS VPC + EC2 Node.js Backend + Private RDS PostgreSQL

This project demonstrates how to build a production-style AWS network architecture where:

- A Node.js backend runs on an EC2 instance.
- EC2 is accessible from the internet.
- PostgreSQL runs on AWS RDS.
- RDS is private and is not directly accessible from the internet.
- Only the EC2 backend is allowed to connect to RDS.
- AWS Security Groups control which resources can communicate.

---

# 1. Final Architecture

```text
                            INTERNET
                                │
                                │ HTTP / HTTPS
                                │
                                ▼
                         Internet Gateway
                                │
                                ▼
┌────────────────────────── AWS VPC ──────────────────────────┐
│                                                            │
│  VPC CIDR: 10.0.0.0/16                                     │
│                                                            │
│  ┌──────────────── PUBLIC SUBNET ──────────────────────┐    │
│  │                                                     │    │
│  │  public-subnet-1                                    │    │
│  │  10.0.1.0/24                                       │    │
│  │  Availability Zone: us-west-1a                      │    │
│  │                                                     │    │
│  │           EC2 Node.js Backend                       │    │
│  │           Security Group: backend-sg                │    │
│  │                                                     │    │
│  └───────────────────────┬─────────────────────────────┘    │
│                          │                                  │
│                          │ PostgreSQL Port 5432             │
│                          │ Private VPC Network              │
│                          ▼                                  │
│  ┌────────────── PRIVATE DB SUBNETS ───────────────────┐    │
│  │                                                     │    │
│  │  private-db-subnet-1                                │    │
│  │  10.0.2.0/24                                       │    │
│  │  us-west-1a                                        │    │
│  │                                                     │    │
│  │  private-db-subnet-2                                │    │
│  │  10.0.3.0/24                                       │    │
│  │  us-west-1b                                        │    │
│  │                                                     │    │
│  │              RDS PostgreSQL                         │    │
│  │              Security Group: rds-sg                 │    │
│  │              Public Access: NO                      │    │
│  │                                                     │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

# 2. Request Flow

When a customer uses the application:

```text
Customer
   │
   │ HTTPS
   ▼
Internet
   │
   ▼
Internet Gateway
   │
   ▼
EC2
   │
   │ Node.js Backend
   │
   │ PostgreSQL connection
   ▼
Private RDS PostgreSQL
```

The customer never connects directly to PostgreSQL.

```text
Customer ────────────────► EC2 Backend    ✅

EC2 Backend ─────────────► RDS            ✅

Customer ────────────────► RDS            ❌

Random Internet User ────► RDS            ❌
```

---

# 3. Components We Will Create

```text
AWS
│
└── nodejs-vpc
    │
    ├── public-subnet-1
    │   └── EC2 Node.js Backend
    │
    ├── private-db-subnet-1
    │
    ├── private-db-subnet-2
    │
    ├── nodejs-igw
    │
    ├── public-route-table
    │
    ├── backend-sg
    │
    ├── rds-sg
    │
    ├── nodejs-db-subnet-group
    │
    └── RDS PostgreSQL
```

---

# 4. IP Address Plan

We will use:

```text
VPC
10.0.0.0/16
│
├── Public Subnet
│   10.0.1.0/24
│
├── Private DB Subnet 1
│   10.0.2.0/24
│
└── Private DB Subnet 2
    10.0.3.0/24
```

## Why `/16` for the VPC?

```text
10.0.0.0/16
```

gives us a large private network range that can later be divided into smaller subnets.

For example:

```text
10.0.1.0/24 → Public servers

10.0.2.0/24 → Database subnet

10.0.3.0/24 → Database subnet in another AZ

10.0.4.0/24 → Future backend subnet

10.0.5.0/24 → Future worker subnet
```

The exact ranges are our network design choice.

---

# STEP 1 — Create the VPC

Go to:

```text
AWS Console
→ VPC
→ Your VPCs
→ Create VPC
```

Choose:

```text
Resources to create:
VPC only
```

Enter:

```text
Name tag:
nodejs-vpc

IPv4 CIDR:
10.0.0.0/16

IPv6 CIDR:
No IPv6 CIDR block

Tenancy:
Default
```

Click:

```text
Create VPC
```

---

## What did this step create?

We created our own private network inside AWS.

```text
AWS
│
├── Default VPC
│
└── nodejs-vpc
    10.0.0.0/16
```

At this point, the VPC is only a network boundary.

It does not yet have:

```text
❌ Public subnet
❌ Private subnet
❌ Internet Gateway
❌ EC2
❌ RDS
```

---

## Why is a VPC useful?

A VPC allows us to control:

```text
Where resources live

Which IP ranges are used

Which resources can access the internet

Which resources remain private

How resources communicate

How traffic is routed
```

Think of it as:

```text
VPC = Private network inside AWS
```

---

# STEP 2 — Create the Public Subnet

Go to:

```text
VPC
→ Subnets
→ Create subnet
```

Select:

```text
VPC:
nodejs-vpc
```

Enter:

```text
Subnet name:
public-subnet-1

Availability Zone:
us-west-1a

IPv4 subnet CIDR:
10.0.1.0/24
```

Click:

```text
Create subnet
```

---

## What did this step create?

```text
nodejs-vpc
10.0.0.0/16
│
└── public-subnet-1
    10.0.1.0/24
    us-west-1a
```

This subnet will contain our:

```text
EC2 Node.js Backend
```

---

## Important

Simply naming a subnet:

```text
public-subnet-1
```

does not make it public.

A subnet becomes public because of its routing configuration.

Later we will connect it to:

```text
Internet Gateway
```

through:

```text
Route Table
```

---

# STEP 3 — Create Private DB Subnet 1

Go to:

```text
VPC
→ Subnets
→ Create subnet
```

Select:

```text
VPC:
nodejs-vpc
```

Enter:

```text
Subnet name:
private-db-subnet-1

Availability Zone:
us-west-1a

IPv4 subnet CIDR:
10.0.2.0/24
```

Click:

```text
Create subnet
```

---

## Current architecture

```text
nodejs-vpc
10.0.0.0/16
│
├── public-subnet-1
│   10.0.1.0/24
│   us-west-1a
│
└── private-db-subnet-1
    10.0.2.0/24
    us-west-1a
```

---

# STEP 4 — Create Private DB Subnet 2

Create another subnet.

Enter:

```text
Subnet name:
private-db-subnet-2

Availability Zone:
us-west-1b

IPv4 subnet CIDR:
10.0.3.0/24
```

Click:

```text
Create subnet
```

---

## Why do we create two DB subnets?

RDS DB subnet groups are designed to span multiple Availability Zones.

Our architecture becomes:

```text
nodejs-vpc
10.0.0.0/16
│
├── public-subnet-1
│   10.0.1.0/24
│   us-west-1a
│
├── private-db-subnet-1
│   10.0.2.0/24
│   us-west-1a
│
└── private-db-subnet-2
    10.0.3.0/24
    us-west-1b
```

Two Availability Zones provide AWS with placement and future failover options.

For example:

```text
Availability Zone 1
us-west-1a
    │
    └── private-db-subnet-1


Availability Zone 2
us-west-1b
    │
    └── private-db-subnet-2
```

---

# STEP 5 — Create an Internet Gateway

Our VPC currently has no direct internet gateway.

Go to:

```text
VPC
→ Internet gateways
→ Create internet gateway
```

Enter:

```text
Name:
nodejs-igw
```

Click:

```text
Create internet gateway
```

Now select the Internet Gateway.

Go to:

```text
Actions
→ Attach to a VPC
```

Select:

```text
nodejs-vpc
```

Click:

```text
Attach internet gateway
```

---

## What did this step do?

Before:

```text
Internet

    X

nodejs-vpc
```

After:

```text
Internet
    │
    ▼
nodejs-igw
    │
    ▼
nodejs-vpc
```

However, traffic still does not automatically know where to go.

We need a:

```text
Route Table
```

---

# STEP 6 — Create the Public Route Table

Go to:

```text
VPC
→ Route tables
→ Create route table
```

Enter:

```text
Name:
public-route-table

VPC:
nodejs-vpc
```

Click:

```text
Create route table
```

---

## Add the internet route

Open:

```text
public-route-table
```

Go to:

```text
Routes
→ Edit routes
→ Add route
```

Add:

```text
Destination:
0.0.0.0/0

Target:
Internet Gateway

Internet Gateway:
nodejs-igw
```

Save the changes.

---

## What does `0.0.0.0/0` mean here?

In a route table:

```text
0.0.0.0/0
```

means:

```text
Any IPv4 destination outside the local VPC network
```

The route says:

```text
If traffic is going outside the VPC
        │
        ▼
Send it to nodejs-igw
        │
        ▼
Internet
```

---

## Route flow

```text
EC2
 │
 ▼
Public Subnet
 │
 ▼
Public Route Table
 │
 ▼
0.0.0.0/0
 │
 ▼
Internet Gateway
 │
 ▼
Internet
```

---

# STEP 7 — Associate the Public Subnet with the Public Route Table

Creating the route table is not enough.

We must tell AWS which subnet should use it.

Open:

```text
public-route-table
```

Go to:

```text
Subnet associations
→ Edit subnet associations
```

Select:

```text
public-subnet-1
```

Save.

---

## Architecture now

```text
Internet
    │
    ▼
nodejs-igw
    │
    ▼
public-route-table
    │
    ▼
public-subnet-1
    │
    ▼
Future EC2 Backend
```

Now the public subnet has a route to the internet.

---

# STEP 8 — Enable Public IPv4 Assignment

Go to:

```text
VPC
→ Subnets
→ public-subnet-1
```

Then:

```text
Actions
→ Edit subnet settings
```

Enable:

```text
Enable auto-assign public IPv4 address
```

Save.

---

## Why is this useful?

When we launch an EC2 instance inside:

```text
public-subnet-1
```

AWS can assign it a public IPv4 address.

Example:

```text
EC2 Private IP:
10.0.1.25

EC2 Public IP:
54.x.x.x
```

The private IP is used inside the VPC.

The public IP allows communication from the internet.

---

# STEP 9 — Keep the Database Subnets Private

Do not associate these subnets with:

```text
public-route-table
```

Keep:

```text
private-db-subnet-1

private-db-subnet-2
```

without a direct:

```text
0.0.0.0/0
        ↓
Internet Gateway
```

---

## Public vs Private

```text
PUBLIC SUBNET

EC2
 │
 ▼
Route Table
 │
 ▼
Internet Gateway
 │
 ▼
Internet
```

```text
PRIVATE DB SUBNET

RDS
 │
 X
No direct Internet Gateway route
```

This helps keep the database away from direct internet access.

---

# STEP 10 — Create the Backend Security Group

Go to:

```text
EC2
→ Network & Security
→ Security Groups
→ Create security group
```

Even though Security Groups appear in the EC2 console, they are VPC networking resources.

Enter:

```text
Security group name:
backend-sg

Description:
Security group for Node.js backend

VPC:
nodejs-vpc
```

---

## Backend inbound rules

### Rule 1 — SSH

```text
Type:
SSH

Protocol:
TCP

Port:
22

Source:
My IP
```

Purpose:

```text
Your Laptop
    │
    │ SSH Port 22
    ▼
EC2
```

Only your public IP can SSH into the server.

---

### Rule 2 — HTTP

```text
Type:
HTTP

Protocol:
TCP

Port:
80

Source:
Anywhere IPv4
```

Purpose:

```text
Internet Users
      │
      │ HTTP
      ▼
     EC2
```

---

### Rule 3 — HTTPS

```text
Type:
HTTPS

Protocol:
TCP

Port:
443

Source:
Anywhere IPv4
```

Purpose:

```text
Internet Users
      │
      │ HTTPS
      ▼
     EC2
```

---

## Backend Security Group

```text
backend-sg

Inbound:

22
Source: Your IP
Purpose: SSH

80
Source: 0.0.0.0/0
Purpose: HTTP

443
Source: 0.0.0.0/0
Purpose: HTTPS
```

---

## Development testing

Before configuring Nginx, we may temporarily expose Node.js port:

```text
Custom TCP
Port: 3000
Source: My IP
```

Then:

```text
Your Laptop
    │
    │ Port 3000
    ▼
Node.js Backend
```

Do not unnecessarily expose:

```text
3000 → 0.0.0.0/0
```

in a proper production setup.

Later:

```text
Internet
    │
    │ 80 / 443
    ▼
Nginx
    │
    │ localhost:3000
    ▼
Node.js
```

---

# STEP 11 — Create the RDS Security Group

Create another security group.

Go to:

```text
EC2
→ Network & Security
→ Security Groups
→ Create security group
```

Enter:

```text
Security group name:
rds-sg

Description:
Allow PostgreSQL access from Node.js backend

VPC:
nodejs-vpc
```

---

## RDS inbound rule

Add:

```text
Type:
PostgreSQL

Protocol:
TCP

Port:
5432

Source:
backend-sg
```

The source is:

```text
backend-sg
```

not:

```text
0.0.0.0/0
```

and not:

```text
EC2_PUBLIC_IP/32
```

---

## Why use another Security Group as the source?

The rule means:

```text
Any AWS resource with backend-sg
        │
        │ PostgreSQL 5432
        ▼
       RDS
```

Therefore:

```text
EC2 with backend-sg → RDS     ✅

Internet → RDS                ❌

Your Laptop → RDS             ❌

Random EC2 → RDS              ❌
```

---

## Security Group relationship

```text
┌──────────────────────┐
│ EC2                  │
│                      │
│ Security Group:      │
│ backend-sg           │
└──────────┬───────────┘
           │
           │ PostgreSQL
           │ TCP 5432
           ▼
┌──────────────────────┐
│ RDS                  │
│                      │
│ Security Group:      │
│ rds-sg               │
│                      │
│ Allows backend-sg    │
└──────────────────────┘
```

---

# STEP 12 — Create the RDS DB Subnet Group

RDS needs to know which subnets it can use.

Go to:

```text
RDS
→ Subnet groups
→ Create DB subnet group
```

Enter:

```text
Name:
nodejs-db-subnet-group

Description:
Private subnets for PostgreSQL RDS

VPC:
nodejs-vpc
```

Select Availability Zones:

```text
us-west-1a

us-west-1b
```

Select subnets:

```text
private-db-subnet-1
10.0.2.0/24

private-db-subnet-2
10.0.3.0/24
```

Create the subnet group.

---

## What does the DB subnet group do?

It tells RDS:

```text
You are allowed to place the database
inside these database subnets.
```

Architecture:

```text
nodejs-db-subnet-group
│
├── private-db-subnet-1
│   us-west-1a
│
└── private-db-subnet-2
    us-west-1b
```

---

# STEP 13 — Create the Private RDS PostgreSQL Database

Go to:

```text
RDS
→ Databases
→ Create database
```

Choose:

```text
Engine:
PostgreSQL
```

Do not accidentally select:

```text
Aurora PostgreSQL
```

if the goal is a normal RDS PostgreSQL instance.

---

## Database settings

Example:

```text
DB identifier:
nodejs-postgres

Master username:
postgres
```

Choose a strong password.

---

## Connectivity settings

Select:

```text
VPC:
nodejs-vpc
```

Select:

```text
DB subnet group:
nodejs-db-subnet-group
```

Set:

```text
Public access:
No
```

Select:

```text
VPC security group:
rds-sg
```

---

## Result

```text
RDS PostgreSQL

VPC:
nodejs-vpc

Subnets:
private-db-subnet-1
private-db-subnet-2

Public Access:
No

Security Group:
rds-sg
```

---

## Why `Public access: No`?

Because our database should be reached through the private VPC network.

```text
Internet
    │
    X
   RDS
```

But:

```text
EC2 Backend
    │
    │ Private VPC network
    ▼
   RDS
```

---

# STEP 14 — Create the EC2 Node.js Server

Go to:

```text
EC2
→ Instances
→ Launch instance
```

Enter:

```text
Name:
nodejs-backend
```

Choose an operating system:

```text
Ubuntu Server
```

Choose an instance type based on your requirements and budget.

---

## EC2 Network Settings

Select:

```text
VPC:
nodejs-vpc
```

Select:

```text
Subnet:
public-subnet-1
```

Set:

```text
Auto-assign public IP:
Enable
```

Select existing Security Group:

```text
backend-sg
```

Launch the instance.

---

## EC2 architecture

```text
Internet
    │
    ▼
Internet Gateway
    │
    ▼
Public Route Table
    │
    ▼
Public Subnet
    │
    ▼
EC2
```

---

# STEP 15 — Connect EC2 to RDS

After RDS is created, AWS gives an endpoint similar to:

```text
nodejs-postgres.xxxxxxxxx.us-west-1.rds.amazonaws.com
```

The Node.js environment variables can be:

```env
DB_HOST=nodejs-postgres.xxxxxxxxx.us-west-1.rds.amazonaws.com
DB_PORT=5432
DB_NAME=postgres
DB_USER=postgres
DB_PASSWORD=your-password
```

---

## Connection flow

```text
Node.js Backend
      │
      │ DB_HOST
      ▼
RDS Endpoint
      │
      ▼
Private VPC Network
      │
      ▼
RDS Security Group
      │
      │ Is source backend-sg?
      │
      ├── YES → Allow
      │
      └── NO  → Block
```

---

# STEP 16 — Node.js Database Configuration

Example:

```typescript
import "dotenv/config";
import { Pool } from "pg";

const pool = new Pool({
  host: process.env.DB_HOST,
  port: Number(process.env.DB_PORT) || 5432,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,

  ssl: {
    rejectUnauthorized: false,
  },

  max: 5,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 10000,
});

export default pool;
```

The Node.js backend connects to the private RDS endpoint.

---

# STEP 17 — Complete Network Flow

```text
                            CUSTOMER
                                │
                                │ HTTPS
                                ▼
                           INTERNET
                                │
                                ▼
                       INTERNET GATEWAY
                                │
                                ▼
                       PUBLIC ROUTE TABLE
                                │
                                ▼
                         PUBLIC SUBNET
                         10.0.1.0/24
                                │
                                ▼
                      EC2 NODE.JS BACKEND
                         backend-sg
                                │
                                │ PostgreSQL
                                │ TCP 5432
                                │
                                ▼
                       PRIVATE VPC NETWORK
                                │
                                ▼
                     PRIVATE DATABASE SUBNETS
                                │
                                ▼
                         RDS POSTGRESQL
                             rds-sg
```

---

# Security Group Summary

## EC2 Backend Security Group

```text
Name:
backend-sg
```

Inbound:

```text
SSH
Port: 22
Source: My IP
```

```text
HTTP
Port: 80
Source: 0.0.0.0/0
```

```text
HTTPS
Port: 443
Source: 0.0.0.0/0
```

Optional temporary development rule:

```text
Custom TCP
Port: 3000
Source: My IP
```

---

## RDS Security Group

```text
Name:
rds-sg
```

Inbound:

```text
PostgreSQL
Port: 5432
Source: backend-sg
```

---

# Security Flow

```text
Internet
   │
   │ 80 / 443
   ▼
backend-sg
   │
   ▼
EC2
   │
   │ 5432
   ▼
rds-sg
   │
   ▼
RDS
```

The database is never directly exposed to the public internet.

---

# Difference Between VPC, Subnet, Route Table and Security Group

| Component | Purpose |
|---|---|
| VPC | Creates the private AWS network |
| Subnet | Divides the VPC into smaller networks |
| Internet Gateway | Connects the VPC to the internet |
| Route Table | Decides where network traffic should go |
| Security Group | Controls which traffic can access a resource |
| DB Subnet Group | Tells RDS which subnets it can use |

---

# Simple Analogy

```text
VPC
=
Gated Community

Subnet
=
Street inside the community

Route Table
=
Road signs telling traffic where to go

Internet Gateway
=
Main gate connecting the community to the outside world

EC2 / RDS
=
Houses

Security Group
=
Security guard at each house
```

---

# Public vs Private Architecture

## Public EC2

```text
Internet
    │
    ▼
Internet Gateway
    │
    ▼
Public Route Table
    │
    ▼
Public Subnet
    │
    ▼
EC2
```

The EC2 server needs to receive traffic from users.

---

## Private RDS

```text
Internet
    │
    X
   RDS
```

Instead:

```text
EC2
 │
 │ Private VPC Network
 ▼
RDS
```

The database only needs to communicate with the backend.

---

# Why Not Use `0.0.0.0/0` for RDS?

This rule:

```text
PostgreSQL
Port: 5432
Source: 0.0.0.0/0
```

means:

```text
Your Laptop          ✅
Your Backend         ✅
Other Developers     ✅
Random Internet IP   ✅
Internet Scanners    ✅
Attackers             ✅
```

The attacker would still need valid database credentials, but the database port is exposed to the internet.

A better rule is:

```text
PostgreSQL
Port: 5432
Source: backend-sg
```

Then:

```text
Node.js Backend       ✅
Random Internet User  ❌
Developer Laptop      ❌
Other AWS Resources   ❌
```

---

# Local Development Consideration

With:

```text
RDS Public Access:
No
```

your laptop cannot directly connect to RDS through the public internet.

```text
Laptop
   │
   X
Private RDS
```

The EC2 backend can connect:

```text
EC2
 │
 │ Private VPC
 ▼
RDS
```

For controlled developer access later, possible approaches include:

```text
VPN
```

```text
SSH tunnel through a controlled server
```

```text
AWS Systems Manager based access
```

The important point is that production RDS does not need to be publicly exposed just because developers sometimes need database access.

---

# Development vs Staging vs Production

## Local Development

```text
Developer Laptop
      │
      ▼
Local PostgreSQL
```

or:

```text
Developer Laptop
      │
      ▼
Shared Development Database
```

---

## Staging

```text
Developers
     │
     ▼
VPN / Controlled Access
     │
     ▼
Staging Environment
     │
     ▼
Staging RDS
```

Backend:

```text
Staging Backend
      │
      ▼
Staging RDS
```

---

## Production

```text
Customers
    │
    ▼
Load Balancer / Web Server
    │
    ▼
Production Backend
    │
    ▼
Private Production RDS
```

Developers should not casually connect directly to the production database.

---

# Complete Resource List

After finishing the setup, we should have:

```text
VPC
└── nodejs-vpc
    └── 10.0.0.0/16
```

```text
Subnets

public-subnet-1
10.0.1.0/24
us-west-1a

private-db-subnet-1
10.0.2.0/24
us-west-1a

private-db-subnet-2
10.0.3.0/24
us-west-1b
```

```text
Internet Gateway

nodejs-igw
```

```text
Route Table

public-route-table

Route:
0.0.0.0/0 → nodejs-igw

Associated Subnet:
public-subnet-1
```

```text
Security Groups

backend-sg
rds-sg
```

```text
DB Subnet Group

nodejs-db-subnet-group
```

```text
Compute

EC2:
nodejs-backend
```

```text
Database

RDS PostgreSQL:
nodejs-postgres
```

---

# Final Architecture Summary

```text
AWS REGION: us-west-1
│
└── VPC: nodejs-vpc
    │
    │  CIDR: 10.0.0.0/16
    │
    ├── Internet Gateway
    │   └── nodejs-igw
    │
    ├── Public Route Table
    │   │
    │   ├── 10.0.0.0/16 → Local
    │   └── 0.0.0.0/0 → Internet Gateway
    │
    ├── Public Subnet
    │   │
    │   ├── Name: public-subnet-1
    │   ├── CIDR: 10.0.1.0/24
    │   ├── AZ: us-west-1a
    │   │
    │   └── EC2
    │       │
    │       ├── Node.js Backend
    │       └── Security Group: backend-sg
    │
    ├── Private DB Subnet 1
    │   │
    │   ├── CIDR: 10.0.2.0/24
    │   └── AZ: us-west-1a
    │
    ├── Private DB Subnet 2
    │   │
    │   ├── CIDR: 10.0.3.0/24
    │   └── AZ: us-west-1b
    │
    └── RDS PostgreSQL
        │
        ├── Public Access: No
        ├── Security Group: rds-sg
        └── Allowed Source: backend-sg
```

---

# Final Security Model

```text
PUBLIC INTERNET
      │
      │ 80 / 443
      ▼
┌──────────────────┐
│ EC2              │
│ Node.js Backend  │
│ backend-sg       │
└────────┬─────────┘
         │
         │ TCP 5432
         │ Private VPC Network
         ▼
┌──────────────────┐
│ RDS PostgreSQL   │
│ rds-sg           │
│ Public: NO       │
└──────────────────┘
```

The final rule is simple:

```text
Internet can access the application.

The application can access the database.

The internet cannot directly access the database.
```

---

# Learning Outcome

After completing this architecture, we should understand:

```text
✓ What a VPC is

✓ Why AWS resources need a network

✓ How CIDR ranges work at a basic level

✓ What subnets are

✓ Why multiple Availability Zones are used

✓ Difference between public and private subnets

✓ What an Internet Gateway does

✓ What a Route Table does

✓ What 0.0.0.0/0 means in a route

✓ What Security Groups do

✓ How one Security Group can reference another

✓ How EC2 communicates with RDS privately

✓ Why production databases should normally not be publicly exposed

✓ How a Node.js backend connects to private PostgreSQL RDS
```

---

# Recommended Build Order

Follow this exact order:

```text
1. Create VPC
        ↓
2. Create Public Subnet
        ↓
3. Create Private DB Subnet 1
        ↓
4. Create Private DB Subnet 2
        ↓
5. Create Internet Gateway
        ↓
6. Attach Internet Gateway to VPC
        ↓
7. Create Public Route Table
        ↓
8. Add 0.0.0.0/0 → Internet Gateway
        ↓
9. Associate Public Subnet with Public Route Table
        ↓
10. Enable Auto-assign Public IPv4
        ↓
11. Create backend-sg
        ↓
12. Create rds-sg
        ↓
13. Allow rds-sg port 5432 from backend-sg
        ↓
14. Create RDS DB Subnet Group
        ↓
15. Create Private RDS PostgreSQL
        ↓
16. Create EC2 Instance
        ↓
17. Deploy Node.js Backend
        ↓
18. Connect EC2 → Private RDS
        ↓
19. Configure PM2
        ↓
20. Configure Nginx
        ↓
21. Add Domain
        ↓
22. Add HTTPS
```

---

# Next Deployment Architecture

After the AWS networking is complete:

```text
GitHub Repository
       │
       ▼
EC2
       │
       ├── Git
       ├── Node.js
       ├── npm
       ├── PM2
       └── Nginx
              │
              ▼
        Node.js Backend
              │
              ▼
      Private RDS PostgreSQL
```

The next practical stage is:

```text
Create EC2
    ↓
SSH into EC2
    ↓
Install Node.js
    ↓
Clone GitHub repository
    ↓
Configure environment variables
    ↓
Test EC2 → RDS connection
    ↓
Run application with PM2
    ↓
Configure Nginx
```
