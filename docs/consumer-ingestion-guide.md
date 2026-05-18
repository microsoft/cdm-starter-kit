# Consumer Ingestion Guide (Manifest + Index Contract)

## Purpose

This guide describes the contracts a data consumer must follow when ingesting CDM parquet data, whether by using the starter-kit pipelines as-is or by building a custom ingestion program.

If your implementation honors the contracts below, ingestion stays compatible with the producer model and supports reliable checkpointing.

## Manifest Contract (Subscription Catalog)

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

The index file path for each table is resolved as:

`abfss://<container>@<storage_account>.dfs.<storageEnvironmentSuffix><base_path><table_name><sub_path>/index-latest.csv`

## Index File Contract (`index-latest.csv`)

The index file is the source of truth for what to read.

| Column | Type | Notes |
| --- | --- | --- |
| RunId | String/Long | Monotonically increasing run identifier from the producer |
| Path | String | Full path to the parquet location for ingestion |
| RecordCount | Long | Optional but recommended for validation |
| IngestionMode | String | `Snapshot` or `Delta` |
| PublishTime | DateTime | Producer publish timestamp |
| Partition | String | Partition identifier (empty for non-partitioned datasets) |
| AdditionalInfo | String | Optional JSON metadata |

### Notes on the Partition column

- Read the `Partition` value directly from the index file; do not parse or construct it from `Path`.
- The value is producer-determined and denormalized so consumers do not perform string manipulation on paths.
- For non-partitioned datasets the column is empty.

Example rows:

```csv
RunId,Path,RecordCount,IngestionMode,PublishTime,Partition,AdditionalInfo
2026033001,abfss://data@acct.dfs.core.windows.net/commerce-data-model/products/CDM_Products/v3/Data/RunId=2026033001,125000,Snapshot,2026-03-30T02:00:00Z,,
2026033001,abfss://data@acct.dfs.core.windows.net/commerce-data-model/charges/CDM_Charges_Daily/v3/Data/RunId=2026033001/BillingMonth=202603,8923000,Snapshot,2026-03-30T02:00:00Z,202603,
```

## Checkpointing Rules

Follow these rules whether you use the starter kit or a custom program:

1. The effective deduplication key is `(RunId, Partition)` when execution is already scoped to a single dataset, or `(DatasetName, RunId, Partition)` when checkpoints are shared across datasets.
2. Use the `Partition` value directly from the index file.
3. If the checkpoint already exists for the effective key, skip ingestion for that row.
4. Commit the checkpoint only after a successful target-load validation.
5. Store checkpoints in durable storage (table or file), not in memory.
6. Ingestion must be idempotent and safe to rerun.

Recommended checkpoint schema:

| Field | Example |
| --- | --- |
| DataProduct | Products |
| DatasetName | CDM_Products |
| Partition | 202603 or empty |
| RunId | 2026033001 |
| LoadedAt | 2026-03-30T02:15:00Z |
| Status | Succeeded / Failed |

## Snapshot vs Delta

The reference notebook is snapshot-oriented:

- Partitioned datasets filter `IngestionMode == "Snapshot"`.
- Non-partitioned datasets pick the latest row by `RunId`.
- Watermark logic uses `(Partition, RunId)` within a per-dataset run.

If your producer emits both snapshot and delta rows, define an explicit policy per dataset and order/merge deltas after the latest snapshot baseline before committing checkpoints.

## Reference Processing Algorithm

1. For each enabled product/table, read `index-latest.csv`.
2. For each row:
    - Read `RunId` and `Partition` directly from the row.
    - Build the effective checkpoint key for your execution model.
    - Skip if the checkpoint already exists.
    - Load parquet from `Path`, validate (row count, checksum), then commit the checkpoint.

## Validation Checklist

- Manifest JSON is valid and has `enabled=true` for intended products.
- `storage_account`, `container`, `base_path`, and table metadata are correct.
- `index-latest.csv` exists and has the required columns (including `Partition`).
- `Path` values in the index are accessible and point to parquet data.
- Checkpoint store is durable and deduplicates by the effective key.
- Snapshot vs delta policy is explicitly defined per dataset.

## Common Pitfalls

- Reading data folders directly instead of using paths from the index file.
- Parsing or extracting partition values from `Path` instead of reading the `Partition` column.
- Advancing the checkpoint before a successful target commit.
- Using incomplete checkpoint keys (e.g., omitting `Partition`).
- Mixing snapshot and delta rows without an explicit ordering/merge policy.
- Hardcoding dataset names, run IDs, or partition patterns instead of reading them from the manifest and index.

For guidance on changing the target database or runtime (e.g., Fabric, SQL, Snowflake) while keeping these contracts, see [extending-the-starter-kit.md](./extending-the-starter-kit.md).
