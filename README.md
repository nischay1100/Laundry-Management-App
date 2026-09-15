# Laundry Management App

A simple, fast, offline-first Android application for managing daily laundry operations for a local laundry business.

The application supports three roles:

- **Customer**
- **Staff**
- **Admin**

The system is designed for a small laundry business with a single business in V1, while keeping the backend structure ready for future expansion.

---

## 1. Project Overview

The Laundry Management App replaces manual/WhatsApp-based laundry management with one simple Android application.

Customers can:

- Create an account
- Verify their email
- Complete their profile
- Choose Personal or Business customer type
- Place laundry orders
- Select items and services
- View estimated amount
- Track order status
- View order history
- View invoices
- View payment information
- Manage their profile

Staff can:

- View operational orders
- Search and filter orders
- Create walk-in/phone/manual orders
- Manage customers
- Update order status
- Record payments
- Generate invoices
- Work when internet is unavailable

Admin can:

- Manage staff
- Manage customers
- Manage items
- Manage services
- Manage prices
- Configure GST
- Manage business settings
- Manage orders
- Manage payments
- Generate reports
- Generate invoices

---

# 2. Main Goals

The application must be:

1. **Simple**
2. **Fast**
3. **Easy to learn**
4. **Android-first**
5. **Offline-friendly**
6. **Secure**
7. **Role-based**
8. **Suitable for a small local business**
9. **Ready for future web administration**

The application must not become an unnecessarily complex ERP system.

---

# 3. User Roles

## Customer

Customers can register themselves.

```text
Register
    ↓
Email + Password
    ↓
Verify Email
    ↓
Complete Profile
    ↓
Customer Dashboard
```

Customer types:

```text
PERSONAL
BUSINESS
```

Business customers can provide:

- Business Name
- Business Type
- Business Address
- Business Phone

---

## Staff

Staff cannot register publicly.

Admin creates staff accounts.

```text
Admin
 ↓
People
 ↓
Staff
 ↓
Add Staff
 ↓
Staff Account Created
 ↓
Staff Login
 ↓
First Password Change
```

Staff accounts can be:

```text
ACTIVE
INACTIVE
```

---

## Admin

Admin accounts are controlled through a secure setup process.

Admin has full business-management permissions.

There is no normal public Admin registration.

---

# 4. Authentication

Authentication uses Firebase Authentication.

## Registration

```text
Email + Password
        ↓
Firebase Authentication
        ↓
Email Verification
        ↓
Complete Your Profile
        ↓
Customer Dashboard
```

## Login

```text
Email + Password
        ↓
Firebase Authentication
        ↓
Check Email Verification
        ↓
Check Profile Completion
        ↓
Check Role
        ↓
Correct Dashboard
```

There is **no phone OTP** in V1.

Phone number is collected as part of the customer profile but is not technically verified.

---

## Forgot Password

```text
Forgot Password
      ↓
Enter Email
      ↓
Firebase Password Reset Email
      ↓
Secure Reset Link
      ↓
Set New Password
      ↓
Login
```

The application does not implement a custom SMS OTP password-reset system.

---

# 5. Technology Stack

## Mobile Application

- Expo
- React Native
- TypeScript
- Expo Router

## Local Database

- Expo SQLite
- SQLite transactions
- Local sync queue

## Backend

- Firebase Authentication
- Cloud Firestore
- Firebase Cloud Functions
- Firebase Admin SDK

## Notifications

- Firebase Cloud Messaging (FCM)

## Documents

- Local A4 PDF invoice generation
- Local XLSX report generation

## Future Web Application

The backend is designed to support a future:

- React / Next.js
- TypeScript
- Admin Web App

The V1 project does **not** include the web application.

---

# 6. Architecture

```text
                         ┌──────────────────────┐
                         │   Android App        │
                         │ Expo + React Native  │
                         └──────────┬───────────┘
                                    │
                              User Action
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │       SQLite         │
                         │ Local Database       │
                         └──────────┬───────────┘
                                    │
                              Sync Queue
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │      Firebase        │
                         │                      │
                         │ Authentication       │
                         │ Firestore            │
                         │ Cloud Functions      │
                         │ FCM                  │
                         └──────────────────────┘
```

