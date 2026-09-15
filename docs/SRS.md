# Laundry Management App
## Complete Software Requirements Specification (SRS) — v3.0

**Platform:** Android Mobile App  
**Future Platform:** Admin Web Application  
**Architecture:** Offline-first  
**Backend:** Firebase  
**Local Database:** SQLite  
**Mobile Framework:** Expo + React Native + TypeScript  
**Authentication:** Firebase Authentication  
**Cloud Database:** Cloud Firestore  
**Notifications:** Firebase Cloud Messaging  
**PDF:** Local A4 PDF generation  
**Excel:** Local `.xlsx` generation  
**Target Business:** Single local laundry business  
**Primary Users:** Admin, Staff, Customer

---

# 1. Project Overview

The Laundry Management App is a simple Android-first application for managing the complete daily operation of a local laundry business.

The system is intentionally **not an ERP** and should not contain unnecessary accounting, inventory, payroll, CRM, multi-branch, or SaaS features.

The application will allow:

- Customers to create accounts.
- Customers to maintain personal/business profiles.
- Customers to place laundry orders.
- Customers to see order status and history.
- Staff to manage day-to-day laundry operations.
- Staff to create walk-in/phone/manual orders.
- Admin to control staff, customers, items, services, pricing, orders, payments, GST settings, and reports.
- Admin to generate invoices and reports.
- The application to continue working when internet connectivity is weak or temporarily unavailable.
- Data to synchronize automatically with Firebase when connectivity returns.
- A future Admin Web Application to use the same Firebase backend.

---

# 2. Core Design Principles

The application must follow these principles:

1. **Simple**
2. **Fast**
3. **Mobile-first**
4. **Offline-first**
5. **Secure**
6. **Easy for non-technical staff**
7. **Indian English**
8. **Indian currency and date formats**
9. **No unnecessary features**
10. **Historical data must remain accurate**
11. **Cloud data must remain backed up**
12. **Local operation must not depend completely on internet connectivity**

---

# 3. Technology Stack

## 3.1 Android Application

- Expo
- React Native
- TypeScript
- Expo Router
- Expo SQLite
- Firebase Authentication
- Cloud Firestore
- Firebase Cloud Messaging
- Firebase Cloud Functions where privileged backend operations are required

The project should use the **current stable Expo SDK supported at implementation time** and locked dependency versions.

---

# 4. Architecture

The system uses an offline-first architecture.

```text
                 ANDROID APP
                      │
        ┌─────────────┴─────────────┐
        │                           │
     SQLite                    Firebase
   Local Database          Authentication
        │                   Firestore
        │                   Cloud Functions
        │                   FCM
        │
   Sync Queue
        │
        └──────────────→ Internet
                            │
                            ↓
                       Firebase Cloud
```

Future:

```text
Android App ──────┐
                  ├── Firebase Backend
Admin Web ────────┘
```

Both applications will use the same backend.

---

# 5. User Roles

There are exactly three roles:

```text
CUSTOMER
STAFF
ADMIN
```

---

# 6. Role Responsibilities

## 6.1 CUSTOMER

Customer can:

- Register
- Verify email
- Complete profile
- Add mobile number
- Select Personal/Business
- Add business information
- Place order
- See estimated amount
- See order status
- See order history
- See invoice
- See payment information
- Update own profile
- Change password
- Reset forgotten password

Customer cannot:

- Access Staff panel
- Access Admin panel
- Create staff
- Create admin
- Change own role
- Change businessId
- Change pricing
- Change GST configuration
- View another customer's data
- Change final order amount
- Modify protected payment records

---

# 6.2 STAFF

Staff can:

- Login
- View operational dashboard
- View permitted orders
- Search orders
- Filter orders
- Create manual/walk-in orders
- Create phone orders
- Select existing customers
- Create customer records when permitted
- Add items
- Select services
- View prices
- Update order status
- Record payments when permitted
- View customer details required for operations
- Generate/view invoice when permitted
- Work offline
- Synchronize pending data

Staff cannot:

- Register themselves
- Create Admin
- Create another Staff account
- Change role
- Change businessId
- Manage Staff
- Change pricing
- Change GST configuration
- Delete historical orders
- Delete financial history
- Access another business
- Modify security permissions

---

# 6.3 ADMIN

Admin has full business control.

Admin can:

- Manage staff
- Manage customers
- Manage items
- Manage services
- Manage prices
- Manage orders
- Manage payments
- Manage business settings
- Configure GST details
- Generate reports
- Generate invoices
- View complete customer history
- Activate/deactivate staff
- Activate/deactivate customers where applicable

Admin cannot be created through public registration.

---

# 7. Customer Registration

## 7.1 Registration Design

Customer registration will use:

```text
Email + Password
```

The customer will NOT require phone OTP.

The customer will NOT receive OTP on every login.

---

# 7.2 Registration Flow

```text
Customer opens app
        ↓
Register
        ↓
Enter Email
Enter Password
        ↓
Create Firebase Authentication Account
        ↓
Email Verification
        ↓
Email verified?
   ├── NO → Remain in verification screen
   └── YES
          ↓
Complete Your Profile
          ↓
Name
Mobile Number
Customer Type
Address
Business details if applicable
          ↓
Profile complete
          ↓
Customer Dashboard
```

---

# 7.3 Registration Screen

Required fields:

- Email
- Password
- Confirm Password

Validation:

### Email

- Must be valid email format.
- Cannot be empty.
- Firebase must reject duplicate account email.

### Password

Password must satisfy the application's configured minimum security requirement.

Recommended:

- Minimum 8 characters.

Confirm Password must match Password.

---

# 8. Email Verification

After registration:

```text
Account created
       ↓
Verification email sent
       ↓
Customer opens email
       ↓
Clicks verification link
       ↓
Email verified
       ↓
Return to app
       ↓
Complete Profile
```

The system must check Firebase Authentication's email verification state.

The application must not create a fully usable customer profile until email verification is completed.

---

# 9. Important Phone Verification Rule

Email verification does **not technically verify the phone number**.

Therefore:

```text
emailVerified = true
phoneVerified = not required
```

The application will simply **not use phone OTP in V1**.

The customer's mobile number is treated as contact information, not as a cryptographically verified identity.

