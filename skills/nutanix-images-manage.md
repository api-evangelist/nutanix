---
name: nutanix-images-manage
description: Manage Nutanix images by listing, retrieving, creating, and deleting them.
api: openapi/nutanix-images-api-openapi.yml
operations:
- listImages
- createImage
- getImage
- deleteImage
generated: '2026-10-04'
method: generated
generator: extract-docs-artifacts.py skills (local)
source: openapi/nutanix-images-api-openapi.yml ; every operationId checked against the contract
---

# nutanix-images-manage

Manage Nutanix images by listing, retrieving, creating, and deleting them.

## Steps

1. 1. Call `listImages` – no request body fields are required; include the Basic Auth header.
2. 2. Call `createImage` – provide the image definition in the request body as specified by the API contract; include the Basic Auth header.
3. 3. Call `getImage` – supply the `uuid` path parameter of the created image; include the Basic Auth header.
4. 4. Call `deleteImage` – supply the same `uuid` path parameter; include the Basic Auth header.

## Rules

- Auth: Use the `Authorization` header with Basic authentication (basicAuth).
- Rate limiting: The service enforces a rate limit of 18 requests per 30 seconds (X‑Small tier). On exhaustion it returns an HTTP response with no special error body.
- Idempotency: `deleteImage` is idempotent – repeated calls with the same `uuid` will succeed without side effects.
