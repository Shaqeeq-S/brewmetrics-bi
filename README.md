
# BrewMetrics BI Dashboard

## 1. Project Overview

BrewMetrics Coffee Co. is a Business Intelligence mini-project developed using Microsoft Power BI, DAX, GitHub, GitHub Copilot, and VS Code.

The objective is to analyze coffee sales data and create an interactive dashboard that provides insights into sales performance across cities, store formats, product categories, products, and time periods.

The project follows this workflow:

**Raw Data → Data Modeling → DAX Analysis → Dashboard → Business Insights**

---

## 2. Project Objectives

- Analyze coffee sales performance using Power BI.
- Transform the flat sales dataset into a star schema.
- Create fact and dimension tables.
- Establish relationships between tables.
- Develop DAX measures for business analysis.
- Calculate month-over-month sales growth.
- Calculate running total sales.
- Rank products based on sales.
- Calculate average selling price.
- Analyze monthly and seasonal sales patterns.
- Analyze Cold Brew seasonal performance.
- Compare sales across cities and store formats.
- Implement slicers and drill-down functionality.
- Use GitHub for version control.
- Use GitHub Copilot to assist with DAX development.
- Document Copilot suggestions and corrections.
- Create a professional 16:9 Power BI dashboard.

---

## 3. Dataset

The project uses:

`brewmetrics_sales.csv`

The dataset contains approximately 15,500 transactions covering April to June.

### Dataset Columns

| Column           | Description                  |
| ---------------- | ---------------------------- |
| `date`         | Transaction date             |
| `city`         | City where the sale occurred |
| `store_format` | Store format                 |
| `category`     | Product category             |
| `item`         | Product/item name            |
| `quantity`     | Quantity sold                |
| `unit_price`   | Price per unit               |
| `sales_amount` | Total sales amount           |

---

## 4. Business Context

BrewMetrics Coffee Co. wants to understand its sales performance and identify important patterns.

The dashboard focuses on:

- Monthly sales performance
- Product category performance
- City-level performance
- Store-format performance
- Product-level sales
- Seasonal product behavior
- Month-over-month growth
- Cumulative sales
- Product rankings

The dataset contains a Cold Brew sales spike during April and May and shows strong performance from Bengaluru.

---

# 5. Technologies and Tools

- Microsoft Power BI Desktop
- DAX
- Git
- GitHub
- GitHub Copilot
- VS Code
- CSV

---

# 6. Data Model

The original CSV file is a flat transactional dataset. It was organized into a star schema.

## Fact Table

### Fact_Sales

Contains transactional sales information:

- Date
- City
- Store Format
- Item
- Quantity
- Unit Price
- Sales Amount

## Dimension Tables

### Dim_Date

Used for time-based analysis.

Fields:

- Date
- Day Name
- Day Number
- Month
- Month Number

### Dim_City

Used for city and store-format analysis.

Fields:

- City
- Store Format

### Dim_Product

Used for product and category analysis.

Fields:

- Category
- Item

## Star Schema

```text
                    Dim_Date
                       |
                       |
Dim_City ------ Fact_Sales ------ Dim_Product
                       |
                       |
                 DAX Measures
```

---

# 7. DAX Measures

## 7.1 Total Sales

```DAX
Total Sales =
SUM(Fact_Sales[sales_amount])
```

## 7.2 Total Quantity

```DAX
Total Quantity =
SUM(Fact_Sales[quantity])
```

## 7.3 Total Transactions

```DAX
Total Transactions =
COUNTROWS(Fact_Sales)
```

## 7.4 MoM Growth %

The dataset covers April to June, so month-over-month growth is used instead of year-over-year growth.

```DAX
MoM Growth % =
VAR CurrentSales =
    [Total Sales]
VAR PreviousMonthSales =
    CALCULATE(
        [Total Sales],
        DATEADD(Dim_Date[Date], -1, MONTH)
    )
RETURN
    DIVIDE(
        CurrentSales - PreviousMonthSales,
        PreviousMonthSales,
        0
    )
```

## 7.5 Running Total Sales

```DAX
Running Total Sales =
CALCULATE(
    [Total Sales],
    FILTER(
        ALLSELECTED(Dim_Date[Date]),
        Dim_Date[Date] <= MAX(Dim_Date[Date])
    )
)
```

## 7.6 Item Sales Rank

