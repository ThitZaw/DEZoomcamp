# SEA Forest Monitor

## Technical Stack Decision --- Version 1.0

**Related document:** [`plan.md`](plan.md) (Project Plan v1.0, Section 11)\
**Decision:** Option B --- Google Cloud / Data Engineering Zoomcamp stack\
**Hard constraint:** **Zero cost.** The project must not create any charge
at any time.

------------------------------------------------------------------------

## 1. Summary

| Layer | Technology | Runs where | Cost |
|---|---|---|---|
| Infrastructure as code | Terraform | Local machine | Free |
| Raster extraction | Google Earth Engine (Python API) | Google (noncommercial) | Free |
| Boundaries | Natural Earth admin-0, GeoPandas | Local machine | Free |
| Data lake | Google Cloud Storage | GCP (Free Trial credit) | $0 billed |
| Data warehouse | BigQuery | GCP (Free Trial credit) | $0 billed |
| Transformation | dbt Core (`dbt-bigquery`) | Local machine | Free |
| Orchestration | Kestra (open source) | **Local Docker only** | Free |
| Data quality | dbt tests + rejected-records and run-log tables | BigQuery | $0 billed |
| Dashboard | Looker Studio (free edition) | Google | Free |
| Permanent dashboard data | Google Sheets (published mart snapshot) | Google Drive | Free |
| CI | GitHub Actions (public repo) | GitHub | Free |

## 2. Zero-Cost Strategy

### 2.1 The core rule: use the Free Trial and never upgrade

New Google Cloud accounts get a **Free Trial** (currently $300 of credit
for 90 days). Google requires a card to verify identity, but **a Free
Trial account is not charged unless you manually upgrade it to a paid
(full) account.** When the credit or the 90 days run out, resources are
paused, not billed.

Rules:

1. **Never click "Upgrade" / "Activate full account".**
2. Note the trial end date in your calendar.
3. Before it ends, publish the final data to Google Sheets (Section 2.3)
   so the public dashboard keeps working forever.

A card verification may show a temporary authorization hold on your
statement. This is not a charge and is released automatically.

> Free Trial terms can change. Re-read the current terms on the sign-up
> page before creating the account, and confirm the "no charge unless
> you upgrade" wording is still there.

### 2.2 Additional safety settings (set them anyway)

Even within the trial, apply these so the credit is never wasted and
bad habits are not formed:

| Setting | Where | Value |
|---|---|---|
| Budget alert | Billing → Budgets & alerts | $1 budget, alerts at 50% / 90% / 100% |
| BigQuery query cap | IAM & Admin → Quotas → *Query usage per day* | 10 GB/day |
| Region | Terraform variables | BigQuery `US`, GCS `us-central1` |
| Table design | dbt models | Partition/cluster by `year`; no `SELECT *` in dashboards |

### 2.3 How the dashboard stays free after the trial

The final mart is about 125 rows (up to 5 countries × 25 years). That fits
easily in a Google Sheet.

- The last pipeline step exports the mart tables
  (`mart_country_year`, `mart_country_summary`, `dim_country`) to a
  Google Sheet using the Sheets API (free, no billing needed).
- The **public** Looker Studio report reads from that Sheet.
- During development, the same report can point to BigQuery directly.
- A CSV snapshot of the marts is also committed to `data/published/` so
  the results are reproducible even without any Google account.

After the trial, the BigQuery and GCS parts become a documented,
reproducible pipeline that anyone (including you, with a new trial) can
run again. The public dashboard does not depend on them.

### 2.4 Never use these (they cost money)

- Compute Engine VMs (including hosting Kestra on a VM)
- Cloud Composer (managed Airflow), Dataproc, Dataflow
- Cloud SQL, AlloyDB, Memorystore
- Cloud Run / Cloud Functions with minimum instances
- BigQuery BI Engine, BigQuery reservations / editions
- Looker Studio Pro, dbt Cloud paid plans, Kestra Cloud / Enterprise
- Earth Engine commercial plans
- Paid APIs (e.g. Maps Platform)

## 3. Component Details

### 3.1 Extraction --- Google Earth Engine

- Register the Cloud project for **noncommercial** Earth Engine use. A
  personal portfolio project qualifies. Noncommercial use is free and
  has its own quota tier.
- Upload simplified Natural Earth admin-0 polygons for the candidate
  countries as an Earth Engine asset.
- For each country:
  - mask `treecover2000 >= 30`
  - compute area with `ee.Image.pixelArea()` (m² → hectares ÷ 10,000);
    never use pixel count × 900 m², because Hansen pixels are in degrees
    and shrink with latitude
  - group by `lossyear` to get annual loss, and sum for baseline area
