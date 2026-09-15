# Laundry Management App — Technical Requirements Document

**Document:** TRD.md  
**Version:** 1.0  
**Status:** Implementation Ready  
**Related Document:** PRD.md  
**Platform:** Android  
**Architecture:** Offline-First  
**Primary Stack:** Expo + React Native + TypeScript  
**Backend:** Firebase  
**Local Database:** SQLite  

---

# 1. Purpose

This document defines the technical architecture and implementation requirements for the Laundry Management App.

It converts the product requirements from `PRD.md` into technical rules covering:

- Application architecture
- Technology stack
- Project structure
- Authentication
- Role-based access
- Firebase
- Firestore
- SQLite
- Offline-first operation
- Synchronization
- Cloud Functions
- Notifications
- PDF generation
- Excel generation
- Security
- Error handling
- State management
- Validation
- Performance
- Testing
- Deployment

This document must be followed during implementation.

---

# 2. Technical Goals

The application must be:

1. Offline-first
2. Fast on low-end Android devices
3. Secure
4. Maintainable
5. Type-safe
6. Scalable for future Admin Web
7. Resistant to duplicate sync operations
8. Reliable during poor network conditions
9. Easy for future developers to understand

---

# 3. Technology Stack

## 3.1 Mobile Application

```text
Expo
React Native
TypeScript
Expo Router
```

---

# 4. Local Data Layer

```text
SQLite
Expo SQLite
```

SQLite is the primary local operational database.

The UI must not depend directly on Firestore for normal screen rendering.

---

# 5. Cloud Backend

```text
Firebase Authentication
Cloud Firestore
Firebase Cloud Functions
Firebase Cloud Messaging
```

---

# 6. Document Generation

## Invoice

Generate locally as:

```text
PDF
```

## Reports

Generate locally as:

```text
XLSX
```

No Firebase Storage is required for V1 invoice/report generation.

---

# 7. Architecture

The application should follow a layered architecture.

```text id="w7u0w4"
Presentation Layer
       ↓
Application / Use Case Layer
       ↓
Repository Layer
       ↓
Local SQLite / Firebase
```

Recommended:

```text id="ax8j09"
UI
 ↓
Hooks / Controllers
 ↓
Use Cases
 ↓
Repositories
 ↓
SQLite / Firebase
```

UI components must not contain large database/business-logic implementations.

---

# 8. High-Level Architecture

```text id="v3aqn6"
                    Android Application
                           │
        ┌──────────────────┴──────────────────┐
        │                                     │
   Presentation                          Application
        │                                     │
   Screens/UI                         Use Cases/Services
        │                                     │
        └──────────────────┬──────────────────┘
                           │
                     Repository Layer
                           │
              ┌────────────┴────────────┐
              │                         │
           SQLite                    Firebase
              │                         │
         Local Data                Cloud Data
              │                         │
         Sync Queue  ───────────→ Firestore
```

---

# 9. Single Application Model

There must be one Android application.

Do not create separate APKs for:

- Customer
- Staff
- Admin

The role determines the available navigation and features.

---

# 10. Navigation

Use:

```text
Expo Router
```

Recommended conceptual structure:

```text
app/
├── _layout.tsx
├── index.tsx
│
├── (auth)/
│   ├── login.tsx
│   ├── register.tsx
│   ├── verify-email.tsx
│   ├── complete-profile.tsx
│   └── forgot-password.tsx
│
├── (customer)/
│   ├── _layout.tsx
│   ├── home.tsx
│   ├── orders.tsx
│   └── profile.tsx
│
├── (staff)/
│   ├── _layout.tsx
│   ├── home.tsx
│   ├── orders.tsx
│   └── profile.tsx
│
└── (admin)/
    ├── _layout.tsx
    ├── home.tsx
    ├── orders.tsx
    ├── people.tsx
    ├── reports.tsx
    └── more.tsx
```

Actual folder organization may be adjusted during implementation, but role separation must remain clear.

---

# 11. Recommended Source Structure

```text
src/
├── components/
├── constants/
├── database/
│   ├── migrations/
│   ├── repositories/
│   └── sqlite.ts
│
├── firebase/
│   ├── auth.ts
│   ├── firestore.ts
│   └── messaging.ts
│
├── hooks/
├── services/
│   ├── auth/
│   ├── orders/
│   ├── payments/
│   ├── reports/
│   ├── invoice/
│   └── sync/
│
├── store/
├── types/
├── utils/
└── validation/
```

---

# 12. TypeScript

TypeScript must be used throughout the application.

Avoid:

```text
any
```

unless there is a genuine technical reason.

Prefer:

```text
type
interface
union types
generics
```

All important domain entities must have strongly typed models.

---

# 13. Domain Roles

Define roles as a strict union/enum:

```text
CUSTOMER
STAFF
ADMIN
```

Do not use arbitrary role strings throughout the application.

---

# 14. Order Status

Order statuses must be centrally defined.

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

Do not duplicate status strings across screens.

---

# 15. Payment Status

```text
PENDING
PARTIALLY_PAID
PAID
```

---

# 16. Payment Method

```text
CASH
UPI
ONLINE
```

---

# 17. Customer Type

```text
PERSONAL
BUSINESS
```

---

# 18. Record Status

Master records should use:

```text
ACTIVE
INACTIVE
```

