# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial open-source release setup
- Microsoft OSS-compliant repository structure
- Core documentation (README, CONTRIBUTING, SECURITY, CODE_OF_CONDUCT, SUPPORT)
- Architecture and FAQ documentation
- Bicep module structure (Storage, Synapse, Kusto/Data Explorer, RBAC)
- GitHub issue templates (bug report, security report)
- Pull request template with compliance checklist
- Dependabot configuration for dependency scanning
- bicepconfig.json for Bicep linting standards
- CODEOWNERS file for maintainer assignments
- GitHub Actions workflows (Bicep linting, secret scanning, PR validation)
- Branch protection policy guide

### Planned
- CreateUIDefinition forms for Azure Portal deployment
- GitHub Actions workflows (Bicep linting, secret scanning, PR validation)
- Sample deployment parameters
- Integration tests
- User guides and deployment walkthroughs

---

## Versioning Notes

**0.1.0 (Initial Release - Planned)**
- First stable release with core Bicep modules
- CreateUIDefinition UI forms
- Complete documentation set
- GitHub Actions CI/CD workflows

---

## Guidelines for Future Releases

### For Contributors

When adding changes, consider:

1. **Breaking Changes** → Increment MAJOR version (1.0.0 → 2.0.0)
   - Changes to Bicep parameter names
   - Changes to resource naming conventions
   - RBAC assignment modifications

2. **New Features** → Increment MINOR version (1.0.0 → 1.1.0)
   - New Bicep modules
   - New CreateUIDefinition forms
   - New deployment options

3. **Bug Fixes & Documentation** → Increment PATCH version (1.0.0 → 1.0.1)
   - Bicep validation fixes
   - Documentation updates
   - Non-breaking improvements

### Release Process

- Tag releases as `v0.1.0`, `v0.2.0`, etc. in Git
- Include release notes matching CHANGELOG format
- Document any breaking changes explicitly
- Update README for major version changes

---

## Archive

### Initial Commit
- Established Microsoft OSS compliance baseline
- Set up repository governance (CODEOWNERS, branch protection)
- Created documentation structure
