# Complete cURL Commands for All Ecommerce Services

This document provides all cURL commands to test each service in all business models.

**Base URLs (configured in `config/platform.envvars`):**
- Via Domain (Nginx): `http://$DOMAIN:$PUBLIC_PORT`
- Direct Access: `http://localhost:$B2C_GATEWAY_PORT` (B2C), `:$B2B_GATEWAY_PORT` (B2B), `:$C2C_GATEWAY_PORT` (C2C), `:$C2B_GATEWAY_PORT` (C2B)

Tip: in your shell you can load them once:
`set -a && . /home/rp/ecommerce/config/platform.envvars && set +a`

---

## B2C (Business to Consumer)
**Gateway Port:** 8001 | **Domain Path:** `/b2c` | **Domain Subdomain:** `b2c.kriss.karkark.com:24477`

### Authentication Service (`/auth`)

```bash
# Health Check
curl http://localhost:8001/auth/health
curl http://kriss.karkark.com:24477/b2c/auth/health

# Register User
curl -X POST http://localhost:8001/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"secure123","name":"John Doe"}'

# Login
curl -X POST http://localhost:8001/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"secure123"}'

# Social Login (Google/Facebook)
curl -X POST http://localhost:8001/auth/login/social \
  -H "Content-Type: application/json" \
  -d '{"provider":"google","accessToken":"token-123"}'

# Password Recovery
curl -X POST http://localhost:8001/auth/password/recover \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com"}'

# Password Reset
curl -X POST http://localhost:8001/auth/password/reset \
  -H "Content-Type: application/json" \
  -d '{"token":"reset-token-123","newPassword":"newpass123"}'

# Get Session
curl http://localhost:8001/auth/session/session-123
```

### Payment Service (`/payments`)

```bash
# Health Check
curl http://localhost:8001/payments/health

# Process Payment
curl -X POST http://localhost:8001/payments/payments \
  -H "Content-Type: application/json" \
  -d '{
    "orderId":"order-123",
    "amount":99.99,
    "currency":"USD",
    "paymentMethod":"stripe",
    "cardData":{"number":"4242424242424242","expiry":"12/25"},
    "requires3DSecure":true
  }'

# Complete 3D Secure
curl -X POST http://localhost:8001/payments/payments/payment-123/3d-secure/complete \
  -H "Content-Type: application/json" \
  -d '{"verificationCode":"123456"}'

# Get Payment Status
curl http://localhost:8001/payments/payments/payment-123

# Get Token Info
curl http://localhost:8001/payments/tokens/token-123

# Webhook (Stripe)
curl -X POST http://localhost:8001/payments/webhooks/stripe \
  -H "Content-Type: application/json" \
  -d '{"type":"payment.succeeded","data":{"id":"stripe-123"}}'
```

### Search Service (`/search`)

```bash
# Health Check
curl http://localhost:8001/search/health

# Basic Search
curl "http://localhost:8001/search/search?q=laptop&category=Electronics&minPrice=500&maxPrice=2000&minRating=4.0"

# Autocomplete
curl "http://localhost:8001/search/autocomplete?q=lap"

# Advanced Search
curl -X POST http://localhost:8001/search/search/advanced \
  -H "Content-Type: application/json" \
  -d '{
    "query":"laptop",
    "filters":{"category":"Electronics","brand":"Dell"},
    "sort":"price_asc",
    "page":1,
    "limit":20
  }'

# Index Product (Admin)
curl -X POST http://localhost:8001/search/index \
  -H "Content-Type: application/json" \
  -d '{
    "id":"prod-1",
    "name":"Laptop",
    "category":"Electronics",
    "price":999,
    "rating":4.5,
    "description":"Powerful laptop"
  }'
```

### Catalog Service (`/catalog`)

```bash
# Get Products
curl http://localhost:8001/catalog/products
```

### Cart Service (`/cart`)

```bash
# Health Check
curl http://localhost:8001/cart/health

# Get Cart
curl http://localhost:8001/cart?userId=user-123

# Add to Cart
curl -X POST http://localhost:8001/cart \
  -H "Content-Type: application/json" \
  -d '{"userId":"user-123","productId":"prod-1","quantity":2}'

# Update Cart Item
curl -X PUT http://localhost:8001/cart/user-123/item-1 \
  -H "Content-Type: application/json" \
  -d '{"quantity":3}'

# Remove from Cart
curl -X DELETE http://localhost:8001/cart/user-123/item-1
```

### Order Service (`/orders`)

```bash
# Health Check
curl http://localhost:8001/orders/health

# Create Order
curl -X POST http://localhost:8001/orders \
  -H "Content-Type: application/json" \
  -d '{
    "customerId":"user-123",
    "items":[{"productId":"prod-1","quantity":2,"price":99.99}]
  }'

# Get Orders
curl http://localhost:8001/orders
```

### Checkout Service (`/checkout`)

```bash
# Health Check
curl http://localhost:8001/checkout/health
curl http://kriss.karkark.com:24477/b2c/checkout/health

# Initiate Checkout
curl -X POST http://localhost:8001/checkout/checkout \
  -H "Content-Type: application/json" \
  -d '{
    "customerId":"user-123",
    "cartId":"cart-123",
    "shippingAddress":{"street":"123 Main St","city":"New York","zip":"10001"},
    "billingAddress":{"street":"123 Main St","city":"New York","zip":"10001"}
  }'

# Get Checkout Summary
curl http://localhost:8001/checkout/checkout/checkout-123/summary

# Complete Checkout
curl -X POST http://localhost:8001/checkout/checkout/checkout-123/complete \
  -H "Content-Type: application/json" \
  -d '{"paymentId":"payment-123","orderId":"order-123"}'

# Get Checkout by ID
curl http://localhost:8001/checkout/checkout/checkout-123
```