where applicable.

---

# 19. Authentication Architecture

Use:

```text
Firebase Authentication
```

for passwords and authentication sessions.

Passwords must never be stored manually in Firestore or SQLite.

---

# 20. Customer Registration

Technical flow:

```text id="v4c1se"
Register Screen
 ↓
Validate Email + Password
 ↓
Firebase createUser
 ↓
Send Verification Email
 ↓
Create initial user/profile record
 ↓
User verifies email
 ↓
Complete Profile
 ↓
profileCompleted = true
```

---

# 21. Email Verification

The app must check Firebase Authentication's email verification state.

A user must not receive normal Customer application access before email verification.

---

# 22. Phone Number

No phone OTP system in V1.

Phone is profile/contact data.

Do not implement:

- SMS OTP
- Phone authentication
- Phone verification flags pretending the phone was verified

unless this requirement is explicitly added in a future version.

---

# 23. Login

Login:

```text
Email
Password
```

Technical sequence:

```text id="t7v1xg"
Firebase signIn
 ↓
Check emailVerified
 ↓
Fetch trusted user profile
 ↓
Validate account status
 ↓
Resolve role
 ↓
Resolve businessId
 ↓
Route to panel
```

---

# 24. Account Status

Inactive users must not be allowed to operate the application.

For Staff:

```text
ACTIVE → Login allowed
INACTIVE → Login blocked
```

Customer account status may similarly be checked.

---

# 25. Role Resolution

The client may read the user's role for UI routing.

However:

> UI role checks are not security.

All privileged operations must also be protected by Firebase Security Rules and/or Cloud Functions.

---

# 26. Admin Creation

Admin must not be created through public registration.

Initial Admin creation must be controlled.

No Admin creation button should exist in:

```text
Register
Login
Customer Profile
Staff Panel
```

---

# 27. Staff Creation

Staff creation is a privileged operation.

Recommended architecture:

```text id="6t8s7x"
Admin App
 ↓
Callable Cloud Function
 ↓
Firebase Admin SDK
 ↓
Create Firebase Auth User
 ↓
Create Staff Profile
```

The Firebase Admin SDK must never be shipped inside the mobile application.

---

# 28. Temporary Staff Password

When Admin creates Staff:

```text
Temporary Password
```

is assigned securely.

On first login:

```text
forcePasswordChange = true
```

Staff must change the temporary password.

---

# 29. Password Storage

Never store:

```text
password
temporaryPassword
passwordHash
```

in normal Firestore user documents.

Firebase Authentication manages credentials.

---

# 30. Secure Local Authentication Data

The application must use secure platform storage for sensitive authentication/session-related information where needed.

Do not store sensitive secrets in plain AsyncStorage.

---

# 31. Business Context

Every business-owned record must contain:

```text
businessId
```

Even though V1 has one business.

This is required for future expansion.

---

# 32. Business Isolation

Every read/write operation must be scoped to the authenticated user's authorized `businessId`.

The client must never be able to arbitrarily change:

```text
businessId
```

---

# 33. SQLite as Local Source for UI

Normal screens should read operational data from SQLite.

Example:

```text id="w7wz8k"
Orders Screen
 ↓
SQLite query
 ↓
Immediate UI
```

Not:

```text
Orders Screen
 ↓
Wait for Firebase
 ↓
Display
```

---

# 34. Firebase Synchronization

Firebase remains the cloud record.

SQLite remains the local operational copy.

Synchronization bridges the two.

```text id="n0whqk"
SQLite
   ↕
Sync Engine
   ↕
Firestore
```

---

# 35. Offline-First Rule

Every supported offline operation must follow:

```text
Validate
 ↓
SQLite Transaction
 ↓
UI Update
 ↓
Sync Queue
```

The operation must not wait for Firebase before updating the local UI.

---

# 36. SQLite Transactions

Operations affecting multiple tables must use SQLite transactions.

Example order creation:

```text id="q9w4pi"
Create Order
+
Create Order Items
+
Create Sync Queue Record
```

These should be committed atomically.

If one part fails:

```text
Rollback
```

---

# 37. Order Creation Offline

Example:

```text id="v7z8gr"
Staff creates ORD1052
 ↓
Validate
 ↓
SQLite transaction
 ↓
orders inserted
 ↓
order_items inserted
 ↓
sync_queue inserted
 ↓
Commit
 ↓
UI displays order
```

---

# 38. Sync Queue

Each pending cloud operation must have a queue record.

Required conceptual fields:

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

---

# 39. Sync Operations

Supported operations:

```text
CREATE
UPDATE
DELETE
```

However, normal business records should preferably use:

```text
UPDATE → status/inactive
```

instead of physical deletion.

---

# 40. Sync Queue Status

Recommended:

```text
PENDING
SYNCING
SYNCED
FAILED
```

---

# 41. Retry Strategy

Failed operations must retry.

Recommended approach:

```text
Attempt 1
 ↓
Failure
 ↓
Backoff
 ↓
Attempt 2
 ↓
Failure
 ↓
Backoff
 ↓
Retry
```

Use bounded exponential backoff with jitter.

Do not create an infinite tight retry loop.

---

# 42. Duplicate Sync Prevention

Every cloud-write operation must be idempotent where possible.

Use client-generated stable identifiers.

Example:

