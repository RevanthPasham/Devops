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

Contains only the logged-in user's own data.

```
My Activities
│
├── Shipments
│
└── Trips
```

---

# Shipments

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

Matched

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

# Add Shipment

The shipment posting process is intentionally simple.

The platform is a marketplace that connects senders with travellers.

Users only need to provide the minimum required information.

```
Add Shipment

│

├── Shipment Name *

├── Pickup Location *

├── Destination *

├── Parcel Type *

├── Description (Optional)

└── Parcel Image (Optional)
```

Example

```
Shipment Name
Laptop Charger

Pickup
Macherla

Destination
Hyderabad

Parcel Type
Electronics

Description
Small laptop charger with adapter.

Photo
(Optional)
```

Once submitted

```
Post Shipment

↓

Visible in Discover

↓

Traveller Finds Shipment

↓

Traveller Sends Request

↓

Sender Accepts Request

↓

Chat Opens

↓

Pickup

↓

Delivery

↓

Completed

↓

Rate Each Other
```

---

## Shipment Details

Displays

```
Shipment Name

Pickup Location

Destination

Parcel Type

Description

Parcel Image

Current Status

Posted Date
```

---

## Important Notes

The following information is **NOT collected while posting a shipment**.

- Reward Amount
- Recipient Name
- Recipient Phone Number
- Parcel Dimensions
- Parcel Weight
- Insurance
- Delivery Instructions

These details can be discussed after a traveller accepts the shipment and the private chat is opened.

This keeps the posting process fast and encourages more users to create shipments.

---

# Trips

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

# Add Trip

Before posting a trip, the user must complete identity verification.

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

Notes (Optional)

↓

Post Trip
```

Transportation Types

- Airplane
- Car
- Bus
- Train
- Bike
- Truck

---

# Authentication Requirement

Users can browse the application after registering with their phone number.

However, the following actions require identity verification:

- Post Trip
- Post Shipment
- Send Request
- Accept Request
- Chat

Verification includes:

- Phone Number (OTP)
- Government ID
- Selfie
- Address Proof

Only administrators can access uploaded documents.

Public profiles display only:

```
✓ Verified

or

Pending Verification
```

---

# Home Address Policy

During registration, every user must provide their permanent home address.

```
Country

↓

State

↓

District

↓

City / Village

↓

House Address
```

After the account is verified, the home address becomes locked.

Users cannot edit it themselves.

If a user needs to update their address:

```
Request Address Change

↓

Provide Reason

↓

Admin Review

↓

Approved

↓

Address Updated
```

All address changes are recorded for administrative purposes.

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
