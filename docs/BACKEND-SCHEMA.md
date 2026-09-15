# BACKEND-SCHEMA.md

## Laundry Management App — Backend & Database Schema

**Version:** 1.0  
**Status:** Implementation Ready  
**Database:** Firebase Cloud Firestore + Local SQLite  
**Authentication:** Firebase Authentication  
**Backend:** Firebase Cloud Functions / Firebase Admin SDK  
**Client:** Expo + React Native + TypeScript  
**Architecture:** Offline-First + Cloud Sync  
**Business Model:** Single Business in V1, `businessId` retained for future expansion

---

# 1. Purpose

This document defines the exact backend and database structure required for the Laundry Management App.

It covers:

- Firebase Authentication relationship
- Firestore collections
- Firestore documents
- Field names
- Data types
- Required/optional fields
- Default values
- Enums
- Relationships
- IDs
- Timestamps
- Historical snapshots
- Orders
- Order items
- Payments
- Status history
- Customers
- Staff
- Items
- Services
- Prices
- Business settings
- Notifications
- Local SQLite database
- Sync queue
- Idempotency
- Conflict handling
- Firestore indexes
- Security requirements
- Data integrity
- Soft deletion/deactivation
- Migration/versioning

---

# 2. Core Architecture

```text
                    ┌──────────────────────┐
                    │   React Native App   │
                    └──────────┬───────────┘
                               │
                    Local First Operation
                               │
                               ▼
                    ┌──────────────────────┐
                    │    SQLite Database   │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │     Sync Queue       │
                    └──────────┬───────────┘
                               │
                         Internet Available
                               │
                               ▼
                    ┌──────────────────────┐
                    │     Firebase        │
                    │ Authentication      │
                    │ Firestore           │
                    │ Cloud Functions     │
                    │ FCM                 │
                    └──────────────────────┘
```

## 2.1 Source of Truth

For operational offline usage:

```text
SQLite = local operational database/cache
Firestore = long-term cloud business record
```

The application must never depend exclusively on network connectivity for normal operational actions.

---

# 3. Global Data Rules

## 3.1 ID Format

All application entities must have stable unique IDs.

Recommended:

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
statusHistoryId
syncId
notificationTokenId
```

IDs may be generated using UUIDs or Firebase-compatible unique IDs.

Client-created IDs must be unique enough to prevent duplicate records during offline synchronization.

---

# 3.2 Timestamps

All cloud timestamps should use Firebase server timestamps wherever possible.

Common fields:

```text
createdAt
updatedAt
```

Type:

```text
Firestore Timestamp
SQLite INTEGER
```

SQLite should store timestamps as Unix milliseconds.

---

# 3.3 Soft Deletion / Deactivation

Historical records must not normally be hard-deleted.

Master data should use:

```text
status = ACTIVE
status = INACTIVE
```

Examples:

- Staff → ACTIVE / INACTIVE
- Customer → ACTIVE / INACTIVE
- Item → ACTIVE / INACTIVE
- Service → ACTIVE / INACTIVE
- Price → ACTIVE / INACTIVE

Orders and payments should never be casually deleted.

---

# 3.4 Business Isolation

Every business-owned document must contain:

```text
businessId
```

Even though V1 supports only one business.

This allows future expansion without redesigning the complete database.

---

# 4. Firebase Authentication

Firebase Authentication is responsible for:

- email/password authentication
- email verification
- password reset
- secure password storage
- authentication sessions

Passwords must NEVER be stored in Firestore.

---

# 5. Firestore Structure

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

businesses/{businessId}/orders/{orderId}/statusHistory/{statusHistoryId}

businesses/{businessId}/settings/general

businesses/{businessId}/notificationTokens/{tokenId}
```

---

# 6. Users Collection

## Path

```text
users/{userId}
```

This document maps a Firebase Authentication user to the application role.

## Schema

| Field | Type | Required | Default |
|---|---|---:|---|
| userId | string | Yes | Auth UID |
| name | string | Yes | — |
| email | string | Yes | Auth email |
| phone | string/null | No | null |
| role | enum | Yes | — |
| businessId | string | Yes for business users | — |
| status | enum | Yes | ACTIVE |
| profileCompleted | boolean | Yes | false |
| forcePasswordChange | boolean | Yes | false |
| createdAt | timestamp | Yes | server timestamp |
| updatedAt | timestamp | Yes | server timestamp |

## Role Enum

```text
CUSTOMER
STAFF
ADMIN
```

## Status Enum

```text
ACTIVE
INACTIVE
```

## Important Rules

Client must not be allowed to freely modify:

```text
role
businessId
status
```

Role changes must be controlled by trusted backend logic.

---

# 7. Business Collection

## Path

```text
businesses/{businessId}
```

## Schema

| Field | Type | Required | Default |
|---|---|---:|---|
| businessId | string | Yes | Document ID |
| businessName | string | Yes | — |
| address | string | Yes | — |
| phone | string | Yes | — |
| email | string/null | No | null |
| GSTIN | string/null | No | null |
| defaultGSTRate | number | Yes | 0 |
| invoicePrefix | string | Yes | `INV` |
| orderPrefix | string | Yes | `ORD` |
| currency | string | Yes | `INR` |
| timezone | string | Yes | `Asia/Kolkata` |
| createdAt | timestamp | Yes | server timestamp |
| updatedAt | timestamp | Yes | server timestamp |

## Rules

Currency is:

```text
INR
```

Timezone:

```text
Asia/Kolkata
```

---

# 8. Business Settings

