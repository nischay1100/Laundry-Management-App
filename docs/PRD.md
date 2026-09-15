# Laundry Management App — Product Requirements Document

**Document:** PRD.md  
**Version:** 1.0  
**Status:** Approved Product Direction  
**Platform:** Android  
**Future Platform:** Admin Web  
**Product Type:** Local Laundry Management Application  
**Architecture:** Offline-first  
**Primary Language:** Indian English  
**Currency:** Indian Rupee (₹)

---

# 1. Product Overview

The Laundry Management App is an Android-first application designed for a local laundry business.

The application will help the business manage:

- Customers
- Staff
- Laundry items
- Laundry services
- Pricing
- Orders
- Pickup and delivery workflow
- Payments
- GST
- Invoices
- Reports

The application must be **simple, fast and easy to operate**.

It must NOT become a complicated ERP system.

The application will support offline operation using a local SQLite database and synchronize data with Firebase when internet connectivity is available.

---

# 2. Product Goal

The main goal is:

> Provide one simple application through which the laundry owner, staff and customers can manage the complete laundry-order lifecycle with reliable offline support and cloud synchronization.

The application should reduce:

- Manual paperwork
- Order tracking problems
- Payment tracking problems
- Customer lookup time
- Pricing mistakes
- GST calculation mistakes
- Report preparation work

---

# 3. Target Users

There are three user roles:

```text
CUSTOMER
STAFF
ADMIN
```

---

# 4. Customer Persona

A customer can be:

### Personal Customer

Someone sending their own clothes/laundry.

Example:

```text
Rahul Sharma
Customer Type: Personal
```

### Business Customer

Someone sending laundry on behalf of a business.

Example:

```text
Rahul Sharma
Customer Type: Business
Business Name: Hotel Paradise
Business Type: Hotel
```

This allows Admin/Staff to identify which business an order belongs to when the person is operating on behalf of a hotel, homestay, restaurant, etc.

---

# 5. Staff Persona

Staff members are employees who operate the laundry business.

Typical responsibilities:

- Receiving orders
- Creating manual orders
- Handling pickup
- Processing orders
- Updating order status
- Recording payments
- Delivering orders
- Searching customers/orders

Staff accounts are created by Admin.

Staff cannot register themselves publicly.

---

# 6. Admin Persona

Admin is the owner/business administrator.

Admin has complete control over the laundry business.

Admin manages:

- Staff
- Customers
- Items
- Services
- Prices
- Orders
- Payments
- GST settings
- Business settings
- Reports
- Invoices

---

# 7. Product Principles

The product must follow these principles:

1. Simple
2. Fast
3. Offline-first
4. Secure
5. Mobile-first
6. Easy for non-technical staff
7. Minimal number of taps
8. Reliable historical records
9. Indian English
10. Indian currency and date formats
11. No unnecessary enterprise features

---

# 8. Platform Scope

## V1

Android mobile application.

The same application contains:

```text
Customer Panel
Staff Panel
Admin Panel
```

The panel shown depends on the authenticated user's role.

---

# 9. Future Platform

A future Admin Web Application will be developed for desktop/laptop use.

The future web application will use the same Firebase backend.

Conceptually:

```text
Android App ──────┐
                  │
                  ├── Firebase Backend
                  │
Admin Web ────────┘
```

The Android application must therefore be designed so that the backend is not tightly coupled to Android-only logic.

---

# 10. Authentication Strategy

Customer authentication is intentionally simple.

## Customer Registration

Customer enters:

```text
Email
Password
Confirm Password
```

Then:

```text
Register
 ↓
Firebase Authentication Account
 ↓
Email Verification
 ↓
Complete Your Profile
```

No phone OTP is required.

---

# 11. Email Verification

Email verification is mandatory for a newly registered customer.

Flow:

```text
Register
 ↓
Verification Email
 ↓
Customer opens email
 ↓
Clicks verification link
 ↓
Email verified
 ↓
Complete Your Profile
```

Email verification proves ownership of the email address.

---

# 12. Phone Number Policy

The application will NOT use phone OTP in V1.

The customer provides a mobile number during profile completion.

