# Windkiez AG — Berlin Burger Chain Sales Dashboard (Power BI)

> **Disclaimer:** Windkiez AG is a fictional company. All data in this project — revenue, costs, product mix and order volumes — is synthetic and AI-generated for demonstration purposes. It does not represent any real business, and no confidential or proprietary information is included.

An end-to-end Power BI project: a star-schema dataset generated in Python, modelled in Power BI Desktop, and turned into an interactive sales and profitability dashboard for a fictional four-branch burger chain in Berlin across calendar year 2026.

![Dashboard overview](images/overview.png)

---

## The problem

A four-branch restaurant business wants to know more than "how much did we sell." It wants to know **which branch actually makes money**, how that changes across the year, and how much margin is lost to delivery-platform commission.

That question is harder than it looks, because the underlying data lives at two different grains:

- **Sales** are recorded per day, per branch, per product, per channel
- **Costs** (rent, salaries, inventory, electricity) are only known per month, per branch

Profit has to be calculated across both without double-counting or breaking when the user filters by quarter.

---

## The data model

Star schema — **3 fact tables, 4 dimension tables**.

| Table | Grain | Rows |
|---|---|---|
| `Fact_Sales` | Date × Branch × Product × Channel | 243,097 |
| `Fact_DayPart` | Date × Branch × Channel × DayPart | 14,600 |
| `Fact_Costs` | Month × Branch × CostType | 240 |
| `Dim_Date` | One row per day, 2026 | 365 |
| `Dim_Branch` | One row per branch | 4 |
| `Dim_Product` | One row per menu item | 45 |
| `Dim_Channel` | One row per sales channel | 5 |

All relationships are **one-to-many, single direction**, from dimension to fact. `Dim_Date` is marked as the model's date table.

Why a star schema rather than one flat table: it keeps the fact tables narrow, lets a single branch or date slicer filter all three facts at once, and makes drill-through from the overview page to a branch page straightforward.

**Channels modelled:** Cash and PayPal for eat-in; Wolt, Lieferando and Uber Eats for delivery, each with its own commission rate (15%, 22% and 30% of gross order value respectively).

---

## DAX measures

```dax
Revenue = SUM(Fact_Sales[GrossRevenueEUR])
```

```dax
Commission = SUM(Fact_Sales[PlatformCommissionEUR])
```

```dax
Operating Costs = SUM(Fact_Costs[AmountEUR])
```

```dax
Profit = [Revenue] - [Commission] - [Operating Costs]
```

```dax
Profit Margin % = DIVIDE([Profit], [Revenue])
```

`Profit` is the measure that matters here — it spans two fact tables at different grains, and resolves correctly at month, quarter and year level because both facts are filtered through the shared `Dim_Date` and `Dim_Branch` dimensions.

**Known limitation:** because costs are stored monthly, profit is only meaningful at month level and above. Daily-grain visuals in this report are restricted to revenue and unit measures.

---

## Findings

**Margin varies far more than revenue.** Charlottenburg turns 46.8% of revenue into profit; Hellersdorf manages 23.5%. Rent is close to fixed while revenue is not, so the smallest branch carries proportionally the heaviest overhead.

| Branch | Monthly revenue | Monthly profit | Margin |
|---|---|---|---|
| Charlottenburg | €175,017 | €81,878 | 46.8% |
| Friedrichshain | €80,061 | €33,824 | 42.2% |
| Alexanderplatz | €64,983 | €22,697 | 34.9% |
| Hellersdorf | €35,126 | €8,259 | 23.5% |

**Delivery commission is the second-largest cost line.** €530,592 a year goes to platforms — more than rent and electricity combined. Uber Eats takes 30% of order value against Wolt's 15%, so channel mix has a direct and underappreciated margin effect.

**Demand and channel mix move in opposite directions seasonally.** Summer is the busiest period, but the eat-in share peaks in July and bottoms out in January — cold weather pushes customers to delivery exactly when total volume is falling, compounding the winter margin squeeze.

**Every branch has a different bestseller**, which argues against a single chain-wide menu strategy:

| Branch | Top burger |
|---|---|
| Charlottenburg | Bacon & Cheese Burger |
| Friedrichshain | Plant Based Burger |
| Alexanderplatz | Avocado & Cheese Burger |
| Hellersdorf | Crispy Chicken Burger |

**Full year, all branches:** revenue €4,262,237 · commission €530,592 · operating costs €1,971,743 · **profit €1,759,903** · 330,885 orders · average basket €12.88.

---

## Report pages

**Overview** — KPI cards for revenue, profit and orders; revenue by branch; revenue by quarter; monthly trend; platform share of revenue; revenue by product category.

**Branch detail** — reached by drill-through from any branch on the overview.

---

## Repository contents

```
├── data/
│   └── Windkiez_AG_2026_Dataset.xlsx    # star schema, 8 sheets
├── images/                               # dashboard screenshots
├── Windkiez_AG_Report.pbix               # Power BI Desktop file
└── README.md
```

The workbook includes a `README_Assumptions` sheet documenting every modelling assumption behind the generated data.

---

## Tools

Power BI Desktop · DAX · Power Query · Python (pandas, openpyxl) for dataset generation