### Shipping Service (`/shipping`)

```bash
# Health Check
curl http://localhost:8001/shipping/health
curl http://kriss.karkark.com:24477/b2c/shipping/health

# Get Available Shipping Methods
curl "http://localhost:8001/shipping/methods?address=New+York&weight=2.5&dimensions=10x10x5"

# Calculate Shipping Cost
curl -X POST http://localhost:8001/shipping/calculate \
  -H "Content-Type: application/json" \
  -d '{
    "address":{"street":"123 Main St","city":"New York","zip":"10001"},
    "items":[{"productId":"prod-1","weight":1.5},{"productId":"prod-2","weight":1.0}],
    "shippingMethod":"standard"
  }'

# Create Shipment
curl -X POST http://localhost:8001/shipping/shipments \
  -H "Content-Type: application/json" \
  -d '{
    "orderId":"order-123",
    "shippingMethod":"express",
    "address":{"street":"123 Main St","city":"New York","zip":"10001"}
  }'

# Update Shipment Status
curl -X PATCH http://localhost:8001/shipping/shipments/shipment-123 \
  -H "Content-Type: application/json" \
  -d '{"status":"IN_TRANSIT","location":"Distribution Center"}'

# Get Shipment
curl http://localhost:8001/shipping/shipments/shipment-123

# Track Shipment
curl http://localhost:8001/shipping/shipments/shipment-123/track
```

### Reviews & Ratings Service (`/reviews`)

```bash
# Health Check
curl http://localhost:8001/reviews/health
curl http://kriss.karkark.com:24477/b2c/reviews/health

# Create Review
curl -X POST http://localhost:8001/reviews/reviews \
  -H "Content-Type: application/json" \
  -d '{
    "productId":"prod-1",
    "customerId":"user-123",
    "rating":5,
    "title":"Great Product!",
    "comment":"Very satisfied with the quality",
    "verifiedPurchase":true
  }'

# Get Reviews for Product
curl "http://localhost:8001/reviews/reviews/product/prod-1?status=APPROVED&sort=helpful&limit=10&offset=0"

# Mark Review as Helpful
curl -X POST http://localhost:8001/reviews/reviews/review-123/helpful

# Moderate Review (Admin)
curl -X PATCH http://localhost:8001/reviews/reviews/review-123/moderate \
  -H "Content-Type: application/json" \
  -d '{"status":"APPROVED","moderatorId":"admin-123"}'

# Get Review by ID
curl http://localhost:8001/reviews/reviews/review-123

# Get Customer's Reviews
curl http://localhost:8001/reviews/reviews/customer/user-123
```

### Wishlist Service (`/wishlist`)

```bash
# Health Check
curl http://localhost:8001/wishlist/health
curl http://kriss.karkark.com:24477/b2c/wishlist/health

# Get Customer's Wishlist
curl http://localhost:8001/wishlist/wishlist/user-123

# Add Item to Wishlist
curl -X POST http://localhost:8001/wishlist/wishlist/user-123/items \
  -H "Content-Type: application/json" \
  -d '{"productId":"prod-1","variantId":"variant-1"}'

# Remove Item from Wishlist
curl -X DELETE http://localhost:8001/wishlist/wishlist/user-123/items/item-123

# Clear Wishlist
curl -X DELETE http://localhost:8001/wishlist/wishlist/user-123

# Move Item to Cart
curl -X POST http://localhost:8001/wishlist/wishlist/user-123/items/item-123/move-to-cart \
  -H "Content-Type: application/json" \
  -d '{"quantity":1}'
```

### Promo Code / Discount Service (`/promo`)

```bash
# Health Check
curl http://localhost:8001/promo/health
curl http://kriss.karkark.com:24477/b2c/promo/health

# Create Promo Code (Admin)
curl -X POST http://localhost:8001/promo/promo-codes \
  -H "Content-Type: application/json" \
  -d '{
    "code":"SAVE20",
    "discountType":"percentage",
    "discountValue":20,
    "minPurchaseAmount":100,
    "maxDiscountAmount":50,
    "validFrom":"2026-01-01T00:00:00Z",
    "validUntil":"2026-12-31T23:59:59Z",
    "usageLimit":1000
  }'

# Validate Promo Code
curl -X POST http://localhost:8001/promo/promo-codes/validate \
  -H "Content-Type: application/json" \
  -d '{
    "code":"SAVE20",
    "customerId":"user-123",
    "cartTotal":150.00,
    "items":[{"productId":"prod-1","price":150.00}]
  }'

# Apply Promo Code
curl -X POST http://localhost:8001/promo/promo-codes/promo-123/apply

# Get All Promo Codes (Admin)
curl "http://localhost:8001/promo/promo-codes?status=ACTIVE"

# Get Promo Code by ID
curl http://localhost:8001/promo/promo-codes/promo-123

# Update Promo Code Status
curl -X PATCH http://localhost:8001/promo/promo-codes/promo-123 \
  -H "Content-Type: application/json" \
  -d '{"status":"INACTIVE"}'
```

### Customer Support / Live Chat Service (`/support`)