```DAX
Item Sales Rank =
RANKX(
    ALLSELECTED(Dim_Product[item]),
    [Total Sales],
    ,
    DESC,
    DENSE
)
```

## 7.7 Average Selling Price

```DAX
Average Selling Price =
DIVIDE(
    [Total Sales],
    [Total Quantity],
    0
)
```

This calculates a quantity-weighted average selling price.

## 7.8 Distinct Items

```DAX
Distinct Items =
DISTINCTCOUNT(Fact_Sales[item])
```

## 7.9 Day Name

Create this as a **New Column**, not a measure.

```DAX
Day Name =
FORMAT(Dim_Date[Date], "dddd")
```

## 7.10 Day Number

```DAX
Day Number =
WEEKDAY(Dim_Date[Date], 2)
```

Use `Day Number` to sort `Day Name` from Monday to Sunday.

---

# 8. KPI Comparison Measures

## Previous Sales

```DAX
Previous Sales =
CALCULATE(
    [Total Sales],
    DATEADD(Dim_Date[Date], -1, MONTH)
)
```

## Previous Quantity

```DAX
Previous Quantity =
CALCULATE(
    [Total Quantity],
    DATEADD(Dim_Date[Date], -1, MONTH)
)
```

## Previous Transactions

```DAX
Previous Transactions =
CALCULATE(
    [Total Transactions],
    DATEADD(Dim_Date[Date], -1, MONTH)
)
```

## Previous Average Price

```DAX
Previous Average Price =
CALCULATE(
    [Average Selling Price],
    DATEADD(Dim_Date[Date], -1, MONTH)
)
```

## Previous Distinct Items

```DAX
Previous Distinct Items =
CALCULATE(
    [Distinct Items],
    DATEADD(Dim_Date[Date], -1, MONTH)
)
```

---

# 9. KPI Percentage Change Measures

## Sales Change %

```DAX
Sales Change % =
DIVIDE(
    [Total Sales] - [Previous Sales],
    [Previous Sales],
    0
)
```

## Quantity Change %

```DAX
Quantity Change % =
DIVIDE(
    [Total Quantity] - [Previous Quantity],
    [Previous Quantity],
    0
)
```

## Transactions Change %

```DAX
Transactions Change % =
DIVIDE(
    [Total Transactions] - [Previous Transactions],
    [Previous Transactions],
    0
)
```

## Average Price Change %

```DAX
Average Price Change % =
DIVIDE(
    [Average Selling Price] - [Previous Average Price],
    [Previous Average Price],
    0
)
```

## Items Change %

```DAX
Items Change % =
DIVIDE(
    [Distinct Items] - [Previous Distinct Items],
    [Previous Distinct Items],
    0
)
```

---

# 10. KPI Comparison Labels

## Sales Comparison

```DAX
Sales Comparison =
IF(
    [Sales Change %] >= 0,
    "▲ " & FORMAT([Sales Change %], "0.0%") &
    "    vs Previous Period",
    "▼ " & FORMAT(ABS([Sales Change %]), "0.0%") &
    "    vs Previous Period"
)
```

## Quantity Comparison

```DAX
Quantity Comparison =
IF(
    [Quantity Change %] >= 0,
    "▲ " & FORMAT([Quantity Change %], "0.0%") &
    "    vs Previous Period",
    "▼ " & FORMAT(ABS([Quantity Change %]), "0.0%") &
    "    vs Previous Period"
)
```

---

# 11. Growth Color

```DAX
Growth Color =
IF(
    [MoM Growth %] >= 0,
    "#278A55",
    "#C94C4C"
)
```

Positive growth is green and negative growth is red.

---

# 12. Category Sales Percentage

```DAX
Category Sales % =
DIVIDE(
    [Total Sales],
    CALCULATE(
        [Total Sales],
        ALLSELECTED(Dim_Product[category])
    ),
    0
)
```

Format this measure as **Percentage**.

For the city/category matrix:

- Rows: `Dim_City[city]`
- Columns: `Dim_Product[category]`
- Values: `Category Sales %`

---

# 13. Dashboard

The dashboard is designed as a 16:9 Power BI report with a warm coffee-themed visual style.

It contains:

1. Total Sales KPI
2. Total Quantity KPI
3. Total Transactions KPI
4. Average Selling Price KPI
5. Distinct Items KPI
6. Monthly Sales Trend
7. Monthly Sales by Category
8. Running Total Sales
9. Top Items by Sales
10. Sales by Category
11. Sales by City and Store Format
12. Sales by Day of Week
13. Category Sales Percentage by City
14. Interactive slicers
15. Drill-down hierarchy

---

# 14. KPI Cards

The five main KPI cards are:

- **Total Sales**
- **Total Quantity**
- **Total Transactions**
- **Average Selling Price**
- **Distinct Items**

The KPI design uses rounded rectangles, icons, large values, and comparison text.

---

# 15. Dashboard Visuals

## Monthly Sales Trend

```text
Axis:
Dim_Date[Month]

Values:
[Total Sales]
```

## Monthly Sales by Category

```text
Axis:
Dim_Date[Month]

Values:
[Total Sales]

Legend:
Dim_Product[category]
```

Recommended category colors:

```text
Coffee       → #6B3518
Bakery       → #D5B88A
Merchandise  → #7D9277
```

## Running Total Sales

```text
X-axis:
Dim_Date[Date]

Y-axis:
[Running Total Sales]
```

## Top Items by Sales

```text
Axis:
Dim_Product[item]

Values:
[Total Sales]
```

`Item Sales Rank` is used for ranking and Top-N analysis.

## Sales by Category

```text
Category:
Dim_Product[category]

Values:
[Total Sales]
```

## Sales by City and Store Format

Drill-down hierarchy:

```text
City
  ↓
Store Format
```

## Sales by Day of Week

```text
Axis:
Dim_Date[Day Name]

Values:
[Total Sales]
```

`Day Name` is sorted by `Day Number`.

## Category Sales Percentage by City

```text
Rows:
Dim_City[city]

Columns:
Dim_Product[category]

Values:
[Category Sales %]
```

Conditional formatting can be applied to the background.

---

# 16. Slicers

The dashboard contains interactive slicers for:

- City
- Store Format
- Category
- Item
- Date / Month

Example fields:

```text
Dim_City[city]
Dim_City[store_format]
Dim_Product[category]
Dim_Product[item]
Dim_Date[Date]
```

---

# 17. Drill-Down

The dashboard includes a city performance hierarchy:

```text
City
  ↓
Store Format
```

This allows the user to move from overall city performance to detailed store-format performance.

---

# 18. Dashboard Color Palette

| Purpose           | Hex Code    |
| ----------------- | ----------- |
| Dark Coffee       | `#45210F` |
| Main Coffee Brown | `#6B3518` |
| Medium Coffee     | `#8B5738` |
| Light Coffee      | `#C9A47E` |
| Page Background   | `#F7F1E8` |
| Card Background   | `#FFFDFC` |
| Bakery            | `#D5B88A` |
| Merchandise       | `#7D9277` |
| Positive Growth   | `#278A55` |
| Negative Growth   | `#C94C4C` |
| Main Text         | `#292929` |
| Secondary Text    | `#6B6B6B` |
| Gridlines         | `#D9D4CC` |
| Heatmap Minimum   | `#F5E9DC` |

## KPI Card Colors

| KPI                   | Background  | Text        |
| --------------------- | ----------- | ----------- |
| Total Sales           | `#6B3518` | `#FFFFFF` |
| Total Quantity        | `#E4D4BD` | `#45210F` |
| Total Transactions    | `#E8DCD2` | `#45210F` |
| Average Selling Price | `#E2C9B7` | `#45210F` |
| Distinct Items        | `#D9DDCF` | `#45210F` |

---

# 19. Business Questions

The dashboard answers:

### Overall Sales

- What are the total sales?
- What is the total quantity sold?
- How many transactions occurred?
- What is the average selling price?

### Time Analysis

- How do sales change month by month?
- What is the month-over-month growth?
- How does cumulative sales change over time?
- Which days of the week have stronger sales?

### Product Analysis

- Which items generate the highest sales?
- Which categories perform best?
- Which products have the highest sales rank?

### Seasonal Analysis

- Does Cold Brew show seasonal behavior?
- Does Cold Brew perform strongly during April and May?

### City Analysis

- Which city performs best?
- How do cities compare?
- Which store formats contribute to city performance?
- How does category performance differ across cities?

---

# 20. Key Insights

### Cold Brew

Cold Brew shows a seasonal increase during April and May.

### City Performance

Bengaluru consistently performs better than the other cities in the dataset.

