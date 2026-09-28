# Power BI

Power BI projects — data modelling, DAX, and Power Query.

Each project folder contains the `.pbix` file, screenshots, and a README covering the model design, measures, and findings.

---

## Projects

### [Superstore Sales Performance & Forecasting](./Superstore%20Sales%20Performance%20%26%20Forecasting)

Sales report on the Kaggle Superstore dataset (2014–2017). Built Star schema with four dimensions, DAX time intelligence, and a 12-month revenue forecast with confidence bands.

Covers: staging query pattern in Power Query, composite keys where the source key isn't unique, `CALENDAR()` date dimension, `SAMEPERIODLASTYEAR` and YoY measures, and Power BI's built-in exponential smoothing forecast.


### [Financial Statement Analysis](./Financial%20Statement%20Analysis)

Financial performance report on 12 large public companies (2009–2022), published to Microsoft Fabric as a semantic model, report, and app. Star schema with company and year dimensions, profitability, growth, cash-flow and leverage measures, and a revenue-to-net-income waterfall.

Covers: data cleaning of mislabelled sectors and mixed units, ratio-of-sums DAX measures, handling negative equity and loss years, a disconnected table for the waterfall, a QA page reconciling measures to the source, and publishing via a Fabric workspace app.

---

**Usman Badar** · [LinkedIn](https://linkedin.com/in/usman-badar)