Important:

> Email verification must NOT be represented technically as phone verification.

The phone number is stored as contact information.

No SMS OTP cost is required for customer registration/login.

---

# 13. Complete Your Profile

After email verification, the customer is redirected to:

```text
Complete Your Profile
```

Required information:

- Full Name
- Mobile Number
- Customer Type
- Address
- PIN Code

If Customer Type is Business:

- Business Name
- Business Type

---

# 14. Customer Type

The customer must select:

```text
Personal
Business
```

Default:

```text
Personal
```

---

# 15. Personal Customer Profile

Example:

```text
Name:
Rahul Sharma

Email:
rahul@example.com

Mobile:
98XXXXXXXX

Customer Type:
Personal

Business:
None
```

Database representation:

```text
customerType = PERSONAL
businessName = null
businessType = null
```

---

# 16. Business Customer Profile

Example:

```text
Name:
Rahul Sharma

Mobile:
98XXXXXXXX

Customer Type:
Business

Business Name:
Hotel Paradise

Business Type:
Hotel
```

When this customer creates an order, the order should identify:

```text
Customer:
Rahul Sharma

Business:
Hotel Paradise
```

---

# 17. Why Business Information Exists

A person may send laundry on behalf of a hotel or another business.

Without business information, Staff/Admin would only see:

```text
Rahul Sharma
```

With business information they can see:

```text
Rahul Sharma
Hotel Paradise
```

This makes business orders easier to identify and report.

---

# 18. Business Information V1

A customer can have one current business association in V1.

Fields:

```text
businessName
businessType
businessAddress
businessPhone
```

Business address/phone can remain optional if not required.

Multiple businesses for one customer are future scope.

---

# 19. Login

Normal login:

```text
Email
Password
```

No OTP.

Flow:

```text
Login
 ↓
Firebase Authentication
 ↓
Email verified?
 ↓
Load User Profile
 ↓
Check Role
 ↓
Open Correct Panel
```

---

# 20. Login with Incomplete Profile

If email is verified but profile is incomplete:

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

The system should maintain:

```text
profileCompleted = false
```

until required profile data is saved.

---

# 21. Forgot Password

The recommended V1 flow uses Firebase password reset.

```text
Forgot Password
 ↓
Enter Email
 ↓
Password Reset Email
 ↓
Open Secure Reset Link
 ↓
Set New Password
 ↓
Login
```

A custom OTP infrastructure is not required.

---

# 22. Staff Authentication

Staff cannot register from the public registration screen.

Only Admin can create staff accounts.

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
Staff Information
 ↓
Secure Backend Function
 ↓
Firebase Auth Account
 ↓
Staff Login
```

First login should require changing the temporary password.

---

# 23. Admin Authentication

Admin does not use public registration.

The initial Admin account is created through a controlled setup process.

There must not be a public:

```text
Create Admin Account
```

option.

---

# 24. Role-Based Application

One Android application is used by all three roles.

After authentication:

```text
Firebase Auth
 ↓
User UID
 ↓
Trusted User Profile
 ↓
Role
```

Routing:

```text
CUSTOMER → Customer Panel

STAFF → Staff Panel