## Path

```text
businesses/{businessId}/settings/general
```

## Schema

```text
businessId
businessName
address
phone
email
GSTIN
defaultGSTRate
invoicePrefix
orderPrefix
invoiceThankYouMessage
updatedAt
updatedBy
```

## GST

GST is configured by Admin as a default business setting.

However, GST application is decided independently for each order.

Therefore:

```text
defaultGSTRate
```

must NOT be used to recalculate historical orders.

---

# 9. Staff Collection

## Path

```text
businesses/{businessId}/staff/{staffId}
```

## Schema

| Field | Type | Required |
|---|---|---:|
| staffId | string | Yes |
| userId | string | Yes |
| businessId | string | Yes |
| name | string | Yes |
| email | string | Yes |
| phone | string | Yes |
| staffCode | string | Yes |
| status | enum | Yes |
| createdAt | timestamp | Yes |
| updatedAt | timestamp | Yes |
| createdBy | string | Yes |
| updatedBy | string | Yes |

## Status

```text
ACTIVE
INACTIVE
```

## Staff Creation

Staff must NOT be publicly registered.

Recommended:

```text
Admin
→ Add Staff
→ Cloud Function
→ Firebase Admin SDK
→ Create Auth account
→ Create user document
→ Create staff document
```

The client must never receive Firebase Admin credentials.

---

# 10. Customer Collection

## Path

```text
businesses/{businessId}/customers/{customerId}
```

## Schema

| Field | Type | Required |
|---|---|---:|
| customerId | string | Yes |
| userId | string/null | No |
| businessId | string | Yes |
| name | string | Yes |
| email | string | Yes |
| phone | string | Yes |
| customerType | enum | Yes |
| businessName | string/null | Conditional |
| businessType | string/null | Conditional |
| businessAddress | string/null | Conditional |
| businessPhone | string/null | Conditional |
| address | string | Yes |
| pinCode | string | Yes |
| status | enum | Yes |
| createdAt | timestamp | Yes |
| updatedAt | timestamp | Yes |
| createdBy | string | Yes |
| updatedBy | string | Yes |

## Customer Type

```text
PERSONAL
BUSINESS
```

## Business Type

Recommended values:

```text
HOTEL
HOMESTAY
RESTAURANT
OFFICE
HOSTEL
SALON
SHOP
OTHER
```

Stored as strings so future business types can be added.

---

# 11. Customer Business Rules

If:

```text
customerType = PERSONAL
```

then:

```text
businessName = null
businessType = null
businessAddress = null
businessPhone = null
```

If:

```text
customerType = BUSINESS
```

then:

```text
businessName = required
businessType = required
businessAddress = optional
businessPhone = optional
```

V1 supports one current business association per customer.

There is no separate business account system for customers.

---

# 12. Customer Profile Change

If:

```text
BUSINESS → PERSONAL
```

the application must:

1. Ask for confirmation.
2. Change customer type.
3. Clear current business fields.
4. Preserve historical order snapshots.

Existing orders must NOT be changed.

---

# 13. Item Collection

## Path

```text
businesses/{businessId}/items/{itemId}
```

## Schema

```text
itemId
businessId
name
status
createdAt
updatedAt
createdBy
updatedBy
```

## Status

```text
ACTIVE
INACTIVE
```

Example:

```text
Shirt
Pant
Jeans
T-Shirt
Saree
Bedsheet
Blanket
```

---

# 14. Service Collection

## Path

```text
businesses/{businessId}/services/{serviceId}
```

## Schema

```text
serviceId
businessId
name
status
createdAt
updatedAt
createdBy
updatedBy
```

Examples:

```text
Wash
Wash + Iron
Iron
Dry Clean
```

---

# 15. Price Collection

## Path

```text
businesses/{businessId}/prices/{priceId}
```

## Schema

| Field | Type | Required |
|---|---|---:|
| priceId | string | Yes |
| businessId | string | Yes |
| itemId | string | Yes |
| serviceId | string | Yes |
| price | number | Yes |
| status | enum | Yes |
| createdAt | timestamp | Yes |
| updatedAt | timestamp | Yes |
| createdBy | string | Yes |
| updatedBy | string | Yes |

## Example

```text
itemId = shirt
serviceId = wash
price = 30
```

---

# 16. Price Uniqueness

For an active price:

```text
businessId + itemId + serviceId
```

should represent one active pricing combination.

Example:

```text
Shirt + Wash
Shirt + Wash + Iron
Pant + Wash
```

Duplicate active price combinations must be prevented.

---

# 17. Order Collection

## Path

```text
businesses/{businessId}/orders/{orderId}
```

## Main Schema

```text
orderId
businessId

source

customerId

customerNameAtOrder
customerPhoneAtOrder
businessNameAtOrder

pickupAddress

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

version

clientCreatedAt
syncId
```

---

# 18. Order Field Definitions

