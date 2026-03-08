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

Three changes in `index.qmd`:

**1. Update `batch_config`** (setup chunk, ~line 99):

```r
# Change "active" to "selling" for Batch 17
batch_config <- tibble(
  batch_num = c("15", "16", "17", "18", "19"),
  status    = c("closed", "closed", "selling", "active", "active")
                                    # ^^^^^^^^ was "active"
)
```

**2. Update the tab heading** (~line 142):

```
# Batch 17 (Selling)
```

**3. Update the render calls** to pass `status = "selling"`:

```r
jc_render_batch_kpis(all_kpis[["kpis17"]], status = "selling")
jc_render_per_bird_kpis(all_kpis[["kpis17"]], status = "selling")
```

This shows all 5 KPIs (Revenue, Cost, Net Profit, Cost/Bird, Profit/Bird) and uses birds purchased for per-bird calculations (since not all birds are sold yet).

---

### When a batch finishes selling (e.g. Batch 17)

Three changes in `index.qmd`:

**1. Update `batch_config`** — change `"selling"` to `"closed"`.

**2. Update the tab heading** to `# Batch 17 (Closed)`.

**3. Remove `status` from the render calls** so they default to `"closed"`:

```r
jc_render_batch_kpis(all_kpis[["kpis17"]])
jc_render_per_bird_kpis(all_kpis[["kpis17"]])
```

This switches per-bird calculations to use birds sold instead of birds purchased.

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

| Metric | Active | Selling | Closed |
|--------|--------|---------|--------|
| Cost of Production | Shown | Shown | Shown |
| Cost per Bird | Shown | Shown | Shown |
| Revenue | Hidden | Shown | Shown |
| Net Profit | Hidden | Shown | Shown |
| Profit per Bird | Hidden | Shown | Shown |

- **Active** — birds are being raised, no sales yet. Uses birds purchased for per-bird calculations.
- **Selling** — sales have started but not finished. Uses birds purchased for per-bird calculations (more accurate while selling is in progress).
- **Closed** — all birds sold. Uses birds sold for per-bird calculations.

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