```bash
# Health Check
curl http://localhost:8001/support/health
curl http://kriss.karkark.com:24477/b2c/support/health

# Create Support Conversation
curl -X POST http://localhost:8001/support/conversations \
  -H "Content-Type: application/json" \
  -d '{
    "customerId":"user-123",
    "subject":"Order Issue",
    "category":"order",
    "priority":"high"
  }'

# Send Message
curl -X POST http://localhost:8001/support/conversations/conv-123/messages \
  -H "Content-Type: application/json" \
  -d '{
    "senderId":"user-123",
    "senderType":"customer",
    "message":"I have not received my order yet",
    "attachments":[]
  }'

# Get Conversation Messages
curl http://localhost:8001/support/conversations/conv-123/messages

# Get Customer Conversations
curl http://localhost:8001/support/conversations/customer/user-123

# Assign Conversation to Agent
curl -X POST http://localhost:8001/support/conversations/conv-123/assign \
  -H "Content-Type: application/json" \
  -d '{"agentId":"agent-123"}'

# Update Conversation Status
curl -X PATCH http://localhost:8001/support/conversations/conv-123 \
  -H "Content-Type: application/json" \
  -d '{"status":"IN_PROGRESS","priority":"urgent"}'

# Get All Conversations (Admin/Agent)
curl "http://localhost:8001/support/conversations?status=OPEN&assignedTo=agent-123"

# Get Conversation by ID
curl http://localhost:8001/support/conversations/conv-123
```

### Admin Dashboard Service (`/admin`)

```bash
# Health Check
curl http://localhost:8001/admin/health
curl http://kriss.karkark.com:24477/b2c/admin/health

# Dashboard Overview
curl "http://localhost:8001/admin/dashboard/overview?startDate=2026-01-01&endDate=2026-01-31"

# Sales Analytics
curl "http://localhost:8001/admin/dashboard/sales?period=daily&startDate=2026-01-01&endDate=2026-01-31"

# Customer Analytics
curl http://localhost:8001/admin/dashboard/customers

# Product Analytics
curl http://localhost:8001/admin/dashboard/products

# Order Management
curl "http://localhost:8001/admin/dashboard/orders?status=PENDING&limit=10&offset=0"

# Pending Reviews
curl http://localhost:8001/admin/dashboard/reviews/pending

# Support Overview
curl http://localhost:8001/admin/dashboard/support

# System Health
curl http://localhost:8001/admin/dashboard/system
```

---

## B2B (Business to Business)
**Gateway Port:** 8002 | **Domain Path:** `/b2b` | **Domain Subdomain:** `b2b.kriss.karkark.com:24477`

### Corporate Account Service (`/accounts`)

```bash
# Health Check
curl http://localhost:8002/accounts/health

# Register Corporate Account
curl -X POST http://localhost:8002/accounts/accounts/register \
  -H "Content-Type: application/json" \
  -d '{
    "companyName":"Acme Corp",
    "email":"admin@acme.com",
    "password":"secure123",
    "taxId":"TAX-12345"
  }'

# OAuth Callback
curl -X POST http://localhost:8002/accounts/accounts/oauth/callback \
  -H "Content-Type: application/json" \
  -d '{"code":"oauth-code-123","state":"state-123"}'

# Get Account by Tenant
curl http://localhost:8002/accounts/accounts/tenant/tenant-123
```

### KYC Verification Service (`/kyc`)

```bash
# Health Check
curl http://localhost:8002/kyc/health

# Submit KYC Verification
curl -X POST http://localhost:8002/kyc/verify \
  -H "Content-Type: application/json" \
  -d '{
    "accountId":"account-123",
    "documentType":"business_license",
    "documentUrl":"https://example.com/doc.pdf",
    "companyName":"Acme Corp"
  }'

# Get Verification Status
curl http://localhost:8002/kyc/verify/verification-123
```

### Bulk Pricing Service (`/pricing`)

```bash
# Health Check
curl http://localhost:8002/pricing/health

# Create Price Tier
curl -X POST http://localhost:8002/pricing/tiers \
  -H "Content-Type: application/json" \
  -d '{
    "productId":"prod-1",
    "tiers":[
      {"tier":"1","minQuantity":1,"maxQuantity":99,"price":100,"discount":0},
      {"tier":"2","minQuantity":100,"maxQuantity":499,"price":95,"discount":5},
      {"tier":"3","minQuantity":500,"price":90,"discount":10}
    ]
  }'

# Calculate Price
curl -X POST http://localhost:8002/pricing/calculate \
  -H "Content-Type: application/json" \
  -d '{"productId":"prod-1","quantity":250,"accountId":"account-123"}'
```

### MOQ Service (`/moq`)

```bash
# Health Check
curl http://localhost:8002/moq/health

# Create MOQ Rule
curl -X POST http://localhost:8002/moq/rules \
  -H "Content-Type: application/json" \
  -d '{
    "productId":"prod-1",
    "minQuantity":100,
    "enforcementLevel":"strict"
  }'

# Validate Order Quantity
curl -X POST http://localhost:8002/moq/validate \
  -H "Content-Type: application/json" \
  -d '{"productId":"prod-1","quantity":50}'

# Get MOQ Rule
curl http://localhost:8002/moq/rules/product/prod-1
```

### RFQ Service (`/rfq`)