```text
syncId = unique operation ID
orderId = stable order ID
```

Retrying the same operation must not create another order.

---

# 43. Network Detection

The application should detect:

```text
Online
Offline
```

and use network state to trigger synchronization.

Network availability does not guarantee Firebase availability.

Therefore Firebase operation failures must still be handled.

---

# 44. Sync Trigger Points

Synchronization should be attempted:

1. On app startup
2. When authenticated session becomes active
3. When network becomes available
4. After a successful local write where appropriate
5. On manual retry
6. Periodically while the application is active

Background execution should not be assumed to run indefinitely on Android.

---

# 45. Background Sync

V1 should use Expo-compatible mechanisms first.

Do not make the product depend on guaranteed continuous background execution.

If native Android WorkManager is introduced later, it must be added only when the required background behavior is technically guaranteed.

---

# 46. Sync Conflict Handling

Potential conflict:

```text
Device A updates order
Device B updates same order
```

The system must track:

```text
updatedAt
updatedBy
```

and where required:

```text
version
```

Important financial/status operations must not silently overwrite each other.

---

# 47. Payment Conflict Protection

Payment records should be append-oriented.

Instead of overwriting the entire payment history:

```text
Create Payment Record
```

for each payment.

This preserves the history of individual payments.

---

# 48. Status Conflict Protection

Status changes should create status-history records.

Example:

```text
PROCESSING
 ↓
READY
```

should create a history record.

Important invalid transitions should be rejected.

---

# 49. Order Status Validation

Status transitions must be centrally validated.

Example:

```text
NEW
→ PICKUP_PENDING
```

valid.

But arbitrary:

```text
NEW
→ DELIVERED
```

should not normally be allowed.

---

# 50. Server-Side Validation

Critical business rules must not depend only on the mobile client.

At minimum, backend protection is required for:

- Role
- businessId
- Pricing
- Payment records
- GST-sensitive data
- Order ownership
- Status changes
- Staff creation
- Admin-only operations

---

# 51. Pricing Architecture

Price records are maintained by Admin.

Conceptually:

```text
Item
+
Service
=
Price
```

The client caches current active prices locally.

---

# 52. Price Snapshot

When order is created:

```text
current price
 ↓
priceAtOrderTime
```

The order item stores the historical price.

Historical orders must never calculate their amount using current price records.

---

# 53. Order Amount Calculation

For each order item:

```text
subtotal = quantity × priceAtOrderTime
```

Order subtotal:

```text
sum(all item subtotals)
```

If GST applies:

```text
gstAmount = subtotal × gstRate / 100
```

Grand total:

```text
grandTotal = subtotal + gstAmount
```

Without GST:

```text
gstAmount = 0
grandTotal = subtotal
```

All monetary calculations must use safe decimal/integer-based handling rather than floating-point assumptions that can introduce rounding errors.

---

# 54. GST Snapshot

Each order stores:

```text
gstApplied
gstRate
gstAmount
```

Changing Admin's default GST rate must not modify historical orders.

---

# 55. Estimated vs Final Amount

Customer-side order creation can calculate:

```text
estimatedAmount
```

Staff/Admin may verify:

```text
finalAmount
```

Historical order values must be preserved.

The UI must clearly distinguish:

```text
Estimated Amount
Final Amount
```

when both exist.

---

# 56. Payment Calculation

```text
dueAmount = finalAmount - paidAmount
```

The system must prevent:

```text
paidAmount < 0
```

and should reject or safely handle payments exceeding the amount due according to the final payment policy.

Payment status:

```text
paidAmount = 0
→ PENDING

0 < paidAmount < finalAmount
→ PARTIALLY_PAID

paidAmount >= finalAmount
→ PAID
```

The exact overpayment policy must be enforced consistently.

---

# 57. Order and Payment Independence

Do not derive order status from payment status.

Do not derive payment status from delivery status.

They are separate state machines.

---

# 58. Firestore Structure

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

Complete field definitions belong in `BACKEND-SCHEMA.md`.

---

# 59. Firebase Authentication + Firestore Relationship

Firebase Auth UID:

```text
auth.uid
```

must correspond to the application user profile.

Conceptually:

```text
Firebase Auth
      │
      ↓
users/{uid}
      │
      ├── role
      ├── businessId
      └── status
```

---

# 60. Customer Data Access

Customer queries must be restricted to:

```text
customerId == authenticated customer
```

The client must not be trusted to simply pass an arbitrary customer ID.

---

# 61. Staff Data Access

Staff access must be limited to their authorized business.

Staff should have operational access, not administrative configuration access.

---

# 62. Admin Data Access

Admin can access all records within the authorized business.

---

# 63. Firebase Security Rules

Firestore Security Rules must enforce:

```text
Authentication
+
Role
+
Business ID
+
Ownership
+
Field restrictions
```

Do not rely on hidden buttons or navigation guards as security.

---

# 64. Protected Fields

Normal users must not be allowed to arbitrarily modify:

```text
role
businessId
createdBy
createdAt
payment history
status history
invoice number
historical price
historical GST
```

unless the operation is explicitly authorized.

---

# 65. Cloud Functions

Cloud Functions should be used for privileged operations that should not be trusted to a client.

Examples:

- Staff account creation
- Sensitive server-side validation
- Privileged administrative operations
- Notification dispatch where appropriate
- Idempotent server-side workflows where required