| Field | Type | Required | Description |
|---|---|---:|---|
| orderId | string | Yes | Stable order ID |
| businessId | string | Yes | Business owner |
| source | enum | Yes | CUSTOMER / STAFF |
| customerId | string | Yes | Customer reference |
| customerNameAtOrder | string | Yes | Historical snapshot |
| customerPhoneAtOrder | string | Yes | Historical snapshot |
| businessNameAtOrder | string/null | No | Historical snapshot |
| pickupAddress | string | Yes | Order pickup address |
| estimatedAmount | number | Yes | Initial estimate |
| finalAmount | number | Yes | Final billed amount |
| gstApplied | boolean | Yes | GST applied for this order |
| gstRate | number | Yes | Saved GST rate |
| gstAmount | number | Yes | Saved GST amount |
| paidAmount | number | Yes | Total payments |
| dueAmount | number | Yes | Remaining amount |
| paymentStatus | enum | Yes | Payment state |
| paymentMethod | enum/null | No | Primary/latest method if applicable |
| orderStatus | enum | Yes | Operational state |
| pickupDate | string | Yes | Local date |
| pickupTime | string | Yes | Local time |
| customerNote | string/null | No | Customer note |
| invoiceNumber | string | Yes | Invoice identifier |
| createdAt | timestamp | Yes | Creation time |
| updatedAt | timestamp | Yes | Last update |
| createdBy | string | Yes | Creator UID |
| lastUpdatedBy | string | Yes | Last updater UID |
| version | number | Yes | Conflict control |
| clientCreatedAt | timestamp | Yes | Client creation timestamp |
| syncId | string | Yes | Idempotency key |

---

# 19. Order Source

```text
CUSTOMER
STAFF
```

`STAFF` includes:

- walk-in order
- phone order
- manually entered order

---

# 20. Order Status Enum

Exact backend values:

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

# 21. Allowed Order Status Flow

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
NEW → CANCELLED
PICKUP_PENDING → CANCELLED
PICKED_UP → CANCELLED
PROCESSING → CANCELLED
READY → CANCELLED
OUT_FOR_DELIVERY → CANCELLED
```

Final states:

```text
DELIVERED
CANCELLED
```

A final state should not normally be changed.

Any exceptional reversal must be an explicitly controlled Admin/backend operation and audited.

---

# 22. Order Item Subcollection

## Path

```text
businesses/{businessId}/orders/{orderId}/items/{orderItemId}
```

Alternative implementation is to store items as an embedded array.

For V1, an embedded `items[]` array is acceptable and simpler for mobile/offline operation.

Recommended order representation:

```text
items: [
  {
    orderItemId,
    itemId,
    itemName,
    serviceId,
    serviceName,
    quantity,
    priceAtOrderTime,
    subtotal
  }
]
```

---

# 23. Order Item Schema

```text
orderItemId
itemId
itemName
serviceId
serviceName
quantity
priceAtOrderTime
subtotal
```

## Field Types

| Field | Type |
|---|---|
| orderItemId | string |
| itemId | string |
| itemName | string |
| serviceId | string |
| serviceName | string |
| quantity | integer |
| priceAtOrderTime | number |
| subtotal | number |

---

# 24. Historical Price Rule

This is mandatory.

When an order is created:

```text
current price
      ↓
priceAtOrderTime
      ↓
saved permanently inside order
```

If Admin later changes:

```text
Shirt + Wash = ₹30
```

to:

```text
Shirt + Wash = ₹35
```

old orders remain:

```text
priceAtOrderTime = ₹30
```

The app must NEVER recalculate historical orders using the current price table.

---

# 25. Order Calculation

For each item:

```text
subtotal = quantity × priceAtOrderTime
```

Order subtotal:

```text
subtotal = SUM(all order item subtotals)
```

If GST is applied:

```text
gstAmount = subtotal × gstRate / 100
```

Final amount:

```text
finalAmount = subtotal + gstAmount
```

Without GST:

```text
gstAmount = 0
finalAmount = subtotal
```

All monetary calculations must use safe decimal/integer-based handling to avoid floating-point errors.

For Indian currency, storing monetary values as paise integers is recommended internally.

Example:

```text
₹500 = 50000 paise
```

The UI converts it back to:

```text
₹500.00
```

If the application uses rupee decimals internally, strict rounding must still be applied.

---

# 26. Estimated vs Final Amount

Customer-created orders may initially contain:

```text
estimatedAmount
```

Staff/Admin may verify:

- item
- service
- quantity
- price

and set:

```text
finalAmount
```

The final invoice must use the final stored values.

Changes to final billing information must be controlled and audited.

---

# 27. GST Per Order

GST is selected independently for every order.

UI:

```text
GST
○ With GST
○ Without GST
```

Backend stores:

```text
gstApplied
gstRate
gstAmount
```

Example:

```text
Subtotal = ₹500
GST Rate = 18%
GST = ₹90
Final = ₹590
```

Without GST:

```text
Subtotal = ₹500
GST = ₹0
Final = ₹500
```

Changing the business default GST rate must not modify existing orders.

---

# 28. Payment Status

Enum:

```text
PENDING
PARTIALLY_PAID
PAID
```

Rules:

```text
paidAmount = 0
→ PENDING
```

```text
0 < paidAmount < finalAmount
→ PARTIALLY_PAID
```

```text
paidAmount >= finalAmount
→ PAID
```

`dueAmount`:

```text
dueAmount = finalAmount - paidAmount
```

Due amount cannot be negative.

---

# 29. Payment Collection

## Path

```text
businesses/{businessId}/orders/{orderId}/payments/{paymentId}
```

## Schema

```text
paymentId
orderId
businessId
amount
paymentMethod
paymentDate
recordedBy
note
createdAt
```

## Payment Method

```text
CASH
UPI
ONLINE
```

Multiple payments may exist for one order.

Example:

```text
Payment 1 = ₹200 CASH
Payment 2 = ₹300 UPI
```

Total:

```text
paidAmount = ₹500
```

---

# 30. Payment Integrity

Payment records should be treated as append-only financial history.

Do not silently overwrite old payment records.

If a correction is required:

```text
new correction/reversal record
```

should be created according to the final payment policy.

Historical financial records must remain auditable.

---

# 31. Payment and Order Status Independence

Payment status and order status are separate.

Valid example:

```text
orderStatus = DELIVERED
paymentStatus = PENDING
```

Another:

```text
orderStatus = PROCESSING
paymentStatus = PAID
```

They must not automatically force each other to change unless a future business rule explicitly requires it.

---

# 32. Status History

## Path

```text
businesses/{businessId}/orders/{orderId}/statusHistory/{statusHistoryId}
```

## Schema

```text
statusHistoryId
orderId
businessId