```bash
# Health Check
curl http://localhost:8002/rfq/health

# Create RFQ
curl -X POST http://localhost:8002/rfq/rfqs \
  -H "Content-Type: application/json" \
  -d '{
    "buyerId":"buyer-123",
    "items":[{"productId":"prod-1","quantity":1000,"specifications":"Grade A"}],
    "deadline":"2026-02-01"
  }'

# Generate RFQ PDF
curl -X POST http://localhost:8002/rfq/rfqs/rfq-123/generate-pdf

# Search RFQs
curl "http://localhost:8002/rfq/rfqs/search?q=laptop&buyerId=buyer-123"

# Get All RFQs
curl http://localhost:8002/rfq/rfqs

# Get RFQ by ID
curl http://localhost:8002/rfq/rfqs/rfq-123
```

### Quotation Approval Service (`/quotations`)

```bash
# Health Check
curl http://localhost:8002/quotations/health

# Create Quotation
curl -X POST http://localhost:8002/quotations/quotations \
  -H "Content-Type: application/json" \
  -d '{
    "rfqId":"rfq-123",
    "supplierId":"supplier-123",
    "items":[{"productId":"prod-1","quantity":1000,"unitPrice":95,"totalPrice":95000}],
    "validUntil":"2026-02-15"
  }'

# Approve Quotation
curl -X POST http://localhost:8002/quotations/quotations/quotation-123/approve \
  -H "Content-Type: application/json" \
  -d '{
    "approverId":"approver-123",
    "role":"APPROVER",
    "approved":true,
    "comments":"Approved for purchase"
  }'

# Get Quotation
curl http://localhost:8002/quotations/quotations/quotation-123
```

### Purchase Order Service (`/purchase-orders`)

```bash
# Health Check
curl http://localhost:8002/purchase-orders/health

# Create PO
curl -X POST http://localhost:8002/purchase-orders/purchase-orders \
  -H "Content-Type: application/json" \
  -d '{
    "quotationId":"quotation-123",
    "buyerId":"buyer-123",
    "supplierId":"supplier-123",
    "items":[{"productId":"prod-1","quantity":1000,"unitPrice":95}]
  }'

# Generate PO PDF
curl -X POST http://localhost:8002/purchase-orders/purchase-orders/po-123/generate-pdf

# Send for Signature (DocuSign)
curl -X POST http://localhost:8002/purchase-orders/purchase-orders/po-123/send-for-signature \
  -H "Content-Type: application/json" \
  -d '{"signerEmail":"signer@example.com"}'

# Create PO Version
curl -X POST http://localhost:8002/purchase-orders/purchase-orders/po-123/version \
  -H "Content-Type: application/json" \
  -d '{"changes":"Updated quantity to 1200"}'

# Send to ERP
curl -X POST http://localhost:8002/purchase-orders/purchase-orders/po-123/send-to-erp \
  -H "Content-Type: application/json" \
  -d '{"erpSystem":"SAP","apiKey":"erp-key-123"}'

# Get PO
curl http://localhost:8002/purchase-orders/purchase-orders/po-123
```

### Invoice Service (`/invoices`)

```bash
# Health Check
curl http://localhost:8002/invoices/health

# Create Invoice
curl -X POST http://localhost:8002/invoices/invoices \
  -H "Content-Type: application/json" \
  -d '{
    "poId":"po-123",
    "supplierId":"supplier-123",
    "buyerId":"buyer-123",
    "items":[{"productId":"prod-1","quantity":1000,"unitPrice":95}],
    "taxRate":10
  }'

# Generate Invoice PDF
curl -X POST http://localhost:8002/invoices/invoices/invoice-123/generate-pdf

# Send Invoice
curl -X POST http://localhost:8002/invoices/invoices/invoice-123/send \
  -H "Content-Type: application/json" \
  -d '{"recipientEmail":"finance@buyer.com"}'

# Get Invoices
curl http://localhost:8002/invoices/invoices

# Get Invoice by ID
curl http://localhost:8002/invoices/invoices/invoice-123
```

### Corporate Payment Service (`/payments`)

```bash
# Health Check
curl http://localhost:8002/payments/health

# Process Payment
curl -X POST http://localhost:8002/payments/payments \
  -H "Content-Type: application/json" \
  -d '{
    "invoiceId":"invoice-123",
    "amount":104500,
    "currency":"USD",
    "paymentMethod":"stripe",
    "accountId":"account-123",
    "encryptedCardData":"encrypted-card-data",
    "mfaCode":"123456"
  }'

# Reconcile Payments
curl -X POST http://localhost:8002/payments/reconcile \
  -H "Content-Type: application/json" \
  -d '{"date":"2026-01-17"}'

# Payment Webhook
curl -X POST http://localhost:8002/payments/webhooks/stripe \
  -H "Content-Type: application/json" \
  -d '{"type":"payment.succeeded","data":{"id":"stripe-123"}}'

# Get Payment
curl http://localhost:8002/payments/payments/payment-123
```

### Inventory Service (`/inventory`)

```bash
# Health Check
curl http://localhost:8002/inventory/health

# Create Warehouse
curl -X POST http://localhost:8002/inventory/warehouses \
  -H "Content-Type: application/json" \
  -d '{
    "name":"Warehouse A",
    "location":"New York",
    "capacity":10000
  }'

# Add Inventory
curl -X POST http://localhost:8002/inventory/inventory \
  -H "Content-Type: application/json" \
  -d '{
    "warehouseId":"warehouse-123",
    "productId":"prod-1",
    "quantity":5000,
    "rfidTag":"RFID-12345"
  }'

# Allocate Inventory
curl -X POST http://localhost:8002/inventory/allocate \
  -H "Content-Type: application/json" \
  -d '{
    "orderId":"order-123",
    "items":[{"productId":"prod-1","quantity":1000}],
    "preferredWarehouses":["warehouse-123"]
  }'

# Get Warehouse Inventory
curl http://localhost:8002/inventory/warehouses/warehouse-123/inventory
```

