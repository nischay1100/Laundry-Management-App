# Laundry Management App — UI/UX Requirements Document

**Document:** UI-UX.md  
**Version:** 1.0  
**Status:** Implementation Ready  
**Related Documents:** PRD.md, TRD.md  
**Platform:** Android  
**UI Language:** Indian English  
**Currency:** Indian Rupee (₹)  
**Design Direction:** Simple, Clean, Fast, Practical

---

# 1. Purpose

This document defines the complete user interface and user experience requirements for the Laundry Management App.

It specifies:

- Screens
- Navigation
- Layout
- Components
- Buttons
- Forms
- User interactions
- Validation
- Loading states
- Empty states
- Error states
- Offline states
- Customer UI
- Staff UI
- Admin UI
- Order workflow UI
- Payment UI
- GST UI
- Invoice UI
- Report UI

The implementation must follow this document unless a requirement is explicitly changed.

---

# 2. Design Philosophy

The application is for a local laundry business and may be operated by non-technical staff.

Therefore the UI must prioritize:

```text
Simple
↓
Clear
↓
Fast
↓
Easy to understand
↓
Easy to operate
```

Avoid unnecessary:

- Animations
- Complicated menus
- Decorative screens
- Excessive cards
- Technical terminology
- Deep navigation
- Tiny buttons

---

# 3. Visual Design Direction

The application should feel:

- Professional
- Clean
- Modern
- Trustworthy
- Friendly
- Business-oriented

It should not look like a complicated ERP.

---

# 4. Responsive Design

The Android application must support common Android screen sizes.

The UI must work correctly on:

- Small phones
- Normal phones
- Large phones
- Different aspect ratios

Do not use fixed dimensions that cause clipping.

---

# 5. Typography

Use a clean, highly readable sans-serif font.

Hierarchy:

```text
Screen Title
Section Heading
Card Title
Body Text
Secondary Text
Caption
```

Important numbers such as:

```text
₹5,900
12 Orders
₹1,250 Pending
```

may use stronger typography.

---

# 6. Currency

Always display Indian Rupee:

```text
₹
```

Examples:

```text
₹30
₹500
₹1,250
₹5,900
```

The ₹ symbol must render correctly on all supported Android devices.

---

# 7. Date Format

Preferred:

```text
14 September 2026
```

or shorter:

```text
14 Sep 2026
```

For reports/files:

```text
07-09-2026
```

---

# 8. Indian English

Use:

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

# 9. Global Navigation Model

After authentication, the role determines the navigation.

```text
CUSTOMER
Home | Orders | Profile

STAFF
Home | Orders | Profile

ADMIN
Home | Orders | People | Reports | More
```

Bottom navigation should remain visible on primary screens.

---

# 10. Global Header

Primary screens should have:

```text
[Screen Title]                 [Optional Action]
```

Examples:

```text
Orders                         [Search]

Customers                      [Search]

Staff                          [+ Add]

Reports
```

Avoid excessive icons without labels.

---

# 11. Global Offline Indicator

When offline, show a subtle but clearly visible indicator.

Example:

```text
⚠ Offline
```

or:

```text
Offline — Changes will sync automatically
```

It should not block the entire application.

---

# 12. Sync Indicator

When synchronization is active:

```text
Syncing...
```

After success:

```text
Synced
```

When pending:

```text
3 items waiting to sync
```

The indicator should be accessible without taking excessive screen space.

---

# 13. Global Loading State

Use:

- Spinner
- Skeleton
- Button loading state

depending on context.

Example:

```text
[ Creating Order... ]
```

The user should not be able to tap the same submit button repeatedly while it is processing.

---

# 14. Global Error State

Errors should be simple.

Example:

> Something went wrong. Please try again.

For permission errors:

> You do not have permission to perform this action.

Do not show raw Firebase errors.

---

# 15. Global Empty State

Example:

```text
No orders found.

Your orders will appear here.
```

Buttons can be included when useful.

Example:

```text
No orders yet.

[ Place New Order ]
```

---

# 16. Authentication Screens

Authentication screens:

```text
Login
Register
Verify Email
Complete Your Profile
Forgot Password
```

---

# 17. Login Screen

Layout:

```text
--------------------------------
Laundry Business
Welcome Back

Email
[________________________]

Password
[________________________]

[ Forgot Password ]

[ Login ]

Don't have an account?
[ Register ]
--------------------------------
```

---

# 18. Login Fields

### Email

Placeholder:

```text
Enter your email
```

Keyboard:

```text
Email
```

### Password

Placeholder:

```text
Enter your password
```

Password should be hidden by default.

Provide show/hide password control.

---

# 19. Login Validation

Email:

> Please enter your email.

Invalid email:

> Please enter a valid email address.

Password:

> Please enter your password.

Invalid credentials:

> Email or password is incorrect.

Unverified email:

> Please verify your email before logging in.

Inactive account:

> Your account is inactive. Please contact the administrator.

---

# 20. Login Flow

```text
Login
 ↓
Authenticate
 ↓
Email Verified?
 ├── No → Verify Email
 │
 └── Yes
       ↓
Load Profile
       ↓
Profile Complete?
 ├── No → Complete Profile
 │
 └── Yes
       ↓
Check Role
       ↓
Open Correct Dashboard
```

---

# 21. Register Screen

