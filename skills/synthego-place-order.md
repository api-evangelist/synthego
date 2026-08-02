---
name: Place a Synthego guide-RNA order
description: Price synthetic guide-RNA products, generate a priced order preview, and track the order to checkout using the Synthego Order API.
api: openapi/synthego-order-openapi.json
operations:
  - "GET /pricing"
  - "POST /order/"
  - "GET /order/{order_request_id}"
auth:
  type: apiKey
  header: SYNTHEGOAPIKEY
---

# Place a Synthego guide-RNA order

Operating instructions for an agent using the **Synthego Order API** (`https://api.synthego.com`,
Swagger 2.0). This API lets a partner/integrator price Synthego synthetic guide-RNA products and
start an order that the customer completes on Synthego's eCommerce checkout.

## Authentication

Every request must carry the `SYNTHEGOAPIKEY` header with a valid API key issued by Synthego.
There is no OAuth flow. Requests without a valid key return HTTP 400.

## Steps

1. **List current pricing — `GET /pricing`.**
   Retrieve `ProductPricingInformation.available_products[]`. Each `ProductDescription` gives a
   `product_type` (the identifier you must use when building an order), a `product_name`, and a
   `product_item_price` (USD list price). Use the returned `product_type` values verbatim.

2. **Generate a priced preview — `POST /order/`.**
   Send an `OrderPreviewRequest` with an optional `integrator_id` and an `items[]` list of
   `OrderItemRequest` objects, each with a `label`, an IUPAC `sequence`, and a `product_type` from
   the pricing catalog (e.g. `ez_sgRNA_oligonucleotide`, `custom_RNA_100`). The response
   (`OrderPreviewResponse`) returns per-item estimates, a `subtotal`, a unique **`order_request_id`**,
   and an **`order_request_url`** — redirect the customer to that URL to complete the purchase.
   This preview call does not itself charge; it only prices and reserves the request.

3. **Track the order — `GET /order/{order_request_id}`.**
   Poll with the `order_request_id` from step 2 to get an `OrderStatusResponse` (`status`,
   `current_order_contents[]`, `subtotal`, and the cached `initial_order`). The status page URL
   points at Synthego's eCommerce site.

## Conventions and error handling

- **No idempotency key** is documented; the preview step is safe to retry because it does not place
  a charge (see `conventions/synthego-conventions.yml`).
- **No pagination** — arrays are returned in full.
- **Errors** use a generic envelope `{ "message": string }`; a `500` is the only declared failure
  response and a missing/invalid key returns `400` (see `errors/synthego-problem-types.yml`).