This keeps the authentication flow simple and avoids unnecessary OTP cost.

---

# 10. Complete Your Profile

After successful email verification:

```text
Complete Your Profile
```

The customer enters:

- Full Name
- Mobile Number
- Customer Type
- Address
- PIN Code
- Optional email display
- Business information if Customer Type = Business

---

# 11. Customer Type

Customer profile must contain:

```text
customerType:
    PERSONAL
    BUSINESS
```

Default:

```text
PERSONAL
```

---

# 12. Personal Customer

Example:

```text
Name: Rahul Sharma
Mobile: 98XXXXXXXX
Customer Type: Personal
Business: None
```

Database:

```text
customerType = PERSONAL
businessName = null
businessType = null
```

---

# 13. Business Customer

A customer may use the laundry service on behalf of a business.

Examples:

- Hotel
- Homestay
- Restaurant
- Office
- Hostel
- Salon
- Shop
- Other business

Example:

```text
Name: Rahul Sharma
Mobile: 98XXXXXXXX

Customer Type: Business
Business Name: Hotel Paradise
Business Type: Hotel
```

When this customer places an order, Staff/Admin can identify:

```text
Customer:
Rahul Sharma

Business:
Hotel Paradise
```

This is important because the same person may send laundry from different businesses.

---

# 14. Business Profile Fields

If:

```text
customerType = BUSINESS
```

show:

- Business Name — Required
- Business Type — Required/optional according to UI decision
- Business Address — Optional
- Business Phone — Optional

If:

```text
customerType = PERSONAL
```

business fields should be hidden.

The database should store:

```text
businessName = null
businessType = null
businessAddress = null
```

for Personal customers.

---

# 15. Login

Login requires:

```text
Email + Password
```

No OTP is required during normal login.

Flow:

```text
Open App
   ↓
Login
   ↓
Email
Password
   ↓
Firebase Authentication
   ↓
Authentication successful
   ↓
Check email verification
   ↓
Check user profile
   ↓
Check role
   ↓
Open correct dashboard
```

---

# 16. Login Role Routing

```text
Login
  ↓
Firebase Authentication
  ↓
User UID
  ↓
Secure user profile
  ↓
Role
 ├── CUSTOMER → Customer Panel
 ├── STAFF    → Staff Panel
 └── ADMIN    → Admin Panel
```

The role must be enforced by backend security rules.

Hiding screens in the UI is NOT sufficient security.

---

# 17. Incomplete Profile During Login

If authentication succeeds but profile is incomplete:

```text
Login
 ↓
Profile incomplete
 ↓
Complete Your Profile
 ↓
Save
 ↓
Dashboard
```

The customer must not be trapped in a broken state if profile completion was interrupted.

The application should store completion state.

Example:

```text
profileCompleted = false
```

After completion:

```text
profileCompleted = true
```

---

# 18. Forgot Password

V1 should NOT build a custom OTP system.

Recommended Firebase password reset flow:

```text
Forgot Password
      ↓
Enter Email
      ↓
Password Reset Email
      ↓
Open secure reset link
      ↓
Set New Password
      ↓
Login
```

This avoids building and maintaining a custom OTP infrastructure.

The exact availability/cost depends on the selected Firebase/email configuration and applicable service limits.

---

# 19. Staff Account Creation

Staff must NOT have public registration.

Only Admin can create staff.

Flow:

```text
Admin
 ↓
People
 ↓
Staff
 ↓
Add Staff
 ↓
Name
Mobile
Staff ID/Login identifier
Temporary Password
 ↓
Secure Backend Function
 ↓
Firebase Authentication account created
 ↓
Staff profile created
 ↓
Staff receives credentials
 ↓
First Login
 ↓
Force Password Change
 ↓
Staff Dashboard
```

Firebase Admin SDK credentials must never be shipped inside the Android application.

---

# 20. Staff Status

Staff has:

```text
ACTIVE
INACTIVE
```

Admin can:

- Activate
- Deactivate
- Reactivate

Deactivation should prevent login/access without destroying historical records.

Permanent deletion should NOT be the default.

Reason:

Historical orders may contain:

```text
createdBy
updatedBy
recordedBy
changedBy
```

---

# 21. Admin Account

Admin does not use public registration.

Initial Admin should be created through a controlled setup process.

There should be no normal UI allowing a Staff or Customer to promote themselves to Admin.

---

# 22. Customer Navigation

Bottom navigation:

```text
Home
Orders
Profile
```

---

# 23. Staff Navigation

Bottom navigation:

```text
Home
Orders
Profile
```

---

# 24. Admin Navigation

Bottom navigation:

```text
Home
Orders
People
Reports
More
```

---

# 25. Admin People Section

```text
People
├── Staff
└── Customers
```

---

# 26. Staff Management

Admin Staff screen:

```text
Staff
├── Active Staff
├── Inactive Staff
└── Add Staff
```

Each staff member can be opened.

Details:

- Name
- Mobile
- Staff ID
- Status
- Created Date
- Last Updated
- Last Login if available

Actions:

- Edit
- Deactivate
- Reactivate

---

# 27. Customer Management

Admin can see:

```text
Customers
```

Features:

- Search
- Customer list
- Customer profile
- Order history
- Payment summary
- Business information

Search by:

- Name
- Mobile
- Email
- Business Name where applicable

---

# 28. Customer Profile

Customer profile should contain:

```text
Name
Email
Mobile Number

Customer Type:
Personal / Business

Business Name
Business Type

Address
PIN Code

Registration Date
Account Status
```

For business customers:

```text
Business:
Hotel Paradise
```

For personal customers:

```text
Business:
Not applicable
```

---

# 29. Customer Order History

Admin opening a customer should see only that customer's orders.

Example:

```text
Customer:
Rahul Sharma

Business:
Hotel Paradise

Orders:
ORD1021
ORD1034
ORD1052
```

Summary:

- Total Orders
- Completed Orders
- Cancelled Orders
- Total Billed
- Total Paid
- Pending Amount

---

# 30. Historical Customer Snapshot

An important historical-data rule:

Changing a customer's profile must NOT alter old invoices/orders.

When an order is created, save snapshots such as:

```text
customerNameAtOrder
customerPhoneAtOrder
pickupAddressAtOrder
businessNameAtOrder
```

Therefore, if Rahul later changes:

```text
Hotel Paradise
```

to:

```text
Hotel Royal
```

old invoices still show:

```text
Hotel Paradise
```

where appropriate.

---

# 31. Items

Items represent garments/articles.

Examples:

- Shirt
- Pant
- Jeans
- T-Shirt
- Saree
- Bedsheet
- Blanket

Admin can:

- Add item
- Edit item
- Disable item

Existing items should preferably be disabled rather than deleted if historical orders reference them.

---

# 32. Services

Services represent work performed.

Examples:

- Wash
- Wash + Iron
- Iron
- Dry Clean

Admin can:

- Add service
- Edit service
- Disable service

---

# 33. Pricing

Price is based on:

```text
Item + Service
```

Example:

```text
Shirt + Wash = ₹30
Shirt + Wash + Iron = ₹50
Pant + Wash = ₹40
```

Prices must be stored in Firebase and cached in SQLite.

Prices must NOT be hardcoded into application logic.

---

# 34. Historical Price Rule

When an order is created, save:

```text
priceAtOrderTime
```

Example:

Today:

```text
Shirt + Wash = ₹30
```

Order created:

```text
priceAtOrderTime = ₹30
```

Later Admin changes price:

```text
Shirt + Wash = ₹40
```

Old order must still remain:

```text
₹30
```

Historical orders must never be recalculated using current pricing.

---

# 35. Order Sources

Every order has:

```text
source:
CUSTOMER
STAFF
```

Customer:

```text
CUSTOMER
```

Walk-in/phone/manual order:

```text
STAFF
```

---

# 36. Customer Order Flow

```text
Customer Login
      ↓
Home
      ↓
Place Order
      ↓
Select Items
      ↓
Select Service
      ↓
Enter Quantity
      ↓
Select Pickup Details
      ↓
Select GST:
  With GST / Without GST
      ↓
System calculates estimated amount
      ↓
Review Order
      ↓
Confirm
      ↓
Order saved locally
      ↓
Sync with Firebase
      ↓
Order Status = NEW
```

---

# 37. Manual Staff Order

Staff can create:

- Walk-in order
- Phone order
- Manual order

Flow:

```text
Staff
 ↓
Orders
 ↓
Create Order
 ↓
Select Existing Customer
       OR
Create Customer
 ↓
Select Items
 ↓
Select Services
 ↓
Quantity
 ↓
Pickup / Delivery details
 ↓
GST selection
 ↓
Payment information
 ↓
Review
 ↓
Create Order
```

---

# 38. Order Search

Admin and Staff Orders screen must support search by:

- Order ID
- Invoice Number
- Customer Name
- Mobile Number

Example:

```text
Search: ORD1052
Search: INV1052
Search: Rahul
Search: 9876543210
```

---

# 39. Order Filters

Orders should support:

### Operational Status

- New
- Pickup
- Processing
- Delivered
- Cancelled

Detailed internal statuses:

```text
NEW
PICKUP_PENDING
PICKED_UP
PROCESSING
READY
OUT_FOR_DELIVERY
DELIVERED
CANCELLED
```

### Payment Status

- Paid
- Partially Paid
- Unpaid

Filters should be combinable.

Example:

```text
Delivered + Unpaid
```

This means:

```text
orderStatus = DELIVERED
AND
paymentStatus != PAID
```

---

# 40. Order Status Flow

Normal flow:

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

Cancellation:

```text
Any allowed pre-final stage
        ↓
CANCELLED
```

Invalid status jumps should normally be blocked.

---

# 41. Status History

Every status change should create a history record.

Example:

```text
NEW
14 Sep 2026 10:30
Changed by Staff A

PICKED_UP
14 Sep 2026 12:20
Changed by Staff B

PROCESSING
14 Sep 2026 15:00
Changed by Staff A
```

Fields:

```text
historyId
orderId
status
changedAt
changedBy
```

Historical status records should not be silently overwritten.

---

# 42. Order Data

Every order should contain at minimum:

```text
orderId
businessId
customerId
source

customerNameAtOrder
customerPhoneAtOrder
businessNameAtOrder
pickupAddress

items[]

estimatedAmount
finalAmount

gstApplied
gstRate
gstAmount

paidAmount
dueAmount
paymentStatus
paymentMethod

orderStatus

pickupDate
pickupTime
customerNote

invoiceNumber

createdAt
updatedAt
createdBy
lastUpdatedBy
```

---

# 43. Order Items

Each order item:

```text
itemId
itemName
serviceId
serviceName
quantity
priceAtOrderTime
subtotal
```

Example:

```text
Shirt
Wash + Iron
Quantity: 3
Rate: ₹50
Subtotal: ₹150
```

---

# 44. Estimated Amount

Customer may initially see an estimated amount.

Example:

```text
Estimated Amount: ₹500
```

Staff/Admin may verify the actual items and quantities.

Final amount can therefore be different.

Fields:

```text
estimatedAmount
finalAmount
```

---

# 45. GST

GST must NOT be globally enabled/disabled for the entire application.

GST is selected **per order**.

Order creation screen:

```text
GST
○ With GST
○ Without GST
```

---

# 46. GST Business Settings

Admin can configure:

```text
GSTIN
Default GST Rate
```

Example:

```text
GSTIN: 09XXXXXXXXXXXXXX
Default GST Rate: 18%
```

The default rate is only a default.

It does NOT force GST on every order.

---

# 47. GST Calculation

With GST:

```text
Subtotal = ₹500
GST Rate = 18%
GST = ₹90
Grand Total = ₹590
```

Without GST:

```text
Subtotal = ₹500
GST = ₹0
Grand Total = ₹500
```

Order stores:

```text
gstApplied
gstRate
gstAmount
```

Historical GST must never change because Admin changes the default GST rate later.

Reports must use stored order GST values.

---

# 48. Payment

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

Online payment gateway is optional/future functionality.

---

# 49. Payment Record

Each payment should contain:

```text
paymentId
orderId
amount
paymentMethod
paymentDate
recordedBy
note
```

Multiple payment records may exist for one order.

Example:

```text
Total = ₹590

Cash = ₹200
UPI = ₹200

Paid = ₹400
Due = ₹190
```

---

# 50. Payment and Order Status Independence

Order status and payment status are separate.

Example:

```text
Order Status: DELIVERED
Payment Status: PENDING
```

This is valid.

Therefore, payment cannot be inferred only from delivery status.

---

# 51. Invoice Number

Invoice number must be separate from Order ID.

Example:

```text
Order ID:
ORD1025

Invoice:
INV1025
```

Business settings can contain:

```text
invoicePrefix
```

---

# 52. Invoice Generation

Invoices are generated locally on the Android device.

Flow:

```text
Order
 ↓
SQLite
 ↓
Invoice Generator
 ↓
A4 PDF
 ↓
Save locally
 ↓
Open / Share
```

Firebase Storage is NOT required for V1 invoice generation.

---

# 53. Invoice Contents

A4 invoice should contain:

### Business

- Laundry business name
- Address
- Phone
- Email if configured
- GSTIN if applicable

### Invoice

- Invoice number
- Order ID
- Invoice date

### Customer

- Customer name
- Mobile
- Address
- Business name if applicable

### Items

- Item
- Service
- Quantity
- Rate
- Amount

### Totals

- Subtotal
- GST rate
- GST amount
- Grand total
- Paid amount
- Due amount
- Payment status
- Payment method

### Footer

- Thank-you note

---

# 54. Invoice Formatting

Invoice must support:

- ₹ symbol
- Indian number formatting
- Long customer names
- Long addresses
- Long business names
- Multiple items
- Text wrapping
- Correct page layout
- No clipped text
- No overlapping text
- No broken characters

Example date:

```text
14 September 2026
```

or:

```text
14 Sep 2026
```

---

# 55. Customer Order Tracking

Customer can see:

```text
Order ID
Invoice Number
Order Date
Items
Estimated Amount
Final Amount
Payment Status
Order Status
```

Status timeline:

```text
✓ Order Placed
✓ Picked Up
✓ Processing
○ Ready
○ Out for Delivery
○ Delivered
```

---

# 56. Customer Notifications

FCM may be used for:

- Order accepted/new order
- Pickup
- Processing
- Ready
- Out for Delivery
- Delivered

Notifications should remain simple.

---

# 57. Staff Notifications

Staff may receive:

- New customer order
- Pickup request
- Important operational updates

---

# 58. Offline-First Operation

The application must continue working during temporary internet loss.

Core rule:

```text
User action
 ↓
Validate
 ↓
SQLite
 ↓
UI updates immediately
 ↓
Sync Queue
 ↓
Internet available
 ↓
Firebase
```

The UI must not unnecessarily wait for Firebase before displaying local changes.

---

# 59. Offline Order Example

Staff creates:

```text
ORD1052
```

Internet is unavailable.

System:

```text
SQLite:
ORD1052 saved

Sync Queue:
PENDING
```

Staff can continue working.

When internet returns:

```text
Sync Queue
 ↓
Upload to Firebase
 ↓
Success
 ↓
Mark SYNCED
```

If upload fails:

```text
Retry
```

Data must never silently disappear.

---

# 60. Sync Queue

SQLite must contain a Sync Queue.

Suggested fields:

```text
syncId
entityType
entityId
operation
status
createdAt
updatedAt
retryCount
lastError
```

Possible operations:

```text
CREATE
UPDATE
DELETE
```

Sensitive/historical records should avoid hard deletion wherever possible.

---

# 61. Sync Error Handling

If synchronization fails:

```text
Data remains in SQLite
Sync status = PENDING/FAILED
Retry later
```

User message:

> Internet connection is unavailable. Your data has been saved on this device and will sync automatically.

If pending records exist:

> Some data is waiting to sync. Please keep the app open when internet is available.

Raw Firebase errors must not be displayed.

---

# 62. Background Synchronization

The application should use an Expo-compatible background/network-aware synchronization approach.

The architecture should keep synchronization logic independent from UI.

If guaranteed native background execution is later required, native Android background scheduling such as WorkManager can be integrated without changing the core database/sync architecture.

Sync must never block normal UI usage.

---

# 63. SQLite Responsibilities

SQLite is the local operational database.

It stores cached/local copies of:

- Users
- Customers
- Items
- Services
- Prices
- Orders
- Order Items
- Payments
- Status History
- Business Settings
- Sync Queue

SQLite enables:

- Offline search
- Fast dashboard
- Fast customer lookup
- Fast order lookup
- Local invoice generation
- Local report generation

---

# 64. Firestore Responsibilities

Firestore is the cloud/master business data store.

It provides:

- Long-term business data
- Cross-device synchronization
- Future Web Admin access
- Cloud backup
- Centralized business records

---

# 65. Local vs Cloud Data

```text
SQLite
= Local/offline operational copy

Firestore
= Cloud/master business record

PDF/XLSX
= Generated local files
```

Generated PDFs and Excel reports do not need to be uploaded to Firebase in V1.

---

# 66. Firestore Structure

Recommended structure:

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
```

---

# 67. User Document

```text
users/{userId}

userId
name
email
phone
role
businessId
status
profileCompleted
createdAt
updatedAt
```

For customers, profile information may additionally be stored in the business customer record according to the final implementation design.

---

# 68. Customer Document

```text
customerId
userId
businessId

name
email
phone

customerType

businessName
businessType
businessAddress
businessPhone

address
pinCode

status

createdAt
updatedAt
```

---

# 69. Business Document

```text
businessId

businessName
address
phone
email

GSTIN
defaultGSTRate

invoicePrefix

createdAt
updatedAt
```

---

# 70. Item Document

```text
itemId
businessId
name
status
createdAt
updatedAt
```

Status:

```text
ACTIVE
INACTIVE
```

---

# 71. Service Document

```text
serviceId
businessId
name
status
createdAt
updatedAt
```

---

# 72. Price Document

```text
priceId
businessId
itemId
serviceId
price
status
createdAt
updatedAt
```

---

# 73. Order Document

```text
orderId
businessId

customerId
source

customerNameAtOrder
customerPhoneAtOrder
businessNameAtOrder
pickupAddress

items[]

estimatedAmount
finalAmount