### Scheduling Service (`/schedules`)

```bash
# Health Check
curl http://localhost:8002/schedules/health

# Create Schedule
curl -X POST http://localhost:8002/schedules/schedules \
  -H "Content-Type: application/json" \
  -d '{
    "orderId":"order-123",
    "scheduleType":"delivery",
    "scheduledDate":"2026-01-25T10:00:00Z",
    "recurring":false
  }'

# Get Schedules
curl "http://localhost:8002/schedules/schedules?orderId=order-123"

# Execute Schedule
curl -X POST http://localhost:8002/schedules/schedules/schedule-123/execute
```

### Analytics Service (`/analytics`)

```bash
# Health Check
curl http://localhost:8002/analytics/health

# Generate Report
curl -X POST http://localhost:8002/analytics/reports \
  -H "Content-Type: application/json" \
  -d '{
    "reportType":"sales",
    "dateRange":{"start":"2026-01-01","end":"2026-01-31"},
    "filters":{"accountId":"account-123"}
  }'

# ML Analytics
curl -X POST http://localhost:8002/analytics/analytics/ml \
  -H "Content-Type: application/json" \
  -d '{
    "type":"forecast",
    "productId":"prod-1",
    "timeframe":"monthly"
  }'

# Get Report
curl http://localhost:8002/analytics/reports/report-123

# Stream Analytics
curl http://localhost:8002/analytics/analytics/stream
```

### Negotiation Service (`/negotiations`)

```bash
# Health Check
curl http://localhost:8002/negotiations/health

# Create Negotiation
curl -X POST http://localhost:8002/negotiations/negotiations \
  -H "Content-Type: application/json" \
  -d '{
    "quotationId":"quotation-123",
    "initiatorId":"buyer-123",
    "type":"price"
  }'

# Send Message
curl -X POST http://localhost:8002/negotiations/negotiations/negotiation-123/messages \
  -H "Content-Type: application/json" \
  -d '{
    "senderId":"buyer-123",
    "message":"Can we reduce price by 5%?",
    "translated":false
  }'

# Update Presence
curl -X POST http://localhost:8002/negotiations/negotiations/negotiation-123/presence \
  -H "Content-Type: application/json" \
  -d '{"userId":"buyer-123","status":"online"}'

# Get Negotiation
curl http://localhost:8002/negotiations/negotiations/negotiation-123

# Search Negotiations
curl "http://localhost:8002/negotiations/negotiations/search?quotationId=quotation-123"
```

### Multi-Users Service (`/users`)

```bash
# Health Check
curl http://localhost:8002/users/health

# Create User
curl -X POST http://localhost:8002/users/users \
  -H "Content-Type: application/json" \
  -d '{
    "accountId":"account-123",
    "email":"user@acme.com",
    "password":"secure123",
    "role":"BUYER"
  }'

# Enable 2FA
curl -X POST http://localhost:8002/users/users/user-123/enable-2fa \
  -H "Content-Type: application/json" \
  -d '{"method":"TOTP"}'

# Check Permission
curl -X POST http://localhost:8002/users/users/user-123/check-permission \
  -H "Content-Type: application/json" \
  -d '{"permission":"approve_orders"}'

# Create Session
curl -X POST http://localhost:8002/users/sessions \
  -H "Content-Type: application/json" \
  -d '{"email":"user@acme.com","password":"secure123"}'

# Get Audit Logs
curl "http://localhost:8002/users/audit-logs?userId=user-123"

# Get User
curl http://localhost:8002/users/users/user-123
```

---

## C2C (Consumer to Consumer)
**Gateway Port:** 8003 | **Domain Path:** `/c2c` | **Domain Subdomain:** `c2c.kriss.karkark.com:24477`

### User Accounts Service (`/users`)

```bash
# Health Check
curl http://localhost:8003/users/health

# Register User
curl -X POST http://localhost:8003/users/register \
  -H "Content-Type: application/json" \
  -d '{"email":"buyer@example.com","password":"secure123","userType":"buyer"}'

# Login
curl -X POST http://localhost:8003/users/login \
  -H "Content-Type: application/json" \
  -d '{"email":"buyer@example.com","password":"secure123"}'

# Get User
curl http://localhost:8003/users/users/user-123

# Verify Token
curl -X POST http://localhost:8003/users/verify-token \
  -H "Content-Type: application/json" \
  -d '{"token":"jwt-token-123"}'
```

### Seller Verification Service (`/verification`)

```bash
# Health Check
curl http://localhost:8003/verification/health

# Submit Verification
curl -X POST http://localhost:8003/verification/verify \
  -H "Content-Type: application/json" \
  -d '{
    "sellerId":"seller-123",
    "documentType":"id_card",
    "documentUrl":"https://example.com/id.jpg"
  }'

# Approve Verification
curl -X POST http://localhost:8003/verification/verify/verification-123/approve \
  -H "Content-Type: application/json" \
  -d '{"approverId":"admin-123","approved":true}'

# Get Verification Status
curl http://localhost:8003/verification/verify/seller-123

# Get Pending Verifications
curl http://localhost:8003/verification/pending
```

### Seller Posting Service (`/posts`)