---

# 7. Offline-First Design

Offline support is a core requirement.

The basic flow is:

```text
User Action
    ↓
Validate
    ↓
Write to SQLite
    ↓
Update UI Immediately
    ↓
Add Sync Queue Entry
    ↓
Internet Available?
    │
    ├── No → Keep Local Data
    │
    └── Yes
          ↓
       Firebase
          ↓
       Synced
```

Example:

```text
Customer creates order ORD1052
        ↓
Saved to SQLite
        ↓
Internet unavailable
        ↓
Order remains visible
        ↓
Sync Queue = PENDING
        ↓
Internet returns
        ↓
Firestore synchronization
        ↓
Sync Queue = SYNCED
```

The application must never silently lose a locally-created order.

---

# 8. Local vs Cloud Data

```text
SQLite
├── Local operational data
├── Offline access
├── Search
├── Pending changes
├── Invoice generation
└── Report generation

Firestore
├── Long-term business records
├── Cross-device synchronization
├── Customer/order records
├── Staff records
├── Master data
└── Future web application
```

Important:

> **Saved locally does not necessarily mean synced to the cloud.**

The UI must clearly communicate synchronization status.

---

# 9. Order Lifecycle

Normal order flow:

```text
NEW
 ↓
PICKUP_PENDING
 ↓
PICKED_UP
 ↓
PROCESSING
 ↓
READY
 ↓
OUT_FOR_DELIVERY
 ↓
DELIVERED
```

Cancellation is also supported:

```text
CANCELLED
```

Invalid status jumps should normally be blocked.

Every important status change should create a status-history record.

---

# 10. Orders

Orders can originate from:

```text
CUSTOMER
STAFF
```

Staff can create:

- Walk-in orders
- Phone orders
- Manual orders

Each order stores historical customer information.

Example:

```text
customerNameAtOrder
customerPhoneAtOrder
businessNameAtOrder
pickupAddress
```

This ensures that changing a customer's profile later does not change historical invoices or orders.

---

# 11. Items, Services and Pricing

Laundry pricing is based on:

```text
Item + Service
```

Example:

```text
Shirt + Wash          = ₹30
Shirt + Wash + Iron   = ₹50
Pant + Wash           = ₹40
```

Admin manages:

- Items
- Services
- Prices

Prices are stored in Firestore and cached locally.

---

# 12. Historical Price Protection

When an order is created, the current price is copied into:

```text
priceAtOrderTime
```

Example:

```text
Current price:
Shirt + Wash = ₹30
```

Order stores:

```text
priceAtOrderTime = ₹30
```

If Admin later changes the price to:

```text
₹35
```

the old order remains:

```text
₹30
```

Historical orders must never be recalculated using current prices.

---

# 13. GST

GST is selected **per order**.

The business has a default GST configuration, but it does not force GST on every order.

Order options:

```text
With GST
Without GST
```

The order stores:

```text
gstApplied
gstRate
gstAmount
```

Example:

```text
Subtotal = ₹500
GST = 18%

GST Amount = ₹90
Grand Total = ₹590
```

Without GST:

```text
Subtotal = ₹500
GST = ₹0
Grand Total = ₹500
```

Historical GST values are never recalculated from the current business setting.

---

# 14. Payments

Payment statuses:

```text
PENDING
PARTIALLY_PAID
PAID
```

Payment methods:

```text
CASH
UPI
ONLINE
```

Multiple payments can exist for one order.

Example:

```text
Order Total = ₹1,000

Payment 1 = ₹400 Cash
Payment 2 = ₹300 UPI

Paid = ₹700
Due = ₹300
Status = PARTIALLY_PAID
```

Payment history must remain auditable.

Order status and payment status are independent.

For example:

```text
Order Status: DELIVERED
Payment Status: PENDING
```