fromStatus
toStatus

changedAt
changedBy

note
```

## Example

```text
fromStatus: PROCESSING
toStatus: READY

changedAt: ...
changedBy: staffUserId
note: "Laundry processing completed"
```

Status history should be append-only.

---

# 33. Customer Historical Snapshot

When an order is created, store:

```text
customerNameAtOrder
customerPhoneAtOrder
businessNameAtOrder
pickupAddress
```

Example:

```text
Customer profile today:
Rahul Sharma
Hotel Paradise
Phone: 9876543210
```

Old order:

```text
customerNameAtOrder = Rahul Sharma
businessNameAtOrder = Hotel Paradise
customerPhoneAtOrder = 9876543210
```

If Rahul later changes the profile, the old order remains unchanged.

This is mandatory for invoice and historical report integrity.

---

# 34. Invoice Number

Invoice number is separate from Order ID.

Example:

```text
Order ID:
ORD1025

Invoice:
INV1025
```

Both must remain stable after creation.

Invoice number should not be regenerated every time the invoice PDF is opened.

---

# 35. Notification Token Collection

## Path

```text
businesses/{businessId}/notificationTokens/{tokenId}
```

## Schema

```text
tokenId
userId
businessId
fcmToken
platform
deviceId
active
createdAt
updatedAt
```

## Platform

```text
ANDROID
IOS
WEB
```

V1 primarily targets Android.

---

# 36. Notification Rules

Customer notifications may be sent for:

```text
Order Received
Pickup
Processing
Ready
Out for Delivery
Delivered
```

Staff notifications may include:

```text
New Customer Order
Pickup Request
```

Notification failure must never cause order creation to fail.

---

# 37. SQLite Local Database

SQLite is the primary local operational store.

Recommended tables:

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

---

# 38. SQLite users Table

```sql
users
```

Fields:

```text
user_id TEXT PRIMARY KEY
name TEXT NOT NULL
email TEXT NOT NULL
phone TEXT
role TEXT NOT NULL
business_id TEXT
status TEXT NOT NULL
profile_completed INTEGER NOT NULL DEFAULT 0
force_password_change INTEGER NOT NULL DEFAULT 0
created_at INTEGER NOT NULL
updated_at INTEGER NOT NULL
```

---

# 39. SQLite customers Table

```sql
customers
```

Fields:

```text
customer_id TEXT PRIMARY KEY
user_id TEXT
business_id TEXT NOT NULL

name TEXT NOT NULL
email TEXT
phone TEXT NOT NULL

customer_type TEXT NOT NULL

business_name TEXT
business_type TEXT
business_address TEXT
business_phone TEXT

address TEXT NOT NULL
pin_code TEXT NOT NULL

status TEXT NOT NULL

created_at INTEGER NOT NULL
updated_at INTEGER NOT NULL

created_by TEXT
updated_by TEXT
```

---

# 40. SQLite staff Table

```sql
staff
```

Fields:

```text
staff_id TEXT PRIMARY KEY
user_id TEXT NOT NULL
business_id TEXT NOT NULL

name TEXT NOT NULL
email TEXT
phone TEXT
staff_code TEXT

status TEXT NOT NULL

created_at INTEGER NOT NULL
updated_at INTEGER NOT NULL

created_by TEXT
updated_by TEXT
```

---

# 41. SQLite items Table

```sql
items
```

Fields:

```text
item_id TEXT PRIMARY KEY
business_id TEXT NOT NULL
name TEXT NOT NULL
status TEXT NOT NULL
created_at INTEGER NOT NULL
updated_at INTEGER NOT NULL
created_by TEXT
updated_by TEXT
```

---

# 42. SQLite services Table

```sql
services
```

Fields:

```text
service_id TEXT PRIMARY KEY
business_id TEXT NOT NULL
name TEXT NOT NULL
status TEXT NOT NULL
created_at INTEGER NOT NULL
updated_at INTEGER NOT NULL
created_by TEXT
updated_by TEXT
```

---

# 43. SQLite prices Table

```sql
prices
```

Fields:

```text
price_id TEXT PRIMARY KEY
business_id TEXT NOT NULL
item_id TEXT NOT NULL
service_id TEXT NOT NULL
price INTEGER NOT NULL
status TEXT NOT NULL
created_at INTEGER NOT NULL
updated_at INTEGER NOT NULL
created_by TEXT
updated_by TEXT
```

Recommended:

```text
price = amount in paise
```

---

# 44. SQLite orders Table

```sql
orders
```

Fields:

```text
order_id TEXT PRIMARY KEY
business_id TEXT NOT NULL

source TEXT NOT NULL

customer_id TEXT NOT NULL

customer_name_at_order TEXT NOT NULL
customer_phone_at_order TEXT NOT NULL
business_name_at_order TEXT

pickup_address TEXT NOT NULL

estimated_amount INTEGER NOT NULL
final_amount INTEGER NOT NULL