Customer-only public registration.

Layout:

```text
--------------------------------
Create Account

Email
[________________________]

Password
[________________________]

Confirm Password
[________________________]

[ Create Account ]

Already have an account?
[ Login ]
--------------------------------
```

Do not include:

```text
Role
Staff
Admin
```

on public registration.

---

# 22. Register Validation

Required:

- Valid email
- Valid password
- Matching password confirmation

Errors:

> Please enter your email.

> Please enter a valid email address.

> Please enter a password.

> Passwords do not match.

> This email is already registered.

---

# 23. Verify Email Screen

After successful registration:

```text
--------------------------------
Verify Your Email

We've sent a verification email to:

rahul@example.com

Please open your email and click
the verification link.

[ I Have Verified My Email ]

[ Resend Verification Email ]

[ Back to Login ]
--------------------------------
```

---

# 24. Resend Email

Prevent unlimited rapid resend requests.

Button may show:

```text
Resend available in 30s
```

After successful resend:

> Verification email sent.

---

# 25. Complete Your Profile Screen

After email verification:

```text
--------------------------------
Complete Your Profile

Full Name
[________________________]

Mobile Number
[________________________]

Customer Type

○ Personal
○ Business

Address
[________________________]
[________________________]

PIN Code
[________________________]

If Business:

Business Name
[________________________]

Business Type
[ Select Business Type ]

Business Address
[________________________]

Business Phone
[________________________]

[ Save & Continue ]
--------------------------------
```

---

# 26. Profile Conditional Fields

Default:

```text
Personal
```

When Personal:

Hide:

```text
Business Name
Business Type
Business Address
Business Phone
```

When Business:

Show them.

---

# 27. Business Type Dropdown

Suggested options:

```text
Hotel
Homestay
Restaurant
Office
Hostel
Salon
Shop
Other
```

---

# 28. Profile Validation

Required:

- Full Name
- Mobile Number
- Customer Type
- Address
- PIN Code

Business:

- Business Name
- Business Type

Optional:

- Business Address
- Business Phone

---

# 29. Profile Completion Success

After saving:

```text
Profile completed
```

Then:

```text
→ Customer Home
```

---

# 30. Forgot Password Screen

```text
--------------------------------
Reset Password

Enter your registered email.

Email
[________________________]

[ Send Reset Email ]

[ Back to Login ]
--------------------------------
```

Success:

> Password reset email sent. Please check your inbox.

---

# 31. Customer Home

Layout:

```text
--------------------------------
Hello, Rahul 👋

[ Place New Order ]

Active Order

ORD1025
Processing
₹590
[ View Order ]

Recent Orders

ORD1024     Delivered
ORD1020     Delivered
--------------------------------

Home | Orders | Profile
--------------------------------
```

The primary CTA must be:

```text
Place New Order
```

---

# 32. Customer Dashboard Information

Show:

- Customer name
- Active order
- Current status
- Amount
- Recent orders
- Place New Order button

Avoid overwhelming the customer with analytics.

---

# 33. Customer Orders Screen

```text
--------------------------------
My Orders

[ Search Orders ]

[ All ] [ Active ] [ Delivered ]

ORD1025
14 Sep 2026
Processing
₹590

ORD1024
10 Sep 2026
Delivered
₹420
--------------------------------

Home | Orders | Profile
--------------------------------
```

---

# 34. Customer Order Card

Display:

```text
Order ID
Date
Status
Amount
Payment Status
```

Example:

```text
ORD1025
14 Sep 2026

Processing

₹590
Payment: Partially Paid
```

---

# 35. Customer Order Detail

```text
--------------------------------
Order Details

ORD1025
14 September 2026

Status
● New
● Pickup
● Processing
○ Ready
○ Out for Delivery
○ Delivered

Items
Shirt       Wash + Iron    5    ₹250
Pant        Wash            2    ₹80

Subtotal                    ₹330
GST 18%                      ₹59.40
Total                       ₹389.40

Paid                        ₹200
Pending                     ₹189.40

Pickup Address
...

[ View Invoice ]
--------------------------------
```

---

# 36. Order Status Timeline

Use a vertical timeline.

Completed:

```text
● New
│
● Pickup
│
● Processing
│
○ Ready
│
○ Out for Delivery
│
○ Delivered
```

Current state should be visually emphasized.

Cancelled orders:

```text
● Cancelled
```

---

# 37. Customer Place Order — Step 1

Screen:

```text
--------------------------------
Place New Order

Select Laundry Items
```

Each item row:

```text
Shirt
[ - ] 1 [ + ]

Pant
[ - ] 0 [ + ]

Jeans
[ - ] 0 [ + ]
```

Only active items should be displayed.

---

# 38. Quantity Input

Quantity must be:

```text
>= 1
```

when selected.

Controls:

```text
[-]  2  [+]
```

may be used for quick adjustment.

---

# 39. Customer Place Order — Step 2

After selecting item:

```text
--------------------------------
Select Service

Shirt

○ Wash — ₹30
○ Wash + Iron — ₹50
○ Iron — ₹20
--------------------------------
```

Prices must come from current active pricing data.

---

# 40. Multiple Order Items

Customer can add multiple combinations.

Example:

```text
Shirt
Wash + Iron
5 × ₹50 = ₹250

Pant
Wash
2 × ₹40 = ₹80
```

