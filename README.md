# Parcel Platform

## Overview

Parcel Platform is a community-based parcel sharing marketplace.

The platform does not provide courier services.

It connects verified travellers with senders who need to transport parcels.

One user can act as both:

- Sender
- Traveller

There are NO separate Sender and Traveller accounts.

A user becomes a Traveller when posting a trip.

A user becomes a Sender when posting a shipment.

---

# Application Flow

```
Splash
    │
    ▼
Onboarding
    │
    ▼
Login
    │
    ▼
OTP Verification
    │
    ▼
Complete Profile
    │
    ▼
Identity Verification
    │
    ▼
Home
```

Users must complete authentication before they can:

- Post Trip
- Post Shipment
- Send Requests
- Accept Requests
- Chat

---

# Bottom Navigation

```
────────────────────────────

🏠 Home

🔍 Discover

📋 My Activities

👤 Account

────────────────────────────
```

---

# Home

Purpose

Quick access to the most common actions.

```
Home
│
├── Header
│     ├── Logo
│     ├── Wallet Balance
│     └── Notifications
│
├── Quick Actions
│     ├── Add Trip
│     └── Add Shipment
│
├── Suggested Trips
│     ├── Trip Card
│     ├── Trip Card
│     └── Trip Card
│
└── Suggested Shipments
      ├── Shipment Card
      └── Shipment Card
```

Clicking a Trip Card opens

```
Trip Details

Traveller Name

Traveller Rating

Verification Badge

Route

Travel Date

Arrival Date

Transportation

Available Weight

Parcel Types

Send Request
```

---

# Discover

Purpose

Search the marketplace.

```
Discover
│
├── Shipments
│
└── Trips
```

---

## Discover → Trips

```
Trips
│
├── Search
├── Sort
├── Filter
│
└── Trip List
      │
      ├── Trip Card
      ├── Trip Card
      └── Trip Card
```

Each Trip Card shows

- Route
- Travel Date
- Arrival Date
- Traveller
- Rating
- Available Weight

Click

↓

Trip Details

↓

Send Request

---

## Discover → Shipments

```
Shipments
│
├── Search
├── Sort
├── Filter
│
└── Shipment List
```

Shipment Card contains

- Parcel Photo
- Shipment Name
- Route
- Reward
- Weight
- Sender
- Rating

Click

↓

Shipment Details

↓

Accept Shipment

---

# My Activities

Contains only the logged-in user's data.

```
My Activities
│
├── Shipments
│
└── Trips
```

---

## Shipments

```
Shipments

↓

+ Add Shipment

↓

My Shipment List
```

Shipment Status

```
Draft

↓

Published

↓

Requested

↓

Accepted

↓

Picked Up

↓

Delivered

↓

Completed
```

Click Shipment

↓

Shipment Details

---

## Add Shipment

Three-step process

---

### Step 1

Shipment Details

```
Shipment Name

Total Weight

Shipment Note

Items

↓

Item Name

Parcel / Document

Length

Width

Height

Item Note

Upload Images
```

Users can add multiple items.

---

### Step 2

Shipment Information

```
Pickup Location

Destination

Maximum Delivery Date

Recipient Name

Recipient Mobile Number

Reward Amount

Insurance (Optional)
```

---

### Step 3

Shipment Summary

Shows

```
Route

Recipient

Items

Reward

Service Fee

Total

Post Shipment
```

---

## Trips

```
Trips

↓

+ Add Trip

↓

Trip List
```

Trip Status

```
Draft

↓

Published

↓

Received Requests

↓

Accepted

↓

Completed
```

Click Trip

↓

Trip Details

---

## Add Trip

```
Trip Information

↓

From

↓

To

↓

Travel Date

↓

Arrival Date

↓

Transportation Type

↓

Available Weight

↓

Can Carry

↓

Note

↓

Post Trip
```

Transportation Types

- Airplane
- Car
- Train
- Bus
- Truck
- Bike

---

# Account

```
Account

│

├── Profile

├── Verification

├── Statistics

├── My Addresses

├── Payment Methods

├── Ratings & Reviews

├── Gift Cards

├── Referrals

├── Help Center

├── Settings

├── Logout

└── Delete Account
```

---

## Profile

Shows

```
Profile Picture

Name

Home Hub

Verification Status

Profile Completion
```

Statistics

```
Total Trips

Total Shipments

Pending Income

Total Income
```

---

## Identity Verification

Users upload

- Selfie
- Government ID
- Address Proof

These files are NEVER visible to other users.

Only Admin can review them.

Public Profile shows only

```
✓ Verified

or

Pending Verification
```

---

# Help Center

```
Help Center

│

├── Frequently Asked Questions

├── Raise Support Ticket

├── Contact Support

├── Terms & Conditions

├── Privacy Policy

└── Prohibited Items
```

---

# Request Flow

```
Sender

↓

Posts Shipment

↓

Traveller Searches

↓

Traveller Sends Request

↓

Sender Accepts

↓

Chat Opens

↓

Pickup

↓

Delivery

↓

Completed

↓

Both Users Rate Each Other
```

---

# Trip Flow

```
Traveller

↓

Posts Trip

↓

Shipment Matches

↓

Receives Requests

↓

Accepts Sender

↓

Chat

↓

Pickup

↓

Delivery

↓

Completed
```

---

# User Permissions

Guest

- Browse only

Authenticated User

- Add Trip
- Add Shipment
- Send Requests
- Accept Requests
- Chat

Verified User

- Higher trust score
- Identity badge

Admin

- View uploaded documents
- Approve verification
- Suspend users
- Manage reports