gstApplied
gstRate
gstAmount

paidAmount
dueAmount
paymentStatus
paymentMethod

orderStatus

pickupDate
pickupTime

customerNote

invoiceNumber

createdAt
updatedAt

createdBy
lastUpdatedBy
```

---

# 74. Audit Fields

Important records should include:

```text
createdAt
updatedAt
createdBy
lastUpdatedBy
```

Payment:

```text
recordedBy
```

Status history:

```text
changedBy
changedAt
```

These fields help identify who performed an operation.

---

# 75. Business ID

Even though V1 contains one laundry business, business-related data should contain:

```text
businessId
```

This prepares the architecture for future expansion without building a multi-business UI now.

There should be NO Super Admin or multi-business SaaS interface in V1.

---

# 76. Security Architecture

Security must be enforced at backend level.

Layers:

```text
Firebase Authentication
        ↓
Authenticated UID
        ↓
User Profile
        ↓
Role
        ↓
Business ID
        ↓
Firestore Security Rules
        ↓
Allowed operation
```

---

# 77. Customer Security

Customer can access only:

```text
Own profile
Own orders
Own permitted payment/invoice information
```

Customer cannot:

```text
Read another customer
Read another business
Modify role
Modify businessId
Modify finalAmount
Modify gstAmount
Modify payment records
Modify status history
```

---

# 78. Staff Security

Staff must belong to the same business.

Staff cannot:

```text
Create ADMIN
Change own role
Change businessId
Manage staff
Change price
Change GST business settings
Delete historical financial records
Access another business
```

---

# 79. Admin Security

Admin has business-level permissions.

Admin can manage:

```text
Staff
Customers
Items
Services
Prices
Orders
Payments
Business Settings
GST Settings
Reports
```

---

# 80. Firebase Admin SDK

Privileged operations such as staff account creation should be performed by secure backend functions.

The Firebase Admin SDK/service credentials must NEVER be included in:

- Android source code
- APK
- React Native bundle
- Git repository
- public configuration

---

# 81. Password Security

Passwords must be handled by Firebase Authentication.

The application must never store:

```text
plainTextPassword
```

inside Firestore.

Temporary staff passwords should be handled securely and replaced on first login.

---

# 82. Customer Profile Security

A customer may edit their own:

- Name
- Mobile
- Address
- PIN Code
- Business details where applicable

Protected fields:

```text
role
businessId
account status controlled by admin
financial fields
order ownership
```

must not be client-editable.

---

# 83. Order Ownership

Every customer order contains:

```text
customerId
```

Firestore security rules must ensure a customer can read only orders where:

```text
order.customerId == authenticatedUser.customerId
```

---

# 84. Order Modification Security

Customers must not be allowed to arbitrarily modify:

```text
finalAmount
paidAmount
dueAmount
gstAmount
paymentStatus
invoiceNumber
orderStatus
createdBy
businessId
```

Any customer-side order changes must use controlled application logic.

---

# 85. Data Consistency

Because Android devices and future Admin Web may modify the same data:

Use:

```text
updatedAt
updatedBy
```

and where useful:

```text
version
```

Important financial/status changes should not be silently overwritten.

---

# 86. Duplicate Order Prevention

Offline mode may retry the same operation.

The system must prevent duplicate creation.

Use:

- Client-generated unique IDs
- Sync IDs
- Idempotent backend operations
- Duplicate detection

Example:

```text
Client creates ORD1052
Network fails
Client retries
```

The backend must recognize the existing operation instead of creating ORD1052 twice.

---

# 87. Conflict Handling

If two clients modify the same important order:

```text
Device A → Update
Device B → Update
```

The system must track:

```text
updatedAt
updatedBy
```

Important payment/status history should be append-only where practical.

The application must not silently destroy financial history.

---

# 88. Reports

Reports are generated from stored historical order data.

The user can select a period.

Available:

```text
Weekly
Monthly
Yearly
```

---

# 89. Weekly Report

User selects:

```text
Month
Year
From Date
To Date
```

Example:

```text
September 2026
07 Sep 2026
13 Sep 2026
```

The date range should normally remain within the selected month.

Filename:

```text
Laundry_Report_07-09-2026_to_13-09-2026.xlsx
```

---

# 90. Monthly Report

User selects:

```text
Month
Year
```

Example:

```text
September 2026
```

Filename:

```text
Laundry_Report_September_2026.xlsx
```

---

# 91. Yearly Report

User selects:

```text
Year
```

Example:

```text
2026
```

Filename:

```text
Laundry_Report_2026.xlsx
```

---

# 92. Report Metrics

Reports should include:

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
- Service quantity
- Service revenue
- Item quantity
- Item revenue

---

# 93. Yearly Analytical Report

The yearly report should additionally contain:

### Monthly breakdown

```text
January
February
March
...
December
```

### Status summary

```text
New
Processing
Ready
Delivered
Cancelled
```

### Payment summary

```text
Cash
UPI
Online
Pending
```

### Service performance

Example:

```text
Wash
Total Quantity
Revenue
```

### Item performance

Example:

```text
Shirt
Quantity
Revenue
```

---

# 94. Report Data Accuracy

Reports must use stored historical values.

Never calculate old orders using current:

- Price
- GST rate
- Customer business name

Use:

```text
priceAtOrderTime
gstRate
gstAmount
finalAmount
```

stored with the order.

---

# 95. Excel Generation

Excel files are generated locally.

Flow:

```text
SQLite
 ↓
Filter by date
 ↓
Calculate report
 ↓
Generate XLSX
 ↓
Save locally
 ↓
Open/Share
```

No Firebase Storage required.

---

# 96. Offline Reports

If required data is already synchronized into SQLite:

```text
Report generation = offline capable
```

If some newer cloud records have not synchronized:

The application should clearly indicate that the report is based on locally synchronized data.

Example:

> Report generated from data available on this device. Some newer data may still be waiting to sync.

---

# 97. Dashboard

## Customer Dashboard

Should remain simple.

Possible information:

```text
Welcome, Rahul

Active Order
ORD1052
Processing