---

# 41. Order Summary

```text
--------------------------------
Order Summary

Shirt
Wash + Iron
5 × ₹50                  ₹250

Pant
Wash
2 × ₹40                   ₹80

Subtotal                 ₹330

GST
○ With GST
○ Without GST

Estimated Total          ₹330
--------------------------------

[ Continue ]
```

If GST selected:

```text
GST 18%                   ₹59.40
Estimated Total          ₹389.40
```

---

# 42. GST Selection

GST is per order.

UI:

```text
GST

○ With GST
○ Without GST
```

Do not provide a global customer GST switch.

---

# 43. Pickup Details Screen

```text
--------------------------------
Pickup Details

Pickup Address
[________________________]
[________________________]

PIN Code
[________________________]

Pickup Date
[ Select Date ]

Pickup Time
[ Select Time ]

Customer Note
[________________________]

[ Continue ]
--------------------------------
```

---

# 44. Review Order Screen

Before final confirmation:

```text
--------------------------------
Review Order

Customer
Rahul Sharma

Business
Hotel Paradise

Pickup Address
...

Items
Shirt — Wash + Iron
5 × ₹50 = ₹250

Pant — Wash
2 × ₹40 = ₹80

Subtotal                  ₹330
GST 18%                    ₹59.40
Estimated Total           ₹389.40

[ Confirm Order ]
[ Edit Order ]
--------------------------------
```

---

# 45. Order Confirmation

After successful local creation:

```text
--------------------------------
Order Created Successfully

ORD1052

Your order has been received.

Status:
New

Estimated Amount:
₹389.40

[ View Order ]

[ Back to Home ]
--------------------------------
```

The order should be visible immediately even when offline.

---

# 46. Offline Order Confirmation

When offline:

```text
--------------------------------
Order Saved

ORD1052

Your order has been saved on this
device and will sync automatically
when internet is available.

Status:
Waiting to Sync

[ View Order ]
--------------------------------
```

Do not tell the customer that the cloud has received it if synchronization has not occurred.

---

# 47. Customer Profile Screen

```text
--------------------------------
Profile

Rahul Sharma
rahul@example.com

Mobile Number
98XXXXXXXX

Customer Type
Business

Business Name
Hotel Paradise

Business Type
Hotel

Address
...

PIN Code
...

[ Edit Profile ]

[ Change Password ]

[ Logout ]
--------------------------------
```

---

# 48. Edit Profile

Customer can edit:

- Name
- Mobile Number
- Customer Type
- Address
- PIN Code
- Business information

Protected fields such as:

```text
Role
Business ID
User ID
```

must not be editable.

---

# 49. Changing Business to Personal

If customer changes:

```text
Business → Personal
```

show confirmation:

> Switching to Personal will remove your current business information from your profile. Your previous orders will remain unchanged.

Buttons:

```text
[ Cancel ]
[ Continue ]
```

---

# 50. Customer Logout

Confirmation:

> Are you sure you want to log out?

Buttons:

```text
Cancel
Logout
```

---

# 51. Staff Home

```text
--------------------------------
Good Morning

Today's Work

New Orders             5
Pickup Pending         3
Processing             8
Ready                  4
Out for Delivery       2
Unpaid                 6

[ Create Order ]

[ View Orders ]
--------------------------------

Home | Orders | Profile
--------------------------------
```

---

# 52. Staff Orders Screen

```text
--------------------------------
Orders

[ Search ]

[ New ] [ Pickup ]
[ Processing ] [ Ready ]
[ Delivered ] [ Unpaid ]

ORD1052
Rahul Sharma
Hotel Paradise
Processing
₹590
--------------------------------
```

Filters can scroll horizontally if needed.

---

# 53. Staff Search

Search placeholder:

```text
Search order, customer or mobile
```

Search by:

- Order ID
- Invoice Number
- Customer Name
- Mobile Number

---

# 54. Staff Create Order

Primary CTA:

```text
[ + Create Order ]
```

Flow:

```text
Select Customer
 ↓
Items
 ↓
Services
 ↓
Quantity
 ↓
Pickup/Delivery
 ↓
GST
 ↓
Payment
 ↓
Review
 ↓
Create
```

---

# 55. Staff Customer Selection

```text
--------------------------------
Select Customer

[ Search name or mobile ]

Rahul Sharma
98XXXXXXXX
Hotel Paradise

Priya Singh
97XXXXXXXX
Personal

[ + New Customer ]
--------------------------------
```

---

# 56. Staff New Customer

Minimal required information should be collected so the order can be created quickly.

Fields:

```text
Name
Mobile Number
Customer Type
Business information if applicable
Address
PIN Code
```

---

# 57. Staff Order Entry

Staff should be able to add multiple item/service combinations.

Example:

```text
Shirt
Wash + Iron
Quantity: 5

[ + Add Item ]
```

---

# 58. Staff Payment Screen

```text
--------------------------------
Payment

Order Total
₹590

Payment Amount
[ ₹200 ]

Payment Method

○ Cash
○ UPI
○ Online

Note
[________________]

[ Record Payment ]
[ Skip Payment ]
--------------------------------
```

---

# 59. Staff Payment Behavior

If payment is skipped:

```text
Payment Status = PENDING
```

If partial:

```text
Payment Status = PARTIALLY_PAID
```