---

# 66. Firebase Admin SDK

Firebase Admin SDK must exist only in trusted server-side environments.

Never bundle:

```text
service account JSON
private key
Admin SDK credentials
```

inside the Android application.

---

# 67. Notifications

Use:

```text
Firebase Cloud Messaging
```

where notifications are enabled.

Store device notification token information securely.

A notification failure must never roll back a successful order.

---

# 68. Notification Architecture

Recommended:

```text
Order Event
 ↓
Backend / Notification Service
 ↓
FCM
 ↓
Customer / Staff Device
```

---

# 69. Notification Types

Customer:

```text
ORDER_CREATED
PICKUP_COMPLETED
PROCESSING
READY
OUT_FOR_DELIVERY
DELIVERED
```

Staff:

```text
NEW_CUSTOMER_ORDER
PICKUP_REQUEST
```

---

# 70. Notification Idempotency

The same event must not generate uncontrolled duplicate notifications.

Notification events should have identifiable event IDs where necessary.

---

# 71. Invoice Generation

Invoice generation must work from the locally available order data.

Recommended flow:

```text
Order
 ↓
Load complete local order
 ↓
Validate invoice data
 ↓
Build A4 document
 ↓
Generate PDF
 ↓
Save locally
 ↓
Open / Share
```

---

# 72. Invoice Data Source

Invoice must use:

```text
Order snapshot
+
Order items
+
Payment records
+
Business settings
```

Historical order values take precedence over current pricing.

---

# 73. Invoice Number

Invoice number must be generated according to the business invoice configuration.

Order ID and invoice number must remain separate.

Example:

```text
ORD1025
INV1025
```

---

# 74. PDF Requirements

PDF must:

- Be A4
- Support ₹
- Support long text
- Wrap addresses
- Handle multiple items
- Calculate totals correctly
- Display GST correctly
- Display payment information
- Avoid overlap
- Avoid clipping

---

# 75. Excel Report Generation

Reports are generated from local SQLite data.

Flow:

```text
Report Filter
 ↓
SQLite Query
 ↓
Aggregate Data
 ↓
Generate XLSX
 ↓
Save Locally
 ↓
Open / Share
```

---

# 76. Report Date Filtering

Weekly:

```text
Selected Month
Selected Year
From Date
To Date
```

The date range must remain inside the selected month unless the product requirements are changed.

Monthly:

```text
Month + Year
```

Yearly:

```text
Year
```

---

# 77. Report Historical Accuracy

Reports must use stored order snapshots.

Never do:

```text
Old Order
+
Current Price
=
Report Amount
```

Correct:

```text
Old Order
+
Stored Historical Values
=
Report Amount
```

---

# 78. Local File Handling

Generated:

```text
PDF
XLSX
```

files are local files in V1.

The application should provide:

```text
Open
Share
```

where supported by the Android environment.

---

# 79. State Management

Application state should be separated into:

### Server/cloud-related state

Firebase synchronization state.

### Local domain state

SQLite-backed application data.

### UI state

Examples:

- Modal open
- Selected filter
- Search text
- Loading state
- Form state

Do not store the entire database in React state.

---

# 80. Repository Pattern

Screens should communicate with repositories/use cases instead of directly performing raw Firestore/SQLite logic everywhere.

Example:

```text id="6hfjq5"
OrderScreen
 ↓
useOrders()
 ↓
OrderService
 ↓
OrderRepository
 ↓
SQLite
```

Sync service separately handles Firebase synchronization.

---

# 81. Validation

Validation must exist at multiple levels:

```text
UI Validation
 ↓
Domain Validation
 ↓
Backend Security/Validation
```

Client validation improves UX.

Server validation provides security.

---

# 82. Customer Registration Validation

Validate:

- Email format
- Password requirements
- Password confirmation
- Required fields

Email verification must happen before normal customer access.

---

# 83. Profile Validation

Validate:

- Name
- Mobile Number
- Customer Type
- Address
- PIN Code
- Business Name when Business is selected
- Business Type when Business is selected

---

# 84. Order Validation

Validate:

- Customer
- At least one item
- Valid item
- Valid service
- Quantity > 0
- Valid price
- GST selection
- Pickup details where required

---

# 85. Payment Validation

Validate:

- Amount > 0
- Valid payment method
- Valid order
- Authorized user
- Payment amount does not violate business rules

---

# 86. Master Data Validation

Admin-created:

- Item
- Service
- Price

must have required fields and valid values.

Duplicate active item/service names should be prevented or handled consistently.

---

# 87. Search Architecture

Search should be performed against SQLite for normal offline functionality.

Searchable order fields:

```text
orderId
invoiceNumber
customerNameAtOrder
customerPhoneAtOrder
```

Customer search:

```text
name
mobile
email
businessName
```

---

# 88. Search Performance

For frequently searched fields, appropriate SQLite indexes should be created.

Exact schema/index definitions belong in:

```text
BACKEND-SCHEMA.md
```

---

# 89. Filtering

Order filtering should support:

```text
Order Status
Payment Status
```

and combinations.

Example:

```text
DELIVERED
+
PENDING
```

---

# 90. Pagination

For large local datasets, SQLite queries should use pagination/limited result loading rather than loading thousands of records into memory at once.

Cloud queries should similarly avoid unnecessary full-collection reads.

