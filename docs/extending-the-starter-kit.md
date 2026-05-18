# Extending the Starter Kit

This guide explains how to adapt the starter kit while preserving the manifest and index-file contracts described in [consumer-ingestion-guide.md](./consumer-ingestion-guide.md). Use it when you want to keep the contracts but change the target database or runtime.

## What to Keep Unchanged

1. Manifest schema and contract.
2. Index file schema and path resolution.
3. Checkpoint key semantics (`(RunId, Partition)` or `(DatasetName, RunId, Partition)`).

## What You Can Replace

1. Target writer interface and logic.
2. Table naming, schema, and lifecycle patterns.
3. Validation and observability hooks.

## Substitution Pattern: Kusto → Fabric

The reference implementation uses a staging-plus-rename pattern in Kusto. Adapt for Fabric as follows.

Current Kusto pattern:

```kusto
// Staging + Main table swap
.create table Staging_CDM_Products from '/path/to/parquet'
// Validate row count
.rename tables Staging_CDM_Products to Main_CDM_Products
.create-or-alter function CDM_Products() { Main_CDM_Products }
```

Fabric equivalent:

```python
# Option 1: Lakehouse managed table promotion
df = spark.read.parquet(parquet_path)
df.write.mode("overwrite").format("delta") \
  .option("mergeSchema", "true") \
  .save(f"abfss://.../{table_name}_staging")
# Validate row count, then promote
spark.sql(
    f"CREATE TABLE IF NOT EXISTS {table_name} "
    f"USING DELTA AS SELECT * FROM delta.`abfss://.../{table_name}_staging`"
)

# Option 2: Warehouse direct load + swap
spark.sql(f"CREATE TEMP VIEW staging_{table_name} AS SELECT * FROM parquet.`{parquet_path}`")
spark.sql(f"CREATE OR REPLACE TABLE warehouse.{table_name} AS SELECT * FROM staging_{table_name}")
```

Checkpoint storage options for Fabric:

- A managed delta table in the lakehouse, or
- Fabric's job metadata API for run state.

## Substitution Pattern: Kusto → SQL (Azure SQL / Synapse Dedicated)

```sql
-- Create external table over parquet (PolyBase)
CREATE EXTERNAL TABLE [staging_CDM_Products]
WITH (
  LOCATION = '/path/to/parquet',
  DATA_SOURCE = AzureStorageDs,
  FILE_FORMAT = ParquetFormat
);

-- Copy to internal staging
INSERT INTO [dbo].[Staging_CDM_Products] SELECT * FROM [staging_CDM_Products];

-- Validate row count
SELECT COUNT(*) FROM [dbo].[Staging_CDM_Products];

-- Swap tables
EXEC sp_rename '[dbo].[Main_CDM_Products]', '[Main_CDM_Products_Old]';
EXEC sp_rename '[dbo].[Staging_CDM_Products]', '[Main_CDM_Products]';
```

Checkpoint storage: a `Checkpoint` table in the same database with columns `(DataProduct, DatasetName, RunId, Partition, LoadedAt, Status, ErrorMsg)`.

## Substitution Pattern: Kusto → Snowflake

```sql
CREATE OR REPLACE STAGE azure_stage
  URL = 'azure://account.blob.core.windows.net/container/path/'
  CREDENTIALS = (AZURE_SAS_TOKEN = '...');

CREATE TEMPORARY TABLE staging_CDM_Products AS
  SELECT * FROM @azure_stage/parquet_path/ (FILE_FORMAT = PARQUET_FMT);

SELECT COUNT(*) FROM staging_CDM_Products;

ALTER TABLE CDM_Products SWAP WITH staging_CDM_Products;
-- or: CREATE OR REPLACE TABLE CDM_Products AS SELECT * FROM staging_CDM_Products;
```

Checkpoint storage: a Snowflake managed table or dedicated schema; consider Time Travel for audit/revert.

## General Rules for Any Substitution

1. **Idempotency**: the target writer must tolerate reruns of the same effective key without duplication.
2. **Checkpoint deduplication**: use the effective key for your execution model (`(RunId, Partition)` or `(DatasetName, RunId, Partition)`), using the `Partition` value directly from the index file.
3. **Staging pattern**: use an explicit staging layer to guarantee atomicity of the final write.
4. **Validation**: validate row counts or checksums before committing the checkpoint.
5. **Checkpoint storage**: persist the dedup key and status metadata in durable storage.
6. **Rerun tests**: verify reruns do not increase row counts and that the checkpoint blocks re-ingestion of the same effective key.

## Adding Delta Support

1. Add an explicit ingestion policy per dataset (`SnapshotOnly`, `DeltaOnly`, `SnapshotPlusDelta`).
2. For `SnapshotPlusDelta`, process the latest snapshot baseline, then apply unseen deltas in ordered `RunId`.
3. Extend the checkpoint schema with `IngestionMode` and optional sequence ordering.
4. Add merge/upsert logic at the target layer where needed.

## Compatibility Guardrails

- Do not break the existing manifest contract.
- Do not bypass index-driven ingestion.
- Keep backward compatibility for snapshot-only datasets.
- Version any contract extension (for example, increment the manifest `version`).
