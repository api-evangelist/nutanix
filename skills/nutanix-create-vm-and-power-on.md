---
name: nutanix-create-vm-and-power-on
description: Create a new virtual machine and power it on.
api: openapi/nutanix-vms-api-openapi.yml
operations:
- createVm
- setVmPowerState
generated: '2026-10-04'
method: generated
generator: extract-docs-artifacts.py skills (local)
source: openapi/nutanix-vms-api-openapi.yml ; every operationId checked against the contract
---

# nutanix-create-vm-and-power-on

Create a new virtual machine and power it on.

## Steps

1. 1. Call `createVm` with the request body fields required to define the VM (e.g., name, cpu, memory, disk).
2. 2. Call `setVmPowerState` with the path parameter `id` set to the UUID returned from `createVm` and include the `state` field in the request body to power on the VM.

## Rules

- Authentication: use the `basicAuth` HTTP scheme and include the `Authorization` header.
- Rate limiting: X‑Small tier allows 18 requests per 30 seconds; on exhaustion the API returns no specific HTTP status code.
- Idempotency: `createVm` is not idempotent; ensure the request body is unique or handle duplicate creation errors.
