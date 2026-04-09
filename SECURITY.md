# Security Policy

## Reporting Security Vulnerabilities

If you believe you have found a security vulnerability in this repository, please report it **privately** and **do not disclose it publicly** until it has been reviewed and addressed.

### How to Report

Please email security concerns to: **[anand.srivastava@microsoft.com](mailto:anand.srivastava@microsoft.com)**

Include:
- Description of the vulnerability
- Steps to reproduce (if applicable)
- Affected components or files
- Potential impact

### Response Timeline

We will acknowledge receipt of your report and work to assess and address the vulnerability. Timelines depend on severity and complexity.

## Security Expectations

### What This Repository Contains

- ✅ Open-source Infrastructure-as-Code (Bicep)
- ✅ Public deployment templates and UI definitions
- ✅ Documentation and configuration examples

### What This Repository Does NOT Contain

- ❌ **No secrets, API keys, or connection strings**
- ❌ **No credentials, certificate files, or private keys**
- ❌ **No Microsoft tenant IDs or customer subscription IDs**
- ❌ **No service principal credentials or managed identities**
- ❌ **No hardcoded authentication tokens or SAS keys**

### Repository Security Features

- **Secret Scanning**: Enabled by default to prevent accidental credential commits
- **Automated Scanning**: Dependabot scans for vulnerable dependencies weekly
- **Branch Protection**: Production branches require reviews before merge
- **Code Review**: All changes reviewed by maintainers before acceptance

## Deployment Security

This is a **reference architecture / starter kit**, not a production service.

**Customers are responsible for:**
- ✅ Deploying into their own Azure subscriptions
- ✅ Configuring appropriate RBAC and access controls
- ✅ Implementing network security policies for their environments
- ✅ Rotating credentials and keys used in their deployments
- ✅ Monitoring and auditing their deployed resources
- ✅ Maintaining compliance with their own organizational policies

**This repository is not responsible for:**
- ❌ Customer execution environments or Azure tenants
- ❌ Customer-supplied credentials or secrets
- ❌ Ongoing operational security of deployed infrastructure
- ❌ Compliance with customer-specific security policies

## Dependency Management

This repository uses Dependabot to automatically check for vulnerable dependencies. See [.github/dependabot.yml](.github/dependabot.yml) for configuration.

## Disclosure Policy

- **Public Disclosure**: After a vulnerability is fixed and a patch is available, the issue may be disclosed publicly
- **Coordinated Disclosure**: We follow responsible disclosure practices and coordinate with affected parties as appropriate

## Questions?

If you have questions about the security of this repository, please open a discussion or contact the maintainers.