ADMIN → Admin Panel
```

---

# 25. Customer Navigation

Customer bottom navigation:

```text
Home
Orders
Profile
```

---

# 26. Staff Navigation

Staff bottom navigation:

```text
Home
Orders
Profile
```

---

# 27. Admin Navigation

Admin bottom navigation:

```text
Home
Orders
People
Reports
More
```

---

# 28. Customer Features

Customer can:

- Register
- Verify email
- Complete profile
- Select Personal/Business
- Add business details
- Edit profile
- Login
- Reset password
- Place order
- View estimated amount
- View order status
- View order history
- View invoice
- View payment status
- Receive order notifications

---

# 29. Staff Features

Staff can:

- Login
- View operational dashboard
- View orders
- Search orders
- Filter orders
- Create manual orders
- Create walk-in orders
- Create phone orders
- Select existing customer
- Create customer where permitted
- Add items
- Select services
- View current prices
- Update order status
- Record payments where permitted
- Generate/view invoice where permitted
- Work offline
- Synchronize pending data

---

# 30. Admin Features

Admin can:

- Manage Staff
- Manage Customers
- Manage Items
- Manage Services
- Manage Prices
- Manage Orders
- Manage Payments
- Configure Business
- Configure GST
- Generate invoices
- Generate reports
- View customer history
- Activate/deactivate staff

---

# 31. People Section

Admin navigation:

```text
People
├── Staff
└── Customers
```

---

# 32. Staff Management

Admin can:

- Add Staff
- View Staff
- Edit Staff
- Deactivate Staff
- Reactivate Staff

Staff status:

```text
ACTIVE
INACTIVE
```

Historical Staff records should be preserved.

Permanent deletion should not be the normal operation.

---

# 33. Customer Management

Admin can:

- View customers
- Search customers
- Open customer profile
- View customer orders
- View customer payment summary
- View business information

Search:

- Name
- Mobile Number
- Email
- Business Name

---

# 34. Customer Profile Summary

Admin customer profile should display:

```text
Name
Email
Mobile Number

Customer Type

Business Name
Business Type

Address
PIN Code

Registration Date
Status
```

And:

```text
Total Orders
Completed Orders
Cancelled Orders
Total Billed
Total Paid
Pending Amount
```

---

# 35. Customer Order History

Opening a customer must show only that customer's orders.

Example:

```text
Rahul Sharma
Hotel Paradise

ORD1001
ORD1025
ORD1052
```

---

# 36. Historical Customer Information

When an order is created, important customer information should be stored as a snapshot.

Examples:

```text
customerNameAtOrder
customerPhoneAtOrder
businessNameAtOrder
pickupAddress
```

This prevents old invoices from changing when the customer later edits their profile.

---

# 37. Items

Items represent laundry articles.

Examples:

- Shirt
- Pant
- Jeans
- T-Shirt
- Saree
- Bedsheet
- Blanket

Admin controls items.

Item status:

```text
ACTIVE
INACTIVE
```

Referenced items should normally be disabled instead of deleted.

---

# 38. Services

Services represent laundry work.

Examples:

- Wash
- Wash + Iron
- Iron
- Dry Clean

Admin controls services.

Service status:

```text
ACTIVE
INACTIVE
```

---

# 39. Pricing Model

Price is based on:

```text
Item + Service
```

Examples:

```text
Shirt + Wash = ₹30
Shirt + Wash + Iron = ₹50
Pant + Wash = ₹40
```

Prices are managed by Admin.

---

# 40. Pricing Rule

Prices must not be hardcoded.

They are stored in Firebase and cached locally in SQLite.

When an order is created:

```text
Current Price
 ↓
Copy into Order
 ↓
priceAtOrderTime
```

---

# 41. Historical Price Rule

Old orders must never change because the Admin changes today's price.

Example:

```text
Old:
Shirt + Wash = ₹30

Order created:
priceAtOrderTime = ₹30

Admin changes price:
₹40
```

Old order remains:

```text
₹30
```

---

# 42. Order Source

Orders have:

```text
CUSTOMER
STAFF
```

Customer-created:

```text
source = CUSTOMER
```

Manual/walk-in/phone:

```text
source = STAFF
```

---

# 43. Customer Order Flow

```text
Customer
 ↓
Place Order
 ↓
Select Item
 ↓
Select Service
 ↓
Enter Quantity
 ↓
Pickup Details
 ↓
GST Selection
 ↓
Estimated Amount
 ↓
Review
 ↓
Confirm
 ↓
Order Created
 ↓
NEW
```

---

# 44. Staff Manual Order Flow

```text
Staff
 ↓
Create Order
 ↓
Select Customer
      OR
Create Customer
 ↓
Items
 ↓
Services
 ↓
Quantity
 ↓
Pickup/Delivery Details
 ↓
GST
 ↓
Payment
 ↓
Create Order
```

---

# 45. Order Statuses

Detailed statuses:

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

---

# 46. Simple Order Filters

UI should present simple operational filters:

```text
New
Pickup
Processing
Delivered
Paid
Unpaid
Cancelled
```

The detailed internal status mapping is:

```text
New
→ NEW

