
# CommerceCore — Business Rules and API Conventions

## 1. Order State Machine

Order statuses:

- PENDING_PAYMENT
- PAID
- CANCELLED

Allowed transitions:

- PENDING_PAYMENT -> PAID when payment succeeds.
- PENDING_PAYMENT -> CANCELLED when payment fails, times out, or an eligible customer cancellation occurs.
- PAID and CANCELLED are terminal states in the initial release.

Inventory is reserved during checkout. When an eligible unpaid order is cancelled, reserved inventory must be restored exactly once.

Paid orders cannot use the unpaid-order cancellation flow. Cancelled orders cannot subsequently become paid.

## 2. Payment Attempt State Machine

Payment attempt statuses:

- PENDING
- SUCCEEDED
- FAILED
- TIMED_OUT

Each attempt records its own outcome.

A successful attempt can mark an eligible order as PAID. A failed or timed-out attempt cancels an eligible pending order and restores its reserved inventory exactly once.

Payment attempts must be idempotent. Repeated requests must not apply the same payment effect twice.

## 3. Business Invariants

1. Stock must never become negative.
2. Checkout must be atomic.
3. Order items preserve purchase-time product names, SKUs, prices, quantities, and line totals.
4. Customers can access only their own carts and orders.
5. Checkout totals are calculated on the server.
6. Checkout requires an idempotency key scoped to the authenticated customer.
7. Reusing a checkout key with a different request fingerprint returns HTTP 409.
8. Payment retries must not duplicate payment effects.
9. Inventory restoration must occur at most once per cancelled order.
10. Paid orders cannot enter the unpaid cancellation flow.
11. Database constraints and transactions protect critical invariants.

## 4. API Conventions

- API prefix: /api
- GET retrieves resources.
- POST creates resources or initiates operations.
- PATCH partially updates resources.
- DELETE removes eligible resources.
- Zod validates request input.
- Authentication derives identity and role from a verified token.
- Resource ownership is checked on protected operations.
- JSON error responses use a consistent format.
- 401 indicates unauthenticated access.
- 403 indicates forbidden access.
- 404 indicates a missing resource.
- 409 indicates a state or idempotency conflict.
- 422 indicates business validation failure.
- Checkout and payment operations use idempotency.
- Checkout inventory reservation, order creation, order-item creation, and cart clearing occur in one database transaction.

## 5. Initial API Surface

- POST /api/auth/register
- POST /api/auth/login
- GET /api/products
- GET /api/products/:id
- GET /api/cart
- POST /api/orders
- GET /api/orders
- GET /api/orders/:id
- POST /api/orders/:id/cancel
- POST /api/orders/:id/payments

Administrator endpoints will be documented as their modules are designed and implemented.