---

# 91. Performance Requirements

The app should feel responsive on low-end Android devices.

Target principles:

- Fast startup
- Minimal unnecessary Firebase reads
- SQLite-first UI
- Lazy loading
- Paginated lists
- Memoized expensive UI
- Avoid unnecessary re-renders
- Avoid large in-memory collections

---

# 92. Offline Startup

If the user has previously synchronized data, the application should display available local data even when offline.

Example:

```text
No Internet
 ↓
Open App
 ↓
SQLite
 ↓
Existing Orders Visible
```

---

# 93. Empty States

Every major list needs an empty state.

Examples:

> No orders found.

> No customers found.

> No staff members found.

> No reports available for this period.

---

# 94. Loading States

Use clear loading states for:

- Login
- Registration
- Profile save
- Sync
- Order creation
- Payment recording
- Report generation
- Invoice generation

Avoid blank screens.

---

# 95. Error Handling

Never expose raw Firebase/SQLite exceptions to normal users.

Convert technical failures into user-friendly messages.

Example:

Technical error:

```text
FirebaseError: permission-denied
```

User message:

> You do not have permission to perform this action.

---

# 96. Offline Error

When network is unavailable:

> Internet connection is unavailable. Your data has been saved on this device and will sync automatically.

---

# 97. Sync Error

When synchronization fails:

> Some data is waiting to sync. Please keep the app open when internet is available.

A retry action may be provided.

---

# 98. Logging

Development builds should provide useful diagnostic logs.

Production builds must not expose:

- Passwords
- Authentication tokens
- Private credentials
- Sensitive customer data unnecessarily

Use structured logging where practical.

---

# 99. Secrets and Configuration

Use environment/configuration mechanisms for:

- Firebase configuration
- Build configuration
- Environment-specific values

Never commit private server credentials.

Client-side Firebase configuration values are not equivalent to Admin credentials, but Firestore Security Rules must still provide the actual authorization boundary.

---

# 100. Firebase Environments

Where practical, maintain separate environments for:

```text
Development
Production
```

Avoid accidentally testing destructive operations against production data.

---

# 101. Database Migration

SQLite schema must use migrations.

Never assume a user's existing database is always empty.

Example:

```text
Migration 1
Migration 2
Migration 3
```

Application startup must ensure required migrations have been applied.

---

# 102. Local Database Version

Maintain a database schema version.

When application updates introduce schema changes:

```text
Existing DB
 ↓
Migration
 ↓
New Schema
```

User data must be preserved.

---

# 103. Firestore Schema Evolution

New fields should be introduced backward-compatibly where practical.

The application should tolerate missing optional fields from older records.

---

# 104. Time Handling

Store timestamps consistently.

Use Firebase/server timestamps where appropriate for cloud records.

Local SQLite timestamps must use a consistent representation.

Display times in the user's/business local timezone.

---

# 105. IDs

Use stable unique IDs.

Recommended conceptual IDs:

```text
userId
businessId
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

Human-readable order/invoice numbers can be separate from internal IDs.

---

# 106. Order ID Generation

Order IDs must be unique even when multiple devices create orders offline.

Therefore V1 must not depend solely on:

```text
local incrementing integer
```

for global uniqueness.

Use a collision-resistant identifier strategy.

The displayed human-readable order number can be generated separately if required.

---

# 107. Invoice Number Generation

Invoice numbering must avoid duplicates across devices.

If invoice numbers are sequential by business policy, the implementation must use a controlled server-side/transaction-safe strategy.

Offline invoice generation must not silently create duplicate official invoice numbers.

If an official invoice number cannot safely be allocated offline, the application must distinguish a local draft/temporary document from a finalized invoice.

---

# 108. Financial Data Integrity

Financial information requires stronger consistency than ordinary UI state.

Examples:

- Final amount
- Paid amount
- Due amount
- GST amount
- Payment records
- Invoice number

These values must not be casually overwritten by stale clients.

---

# 109. Auditability

Important operations should retain:

```text
createdBy
createdAt
updatedBy
updatedAt
```

Status changes:

```text
changedBy
changedAt
```

Payments:

```text
recordedBy
paymentDate
```

---

# 110. No Hardcoded Business Data

Do not hardcode:

```text
Business Name
Phone
GSTIN
Prices
Items
Services
GST Rate
Invoice Prefix
```

These belong to business settings/master data.

UI may contain fallback text, but business configuration must come from stored settings.

---

# 111. No Hardcoded Prices

Incorrect:

```text
Shirt = ₹30
Pant = ₹40
```

inside application code.

Correct:

```text
SQLite cached price
   ↑