gst_applied INTEGER NOT NULL DEFAULT 0
gst_rate INTEGER NOT NULL DEFAULT 0
gst_amount INTEGER NOT NULL DEFAULT 0

paid_amount INTEGER NOT NULL DEFAULT 0
due_amount INTEGER NOT NULL DEFAULT 0

payment_status TEXT NOT NULL
payment_method TEXT

order_status TEXT NOT NULL

pickup_date TEXT NOT NULL
pickup_time TEXT NOT NULL

customer_note TEXT

invoice_number TEXT NOT NULL

created_at INTEGER NOT NULL
updated_at INTEGER NOT NULL

created_by TEXT NOT NULL
last_updated_by TEXT NOT NULL

version INTEGER NOT NULL DEFAULT 1

client_created_at INTEGER NOT NULL
sync_id TEXT NOT NULL UNIQUE
```

---

# 45. SQLite order_items Table

```sql
order_items
```

Fields:

```text
order_item_id TEXT PRIMARY KEY
order_id TEXT NOT NULL

item_id TEXT NOT NULL
item_name TEXT NOT NULL

service_id TEXT NOT NULL
service_name TEXT NOT NULL

quantity INTEGER NOT NULL

price_at_order_time INTEGER NOT NULL
subtotal INTEGER NOT NULL

created_at INTEGER NOT NULL

FOREIGN KEY(order_id) REFERENCES orders(order_id)
```

---

# 46. SQLite payments Table

```sql
payments
```

Fields:

```text
payment_id TEXT PRIMARY KEY
order_id TEXT NOT NULL
business_id TEXT NOT NULL

amount INTEGER NOT NULL
payment_method TEXT NOT NULL

payment_date INTEGER NOT NULL

recorded_by TEXT NOT NULL
note TEXT

created_at INTEGER NOT NULL

FOREIGN KEY(order_id) REFERENCES orders(order_id)
```

---

# 47. SQLite status_history Table

```sql
status_history
```

Fields:

```text
status_history_id TEXT PRIMARY KEY
order_id TEXT NOT NULL
business_id TEXT NOT NULL

from_status TEXT
to_status TEXT NOT NULL

changed_at INTEGER NOT NULL
changed_by TEXT NOT NULL

note TEXT

FOREIGN KEY(order_id) REFERENCES orders(order_id)
```

---

# 48. SQLite business_settings Table

```sql
business_settings
```

Fields:

```text
business_id TEXT PRIMARY KEY

business_name TEXT NOT NULL
address TEXT NOT NULL
phone TEXT NOT NULL
email TEXT

gstin TEXT
default_gst_rate INTEGER NOT NULL DEFAULT 0

invoice_prefix TEXT NOT NULL
order_prefix TEXT NOT NULL

invoice_thank_you_message TEXT

updated_at INTEGER NOT NULL
updated_by TEXT
```

---

# 49. Sync Queue

## Table

```text
sync_queue
```

This table is critical to offline-first functionality.

## Schema

```text
sync_id TEXT PRIMARY KEY

entity_type TEXT NOT NULL
entity_id TEXT NOT NULL

operation TEXT NOT NULL

status TEXT NOT NULL

created_at INTEGER NOT NULL
updated_at INTEGER NOT NULL

retry_count INTEGER NOT NULL DEFAULT 0

last_error TEXT

payload_version INTEGER
```

---

# 50. Sync Queue Enums

## Entity Type

```text
USER
CUSTOMER
STAFF
ITEM
SERVICE
PRICE
ORDER
ORDER_ITEM
PAYMENT
STATUS_HISTORY
BUSINESS_SETTINGS
```

## Operation

```text
CREATE
UPDATE
DELETE
```

Although `DELETE` exists for sync architecture, historical business records should normally use `INACTIVE` rather than actual deletion.

## Sync Status

```text
PENDING
SYNCING
SYNCED
FAILED
```

---

# 51. Offline Order Creation

Example:

```text
User creates ORD1052
        ↓
SQLite transaction
        ↓
orders inserted
order_items inserted
status_history inserted
sync_queue inserted
        ↓
UI immediately shows order
        ↓
Internet unavailable
        ↓
Order remains locally available
        ↓
Internet returns
        ↓
Sync worker processes queue
        ↓
Firestore
        ↓
