# CDM Starter Kit - Architecture Overview

## High-Level Design

This starter kit provides Infrastructure-as-Code templates to deploy a foundational analytics stack on Azure. It is a **consumer-deployed reference architecture** intended to run in the data consumer's own Azure subscription.

## Core Principles

### Infrastructure-as-Code (Bicep)

All infrastructure is declared using Azure Bicep:

- Version-controlled, repeatable, idempotent deployments
- Parameterized so values are supplied at deploy time
- Modular components for reuse and maintainability

### Consumer-Owned Identity & Access

- RBAC-first access model
- All identities and role assignments are managed by the deploying organization
- Least-privilege role mappings are surfaced as parameters, not embedded

### No Microsoft-Managed Runtime

This architecture does **not** provision:

- Microsoft-hosted services on the consumer's behalf
- First-party production infrastructure
- Long-running Microsoft-managed processes
- Internal authentication or tenant-scoped components

It deploys **standard Azure resources** that the data consumer operates directly.

## Infrastructure Components

| Component | Purpose |
| --- | --- |
| Azure Storage | Foundation data storage layer |
| Azure Data Explorer (Kusto/ADX) | Analytics and telemetry ingestion |
| Azure Synapse Analytics | Data warehousing and processing |
| RBAC scaffolding | Parameterized role assignments for users and service principals |

Configuration of lifecycle, network security, retention, scaling, and ongoing access reviews is the responsibility of the deploying organization.

## Deployment Flow

### Option 1: Azure Portal (CreateUIDefinition)

1. Open **Create** in the Azure Portal and search for "CDM Starter Kit".
2. The guided form captures subscription, resource group, naming conventions, and tags.
3. Submit to deploy resources into your own subscription.

### Option 2: Bicep CLI

```bash
az deployment group create \
  --resource-group myResourceGroup \
  --template-file main.bicep \
  --parameters @params.json
```

Both paths produce identical resource configurations.

## Security Boundaries

What the template provides out of the box:

- Encryption at rest and in transit (Azure defaults)
- Audit-log capture on supported resources
- Configurable network access via NSGs and resource firewalls

What you need to harden yourself:

- Network policies (private endpoints, firewall rules, NSG tuning)
- Identity assignments (AAD users, groups, service principals)
- Data retention, lifecycle, and backup policies
- Workload-specific compliance controls

See [extending-the-starter-kit.md](./extending-the-starter-kit.md) for patterns to adapt the kit to other target databases or runtimes.
