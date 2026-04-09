# Consumer Ingestion Guide (Manifest + Index Contract)

## Purpose

This guide explains how a consumer can ingest CDM parquet data into their own analytics system in two ways:

1. Use the existing CDM starter kit workflow.
2. Build a custom ingestion program while adhering to the same manifest and index-file contracts.

If you follow the contracts in this guide, your ingestion remains compatible with the producer model and supports reliable checkpointing.

## Two Supported Approaches

### Approach A: Use the starter kit

Use this if you want a ready-made orchestration pattern.

Starter-kit ingestion flow:

1. `KustoOrchestratorPipeline` reads the subscription catalog manifest.
2. It loops through enabled data products.
3. It calls `KustoIngestionPipeline` per data product/table.
4. A Spark notebook (`KustoSnapshotLoadUsingIndexFile`) resolves `index-latest.csv`, finds latest paths, and ingests parquet into ADX/Kusto.
5. Watermark/checkpoint is stored in internal logs to avoid duplicate reprocessing.

Reference assets in this repo:

- [source/CDM/resources/manifest/v1.0/subscription-catalog.json](../resources/manifest/v1.0/subscription-catalog.json)
- [source/CDM/resources/pipelines/KustoOrchestratorPipeline.json](../resources/pipelines/KustoOrchestratorPipeline.json)
- [source/CDM/resources/pipelines/KustoIngestionPipeline.json](../resources/pipelines/KustoIngestionPipeline.json)
- [source/CDM/resources/notebooks/KustoSnapshotLoadUsingIndexFile.ipynb](../resources/notebooks/KustoSnapshotLoadUsingIndexFile.ipynb)

### Approach B: Build your own ingestion program

Use this if you do not want to use Synapse pipelines/notebooks from the starter kit.

You can implement your own program (for example Python, Spark, .NET, or Java), but it must:

1. Read and honor the manifest contract.
2. Resolve and read `index-latest.csv` for each dataset.
3. Ingest only paths listed in the index file (do not directly scan raw data folders).
4. Maintain durable checkpointing (RunId + Partition minimum).
5. Be idempotent (safe to rerun without duplicate ingestion).

## How the Starter Kit Works (Execution Blueprint)

This is the exact logical flow a consumer should understand before operating or extending the starter kit:

1. **Manifest lookup**: `KustoOrchestratorPipeline` reads `subscription-catalog.json`.
2. **Product filtering**: only `data_products` where `enabled=true` are processed.
3. **Dataset fan-out**: each table in a product is passed to `KustoIngestionPipeline`.
4. **Index resolution**: notebook builds `<...>/index-latest.csv` from manifest metadata.
5. **Path selection**:
  - non-partitioned: latest row by `RunId`
  - partitioned: latest `RunId` per `Partition` (snapshot entries)
6. **Checkpoint gate**: skip ingestion if the effective watermark key already exists:
   - starter-kit style: `(RunId, Partition)` within the dataset-scoped run
   - shared-store style: `(DatasetName, RunId, Partition)`
7. **Load pattern**: ingest into staging table, validate, swap/rename to main table, create/update view.
8. **Commit checkpoint**: watermark is saved only after successful load.

This pattern is what makes reruns safe and prevents duplicate ingestion.

## Required Input Contracts

### 1) Manifest contract (subscription catalog)

Manifest location pattern:

`abfss://config@<storageAccount>.dfs.<storageEnvironmentSuffix>/manifest/v1.0/subscription-catalog.json`

Minimum expected structure:

```json
{
  "schema": "https://cdm.microsoft.com/schemas/data-subscription/v1.0",
  "version": "1.0",
  "subscriber": {
    "tenant_id": "<tenantId>",
    "subscription_id": "<subscriptionId>",
    "org_name": "<orgName>",
    "last_updated_at": "2026-03-30T00:00:00Z"
  },
  "data_products": [
    {
      "name": "Products",
      "enabled": true,
      "output_configuration": {
        "storage_account": "<primaryStorageAccount>",
        "container": "data",
        "base_path": "/commerce-data-model/products/",
        "additional_storage_accounts": ["<secondaryStorageAccount>"]
      },
      "tables": [
        {
          "name": "CDM_Products",
          "version": "3.0",
          "sub_path": "/v3"
        }
      ]
    }
  ]
}
```