If full:

```text
Payment Status = PAID
```

---

# 60. Staff Order Detail

```text
--------------------------------
ORD1052

Rahul Sharma
Hotel Paradise
98XXXXXXXX

Processing

Items
...

Amount
Subtotal
GST
Final Amount

Payment
Paid
Due
Status

Order Timeline
...

[ Update Status ]

[ Record Payment ]

[ Generate Invoice ]
--------------------------------
```

---

# 61. Staff Update Status

Use a bottom sheet/modal:

```text
Update Order Status

Current:
PROCESSING

Next Status:

[ Ready ]

[ Cancel Order ]
```

Only valid transitions should be offered.

Do not show invalid status jumps as normal choices.

---

# 62. Ready Order

When order is:

```text
READY
```

staff should have:

```text
[ Mark Out for Delivery ]
```

or appropriate next action.

---

# 63. Delivery

When:

```text
OUT_FOR_DELIVERY
```

show:

```text
[ Mark Delivered ]
```

---

# 64. Cancel Order

Cancellation should require confirmation.

```text
Cancel this order?

This action will mark the order
as Cancelled.

[ Keep Order ]
[ Cancel Order ]
```

If a cancellation reason is required by business policy, provide:

```text
Cancellation Reason
```

---

# 65. Staff Profile

```text
--------------------------------
Profile

Staff Name
Staff ID

Mobile Number
Email

Status
Active

[ Change Password ]

[ Logout ]
--------------------------------
```

Staff must not see Admin configuration controls.

---

# 66. First Staff Login

If temporary password is assigned:

```text
--------------------------------
Change Password

For security, please create a
new password before continuing.

Current Password
[____________]

New Password
[____________]

Confirm Password
[____________]

[ Change Password ]
--------------------------------
```

---

# 67. Admin Home

Admin dashboard:

```text
--------------------------------
Good Morning, Admin

Today

Orders
24

Revenue
₹8,450

Pending Orders
8

Ready
5

Unpaid
7

--------------------------------
Order Status

New             4
Pickup          3
Processing      6
Ready           5
Delivery        3
Delivered       3

--------------------------------
[ View Orders ]
--------------------------------

Home | Orders | People | Reports | More
```

---

# 68. Admin Orders

Admin uses the same operational order interface as Staff, with additional administrative controls where authorized.

Features:

- Search
- Filter
- View order
- Update status
- Payment
- Invoice
- Customer information

---

# 69. Admin People Screen

```text
--------------------------------
People

[ Staff ]
[ Customers ]
--------------------------------
```

Two clear sections/cards.

---

# 70. Admin Staff List

```text
--------------------------------
Staff

Active (6)

Rahul
Staff ID: ST001
Active

Amit
Staff ID: ST002
Active

...

[ + Add Staff ]
--------------------------------
```

Tabs/filter:

```text
Active
Inactive
```

---

# 71. Add Staff Screen

```text
--------------------------------
Add Staff

Full Name
[________________]

Mobile Number
[________________]

Staff ID / Login Identifier
[________________]

Temporary Password
[________________]

[ Create Staff ]
--------------------------------
```

The final implementation may use the chosen Firebase login identifier strategy, but the UI must clearly communicate the staff's login credentials.

---

# 72. Staff Creation Confirmation

```text
Staff account created successfully.

Staff:
Amit Kumar

Staff ID:
ST002

The staff member must use the
provided login information and
change the temporary password
during first login.
```

Do not expose sensitive credentials unnecessarily after creation.

---

# 73. Staff Detail Screen

```text
--------------------------------
Amit Kumar

Staff ID
ST002

Mobile Number
98XXXXXXXX

Status
Active

Created:
14 September 2026

[ Edit ]

[ Deactivate ]
--------------------------------
```

---

# 74. Deactivate Staff

Confirmation:

> Deactivate this staff account?

> The staff member will no longer be able to use the application. Historical records will remain unchanged.

Buttons:

```text
[ Cancel ]
[ Deactivate ]
```

---

# 75. Reactivate Staff

Confirmation:

> Reactivate this staff account?

Buttons:

```text
[ Cancel ]
[ Reactivate ]
```

---

# 76. Admin Customer List

```text
--------------------------------
Customers

[ Search customers ]

Rahul Sharma
98XXXXXXXX
Hotel Paradise

Priya Singh
97XXXXXXXX
Personal

Amit Verma
99XXXXXXXX
Hotel Sunrise
--------------------------------
```

Search by:

- Name
- Mobile
- Email
- Business Name

---

# 77. Admin Customer Detail

```text
--------------------------------
Rahul Sharma

rahul@example.com
98XXXXXXXX

Business
Hotel Paradise
Hotel

Address
...

--------------------------------
Orders

Total Orders        25
Completed           20
Cancelled            1

--------------------------------
Billing

Total Billed       ₹25,400
Total Paid         ₹21,400
Pending             ₹4,000

[ View Orders ]
--------------------------------
```

---

# 78. Admin Items Screen

Accessible through:

```text
More → Items
```

Layout:

```text
--------------------------------
Items

Shirt                 Active
Pant                  Active
Jeans                 Active
Saree                 Active
Bedsheet              Inactive

[ + Add Item ]
--------------------------------
```

---

# 79. Add Item

```text
Item Name
[________________]

Status
Active

[ Save ]
```