SYNCED
```

The user must never lose the order because the internet disappeared after pressing Confirm.

---

# 52. SQLite Transaction Rule

Creating an order must be atomic locally.

These operations should happen inside one SQLite transaction:

```text
Create order
Create order items
Create initial status history
Create sync queue entry
```

If any part fails:

```text
ROLLBACK
```

No partially-created local order should remain.

---

# 53. Sync Idempotency

Every client-created operation must have:

```text
syncId
```

The backend must recognize already-processed operations.

Example:

```text
Client sends ORD1052
Network timeout occurs
Client retries
```

Without idempotency:

```text
ORD1052
ORD1053
```

could accidentally be created twice.

With idempotency:

```text
same syncId
→ detect already processed
→ return existing result
```

---

# 54. Client-Generated Order IDs

Orders should preferably receive a unique client-generated ID before cloud synchronization.

Example:

```text
ORD-20260914-8F4A21
```

The exact visual format may be different, but uniqueness is mandatory.

This allows:

- offline creation
- local references
- retries
- invoice generation
- sync without waiting for Firestore-generated IDs

---

# 55. Conflict Control

Important records contain:

```text
version
updatedAt
updatedBy
```

When updating an order:

```text
local version = 5
```

Cloud currently:

```text
version = 6
```

The client must not silently overwrite version 6.

Possible strategy:

```text
detect conflict
→ fetch latest
→ compare fields
→ merge if safe
→ otherwise require controlled resolution
```

Payment and status changes should use stronger server-side validation.

---

# 56. Order Update Protection

The following fields require special protection:

```text
finalAmount
gstApplied
gstRate
gstAmount
paidAmount
dueAmount
paymentStatus
orderStatus
invoiceNumber
customerId
businessId
createdBy
```

Customers must not directly update these fields.

---

# 57. Firestore Security Model

Security must be based on:

```text
Firebase Auth UID
+
trusted role
+
businessId
+
resource ownership
```

Never trust only values supplied by the client.

---

# 58. Customer Access

Customer can:

```text
read own customer profile
update permitted own profile fields
read own orders
create own orders
```

Customer cannot:

```text
read another customer
read all orders
modify order final amount
modify payment history
change role
change businessId
change GST settings
manage staff
manage prices
change order status
```

---

# 59. Staff Access

Staff can access only their business.

Staff may:

```text
read customers
read orders
create manual orders
update operational order status
record payments
read items/services/prices
generate invoices
```

subject to the exact permissions implemented.

Staff cannot:

```text
create Admin
create Staff
change role
change businessId
manage staff
change prices
change GST settings
delete historical orders
delete financial history
```

---

# 60. Admin Access

Admin can access all data belonging to the business.

Admin can:

```text
manage staff
manage customers
manage items
manage services
manage prices
manage GST settings
manage business settings
manage orders
record payments
generate invoices
generate reports
```

Admin still should not casually delete historical financial records.

---

# 61. Role Security

The UI may hide unauthorized features, but UI hiding is NOT security.

Backend must enforce:

```text
CUSTOMER
STAFF
ADMIN
```

at the Firestore/security-function layer.

A modified APK or malicious client must not be able to bypass these restrictions.

---

# 62. Trusted Role Source

Role should be determined from a trusted backend-controlled user profile.

For stronger security, custom claims may also be used:

```text
role
businessId
```

However, Firestore user profile remains the application data source.

Any change to privileged roles should be performed through controlled backend functionality.

---

# 63. Staff Account Security

Admin-created staff accounts should use:

```text
temporary password
```

and:

```text
forcePasswordChange = true
```

on first login.

Staff must set a new password before normal application use.

---

# 64. Password Storage

Never store:

```text
password
plainPassword
passwordHash
temporaryPassword
```

inside Firestore application documents.

Firebase Authentication manages password security.

---

# 65. Firestore Indexes

Composite indexes will likely be required for queries such as:

### Orders by business + status + date

```text
businessId
orderStatus
createdAt DESC
```

### Orders by business + payment status + date

```text
businessId
paymentStatus
createdAt DESC
```

### Orders by business + customer + date

```text
businessId
customerId
createdAt DESC
```

### Orders by business + source + date

```text
businessId
source
createdAt DESC
```

### Customers by business + status

```text
businessId
status
name
```

Exact indexes should be finalized from actual Firestore query usage during implementation.

Do not create unnecessary indexes for every field.

---

# 66. Search Strategy

Firestore is not intended to provide unrestricted full-text search.

Required searches include:

```text
Order ID
Invoice number
Customer name
Mobile number
Business name
Email
```

For V1, recommended approach:

```text
SQLite local search
```

for offline-capable operational search.

Cloud search can use:

- normalized searchable fields
- prefix queries
- carefully designed indexes

A dedicated search engine is NOT required in V1.

---

# 67. Normalized Search Fields

For efficient local/cloud lookup, optional normalized fields may be stored:

```text
nameNormalized
phoneNormalized
emailNormalized
businessNameNormalized
```

Example:

```text
Rahul Sharma
```

becomes:

```text
rahul sharma
```

Phone:

```text
9876543210
```

remains normalized numeric text.

---

# 68. Reports Data Source

Reports must use stored historical order values.

Never calculate historical reports using today's:

```text
price
GST rate
service price
item price
```

Reports use:

```text
order.finalAmount
order.gstAmount
order.paidAmount
order.paymentStatus
order.orderStatus
order.items[].priceAtOrderTime
```

---

# 69. Report Metrics

The database must support:

```text
Total Orders
Completed Orders
Cancelled Orders
Active/Pending Orders

Subtotal
GST
Grand Total

Paid Amount
Pending Amount

Cash
UPI
Online

Orders by Status
Service Quantity
Service Revenue
Item Quantity
Item Revenue
```

Yearly reports additionally require:

```text
Monthly Breakdown
```

---

# 70. Weekly Report Date Rule

For a selected month:

```text
Month + Year
+
From Date
+
To Date
```

The selected dates must remain within the selected month.

Example:

```text
September 2026

From: 07 Sep
To: 13 Sep
```

must query only:

```text
2026-09-07 → 2026-09-13
```

---

# 71. Local Invoice Generation

Invoice PDF does not require Firebase Storage in V1.

Flow:

```text
SQLite
→ Order data
→ Invoice generator
→ A4 PDF
→ Local device storage
→ Open / Share
```

Invoice data must come from the stored order snapshot.

---

# 72. Invoice Required Data

```text
Business Name
Business Address
Business Phone
Business Email
GSTIN if configured

Invoice Number
Order ID
Invoice Date

Customer Name
Customer Mobile
Customer Address
Business Name if applicable

Item
Service
Quantity
Rate
Amount

Subtotal
GST Rate
GST Amount
Grand Total