Firebase master price
```

---

# 112. No Hardcoded GST

Incorrect:

```text
GST = 18%
```

as a business rule.

Correct:

```text
Admin default GST rate
+
Per-order GST selection
+
Stored order GST snapshot
```

---

# 113. Customer Profile Change

If a customer changes:

```text
Business
→
Personal
```

the current business fields should be cleared after confirmation.

Historical orders remain unchanged because order snapshots are stored separately.

---

# 114. Customer Data Change

If customer changes:

```text
Name
Mobile
Address
Business Name
```

future orders use the new profile values.

Old orders retain their historical snapshots.

---

# 115. Staff Deactivation

When Staff becomes inactive:

```text
status = INACTIVE
```

Historical records remain linked to that Staff account.

Do not remove historical `createdBy`, `changedBy`, or `recordedBy` references.

---

# 116. Item/Service Deactivation

If an item/service is referenced by historical orders:

```text
ACTIVE
→
INACTIVE
```

rather than hard deletion.

Old orders continue displaying the historical item/service name.

---

# 117. Offline Master Data

Items, services, prices and business settings should be cached locally.

If offline:

```text
Use latest successfully synchronized local master data.
```

The UI should not pretend the cloud has been queried when offline.

---

# 118. Offline Customer Order

A customer should be able to create an order offline if the required local master data is available.

The application should clearly communicate that the order is saved locally and awaiting synchronization.

---

# 119. Offline Staff Operation

Staff should be able to perform supported operational work offline.

Examples:

- View cached customers
- View cached orders
- Create supported orders
- Update supported statuses
- Record supported payments
- Generate invoice from local data

---

# 120. Offline Admin Operation

Admin can perform supported locally available management operations offline where technically safe.

Operations that require authoritative cloud uniqueness/transactional allocation may need to remain pending or require connectivity.

The UI must clearly explain such limitations rather than silently pretending the operation is finalized.

---

# 121. Security Architecture

Security layers:

```text id="j9n7h3"
Firebase Authentication
        ↓
Firestore Security Rules
        ↓
Cloud Functions for privileged operations
        ↓
Application role checks
        ↓
SQLite/local protection
```

No single client-side layer should be treated as sufficient security.

---

# 122. Firestore Rules Principles

Rules must verify:

1. User is authenticated
2. User profile exists
3. User has correct role
4. User belongs to correct business
5. Customer owns the requested record where applicable
6. Protected fields cannot be modified arbitrarily

---

# 123. Customer Order Security

Customer should only be able to:

```text
Create own order
Read own orders
```

Customer must not be able to modify:

```text
finalAmount
paidAmount
payment history
status history
invoice number
businessId
customerId
```

without authorized server logic.

---

# 124. Staff Security

Staff can operate within their business.

Staff cannot modify Admin-only configuration.

---

# 125. Admin Security

Admin has full authorized business-level access.

Admin operations must still be validated server-side where necessary.

---

# 126. Sensitive Local Data

Avoid storing unnecessary sensitive information locally.

Local database should contain only data required for offline operation.

---

# 127. App Lock / Device Security

A separate PIN/biometric app lock is not required for V1 unless explicitly added later.

The application should rely on Android device security and authenticated sessions.

---

# 128. API Architecture

The Android app will primarily communicate with:

```text
Firebase Auth
Firestore
Cloud Functions
FCM
```

There is no separate custom REST backend required for V1.

---

# 129. Cloud Function API Style

Callable HTTPS Functions may be used for privileged application actions.

Examples:

```text
createStaff
```

Additional functions should be introduced only where they provide a real security/consistency benefit.

Do not create unnecessary APIs for every simple Firestore read/write.

---

# 130. Repository Interfaces

Conceptually:

```text
AuthRepository
UserRepository
CustomerRepository
StaffRepository
ItemRepository
ServiceRepository
PriceRepository
OrderRepository
PaymentRepository
ReportRepository
BusinessSettingsRepository
SyncRepository
```

Implementations can use SQLite and Firebase as appropriate.

---

# 131. Order Repository

Order repository responsibilities include:

- Create order locally
- Read order locally
- Search
- Filter
- Update supported order data
- Queue synchronization
- Read status history
- Read payments

It should not contain UI rendering logic.

---

# 132. Sync Service

Sync service responsibilities:

```text
Read pending queue
 ↓
Validate operation
 ↓
Send cloud operation
 ↓
Handle success
 ↓
Mark synced
```

On failure:

```text
Increment retryCount
Save lastError
Keep operation pending/failed
```

---

# 133. Sync Ordering

Related operations should be synchronized in safe order.

Example:

```text
Create Customer
 ↓
Create Order
 ↓
