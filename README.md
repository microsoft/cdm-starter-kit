# CDM Starter Kit

An Azure infrastructure starter kit for deploying CDM Starter Kit utility resources using Bicep and Azure CreateUIDefinition.

## What This Repository Is

This repository provides **infrastructure-as-code templates and deployment automation** for data consumers who want to establish foundational CDM Starter Kit capabilities in their own Azure environments. It is **not** a Microsoft service, does not deploy into Microsoft tenants, and contains no first-party production code or secrets.

## Who This Is For

- Data consumers establishing CDM Starter Kit practices in their own Azure subscriptions
- Teams needing a structured infrastructure foundation for data analytics workloads

## What It Deploys

This starter kit provisions the following infrastructure components:

- **Azure Storage** - Foundation data storage with proper access controls
- **Azure Data Explorer (Kusto/ADX)** - Analytics and telemetry ingestion
- **Azure Synapse Analytics** - Data warehousing and analytics
- **RBAC Scaffolding** - Role-based access control structure for secure multi-user environments

## Safety & Compliance

### Important Usage Constraints

This starter kit is designed for **consumer-owned Azure subscriptions only**:

- ✅ Deploy into your own Azure subscriptions
- ✅ Verify subscription quotas, target-region service availability, and selected SKUs before deployment
- ❌ This repository **does not contain secrets, API keys, certificates, or tenant-specific credentials**
- ❌ **Never** add secrets, connection strings, or subscription IDs to this repository

### Implementation Details

- **Infrastructure-as-Code**: Uses Azure Bicep for declarative, version-controlled infrastructure
- **UI-Driven Deployment**: Includes Azure CreateUIDefinition forms for guided Azure Portal deployments
- **Consumer-Owned Identity**: All RBAC and access is configured by and for the deploying organization
- **No Microsoft Runtime**: Does not provision Microsoft-managed services or long-running cloud infrastructure on our behalf

## Getting Started

1. Review [CONTRIBUTING.md](./CONTRIBUTING.md) for contribution guidelines
2. See [docs/architecture.md](./docs/architecture.md) for infrastructure overview
3. Check [docs/faq.md](./docs/faq.md) for common questions
4. See [docs/extending-the-starter-kit.md](./docs/extending-the-starter-kit.md) for guidance on enhancing the kit
5. Report security concerns privately via [SECURITY.md](./SECURITY.md)

## Code of Conduct

This project adheres to the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). See [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md) for details.

## License

This project is licensed under the [MIT License](./LICENSE).

## Security & Transparency

- **Secret Scanning**: This repository has Secret Scanning enabled to prevent accidental credential commits
- **No Hardcoded Credentials**: All examples use parameterized deployment with consumer-supplied values
- **Vulnerable Dependency Scanning**: Automated scanning is configured via Dependabot

For security concerns, see [SECURITY.md](./SECURITY.md).

## Support & Contribution

- Contributions are welcome! See [CONTRIBUTING.md](./CONTRIBUTING.md)
- This is a community-supported open-source project, not a Microsoft service with SLAs
- Maintainers review PRs in good faith but provided as-is without production support guarantees

---

**Not a Microsoft service.** Consumer-deployed reference architecture for CDM Starter Kit infrastructure on Azure. Deploy and use at your own discretion.