---

# 80. Edit Item

Admin can change:

```text
Item Name
Status
```

If historical orders reference the item, do not delete it.

---

# 81. Admin Services Screen

```text
--------------------------------
Services

Wash
Wash + Iron
Iron
Dry Clean

[ + Add Service ]
--------------------------------
```

---

# 82. Add Service

```text
Service Name
[________________]

Status
Active

[ Save ]
```

---

# 83. Admin Pricing Screen

Pricing must clearly show Item + Service.

Example:

```text
--------------------------------
Pricing

Shirt
Wash              ₹30
Wash + Iron       ₹50
Iron              ₹20

Pant
Wash              ₹40
Wash + Iron       ₹60

[ + Add Price ]
--------------------------------
```

---

# 84. Add/Edit Price

```text
Item
[ Select Item ]

Service
[ Select Service ]

Price
[ ₹ ______ ]

Status
Active

[ Save Price ]
```

---

# 85. Price Warning

When changing a price:

> This price will apply to new orders. Existing orders will keep their original price.

Buttons:

```text
[ Cancel ]
[ Save Price ]
```

---

# 86. Admin GST Settings

Accessible through:

```text
More → Business Settings → GST
```

Screen:

```text
--------------------------------
GST Settings

GSTIN
[________________]

Default GST Rate
[ 18 % ]

Note:
GST can be selected separately
for each order.

[ Save ]
--------------------------------
```

The default rate does not mean every order automatically includes GST.

---

# 87. Business Settings Screen

```text
--------------------------------
Business Settings

Business Name
[________________]

Address
[________________]

Phone
[________________]

Email
[________________]

GSTIN
[________________]

Default GST Rate
[________________]

Invoice Prefix
[________________]

[ Save ]
--------------------------------
```

---

# 88. Invoice Settings

```text
--------------------------------
Invoice Settings

Invoice Prefix
[ INV ]

Footer Message
[ Thank you for your business! ]

[ Save ]
--------------------------------
```

Business details may be displayed as part of the same settings area if desired.

---

# 89. Admin More Screen

Suggested structure:

```text
--------------------------------
More

Business Settings
Items
Services
Pricing
GST
Invoice Settings
Sync Status
App Settings
Logout
--------------------------------
```

Do not overload this page with unrelated future features.

---

# 90. Reports Screen

```text
--------------------------------
Reports

Weekly
Monthly
Yearly
--------------------------------
```

Three clear options.

---

# 91. Weekly Report Screen

```text
--------------------------------
Weekly Report

Month
[ September ▼ ]

Year
[ 2026 ▼ ]

From Date
[ 07 Sep 2026 ]

To Date
[ 13 Sep 2026 ]

[ Generate Report ]
--------------------------------
```

---

# 92. Monthly Report Screen

```text
--------------------------------
Monthly Report

Month
[ September ▼ ]

Year
[ 2026 ▼ ]

[ Generate Report ]
--------------------------------
```

---

# 93. Yearly Report Screen

```text
--------------------------------
Yearly Report

Year
[ 2026 ▼ ]

[ Generate Report ]
--------------------------------
```

---

# 94. Report Preview

Before generating/sharing, show summary:

```text
--------------------------------
September 2026

Total Orders             245
Completed                210
Cancelled                  8
Active                    27

Subtotal             ₹85,000
GST                   ₹15,300
Grand Total          ₹100,300

Paid                  ₹92,000
Pending                ₹8,300

Cash                  ₹40,000
UPI                   ₹45,000
Online                 ₹7,000

[ Generate Excel ]
--------------------------------
```

---

# 95. Report Generation

When generating:

```text
Generating report...
```

Then:

```text
Report generated successfully.

Laundry_Report_September_2026.xlsx

[ Open ]
[ Share ]
```

---

# 96. Offline Report Message

If report uses only local data:

> Report generated from data available on this device. Some newer data may still be waiting to sync.

---

# 97. Sync Status Screen

Accessible from More.

```text
--------------------------------
Sync Status

Connection
Offline

Pending
3 items

Syncing
0

Failed
1

Last successful sync
14 Sep 2026, 4:15 PM

[ Retry Sync ]
--------------------------------
```

---

# 98. Order Status Colors

The design system may assign distinct visual treatments to:

```text
NEW
PICKUP
PROCESSING
READY
OUT FOR DELIVERY
DELIVERED
CANCELLED
```

However:

> Do not rely on color alone.

Always display the status text.

---

# 99. Payment Status UI

Use text:

```text
Paid
Partially Paid
Pending
```

Payment amount should always be visible when relevant.

---

# 100. Confirmation Dialog Rules

Destructive or important operations require confirmation.

Examples:

- Deactivate Staff
- Reactivate Staff where appropriate
- Cancel Order
- Change Business → Personal
- Logout where appropriate
- Important price changes

Normal navigation should not require confirmation.

---

# 101. Bottom Sheets

Use bottom sheets for compact selections such as:

- Order status
- Payment method
- Item/service selection
- Filters

Avoid opening unnecessary full-screen pages.

---

# 102. Dropdowns

Use dropdown/select components for:

- Business Type
- Month
- Year
- GST rate where configured
- Service
- Item
- Payment Method where appropriate

---

# 103. Search UX

Search field:

```text
🔍 Search...
```