```bash
# Health Check
curl http://localhost:8003/posts/health

# Create Post
curl -X POST http://localhost:8003/posts/posts \
  -H "Content-Type: application/json" \
  -d '{
    "sellerId":"seller-123",
    "title":"Used iPhone 12",
    "images":[{"url":"https://example.com/img1.jpg","filename":"img1.jpg"}],
    "jwtToken":"jwt-token-123"
  }'

# Get Posts by Seller
curl http://localhost:8003/posts/posts/seller/seller-123

# Get All Posts
curl http://localhost:8003/posts/posts
```

### Product Description Service (`/products`)

```bash
# Health Check
curl http://localhost:8003/products/health

# Create Product
curl -X POST http://localhost:8003/products/products \
  -H "Content-Type: application/json" \
  -d '{
    "postId":"post-123",
    "description":"Great condition iPhone 12, 128GB, unlocked",
    "price":599.99,
    "category":"Electronics"
  }'

# Update Product
curl -X PATCH http://localhost:8003/products/products/product-123 \
  -H "Content-Type: application/json" \
  -d '{"price":549.99}'

# Get Product
curl http://localhost:8003/products/products/product-123
```

### Chat Service (`/chat`)

```bash
# Health Check
curl http://localhost:8003/chat/health

# Create Conversation
curl -X POST http://localhost:8003/chat/conversations \
  -H "Content-Type: application/json" \
  -d '{
    "buyerUserId":"buyer-123",
    "sellerUserId":"seller-123",
    "listingId":"listing-123",
    "firebaseAuthToken":"firebase-token-123"
  }'

# Send Message
curl -X POST http://localhost:8003/chat/messages \
  -H "Content-Type: application/json" \
  -d '{
    "conversationId":"conv-123",
    "senderId":"buyer-123",
    "message":"Is this still available?",
    "firebaseAuthToken":"firebase-token-123"
  }'

# Get Messages
curl http://localhost:8003/chat/conversations/conv-123/messages
```

### Escrow Service (`/escrow`)

```bash
# Health Check
curl http://localhost:8003/escrow/health

# Create Escrow
curl -X POST http://localhost:8003/escrow/escrows \
  -H "Content-Type: application/json" \
  -d '{
    "listingId":"listing-123",
    "buyerUserId":"buyer-123",
    "sellerUserId":"seller-123",
    "amount":599.99,
    "paymentMethod":"aba"
  }'

# Release Payment
curl -X POST http://localhost:8003/escrow/escrows/escrow-123/release \
  -H "Content-Type: application/json" \
  -d '{"releasedBy":"buyer-123"}'

# Refund Payment
curl -X POST http://localhost:8003/escrow/escrows/escrow-123/refund \
  -H "Content-Type: application/json" \
  -d '{"refundReason":"Item not received"}'

# Get Escrow
curl http://localhost:8003/escrow/escrows/escrow-123
```

### Wallet Service (`/wallets`)

```bash
# Health Check
curl http://localhost:8003/wallets/health

# Create Wallet
curl -X POST http://localhost:8003/wallets/wallets \
  -H "Content-Type: application/json" \
  -d '{"userId":"user-123"}'

# Create Transaction
curl -X POST http://localhost:8003/wallets/transactions \
  -H "Content-Type: application/json" \
  -d '{
    "fromUserId":"buyer-123",
    "toUserId":"platform",
    "amount":10,
    "type":"fee",
    "orderId":"order-123"
  }'

# Get Wallet
curl http://localhost:8003/wallets/wallets/user-123

# Get Transactions
curl http://localhost:8003/wallets/transactions/user-123
```

### Reviews & Rating Service (`/reviews`)

```bash
# Health Check
curl http://localhost:8003/reviews/health

# Create Review
curl -X POST http://localhost:8003/reviews/reviews \
  -H "Content-Type: application/json" \
  -d '{
    "listingId":"listing-123",
    "reviewerId":"buyer-123",
    "rating":5,
    "comment":"Great seller, fast delivery!"
  }'

# Moderate Review
curl -X POST http://localhost:8003/reviews/reviews/review-123/moderate \
  -H "Content-Type: application/json" \
  -d '{"action":"approve","moderatorId":"mod-123"}'

# Get Reviews by User
curl http://localhost:8003/reviews/reviews/user/user-123

# Get Pending Reviews
curl http://localhost:8003/reviews/reviews/pending
```

### Dispute Management Service (`/disputes`)

```bash
# Health Check
curl http://localhost:8003/disputes/health

# Create Dispute
curl -X POST http://localhost:8003/disputes/disputes \
  -H "Content-Type: application/json" \
  -d '{
    "listingId":"listing-123",
    "buyerId":"buyer-123",
    "sellerId":"seller-123",
    "reason":"Item not as described",
    "description":"The phone has scratches not mentioned"
  }'

# Review Dispute
curl -X POST http://localhost:8003/disputes/disputes/dispute-123/review \
  -H "Content-Type: application/json" \
  -d '{"reviewerId":"admin-123","decision":"pending"}'

# Resolve Dispute
curl -X POST http://localhost:8003/disputes/disputes/dispute-123/resolve \
  -H "Content-Type: application/json" \
  -d '{
    "resolverId":"admin-123",
    "resolution":"refund_buyer",
    "refundAmount":599.99
  }'

# Get Disputes
curl "http://localhost:8003/disputes/disputes?status=pending"

# Get Dispute
curl http://localhost:8003/disputes/disputes/dispute-123
```