How index file path is resolved per table:

`abfss://<container>@<storage_account>.dfs.<storageEnvironmentSuffix><base_path><table_name><sub_path>/index-latest.csv`

### 2) Index file contract (`index-latest.csv`)

Index file is the checkpoint-aware source of truth for what to read.

Expected columns:

| Column | Type | Notes |
|---|---|---|
| RunId | String/Long | Monotonically increasing run identifier from producer |
| Path | String | Full path to parquet location for ingestion |
| RecordCount | Long | Optional but recommended for validation |
| IngestionMode | String | `Snapshot` or `Delta` |
| PublishTime | DateTime | Producer publish timestamp |
| Partition | String | Partition identifier (empty for non-partitioned datasets) |
| AdditionalInfo | String | Optional JSON metadata |

#### Notes on Partition Key

- **Current starter-kit notebook runs once per dataset**: `DatasetName` is already implicit in the notebook execution context and watermark location.
- **Effective starter-kit dedup key is**: `(RunId, Partition)` within that dataset-scoped run.
- **Custom shared checkpoint stores may use three components**: `(DatasetName, RunId, Partition)` when a single dedup store is shared across multiple datasets.
- **Use the `Partition` column directly from index file**: Never parse, extract, or construct the partition value from the `Path` column or hardcode patterns.
- **Partition value is producer-determined**: The copy pipeline on the producer side determines the partition value and writes it to the index file's `Partition` column.
- **Denormalized for consumer convenience**: The `Partition` column is a denormalized, easily accessible value that avoids string manipulation on paths.
  - `Path`: `abfss://data@acct.dfs.core.windows.net/commerce-data-model/charges/.../BillingMonth=202603/...`
  - `Partition`: `202603` (read directly, use as-is)
- **For non-partitioned datasets**: The `Partition` column is empty (null or blank string).
- **Checkpoint storage**:
  - starter kit: store `run_id` + exact `partition` value for that dataset-specific run
  - custom centralized store: store `dataset_name` + `run_id` + exact `partition` value

Example workflow:
```
For each row in index-latest.csv:
  dataset = "CDM_Products"
  run_id = row['RunId']              # e.g., "2026033001"
  partition = row['Partition']        # e.g., "202603" or empty for non-partitioned
  checkpoint_key = (run_id, partition)   # current starter-kit style within a dataset run
  
  if NOT checkpoint_exists(checkpoint_key):
    Load data from row['Path']
    Save checkpoint with this exact key
```

Consumer rule: treat the index file as the single source of truth for data paths and ingestion checkpoints.

Sample `index-latest.csv`:

```csv
RunId,Path,RecordCount,IngestionMode,PublishTime,Partition,AdditionalInfo
2026033001,abfss://data@acct.dfs.core.windows.net/commerce-data-model/products/CDM_Products/v3/Data/RunId=2026033001,125000,Snapshot,2026-03-30T02:00:00Z,,
```

Partitioned sample:

```csv
RunId,Path,RecordCount,IngestionMode,PublishTime,Partition,AdditionalInfo
2026033001,abfss://data@acct.dfs.core.windows.net/commerce-data-model/charges/CDM_Charges_Daily/v3/Data/RunId=2026033001/BillingMonth=202603,8923000,Snapshot,2026-03-30T02:00:00Z,202603,
2026033001,abfss://data@acct.dfs.core.windows.net/commerce-data-model/charges/CDM_Charges_Daily/v3/Data/RunId=2026033001/BillingMonth=202602,8644000,Snapshot,2026-03-30T02:00:00Z,202602,
```

## Checkpointing Rules (Must Follow)