Should support:

- Typing
- Clear button
- Immediate/local results
- Empty result state

---

# 104. Filter UX

Filter button may open:

```text
--------------------------------
Filters

Order Status
☐ New
☐ Pickup
☐ Processing
☐ Ready
☐ Delivered
☐ Cancelled

Payment Status
☐ Paid
☐ Partially Paid
☐ Pending

[ Clear ]
[ Apply ]
--------------------------------
```

---

# 105. Combined Filter

Example:

```text
Order Status:
Delivered

Payment:
Pending
```

Result:

```text
Delivered + Pending Payment
```

---

# 106. Order Detail Actions

Actions should depend on role.

### Customer

```text
View Invoice
```

### Staff

```text
Update Status
Record Payment
Generate Invoice
```

### Admin

```text
Update Status
Record Payment
Generate Invoice
```

---

# 107. Invoice Screen

After generating/opening invoice:

```text
--------------------------------
Invoice

INV1025

ORD1025

14 September 2026

Customer
Rahul Sharma
Hotel Paradise

...

Total              ₹590
Paid               ₹200
Pending            ₹390

[ Generate PDF ]
[ Share ]
--------------------------------
```

---

# 108. Invoice Permissions

Customer:

- Own invoices only

Staff:

- Operationally permitted invoices

Admin:

- All business invoices

---

# 109. Offline Invoice

If complete required order/payment/business information exists locally:

```text
Generate PDF
```

can work offline.

---

# 110. Invoice Failure

Message:

> Unable to generate the invoice right now. Please try again.

Do not display raw PDF/library exceptions.

---

# 111. Customer Order Creation — Progress Indicator

Multi-step order flow should show progress.

Example:

```text
1 Items
2 Service
3 Pickup
4 Review
```

Current step is highlighted.

---

# 112. Back Navigation

When moving backward through an order form:

- Preserve entered information
- Do not reset the entire order
- Recalculate totals when relevant

---

# 113. Unsaved Form Protection

If user attempts to leave a form with important unsaved data:

> Your changes have not been saved. Do you want to leave?

Buttons:

```text
Stay
Leave
```

---

# 114. Order Draft Handling

The app may maintain temporary order form state locally during the current order creation process.

A confirmed order must be persisted to SQLite.

---

# 115. Double Submission Prevention

Buttons such as:

```text
Create Account
Login
Save
Create Order
Record Payment
Generate Report
```

must become disabled while the operation is being committed.

---

# 116. Accessibility

All interactive elements should have:

- Accessible labels
- Adequate touch area
- Readable text
- Visible focus/pressed state where applicable

Do not communicate important information using color only.

---

# 117. Touch Target

Interactive controls should be comfortably tappable.

Avoid tiny:

```text
icon-only
```

controls when the action is important.

---

# 118. Long Text Handling

The UI must support:

- Long customer names
- Long business names
- Long addresses
- Long notes
- Long email addresses

Use wrapping/truncation carefully.

Never allow text to overlap another component.

---

# 119. Keyboard Handling

Forms must automatically handle the Android keyboard.

The keyboard must not cover:

- Active input
- Save button
- Continue button
- Important validation errors

Use appropriate scrolling behavior.

---

# 120. Mobile Number Input

Use numeric keyboard.

Validate according to Indian mobile number requirements.

Display:

```text
Mobile Number
```

not:

```text
Phone #
```

---

# 121. PIN Code Input

Use numeric keyboard.

Label:

```text
PIN Code
```

---

# 122. Quantity Input

Use numeric keyboard.

Minimum:

```text
1
```

Do not allow negative quantities.

---

# 123. Amount Input

Currency amounts should use numeric/decimal input as appropriate.

Do not allow invalid negative payment amounts.

---

# 124. Order Status UX

Status should be easy for Staff to update.

Preferred approach:

```text
Current Status
     ↓
Next Valid Action
```

Example:

```text
PROCESSING

[ Mark Ready ]
```

rather than forcing staff to navigate through a large dropdown every time.

---

# 125. Admin Order Status

Admin can access status controls but should still use the same valid transition rules.

---

# 126. Cancelled Order UI

Cancelled order should clearly display:

```text
CANCELLED
```

and should not appear as active.

Historical information remains accessible.

---

# 127. Customer Home Empty State

If there are no orders:

```text
No orders yet.

Place your first laundry order today.

[ Place New Order ]
```

---

# 128. Staff Empty State

If no pending work:

```text
No pending work.

All current orders are up to date.
```

---

# 129. Admin Empty State

If no customers:

```text
No customers found.
```

If no staff:

```text
No staff members found.

[ Add Staff ]
```

---

# 130. Offline Global UX

When offline:

The app should remain usable where local data is available.

Do not display a full-screen:

```text
No Internet
```

blocking page.

Instead:

```text
Offline
```

indicator + local data.

---

# 131. Sync Failure UX

If synchronization fails:

```text
Some data is waiting to sync.
```

Optional:

```text
[ Retry ]
```

The local record must remain available.

---

# 132. Data Freshness Indicator

Where useful, show:

```text
Last synced:
14 Sep 2026, 4:15 PM
```

especially on Admin Sync Status.

---

# 133. Profile Save Offline

If supported by the offline model:

```text
Save Profile
 ↓
SQLite
 ↓
Pending Sync
```

