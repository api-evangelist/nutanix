# Nutanix (nutanix)

Nutanix is a hyper-converged infrastructure solution that integrates compute, virtualization, storage, networking, and security to power enterprise applications. Nutanix provides public APIs for managing and automating infrastructure including Prism Central (v3 and GA v4), Prism Element, Karbon Kubernetes, Nutanix Database Service (NDB), Cloud Clusters (NC2), NCM Self-Service, and Foundation.

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/nutanix/refs/heads/main/apis.yml)

## Scope

- **Type:** Index
- **Position:** Producer
- **Access:** 1st-Party

## Tags:

 - Cloud Management, Hyperconverged, Infrastructure, Virtualization, Kubernetes, Database

## Timestamps

- **Created:** 2025-03-14
- **Modified:** 2026-04-28

## APIs

### Nutanix Prism Central API V3
RESTful API for managing Nutanix clusters, VMs, storage, networking, and other infrastructure components through Prism Central. The v3 API uses an intent-based model where resources are defined by their desired state.

**Human URL:** [https://www.nutanix.dev/api_references/prism-central-v3/](https://www.nutanix.dev/api_references/prism-central-v3/)

**Base URL:** `https://{{prism-central-ip}}:9440/api/nutanix/v3`

#### Tags:
 - Cloud Management, Infrastructure, Virtualization

#### Properties
- [Documentation](https://www.nutanix.dev/api_references/prism-central-v3/)
- [Authentication](https://www.nutanix.dev/api_references/prism-central-v3/#authentication)
- [OpenAPI](https://raw.githubusercontent.com/api-evangelist/nutanix/refs/heads/main/openapi/nutanix-prism-central-v3-openapi.yml)

### Nutanix Prism Central API V4
The next-generation v4 API for managing the Nutanix Cloud Platform through Prism Central with GA SDKs for Python, Java, Go, and JavaScript. The v4 API is now the recommended version for production environments.

**Human URL:** [https://www.nutanix.dev/api-reference-v4/](https://www.nutanix.dev/api-reference-v4/)

**Base URL:** `https://{{prism-central-ip}}:9440/api`

#### Tags:
 - Cloud Management, Infrastructure, SDK

#### Properties
- [Documentation](https://www.nutanix.dev/api-reference-v4/)
- [Getting Started](https://www.nutanix.dev/nutanix-api-user-guide/)
- [SDKs](https://www.nutanix.dev/sdk_reference/)
- [Change Log](https://www.nutanix.dev/api-versions/)
- [Developer Portal](https://developers.nutanix.com/)

### Nutanix Prism Element API V2
Cluster-local API for managing individual Nutanix clusters through Prism Element, including storage containers, hosts, virtual machines, and cluster operations.

**Human URL:** [https://www.nutanix.dev/api_references/prism-element/](https://www.nutanix.dev/api_references/prism-element/)

**Base URL:** `https://{{cluster-ip}}:9440/PrismGateway/services/rest/v2.0`

#### Tags:
 - Cluster Management, Infrastructure, Storage

#### Properties
- [Documentation](https://www.nutanix.dev/api_references/prism-element/)
- [OpenAPI](https://raw.githubusercontent.com/api-evangelist/nutanix/refs/heads/main/openapi/nutanix-prism-element-v2-openapi.yml)

### Nutanix Karbon API
API for managing Kubernetes clusters through Nutanix Karbon, including cluster lifecycle, upgrades, and configuration.

**Human URL:** [https://www.nutanix.dev/api_references/karbon/](https://www.nutanix.dev/api_references/karbon/)

#### Tags:
 - Container Management, Kubernetes, Orchestration

#### Properties
- [Documentation](https://www.nutanix.dev/api_references/karbon/)

### Nutanix Database Service API
REST API for Nutanix Database Service (NDB) providing database-as-a-service capabilities for PostgreSQL, MySQL, SQL Server, Oracle, and MongoDB.

**Human URL:** [https://www.nutanix.dev/api_reference/apis/ndb0.9.html](https://www.nutanix.dev/api_reference/apis/ndb0.9.html)

#### Tags:
 - Database, DBaaS

#### Properties
- [Documentation](https://www.nutanix.dev/api_reference/apis/ndb0.9.html)

### Nutanix Cloud Clusters API
REST API for Nutanix Cloud Clusters (NC2), enabling creation and management of Nutanix clusters on AWS and Azure public clouds.

**Human URL:** [https://www.nutanix.dev/api_reference/apis/nc2.html](https://www.nutanix.dev/api_reference/apis/nc2.html)

**Base URL:** `https://api.nutanix.com`

#### Tags:
 - AWS, Azure, Hybrid Cloud

#### Properties
- [Documentation](https://www.nutanix.dev/api_reference/apis/nc2.html)

### Nutanix NCM Self-Service API
API for Nutanix Cloud Manager Self-Service (formerly Calm), enabling automation of application deployment and lifecycle management through blueprints and runbooks.

**Human URL:** [https://www.nutanix.dev/api_references/ncm-self-service/](https://www.nutanix.dev/api_references/ncm-self-service/)

#### Tags:
 - Automation, Application Management, Orchestration, DevOps

#### Properties
- [Documentation](https://www.nutanix.dev/api_references/ncm-self-service/)

### Nutanix Foundation API
API for Foundation and Foundation Central, enabling automated cluster deployment and remote node imaging.

**Human URL:** [https://www.nutanix.dev/api_references/foundation/](https://www.nutanix.dev/api_references/foundation/)

#### Tags:
 - Cluster Deployment, Automation

#### Properties
- [Documentation](https://www.nutanix.dev/api_references/foundation/)

## Common Properties

- [Website](https://www.nutanix.com)
- [Documentation](https://www.nutanix.dev/)
- [Getting Started](https://www.nutanix.dev/nutanix-api-user-guide/)
- [SDKs](https://www.nutanix.dev/sdk_reference/)
- [Reference](https://www.nutanix.dev/api_references/)
- [Code Samples](https://www.nutanix.dev/code_samples/)
- [Change Log](https://www.nutanix.dev/api-versions/)
- [Blog](https://www.nutanix.dev/blog/)
- [Community](https://next.nutanix.com/)
- [Support](https://www.nutanix.com/support-services/product-support)
- [Status](https://status.nutanix.com/)
- [Login](https://my.nutanix.com/)
- [Sign Up](https://my.nutanix.com/page/signup)
- [GitHub Organization](https://github.com/nutanix)
- [Developer Portal](https://developers.nutanix.com/)

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