Whether you use the starter kit or custom code, follow these rules:

1. **Current starter-kit notebook key is**: `(RunId, Partition)` because each notebook run is already scoped to a single dataset.
2. **Custom centralized implementations may use**: `(DatasetName, RunId, Partition)` if checkpoints are shared across multiple datasets.
3. **Use `Partition` column directly**: Extract the partition value from the index file's `Partition` column - never parse or derive it from the `Path` column.
4. If checkpoint already exists for the same effective key, skip ingestion for that row.
5. Commit checkpoint only after successful target load validation.
6. Store checkpoint in durable storage (table or file), not in memory.
7. Support reruns safely (idempotent behavior).

Recommended checkpoint schema:

| Field | Example |
|---|---|
| DataProduct | Products |
| DatasetName | CDM_Products |
| Partition | 202603 or empty |
| RunId | 2026033001 |
| LoadedAt | 2026-03-30T02:15:00Z |
| Status | Succeeded/Failed |

## Snapshot and Delta Handling in Current Starter Kit

Current Synapse notebook behavior is **snapshot-oriented**:

1. Partitioned datasets explicitly filter `IngestionMode == "Snapshot"`.
2. Non-partitioned datasets pick latest row by `RunId` and do not implement explicit delta-merge semantics.
3. Watermark logic is by `(Partition, RunId)` to avoid replay, with dataset scope already implied by the per-dataset notebook run.

Implication for consumers:

- If your producer emits both snapshot and delta rows, define a clear policy before ingestion.
- For strict starter-kit parity, publish/consume snapshot-safe latest index entries.

## Guide 1: Create Your Own Ingestion Pipeline (Without Starter Kit)

Use this guide if you want full control over orchestration and target system.

### Design Contract (must keep)

1. Read only enabled data products from manifest.
2. Resolve index path using manifest fields (`storage_account`, `container`, `base_path`, `table`, `sub_path`).
3. Read only parquet paths listed in `index-latest.csv`.
4. **Use the `Partition` column from index file directly** - never derive or construct partition values.
5. Persist checkpoint using:
  - `(RunId, Partition)` if execution is already one dataset per job/notebook
  - `(DatasetName, RunId, Partition)` if the checkpoint store spans multiple datasets
6. Ensure idempotent retries and partial-failure recovery.

### Reference processing algorithm

1. For each enabled product/table, read `index-latest.csv`.
2. For each row in the index:
   - Extract `dataset_name` from manifest/parameters
   - Extract `run_id` from the `RunId` column
   - Extract `partition_id` from the `Partition` column (do not parse or construct from `Path`)
  - Build checkpoint key:
    - starter-kit style: `(partition_id, run_id)` within a dataset-scoped run
    - shared-store style: `(dataset_name, partition_id, run_id)`
3. Check checkpoint store: if the effective key already exists, skip this row.
4. If checkpoint does not exist:
   - Load data from the `Path` column into target store
   - Validate load (row count, checksums, etc.)
  - Write checkpoint with the exact `Partition` column value using the same key structure selected above
- Auditable run metadata (run id, partition, loaded at, status, error if any).

## Guide 2: Enhance the Starter Kit

Use this guide if you want to keep starter-kit contracts but change runtime or target (for example Fabric instead of Kusto).

### Enhancement path A: Replace target database (Fabric, SQL, Snowflake, etc.)

Keep these unchanged:

1. Manifest schema and contract.
2. Index file schema and path resolution.
3. Checkpoint key semantics.

Replace these components:

1. Target writer interface and logic.
2. Table naming, schema, and lifecycle patterns.
3. Validation and observability hooks.

#### Substitution pattern: Kusto → Fabric

Current starter kit uses `KustoIngestionManager` class in the notebook. Here is how to adapt for Fabric:

**Current Kusto pattern:**
```scala
// Staging + Main table swap pattern
CREATE TABLE Staging_CDM_Products FROM /path/to/parquet
// Validate row count
RENAME Staging_CDM_Products → Main_CDM_Products
CREATE VIEW CDM_Products AS SELECT * FROM Main_CDM_Products
```