Pickup
→ PICKUP_PENDING / PICKED_UP

Processing
→ PROCESSING

Delivered
→ DELIVERED

Cancelled
→ CANCELLED
```

Payment filters are independent.

---

# 47. Combined Filters

Filters should be combinable.

Example:

```text
Delivered
+
Unpaid
```

Result:

```text
orderStatus = DELIVERED
paymentStatus != PAID
```

---

# 48. Order Status Flow

Normal:

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
Allowed operational stage
 ↓
CANCELLED
```

Invalid status jumps should normally be prevented.

---

# 49. Status History

Every status change creates a history entry.

Information:

```text
Status
Changed At
Changed By
```

Example:

```text
NEW
14 Sep 2026
Staff A

PICKED_UP
14 Sep 2026
Staff B

PROCESSING
14 Sep 2026
Staff A
```

---

# 50. Order Search

Admin/Staff can search by:

- Order ID
- Invoice Number
- Customer Name
- Mobile Number

Examples:

```text
ORD1052
INV1052
Rahul
98XXXXXXXX
```

---

# 51. Order Items

Each order item contains:

```text
Item
Service
Quantity
Rate
Subtotal
```

Example:

```text
Shirt
Wash + Iron
5
₹50
₹250
```

---

# 52. Estimated Amount

When customer creates an order:

```text
estimatedAmount
```

is shown.

Staff/Admin may later verify actual quantity and final amount.

The order therefore maintains:

```text
estimatedAmount
finalAmount
```

---

# 53. GST Product Requirement

GST is **not globally enabled or disabled**.

GST is selected independently for each order.

Order UI:

```text
GST

○ With GST
○ Without GST
```

---

# 54. GST Business Configuration

Admin can configure:

```text
GSTIN
Default GST Rate
```

The default rate is only the default value for applicable new orders.

It does not force GST on every order.

---

# 55. GST Calculation

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

Order stores:

```text
gstApplied
gstRate
gstAmount
```

---

# 56. Historical GST Rule

If the Admin later changes the default GST rate, old orders must remain unchanged.

Example:

```text
Old order:
GST Rate = 18%
GST Amount = ₹90
```

Admin changes default to:

```text
12%
```

Old order remains:

```text
18%
₹90
```

---

# 57. Payment Status

Payment statuses:

```text
PENDING
PARTIALLY_PAID
PAID
```

---

# 58. Payment Methods

Supported V1 methods:

```text
CASH
UPI
ONLINE
```

Online payment gateway is future/optional.

---

# 59. Payment Records

Each payment contains:

```text
paymentId
orderId
amount
paymentMethod
paymentDate
recordedBy
note
```

Multiple payments can be made against one order.

---

# 60. Payment Example

Order:

```text
Grand Total = ₹590
```

Customer pays:

```text
₹200 Cash
₹200 UPI
```

Then:

```text
Paid = ₹400
Due = ₹190
Payment Status = PARTIALLY_PAID
```

---

# 61. Payment/Order Independence

Order status and payment status are separate.

Valid example:

```text
Order Status:
DELIVERED

Payment Status:
PENDING
```

Therefore:

> Delivered does not automatically mean Paid.

---

# 62. Invoice

Every order should have:

```text
Order ID
Invoice Number
```

Example:

```text
ORD1025
INV1025
```

They must be separate identifiers.

---

# 63. Invoice Generation

Invoice is generated locally.

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

No Firebase Storage is required for V1 invoices.

---

# 64. Invoice Content

Invoice includes:

## Business

- Business Name
- Address
- Phone
- Email if configured
- GSTIN if applicable

## Invoice

- Invoice Number
- Order ID
- Invoice Date

## Customer

- Name
- Mobile
- Address
- Business Name where applicable

## Items

- Item
- Service
- Quantity
- Rate
- Amount

## Totals

- Subtotal
- GST Rate
- GST Amount
- Grand Total
- Paid Amount
- Due Amount
- Payment Status
- Payment Method

## Footer

- Thank-you note

---

# 65. Invoice Quality

PDF must correctly support:

