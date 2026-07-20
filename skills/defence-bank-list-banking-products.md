---
name: List and inspect Defence Bank banking products
description: Query Defence Bank's public Consumer Data Right Product Reference Data feed to list banking products and fetch full product detail, with no authentication.
api: openapi/defence-bank-cds-banking-products-openapi.yml
operations: [listBankingProducts, getBankingProductDetail]
generated: '2026-07-20'
method: generated
---

# List and inspect Defence Bank banking products

Defence Bank is an Australian CDR data holder. Its Product Reference Data (PRD)
endpoints are **public and unauthenticated** — no API key, token, or consent is
required. Base URL: `https://product.defencebank.com.au/cds-au/v1`.

## Rules

- Send the version header `x-v: 4` on `listBankingProducts` and `x-v: 7` on
  `getBankingProductDetail` (the highest versions this data holder serves). If a
  version is unsupported you get `406` (`urn:au-cds:error:cds:header:unsupported-version`).
- Optionally send `x-fapi-interaction-id` (a UUID); it is echoed back on the response
  for tracing.
- Responses are `application/json`. Errors use the CDS `ResponseErrorListV2`
  envelope: `{ "errors": [ { "code": "urn:au-cds:error:cds:*", "title", "detail" } ] }`.
- Do NOT attempt the account/transaction/payee endpoints here — they require CDR
  FAPI OAuth2 + MTLS and accreditation.

## Steps

1. **List products** — call `listBankingProducts`:
   `GET /banking/products?page-size=100` with `x-v: 4`.
   Filter with `product-category` (e.g. `TRANS_AND_SAVINGS_ACCOUNTS`, `TERM_DEPOSITS`,
   `CRED_AND_CHRG_CARDS`, `RESIDENTIAL_MORTGAGES`, `PERS_LOANS`), `effective`, or
   `updated-since`. Paginate via `page` / `page-size`; read `meta.totalRecords` and
   `meta.totalPages`, follow `links.next` until exhausted.
2. **Pick a product** — take a `productId` from `data.products[]`.
3. **Fetch detail** — call `getBankingProductDetail`:
   `GET /banking/products/{productId}` with `x-v: 7`. The `data` object adds
   `bundles`, `features`, `constraints`, `eligibility`, `fees`, `depositRates`, and
   `lendingRates`. A `404` (`urn:au-cds:error:cds:resource:not-found`) means the
   productId is unknown or no longer effective.
4. **Compare** — use `name`, `productCategory`, `depositRates`/`lendingRates`, `fees`,
   and `eligibility` across products to answer rate/fee/eligibility questions.
