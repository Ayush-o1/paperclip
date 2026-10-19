
# CommerceCore — Database Design

## 1. Core Entities

### User
Stores customer and administrator accounts.

Fields:
- id
- name
- email
- passwordHash
- role
- createdAt

### Category
Groups related products.

Fields:
- id
- name
- slug
- createdAt

### Product
Stores catalogue information and inventory.

Fields:
- id
- categoryId
- name
- sku
- description
- pricePaise
- stock
- isActive
- createdAt
- updatedAt

### CartItem
Connects a user to a product in their cart.

Fields:
- id
- userId
- productId
- quantity
- createdAt
- updatedAt

### Order
Represents a purchase and its state.

Fields:
- id
- userId
- status
- totalPaise
- idempotencyKey
- requestFingerprint
- createdAt
- updatedAt

### OrderItem
Preserves purchase-time product information.

Fields:
- id
- orderId
- productId
- productName
- sku
- unitPricePaise
- quantity
- lineTotalPaise

### PaymentAttempt
Records simulated payment attempts.

Fields:
- id
- orderId
- status
- idempotencyKey
- createdAt
- updatedAt

## 2. Relationships

- User has many CartItems.
- User has many Orders.
- Category has many Products.
- Product has many CartItems.
- Order has many OrderItems.
- Product can be referenced by many OrderItems.
- Order has many PaymentAttempts.

CartItem connects a user and a product.
OrderItem connects an order and a product while preserving
the details of the purchased item.

## 3. Integrity Constraints

- Normalized user email must be unique.
- Product SKU must be unique.
- (userId, productId) must be unique for CartItem.
- Product stock must never be negative.
- CartItem quantity must be between 1 and 99.
- Monetary amounts are stored as integer paise.
- (userId, idempotencyKey) must be unique for Order.
- Foreign keys enforce valid relationships.
- Historical order items preserve product and price snapshots.

## 4. Design Principles

- PostgreSQL is the source of truth for persistent application data.
- Database constraints protect critical invariants.
- Checkout must be atomic.
- Order history must remain accurate when catalogue data changes.
- Payment attempts must be recorded and processed idempotently.