- ₹ symbol
- Indian number formatting
- Long names
- Long addresses
- Long business names
- Multiple items
- Text wrapping
- Correct A4 layout
- No clipping
- No overlapping
- No broken characters

---

# 66. Customer Order Tracking

Customer should see:

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

And a status timeline.

---

# 67. Notifications

Firebase Cloud Messaging may be used.

Customer notifications:

- Order received
- Pickup completed
- Processing
- Ready
- Out for Delivery
- Delivered

Staff notifications:

- New customer order
- Pickup request
- Operational updates

Notification failure must never prevent the order itself from being saved.

---

# 68. Offline-First Requirement

Offline support is a core product requirement.

The app must not become unusable simply because the internet is temporarily unavailable.

---

# 69. Offline Architecture

```text
User Action
 ↓
Validation
 ↓
SQLite
 ↓
UI Updated
 ↓
Sync Queue
 ↓
Internet Available
 ↓
Firebase
```

---

# 70. Offline Example

Staff creates:

```text
ORD1052
```

Internet unavailable.

The application:

```text
Saves order to SQLite
Adds Sync Queue record
Shows order immediately
```

When internet returns:

```text
Sync Queue
 ↓
Firebase
 ↓
Success
 ↓
Mark Synced
```

---

# 71. Failed Sync

If synchronization fails:

```text
Keep Local Data
 ↓
Retry
```

The application must never silently delete the local record.

User message:

> Internet connection is unavailable. Your data has been saved on this device and will sync automatically.

---

# 72. Sync Status

The user should be able to understand synchronization state.

Possible states:

```text
Synced
Syncing...
Waiting to sync
Sync failed
```

Example:

> 3 items waiting to sync.

---

# 73. Local Database

SQLite is responsible for fast local operation.

Local data includes:

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

---

# 74. Cloud Database

Firebase Firestore is the cloud/master data source.

It provides:

- Cloud persistence
- Cross-device synchronization
- Future Web Admin access
- Long-term business records

---

# 75. Data Ownership Model

```text
SQLite
= Local operational copy

Firestore
= Cloud master record

PDF/XLSX
= Generated local files
```

---

# 76. Reports

Admin can generate:

```text
Weekly
Monthly
Yearly
```

reports.

---

# 77. Weekly Report

Admin selects:

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

Filename:

```text
Laundry_Report_07-09-2026_to_13-09-2026.xlsx
```

---

# 78. Monthly Report

Admin selects:

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

# 79. Yearly Report

Admin selects:

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

# 80. Report Metrics

Reports must contain:

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

---

# 81. Yearly Analytics

Yearly report should additionally include:

### Monthly Breakdown

January–December.

### Status Summary

- New
- Pickup
- Processing
- Ready
- Delivered
- Cancelled

### Payment Summary

- Cash
- UPI
- Online
- Pending

### Service Performance

- Service
- Quantity
- Revenue

### Item Performance

- Item
- Quantity
- Revenue

---

# 82. Report Accuracy

Reports must use historical order values.

Reports must NOT recalculate historical orders using current prices or current GST settings.

Use stored:

```text
priceAtOrderTime
gstRate
gstAmount
finalAmount
```

---

# 83. Excel Generation

Excel is generated locally.

```text
SQLite
 ↓
Filter Data
 ↓
Calculate Metrics
 ↓
Generate XLSX
 ↓
Save
 ↓
Open / Share
```

No cloud upload is required.

---

# 84. Offline Reports

If required data exists in SQLite:

```text
Report generation works offline.
```

If some cloud records are not yet synchronized, the application must clearly inform the user.

Example:

> Report generated from data available on this device. Some newer data may still be waiting to sync.

---

# 85. Admin Dashboard

Admin dashboard should provide quick operational information:

```text
Today's Orders
Pending Orders
Processing
Ready
Delivered
Unpaid
Today's Revenue
```

It must remain simple.

---

# 86. Staff Dashboard

Staff dashboard should prioritize daily work:

```text
New Orders
Pickup Pending
Processing
Ready
Out for Delivery
Unpaid
```

The Orders screen should be easily accessible.

---