Create Payment
```

An order referring to a locally created customer should not be uploaded before the customer record is available unless the backend strategy explicitly supports it.

---

# 134. Sync Dependencies

Sync queue should support dependencies where required.

Conceptual:

```text
Order depends on Customer
Payment depends on Order
Status History depends on Order
```

---

# 135. Local Referential Integrity

SQLite foreign keys should be used where appropriate.

Deleting a parent record should not accidentally destroy required historical records.

---

# 136. Search Index Strategy

Create SQLite indexes for frequently queried fields.

Likely candidates:

```text
orderId
invoiceNumber
customerId
customerPhoneAtOrder
orderStatus
paymentStatus
createdAt
```

Exact SQL belongs in `BACKEND-SCHEMA.md`.

---

# 137. Reporting Query Strategy

Reports should be generated using SQL aggregation where practical.

Avoid loading the entire lifetime database into JavaScript if SQL can perform the aggregation efficiently.

---

# 138. Report Data Consistency

A report generation operation should operate on a consistent local dataset snapshot as much as practical.

If sync is occurring simultaneously, the report should either:

- use a defined snapshot boundary, or
- clearly indicate that it represents currently synchronized/local data.

---

# 139. File Naming

Weekly:

```text
Laundry_Report_DD-MM-YYYY_to_DD-MM-YYYY.xlsx
```

Monthly:

```text
Laundry_Report_Month_YYYY.xlsx
```

Yearly:

```text
Laundry_Report_YYYY.xlsx
```

---

# 140. Internationalization

V1 can use a centralized Indian-English string dictionary.

Example:

```text
strings.en-IN.ts
```

Do not scatter user-visible text throughout complex business logic.

Future languages can be added later.

---

# 141. Accessibility

The application should provide:

- Readable font sizes
- Adequate contrast
- Large enough touch targets
- Clear labels
- Visible error states
- No color-only status indication

---

# 142. Form UX Technical Requirements

Forms must:

- Validate before submission
- Disable duplicate submission while saving
- Show field-level errors
- Preserve entered values when a non-fatal error occurs
- Hide passwords by default
- Use appropriate keyboard types

---

# 143. Duplicate Button Protection

When a user taps:

```text
Create Order
Pay
Save
Register
```

multiple times rapidly, the application must not create duplicate records.

Use:

```text
loading/submitting state
+
idempotent operation
```

---

# 144. App Restart During Operation

If the app is closed after a successful SQLite transaction but before Firebase synchronization:

```text
Sync Queue remains
```

On next launch:

```text
Sync Engine resumes
```

No data should be lost.

---

# 145. Crash During Transaction

Critical local operations must use SQLite transactions.

If the process crashes during a transaction:

```text
Transaction rollback
```

must prevent partial order data from appearing as a valid completed operation.

---

# 146. App Update

App updates must preserve:

- SQLite data
- Authentication session where supported
- Pending sync queue
- Historical orders
- Payment records

Database migrations must be backward-safe.

---

# 147. Testing Strategy

Testing must cover four major layers:

```text
Unit Tests
Integration Tests
Security Tests
End-to-End Tests
```

---

# 148. Unit Testing

Test:

- Amount calculations
- GST calculation
- Payment status
- Due amount
- Status transition validation
- Customer type logic
- Business field clearing
- Date range filtering
- Report calculations
- ID generation

---

# 149. SQLite Testing

Test:

- Insert
- Update
- Query
- Search
- Filters
- Transactions
- Migration
- Foreign-key behavior
- Sync queue

---

# 150. Sync Testing

Must test:

### Online

```text
Create → Sync → Synced
```

### Offline

```text
Create → Queue → Reconnect → Sync
```

### Failure

```text
Create → Sync Failure → Retry
```

### Duplicate retry

```text
Same operation retried
→
No duplicate order
```

### App restart

```text
Offline operation
→
Close App
→
Open App
→
Queue remains
```

---

# 151. Security Testing

Test that:

- Customer cannot read another customer
- Customer cannot access Staff data
- Customer cannot access Admin data
- Staff cannot create Admin
- Staff cannot change prices
- Staff cannot change role
- Staff cannot access another business
- Client cannot change businessId
- Client cannot modify protected financial fields
- Unauthenticated users cannot access protected Firestore data

---

# 152. Role Testing

Test every role:

```text
CUSTOMER
STAFF
ADMIN
```

and verify both:

```text
UI access
+
Backend access
```

---

# 153. Order Testing

Test:

```text
New Order
Pickup
Picked Up
Processing
Ready
Out for Delivery
Delivered
Cancelled
```

including invalid transitions.

---

# 154. Payment Testing

Test:

```text
₹0
Partial Payment
Full Payment
Multiple Payments
Cash
UPI
Online
Pending
Paid
Partially Paid
```

---

# 155. GST Testing

Test:

```text
With GST
Without GST
18%
Other configured rate
GST rate change
Historical order
```

Example:

```text
₹500
18%
→ ₹590
```

---

# 156. Historical Data Testing

Change:

- Customer name
- Customer phone
- Business name
- Address
- Price
- GST rate

Then verify old orders/invoices remain unchanged.

---

# 157. Invoice Testing

Test:

- ₹ symbol
- Long customer name
- Long business name
- Long address
- Many items
- GST
- No GST
- Partial payment
- Full payment
- Pending payment
- A4 output

---

# 158. Report Testing

Test:

- Weekly
- Monthly
- Yearly
- Date ranges
- Cancelled orders
- Delivered orders
- Paid orders
- Unpaid orders
- GST
- Multiple payments
- Historical prices

---

# 159. Performance Testing

Test on low-end Android hardware where practical.

Minimum product expectation:

- No unnecessary freezing
- Lists remain responsive
- SQLite operations remain fast
- Large order lists are paginated
- Report generation does not block UI unnecessarily

---

# 160. Network Testing

Test:

```text
Fast Internet
Slow Internet
No Internet
Internet drops during operation
Internet returns
Firebase temporarily fails
```

---

# 161. Android Lifecycle Testing

Test:

```text
App Open
App Background
App Resume
App Killed
App Restart
Device Reboot
```

especially for:

- Pending sync
- Authentication
- Local database
- Notifications

---

# 162. Development Rules

Developers must:

1. Follow PRD
2. Follow TRD
3. Keep business logic out of UI where possible
4. Use TypeScript
5. Avoid hardcoded prices
6. Avoid hardcoded GST
7. Use SQLite-first data flow
8. Protect Firebase operations
9. Use transactions for critical local operations
10. Preserve historical data

---

# 163. Things Developers Must NOT Do

Do not:

```text
Store passwords in Firestore
Store Firebase Admin credentials in app
Hardcode prices
Hardcode GST
Trust client role checks as security
Depend entirely on Firebase for offline screens
Delete historical orders casually
Recalculate old orders using current prices
Recalculate old orders using current GST
Create duplicate orders during retry
Show raw Firebase errors
```

---

# 164. Build Configuration

Separate:

```text
Development
Production
```

configuration.

Production builds must not point accidentally to development data and vice versa.

---

# 165. Release Checklist

Before production release:

### Authentication

- Registration works
- Email verification works
- Login works
- Password reset works
- Staff login works
- Admin login works

### Customer

- Profile works
- Personal/Business works
- Orders work

### Staff

- Manual orders work
- Status updates work
- Payments work

### Admin

- Staff management works
- Customer management works
- Pricing works
- GST settings work
- Reports work

### Offline

- SQLite works
- Queue works
- Retry works
- Duplicate protection works

### Documents

- PDF works
- XLSX works

### Security

- Firestore rules tested
- Role access tested
- Business isolation tested

---

# 166. Production Observability

Production builds should provide enough diagnostics to investigate:

- Sync failures
- Crashes
- Authentication problems
- Report failures
- Invoice failures

But logs must not expose sensitive credentials.

---

# 167. Future Web Compatibility

The backend must not contain Android-specific assumptions.

For example, Firestore records should represent business entities rather than screen-specific structures.

Future:

```text id="8qmy4d"
React/Next.js Admin Web
          ↓