If a particular profile operation requires cloud authorization, the UI must clearly show pending synchronization rather than falsely reporting cloud completion.

---

# 134. Master Data Offline

Items, Services and Prices shown offline must be based on the latest successfully synchronized local copy.

Do not display a misleading:

```text
Live
```

indicator when the device is offline.

---

# 135. Order Card Design

Recommended structure:

```text
┌──────────────────────────────┐
│ ORD1052            Processing│
│ Rahul Sharma                 │
│ Hotel Paradise               │
│ 14 Sep 2026                  │
│                              │
│ Total             ₹590       │
│ Payment: Pending             │
└──────────────────────────────┘
```

The card should be tappable.

---

# 136. Customer Business Display

If customer is Business:

```text
Rahul Sharma
Hotel Paradise
```

If Personal:

```text
Rahul Sharma
Personal
```

Do not show empty business labels.

---

# 137. Order Customer Display

Order cards should show business information when available.

Example:

```text
Rahul Sharma
Hotel Paradise
```

This is especially important for business customers.

---

# 138. Customer Type Badge

Possible display:

```text
PERSONAL
```

or:

```text
BUSINESS
```

Use a small text badge/chip.

---

# 139. Payment Summary Component

Reusable component:

```text
Subtotal                 ₹500
GST 18%                   ₹90
Grand Total              ₹590
Paid                     ₹200
Pending                  ₹390
```

Use the same component across:

- Order Detail
- Invoice preview
- Payment screen
- Customer history where relevant

---

# 140. Order Timeline Component

Reusable across Customer, Staff and Admin.

Must display:

- Status
- Timestamp
- Changed by where appropriate for Staff/Admin

Customer may not need internal staff identity details.

---

# 141. Role-Specific Information

Do not show unnecessary internal information to Customers.

Example:

Staff/Admin may see:

```text
Changed by Amit
```

Customer may simply see:

```text
Processing
14 Sep, 3:20 PM
```

---

# 142. Admin Reports Navigation

Bottom navigation:

```text
Home
Orders
People
Reports
More
```

Reports should be directly accessible.

---

# 143. Admin More Navigation

More should not become a dumping ground.

Group settings logically:

```text
Business
Operations
Account
```

Example:

```text
Business Settings
GST
Invoice Settings

Items
Services
Pricing

Sync Status
App Settings

Logout
```

---

# 144. Profile Security

Password fields should:

- Be masked
- Support show/hide
- Never display existing passwords

---

# 145. Permission UI

If an action is unavailable because of role:

Prefer hiding the action entirely.

If the user reaches a protected operation through another route:

> You do not have permission to perform this action.

---

# 146. Customer Access Boundary

Customer must never see:

- Staff list
- Admin dashboard
- Pricing management
- GST business settings
- Other customers
- Business settings
- Reports

---

# 147. Staff Access Boundary

Staff must never see or modify:

- Staff management
- Pricing management
- GST business configuration
- Business settings
- Admin controls

unless a future explicit permission system adds them.

---

# 148. Admin Access

Admin has access to all V1 business operations.

---

# 149. Logout Flow

All roles:

```text
Profile/More
 ↓
Logout
 ↓
Confirmation
 ↓
Login Screen
```

---

# 150. Session Expiry

If Firebase authentication session becomes invalid:

```text
Session expired.
Please log in again.
```

The application must safely return to Login.

Any locally queued data must not be silently deleted because of session expiry.

---

# 151. Account Inactive

If an account becomes inactive:

```text
Your account is inactive.
Please contact the administrator.
```

Do not leave the user stuck on a blank/loading screen.

---

# 152. Network Timeout

If a cloud operation times out but local operation has already succeeded:

Do not create another record automatically.

Show:

> Your data is saved on this device and is waiting to sync.

---

# 153. Sync Badge

Optional global badge:

```text
↻ 3
```

indicating pending synchronization.

Tapping it can open Sync Status.

---

# 154. Important UX Rule

The application must clearly distinguish:

```text
Saved Locally
```

from:

```text
Synced to Cloud
```

These are not the same state.

---

# 155. Customer Order State Labels

Customer-facing labels should be simple:

```text
New
Pickup Pending
Picked Up
Processing
Ready
Out for Delivery
Delivered
Cancelled
```

---

# 156. Internal Status Values

The UI must map internal enum values to readable labels.

Example:

```text
PICKUP_PENDING
```

display:

```text
Pickup Pending
```

Never display raw enum values to normal users.

---

# 157. Payment Labels

Internal:

```text
PARTIALLY_PAID
```

Display:

```text
Partially Paid
```

Internal:

```text
PENDING
```

Display:

```text
Pending
```

---

# 158. Error Copy Standards

Use short sentences.

Good:

> Please enter a valid mobile number.

Bad:

> Error: validation failed for phone field.

Good:

> Unable to save the order. Your local data is safe.

Bad:

> SQLite transaction exception code 19.

---

# 159. Success Message Standards

Good:

> Order created successfully.

> Payment recorded successfully.

> Staff account created successfully.

> Price updated successfully.

> Report generated successfully.

---

# 160. Toast/Snackbar Usage

Use snackbar/toast for lightweight confirmation.

Example:

```text
Payment recorded successfully.
```

Do not use toast for critical information that the user must read carefully.

---

