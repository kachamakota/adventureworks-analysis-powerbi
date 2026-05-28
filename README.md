# AdventureWorks Power BI SALES ANALYSIS

**This project demonstrates how Power BI can be used to build an interactive sales analysis report from a structured business dataset.**

The main focus of the project was to create an interactive dashboard design, which allows users to analyze sales, profit, quantity sold, customer behavior, and product performance across different time periods and regions.

## PROJECT STRUCTURE

```text
powerbi-adventureworks-analysis/
├── README.md
├── Adventure_Works_Dashboard.pbix
├── dataset/
│   └── Sales Data/
└── screenshots/
```

## Project Overview

This project is an interactive Power BI sales analysis dashboard built using the AdventureWorks dataset.

The goal of the project was to analyze sales performance, customer behavior, and product performance using Power BI data modeling, DAX measures, calculated columns, time intelligence functions, and interactive report visuals.

The report was created fully in Power BI. The project focuses on transforming raw business data into a structured analytical model and building dashboards that allow users to explore key business metrics across years, countries, customer segments, and product categories.

## Dataset

The project uses the AdventureWorks dataset, a sample business dataset commonly used for analytics and business intelligence practice. The dataset contains sales-related data and supporting dimension tables, including information about customers, products, geography, dates, and sales transactions.

In this project, the data was modeled into a fact-and-dimension structure, with `Fact_Sales` used as the main transactional table and several dimension tables used for filtering, segmentation, and reporting.

## Tools Used

- Power Query 
- DAX
  - SWITCH()
  - CALCULATE()
  - DIVIDE()
  - ALL()
  - ALLEXCEPT()
  - RELATED() and RELATEDTABLE()
  - FILTER()
  - DAX variables `VAR` / `RETURN`
  - various date operations such as DATEADD(), EDATE(), DATESYTD()
- Data modeling in Power BI
- PowerBI Visuals
  - Bar charts
  - Line charts
  - Waterfall charts
  - Scatter chart
  - Treemap chart
  - Cards
  - Tables
  - Average lines
  - Slicers

## Data Preparation and Modeling

Before creating the report pages and measures, the dataset was prepared and modeled in Power BI.

The preparation process included:

- identifying key columns,
- creating relationships between fact and dimension tables,
- building a star-schema-style model,
- preparing customer segmentation fields,
- creating a dedicated calendar table,
- creating a separate measures table,
- creating a disconnected slicer table for dynamic KPI switching.

Screenshots of the data model and report pages are available in the `screenshots` folder.

## Data Model

The report is based on a fact-and-dimension model.

The main sales table is connected to dimension tables such as:

- customers,
- products,
- calendar/date table,
- geography-related dimensions,
- other supporting lookup tables.

This structure allows the report visuals to respond properly to filters and slicers, such as year, country, customer segment, product category, and selected KPI.

### Calendar Table

A dedicated `Dim_Calendar` table was created to ensure a continuous date range for time intelligence calculations.

This was important because many DAX calculations in the report rely on time-based comparisons, including:

- YTD values,
- PYTD values,
- Year-over-Year comparisons,
- monthly trend analysis,
- selected-year filtering.

Using a separate calendar table also made the report more reliable than relying only on dates available directly in the sales table.

Example structure of the calendar table:

```DAX
Dim_Calendar =
CALENDAR(
    MIN(Fact_Sales[OrderDate]),
    MAX(Fact_Sales[OrderDate])
)
```
## Calculated Columns

Several calculated columns were created to support customer segmentation and behavioral analysis.

Income Band

The IncomeBand column groups customers into income segments. This was later used in the Customer Dashboard to analyze customer distribution and performance by income level.

``` DAX
IncomeBand =
SWITCH(
    TRUE(),
    Dim_Customers[AnnualIncome] < 50000, "1) Low",
    Dim_Customers[AnnualIncome] < 90000, "2) Mid",
    Dim_Customers[AnnualIncome] < 140000, "3) High",
    "4) Very High"
)
```
Parent / Non-Parent Flag

The IsParent column identifies whether a customer has children. This was used as another segmentation dimension in the customer analysis.

``` DAX
IsParent =
IF(
    Dim_Customers[TotalChildren] > 0,
    "Y",
    "N"
)
```

One-Time Customer Flag

The One_Time_Customer_Flag column was created to identify customers with only one lifetime order.

``` DAX
One_Time_Customer_Flag =
VAR order_count =
    CALCULATE(
        DISTINCTCOUNT(Fact_Sales[OrderNumber]),
        RELATEDTABLE(Fact_Sales)
    )
RETURN
    IF(order_count = 1, 1, 0)
```

This flag was used to support customer behavior analysis, especially when comparing one-time and returning customers.

Report Pages

The Power BI report contains three main dashboard pages:

General Dashboard
Customer Dashboard
Product Dashboard

## Measures Table

A separate measures table was created to keep DAX measures organized and easier to manage.

This helped separate business calculations from raw data tables and made the report structure cleaner, especially as the number of measures increased.

The project includes measures related to:

- total sales,
- total profit,
- total quantity sold,
- YTD values,
- PYTD values,
- differences between current and previous periods,
- percentage differences,
- average revenue per customer,
- customer counts,
- new and returning customer calculations,
- product performance metrics.

Using a dedicated measures table made the report easier to maintain and helped keep all business logic in one clearly organized place.

## Dynamic KPI Switching

One of the main features of the report is a <ins>dynamic KPI switch</ins> that allows the user to switch between different business metrics within the same visuals.

**The available KPI options are:**

- Sales / Revenue (USD)
- Quantity Sold (units)
- Profit (USD)

To support this functionality, a disconnected table called slc_values was created. This table contains the metric names used in the KPI slicer.

The selected value from this slicer is then used inside DAX measures <ins>to dynamically return the metric</ins>.

Example:

```DAX
S_YTD =
VAR selected_value =
    SELECTEDVALUE(slc_values[Values])
RETURN
    SWITCH(
        selected_value,
        "Sales", [YTD_Sales],
        "Quantity", [YTD_Quantity],
        "Profit", [YTD_Profit],
        BLANK()
    )
```

The underlying YTD measures were created separately:

```DAX
YTD_Sales =
CALCULATE(
    [Total_Sales],
    DATESYTD(Dim_Calendar[Date])
)
```

```DAX
YTD_Quantity =
CALCULATE(
    [Total_Quantity],
    DATESYTD(Dim_Calendar[Date])
)
```

```DAX
YTD_Profit =
CALCULATE(
    [Total_Profit],
    DATESYTD(Dim_Calendar[Date])
)
```

This approach allowed the same visuals to <ins>display different metrics</ins> depending on the user’s selection. 
It reduced the need to create separate visuals for each KPI and made the dashboard more interactive and also compact.


## Report Pages

The Power BI report contains three main dashboard pages:

- **General Dashboard**
- **Customer Dashboard**
- **Product Dashboard**

### Dashboard page 1 - General Dashboard

The General Dashboard provides a high-level overview of business performance.

It includes interactive KPI analysis for:
  - Sales / Revenue
  - Quantity Sold
  - Profit

The dashboard is controlled by a KPI slicer and a year slicer, allowing users to analyze selected metrics across 2020, 2021, and 2022.

#### Main Metrics

The dashboard includes cards and visuals showing:
   - selected KPI YTD
   - selected KPI PYTD
  - difference between YTD and PYTD
  - percentage difference between YTD and PYTD
  - difference between current year and previous year
  - percentage difference between current year and previous year

These calculations react dynamically to the selected year and, s described previously, the selected KPI.

#### Visuals

The General Dashboard includes:
  - KPI cards,
  - YTD vs PYTD combo chart,
  - waterfall chart showing month-by-month changes with drill-down functionality by country, which allows for more in depth analysis
  - monthly trend visuals

The YTD vs PYTD visual helps compare the selected year’s performance against the equivalent period from the previous year.


### Dashboard page 2 - Customer Dashboard

The Customer Dashboard focuses on customer-level analysis and customer segmentation.

It includes metrics and visuals related to:
- total customers
- average revenue per customer
- top 5 customers by revenue
- customers by country
- customers by income band
- parent vs non-parent customer segmentation,
- new vs returning customers

**Customer Segmentation**

Customers were analyzed using several segmentation fields, including:

- income level
- parent/non-parent status
- country

This makes it possible to compare customer groups and identify differences in sales contribution, customer count, and customer behavior.

**Top Customers**

The dashboard includes a Top 5 Customers by Revenue visual.

This visual reacts to other report filters and segmentations, including country and income level, allowing a more detailed analysis of high-value customers within selected segments.

**New Customers Calculation**

A <ins>first purchase date</ins> measure was created to identify when each customer made their first purchase.

```DAX
FPD =
CALCULATE(
    MIN(Fact_Sales[OrderDate]),
    ALLEXCEPT(Fact_Sales, Fact_Sales[CustomerKey])
)
```
This measure was then used to calculate <ins>new customers</ins> in the selected period.

```DAX
New_Customers =
CALCULATE(
    DISTINCTCOUNT(Fact_Sales[CustomerKey]),
    FILTER(
        VALUES(Fact_Sales[CustomerKey]),
        [FPD] IN VALUES(Dim_Calendar[Date])
    )
)
```
This allowed the report to compare new and returning customer behavior over time.

### Dashboard page 3 - Product Dashboard

The Product Dashboard focuses on product performance, product-level profitability and return rates.

It includes:

- top 10 products by revenue
- scatter plot analysis of top 25 high-revenue products, showing return rates % and profit margin %
- bar chart with monthly quantity sold by product category, and a line chart with previous year comparison
- average order value analysis
- KPI cards for selected product insights such as return rates or highest profit margin category

This page also includes a year slicer, allowing the user to analyze product performance overall, or for a selected year only.

The purpose of this page was to identify which products and product groups generated revenue, while also checking whether high-performing products had potential issues with returns or profit margin.

The scatter plot was used to analyze whether high-revenue products were also profitable and whether they had relatively high or low return rates. This made it possible to look beyond revenue alone and compare products from both profitability and product quality/customer satisfaction perspectives.

  