**Fabric equivalent pattern:**
```python
# Approach 1: Lakehouse managed table promotion
df = spark.read.parquet(parquet_path)
df.write.mode("overwrite").format("delta").option("mergeSchema", "true").save(f"abfss://.../{table_name}_staging")
# Validate row count
# Promote to managed table
spark.sql(f"CREATE TABLE IF NOT EXISTS {table_name} USING DELTA AS SELECT * FROM delta.`abfss://.../{table_name}_staging`")

# Approach 2: Warehouse direct load and swap
spark.sql(f"""
  CREATE TEMP VIEW staging_{table_name} AS
    SELECT * FROM parquet.`{parquet_path}`
""")
spark.sql(f"""
  CREATE OR REPLACE TABLE warehouse.{table_name}
  AS SELECT * FROM staging_{table_name}
""")
```

**Checkpoint storage for Fabric:**
- Option 1: Store in lakehouse managed delta table (similar to current watermark CSV in internal storage)
- Option 2: Use Fabric's job metadata API to persist run state

#### Substitution pattern: Kusto → SQL Database (Azure SQL / Synapse SQL Dedicated)

Current Kusto pattern uses staging/main table swaps for atomic updates.

**Synapse SQL equivalent pattern:**
```sql
-- Create staging table (external table from parquet via PolyBase)
CREATE EXTERNAL TABLE [staging_CDM_Products]
WITH (
  LOCATION = '/path/to/parquet',
  DATA_SOURCE = AzureStorageDs,
  FILE_FORMAT = ParquetFormat
)

-- Copy to internal staging
INSERT INTO [dbo].[Staging_CDM_Products]
SELECT * FROM [staging_CDM_Products]

-- Validate row count
SELECT COUNT(*) FROM [dbo].[Staging_CDM_Products]

-- Swap tables (rename technique for ACID guarantees)
-- Alternatively: truncate main and copy from staging
EXEC sp_rename '[dbo].[Main_CDM_Products]', '[Main_CDM_Products_Old]'
EXEC sp_rename '[dbo].[Staging_CDM_Products]', '[Main_CDM_Products]'
```

**Checkpoint storage for SQL:**
- Create a `Checkpoint` table in same SQL database
- Schema: `(DataProduct, DatasetName, RunId, Partition, LoadedAt, Status, ErrorMsg)`

#### Substitution pattern: Kusto → Snowflake

Current Kusto is columnar, compressed, ingestion-optimized. Snowflake is cloud-data-warehouse oriented with different cost/performance tradeoffs.

**Snowflake equivalent pattern:**
```sql
-- Stage data from Azure ADLS
CREATE OR REPLACE STAGE azure_stage
  URL = 'azure://account.blob.core.windows.net/container/path/'
  CREDENTIALS = (AZURE_SAS_TOKEN = '...');

-- Load into temporary table
CREATE TEMPORARY TABLE staging_CDM_Products AS
  SELECT * FROM @azure_stage/parquet_path/ (FILE_FORMAT = PARQUET_FMT);

-- Validate
SELECT COUNT(*) FROM staging_CDM_Products;

-- Merge into main table (or replace)
ALTER TABLE CDM_Products 
  SWAP WITH staging_CDM_Products;
 -- or: CREATE OR REPLACE TABLE CDM_Products AS SELECT * FROM staging_CDM_Products;
