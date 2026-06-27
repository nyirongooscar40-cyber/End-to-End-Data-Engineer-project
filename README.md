# End-to-End Azure Data Engineering Project — Metadata-Driven Ingestion Pipeline

## Overview

This project implements a **dynamic, metadata-driven data ingestion pipeline** using **Azure Data Factory (ADF)** to copy multiple AdventureWorks CSV datasets from a public GitHub repository into **Azure Data Lake Storage Gen2 (ADLS Gen2)**.

Rather than building a separate Copy Activity for every source file, the pipeline uses a single **Lookup → ForEach → Copy** pattern driven entirely by a configuration file (`file_list.json`). This means new source files can be added to the pipeline simply by adding a new entry to the config — with **zero changes to the pipeline itself**.

## Architecture


GitHub (raw CSV files)
        │
        ▼
┌───────────────────┐
│   Lookup Activity  │  → Reads file_list.json (array of source file metadata)
└───────────────────┘
        │
        ▼
┌───────────────────┐
│  ForEach Activity  │  → Iterates over each file entry
└───────────────────┘
        │
        ▼
┌───────────────────┐
│   Copy Activity    │  → Pulls each CSV via HTTP and lands it in ADLS Gen2
└───────────────────┘
        │
        ▼
   ADLS Gen2 (Raw / Bronze layer)
   /raw/<sink_folder>/<file_name>
```

## How it works

1. **`file_list.json`** is a single configuration file containing an array of objects, one per source file:
      json
   {
     "p_relative_url": "relative path to the file on GitHub",
     "p_sink_folder": "destination folder name in ADLS",
     "p_file_name": "destination file name"
   }
   
2. A **Lookup activity** reads this JSON file and returns the full array of file metadata.
3. A **ForEach activity** iterates over each item in that array.
4. Inside the ForEach, a **Copy activity** dynamically builds:
   - the **source URL** by concatenating the GitHub raw content base URL with `p_relative_url`
   - the **sink path** in ADLS Gen2 using `p_sink_folder` and `p_file_name`
5. Each CSV file is landed in its own folder under the raw/bronze zone of the data lake, ready for downstream transformation.

## Why metadata-driven design?

This pattern is a standard data engineering best practice because it:
- **Scales without code changes** — adding a new file to ingest is a one-line config change, not a new pipeline activity
- **Reduces maintenance overhead** — one Copy activity definition handles every file, instead of N near-identical Copy activities
- **Separates configuration from logic** — file paths and names live in data (JSON), not embedded in the pipeline
- **Mirrors real-world enterprise patterns** — this is how production ingestion frameworks are typically built at scale (e.g. ingesting hundreds of source tables from an ERP or CRM system)

## Source Data

The pipeline ingests the **AdventureWorks sample dataset**, including:
- Customers
- Calendar
- Product Subcategories
- Products (selected columns)
- Sales (2015, 2016, 2017)
- Returns
- Territories

## Tech Stack

- **Azure Data Factory** — orchestration (Lookup, ForEach, Copy activities)
- **Azure Data Lake Storage Gen2** — raw/bronze landing zone
- **GitHub** — source file hosting (raw CSV over HTTP)
- **JSON** — metadata-driven configuration

## Project Structure

├── pipeline/
│   └── ingest_adventureworks_pipeline.json   # ADF pipeline definition
├── config/
│   └── file_list.json                        # Metadata-driven file list
├── datasets/
│   ├── http_source_dataset.json
│   └── adls_sink_dataset.json
└── README.md


## Next Steps / Roadmap

- [ ] Add Silver layer transformation (Databricks/PySpark) for cleaning and conforming the raw CSVs
- [ ] Add Gold layer modelling (star schema for sales/returns analysis)
- [ ] Add data quality checks (schema validation, null checks, row counts)
- [ ] Parameterize the pipeline further to support multiple source repos/environments (dev/test/prod)
- [ ] Add CI/CD deployment via Azure DevOps or GitHub Actions

## Author

**Oscar Nyirongo** — Data Engineer & AI Automation Specialist
GitHub: [nyirongooscar40-cyber](https://github.com/nyirongooscar40-cyber)