Paid Amount
Due Amount
Payment Status
Payment Method

Thank-you message
```

---

# 73. Local Excel Report

Reports are generated locally.

Flow:

```text
SQLite
→ Query historical data
→ Build report dataset
→ XLSX generator
→ Local .xlsx file
→ Open / Share
```

No Firebase Storage is required for V1.

---

# 74. Local File Rules

Generated files:

```text
PDF
XLSX
```

are local device files.

Example:

```text
Laundry_Report_September_2026.xlsx
Laundry_Report_2026.xlsx
Laundry_Report_07-09-2026_to_13-09-2026.xlsx
```

---

# 75. Data Synchronization Direction

Initial synchronization:

```text
Firestore
→ SQLite
```

Offline-created changes:

```text
SQLite
→ Sync Queue
→ Firestore
```

Cloud changes from another device:

```text
Firestore
→ Sync
→ SQLite
```

---

# 76. Master Data Sync

Master data includes:

```text
Customers
Items
Services
Prices
Business Settings
Staff
```

The app should periodically refresh these records when online.

---

# 77. Order Sync Priority

Recommended priority:

```text
1. Authentication/user state
2. Orders
3. Payments
4. Status history
5. Customers
6. Master data
7. Other non-critical data
```

However, dependency order must be respected.

For example:

```text
Customer → Order
```

when a new customer must exist before the order can reference it.

---

# 78. Retry Strategy

Failed synchronization should not delete local data.

Example:

```text
Attempt 1 → failed
Attempt 2 → failed
Attempt 3 → success
```

Recommended backoff:

```text
5 sec
15 sec
30 sec
1 min
5 min
15 min
```

with a reasonable maximum retry delay.

The exact retry strategy can be implemented in the sync service.

---

# 79. Failed Sync UI

If records are waiting:

```text
Some data is waiting to sync.
Please keep the app open when internet is available.
```

If an operation fails permanently:

```text
Sync could not be completed.
Your local data is still محفوظ/saved on this device.
Please try again when internet is available.
```

The final user-facing wording should remain in simple Indian English.

Raw Firebase errors must never be shown.

---

# 80. Sync States in UI

Recommended:

```text
Saved locally
Syncing
Synced
Sync pending
Sync failed
```

Important distinction:

```text
Saved locally ≠ Synced to cloud
```

---

# 81. Data Integrity Rules

The following must always be true:

```text
finalAmount >= 0
paidAmount >= 0
dueAmount >= 0
quantity >= 1
priceAtOrderTime >= 0
gstRate >= 0
gstAmount >= 0
```

Also:

```text
dueAmount = max(finalAmount - paidAmount, 0)
```

and payment status must match the payment totals.

---

# 82. Order Validation

Before creating an order:

```text
customer exists
customer belongs to same business
items exist
services exist
price exists
quantity >= 1
pickup details valid
GST selection valid
calculated amount valid
```

Client validates first.

Backend must validate privileged/critical operations again.

---

# 83. Business Isolation Validation

For every business-owned resource:

```text
resource.businessId == authenticatedUser.businessId
```

must be verified where applicable.

A user from Business A must never access:

```text
Business B customers
Business B orders
Business B staff
Business B prices
Business B reports
```

---

# 84. Historical Record Protection

The following historical information should be treated as immutable after order creation unless controlled correction is required:

```text
orderId
invoiceNumber
customerNameAtOrder
customerPhoneAtOrder
businessNameAtOrder
priceAtOrderTime
GST rate applied
GST amount
payment history
status history
createdAt
createdBy
```

---

# 85. Audit Fields

Operational documents should contain:

```text
createdAt
updatedAt
createdBy
updatedBy
```

For orders:

```text
lastUpdatedBy
```

For status:

```text
changedBy
changedAt
```

For payments:

```text
recordedBy
paymentDate
```

This makes important actions traceable.

---

# 86. App Metadata

## SQLite

```text
app_metadata
```

Fields:

```text
key TEXT PRIMARY KEY
value TEXT
updated_at INTEGER
```

Recommended keys:

```text
schema_version
last_full_sync
last_order_sync
last_master_data_sync
device_id
```

---

# 87. Database Versioning

SQLite schema must have a migration version.

Example:

```text
schema_version = 1
```

Future:

```text
schema_version = 2
schema_version = 3
```

Every database change must have a migration.

Never simply delete the existing database during an app update.

---

# 88. Firestore Data Migration

When changing Firestore schema:

```text
old field
→ migration strategy
→ new field
```

must be documented.

Historical orders should remain readable after app updates.

---

# 89. Relationship Diagram

```text
Firebase Auth User
        │
        ▼
     users
        │
        ├──────────────┐
        ▼              ▼
   CUSTOMER          STAFF
        │              │
        ▼              ▼
   customers        staff
        │
        │
        ▼
      orders
        │
        ├───────────────┐
        ▼               ▼
  order items        payments
        │
        ▼
 price snapshot

orders
   │
   ▼
statusHistory
```

Business:

```text
business
 ├── staff
 ├── customers
 ├── items
 ├── services
 ├── prices
 ├── orders
 │    ├── payments
 │    └── statusHistory
 ├── settings
 └── notificationTokens
