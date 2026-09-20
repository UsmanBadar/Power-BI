# Superstore Sales Performance & Forecasting

Power BI dashboard on the Kaggle Superstore dataset (9,994 order lines, 2014–2017). Built star schema data model, DAX measures with time intelligence, and a 12-month revenue forecast with confidence bands.

**Tools:** Power BI Desktop, Power Query (M), DAX

---

## Dashboard

### Sales Overview
KPI cards (revenue, profit, margin, orders, average order value), monthly revenue trend across all 48 months, revenue by category, and a state-level choropleth. Year, Region and Segment slicers.

### Profitability
Diverging bar chart of profit by sub-category, a discount-vs-profit scatter, and a product-level detail table of the worst loss-makers.

### Forecast
Current year vs prior year by month, YoY growth and target achievement cards, and a 12-month forecast with a 95% confidence band.

---

## Data model

Star schema: one fact table, four dimensions.

| Table | Grain | Rows |
|---|---|---|
| Sales (fact) | one row per order line | 9,994 |
| Date | one row per day | 1,461 |
| Product | one row per product | 1,894 |
| Customer | one row per customer | 793 |
| Geography | one row per city/state | ~600 |

**Staging query pattern.** A single load-disabled query handles the raw import and shared transformations. Every table in the model references it, so encoding, type and key changes are made once and inherit downstream.

**Date dimension** built with `CALENDAR()` covering 2014-01-01 to 2017-12-31 — continuous, no gaps, marked as the model's date table. Time intelligence functions require this; they can't run reliably against date columns on a fact table that has gaps and two competing date fields.

**Composite product key.** 32 Product IDs in the source appear against two genuinely different product names. Deduplicating on Product ID alone would have silently attributed one product's revenue to the other — the totals would still balance, so the error would never surface. Key is built as `Product ID | Product Name` instead.

---

## DAX measures

```dax
Total Sales   = SUM(Sales[Sales])
Total Profit  = SUM(Sales[Profit])
Profit Margin = DIVIDE([Total Profit], [Total Sales])
Total Orders  = DISTINCTCOUNT(Sales[Order ID])
Avg Order Value = DIVIDE([Total Sales], [Total Orders])
Avg Discount  = AVERAGE(Sales[Discount])

Sales LY = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))
YoY Growth % = DIVIDE([Total Sales] - [Sales LY], [Sales LY])

Sales Target = [Sales LY] * 1.10
Target Achievement % = DIVIDE([Total Sales], [Sales Target])
```

`DISTINCTCOUNT` on Order ID, not `COUNT` — the fact table's grain is the order line, so a three-item order occupies three rows. Line count is 9,994; order count is 5,009.

`DIVIDE()` throughout rather than `/`, so empty denominators return blank instead of erroring. 2014 has no prior year, and every unfiltered slice would otherwise break the visual.

**Target is an assumption.** The source has no budget data. Target is defined as prior year +10% and labelled as illustrative on the dashboard rather than presented as a real figure.

---

## Power Query

- Locale set to `English (United States)` at import. Source dates are US-format; read under an Australian locale, `11/8/2016` parses as 11 August instead of 8 November — silently, with no error, because both are valid days.
- File origin set to Windows-1252. Loaded as UTF-8, product names containing special characters render as replacement marks.
- Dropped Row ID, Country (single distinct value) and Postal Code.
- Descriptive columns removed from the fact table once dimensions were built — keys and measures only.

---

## Forecasting

Power BI's built-in forecast (exponential smoothing), 12 months out, 95% confidence interval, seasonality set to 12.

**Continuous axis requirement.** The forecast option only appears on a line chart with a continuous date axis. A text column like `2017-01` sorts correctly but is categorical, and a Year/Month hierarchy is categorical too — neither exposes the feature. A `Month Start` date column (first of the month, date type) solves it.

**Seasonality set explicitly.** Left on auto, detection can miss the annual cycle on a series this short. Set to 12, the projection carries the Q4 peak into 2018 rather than extending a flat trend.

**The forecast is not available to DAX.** It's computed by the visual at render time and never written to the model — there's no measure that can reference it. A "next 12 months" KPI card would have to be hard-coded or independently calculated, so the chart carries the projection on its own.

---

## Findings

- Revenue grew 20.4% in 2017 ($733K vs $609K), with a consistent Q4 peak each year.
- Overall margin is 12.5%, but three sub-categories lose money outright: Tables, Bookcases and Supplies.
- Discount is the driver. Profit falls as discount rises, and past roughly 20% most order lines are unprofitable. The loss-making furniture lines are heavily discounted, not inherently bad products.

---

## Data quality notes

Four issues in this dataset that fail silently rather than erroring:

1. **Date locale** — wrong parse produces valid-looking dates.
2. **Duplicate product keys** — 32 IDs against different products; totals still reconcile.
3. **Encoding** — UTF-8 read of a Windows-1252 file corrupts product names, and corrupted strings break key matching between tables.
4. **Type narrowing** — converting a decimal column to whole number rounds the values permanently. Converting back does not recover them.

---

## Source

[Superstore Dataset | Kaggle](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final)
