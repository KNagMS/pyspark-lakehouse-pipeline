# PySpark Lakehouse Pipeline

End-to-end data pipeline built on the medallion architecture (Bronze → Silver → Gold) 
using PySpark and Delta Lake. Includes built-in pipeline observability — every run writes 
metrics (duration, shuffle, executor time) to a Delta metrics table for monitoring and 
cost tracking.

## Architecture
Raw Data (CSV/JSON)

│

▼

┌─────────────┐

│   BRONZE    │  Raw ingestion + schema enforcement + bad row quarantine

└─────────────┘

│

▼

┌─────────────┐

│   SILVER    │  Deduplication + business logic transforms + window functions

└─────────────┘

│

▼

┌─────────────┐

│    GOLD     │  Final aggregations + partitioned + Z-ordered for BI queries

└─────────────┘

│

▼

┌─────────────┐

│  METRICS    │  Job duration · executor time · shuffle GB · per run

└─────────────┘
## Tech Stack

| Layer | Technology |
|---|---|
| Processing | PySpark 3.x |
| Storage | Delta Lake |
| Orchestration | Databricks Workflows |
| Testing | pytest + chispa |
| CI | GitHub Actions |
| Monitoring | Custom metrics → Delta table |

## Project Structure
pyspark-lakehouse-pipeline/

├── pipeline/

│   ├── ingest.py            # Raw ingestion → Bronze Delta table

│   ├── bronze_clean.py      # Dedup + null handling + quality flags

│   ├── silver_transform.py  # Business logic + window functions

│   └── gold_aggregate.py    # Final aggregations + partitioning

├── monitoring/

│   └── job_metrics.py       # Spark REST API metrics collector

├── tests/

│   └── test_transforms.py   # Unit tests for all transformations

├── notebooks/

│   └── dashboard.ipynb      # Metrics dashboard + job duration trends

├── .github/

│   └── workflows/

│       └── ci.yml           # Run pytest on every push

└── README.md
## Key Design Decisions

**Deduplication strategy** — window function with ROW_NUMBER partitioned by 
`customer_id`, ordered by `updated_at DESC`. Idempotent — safe to rerun.

**Observability** — every job run writes one row to a Delta metrics table via 
the Spark REST API (no external dependencies). Metrics include executor time, 
shuffle read/write, peak memory, and failed tasks.

**Partitioning** — Gold layer partitioned by `year/month`, Z-ordered by 
`customer_id` to optimise the most common BI query pattern.

**Testing** — transformations tested with known input/output DataFrames using 
chispa for PySpark-native assertions.

## Status

🚧 In progress — pipeline stages being built week by week.