# 87. Customer Dashboard

Customer dashboard should show:

```text
Welcome
Active Order
Recent Orders
```

Primary action:

```text
Place New Order
```

---

# 88. Business Settings

Admin can manage:

```text
Business Name
Address
Phone
Email
GSTIN
Default GST Rate
Invoice Prefix
```

---

# 89. Invoice Settings

Admin may configure supported invoice settings such as:

- Invoice prefix
- Business information used on invoice
- Footer/thank-you text where supported

---

# 90. Data Retention

Firestore stores long-term business records.

SQLite stores local operational/cache data.

Generated files are stored locally.

Historical orders and financial records should be preserved.

---

# 91. Deletion Policy

Hard deletion should be avoided for records referenced by historical data.

Prefer:

```text
ACTIVE
INACTIVE
```

Examples:

- Staff
- Items
- Services
- Customers where appropriate

Historical orders must remain intact.

---

# 92. Security Product Requirements

The product must enforce:

### Customer

Can access only own data.

### Staff

Can access permitted business operational data.

### Admin

Can access complete business data.

---

# 93. Customer Restrictions

Customer cannot:

- Access another customer's orders
- Access Staff functions
- Access Admin functions
- Change role
- Change businessId
- Change final amount
- Change payment records
- Change GST records
- Change invoice number
- Change status history

---

# 94. Staff Restrictions

Staff cannot:

- Register as Staff
- Create Admin
- Create Staff accounts
- Change role
- Change businessId
- Manage Staff
- Change pricing
- Change GST business configuration
- Delete protected historical orders
- Access another business

---

# 95. Admin Restrictions

Admin operations must still respect business boundaries.

Admin must only access the configured business.

---

# 96. Audit Requirements

Important actions should record:

```text
createdAt
updatedAt
createdBy
lastUpdatedBy
```

Payments:

```text
recordedBy
```

Status history:

```text
changedBy
changedAt
```

---

# 97. Business Isolation

All business records should contain:

```text
businessId
```

Even though V1 contains one business.

This prepares the backend for possible future expansion.

V1 must NOT expose multi-business UI.

---

# 98. Data Consistency

Because multiple devices may work simultaneously:

The system should track:

```text
updatedAt
updatedBy
```

and where required:

```text
version
```

Important payment/status history must not be silently overwritten.

---

# 99. Duplicate Prevention

Offline retries must not create duplicate orders.

The system should use:

- Unique IDs
- Sync IDs
- Idempotent operations
- Duplicate protection

Example:

```text
Device creates ORD1052
 ↓
Network fails
 ↓
Retry
```

The server must not create another identical order.

---

# 100. UI Language

The app must use Indian English.

Preferred:

```text
Mobile Number
PIN Code
GST
Pending Amount
Pickup
Delivery
Customer
Staff
Admin
```

Avoid:

```text
Zip Code
Sales Tax
Fulfilled
```

---

# 101. Currency UI

Use:

```text
₹
```

Examples:

```text
₹500
₹1,250
₹5,900
```

---

# 102. Date UI

Preferred:

```text
14 September 2026
```

or:

```text
14 Sep 2026
```

---

# 103. Input UX

Mobile Number:

```text
Numeric Keyboard
```

PIN Code:

```text
Numeric Keyboard
```

Quantity:

```text
Numeric Keyboard
```

Email:

```text
Email Keyboard
```

Password:

```text
Secure Password Input
```

---

# 104. UI Quality

The application must avoid:

- Text clipping
- Broken ₹ symbol
- Overlapping elements
- Broken characters
- Long address clipping
- Long business name clipping
- Buttons overflowing
- Tiny touch targets

Long text must wrap properly.

---

# 105. Error Messages

Errors should be human-readable.

Example permission error:

> You do not have permission to perform this action.

Offline:

> Internet connection is unavailable. Your data has been saved on this device and will sync automatically.

Pending sync:

> Some data is waiting to sync. Please keep the app open when internet is available.

Raw Firebase errors must never be shown directly to normal users.

---

# 106. V1 Scope

V1 includes:

```text
Authentication
Customer Profiles
Personal/Business Customer Type
Staff Management
Customer Management
Items
Services
Pricing
Orders
Pickup/Delivery Workflow
Order Status
Payments
GST
Invoices
Reports
Offline SQLite
Firebase Sync
Notifications
Security
```

---

# 107. V1 Explicitly Excludes

Do NOT build:

```text
Super Admin
Multi-business SaaS UI
Multi-branch
Multiple Processing Centres
Separate Delivery App
Separate Manager App
Inventory
Salary
Payroll
Expense Management
Advanced Accounting
GPS Route Optimization
QR Garment Tracking
Barcode Tracking
POS Hardware
Printer Integration
Loyalty
Coupons
Corporate Accounts
Advanced CRM
AI Features
```

---

# 108. Future Scope

Potential future features:

- Online payment gateway
- WhatsApp
- SMS
- Business logo upload
- Clothing/order photos
- Expense management
- Advanced reports
- Multi-business
- Multi-branch
- QR/barcode
- Loyalty
- Coupons
- Delivery tracking
- Route optimization
- Multiple business associations per customer

---

# 109. End-to-End Customer Flow

```text
OPEN APP
 ↓
REGISTER
 ↓
EMAIL + PASSWORD
 ↓
EMAIL VERIFICATION
 ↓
COMPLETE YOUR PROFILE
 ↓
NAME
MOBILE
CUSTOMER TYPE
ADDRESS
PIN CODE
 ↓
If BUSINESS
 ↓
BUSINESS NAME
BUSINESS TYPE
 ↓
CUSTOMER HOME
 ↓
PLACE ORDER
 ↓
ITEM
 ↓
SERVICE
 ↓
QUANTITY
 ↓
PICKUP DETAILS
 ↓
GST
 ↓
ESTIMATED AMOUNT
 ↓
CONFIRM
 ↓
ORDER CREATED
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

---

# 110. End-to-End Staff Flow

```text
ADMIN CREATES STAFF
 ↓
STAFF LOGIN
 ↓
CHANGE TEMPORARY PASSWORD
 ↓
STAFF DASHBOARD
 ↓
ORDERS
 ↓
CREATE / MANAGE ORDER
 ↓
PICKUP
 ↓
PROCESSING
 ↓
READY
 ↓
DELIVERY
 ↓
PAYMENT
 ↓
DELIVERED
```

---

# 111. End-to-End Admin Flow

```text
ADMIN LOGIN
 ↓
ADMIN DASHBOARD
 ↓
PEOPLE
 ├── STAFF
 └── CUSTOMERS

ORDERS
 ↓
SEARCH / FILTER
 ↓
OPEN ORDER
 ↓
STATUS / PAYMENT / INVOICE

REPORTS
 ├── WEEKLY
 ├── MONTHLY
 └── YEARLY

MORE
 ├── ITEMS
 ├── SERVICES
 ├── PRICES
 ├── BUSINESS SETTINGS
 ├── GST
 └── INVOICE SETTINGS
```

---

# 112. End-to-End Offline Flow

```text
USER ACTION
 ↓
VALIDATE
 ↓
WRITE TO SQLITE
 ↓
UI UPDATE
 ↓
ADD SYNC QUEUE
 ↓
NETWORK AVAILABLE?
 ├── NO
 │    ↓
 │  KEEP PENDING
 │
 └── YES
      ↓
   FIREBASE
      ↓
   SUCCESS
      ↓
   SYNCED
```

---

# 113. Core Product Data Flow

```text
Customer
   ↓
Order
   ↓
Order Items
   ↓
Historical Prices
   ↓
GST Snapshot
   ↓
Payment
   ↓
Status History
   ↓
Invoice
   ↓
