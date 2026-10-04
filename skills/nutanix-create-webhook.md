---
name: nutanix-create-webhook
description: Create a new webhook and retrieve its details.
api: openapi/nutanix-webhooks-api-openapi.yml
operations:
- createWebhook
- getWebhook
generated: '2026-10-04'
method: generated
generator: extract-docs-artifacts.py skills (local)
source: openapi/nutanix-webhooks-api-openapi.yml ; every operationId checked against the contract
---

# nutanix-create-webhook

Create a new webhook and retrieve its details.

## Steps

1. 1. `createWebhook` – required request body fields as defined in the contract.
2. 2. `getWebhook` – provide the `uuid` path parameter returned from the create step.

## Rules

- Authentication: use the `basicAuth` HTTP scheme (Authorization header).
- Rate limiting: X‑Small tier – 18 requests per 4 seconds, burst up to 100, reset after 30 seconds; on exhaustion returns HTTP status with no body.
