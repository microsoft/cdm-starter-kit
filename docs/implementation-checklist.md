# CDM Starter Kit - Implementation Capability Checklist

Use this checklist to verify implementation completeness against:
- `README.md`
- `docs/faq.md`
- `docs/architecture.md`
- `CHANGELOG.md`
- `bicepconfig.json`

## 1) Product Scope & Positioning

- [ ] Treated as a **starter kit / reference architecture**, not a managed Microsoft service
- [ ] Deployment target is **customer-owned Azure subscriptions**
- [ ] Internal tenant usage limited to authorized maintainer/internal validation scenarios
- [ ] No SLA/support guarantee claims in docs or release notes
- [ ] No Microsoft Fabric dependency or deployment requirement

## 2) Core Capability Implementation

### Azure Storage
- [ ] Data foundation storage deployed
- [ ] Access model uses customer-managed RBAC/SAS policies
- [ ] Diagnostic logging enabled/configurable
- [ ] Lifecycle/retention policies configurable

### Azure Data Explorer (Kusto/ADX)
- [ ] ADX resources deploy successfully
- [ ] Ingestion + analytics flow implemented for target scenario
- [ ] Retention and cluster sizing parameters defined

### Azure Synapse Analytics
- [ ] Synapse workspace/resources deploy successfully
- [ ] SQL/Spark processing path validated for expected workloads
- [ ] Integration points (e.g., BI/reporting) documented/configured as needed

### RBAC Scaffolding
- [ ] Role assignment templates are parameterized
- [ ] Least-privilege role mappings are documented
- [ ] Customer identity/group assignment process defined

## 3) Security & Credential Hygiene

- [ ] No secrets/keys/certificates committed to repo
- [ ] No tenant IDs / subscription IDs hardcoded in tracked files
- [ ] Sensitive values provided via secure parameters at deploy time
- [ ] Security reporting mailbox is valid and monitored
- [ ] Secret scanning workflow is enabled and passing

## 4) Deployment & Operations

- [ ] Bicep CLI deployment path tested (`az deployment ...`)
- [ ] Guided portal deployment path (CreateUIDefinition) aligned with actual implementation status
- [ ] Regional deployment constraints are documented
- [ ] Monitoring/alerting baseline defined
- [ ] Rollback or cleanup process documented and validated

## 5) Data, Governance & Compliance Readiness

- [ ] Data residency and storage region decisions documented
- [ ] Required compliance controls are mapped to customer responsibilities
- [ ] Audit logging and evidence collection approach defined
- [ ] Cost/compliance tagging strategy defined and applied
- [ ] Hardening items clearly separated from base starter kit functionality

## 6) Versioning & Release Discipline (from CHANGELOG)

- [ ] `CHANGELOG.md` kept in Keep-a-Changelog format
- [ ] Semantic Versioning rules enforced for release decisions
- [ ] Unreleased section maintained with current changes
- [ ] Release tags follow `vX.Y.Z`
- [ ] Breaking changes explicitly called out in release notes

## 7) Bicep Quality Gates (from bicepconfig.json)

### Must-pass errors
- [ ] `adminusername-should-not-be-literal`
- [ ] `outputs-should-not-contain-secrets`
- [ ] `password-properties-should-be-secure`
- [ ] `secure-secrets-in-params`

### Warning-level controls reviewed
- [ ] Hardcoded location/API version checks reviewed
- [ ] `secure-parameter-default` and nested deployment security checks reviewed
- [ ] Complexity limits (`max-*`) reviewed where applicable
- [ ] Dependency/resource hygiene warnings (`redundant-dependson`, nesting, interpolation) reviewed

## 8) Repository Governance & PR Quality

- [ ] CODEOWNERS references valid and active GitHub teams
- [ ] Dependabot reviewers point to valid team/users
- [ ] PR template checklist is being used by contributors
- [ ] Required status checks align with workflow job names
- [ ] Branch protection is configured (app-managed or manual)

## 9) CODEOWNERS Validation Snapshot (Current)

Validated against current repository paths and files:

### Exists
- [x] `README.md`
- [x] `CONTRIBUTING.md`
- [x] `docs/`
- [x] `SECURITY.md`
- [x] `CODE_OF_CONDUCT.md`
- [x] `LICENSE`
- [x] `.github/`
- [x] `bicepconfig.json`
- [x] `bicep/`
- [x] `ui/`
- [x] `deploy/`
- [x] `*.ps1` files
- [x] `*.sh` files

### Missing / Needs attention
- [ ] `.gitignore` (not found under `source/CDM`)
- [ ] `test/` folder (not found under `source/CDM`)
- [ ] CODEOWNERS still references `@microsoft/cdm-team` for test/deploy/script patterns (legacy handle)

## 10) Definition of Done (Go/No-Go)

- [ ] Clean deployment succeeds in target region/subscription
- [ ] Re-deployment is idempotent
- [ ] Security/lint/PR validation workflows pass
- [ ] Core Storage + ADX + Synapse + RBAC scenarios validated
- [ ] Docs accurately reflect implemented behavior (no scope drift)

---

## Notes

- This checklist is intentionally implementation-focused and should be used as sprint acceptance criteria.
- For each checked item, link evidence (PR, pipeline run, deployment output, test artifact).