### Delivery Options Service (`/deliveries`)

```bash
# Health Check
curl http://localhost:8003/deliveries/health

# Create Delivery
curl -X POST http://localhost:8003/deliveries/deliveries \
  -H "Content-Type: application/json" \
  -d '{
    "orderId":"order-123",
    "shippingMethod":"standard",
    "address":{"street":"123 Main St","city":"New York","zip":"10001"}
  }'

# Confirm Delivery
curl -X POST http://localhost:8003/deliveries/deliveries/delivery-123/confirm \
  -H "Content-Type: application/json" \
  -d '{"confirmedBy":"buyer-123"}'

# Update Tracking
curl -X PATCH http://localhost:8003/deliveries/deliveries/delivery-123/tracking \
  -H "Content-Type: application/json" \
  -d '{"trackingNumber":"TRACK123","status":"in_transit"}'

# Get Delivery
curl http://localhost:8003/deliveries/deliveries/delivery-123
```

### Listing Service (`/listings`)

```bash
# Get Listings
curl http://localhost:8003/listings/listings
```

---

## C2B (Consumer to Business)
**Gateway Port:** 8004 | **Domain Path:** `/c2b` | **Domain Subdomain:** `c2b.kriss.karkark.com:24477`

### Creator Profile Service (`/profiles`)

```bash
# Health Check
curl http://localhost:8004/profiles/health

# Create Profile
curl -X POST http://localhost:8004/profiles/profiles \
  -H "Content-Type: application/json" \
  -d '{
    "userId":"creator-123",
    "skills":["design","development"],
    "portfolio":["https://example.com/work1.jpg"],
    "hourlyRate":50,
    "oauthToken":"oauth-token-123"
  }'

# Get Profile
curl http://localhost:8004/profiles/profiles/creator-123
```

### Project Posting Service (`/projects`)

```bash
# Health Check
curl http://localhost:8004/projects/health

# Create Project
curl -X POST http://localhost:8004/projects/projects \
  -H "Content-Type: application/json" \
  -d '{
    "businessId":"business-123",
    "title":"Website Redesign",
    "description":"Need a new website design",
    "budget":5000,
    "deadline":"2026-02-15",
    "tags":["design","web","ui/ux"]
  }'

# Get Projects
curl http://localhost:8004/projects/projects

# Get Project
curl http://localhost:8004/projects/projects/project-123
```

### Bidding Service (`/bids`)

```bash
# Health Check
curl http://localhost:8004/bids/health

# Get Bids for Project
curl http://localhost:8004/bids/projects/project-123/bids

# Get Bids for Demand
curl http://localhost:8004/bids/demands/demand-123/bids

# Submit Bid
curl -X POST http://localhost:8004/bids/projects/project-123/bids \
  -H "Content-Type: application/json" \
  -d '{
    "creatorId":"creator-123",
    "proposal":"I can complete this in 2 weeks",
    "bidAmount":4500,
    "timeline":"14 days"
  }'

# Submit Bid for Demand
curl -X POST http://localhost:8004/bids/demands/demand-123/bids \
  -H "Content-Type: application/json" \
  -d '{
    "creatorId":"creator-123",
    "proposal":"Expert in this field",
    "bidAmount":3000
  }'
```

### Contract Service (`/contracts`)

```bash
# Health Check
curl http://localhost:8004/contracts/health

# Create Contract
curl -X POST http://localhost:8004/contracts/contracts \
  -H "Content-Type: application/json" \
  -d '{
    "projectId":"project-123",
    "bidId":"bid-123",
    "businessId":"business-123",
    "creatorId":"creator-123",
    "terms":"Work will be completed within 14 days",
    "milestones":[
      {"name":"Design Mockups","amount":2000,"dueDate":"2026-02-01"},
      {"name":"Final Delivery","amount":2500,"dueDate":"2026-02-15"}
    ]
  }'

# Generate Contract PDF
curl -X POST http://localhost:8004/contracts/contracts/contract-123/generate-pdf

# Send for Signature
curl -X POST http://localhost:8004/contracts/contracts/contract-123/send-for-signature \
  -H "Content-Type: application/json" \
  -d '{"signerEmail":"creator@example.com"}'

# Get Contract
curl http://localhost:8004/contracts/contracts/contract-123
```

### Delivery Service (`/deliveries`)

```bash
# Health Check
curl http://localhost:8004/deliveries/health

# Submit Delivery
curl -X POST http://localhost:8004/deliveries/deliveries \
  -H "Content-Type: application/json" \
  -d '{
    "projectId":"project-123",
    "contractId":"contract-123",
    "files":[{"name":"design.pdf","url":"https://s3.example.com/design.pdf"}],
    "version":1,
    "description":"First draft of design mockups"
  }'

# Initiate Chunk Upload (Tus.io)
curl -X POST http://localhost:8004/deliveries/upload/initiate \
  -H "Content-Type: application/json" \
  -d '{
    "projectId":"project-123",
    "filename":"large-file.zip",
    "fileSize":10485760
  }'
```

### Payment Service (`/payments`)

```bash
# Health Check
curl http://localhost:8004/payments/health

# Create Milestone Payment
curl -X POST http://localhost:8004/payments/payments \
  -H "Content-Type: application/json" \
  -d '{
    "contractId":"contract-123",
    "milestoneId":"milestone-1",
    "amount":2000,
    "paymentMethod":"paypal",
    "walletId":"wallet-123"
  }'

# Release Milestone Payment
curl -X POST http://localhost:8004/payments/payments/payment-123/release \
  -H "Content-Type: application/json" \
  -d '{"releasedBy":"business-123"}'
```