```

**Checkpoint storage for Snowflake:**
- Use Snowflake managed table or dedicated schema
- Enable Snowflake's Time Travel for audit/revert if needed

#### General rules for any substitution

1. **Idempotency**: Your target writer must handle rerun of the same effective key without duplication:
  - dataset-scoped style: `(RunId, Partition)`
  - shared-store style: `(DatasetName, RunId, Partition)`
2. **Checkpoint deduplication**: Use the effective key for your execution model:
   - dataset-scoped style: `(RunId, Partition)`
   - shared-store style: `(DatasetName, RunId, Partition)`
   - in both cases, use the `Partition` column value directly, not parsed from paths
3. **Staging pattern**: Use explicit staging layers to ensure atomicity of final write.
4. **Validation**: Always validate (row count, checksum) before committing checkpoint.
5. **Checkpoint storage**: Persist the effective dedup key plus status metadata in durable storage, using the exact `Partition` value from the index file.
6. **Test reruns**: Verify that reruns do not increase row counts, and checkpoint prevents re-ingestion of the same effective key.
1. Add explicit ingestion policy per dataset (`SnapshotOnly`, `DeltaOnly`, `SnapshotPlusDelta`).
2. For `SnapshotPlusDelta`, process latest snapshot baseline then apply unseen deltas in ordered `RunId`.
3. Extend checkpoint schema to include `IngestionMode` and optional sequence ordering metadata.
4. Add merge/upsert logic at target layer where needed.

### Compatibility guardrails

- Do not break existing manifest contract.
- Do not bypass index-driven ingestion.
- Keep backward compatibility for snapshot-only datasets.
- Version any contract extension (for example manifest `version` increment).

## Consumer Workflow You Can Follow

### If using starter kit

1. Populate `subscription-catalog.json` with your subscribed data products and tables.
2. Ensure producer writes parquet and updates `index-latest.csv` for each dataset.
3. Deploy starter-kit resources and pipelines.
4. Run `KustoOrchestratorPipeline` with manifest path and cluster parameters.
5. Monitor watermark/checkpoint logs and ADX table/view outputs.

### If building your own program

1. Parse manifest and collect enabled `data_products` + `tables`.
2. Resolve `index-latest.csv` path for each dataset.
3. For each row in index file:
   - Read `DatasetName`, `RunId`, and **`Partition` column directly** (don't parse from Path).
  - Build checkpoint key:
    - `(RunId, Partition)` if execution is already dataset-scoped
    - `(DatasetName, RunId, Partition)` if checkpointing is centralized across datasets
   - Skip if checkpoint already exists for this key.
4. Load parquet from the `Path` column.
5. Validate load (row count, checksum policy, etc.).
6. Write checkpoint with the same key structure used for deduplication.
7. Continue to next row.

If you support delta, add an explicit delta ordering and merge/upsert step before writing checkpoint.

## Adding a New Product

To onboard one more product:

1. Add a new `data_products` entry in manifest.
2. Ensure table metadata includes `name`, `version`, and optional `sub_path`.
3. Publish parquet to paths represented in index rows.
4. Publish/update `index-latest.csv` for the new dataset.
5. Run starter-kit pipeline or your custom ingestion program.

No pipeline code change is required if contract remains valid.

## Validation Checklist

- Manifest JSON is valid and has `enabled=true` for intended products.
- `storage_account`, `container`, `base_path`, and table metadata are correct.
- `index-latest.csv` exists and has required columns (including `Partition`).
- `Path` values in index are accessible and point to parquet data.
- Checkpoint store is durable and deduplicates by the effective key for your execution model:
  - starter-kit style: `(RunId, Partition)` per dataset run
  - shared-store style: `(DatasetName, RunId, Partition)`
- Ingestion job is idempotent and safe to rerun for the same checkpoint key.
- Snapshot vs delta policy is explicitly defined per dataset.
- `Partition` column values are read directly from index file, not parsed or constructed from paths.

## Common Pitfalls

- Reading data folders directly instead of using paths from index file.
- Parsing or extracting partition values from `Path` instead of using the `Partition` column directly.
- Advancing checkpoint before successful target commit.
- Ignoring partition granularity or using incomplete checkpoint keys.
- Treating all rows as latest without proper RunId and Partition selection and deduplication.
- Mixing snapshot and delta rows without explicit ordering/merge policy.
- Hardcoding dataset names, run IDs, or partition patterns instead of reading from manifest and index.
- Missing permissions to read ADLS paths referenced by index file.