```

---

# 90. V1 Database Does NOT Include

The following are intentionally excluded:

```text
inventory
expenses
payroll
salary
advanced accounting
loyalty points
coupons
GPS tracking
route optimization
QR/barcode garment tracking
multiple branches
multiple processing centres
delivery app
manager role
super admin
corporate account system
AI
```

---

# 91. Future-Compatible Fields

The schema intentionally leaves room for:

```text
branchId
deliveryAgentId
photoUrls
qrCode
barcode
paymentGatewayId
externalTransactionId
```

These should NOT be implemented in V1 unless required.

---

# 92. Security Checklist

Implementation must verify:

```text
[ ] Firebase Auth enabled
[ ] Email verification enforced
[ ] Password reset uses Firebase
[ ] Passwords never stored in Firestore
[ ] Role enforced server-side
[ ] businessId enforced server-side
[ ] Customer ownership enforced
[ ] Staff permissions enforced
[ ] Admin-only operations protected
[ ] Staff creation uses trusted backend
[ ] Firebase Admin credentials never in app
[ ] Payment records protected
[ ] Status transitions validated
[ ] Historical values protected
[ ] Invalid business access blocked
[ ] Raw Firebase errors hidden from users
```

---

# 93. Backend Rules Checklist

```text
[ ] Every business record has businessId
[ ] Every order has customerId
[ ] Every order stores customer snapshot
[ ] Every order item stores priceAtOrderTime
[ ] GST stored per order
[ ] Payment history stored separately
[ ] Status history append-only
[ ] Invoice number immutable
[ ] Order ID immutable
[ ] Prices never recalculate old orders
[ ] Customer profile changes never rewrite historical orders
[ ] Historical reports use stored values
[ ] Local writes happen before sync
[ ] Sync queue persists failed operations
[ ] Sync is idempotent
[ ] Duplicate retry cannot create duplicate order
[ ] Important conflicts are detected
```

---

# 94. Final Backend Flow

## Customer Registration

```text
Firebase Auth
      ↓
Email verification
      ↓
users/{uid}
      ↓
Customer profile
      ↓
customers/{customerId}
```

---

## Customer Order

```text
Customer
   ↓
Select Item
   ↓
Select Service
   ↓
Quantity
   ↓
Read current price
   ↓
Copy price into priceAtOrderTime
   ↓
Pickup details
   ↓
GST selection
   ↓
Calculate estimate
   ↓
SQLite transaction
   ↓
Order + Items + Status History + Sync Queue
   ↓
UI immediately updated
   ↓
Firestore sync
```

---

## Staff Order

```text
Staff
 ↓
Select/Create Customer
 ↓
Items
 ↓
Services
 ↓
Quantity
 ↓
Price
 ↓
GST
 ↓
Payment
 ↓
SQLite
 ↓
Firestore
```

---

## Payment

```text
Payment
 ↓
SQLite payment record
 ↓
Recalculate paidAmount
 ↓
Recalculate dueAmount
 ↓
Set paymentStatus
 ↓
Sync payment
 ↓
Sync order totals
```

---

## Status Update

```text
Current Status
      ↓
Validate allowed transition
      ↓
SQLite transaction
      ↓
Update orderStatus
      ↓
Create statusHistory
      ↓
Sync queue
      ↓
Firestore
```

---

# 95. Final Schema Principle

The backend must follow these rules above everything else:

```text
LOCAL FIRST
+
NO DATA LOSS
+
BUSINESS ISOLATION
+
ROLE SECURITY
+
HISTORICAL DATA IMMUTABILITY
+
IDEMPOTENT SYNC
+
AUDITABLE PAYMENTS
+
AUDITABLE STATUS CHANGES
+
STORED PRICE SNAPSHOTS
+
STORED GST SNAPSHOTS
```

The database should remain simple enough for a small local laundry business while being structured correctly for future web administration and controlled expansion.

---

# 96. Implementation Definition of Done

Backend schema is considered ready when:

```text
[ ] Firebase Auth is configured
[ ] Firestore structure is implemented
[ ] User roles are implemented
[ ] Business isolation is implemented
[ ] Customer schema is implemented
[ ] Staff schema is implemented
[ ] Item/service/price schema is implemented
[ ] Order schema is implemented
[ ] Order item snapshots are implemented
[ ] GST-per-order storage is implemented
[ ] Payment history is implemented
[ ] Status history is implemented
[ ] Historical snapshots are implemented
[ ] SQLite schema is implemented
[ ] SQLite migrations are implemented
[ ] Sync queue is implemented
[ ] Retry mechanism is implemented
[ ] Idempotency is implemented
[ ] Conflict detection is implemented
[ ] Security rules are implemented
[ ] Privileged operations use Cloud Functions/Admin SDK
[ ] Firestore indexes are configured
[ ] Invoice data source is implemented
[ ] Report data source is implemented
[ ] Notification token storage is implemented if FCM is enabled
[ ] No historical order can be recalculated using current prices
[ ] No duplicate order is created by sync retry
[ ] No unauthorized role/business access is possible
```

---

# 97. Final Database Summary

```text
AUTH
└── Firebase Authentication

FIRESTORE
├── users
└── businesses
    ├── staff
    ├── customers
    ├── items
    ├── services
    ├── prices
    ├── orders
    │   ├── payments
    │   └── statusHistory
    ├── settings
    └── notificationTokens

SQLITE
├── users
├── business
├── customers
├── staff
├── items
├── services
├── prices
├── orders
├── order_items
├── payments
├── status_history
├── business_settings
├── notification_tokens
├── sync_queue
└── app_metadata
```

**This document is the exact backend/data-storage contract for V1. Any implementation that changes field names, enums, relationships, historical-data behavior, or permission boundaries should update this document and the dependent TRD/UI-UX implementation before coding.**