Same Firebase Backend
          ↑
Android App
```

---

# 168. Future Expansion Compatibility

The architecture should leave room for:

- Multiple businesses
- Multiple branches
- Delivery module
- Online payments
- WhatsApp
- SMS
- QR/barcode
- Logo upload
- Advanced reports

However, these must NOT complicate V1 unnecessarily.

---

# 169. V1 Technical Exclusions

Do not implement:

```text
Custom REST Backend
Super Admin
Multi-business UI
Multi-branch UI
Inventory System
Payroll System
Expense Accounting
AI
GPS Routing
Barcode System
QR Garment Tracking
POS Hardware
Printer Integration
Separate Delivery Application
```

unless explicitly moved into scope.

---

# 170. Recommended Implementation Order

Implementation must follow this sequence:

```text
1. Project Setup
        ↓
2. TypeScript Domain Models
        ↓
3. SQLite Database + Migrations
        ↓
4. Authentication
        ↓
5. User/Role System
        ↓
6. Firebase Integration
        ↓
7. Repository Layer
        ↓
8. Offline Sync Engine
        ↓
9. Master Data
   Items/Services/Prices
        ↓
10. Customer Module
        ↓
11. Order Engine
        ↓
12. Payments
        ↓
13. GST
        ↓
14. Staff Module
        ↓
15. Admin Module
        ↓
16. Invoice Generator
        ↓
17. Report Generator
        ↓
18. Notifications
        ↓
19. Security Hardening
        ↓
20. Testing
        ↓
21. Production Build
```

---

# 171. Critical Implementation Priority

The following are higher priority than visual polish:

```text
Data Integrity
Offline Reliability
Authentication
Authorization
Sync Correctness
Historical Accuracy
Payment Accuracy
GST Accuracy
```

Do not begin by spending excessive time on animations or 3D effects.

---

# 172. Final Technical Architecture

```text
                         ┌─────────────────────┐
                         │   Android App       │
                         │ Expo + React Native  │
                         │    TypeScript        │
                         └──────────┬──────────┘
                                    │
                         ┌──────────▼──────────┐
                         │   Presentation      │
                         │  Expo Router/UI     │
                         └──────────┬──────────┘
                                    │
                         ┌──────────▼──────────┐
                         │ Application Layer   │
                         │ Hooks / Use Cases   │
                         └──────────┬──────────┘
                                    │
                         ┌──────────▼──────────┐
                         │ Repository Layer    │
                         └──────────┬──────────┘
                                    │
                  ┌─────────────────┴─────────────────┐
                  │                                   │
         ┌────────▼────────┐                 ┌────────▼────────┐
         │     SQLite      │                 │     Firebase    │
         │ Local Database  │                 │ Cloud Backend   │
         └────────┬────────┘                 └────────┬────────┘
                  │                                   │
         ┌────────▼────────┐                 ┌────────▼────────┐
         │   Sync Queue    │◄───────────────►│   Firestore     │
         └─────────────────┘                 └─────────────────┘
                                                     │
                                      ┌──────────────┼──────────────┐
                                      │              │              │
                                     Auth       Cloud Functions     FCM
                                      │              │              │
                                      └──────────────┴──────────────┘
```

---

# 173. Technical Source-of-Truth Rule

The technical implementation must follow:

```text
PRD.md
   ↓
TRD.md
   ↓
UI-UX.md
   ↓
BACKEND-SCHEMA.md
```

`PRD.md` defines **what the product must do**.

`TRD.md` defines **how the application should technically implement it**.

`UI-UX.md` will define **exact screens, layouts, components, navigation and user interactions**.

`BACKEND-SCHEMA.md` will define **exact SQLite tables, Firestore collections/documents, fields, types, relationships, indexes and security-rule requirements**.

If implementation conflicts with the PRD, the conflict must be resolved before coding.