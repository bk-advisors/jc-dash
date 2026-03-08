# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

JC Poultry Farm Dashboard — an R/Quarto analytics platform for tracking financial performance, operational metrics, and statistical process control (SPC) across poultry production batches. Data is sourced live from KoboToolbox via API and from local CSV/XLSX files.

## Rendering Commands

```bash
# Render the main dashboard
quarto render index.qmd

# Render trend analysis reports
quarto render trends.qmd
quarto render trends_jc_farms.qmd

# Render all documents
quarto render
```

RStudio shortcut: Ctrl+Shift+K renders the active `.qmd` file.

## Architecture

### Quarto Documents (entry points)

- **index.qmd** — Main dashboard (`format: dashboard`). Connects to KoboToolbox API, displays batch-level KPIs with per-batch tabs. Uses `bslib` value boxes for KPI cards. Each render also archives raw data to `data/archive/`.
- **trends.qmd** — SPC analysis report (`format: html`). Reads from local CSV data files. Builds control charts (mean +/- 2s/3s) for profit margin, cost per bird, sale price, and operational metrics.
- **trends_jc_farms.qmd** — Operational trends report focusing on bird production efficiency, mortality rates, and feed conversion ratios.

### Shared Functions (`R-scripts/00-functions.R`)

Sourced by `index.qmd`. Contains:

- `jc_dashboard_kpis(data, batch_num, num_birds)` — returns a 5-element character vector: [Revenue, Cost, Profit, Cost/Bird, Profit/Bird]. Defaults to 0 when no revenue or cost rows exist (prevents empty-vector shifting).
- `jc_pnl_by_batch(data, batch_num)` — builds a P&L dataframe with category hierarchy (Revenues > COGS > OPEX > Net Profit/Loss). Uses `df_raw` (original column names: `batch`, `amount_manual`).
- `jc_get_bird_count(data, batch_num, status)` — extracts bird count from data. Closed batches use birds sold (`account_type == "revenues"` quantity); active/selling batches use birds purchased (`category_cogs == "cogs_chicks_purchased"` quantity).
- `jc_render_batch_kpis(kpis, status)` — renders value boxes. Closed/selling: Revenue + Cost + Net Profit. Active: Cost of Production only.
- `jc_render_per_bird_kpis(kpis, status)` — renders per-bird value boxes. Closed/selling: Cost/Bird + Profit/Bird. Active: Cost/Bird only.

### Batch Configuration Pattern (`index.qmd`, setup chunk)

All batches are defined in a single `batch_config` tibble:

```r
batch_config <- tibble(
  batch_num = c("15", "16", "17", "18", "19"),
  status    = c("closed", "closed", "active", "active", "selling")
)
```

Bird counts and KPIs are computed automatically from this config. To add a new batch or change its status, edit this tibble and update the corresponding markdown tab below it (heading text and `status` parameter in render calls).

### Data Pipeline

1. **index.qmd**: KoboToolbox API (`robotoolbox`) > `kobo_data()` > archive to `data/archive/` > rename/filter into `df` > `batch_config` drives KPI computation
2. **trends.qmd / trends_jc_farms.qmd**: Local CSV/XLSX files from `data/` > tidyverse transforms > SPC charts

### Data Archiving

Each render of `index.qmd` saves a flattened copy of the raw KoboToolbox pull to `data/archive/jc_farms_transaction_data_YYYYMMDD.csv`. List columns are collapsed with `"; "` separators.

### Key Conventions

- **Pipe operators**: Mixed `|>` and `%>%` — both acceptable
- **Column names differ by context**: `df` uses `Batch_Number`/`Amount_UGX` (renamed); `df_raw` uses `batch`/`amount_manual` (original). Functions document which they expect.
- **Currency**: UGX (Ugandan Shilling), comma-formatted
- **Account types**: `"revenues"`, `"cogs"`, `"opex"` — lowercase strings
- **Batch status**: `"active"` (still raising birds, no sales), `"selling"` (started selling, uses birds purchased for per-bird metrics), or `"closed"` (finished selling, all metrics available)
- **Indentation**: 2 spaces

### Important Gotcha

`jc_dashboard_kpis` must always return exactly 5 elements. If a batch has no revenue or cost rows, the `pull()` calls default to `0` to prevent `numeric(0)` from silently collapsing the vector positions.

## Dependencies

R packages: tidyverse, lubridate, highcharter, DT, bslib, bsicons, shiny, gt, glue, gtExtras, kableExtra, readxl, scales, formattable, robotoolbox, dm, qcc, plotly, knitr, gridExtra, ggthemes

```r
install.packages(c("tidyverse", "lubridate", "highcharter", "DT", "bslib", "bsicons", "shiny", "gt", "glue", "gtExtras", "kableExtra", "readxl", "scales", "formattable", "robotoolbox", "dm", "qcc", "plotly", "knitr", "gridExtra", "ggthemes"))
```

## Note

`R-scripts/01-data-acquisition.R` and `R-scripts/02-nice-shot-charts.R` are legacy NBA basketball scripts unrelated to the poultry farm dashboard — ignore them.
