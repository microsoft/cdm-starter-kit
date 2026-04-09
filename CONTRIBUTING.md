# Contributing to CDM Starter Kit

Thank you for your interest in contributing! We welcome contributions from the community.

## Before You Start

Please take a moment to review:

- [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md) - Our community standards
- [SECURITY.md](./SECURITY.md) - Security reporting and expectations
- This file - Guidelines for contributions

## How to Contribute

### Reporting Issues

Found a bug or have a suggestion?

1. Check [existing issues](https://github.com/microsoft/cdm-starter-kit/issues) to avoid duplicates
2. Open a new issue using the appropriate template:
   - [Bug Report](.github/ISSUE_TEMPLATE/bug_report.md)
   - [Security Issue](.github/ISSUE_TEMPLATE/security_report.md)
3. Include relevant details: environment, steps to reproduce, expected vs. actual behavior

### Proposing Changes

1. **Fork** this repository
2. **Create a branch** for your changes (`git checkout -b feature/your-feature`)
3. **Make your changes** and test locally
4. **Commit** with clear, descriptive messages
5. **Push** to your fork
6. **Open a Pull Request** and reference any related issues

### Pull Request Guidelines

Before submitting a PR, ensure:

- ✅ **No secrets or credentials** - No API keys, connection strings, tenant IDs, or subscription IDs
- ✅ **No tenant-specific values** - All examples use parameterized inputs
- ✅ **Documentation updated** - If behavioral changes, update relevant docs
- ✅ **Code follows conventions** - Consistent formatting and naming
- ✅ **Tests pass** - If applicable, ensure your changes don't break existing functionality
- ✅ **License compliance** - Your contribution will be licensed under MIT

See [.github/PULL_REQUEST_TEMPLATE.md](.github/PULL_REQUEST_TEMPLATE.md) for our PR checklist.

## Review Process

- Pull requests are reviewed by maintainers in good faith
- You may be asked for clarifications or changes
- Reviews may take time; please be patient
- Once approved, your PR will be merged

## Scope & Limitations

### What We Welcome

✅ Infrastructure improvements and bug fixes
✅ Documentation enhancements
✅ Security issues (reported privately via [SECURITY.md](./SECURITY.md))
✅ Sample configurations and best practices

### What We Cannot Accept

❌ Microsoft internal code or proprietary assets
❌ Secrets, credentials, or sensitive configuration
❌ Tenant-specific or customer-identifying information
❌ Code designed for Microsoft internal environments
❌ Changes requiring Microsoft internal dependencies

## Important Notes

- **No SLA or Production Support**: This is a community project, not a Microsoft service. Contributions and responses are provided on a best-effort basis.
- **Reference Architecture**: This is a starter kit and reference implementation. Production deployments require customer validation and additional controls.
- **Your Responsibility**: When you deploy resources, you are responsible for their security, compliance, and operational management.

## Licensing

By contributing, you agree that your contributions will be licensed under the [MIT License](./LICENSE).

## Questions?

If you have questions about contributing:

1. Review [docs/faq.md](./docs/faq.md)
2. Check existing issues and discussions
3. Open a new discussion or issue

Thank you for helping make this project better!