### Commission Service (`/commission`)

```bash
# Health Check
curl http://localhost:8004/commission/health

# Calculate Commission
curl -X POST http://localhost:8004/commission/calculate \
  -H "Content-Type: application/json" \
  -d '{
    "amount":5000,
    "ratePct":10,
    "paymentPlatform":"stripe"
  }'

# Get Analytics
curl "http://localhost:8004/commission/analytics?startDate=2026-01-01&endDate=2026-01-31"
```

### Payout Service (`/payouts`)

```bash
# Health Check
curl http://localhost:8004/payouts/health

# Verify Identity
curl -X POST http://localhost:8004/payouts/verify-identity \
  -H "Content-Type: application/json" \
  -d '{
    "userId":"creator-123",
    "documentType":"id_card",
    "documentUrl":"https://example.com/id.jpg"
  }'

# Request Payout
curl -X POST http://localhost:8004/payouts/payouts \
  -H "Content-Type: application/json" \
  -d '{
    "userId":"creator-123",
    "amount":3000,
    "currency":"USD",
    "payoutMethod":"aba",
    "accountDetails":{"accountNumber":"123456","bankName":"ABA"}
  }'

# Get Payout
curl http://localhost:8004/payouts/payouts/payout-123
```

### Rating Service (`/ratings`)

```bash
# Health Check
curl http://localhost:8004/ratings/health

# Create Rating
curl -X POST http://localhost:8004/ratings/ratings \
  -H "Content-Type: application/json" \
  -d '{
    "projectId":"project-123",
    "raterId":"business-123",
    "ratedId":"creator-123",
    "rating":5,
    "comment":"Excellent work, delivered on time!"
  }'

# Get User Ratings
curl http://localhost:8004/ratings/users/creator-123/ratings
```

### Messaging Service (`/messages`)

```bash
# Health Check
curl http://localhost:8004/messages/health

# Send Message
curl -X POST http://localhost:8004/messages/conversations \
  -H "Content-Type: application/json" \
  -d '{
    "projectId":"project-123",
    "senderId":"creator-123",
    "receiverId":"business-123",
    "message":"I have a question about the requirements"
  }'

# Get Messages
curl "http://localhost:8004/messages/conversations?projectId=project-123"
```

### Project Management Service (`/tasks`)

```bash
# Health Check
curl http://localhost:8004/tasks/health

# Create Task
curl -X POST http://localhost:8004/tasks/projects/project-123/tasks \
  -H "Content-Type: application/json" \
  -d '{
    "title":"Design Mockups",
    "description":"Create initial design mockups",
    "assigneeId":"creator-123",
    "dueDate":"2026-02-01"
  }'

# Update Task Position
curl -X PATCH http://localhost:8004/tasks/tasks/task-123/position \
  -H "Content-Type: application/json" \
  -d '{"newPosition":2}'

# Create Timeline
curl -X POST http://localhost:8004/tasks/projects/project-123/timeline \
  -H "Content-Type: application/json" \
  -d '{
    "tasks":["task-1","task-2"],
    "startDate":"2026-01-20",
    "endDate":"2026-02-15"
  }'

# Get Tasks
curl http://localhost:8004/tasks/projects/project-123/tasks

# Get Timeline
curl http://localhost:8004/tasks/projects/project-123/timeline
```

### Proposal Service (`/proposals`)

```bash
# Health Check
curl http://localhost:8004/proposals/health

# Create Proposal
curl -X POST http://localhost:8004/proposals/proposals \
  -H "Content-Type: application/json" \
  -d '{
    "projectId":"project-123",
    "creatorId":"creator-123",
    "content":"<h1>My Proposal</h1><p>I will deliver...</p>",
    "editor":"quill"
  }'

# Generate Proposal PDF
curl -X POST http://localhost:8004/proposals/proposals/proposal-123/generate-pdf

# Get Proposal
curl http://localhost:8004/proposals/proposals/proposal-123
```

### Demand Service (`/demand`)

```bash
# Health Check
curl http://localhost:8004/demand/health

# Create Demand
curl -X POST http://localhost:8004/demand/demands \
  -H "Content-Type: application/json" \
  -d '{
    "businessId":"business-123",
    "title":"Need Graphic Designer",
    "description":"Looking for a graphic designer for logo design",
    "budget":1000
  }'

# Get Demands
curl http://localhost:8004/demand/demands
```

---

## Quick Reference: Gateway Health Checks

```bash
# B2C Gateway
curl http://localhost:8001/health
curl http://kriss.karkark.com:24477/b2c/health

# B2B Gateway
curl http://localhost:8002/health
curl http://kriss.karkark.com:24477/b2b/health

# C2C Gateway
curl http://localhost:8003/health
curl http://kriss.karkark.com:24477/c2c/health

# C2B Gateway
curl http://localhost:8004/health
curl http://kriss.karkark.com:24477/c2b/health

# Nginx Root
curl http://localhost:24477/health
curl http://kriss.karkark.com:24477/health
```

---

## Notes

- Replace placeholder IDs (e.g., `user-123`, `project-123`) with actual IDs from your system
- All POST/PUT/PATCH requests require `Content-Type: application/json` header
- JWT tokens and authentication tokens should be obtained from login endpoints
- Some endpoints may require specific roles or permissions
- Domain URLs require DNS setup for subdomain routing or use path-based routing
