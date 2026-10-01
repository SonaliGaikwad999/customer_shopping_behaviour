# Customer Behavior Dashboard (Power BI)

A single-page, interactive Power BI dashboard that analyses **3,900 retail customers** to show who buys, what they buy and how much they spend. It looks at revenue and customer mix by product category and age group, subscription uptake, and satisfaction, with filters for subscription status, gender, category and shipping type.

![Dashboard preview](images/dashboard.png)
<!-- Add a screenshot of the dashboard at images/dashboard.png -->

---

## Key Results

| KPI | Value |
|---|---|
| Customers | **3,900** |
| Total purchase revenue | **233,081** |
| Average purchase amount | **59.76** |
| Average review rating | **3.75 / 5** |
| Subscribers | 1,053 (**27.0%**) |

> Currency is not stored in the file. The US state names in `location` suggest USD.

### Headline findings

1. **Clothing is the core category.** It brings in 44.7% of revenue (104,264), followed by Accessories (31.8%), Footwear (15.5%) and Outerwear (7.9%).
2. **Men make up 68% of customers** and 67.7% of revenue, but men and women spend about the same per purchase (59.54 vs 60.25).
3. **Spending is remarkably uniform.** Average purchase amount stays between 57 and 62 across categories, age groups, seasons, shipping types and subscription status, so segment size matters more than segment spend.
4. **Subscribers are 27% of customers** but do not spend or rate differently (59.49 vs 59.87 average purchase, 3.75 rating for both).
5. **Age groups are evenly balanced**, each holding 24–26% of customers; Young Adults (18–31) lead slightly with 26.7% of revenue.

> **Important:** the subscription donut currently shows the wrong split (about 93% / 7% instead of the true 73% / 27%) because it sums `customer_id`. Fix is in [Known Issues](DOCUMENTATION.md#7-known-issues--roadmap).

Full analysis: [Insights](DOCUMENTATION.md#6-insights--recommendations)

---

## Dashboard Features

- **3 KPI cards:** Number of Customers, Average Purchase Amount, Average Review Rating
- **5 charts:** subscription donut, revenue and customer count by category, revenue and customer count by age group
- **4 slicers:** Subscription Status, Gender, Category, Shipping Type
- Cross-filtering between all visuals

See the [Dashboard Guide](DOCUMENTATION.md#5-dashboard-guide).

## Tech Stack

| Layer | Tool |
|---|---|
| Database | PostgreSQL (`customer_behavior` database, `public.customer` table) |
| Connection | Power BI PostgreSQL connector (Import mode) |
| Calculations | DAX (3 measures) |
| Visualisation | Power BI Desktop |

## Repository Structure

```
Customer-Behavior-Dashboard/
├── README.md            # Project overview (this file)
├── DOCUMENTATION.md     # Full technical documentation
├── customer_behavior_dashboard.pbix
└── images/              # Screenshots
```

## Getting Started

1. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (Windows).
2. Open `customer_behavior_dashboard.pbix`. The data is already loaded in the file, so the report opens and filters without a database.
3. To **refresh** the data you need PostgreSQL running at `localhost:5432` with a `customer_behavior` database containing `public.customer`. Otherwise, repoint the source under **Home > Transform data > Data source settings**. See [Data Source & Preparation](DOCUMENTATION.md#2-data-source--preparation).

## Skills Demonstrated

SQL database connectivity (PostgreSQL) · Data modelling · DAX measures · KPI design · Dashboard layout and slicers · Customer segmentation · Documentation

## Known Limitations

One row per customer (a single purchase each), no date field, and a few visuals use the wrong aggregation. Details and fixes: [Known Issues & Roadmap](DOCUMENTATION.md#7-known-issues--roadmap).

## Author

**<Your Name>** — Data Analyst (transitioning from Software Test Engineering)
[LinkedIn](https://linkedin.com/in/your-profile) · [GitHub](https://github.com/your-username)