is valid.

---

# 15. Invoices

Invoices are generated locally as A4 PDFs.

```text
Order Data
    ↓
SQLite
    ↓
Invoice Generator
    ↓
A4 PDF
    ↓
Local Device
    ↓
Open / Share
```

Invoice includes:

- Business information
- GSTIN if configured
- Invoice number
- Order ID
- Invoice date
- Customer information
- Item
- Service
- Quantity
- Rate
- Amount
- Subtotal
- GST
- Grand Total
- Paid Amount
- Due Amount
- Payment Status
- Payment Method
- Thank-you message

Order ID and Invoice Number are separate identifiers.

Example:

```text
Order ID       = ORD1025
Invoice Number = INV1025
```

---

# 16. Reports

Admin can generate:

### Weekly

- Selected month/year
- From date
- To date

### Monthly

- Selected month
- Selected year

### Yearly

- Selected year

Reports include:

- Total Orders
- Completed Orders
- Cancelled Orders
- Active/Pending Orders
- Subtotal
- GST
- Grand Total
- Paid Amount
- Pending Amount
- Cash
- UPI
- Online
- Orders by Status
- Service Quantity
- Service Revenue
- Item Quantity
- Item Revenue

Yearly reports also include monthly breakdowns.

Reports are generated locally as `.xlsx` files.

---

# 17. Navigation

## Customer

```text
Home
Orders
Profile
```

## Staff

```text
Home
Orders
Profile
```

## Admin

```text
Home
Orders
People
Reports
More
```

Admin People:

```text
People
├── Staff
└── Customers
```

---

# 18. Admin Management

Admin can manage:

```text
Staff
Customers
Items
Services
Prices
GST
Business Settings
Orders
Payments
Reports
Invoices
```

Staff management supports:

```text
Add
View
Edit
Deactivate
Reactivate
```

Staff should be deactivated rather than deleted when historical records reference them.

---

# 19. Security

Security is enforced at the backend level.

The app uses:

- Firebase Authentication
- Firestore Security Rules
- Cloud Functions
- Firebase Admin SDK
- Role-based access
- Business-level isolation

The following must never be trusted solely from the mobile client:

```text
role
businessId
finalAmount
paidAmount
paymentStatus
orderStatus
invoiceNumber
```

Firebase Admin credentials must never be included in the Android application.

Passwords are managed only by Firebase Authentication.

---

# 20. Business Isolation

Every business-owned record contains:

```text
businessId
```

V1 supports one business, but this structure allows future expansion.

The backend must prevent one business from accessing another business's:

- Customers
- Orders
- Staff
- Items
- Services
- Prices
- Reports
- Settings

---

# 21. Main Firestore Structure

```text
users/{userId}

businesses/{businessId}

businesses/{businessId}/staff/{staffId}

businesses/{businessId}/customers/{customerId}

businesses/{businessId}/items/{itemId}

businesses/{businessId}/services/{serviceId}

businesses/{businessId}/prices/{priceId}

businesses/{businessId}/orders/{orderId}

businesses/{businessId}/orders/{orderId}/payments/{paymentId}

businesses/{businessId}/orders/{orderId}/statusHistory/{historyId}

businesses/{businessId}/settings/general

businesses/{businessId}/notificationTokens/{tokenId}
```

---

# 22. Main SQLite Tables

```text
users
business
customers
staff
items
services
prices
orders
order_items
payments
status_history
business_settings
notification_tokens
sync_queue
app_metadata
```

The detailed database contract is defined in:

```text
BACKEND-SCHEMA.md
```

---

# 23. Documentation

The project documentation is divided into four major documents.

```text
PRD.md
   ↓
TRD.md
   ↓
UI-UX.md
   ↓
BACKEND-SCHEMA.md
```

### PRD.md

Defines:

> **What the product must do and why.**

Contains:

- Product goals
- Users
- Features
- Requirements
- Scope
- Business rules
- Acceptance criteria

### TRD.md

Defines:

> **How the system works technically.**