Reports
```

---

# 114. Critical Business Rules

## Rule 1 — Email

Customer must verify email after registration.

## Rule 2 — Phone

Phone OTP is not required in V1.

## Rule 3 — Customer Type

Default customer type is Personal.

## Rule 4 — Business

Business details are used only when Customer Type = Business.

## Rule 5 — Pricing

Current price is copied into the order.

## Rule 6 — GST

GST is selected per order.

## Rule 7 — Historical GST

Old GST values never change.

## Rule 8 — Payment

Payment status is independent from order status.

## Rule 9 — Invoice

Invoice number is separate from Order ID.

## Rule 10 — Offline

Local SQLite data must not be lost because of network failure.

## Rule 11 — Sync

Pending operations must retry.

## Rule 12 — Security

Role and business permissions must be enforced on the backend.

## Rule 13 — Staff

Staff cannot self-register.

## Rule 14 — Admin

Admin cannot be created through public registration.

## Rule 15 — Historical Data

Referenced records should be deactivated rather than deleted.

---

# 115. Product Success Criteria

The product succeeds if a laundry owner/staff member can perform daily work without needing technical knowledge.

A normal Staff user should be able to:

```text
Open App
 ↓
See Orders
 ↓
Find Customer
 ↓
Create/Update Order
 ↓
Update Status
 ↓
Record Payment
 ↓
Generate Invoice
```

with minimum unnecessary steps.

A Customer should be able to:

```text
Register
 ↓
Verify Email
 ↓
Complete Profile
 ↓
Place Order
 ↓
Track Order
```

without confusion.

Admin should be able to:

```text
Manage Business
Manage Staff
Manage Customers
Manage Pricing
Manage Orders
Manage Payments
Generate Reports
```

from the mobile app.

---

# 116. Definition of V1 Completion

V1 is complete only when all following areas work:

## Authentication

- Customer registration
- Email verification
- Login
- Forgot password
- Staff login
- Admin login
- Role routing

## Customer

- Profile
- Personal/Business
- Business information
- Orders
- Order history
- Status tracking

## Staff

- Operational dashboard
- Orders
- Manual orders
- Customer lookup
- Status updates
- Payments

## Admin

- Staff
- Customers
- Items
- Services
- Prices
- Orders
- Payments
- GST
- Business Settings
- Reports

## Offline

- SQLite
- Sync Queue
- Offline order creation
- Offline search
- Automatic synchronization
- Retry handling

## Documents

- A4 invoice PDF
- Weekly XLSX
- Monthly XLSX
- Yearly XLSX

## Security

- Role-based access
- Customer data isolation
- Business isolation
- Protected financial fields
- Secure Staff creation
- No exposed Admin credentials

---

# 117. Product Architecture Summary

```text
                         LAUNDRY APP
                              │
             ┌────────────────┼────────────────┐
             │                │                │
         CUSTOMER           STAFF            ADMIN
             │                │                │
             └────────────────┼────────────────┘
                              │
                         ANDROID APP
                              │
                    Expo + React Native
                         + TypeScript
                              │
             ┌────────────────┴────────────────┐
             │                                 │
          SQLite                           Firebase
       Offline Layer                     Cloud Layer
             │                                 │
       Sync Queue                    Authentication
             │                       Firestore
             │                       Cloud Functions
             │                       FCM
             │                                 │
             └────────────────┬────────────────┘
                              │
                       Future Admin Web
```

---

# 118. Final Product Statement

The Laundry Management App is a **simple, secure, offline-first Android application** for a local laundry business.

Its core workflow is:

```text
CUSTOMER
Register → Verify Email → Complete Profile → Order → Track

STAFF
Login → Orders → Pickup → Processing → Delivery → Payment

ADMIN
Login → Manage Business → Manage Orders → Manage People → Reports
```

The application must prioritise:

```text
Simplicity
Speed
Offline Reliability
Data Accuracy
Security
Historical Record Integrity
```

The product must remain intentionally small and practical in V1.

No feature should be added merely because it is common in large ERP systems.

---

# 119. Source-of-Truth Rule

This `PRD.md` defines the **product-level requirements**.

The following documents must follow this PRD:

```text
PRD.md
   ↓
TRD.md
   ↓
UI-UX.md
   ↓
BACKEND-SCHEMA.md
```

If a later document proposes a feature that conflicts with this PRD, the conflict must be resolved before implementation.

Technical implementation details belong in `TRD.md`.

Screen-level design details belong in `UI-UX.md`.

Database/Firebase/SQLite structures belong in `BACKEND-SCHEMA.md`.