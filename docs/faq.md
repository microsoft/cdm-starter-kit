# Frequently Asked Questions

## General Questions

### Q: Is this a Microsoft service?

**A**: No. This is an **open-source infrastructure template and reference architecture**. It is maintained by Microsoft but is not a Microsoft service with SLAs, support agreements, or operational guarantees.

### Q: What license is this under?

**A**: MIT License. Free to use, modify, and distribute with attribution.

### Q: Who owns the deployed infrastructure?

**A**: The deploying customer. You own and operate all deployed Azure resources in your subscription. Microsoft provides the template only.

### Q: Do I need a Microsoft internal tenant to use this?

**A**: No for standard usage. The primary supported scenario is **customer-owned Azure subscriptions**. Microsoft internal tenant deployment is intended only for authorized maintainers and internal engineering validation, and must follow all applicable internal governance and production policies.

---

## Deployment & Architecture

### Q: What does this starter kit deploy?

**A**: The kit provisions:

- Azure Storage (data foundation)
- Azure Data Explorer / Kusto (analytics)
- Azure Synapse Analytics (data warehouse)
- RBAC scaffolding (access control)

All resources are created in your Azure subscription under your control.

### Q: Does this deploy into Microsoft tenants?

**A**: No. This is explicitly designed for **customer-owned subscriptions only**. It does not:

- Provision resources in Microsoft internal Azure environments
- Connect to Microsoft-managed services
- Create Microsoft-hosted service components
- Deploy first-party production infrastructure

### Q: Can I use this in production?

**A**: This is a **reference architecture and starter kit**. It provides a foundation that you can deploy and then:

- Customize for your specific requirements
- Harden with additional security controls
- Scale based on workload demands
- Integrate with existing infrastructure

You are responsible for assessing suitability, testing, and operational readiness for your production environment.

### Q: What regions does this support?

**A**: This kit is designed to work in all Azure commercial regions. Regional availability depends on:

- Resource availability in your chosen region
- Subscription quotas and capacity availability for the selected services/SKUs
- Data residency requirements
- Compliance and regulatory constraints

Specify your target region during deployment, and verify quotas and SKU availability before rollout.

---

## Security & Credentials

### Q: Does this include secrets or credentials?

**A**: No. This repository contains:

- ❌ Zero hardcoded secrets
- ❌ Zero API keys or connection strings
- ❌ Zero tenant IDs or subscription IDs
- ❌ Zero certificate files or private keys

All sensitive values are parameterized and supplied at deployment time by you.

### Q: How do I provide credentials during deployment?

**A**: You supply credentials during the deployment:

- **Azure Portal UI**: Guided forms capture values
- **Bicep CLI**: Parameters file contains your values
- **Never**: Commit credentials to the repository

Azure Key Vault is recommended for managing sensitive values post-deployment.

### Q: Is Secret Scanning enabled?

**A**: Yes. GitHub Secret Scanning is enabled on this repository to automatically detect and prevent accidental credential commits.

### Q: What if I accidently commit a secret?

**A**: GitHub Secret Scanning will flag it. Immediately:

1. Rotate the secret
2. Create an issue documenting the rotation
3. Work with maintainers to remove the secret from commit history

---

## Data & Compliance

### Q: Where is my data stored?

**A**: Data is stored in Azure resources in the region you specify during deployment. You have full control over data location, encryption, and retention.

### Q: Does Microsoft access my data?

**A**: No. Microsoft does not access customer data or deployed resources. You own and control all data. Microsoft provides only the deployment template.

### Q: Can I use this for regulated workloads (HIPAA, PCI, FedRAMP)?

**A**: The starter kit provides infrastructure components compatible with regulated workloads. However:

- You are responsible for compliance assessment
- You must implement required controls for your regulatory framework
- Consult with your compliance and security teams
- Additional hardening and monitoring are required

### Q: Does this kit implement compliance controls?

**A**: This kit provides a **foundation** compatible with common compliance frameworks. You are responsible for:

- Assessing applicability to your regulatory requirements
- Implementing additional controls as needed
- Auditing and monitoring deployed resources
- Maintaining compliance documentation

---

## Operations & Support

### Q: Is there a Service Level Agreement (SLA)?

**A**: No. This is an open-source project. We provide no SLAs, response time guarantees, or operational support.

### Q: How do I report bugs?

**A**: Use the [Bug Report](../.github/ISSUE_TEMPLATE/bug_report.md) issue template. Include:

- Environment details (Azure region, resource types)
- Steps to reproduce
- Expected vs. actual behavior
- Relevant error messages

### Q: How do I report security vulnerabilities?

**A**: See [SECURITY.md](../SECURITY.md). Report privately via email, not in public issues.

### Q: Can I make changes to this starter kit?

**A**: Yes! See [CONTRIBUTING.md](../CONTRIBUTING.md) for guidelines. Contributions are welcome via pull requests.

### Q: How long will this project be maintained?

**A**: We commit to maintaining the core repository and addressing critical issues. However, as an open-source project, maintenance is best-effort.

---

## Integration & Customization

### Q: Can I integrate this with my existing Azure infrastructure?

**A**: Yes. The Bicep templates are designed to be modular and compatible with existing infrastructure. You can:

- Customize parameter files for your environment
- Reference existing resource groups and VNets
- Adjust RBAC for existing service principals
- Extend templates for additional components

### Q: Can I use this with infrastructure-as-code tools (Terraform, ARM, etc.)?

**A**: This kit uses Azure Bicep. You can:

- Convert Bicep to ARM templates if needed (tools available)
- Use Terraform to orchestrate Bicep deployments
- Reference this architecture in non-Bicep IaC

### Q: Can I customize RBAC?

**A**: Yes. RBAC scaffolding is parameterized. Specify your users, groups, and roles at deployment time or modify post-deployment via Azure Portal.

### Q: Can I remove or replace components?

**A**: Yes. Bicep templates are modular. You can:

- Remove unnecessary components
- Substitute alternative Azure services
- Extend with additional modules

Maintain template structure and document your changes.

---

## Legal & Licensing

### Q: Can I use this for commercial purposes?

**A**: Yes. MIT License permits commercial use, modification, and distribution.

### Q: Do I need to credit Microsoft?

**A**: Per MIT License, you must include the license and copyright notice. No public credit required.

### Q: Can I create a custom bundle or distribution?

**A**: Yes. MIT License allows redistribution. Include the original license file.

### Q: Does this include any non-MIT licensed code?

**A**: No. All code is MIT licensed. Dependencies are listed in documentation.

---

## Troubleshooting

### Q: Deployment fails with "Insufficient permissions"

**A**: Ensure your Azure account has:

- Owner or Contributor role on the target subscription
- Permissions to create resources in the target resource group
- No organization policies blocking resource creation

### Q: How do I customize resource names?

**A**: Bicep parameters control naming. Provide custom prefixes and conventions during deployment.

### Q: How do I delete the deployed resources?

**A**: Use Azure Portal or Azure CLI:

```bash
az group delete --name myResourceGroup --yes
```

This removes all resources in the group.

### Q: Where can I find Bicep documentation?

**A**: See [Azure Bicep documentation](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview)

---

## More Questions?

- Check [issues](https://github.com/microsoft/cdm-starter-kit/issues) for existing Q&A
- Review [architecture.md](./architecture.md) for technical details
- Open a new discussion or issue
- For security questions, see [SECURITY.md](../SECURITY.md)