Recent Orders
```

Primary action:

```text
Place New Order
```

---

# 98. Staff Dashboard

Should focus on operations.

Example:

```text
New Orders
Pickup Pending
Processing
Ready
Out for Delivery
Unpaid
```

Staff should be able to quickly reach Orders.

---

# 99. Admin Dashboard

Admin dashboard can show:

```text
Today's Orders
Pending Orders
Processing
Ready
Delivered
Unpaid
Today's Revenue
```

Keep dashboard simple rather than turning it into a complicated ERP dashboard.

---

# 100. Customer Profile Screen

Customer profile:

```text
Name
Email
Mobile Number

Customer Type
Personal / Business

Business Name
Business Type

Address
PIN Code

Edit Profile
Change Password
Logout
```

Business fields are shown only when applicable.

---

# 101. Admin More Section

Possible:

```text
Business Settings
GST Settings
Invoice Settings
Items
Services
Prices
Sync Status
App Information
Logout
```

Only appropriate admin functions should be visible to Admin.

---

# 102. Business Settings

Admin can configure:

```text
Business Name
Address
Phone
Email
GSTIN
Default GST Rate
Invoice Prefix
```

These settings affect future operations.

Historical invoices/orders must retain their own historical values.

---

# 103. No Firebase Storage in V1

Firebase Storage is NOT required for:

- Invoices
- Excel reports
- Normal business records

Because:

- PDF is generated locally
- XLSX is generated locally
- No major image/file requirement exists in V1

Future Storage use may include:

- Business logo upload
- Clothing photos
- Order photos
- Other documents

---

# 104. Business Logo

V1 can bundle the business logo as an application asset.

Future version may allow Admin to upload a logo using Firebase Storage.

---

# 105. Notifications

FCM is optional but recommended.

Customer notifications:

```text
Order received
Pickup completed
Processing
Ready
Out for Delivery
Delivered
```

Notification failure must not affect order creation.

---

# 106. Network States

Application should distinguish:

```text
ONLINE
OFFLINE
SYNCING
SYNCED
SYNC_ERROR
```

User should be able to see when pending data exists.

---

# 107. Error Handling

Do not show raw technical errors.

Bad:

```text
FirebaseError: PERMISSION_DENIED
```

Good:

> You do not have permission to perform this action.

Bad:

```text
Network request failed
```

Good:

> Internet connection is unavailable. Your data has been saved on this device and will sync automatically.

---

# 108. Indian English

Use:

- Mobile Number
- PIN Code
- GST
- Pending Amount
- Pickup
- Delivery
- Customer
- Staff
- Admin

Avoid:

- Zip Code
- Sales Tax
- Fulfilled

---

# 109. Currency

All monetary values should use:

```text
₹
```

Example:

```text
₹500
₹1,250
₹5,900
```

Never display:

```text
Rs
$
```

unless specifically required for another context.

---

# 110. Date Format

Preferred:

```text
14 September 2026
```

Short:

```text
14 Sep 2026
```

Report filenames can use:

```text
07-09-2026_to_13-09-2026
```

---

# 111. UI Quality Requirements

The application must avoid:

- Text clipping
- Broken characters
- Broken ₹ symbol
- Overlapping buttons
- Text overflowing cards
- Long address clipping
- Long business name clipping
- Incorrect keyboard types
- Tiny touch targets

Long text should wrap properly.

---

# 112. Keyboard/Input Requirements

Email field:

```text
email keyboard
```

Mobile Number:

```text
numeric keyboard
```

PIN Code:

```text
numeric keyboard
```

Quantity:

```text
numeric keyboard
```

Password:

```text
secure password input
```

---

# 113. Validation

Examples:

### Mobile

- Required for completed customer profile.
- Validate Indian mobile format according to business requirement.
- Store consistently.

### PIN Code

- Numeric
- Appropriate Indian PIN format

### Quantity

- Must be greater than 0.

### Price

- Must not be negative.

### GST rate

- Must be valid according to configured business requirements.

---

# 114. Customer Business Selection

When profile is edited:

```text
Customer Type
```

Options:

```text
Personal
Business
```

If Business:

```text
Business Name
Business Type
```

If changed from Business to Personal:

The application should confirm the change before clearing business-specific information.

Historical order snapshots remain unchanged.

---

# 115. Customer Can Have Business Identity

The system does not create a separate Business User account in V1.

Instead:

```text
Customer
   ↓
Customer Type = BUSINESS
   ↓
Business Name
```

This keeps V1 simple.

A future version can introduce separate business accounts if required.

---

# 116. Multiple Business Handling

V1:

A customer profile contains one current business association.

If a customer regularly handles laundry for multiple businesses, future versions can introduce:

```text
Customer
 ├── Business A
 ├── Business B
 └── Business C
```

This is explicitly future scope.

---

# 117. Order Pickup Information

Order may contain:

```text
pickupAddress
pickupDate
pickupTime
```

Customer note:

```text
customerNote
```

Example:

> Please call before pickup.

---

# 118. Delivery Information

Current V1 can store required delivery/pickup information without building GPS route optimization.

Future:

- Delivery staff tracking
- GPS
- Route optimization

are excluded.

---

# 119. Data Retention

Firestore:

```text
Long-term business data
```

SQLite:

```text
Local operational/cache data
```

Generated PDF/XLSX:

```text
Local files
```

Historical business records should not be casually deleted.

---

# 120. Deletion Policy

For referenced master data:

Prefer:

```text
INACTIVE
```

instead of hard deletion.

Examples:

- Item
- Service
- Staff
- Customer

Orders and financial history should be protected.

---

# 121. Performance Requirements

Application should:

- Launch quickly
- Load local data quickly
- Search locally without unnecessary network calls
- Not block UI during sync
- Generate invoices locally
- Generate reports locally
- Avoid unnecessary Firebase reads
- Use pagination/efficient queries as data grows

---

# 122. Local Search

The following should work from SQLite:

- Customer search
- Order search
- Item search
- Service lookup
- Business name search where applicable

This allows staff to continue basic work offline.

---

# 123. Sync Indicator

A simple indicator should show:

```text
✓ Synced
```

or:

```text
↻ Syncing...
```

or:

```text
! 3 items waiting to sync
```

This is especially important because users may otherwise assume cloud backup has already happened.

---

# 124. Application Startup

Startup flow:

```text
Open App
 ↓