### Product Performance

The ranking measure identifies high-performing products based on total sales.

### Category Performance

Category-level visuals allow comparison of available product categories.

---

# 21. GitHub and Version Control

Repository name:

```text
brewmetrics-bi
```

Important development stages are committed separately.

Example commit sequence:

```text
Initial project setup
Create star schema
Add Total Sales measure
Add MoM Growth measure
Add Running Total measure
Add Item Sales Rank measure
Add Average Selling Price measure
Add KPI measures
Build dashboard visuals
Add slicers and drill-down
Update documentation
```

---

# 22. GitHub Copilot Usage

GitHub Copilot was used to assist with initial DAX development.

Copilot was used for:

- Initial DAX suggestions
- Time intelligence calculations
- Running totals
- RANKX calculations
- Supporting DAX expressions

The generated DAX was reviewed before implementation.

Corrections were made when:

- Table names did not match the actual model.
- Column names were different.
- The calculation did not match the business requirement.
- Filter behavior needed to respect slicers.
- The average selling price needed to be quantity-weighted.

The detailed Copilot suggestions and corrections are documented in:

`NOTES.md`

---

# 23. Project Structure

```text
brewmetrics-bi/
│
├── brewmetrics_sales.csv
├── BrewMetrics.pbip
├── NOTES.md
├── README.md
├── REFLECTION.md
│
└── Power BI Project Files/
    ├── Report/
    ├── Model/
    └── Other project files
```

The exact PBIP internal structure may vary depending on the Power BI Project format generated by Power BI Desktop.

---

# 24. How to Run the Project

## Step 1: Clone the Repository

```bash
git clone <repository-url>
```

## Step 2: Open the Power BI Project

Open:

```text
BrewMetrics.pbip
```

using Power BI Desktop.

## Step 3: Verify the Data Model

Check:

```text
Fact_Sales
Dim_Date
Dim_City
Dim_Product
```

Verify that the relationships are correctly configured.

## Step 4: Open the Dashboard

Open the report page containing the BrewMetrics dashboard.

## Step 5: Use the Slicers

Filter by:

- City
- Store Format
- Category
- Item
- Date / Month

## Step 6: Explore Drill-Down

Use the City → Store Format hierarchy.

---

# 25. Documentation

## README.md

Contains the complete project documentation, including:

- Project overview
- Dataset
- Objectives
- Data model
- DAX measures
- KPI calculations
- Dashboard visuals
- Slicers
- Drill-down
- Color palette
- GitHub workflow
- Copilot usage
- Project structure
- Running instructions

## NOTES.md

Contains:

- Copilot initial suggestions
- DAX corrections
- Reasons for modifications
- Validation notes

## REFLECTION.md

Contains the project reflection covering:

- Power BI learning
- Data modeling
- DAX
- Dashboard design
- GitHub
- GitHub Copilot
- Challenges
- Improvements

---

# 26. Project Workflow

```text
                 BrewMetrics CSV
                       |
                       ↓
              Data Preparation
                       |
                       ↓
                Star Schema
                       |
          ┌────────────┼────────────┐
          ↓            ↓            ↓
      Dim_Date     Dim_City    Dim_Product
          \            |            /
           \           |           /
            └────── Fact_Sales ───┘
                       |
                       ↓
                 DAX Measures
                       |
                       ↓
              Power BI Visuals
                       |
                       ↓
             Interactive Dashboard
                       |
                       ↓
               Business Insights
```

---

# 27. Conclusion

The BrewMetrics BI Dashboard transforms transactional coffee sales data into an interactive Business Intelligence solution.

The project demonstrates:

- Data modeling
- Star schema design
- Power BI
- DAX
- Time-based analysis
- Product ranking
- KPI development
- Interactive filtering
- Drill-down analysis
- Dashboard design
- Git version control
- GitHub
- GitHub Copilot
- AI-assisted DAX development

The final dashboard provides a clear way to explore sales performance, product performance, seasonal behavior, and city-level differences for BrewMetrics Coffee Co.

---

# 28. Author

**Name:** Shaqeeq
**Roll Number:** 24BAD109
**Department:** AI & Data Science
**Class:** AIDS-B

---

# 29. Repository

GitHub Repository:

```text
brewmetrics-bi
```

The repository contains the Power BI Project, dataset, DAX documentation, README, reflection, and Copilot notes.