Contains:

- Architecture
- Technology stack
- Offline-first implementation
- Sync
- Security
- Notifications
- Testing
- Deployment

### UI-UX.md

Defines:

> **How users interact with the application.**

Contains:

- Screens
- Navigation
- User flows
- Components
- States
- Validation
- UX rules

### BACKEND-SCHEMA.md

Defines:

> **Exactly how data is stored and related.**

Contains:

- Firestore schema
- SQLite schema
- Fields
- Types
- Relationships
- Indexes
- Security requirements
- Sync metadata
- Historical-data rules

---

# 24. Recommended Project Structure

```text
laundry-management-app/
│
├── app/
│   ├── (auth)/
│   ├── (customer)/
│   ├── (staff)/
│   ├── (admin)/
│   └── _layout.tsx
│
├── src/
│   ├── components/
│   ├── constants/
│   ├── hooks/
│   ├── services/
│   │   ├── auth/
│   │   ├── firebase/
│   │   ├── sqlite/
│   │   ├── sync/
│   │   ├── invoice/
│   │   ├── reports/
│   │   └── notifications/
│   │
│   ├── repositories/
│   ├── models/
│   ├── types/
│   ├── utils/
│   └── validation/
│
├── functions/
│   └── src/
│
├── assets/
│
├── docs/
│
├── PRD.md
├── TRD.md
├── UI-UX.md
├── BACKEND-SCHEMA.md
├── README.md
├── package.json
└── app.json
```

The exact folder structure may be adjusted during implementation if doing so improves maintainability without violating the architecture defined in the project documents.

---

# 25. Development Principles

## Keep It Simple

Do not introduce unnecessary enterprise architecture.

## Local First

Important user actions should work without internet whenever possible.

## No Data Loss

Offline data must remain until successful synchronization.

## Secure by Default

Never trust client-controlled role or business information.

## Historical Integrity

Old orders, invoices, prices, GST and payments must remain historically correct.

## Role Separation

Customer, Staff and Admin must have clearly separated permissions.

## Reusable Components

Common UI and business logic should be reusable.

## Centralized Constants

Do not scatter:

- status values
- roles
- payment methods
- strings
- route names

throughout the application.

---

# 26. Error Handling

Do not expose raw technical/Firebase errors.

Use clear messages such as:

```text
You do not have permission to perform this action.
```

```text
Internet connection is unavailable. Your data has been saved on this device and will sync automatically.
```

```text
Some data is waiting to sync. Please keep the app open when internet is available.
```

Errors should be:

- understandable
- actionable
- short
- user-friendly

---

# 27. V1 Scope

V1 includes:

```text
Customer Registration
Email Verification
Login
Password Reset

Customer Profile
Personal/Business Customer Type

Customer Orders
Staff Orders
Order Search
Order Filters
Order Status Tracking

Items
Services
Pricing

GST Per Order

Payments
Payment History

A4 PDF Invoices

Weekly Reports
Monthly Reports
Yearly Reports

Offline SQLite Storage
Sync Queue
Cloud Synchronization

Role-Based Access

Staff Management
Customer Management

FCM Notifications
```

---

# 28. V1 Exclusions

The following are intentionally NOT part of V1:

```text
Super Admin
Multi-business SaaS UI
Multi-branch support
Multiple processing centres
Separate delivery app
Manager role
Inventory management
Salary/payroll
Expense management
Advanced accounting
GPS route optimization
Garment QR/barcode tracking
POS hardware integration
Printer integration
Loyalty system
Coupons
Corporate account system
AI features
```

Do not add these features simply because they could be useful later.

---

# 29. Future Scope

Possible future features:

```text
Online Payment Gateway
WhatsApp Notifications
SMS Notifications
Business Logo Upload
Order/Garment Photos
Expense Management
Advanced Analytics
Multi-business Support
Multi-branch Support
QR/Barcode Tracking
Loyalty Program
Coupons
Delivery Route Tracking
Multiple Business Associations per Customer
```

