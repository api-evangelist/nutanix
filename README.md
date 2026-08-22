# Nutanix (nutanix)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Nutanix is a hyper-converged infrastructure solution that integrates compute, virtualization, storage, networking, and security to power enterprise applications. Nutanix provides public APIs for managing and automating infrastructure including Prism Central, Prism Element, Karbon Kubernetes, Nutanix Database Service (NDB), Cloud Clusters (NC2), NCM Self-Service, and the GA v4 API platform.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/nutanix/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/nutanix/refs/heads/main/apis.yml)

## Scope

- **Type:** Index
- **Position:** Producer
- **Access:** 1st-Party

## Tags

- Cloud Management
- Hyperconverged
- Infrastructure
- Virtualization
- Kubernetes
- Database

## Timestamps

- **Created:** 2025-03-14
- **Modified:** 2026-05-19

## APIs

### Nutanix Prism Central API V3

RESTful API for managing Nutanix clusters, VMs, storage, networking, and other infrastructure components through Prism Central. The v3 API uses an intent-based model where resources are defined by their desired state.

- **Human URL:** [https://www.nutanix.dev/api_references/prism-central-v3/](https://www.nutanix.dev/api_references/prism-central-v3/)
- **Base URL:** `https://{{prism-central-ip}}:9440/api/nutanix/v3`

#### Tags

- Cloud Management
- Infrastructure
- Virtualization

#### Properties

- [Documentation](https://www.nutanix.dev/api_references/prism-central-v3/)
- [Authentication](https://www.nutanix.dev/api_references/prism-central-v3/#authentication)
- [OpenAPI](https://raw.githubusercontent.com/api-evangelist/nutanix/refs/heads/main/openapi/nutanix-prism-central-v3-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/nutanix-prism-central-v3.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nutanix-prism-central-v3.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Postman Collection](collections/nutanix-prism-element-v2.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nutanix-prism-element-v2.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### Nutanix Prism Central API V4

The next-generation v4 API for managing the Nutanix Cloud Platform through Prism Central with GA SDKs for Python, Java, Go, and JavaScript. The v4 API is now the recommended version for production environments.

- **Human URL:** [https://www.nutanix.dev/api-reference-v4/](https://www.nutanix.dev/api-reference-v4/)
- **Base URL:** `https://{{prism-central-ip}}:9440/api`

#### Tags

- Cloud Management
- Infrastructure
- SDK

#### Properties

- [Documentation](https://www.nutanix.dev/api-reference-v4/)
- [Getting Started](https://www.nutanix.dev/nutanix-api-user-guide/)
- [S D Ks](https://www.nutanix.dev/sdk_reference/)
- [Changelog](https://www.nutanix.dev/api-versions/)
- [Developer  Portal](https://developers.nutanix.com/)
- [Postman Collection](collections/nutanix-prism-central-v3.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nutanix-prism-central-v3.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Postman Collection](collections/nutanix-prism-element-v2.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nutanix-prism-element-v2.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### Nutanix Prism Element API V2

Cluster-local API for managing individual Nutanix clusters through Prism Element, including storage containers, hosts, virtual machines, and cluster operations.

- **Human URL:** [https://www.nutanix.dev/api_references/prism-element/](https://www.nutanix.dev/api_references/prism-element/)
- **Base URL:** `https://{{cluster-ip}}:9440/PrismGateway/services/rest/v2.0`

#### Tags

- Cluster Management
- Infrastructure
- Storage

#### Properties

- [Documentation](https://www.nutanix.dev/api_references/prism-element/)
- [OpenAPI](https://raw.githubusercontent.com/api-evangelist/nutanix/refs/heads/main/openapi/nutanix-prism-element-v2-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/nutanix-prism-central-v3.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nutanix-prism-central-v3.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Postman Collection](collections/nutanix-prism-element-v2.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nutanix-prism-element-v2.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### Nutanix Karbon API

API for managing Kubernetes clusters through Nutanix Karbon, including cluster lifecycle, upgrades, and configuration.

- **Human URL:** [https://www.nutanix.dev/api_references/karbon/](https://www.nutanix.dev/api_references/karbon/)

#### Tags

- Container Management
- Kubernetes
- Orchestration

#### Properties

- [Documentation](https://www.nutanix.dev/api_references/karbon/)
- [Postman Collection](collections/nutanix-prism-central-v3.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nutanix-prism-central-v3.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Postman Collection](collections/nutanix-prism-element-v2.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nutanix-prism-element-v2.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### Nutanix Database Service API

REST API for Nutanix Database Service (NDB) providing database-as-a-service capabilities for PostgreSQL, MySQL, SQL Server, Oracle, and MongoDB.

- **Human URL:** [https://www.nutanix.dev/api_reference/apis/ndb0.9.html](https://www.nutanix.dev/api_reference/apis/ndb0.9.html)

#### Tags

- Database
- DBaaS

#### Properties

- [Documentation](https://www.nutanix.dev/api_reference/apis/ndb0.9.html)
- [Postman Collection](collections/nutanix-prism-central-v3.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nutanix-prism-central-v3.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Postman Collection](collections/nutanix-prism-element-v2.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nutanix-prism-element-v2.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### Nutanix Cloud Clusters API

REST API for Nutanix Cloud Clusters (NC2), enabling creation and management of Nutanix clusters on AWS and Azure public clouds.

- **Human URL:** [https://www.nutanix.dev/api_reference/apis/nc2.html](https://www.nutanix.dev/api_reference/apis/nc2.html)
- **Base URL:** `https://api.nutanix.com`

#### Tags

- AWS
- Azure
- Hybrid Cloud

#### Properties

- [Documentation](https://www.nutanix.dev/api_reference/apis/nc2.html)
- [Postman Collection](collections/nutanix-prism-central-v3.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nutanix-prism-central-v3.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Postman Collection](collections/nutanix-prism-element-v2.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nutanix-prism-element-v2.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### Nutanix NCM Self-Service API

API for Nutanix Cloud Manager Self-Service (formerly Calm), enabling automation of application deployment and lifecycle management through blueprints and runbooks.

- **Human URL:** [https://www.nutanix.dev/api_references/ncm-self-service/](https://www.nutanix.dev/api_references/ncm-self-service/)

#### Tags

- Automation
- Application Management
- Orchestration
- DevOps

#### Properties

- [Documentation](https://www.nutanix.dev/api_references/ncm-self-service/)
- [Postman Collection](collections/nutanix-prism-central-v3.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nutanix-prism-central-v3.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Postman Collection](collections/nutanix-prism-element-v2.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nutanix-prism-element-v2.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### Nutanix Foundation API

API for Foundation and Foundation Central, enabling automated cluster deployment and remote node imaging.

- **Human URL:** [https://www.nutanix.dev/api_references/foundation/](https://www.nutanix.dev/api_references/foundation/)

#### Tags

- Cluster Deployment
- Automation

#### Properties

- [Documentation](https://www.nutanix.dev/api_references/foundation/)
- [Postman Collection](collections/nutanix-prism-central-v3.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nutanix-prism-central-v3.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Postman Collection](collections/nutanix-prism-element-v2.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nutanix-prism-element-v2.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

## Common Properties

- [LinkedIn](https://www.linkedin.com/company/nutanix)
- [Website](https://www.nutanix.com)
- [Documentation](https://www.nutanix.dev/)
- [Getting Started](https://www.nutanix.dev/nutanix-api-user-guide/)
- [S D Ks](https://www.nutanix.dev/sdk_reference/)
- [Reference](https://www.nutanix.dev/api_references/)
- [Code  Samples](https://www.nutanix.dev/code_samples/)
- [Changelog](https://www.nutanix.dev/api-versions/)
- [Blog](https://www.nutanix.dev/blog/)
- [Community](https://next.nutanix.com/)
- [Support](https://www.nutanix.com/support-services/product-support)
- [Status Page](https://status.nutanix.com/)
- [Login](https://my.nutanix.com/)
- [Sign Up](https://my.nutanix.com/page/signup)
- [GitHub Organization](https://github.com/nutanix)
- [Developer  Portal](https://developers.nutanix.com/)
- [Terms of Service](https://www.nutanix.com/legal/terms-of-use)
- [Privacy Policy](https://www.nutanix.com/legal/privacy-notice)
- [L L Ms Txt](https://developers.nutanix.com/llms.txt)

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
