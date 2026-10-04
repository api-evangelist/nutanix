---
name: nutanix-projects-fetch-details
description: Fetch a list of projects and retrieve details for a specific project.
api: openapi/nutanix-projects-api-openapi.yml
operations:
- listProjects
- getProject
generated: '2026-10-04'
method: generated
generator: extract-docs-artifacts.py skills (local)
source: openapi/nutanix-projects-api-openapi.yml ; every operationId checked against the contract
---

# nutanix-projects-fetch-details

Fetch a list of projects and retrieve details for a specific project.

## Steps

1. 1. Call `listProjects` – no request body or headers are documented.
2. 2. Call `getProject` – provide the `uuid` path parameter of the desired project; no additional headers are documented.

## Rules

- Auth: Use the `basicAuth` scheme (HTTP Basic Authentication) in the Authorization header.
- Rate limit: X‑Small tier – 18 requests per second, burst up to 4, max 100 requests per minute, reset after 30 seconds; on exhaustion returns HTTP status with no body.