Initialize SQLite
 ↓
Check Firebase Auth
 ↓
Load local profile
 ↓
Load local dashboard
 ↓
Check network
 ↓
Sync if required
 ↓
Update UI
```

The user should not have to wait unnecessarily for the cloud.

---

# 125. Logout

Logout should:

- Sign out Firebase Authentication.
- Clear sensitive authentication state from secure local storage.
- Keep only non-sensitive cached business data where appropriate.
- Prevent access to protected screens after logout.

For shared staff devices, sensitive local data handling should be carefully considered.

---

# 126. Session Handling

Firebase Authentication manages authentication sessions.

The app must verify current authenticated user before accessing protected operations.

Role must be fetched from trusted user data.

Client-side stored role alone must not be trusted for security.

---

# 127. Future Admin Web

Future application:

```text
React / Next.js
TypeScript
```

It will connect to the same Firebase backend.

Architecture:

```text
Android App ─── Firebase
                   │
                   │
             Admin Web
```

The Web Admin should be able to perform Admin operations without creating a separate database.

---

# 128. Future Web Admin Functions

Potential:

- Dashboard
- Orders
- Customers
- Staff
- Items
- Services
- Prices
- Reports
- Business Settings
- GST
- Invoice management

The exact Web UI is outside V1 Android scope.

---

# 129. V1 Exclusions

The following must NOT be implemented in V1:

- Super Admin
- Multi-business SaaS
- Multi-branch
- Multiple processing centres
- Separate delivery application
- Separate manager application
- Inventory
- Salary
- Payroll
- Expense management
- Advanced accounting
- GPS route optimization
- Garment QR tracking
- Barcode tracking
- POS hardware
- Printer integration
- Loyalty system
- Coupons
- Corporate accounts
- Advanced CRM
- AI features

---

# 130. Future Scope

Possible future features:

- Online payment gateway
- WhatsApp notifications
- SMS
- Business logo upload
- Order/clothing photos
- Expense management
- Advanced analytics
- Multi-business
- Multi-branch
- QR/barcode tracking
- Loyalty
- Coupons
- Delivery tracking
- Route optimization
- Separate delivery staff module
- Multiple business associations per customer

---

# 131. Complete Customer Journey

```text
Install App
    ↓
Register
    ↓
Email + Password
    ↓
Email Verification
    ↓
Complete Profile
    ↓
Name
Mobile
Personal / Business
Address
    ↓
If Business
    ↓
Business Name
Business Type
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
GST:
With / Without
    ↓
Estimated Amount
    ↓
Confirm
    ↓
Order Created
    ↓
NEW
    ↓
Pickup
    ↓
Processing
    ↓
Ready
    ↓
Out for Delivery
    ↓
Delivered
```

---

# 132. Complete Staff Journey

```text
Admin Creates Staff
        ↓
Staff Receives Login
        ↓
Staff Login
        ↓
First Password Change
        ↓
Staff Dashboard
        ↓
New Orders
        ↓
Pickup
        ↓
Processing
        ↓
Ready
        ↓
Delivery
        ↓
Payment
        ↓
Delivered
```

Manual order:

```text
Staff
 ↓
Create Order
 ↓
Customer
 ↓
Items
 ↓
Services
 ↓
Quantity
 ↓
GST
 ↓
Payment
 ↓
Order Created
```

---

# 133. Complete Admin Journey

```text
Admin Login
    ↓
Dashboard
    ↓
People
 ├── Staff
 └── Customers

Orders
    ↓
Search / Filter
    ↓
Open Order
    ↓
Update / Payment / Invoice

More
 ├── Items
 ├── Services
 ├── Prices
 ├── Business Settings
 ├── GST
 └── Invoice Settings

Reports
 ├── Weekly
 ├── Monthly
 └── Yearly
```

---

# 134. Complete Offline Journey

```text
User Action
    ↓
Validate
    ↓
SQLite Transaction
    ↓
UI Updated Immediately
    ↓
Sync Queue
    ↓
Network Available?
   ├── NO → Keep Pending
   └── YES
          ↓
      Firebase Sync
          ↓
       Success?
       ├── YES → SYNCED
       └── NO  → RETRY
```

---

# 135. Complete Order Calculation

Example:

```text
Shirt + Wash + Iron
Quantity = 5
Rate = ₹50

Subtotal:
5 × ₹50 = ₹250
```

With GST:

```text
GST = 18%
GST = ₹45

Grand Total = ₹295
```

If customer pays ₹200:

```text
Paid = ₹200
Due = ₹95

Payment Status:
PARTIALLY_PAID
```

---

# 136. Example Business Customer Order

Customer:

```text
Name: Amit
Mobile: 98XXXXXXXX
Type: Business
Business: Hotel Paradise
```

Order:

```text
Order ID: ORD1052
Invoice: INV1052

Customer:
Amit

Business:
Hotel Paradise

Items:
10 Shirts
5 Pants

Service:
Wash + Iron

Subtotal:
₹750

GST:
18%

GST:
₹135

Grand Total:
₹885
```

Staff sees:

```text
Amit
Hotel Paradise
ORD1052
Processing
₹885
₹500 Paid
₹385 Pending
```

---

# 137. Database Relationship

Conceptually:

```text
USER
 │
 ├── CUSTOMER
 │      │
 │      ├── Business Information
 │      │
 │      └── ORDERS
 │              │
 │              ├── ORDER ITEMS
 │              ├── PAYMENTS
 │              └── STATUS HISTORY
 │
 ├── STAFF
 │
 └── ADMIN
```

Business:

```text
BUSINESS
 ├── Staff
 ├── Customers
 ├── Items
 ├── Services
 ├── Prices
 ├── Orders
 └── Settings
