# JC Poultry Farm Dashboard — User Guide

## How to Render the Dashboard

In RStudio, open `index.qmd` and press **Ctrl+Shift+K**, or run in the terminal:

```bash
quarto render index.qmd
```

Each render pulls fresh data from KoboToolbox and saves a backup CSV to `data/archive/`.

---

## Common Tasks

### When you start selling a batch (e.g. Batch 17)

Two changes in `index.qmd`:

**1. Update `batch_config`** (setup chunk, ~line 99):

```r
# Change "active" to "closed" for Batch 17
batch_config <- tibble(
  batch_num = c("15", "16", "17", "18", "19"),
  status    = c("closed", "closed", "closed", "active", "active")
                                    # ^^^^^^^ was "active"
)
```

**2. Update the tab heading** (~line 142):

```
# Batch 17 (Closed)
```

**3. Remove `status = "active"` from the render calls** so it defaults to `"closed"`:

```r
# Before:
jc_render_batch_kpis(all_kpis[["kpis17"]], status = "active")
jc_render_per_bird_kpis(all_kpis[["kpis17"]], status = "active")

# After:
jc_render_batch_kpis(all_kpis[["kpis17"]])
jc_render_per_bird_kpis(all_kpis[["kpis17"]])
```

This switches the batch to show all 5 KPIs (Revenue, Cost, Net Profit, Cost/Bird, Profit/Bird) and uses birds sold for per-bird calculations.

---

### Adding a new batch (e.g. Batch 20)

**1. Add a row to `batch_config`** in the setup chunk of `index.qmd`:

```r
batch_config <- tibble(
  batch_num = c("15", "16", "17", "18", "19", "20"),
  status    = c("closed", "closed", "active", "active", "active", "active")
)
```

**2. Add a new tab section** in `index.qmd`, before the "Batch Trends and Patterns Analysis" heading:

```
# Batch 20 (Active)

```{r}
jc_render_batch_kpis(all_kpis[["kpis20"]], status = "active")
```

## Per Bird

```{r}
jc_render_per_bird_kpis(all_kpis[["kpis20"]], status = "active")
```
```

**3. Update the P&L loop range** if needed (currently covers batches 4–19):

```r
for (bn in as.character(4:20)) {
```

That's it — bird counts and KPIs are computed automatically from the data.

---

### Understanding what each status shows

| Metric | Active | Closed |
|--------|--------|--------|
| Cost of Production | Shown | Shown |
| Cost per Bird | Shown | Shown |
| Revenue | Hidden | Shown |
| Net Profit | Hidden | Shown |
| Profit per Bird | Hidden | Shown |

- **Active** uses birds purchased for per-bird calculations
- **Closed** uses birds sold for per-bird calculations

---

### Data archiving

Every time the dashboard renders, a snapshot of the raw KoboToolbox data is saved to:

```
data/archive/jc_farms_transaction_data_YYYYMMDD.csv
```

These accumulate over time. If you render multiple times on the same day, the file is overwritten (same date = same filename).

---

## File Reference

| File | Purpose |
|------|---------|
| `index.qmd` | Main dashboard — batch KPI tabs, P&L, data table |
| `trends.qmd` | SPC analysis report with control charts |
| `trends_jc_farms.qmd` | Operational trends (mortality, FCR, efficiency) |
| `R-scripts/00-functions.R` | Shared functions used by the dashboard |
| `data/` | Transaction CSVs, Excel models, tracking files |
| `data/archive/` | Timestamped backups from each KoboToolbox pull |
| `sandbox/` | Experimental scripts for prototyping |
| `style.css` | Custom styling for the trend reports |

---

## Functions Quick Reference

| Function | What it does | Used by |
|----------|-------------|---------|
| `jc_dashboard_kpis(data, batch_num, num_birds)` | Returns 5 formatted KPI strings | Setup chunk |
| `jc_pnl_by_batch(data, batch_num)` | Builds P&L dataframe for one batch | P&L section |
| `jc_get_bird_count(data, batch_num, status)` | Gets bird count (sold or purchased) | Setup chunk |
| `jc_render_batch_kpis(kpis, status)` | Renders top-row value boxes | Batch tabs |
| `jc_render_per_bird_kpis(kpis, status)` | Renders per-bird value boxes | Batch tabs |
