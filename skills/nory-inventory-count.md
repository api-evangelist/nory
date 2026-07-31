---
name: Ingest items and record an inventory count
description: Authenticate to Nory Middleware, ingest brand items/menu items, then create and read inventory counts for a branch.
api: openapi/nory-middleware-openapi.json
operations:
  - "POST /auth/token"
  - "POST /inventory/ingest"
  - "GET /inventory/branches/{branchId}/counts"
  - "POST /inventory/branches/{branchId}/counts"
  - "GET /inventory/branches/{branchId}/deliveries"
---

# Ingest items and record an inventory count

Feed item/menu data into Nory and capture stock counts for a branch.

## Steps

1. **Authenticate.** `POST /auth/token` (email + password) → access token; send
   as `Authorization: Bearer <token>`.
2. **Ingest.** `POST /inventory/ingest` with an `Ingest` body
   (`brand_id`, `ingest_type`, `content`) to load items, batch items or menu items.
3. **Record a count.** `POST /inventory/branches/{branchId}/counts` with a `Count`
   payload (`brand_item_counts[]`, each carrying `unit_counts[]` of `unit` + `quantity`).
4. **Read back.** `GET /inventory/branches/{branchId}/counts` to list counts and
   `GET /inventory/branches/{branchId}/deliveries` to reconcile against deliveries.

## Rules

- Units follow the `Unit` / `BaseUnit` model (`unit_of_measure`, `unit_type`) —
  see `data-model/nory-data-model.yml`.
- Validation failures return `400` (`Incorrect data supplied` / `Validation error`).
- No idempotency key — de-duplicate counts on the client side.