```

---

# 138. Recommended SQLite Tables

```text
users
customers
items
services
prices
orders
order_items
payments
status_history
business_settings
sync_queue
```

Potential supporting tables:

```text
app_metadata
notification_tokens
```

depending on implementation.

---

# 139. IDs

Every major entity must have a unique ID.

Examples:

```text
customerId
staffId
itemId
serviceId
priceId
orderId
paymentId
historyId
syncId
```

IDs should be unique across offline devices.

UUID/client-generated IDs are recommended.

---

# 140. Transactions

Important local operations should use SQLite transactions.

For order creation:

```text
Create Order
+
Create Order Items
+
Create Sync Queue Entry
```

should be treated as one logical operation.

If one part fails, the application should not leave an incomplete local order.

---

# 141. Financial Integrity

The application must maintain:

```text
Subtotal
GST
Grand Total
Paid
Due
```

with consistent calculations.

Recommended:

```text
dueAmount = finalAmount - paidAmount
```

Payment records should provide the audit trail.

---

# 142. Final Amount Rules

Customer initially sees:

```text
estimatedAmount
```

Staff/Admin may confirm:

```text
finalAmount
```

Once finalized, changes should be controlled and audited.

---

# 143. GST Historical Integrity

Example:

Business default:

```text
18%
```

Order created:

```text
gstRate = 18
gstAmount = ₹90
```

Later Admin changes default:

```text
12%
```

Old order remains:

```text
18%
₹90
```

Only new applicable orders use the new default.

---

# 144. Price Historical Integrity

Example:

Before:

```text
Shirt Wash = ₹30
```

Order:

```text
priceAtOrderTime = ₹30
```

Admin changes:

```text
Shirt Wash = ₹40
```

Old order:

```text
₹30
```

New order:

```text
₹40
```

---

# 145. Access Matrix

| Feature | Customer | Staff | Admin |
|---|---:|---:|---:|
| Register | Yes | No | No |
| Login | Yes | Yes | Yes |
| Edit Own Profile | Yes | Limited | Yes |
| View Own Orders | Yes | Yes | Yes |
| View All Orders | No | Yes | Yes |
| Create Customer Order | Yes | Yes | Yes |
| Create Manual Order | No | Yes | Yes |
| Manage Staff | No | No | Yes |
| Manage Customers | Own | Limited | Yes |
| Manage Items | No | No | Yes |
| Manage Services | No | No | Yes |
| Manage Prices | No | No | Yes |
| Manage GST Settings | No | No | Yes |
| Record Payment | No/limited | Yes | Yes |
| Change Order Status | No | Yes | Yes |
| Generate Invoice | Own | Yes | Yes |
| Generate Reports | No | Limited/No | Yes |
| Business Settings | No | No | Yes |
| Create Admin | No | No | Controlled setup only |

---

# 146. Important Security Rule

The Access Matrix is a product requirement.

The actual implementation must enforce it using:

```text
Firebase Authentication
+
Firestore Security Rules
+
Cloud Functions
```

The application UI alone must never be considered security.

---

# 147. V1 Definition of Done

V1 is considered complete when:

### Authentication

- Customer registration works.
- Email verification works.
- Login works with email/password.
- Forgot password works.
- Staff cannot publicly register.
- Admin can create staff.
- Roles route correctly.

### Customer

- Profile works.
- Personal/Business selection works.
- Business information works.
- Customer can place order.
- Customer sees own orders.
- Customer cannot access other customers.

### Staff

- Staff can log in.
- Staff can create manual orders.
- Staff can manage permitted statuses.
- Staff can record permitted payments.
- Staff can search orders.

### Admin

- Staff management works.
- Customer management works.
- Item management works.
- Service management works.
- Price management works.
- GST settings work.
- Order management works.
- Reports work.

### Offline

- Core operations work without internet.
- SQLite stores changes.
- Sync Queue records pending operations.
- Data synchronizes after connection returns.
- Failed syncs retry.

### Invoice

- A4 PDF works.
- ₹ displays correctly.
- Long text wraps.
- GST appears correctly.
- Payment status appears correctly.

### Reports

- Weekly works.
- Monthly works.
- Yearly works.
- XLSX generation works.
- Historical values are correct.

### Security

- Role-based access works.
- Customer isolation works.
- Business isolation works.
- Staff restrictions work.
- Admin-only operations are protected.
- Firebase Admin credentials are never exposed.

---

# 148. Final Product Flow

```text
                         ┌──────────────┐
                         │   OPEN APP   │
                         └──────┬───────┘
                                ↓
                       ┌─────────────────┐
                       │ Authentication  │
                       └────────┬────────┘
                                ↓
                    ┌──────────────────────┐
                    │ Firebase Auth + UID  │
                    └──────────┬───────────┘
                               ↓
                         Role + Profile
                               ↓
             ┌─────────────────┼─────────────────┐
             ↓                 ↓                 ↓
         CUSTOMER            STAFF             ADMIN
             │                 │                 │
             ↓                 ↓                 ↓
        Customer Home     Staff Home       Admin Home
             │                 │                 │
             ↓                 ↓                 ↓
          Orders            Orders            Orders
             │                 │                 │
             ↓                 ↓                 ↓
       Own Orders       Operational Work    Full Control
             │                 │                 │
             └─────────────────┼─────────────────┘
                               ↓
                           SQLite
                               ↓
                         Sync Queue
                               ↓
                            Firebase
                               ↓
                           Firestore
                               ↓
                    Future Admin Web App
```

---

# 149. Final Architecture Summary

```text
                 ┌─────────────────────┐
                 │   React Native App  │
                 │       Expo + TS     │
                 └──────────┬──────────┘
                            │
                 ┌──────────┴──────────┐
                 │                     │
          ┌──────▼──────┐      ┌──────▼──────┐
          │   SQLite    │      │   Firebase  │
          │ Local DB    │      │   Backend   │
          └──────┬──────┘      └──────┬──────┘
                 │                     │
          ┌──────▼──────┐       ┌──────▼─────────┐
          │ Sync Queue  │       │ Authentication │
          └─────────────┘       │ Firestore      │
                                │ Cloud Functions│
                                │ FCM             │
                                └──────┬─────────┘
                                       │
                              Future Admin Web
```

---

# 150. Final Product Philosophy

The final application should feel like a **simple laundry business operating app**, not a large enterprise ERP.

The most important goals are:

```text
FAST
SIMPLE
OFFLINE
SECURE
ACCURATE
EASY TO USE
```

The application should allow the owner and staff to perform normal laundry operations with minimum taps while maintaining reliable historical business records.

The architecture should remain strong enough to support a future Admin Web application without rebuilding the backend.

**V1 should build only what the laundry business actually needs.**