# Customer Behavior Dashboard: Documentation

## Contents

1. [Overview](#1-overview)
2. [Data Source & Preparation](#2-data-source--preparation)
3. [Data Dictionary](#3-data-dictionary)
4. [Data Model & DAX](#4-data-model--dax)
5. [Dashboard Guide](#5-dashboard-guide)
6. [Insights & Recommendations](#6-insights--recommendations)
7. [Known Issues & Roadmap](#7-known-issues--roadmap)

---

## 1. Overview

| Item | Detail |
|---|---|
| Report | `customer_behavior_dashboard.pbix`, 1 page (1500 × 820) |
| Data | 3,900 customers × 19 columns, one row per customer |
| Business question | Who are our customers, what do they buy, and how do segments differ in spend and satisfaction? |
| Audience | Retail marketing and merchandising teams |
| Coverage | 50 US states, 4 categories, 25 items |

The dataset has no dates, so the analysis is a **snapshot of customer segments**, not a time trend.

## 2. Data Source & Preparation

### Source

| Item | Detail |
|---|---|
| Database | PostgreSQL |
| Server | `localhost:5432` |
| Database name | `customer_behavior` |
| Table | `public.customer` |
| Load mode | Import (data is stored inside the `.pbix`) |

### Power Query

The query is a direct table load with **no transformation steps**:

```m
let
    Source = PostgreSQL.Database("localhost:5432", "customer_behavior"),
    public_customer = Source{[Schema="public", Item="customer"]}[Data]
in
    public_customer
```

All cleaning and derived columns (`age_group`, `purchase_frequency_days`) were therefore done **before** Power BI, most likely in SQL or Python. If you publish this project, include that script in the repository to make the pipeline reproducible.

### Data quality checks

| Check | Result |
|---|---|
| Missing values | None |
| Duplicate customer IDs | None (IDs 1–3900) |
| Age vs age_group | Consistent (bands listed below) |
| Frequency vs days mapping | Consistent, but contains duplicate labels (see Known Issues) |

### Re-pointing the data source

1. **Home > Transform data > Data source settings**
2. Select the PostgreSQL source > **Change Source...**
3. Enter your server and database name, then enter credentials when prompted.

To demo without a database, export `public.customer` to CSV and replace the source step with `Csv.Document(...)`.

## 3. Data Dictionary

Table: **`public customer`**. 3,900 rows × 19 columns.

### Customer

| Column | Type | Values / Range | Description |
|---|---|---|---|
| customer_id | Whole number | 1–3900, unique | Customer identifier |
| age | Whole number | 18–70 (mean 44.1) | Age in years |
| age_group | Text | Young Adult (18–31), Adult (32–44), Middle-aged (45–57), Senior (58–70) | Age band (derived upstream) |
| gender | Text | Male (2,652), Female (1,248) | Gender |
| location | Text | 50 US states | State of the customer |
| subscription_status | Text | Yes (1,053), No (2,847) | Subscribed to the store |
| previous_purchases | Whole number | 1–50 (mean 25.4) | Number of earlier purchases |

### Purchase

| Column | Type | Values / Range | Description |
|---|---|---|---|
| item_purchased | Text | 25 items (Blouse, Shirt, Dress, Pants, Jewelry, Sneakers, ...) | Item bought |
| category | Text | Clothing, Accessories, Footwear, Outerwear | Product category |
| purchase_amount | Whole number | 20–100 (mean 59.76) | Amount of the purchase (currency not stated) |
| size | Text | S, M, L, XL | Item size |
| color | Text | 25 colours | Item colour |
| season | Text | Spring, Summer, Fall, Winter | Season of purchase |
| review_rating | Decimal | 2.5–5.0 (mean 3.75) | Customer rating |
| discount_applied | Text | Yes (1,677), No (2,223) | Whether a discount was used |
| shipping_type | Text | Free Shipping, Standard, Store Pickup, Next Day Air, Express, 2-Day Shipping | Delivery option |
| payment_method | Text | PayPal, Credit Card, Cash, Debit Card, Venmo, Bank Transfer | Payment type |

### Behaviour

| Column | Type | Values | Description |
|---|---|---|---|
| frequency_of_purchases | Text | Weekly, Bi-Weekly, Fortnightly, Monthly, Quarterly, Every 3 Months, Annually | How often the customer buys |
| purchase_frequency_days | Whole number | 7, 14, 30, 90, 365 | Frequency converted to days (derived upstream) |

## 4. Data Model & DAX

A **single flat table** with no relationships, which suits a one-row-per-customer dataset.

```
┌──────────────────────────┐
│     public customer      │
│  3,900 rows · 19 columns │
│  3 measures              │
└──────────────────────────┘
```

### Measures

```dax
Number of Customers     = COUNT('public customer'[customer_id])
Average Purchase Amount = AVERAGE('public customer'[purchase_amount])
Average Review Rating   = AVERAGE('public customer'[review_rating])
```

| Measure | Meaning | Unfiltered value |
|---|---|---|
| Number of Customers | Customers in the current filter context (one row per customer) | 3,900 |
| Average Purchase Amount | Mean purchase value | 59.76 |
| Average Review Rating | Mean rating | 3.75 |

The charts that show **revenue** use Power BI's implicit `Sum of purchase_amount` rather than a named measure. Creating a `Total Revenue` measure is recommended (see Roadmap).

## 5. Dashboard Guide

### Layout

```
┌──────────────────────────────────────────────────────────────┐
│                  Customer Behavior Dashboard                 │
├────────────┬─────────────────────────────────────────────────┤
│ Slicers    │   KPI cards: Customers · Avg Purchase · Rating  │
│ Subscript. ├───────────────┬───────────────┬─────────────────┤
│ Gender     │ Subscription  │ Revenue by    │ Customers by    │
│ Category   │ donut         │ Category      │ Category        │
│ Shipping   ├───────────────┴──┬────────────┴──┬──────────────┤
│            │ Revenue by Age   │               │ Customers by │
│            │ Group (bar)      │               │ Age Group    │
└────────────┴──────────────────┴───────────────┴──────────────┘
```

### KPI cards

| Card | Measure |
|---|---|
| Number of Customers | `[Number of Customers]` |
| Average Purchase Amount | `[Average Purchase Amount]` |
| Average Review Rating | `[Average Review Rating]` |

### Visuals

| Visual | Type | Fields | Question answered |
|---|---|---|---|
| % of Customers by Subscription Status | Donut | subscription_status, Sum of customer_id | What share of customers subscribe? *(see Known Issues)* |
| Revenue by Category | Clustered column | category, Sum of purchase_amount | Which categories earn the most? |
| Sales by Category | Clustered column | category, Sum of customer_id | Meant to show customers per category *(see Known Issues)* |
| Revenue by Age Group | Clustered bar | age_group, Sum of purchase_amount | Which age groups spend most in total? |
| Sales by Age Group | Clustered bar | age_group, Sum of customer_id | Meant to show customers per age group *(see Known Issues)* |

### Slicers

| Slicer | Type | Field |
|---|---|---|
| Subscription Status | Tile slicer | subscription_status |
| Gender | Tile slicer | gender |
| Category | Tile slicer | category |
| Shipping Type | List slicer | shipping_type |

### How to use it

1. Read the KPI cards for the overall picture.
2. Use the slicers to isolate a segment, for example Female + Subscribed, and watch every card and chart update.
3. Click a chart element (a category column, an age bar) to cross-filter the rest of the page.
4. Compare the revenue chart with the customer count chart to see whether a segment is big because of **volume** or **spend**.

## 6. Insights & Recommendations

Totals: **3,900 customers, 233,081 revenue, average purchase 59.76, average rating 3.75.**

> Percentages are calculated from the data in the model. The data looks synthetic (very even distributions), so treat patterns as illustrative, and they show association only.

### Category

| Category | Customers | Revenue | Revenue share | Avg purchase |
|---|---|---|---|---|
| Clothing | 1,737 | 104,264 | 44.7% | 60.03 |
| Accessories | 1,240 | 74,200 | 31.8% | 59.84 |
| Footwear | 599 | 36,093 | 15.5% | 60.26 |
| Outerwear | 324 | 18,524 | 7.9% | 57.17 |

Outerwear is both the smallest and lowest-spending category.

### Demographics

| Segment | Customers | Revenue share | Avg purchase | Avg rating |
|---|---|---|---|---|
| Male | 68.0% | 67.7% | 59.54 | 3.75 |
| Female | 32.0% | 32.3% | 60.25 | 3.74 |
| Young Adult (18–31) | 26.4% | 26.7% | 60.45 | 3.80 |
| Adult (32–44) | 24.2% | 24.0% | 59.42 | 3.73 |
| Middle-aged (45–57) | 25.3% | 25.4% | 60.04 | 3.71 |
| Senior (58–70) | 24.2% | 23.9% | 59.07 | 3.75 |

### Subscription, discounts and shipping

- **Subscribers:** 27.0% of customers; average purchase 59.49 vs 59.87 for non-subscribers; same 3.75 rating. Subscribers have slightly more previous purchases (26.1 vs 25.1).
- **Discounts:** applied on 43.0% of purchases; average purchase 59.28 with a discount vs 60.13 without.
- **Shipping:** six options each hold 16–17% of customers. Standard has the highest average rating (3.82); Store Pickup the lowest (3.71).

### Other patterns

- **Season:** the four seasons hold 24.5–25.6% of customers each; Fall has the highest average purchase (61.56).
- **Location:** Montana (5,784), Illinois, California, Idaho and Nevada lead on revenue; Kansas (3,437), Hawaii and Florida are lowest.
- **Payment methods:** all six sit at 15.7–17.4%.

### Recommendations

1. **Invest in Clothing and Accessories**, which together deliver about 76% of revenue.
2. **Review the subscription programme.** Subscribers buy and rate the same as non-subscribers, so test whether perks actually change behaviour.
3. **Look at the female customer segment.** It is only 32% of customers but spends as much per purchase, so there is room to grow it.
4. **Investigate Outerwear**: low volume and the lowest spend per purchase suggest a pricing or range issue (it may also be seasonal).
5. **Test discounts.** Discounted purchases are not larger than full-price ones, which suggests they may not lift basket size.

## 7. Known Issues & Roadmap

### Known issues

#### 1. Donut sums `customer_id` instead of counting customers
The subscription donut uses `Sum of customer_id`. Adding up ID numbers has no business meaning and distorts the result:

| Subscription | Donut shows | Actual share of customers |
|---|---|---|
| No | 92.7% | **73.0%** |
| Yes | 7.3% | **27.0%** |

Subscribers have lower ID numbers in this table, so the donut shows them at about a quarter of their true share. **Fix:** replace the value with `[Number of Customers]`.

#### 2. "Sales by ..." charts also sum `customer_id`
"Sales by Category" and "Sales by Age Group" use `Sum of customer_id` and are titled *Sales*. They are neither sales nor customer counts. The distortion is small here (category shares are within about 0.2 points) but only by coincidence. **Fix:** use `[Number of Customers]` and rename to "Customers by Category" and "Customers by Age Group".

#### 3. Revenue uses implicit sums
Revenue charts rely on an implicit `Sum of purchase_amount`. **Fix:** add `Total Revenue = SUM('public customer'[purchase_amount])` and use it everywhere. No KPI card shows revenue even though it is the headline number.

#### 4. Age group order
Age groups are text, so they sort alphabetically (Adult, Middle-aged, Senior, Young Adult). **Fix:** add a sort-order column (1–4) and use *Sort by column*.

#### 5. Duplicate frequency labels
`Fortnightly` and `Bi-Weekly` both mean 14 days; `Quarterly` and `Every 3 Months` both mean 90. Charts by `frequency_of_purchases` split one group into two. **Fix:** standardise the labels.

#### 6. Pipeline not reproducible
The source is a local PostgreSQL database and the derived columns were built outside Power BI. **Fix:** add the SQL/Python scripts to the repo, and use a parameter for the server name.

### Limitations

- One row per customer with a single purchase, so no repeat-purchase or trend analysis.
- No date field, so no seasonality over time (the `season` column is a label only).
- Currency is not specified.
- Synthetic-looking data limits real-world conclusions.

### Roadmap

- [ ] Fix issues 1–5
- [ ] Add a `Total Revenue` KPI card
- [ ] Add a second page: payment method, shipping and discount analysis
- [ ] Add a US map of revenue by state
- [ ] Add purchase frequency and previous purchases analysis (customer loyalty)
- [ ] Add tooltips showing revenue, customers and average rating together
- [ ] Publish to Power BI Service and link the live report
