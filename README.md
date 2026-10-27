# Who Drives the Revenue? RFM Customer Segmentation (Online Retail)

RFM segmentation of an online retailer's customers using Python, SQL and Tableau. About 27% of customers (the Champions) generate 71% of revenue, and a large group of former regulars has gone quiet and is worth winning back.

**[View the live dashboard on Tableau Public](https://public.tableau.com/views/RFMCustomerSegmentation-OnlineRetail_17931012000740/RFMDashboard?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)**


## 1. Problem

Which customers bring in the money, which ones are drifting away, and where should a retailer spend its marketing budget? This project scores every customer on **Recency** (how recently they bought), **Frequency** (how often) and **Monetary value** (how much), then groups them into segments with a recommended action for each.

## 2. Data

- **Source:** [UCI Machine Learning Repository, Online Retail II](https://archive.ics.uci.edu/dataset/502/online+retail+ii) (Chen, Daqing, 2019), licensed CC BY 4.0.
- **What it is:** real transactions from a UK-based online gift retailer, 1 December 2009 to 9 December 2011. 1,067,371 rows and 8 columns (Invoice, StockCode, Description, Quantity, InvoiceDate, Price, Customer ID, Country). Prices are in pounds sterling.
- The raw file is not included in this repo. Download it from the link above.

## 3. Approach

1. **Clean (Python, pandas):** removed duplicates, rows without a Customer ID, cancellations, zero or negative quantities and prices, and non-product codes (postage, manual entries, vouchers). Details below.
2. **Score (SQL on SQLite):** a CTE pipeline with window functions. For each customer it calculates days since last purchase, number of distinct invoices and total spend. `CUME_DIST()` converts each into a percentile score from 1 to 5, where 5 is best. Recency is scored in reverse so that recent buyers score high. Percentiles were used because many customers have only one order, so equal-sized groups would split identical customers arbitrarily.
3. **Segment:** six segments from the Recency and Frequency scores (rules in the table below).
4. **Visualise (Tableau Public):** KPI cards, a recency-by-frequency heatmap, a treemap of revenue share, average spend per segment and a customer-level scatter plot.
5. **Validate:** segment customer counts and revenue totals were cross-checked against an Excel PivotTable and match.

### Cleaning log

| Check (counted on the raw data, so groups overlap) | Rows |
|---|---|
| Raw rows | 1,067,371 |
| Exact duplicate rows | 34,335 |
| Missing Customer ID | 243,007 |
| Cancellations (Invoice starts with "C") | 19,494 |
| Quantity of zero or less | 22,950 |
| Price of zero or less | 6,207 |
| Non-product codes removed (after the steps above) | 2,848 |
| Extreme bulk orders removed (see Limitations) | 2 invoices |
| **Final dataset** | **about 776.6K rows, 5,852 customers** |

About 27% of raw rows were removed in total. The full log is in [`data/cleaning_log.csv`](data/cleaning_log.csv).

## 4. Insights

Total revenue analysed: **£16.8M** across **5,852 customers**.

| Segment | Rule (R = recency score, F = frequency score) | Customers | % of customers | Avg days since last order | Avg orders | Avg spend | Revenue | % of revenue |
|---|---|---|---|---|---|---|---|---|
| Champions | R 4-5 and F 4-5 | 1,583 | 27.1% | 21 | 14.7 | £7,560 | £11,968,093 | 71.1% |
| Loyal Customers | R 3+ and F 3+ (not Champions) | 1,444 | 24.7% | 78 | 4.7 | £1,756 | £2,535,761 | 15.1% |
| At Risk | R 1-2 and F 3-5 | 1,206 | 20.6% | 380 | 4.1 | £1,462 | £1,763,058 | 10.5% |
| Lost | R 1-2 and F 1-2 | 1,134 | 19.4% | 474 | 1.0 | £331 | £375,432 | 2.2% |
| Need Attention | R 3 and F 1-2 | 246 | 4.2% | 105 | 1.0 | £410 | £100,891 | 0.6% |
| New & Promising | R 4-5 and F 1-2 | 239 | 4.1% | 30 | 1.0 | £333 | £79,679 | 0.5% |

1. **Revenue is highly concentrated.** The 27% of customers who are Champions generate 71% of revenue.
2. **A Champion is worth about 23 times a Lost customer** (£7,560 against £331 on average).
3. **The biggest opportunity is the At Risk group.** These 1,206 customers averaged 4.1 orders and about £1.76M in past spend, but haven't ordered in around 380 days.
4. **Lost customers are low value.** They are 19% of customers but only 2% of revenue.

## 5. Recommendations

- **Champions:** protect them with early access to new products and loyalty rewards.
- **At Risk:** start with a win-back offer. This group has the largest upside.
- **Loyal Customers:** encourage bigger baskets with bundles and upsells, to move them towards Champions.
- **New & Promising:** send a second-order incentive within 30 days of the first purchase.
- **Need Attention:** one-time buyers from about 3 to 4 months ago, so a reminder email with a time-limited offer is a cheap way to stop them becoming Lost.
- **Lost:** low priority. A low-cost email at most.

## 6. Limitations

- **Cancelled orders.** Cancellation rows (invoice numbers starting with "C") were removed, but the original orders they reversed cannot be matched automatically because the invoice numbers differ. Some cancelled orders may therefore still count as spend. Two extreme cases (74,215 and 80,995 units, about £245K combined) were confirmed as cancelled within minutes by finding their cancellation rows (C541433 and C581484), and both orders were removed manually.
- **Anonymous purchases are excluded.** Rows without a Customer ID (about 23% of the raw data) cannot be assigned to a customer, so the analysis covers identified customers only.
- **Tied frequency scores.** Many customers have exactly one order, so they share the same score. The lowest frequency score that appears in the data is 2.
- **Product filter.** Non-product rows were removed by a rule (stock codes that do not start with five digits), which could drop a few legitimate items.
- **Judgement calls.** The segment rules and 1-5 cut-offs are standard choices and not the only valid ones. Results describe 2009-2011 behaviour at a single retailer that appears to serve many wholesale buyers, so they should not be read as a forecast.

## Project structure

```
rfm-customer-segmentation-online-retail/
├── README.md
├── rfm_analysis.ipynb
├── data/
    ├── rfm_customers.csv
    ├── segment_summary.csv
    └── cleaning_log.csv
```

## Tools

Python (pandas), SQL (SQLite: CTEs, window functions), Google Colab, Tableau Public, Excel (validation PivotTable).

*Data: Chen, Daqing (2019). Online Retail II. UCI Machine Learning Repository. CC BY 4.0.*
