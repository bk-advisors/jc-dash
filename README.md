# JC Poultry Farm Dashboard

An R/Quarto analytics dashboard for tracking financial performance, operational metrics, and statistical process control (SPC) across poultry production batches at JC Farms.

**Live dashboard:** [bk-advisors.github.io/jc-dash](https://bk-advisors.github.io/jc-dash/)

## Features

- **Batch KPI tabs** — Revenue, cost, net profit, and per-bird metrics for each batch
- **Three-stage batch lifecycle** — Active (raising birds), Selling (sales in progress), Closed (fully sold)
- **Profit & Loss statements** — Auto-generated P&L by batch with COGS/OPEX breakdown
- **SPC trend analysis** — Control charts for profit margin, cost per bird, mortality, and feed conversion
- **Live data** — Pulls transaction data from KoboToolbox API on each render, with automatic archiving

## Quick Start

### Prerequisites

- [R](https://cran.r-project.org/) (>= 4.0)
- [Quarto](https://quarto.org/docs/get-started/)
- A KoboToolbox API token (set via `robotoolbox`)

### Install dependencies

```r
install.packages(c(
  "tidyverse", "lubridate", "highcharter", "DT", "bslib", "bsicons",
  "shiny", "gt", "glue", "gtExtras", "kableExtra", "readxl", "scales",
  "formattable", "robotoolbox", "dm", "qcc", "plotly", "knitr",
  "gridExtra", "ggthemes"
))
```

### Render

```bash
# Main dashboard
quarto render index.qmd

# Trend reports
quarto render trends.qmd
quarto render trends_jc_farms.qmd
```

Or open any `.qmd` file in RStudio and press **Ctrl+Shift+K**.

## Project Structure

```
index.qmd                  Main dashboard (batch KPIs, P&L, data table)
trends.qmd                 SPC analysis (profit margin, cost per bird)
trends_jc_farms.qmd        Operational trends (mortality, FCR, efficiency)
R-scripts/00-functions.R   Shared functions for KPI computation and rendering
data/                      Transaction CSVs and Excel files
data/archive/              Timestamped backups from each KoboToolbox pull
style.css                  Custom styling for trend reports
DASHBOARD_GUIDE.md         Step-by-step guide for common tasks
```

## Documentation

See [DASHBOARD_GUIDE.md](DASHBOARD_GUIDE.md) for instructions on:
- Changing a batch status (active → selling → closed)
- Adding a new batch
- Understanding what each status displays

## Built With

- [Quarto](https://quarto.org/) — document rendering
- [bslib](https://rstudio.github.io/bslib/) — dashboard layout and value boxes
- [robotoolbox](https://docs.ropensci.org/robotoolbox/) — KoboToolbox API integration
- [highcharter](https://jkunst.com/highcharter/) — interactive charts
- [gt](https://gt.rstudio.com/) — formatted tables
