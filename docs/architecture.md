# CDM Starter Kit - Architecture Overview

## High-Level Design

This starter kit provides Infrastructure-as-Code templates to deploy a foundational CDM Starter Kit infrastructure stack on Azure. It is designed as a **customer-deployed reference architecture** on customer-owned Azure subscriptions.

## Core Principles

### 1. Infrastructure-as-Code (Bicep)

All infrastructure is declared using Azure Bicep:

- **Version-controlled** - All configurations tracked in Git
- **Repeatable** - Consistent, idempotent deployments
- **Parameterized** - Customer-supplied values at deployment time
- **Modular** - Components organized for reusability and maintainability

### 2. Customer-Owned Identity & Access

- **RBAC-First**: Role-based access control is the foundation
- **No Service Accounts**: Customers manage all identities
- **Least Privilege**: RBAC scaffolding supports principled access assignment
- **Customer-Controlled**: All access decisions made by deploying organization

### 3. No Microsoft-Managed Runtime

This architecture does NOT provision:

- ❌ Microsoft-hosted services on behalf of the customer
- ❌ First-party production infrastructure
- ❌ Long-running Microsoft managed processes
- ❌ Internal authentication or tenant-scoped components

Instead, it deploys **standard Azure customer-owned resources** that the organization operates directly.

## Infrastructure Components

### 1. Azure Storage

**Purpose**: Foundation data storage layer

- Supports data lakes, backups, and artifact repositories
- RBAC configured by customer for multi-user access
- Diagnostic data and logs stored securely
- Network access restricted via SAS/RBAC policies

**Customer Responsibility**: Configure lifecycle policies, network security, data retention

### 2. Azure Data Explorer (Kusto/ADX)

**Purpose**: High-performance analytics and telemetry ingestion

- Time-series data analysis and exploration
- Real-time dashboard and alerting capabilities
- Data aggregation
- Query-based insights and reporting

**Customer Responsibility**: Query optimization, data retention policies, cluster sizing

### 3. Azure Synapse Analytics

**Purpose**: Data warehousing, ELT/ETL, and advanced analytics

- Structured data warehouse for BI and reporting
- SQL and Spark pools for processing
- Integration with Power BI for visualization
- Cost and resource optimization analysis

**Customer Responsibility**: Workload tuning, compute scaling, metadata management

### 4. RBAC Scaffolding

**Purpose**: Principled access control foundation

- Role assignments for service principals and users
- Audit-ready access management

**Customer Responsibility**: Assigning actual users/services, ongoing access reviews

## Deployment Flow

### Option 1: Azure Portal (CreateUIDefinition)

1. Customer opens **Create** → Search "CDM Starter Kit"
2. Guided form captures:
   - Subscription and resource group
   - Resource naming conventions
   - Compliance/regulatory tags
3. One-click deployment
4. Resources created in customer's subscription

### Option 2: Infrastructure-as-Code (Bicep CLI)

```bash
az deployment group create \
  --resource-group myResourceGroup \
  --template-file main.bicep \
  --parameters @params.json
```

Both paths produce identical resource configurations.

## Security Boundaries

### What Is Secure

✅ Customer controls all identities (AAD users, service principals)
✅ Customer controls RBAC and permissions
✅ Data is encrypted at rest and in transit (Azure defaults)
✅ Audit logs are captured and retained
✅ Network access is configurable via NSGs

### What Requires Customer Hardening

⚠️ Network security policies (firewalls, private endpoints, NSGs)
⚠️ Identity federation and conditional access policies
⚠️ Data retention and compliance scoping
⚠️ Backup and disaster recovery strategies
⚠️ Monitoring and alerting configuration

### What Is NOT Secured

❌ This architecture does not include:
- Multi-tenant isolation (customer-owned resources only)
- Advanced threat detection (customer enablement required)
- Compliance automation (customer responsibility)
- Secrets/credentials management (customer's Azure Key Vault)

## Operational Responsibility Model

| Component | Deployed | Managed | Secured |
|-----------|----------|---------|---------|
| Azure Storage | Starter Kit | Customer | Customer |
| Data Explorer | Starter Kit | Customer | Customer |
| Synapse | Starter Kit | Customer | Customer |
| RBAC Foundation | Starter Kit | Customer | Customer |
| VNets / NSGs | Starter Kit | Customer | Customer |
| Credentials | Customer | Customer | Customer |
| Access Reviews | - | Customer | Customer |
| Monitoring/Alerting | Starter Kit | Customer | Customer |

**Summary**: This kit provides the **infrastructure template**. Customers are responsible for **deployment configuration, operational management, and security hardening**.

## Scaling & Customization

The starter kit is designed for easy customization:

- **SKU Flexibility**: Bicep parameters support resource sizing
- **Regional Deployment**: Works across Azure regions
- **Tagging Strategy**: Customer-defined tag schemas (TBD)
- **Module Reusability**: Components can be extended or replaced
- **Integration**: Designed to connect with existing Azure infrastructure

## Compliance & Governance

- **Audit Logging**: All deployments and access logged to Activity Log
- **Blueprint Support**: Can be packaged as Azure Blueprint
- **Policy-Compatible**: Works with Azure Policy for governance
- **Tagging**: Supports cost allocation and compliance tagging (TBD)
- **Documentation**: Clear parameter descriptions for compliance review

---

**Key Takeaway**: This is a **reference architecture starter kit**, not a production service. Customers deploy, operate, and secure the infrastructure. Microsoft provides the template and examples.