Future features must not unnecessarily complicate V1.

---

# 30. Testing Requirements

Before release, test at minimum:

### Authentication

```text
Register
Email Verification
Login
Wrong Password
Unverified Email
Forgot Password
Inactive User
Role Routing
```

### Customer

```text
Profile Completion
Personal Customer
Business Customer
Business → Personal
Place Order
Order History
Order Detail
```

### Staff

```text
Staff Login
First Password Change
Manual Order
Customer Search
Order Status Updates
Payment Recording
Invoice
```

### Admin

```text
Staff Creation
Deactivate Staff
Reactivate Staff
Customer Management
Item Management
Service Management
Price Management
GST Settings
Reports
Business Settings
```

### Offline

```text
Create order offline
Update order offline
Record payment offline
Restart app while offline
Reconnect internet
Sync pending changes
Retry failed sync
Prevent duplicate sync
```

### Security

```text
Customer cannot access another customer
Customer cannot access Staff/Admin
Staff cannot manage Staff
Staff cannot change prices
Staff cannot change GST settings
Customer cannot change final amount
Customer cannot change payment history
Business isolation works
```

---

# 31. Definition of Done

The application is ready for V1 release only when:

```text
[ ] Authentication works
[ ] Email verification works
[ ] Password reset works
[ ] Role routing works
[ ] Customer profile works
[ ] Personal/Business customer works
[ ] Customer order creation works
[ ] Staff manual order works
[ ] Order lifecycle works
[ ] Order search/filter works
[ ] Items work
[ ] Services work
[ ] Pricing works
[ ] Historical price snapshots work
[ ] GST-per-order works
[ ] Payment history works
[ ] Historical payment data is protected
[ ] Invoice generation works
[ ] Weekly report works
[ ] Monthly report works
[ ] Yearly report works
[ ] SQLite works
[ ] Offline order creation works
[ ] Sync queue works
[ ] Retry works
[ ] Idempotency works
[ ] Conflict handling works
[ ] Firebase Security Rules work
[ ] Staff privileged creation works
[ ] FCM notifications work if enabled
[ ] No raw Firebase errors are shown
[ ] No historical order data is accidentally changed
[ ] No unauthorized role access is possible
[ ] No duplicate orders are created by sync retry
```

---

# 32. Final Product Flow

## Customer

```text
Install App
   ↓
Register
   ↓
Verify Email
   ↓
Complete Profile
   ↓
Customer Dashboard
   ↓
Place Order
   ↓
Items
   ↓
Services
   ↓
Quantity
   ↓
Pickup Details
   ↓
GST Selection
   ↓
Estimated Amount
   ↓
Confirm
   ↓
NEW
   ↓
PICKUP
   ↓
PROCESSING
   ↓
READY
   ↓
OUT FOR DELIVERY
   ↓
DELIVERED
```

## Staff

```text
Login
   ↓
Dashboard
   ↓
Orders / Customers
   ↓
Create or Manage Order
   ↓
Update Status
   ↓
Record Payment
   ↓
Generate Invoice
```

## Admin

```text
Login
   ↓
Dashboard
   ↓
Orders
People
Reports
More
   ↓
Manage Complete Business Operations
```

---

# 33. Core Project Rule

The project should always prioritize:

```text
Simple
      +
Fast
      +
Reliable
      +
Offline-Friendly
      +
Secure
      +
Historically Accurate
```

over unnecessary features or technical complexity.

---

# 34. Final Statement

This project is intentionally designed as a **small-business laundry management system**, not a general-purpose ERP.

The V1 implementation should remain focused on the real operational needs of:

- Customers
- Laundry Staff
- Business Admin

The four project documents together form the implementation contract:

```text
PRD.md
"What are we building?"

TRD.md
"How will it work technically?"

UI-UX.md
"How will users use it?"

BACKEND-SCHEMA.md
"How will the data be stored and protected?"
```

Any major implementation change that affects product behavior, architecture, UI flow, or database structure should update the relevant documentation before the change is considered final.