# 161. Confirmation Dialog Usage

Use dialog for:

- Destructive action
- Important irreversible/semantically significant action
- Switching customer type where data is cleared

---

# 162. Form Error Placement

Validation error should appear near the relevant field.

Example:

```text
Mobile Number
[ 9876 ]

Please enter a valid mobile number.
```

---

# 163. Button Hierarchy

Primary button:

```text
[ Confirm Order ]
```

Secondary:

```text
[ Edit Order ]
```

Destructive:

```text
[ Cancel Order ]
```

Avoid multiple equally prominent primary buttons on the same screen.

---

# 164. Customer Primary Actions

Most important:

```text
Place New Order
```

---

# 165. Staff Primary Actions

Most important:

```text
Create Order
Update Status
Record Payment
```

depending on context.

---

# 166. Admin Primary Actions

Most important:

```text
Manage Orders
Manage People
Generate Reports
```

---

# 167. Screen Transition Principle

Navigation should follow user intent.

Customer:

```text
Home → Order → Review → Confirmation
```

Staff:

```text
Orders → Order → Action
```

Admin:

```text
People → Staff/Customer → Detail
```

Avoid unnecessary intermediate screens.

---

# 168. Deep Navigation Limit

Frequently used operational tasks should ideally require no more than a few meaningful taps.

Example:

```text
Staff Home
→ Orders
→ Order
→ Mark Ready
```

---

# 169. UI Component Reuse

Create reusable components for:

- Buttons
- Inputs
- Cards
- Status chips
- Payment summary
- Order cards
- Empty states
- Loading states
- Error states
- Confirmation dialogs
- Search bars
- Filter sheets
- Timeline

---

# 170. Design System Consistency

The same component must look and behave consistently throughout:

```text
Customer
Staff
Admin
```

Only role-specific content/actions should differ.

---

# 171. No Unnecessary Complexity

Do not introduce:

- Drawer navigation unless required
- Nested tabs everywhere
- Complex dashboards
- Excessive charts
- Animated onboarding
- Gamification
- AI assistant

These are outside V1.

---

# 172. Final Navigation Map

```text
APP
│
├── AUTH
│   ├── Login
│   ├── Register
│   ├── Verify Email
│   ├── Complete Profile
│   └── Forgot Password
│
├── CUSTOMER
│   ├── Home
│   ├── Orders
│   │   └── Order Detail
│   └── Profile
│       └── Edit Profile
│
├── STAFF
│   ├── Home
│   ├── Orders
│   │   ├── Order Detail
│   │   ├── Create Order
│   │   └── Payment
│   └── Profile
│
└── ADMIN
    ├── Home
    ├── Orders
    │   └── Order Detail
    ├── People
    │   ├── Staff
    │   │   ├── Add Staff
    │   │   └── Staff Detail
    │   └── Customers
    │       └── Customer Detail
    ├── Reports
    │   ├── Weekly
    │   ├── Monthly
    │   └── Yearly
    └── More
        ├── Business Settings
        ├── Items
        ├── Services
        ├── Pricing
        ├── GST
        ├── Invoice Settings
        └── Sync Status
```

---

# 173. Complete Customer Journey

```text
Install App
 ↓
Register
 ↓
Email + Password
 ↓
Verify Email
 ↓
Complete Profile
 ↓
Customer Home
 ↓
Place New Order
 ↓
Select Items
 ↓
Select Services
 ↓
Quantity
 ↓
Pickup Details
 ↓
GST
 ↓
Review
 ↓
Confirm
 ↓
Order Created
 ↓
Track Status
 ↓
Invoice
 ↓
Payment
```

---

# 174. Complete Staff Journey

```text
Staff Created by Admin
 ↓
Login
 ↓
Change Temporary Password
 ↓
Staff Home
 ↓
Orders
 ↓
Create / Open Order
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
 ↓
Record Payment
 ↓
Generate Invoice
```

---

# 175. Complete Admin Journey

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
Order Detail
 ↓
Status / Payment / Invoice

Reports
 ↓
Weekly / Monthly / Yearly

More
 ↓
Items
Services
Pricing
GST
Business Settings
Invoice Settings
```

---

# 176. Complete Offline UX

```text
User Action
 ↓
Validate
 ↓
Save SQLite
 ↓
Update UI
 ↓
Show Local Saved State
 ↓
Sync Queue
 ↓
Network Available
 ↓
Sync Firebase
 ↓
Show Synced
```

If sync fails:

```text
Keep Local Data
 ↓
Show Waiting to Sync
 ↓
Retry
```

---

# 177. Final UX Principles

The final UI must satisfy these rules:

```text
1. Simple
2. Fast
3. Clear
4. Mobile-first
5. Offline-friendly
6. Role-aware
7. Indian English
8. ₹ currency
9. No raw technical errors
10. No unnecessary complexity
11. Historical information remains clear
12. Important actions require confirmation
13. Long text must wrap
14. No clipping/overlap
15. Saved locally ≠ synced to cloud
```

---

# 178. UI/UX Source-of-Truth Rule

The project documentation hierarchy is:

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

`TRD.md` defines **how it should technically work**.

`UI-UX.md` defines **how users interact with it**.

`BACKEND-SCHEMA.md` will define **exactly how the data is stored and related**.

Any implementation must remain consistent with all four documents.