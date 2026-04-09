# Development Guide

This guide explains how to set up a local development environment, test changes, and contribute to the CDM Starter Kit.

## Prerequisites

Before you start, ensure you have the following installed:

### Required

- **Azure CLI** (v2.50+)
  - [Installation Guide](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)
  - Verify: `az --version`

- **Bicep CLI** (v0.20+)
  - Installed with Azure CLI by default, or:
  - `az bicep install`
  - Verify: `az bicep --version`

- **Git**
  - [Windows](https://git-scm.com/download/win)
  - [macOS](https://git-scm.com/download/mac)
  - [Linux](https://git-scm.com/download/linux)
  - Verify: `git --version`

- **PowerShell** (v7+) or **Bash**
  - For running deployment scripts
  - [PowerShell Core](https://github.com/PowerShell/PowerShell)

### Optional (Recommended)

- **Visual Studio Code**
  - [Bicep extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-bicep)
  - [Azure Tools extension pack](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack)

- **Azure Storage Explorer**
  - For testing storage account connectivity

---

## Setting Up Your Environment

### 1. Clone the Repository

```bash
git clone https://github.com/microsoft/cdm-starter-kit.git
cd cdm-starter-kit
```

### 2. Authenticate with Azure

```bash
az login
az account set --subscription <your-subscription-id>
```

**Verify authentication:**
```bash
az account show
```

### 3. Validate Bicep Installation

```bash
az bicep version
bicep --version
```

---

## Working with Bicep Files

### Validating Bicep Syntax

Before committing or submitting a PR, validate your Bicep files:

```bash
# Validate a single file
az bicep build --file bicep/main.bicep

# Validate all Bicep files in a directory
for file in bicep/*.bicep; do
  echo "Validating $file..."
  az bicep build --file "$file"
done
```

**Expected output:** No errors, generated ARM template in same directory as `.json`

### Linting Bicep Code

The repository uses `bicepconfig.json` for linting standards. Errors block deployment; warnings are advisory.

**Check linting rules:**
```bash
# bicepconfig.json is in the repo root
# Rules are automatically applied when you run bicep build
az bicep build --file bicep/main.bicep
```

**Common errors you'll see:**
- `adminusername-should-not-be-literal` - Don't hardcode admin usernames
- `outputs-should-not-contain-secrets` - Never output secrets
- `password-properties-should-be-secure` - Use `@secure()` for passwords
- `secure-secrets-in-params` - No secrets in parameter files

### Building Bicep Locally

Generate ARM templates (for testing/understanding):

```bash
az bicep build --file bicep/main.bicep --outdir ./build/
```

Output: `bicep/main.json` (generated ARM template)

---

## Testing Deployments

### Pre-Deployment Checklist

Before validating or deploying, confirm the following in the target subscription and region:

- Required resource providers are registered
- Subscription quotas are sufficient for the planned deployment
- Selected SKUs are available in the target region
- Regional capacity or service restrictions do not block deployment
- Required permissions exist on the subscription and resource group

### Validate without Deployment

Test Bicep syntax and permissions without creating resources:

```bash
az deployment group validate \
  --subscription <subscription-id> \
  --resource-group <resource-group-name> \
  --template-file bicep/main.bicep \
  --parameters @parameters.json
```

**Expected output:** No errors if template and parameters are valid

### What-If Deployment

See what resources will be created/modified without deploying:

```bash
az deployment group what-if \
  --subscription <subscription-id> \
  --resource-group <resource-group-name> \
  --template-file bicep/main.bicep \
  --parameters @parameters.json
```

**Review the output** to ensure expected resources are listed.

### Full Deployment

Deploy to your test resource group:

```bash
az deployment group create \
  --subscription <subscription-id> \
  --resource-group <resource-group-name> \
  --template-file bicep/main.bicep \
  --parameters @parameters.json
```

**After deployment:** Review Azure Portal to verify resources created successfully.

### Cleanup

Delete test resources:

```bash
az group delete \
  --resource-group <resource-group-name> \
  --yes
```

---

## Parameter Files

### Creating a Test Parameters File

Create `parameters.test.json` (NOT tracked in git):

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "projectName": {
      "value": "cdmtest"
    },
    "environment": {
      "value": "dev"
    },
    "location": {
      "value": "eastus"
    }
  }
}
```

**Important:**
- Never commit actual credentials or subscription IDs
- Use `.gitignore` to exclude `parameters.*.local.json` files
- Use parameterized values for all sensitive inputs

---

## Making Changes

### 1. Create a Feature Branch

```bash
git checkout -b feature/your-feature-name
```

**Branch naming conventions:**
- `feature/new-module` - New Bicep modules
- `fix/bug-description` - Bug fixes
- `docs/documentation-topic` - Documentation updates

### 2. Make Your Changes

Edit Bicep files in `bicep/` or documentation files.

### 3. Validate Your Changes

```bash
# Validate Bicep syntax
az bicep build --file bicep/your-file.bicep

# Test deployment (if infrastructure changes)
az deployment group validate \
  --resource-group <test-rg> \
  --template-file bicep/your-file.bicep \
  --parameters @parameters.test.json

# Check for secrets (visual scan)
grep -r "password\|secret\|key\|token" bicep/
```

### 4. Commit Your Changes

```bash
git add .
git commit -m "feat: description of your change"
```

**Commit message format:**
- `feat:` for new features
- `fix:` for bug fixes
- `docs:` for documentation
- `chore:` for tooling/config updates
- `refactor:` for non-breaking code restructuring

### 5. Push and Create a Pull Request

```bash
git push origin feature/your-feature-name
```

Then open a PR on GitHub. See [CONTRIBUTING.md](./CONTRIBUTING.md) for PR requirements.

---

## Testing Checklist

Before submitting a PR, verify:

- [ ] ✅ Bicep files validate without errors
  ```bash
  az bicep build --file bicep/*.bicep
  ```

- [ ] ✅ No hardcoded secrets, credentials, or tenant IDs
  ```bash
  grep -ri "password\|secret\|key\|token\|subscription\|tenant" bicep/
  ```

- [ ] ✅ Parameters are properly parameterized
  - No hardcoded values for environment-specific settings

- [ ] ✅ RBAC assignments use proper role names
  - Avoid hardcoded role GUIDs; use role display names where possible

- [ ] ✅ Deployment tested with `what-if`
  - Review expected resource changes

- [ ] ✅ Documentation updated
  - Updated README.md, docs/architecture.md, or CHANGELOG.md if needed

- [ ] ✅ No breaking changes (or clearly documented)
  - Parameter names unchanged
  - Resource naming conventions consistent

---

## Troubleshooting

### Bicep Build Fails with Linting Error

**Error:** `adminusername-should-not-be-literal`

**Fix:** Don't hardcode the admin username; use a parameter:
```bicep
// ❌ Wrong
param adminUsername string = 'azureuser'

// ✅ Correct
param adminUsername string
```

---

### Deployment Fails with Permission Error

**Error:** `Insufficient privileges to complete the operation`

**Fix:**
```bash
# Ensure you have Contributor or Owner role
az role assignment list --assignee $(az account show --query user.name -o tsv)

# If needed, request access from subscription owner
```

---

### What-If Shows Unexpected Changes

**Problem:** Deployment would create/modify unexpected resources

**Fix:**
1. Review parameter values in your `.json` file
2. Check Bicep variable mappings
3. Verify resource naming conventions
4. Run locally and review ARM template: `az bicep build --file bicep/main.bicep --outdir ./build/`

---

### Bicep Doesn't Validate Locally but Passes in Pipeline

**Problem:** Local validation works, but CI fails

**Fix:**
1. Ensure Azure CLI and Bicep versions match:
   ```bash
   az --version
   az bicep version
   ```
2. Update tools:
   ```bash
   az upgrade
   az bicep upgrade
   ```

---

## Resources

### Official Documentation

- [Azure Bicep Documentation](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/)
- [Bicep Best Practices](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/best-practices)
- [Azure CLI Documentation](https://learn.microsoft.com/en-us/cli/azure/)

### Community

- [Bicep GitHub Discussions](https://github.com/Azure/bicep/discussions)
- [Microsoft Learn - Bicep](https://learn.microsoft.com/en-us/training/modules/fundamentals-bicep/)

### Related

- [CONTRIBUTING.md](./CONTRIBUTING.md) - PR submission guidelines
- [docs/architecture.md](./docs/architecture.md) - Architecture overview
- [SECURITY.md](./SECURITY.md) - Security reporting

---

## Getting Help

1. Check [docs/faq.md](./docs/faq.md) for common questions
2. Search [existing issues](https://github.com/microsoft/cdm-starter-kit/issues)
3. Review [GitHub Discussions](https://github.com/microsoft/cdm-starter-kit/discussions)
4. For security concerns, see [SECURITY.md](./SECURITY.md)

**Happy developing!**
