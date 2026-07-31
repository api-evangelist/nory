---
name: Generate and accept a recommended purchase order
description: Use Nory's AI demand-based ordering to generate a recommended purchase order for a branch and accept (or amend) it into a real order.
api: openapi/nory-middleware-openapi.json
operations:
  - "POST /auth/token"
  - "GET /brands/brand"
  - "GET /branches/{id}"
  - "GET /inventory/suppliers"
  - "POST /inventory/brands/{brandId}/branches/{branchId}/recommended-order"
  - "GET /inventory/brands/{brandId}/branches/{branchId}/recommended-order"
  - "POST /inventory/brands/{brandId}/branches/{branchId}/recommended-order/accept"
---

# Generate and accept a recommended purchase order

Nory Middleware exposes AI demand-based ordering (DBO). Use it to produce a
recommended order for a branch, review it, then accept or amend it.

## Steps

1. **Authenticate.** `POST /auth/token` with `{ "email", "password" }`. Store the
   returned access token; send it as `Authorization: Bearer <token>` on every
   call. Refresh with `POST /auth/refreshToken` when it expires.
2. **Resolve context.** `GET /brands/brand` for the brand, then `GET /branches/{id}`
   for the target branch. Optionally `GET /inventory/suppliers` to see available
   suppliers.
3. **Generate.** `POST /inventory/brands/{brandId}/branches/{branchId}/recommended-order`
   to run the recommendation. Handle `409` — a recommendation may already be
   mid-creation or submitted.
4. **Review.** `GET /inventory/brands/{brandId}/branches/{branchId}/recommended-order`
   to read the recommended items (order_quantity, urgency, delivery_date,
   run_out_date). A `404` means the order is missing or stale — regenerate.
5. **Accept or amend.** `POST /inventory/brands/{brandId}/branches/{branchId}/recommended-order/accept`
   with any amendments to create the purchase order.

## Rules

- This is a **write / financial** flow — never accept an order without an
  explicit human confirmation of quantities and supplier.
- No idempotency key is supported; do not blindly retry `accept` on a timeout —
  re-read the order first (it may be `409` already submitted).
- Errors are plain JSON with conventional status codes — see
  `errors/nory-problem-types.yml`.