- Use `Export.table.toCloudStorage` (batch export), not `getInfo()`,
  because large countries such as Indonesia exceed interactive limits.
- Record `dataset_version` (e.g. `UMD/hansen/global_forest_change_20XX_v1_X`),
  extraction timestamp and run ID with every row (DQ-07).
- Fallback that needs no billing at all: `Export.table.toDrive`.

### 3.2 Storage --- GCS and BigQuery

| Zone | Location | Contents |
|---|---|---|
| Raw | `gs://<project>-sea-forest-raw/hansen/<version>/<run_id>/` | GEE CSV exports, boundary GeoJSON |
| Staging | BigQuery `sea_forest_staging` | External/loaded raw tables |
| Marts | BigQuery `sea_forest_marts` | Cleaned, aggregated outputs |
| Audit | BigQuery `sea_forest_audit` | `pipeline_runs`, `rejected_records` |

Total size is well under 1 GB.

### 3.3 Transformation --- dbt Core

Models:

- `stg_hansen_loss`, `stg_hansen_baseline`, `stg_countries`
- `int_country_year_loss` --- annual loss per country, year and version
- `mart_country_year` --- annual loss, cumulative loss (window function),
  cumulative loss % = cumulative loss ÷ baseline × 100
- `mart_country_summary` --- totals and rankings for the overview page

**Natural key:** `(country_iso3, year, dataset_version)`. Incremental
models use `merge` on this key, so reruns never duplicate rows (§8.3) and
a new Hansen version is loaded side by side (historical corrections).

### 3.4 Orchestration --- Kestra (local)

- Run with the official `docker-compose.yml` on your own machine.
- One flow, `sea_forest_pipeline`, triggered manually (§8.1), with inputs:
  `dataset_version`, `countries`, `start_year`, `end_year`, `full_refresh`.
- Tasks: extract (GEE) → validate inputs → load to BigQuery → `dbt build`
  → write run log → publish to Google Sheets.
- Service-account key is stored as a Kestra secret, **never** committed
  to git.

### 3.5 Data quality mapping

| ID | Implementation |
|---|---|
| DQ-01 | dbt `not_null` + `relationships` to `dim_country` |
| DQ-02 | Validation step routes invalid rows to `rejected_records`; dbt `not_null` |
| DQ-03 | dbt `unique_combination_of_columns` on the natural key |
| DQ-04 | dbt `accepted_range` (min 0); negative rows rejected |
| DQ-05 | Custom dbt test: cumulative loss never decreases within a version |
| DQ-06 | Model that cross-joins expected country × year and reports gaps; missing stays `NULL`, never `0` |
| DQ-07 | `dataset_version`, `source`, `extracted_at` columns + `pipeline_runs` |
| DQ-08 | `rejected_records(run_id, record, reason, rejected_at)` |

### 3.6 Dashboard --- Looker Studio

Three pages as specified in the plan (§6): Regional Overview (KPI
scorecards + filled-map geo chart by ISO country code), Country
Comparison (bar/line charts, ranking table) and Historical Trends.
Country and year are report-level filter controls. Attribution to Hansen
et al. (CC BY 4.0) and Natural Earth appears on every page.

### 3.7 Testing and CI

- `pytest` for extraction/validation code, with a small fixture of known
  values.
- `dbt build` against a dev dataset for DQ tests.
- GitHub Actions (free for public repos): lint, `pytest`,
  `terraform fmt -check`, `terraform validate`. CI does **not** call GCP,
  so it needs no credentials and costs nothing.

## 4. Setup Checklist

1. Create a Google account for the project (optional but tidy).
2. Start the Google Cloud **Free Trial**. Do not upgrade.
3. Create a budget alert ($1) and the BigQuery daily query quota.
4. Register the project for noncommercial Earth Engine.
5. Create a service account with only BigQuery, Storage and Earth Engine
   roles; store the key outside the repo (`.gitignore` it).
6. `terraform apply` to create the bucket and datasets.
7. Start Kestra locally with Docker Compose.
8. Run the pipeline, build the dashboard.
9. Publish marts to Google Sheets and switch the public dashboard source.
10. Before the trial ends: confirm the Sheet-based dashboard works, then
    optionally `terraform destroy`.

## 5. Open Items (data validation stage)

- Confirm which Hansen GFC version includes 2025 loss, and whether any
  candidate country has gaps.
- Check Natural Earth coastline resolution (10m) and disputed-area
  handling for the candidate countries.
- Confirm current Earth Engine noncommercial quota and Free Trial terms
  at sign-up time.
