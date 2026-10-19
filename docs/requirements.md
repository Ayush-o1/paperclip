
# CommerceCore — Requirements

## 1. Project Overview

CommerceCore is a transactional e-commerce platform built with Node.js, Express, JavaScript ES modules, PostgreSQL, and Prisma. It supports customer authentication, product catalogue management, persistent shopping carts, transactional checkout, order management, and simulated payments.

The frontend will use HTML, CSS, and vanilla JavaScript.

## 2. Technology Stack

- Runtime: Node.js
- Backend: Express.js
- Language: JavaScript ES modules
- Database: PostgreSQL
- ORM: Prisma
- Authentication: JWT and bcrypt
- Validation: Zod
- Testing: Vitest and Supertest
- Logging: Pino
- API Documentation: OpenAPI / Swagger
- Frontend: HTML, CSS, vanilla JavaScript
- CI: GitHub Actions

## 3. Functional Requirements

### FR-01: Authentication and Authorization

- Customers can register and log in.
- Passwords must be hashed before storage.
- Public registration must always assign the CUSTOMER role.
- Administrator access must be restricted to explicitly provisioned administrator accounts.
- Protected endpoints must validate authentication tokens.
- Customers must not access another customer's private resources.
- Administrator-only operations must enforce role-based authorization.

### FR-02: Product Catalogue

- Customers can browse active products.
- Customers can view individual product details.
- Product listings support pagination, category filtering, price filtering, and allowlisted sorting.
- Inactive products must not appear in the public catalogue.
- Each product has a unique SKU.
- Prices are stored as integer paise.
- Stock quantities must never become negative.

### FR-03: Catalogue Administration

- Administrators can create and manage categories.
- Administrators can create, update, and deactivate products.
- Administrators can manage product prices and inventory.
- Invalid prices, stock quantities, and category references must be rejected.

### FR-04: Shopping Cart

- Authenticated customers can add products to their cart.
- Customers can update item quantities and remove items.
- Cart data must persist in PostgreSQL.
- Each cart item belongs to one customer and one product.
- A customer must not modify another customer's cart.
- Quantities must be between 1 and 99.

### FR-05: Checkout and Inventory

- Checkout uses the authenticated customer's server-side cart.
- The client must not determine the authoritative price, total, stock, or payment outcome.
- The server calculates order totals using database prices.
- Orders preserve purchase-time product names, SKUs, prices, quantities, and line totals.
- Stock reservation and order creation must happen in one database transaction.
- A failed transaction must not leave partial orders, order items, stock changes, or cart changes.
- Concurrent checkouts must not oversell inventory.
- Checkout requires an Idempotency-Key.
- Repeating the same request with the same key must return the existing result.
- Reusing a key for a different request must return HTTP 409 Conflict.

### FR-06: Order Management

- Customers can view their own order history and order details.
- Customers must not access another customer's orders.
- Customers can cancel only eligible unpaid orders.
- Order state transitions must follow the defined business rules.
- Inventory must be restored correctly when cancellation requires it.

### FR-07: Simulated Payments

- The application must simulate payment success, failure, and timeout outcomes.
- No real payment processing or card data collection is required.
- Payment attempts must support idempotent handling.
- Payment outcomes must be coordinated safely with order state changes.
- Failed or timed-out payments must cancel eligible orders and restore reserved inventory exactly once.
- Repeated requests must not create duplicate payment effects.

### FR-08: Reliability and Security

- Request input must be validated.
- Errors must follow a consistent response format.
- Authentication endpoints must have appropriate rate limiting.
- Logs must include request identifiers and redact sensitive information.
- The application must provide liveness and readiness endpoints.
- API endpoints must be documented using OpenAPI.
- Automated tests must cover critical business and security rules.
- GitHub Actions must run the automated test suite.

## 4. Acceptance Criteria

### AC-01: Authentication

- Registration and login work with valid input.
- Duplicate email registration is rejected.
- Passwords are not stored in plaintext.
- Invalid credentials and invalid tokens are rejected.
- Public registration cannot grant administrator privileges.
- Ownership and role restrictions are tested.

### AC-02: Catalogue and Administration

- Active products can be listed and retrieved.
- Pagination and supported filters behave correctly.
- Invalid sorting parameters are rejected.
- Unauthorized users cannot perform administrative operations.
- Product SKU uniqueness and stock constraints are enforced.

### AC-03: Shopping Cart

- Customers can add, update, and remove cart items.
- Cart contents persist across requests.
- Invalid quantities and unauthorized access are rejected.
- A product cannot appear more than once per customer cart.

### AC-04: Checkout

- Order totals are calculated from database prices.
- Order items retain purchase-time snapshots.
- Stock is reserved atomically with order creation.
- Insufficient stock causes checkout to fail without partial changes.
- Concurrent checkout tests demonstrate that stock cannot become negative.
- Identical idempotent requests do not create duplicate orders.
- Reusing an idempotency key for a different request returns HTTP 409.

### AC-05: Orders and Payments

- Customers can access only their own orders.
- Cancellation follows permitted order-state transitions.
- Successful payment marks the order as paid.
- Failed and timed-out payments follow the defined cancellation and inventory-restoration rules.
- Duplicate payment requests do not apply the same effect twice.

### AC-06: Reliability and Delivery

- Input validation and consistent error handling are covered by tests.
- Logging does not expose secrets or credentials.
- Liveness and readiness endpoints behave correctly.
- OpenAPI documentation describes the implemented API.
- Automated tests run in GitHub Actions.
- Setup instructions support a fresh local installation.

## 5. Core Business Invariants

The following rules must always hold:

1. Email addresses are unique after normalization.
2. Public registration always creates a CUSTOMER account.
3. Product SKUs are unique.
4. Prices are represented as integer paise.
5. Stock quantities cannot be negative.
6. Cart quantities are between 1 and 99.
7. Customers can access only their own carts and orders.
8. Checkout totals are calculated on the server.
9. Checkout changes must be atomic.
10. Idempotency prevents duplicate order and payment effects.
11. Paid orders cannot be cancelled through the unpaid-order cancellation flow.
12. Inventory must be restored exactly once when a qualifying unpaid order is cancelled.
13. Order and payment state transitions must be validated.

## 6. Out of Scope

The initial release will not include:

- Real payment gateway integration
- Collection or storage of card details
- Shipping-provider integration
- Tax calculation
- Coupons and promotional codes
- Product variants
- Microservices
- Kubernetes
- Redis as a required infrastructure dependency

## 7. Completion Criteria

The initial release is considered complete when:

- All required customer and administrator workflows are implemented.
- Backend APIs and frontend pages work together.
- Database migrations and setup instructions are documented.
- Automated tests cover critical business logic, security, rollback, idempotency, and concurrency.
- OpenAPI documentation is available.
- GitHub Actions runs the test suite successfully.
- A fresh setup can follow the documented instructions.
- The application passes end-to-end acceptance testing.
