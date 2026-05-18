# Frequently Asked Questions

## General

### Q: Is this a Microsoft service?

No. This is an open-source infrastructure template and reference architecture. It is maintained by Microsoft but is not a Microsoft service and carries no SLA or operational support.

### Q: What license is this under?

MIT. Free to use, modify, and distribute with attribution.

### Q: Who owns the deployed infrastructure?

The deploying organization (the data consumer). You own and operate every Azure resource produced by the templates.

## Deployment

### Q: What does this starter kit deploy?

- Azure Storage (data foundation)
- Azure Data Explorer / Kusto (analytics)
- Azure Synapse Analytics (data warehouse)
- RBAC scaffolding (access control)

All resources are created in your subscription, under your control.

### Q: Does this deploy into Microsoft tenants?

No. It is designed for **consumer-owned subscriptions only** and does not connect to or provision Microsoft-managed infrastructure.

### Q: What regions are supported?

All Azure commercial regions, subject to resource availability, your subscription quotas, and any data-residency or compliance requirements you must meet. Verify region/SKU availability before deployment.

### Q: Can I use this in production?

It is a reference architecture. You are responsible for customization, hardening, testing, and operational readiness before deploying production workloads.

## Security

### Q: Does this repo include secrets or credentials?

No. There are no API keys, connection strings, tenant IDs, subscription IDs, or certificate files in this repository. All sensitive values are parameterized and supplied by you at deploy time.

### Q: Is Secret Scanning enabled?

Yes. GitHub Secret Scanning is enabled on this repository.

### Q: How do I report a security vulnerability?

See [SECURITY.md](../SECURITY.md). Report privately, not via a public issue.

## Contributing & Support

### Q: How do I report bugs or request features?

Use the issue templates in [.github/ISSUE_TEMPLATE](../.github/ISSUE_TEMPLATE). For contribution guidelines, see [CONTRIBUTING.md](../CONTRIBUTING.md).

### Q: Can I customize or extend the templates?

Yes. See [extending-the-starter-kit.md](./extending-the-starter-kit.md) for substitution patterns (e.g., swapping the target database) while keeping the manifest and index-file contracts intact.

### Q: Where can I find Bicep documentation?

[Azure Bicep documentation](https://learn.microsoft.com/azure/azure-resource-manager/bicep/overview).
