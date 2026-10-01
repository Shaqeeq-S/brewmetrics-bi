
# DAX Development Notes

## BrewMetrics Coffee Co. — Phase 2

This file documents the development of the four required DAX measures using GitHub Copilot as an assistant. Each measure was reviewed and tested before being added to the Power BI report.

---

## 1. Month-over-Month Growth

### Measure

`MoM Growth %`

### Copilot Initial Suggestion

Copilot suggested calculating the current month's sales and comparing them with the previous month's sales using the `DATEADD()` function.

### Final DAX

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

### What I Corrected

I verified that the date column used for time-intelligence calculations comes from the `Dim_Date` table. I also used `DIVIDE()` to avoid errors when the previous month's sales are zero.

### Purpose

This measure calculates the percentage change in sales compared with the previous month. It is used in the monthly sales trend visual.

---

## 2. Running Total Sales

### Measure

`Running Total Sales`

### Copilot Initial Suggestion

Copilot suggested using `CALCULATE()` with `FILTER()` to accumulate sales from the beginning of the selected period up to the current date.

### Final DAX

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

### What I Corrected

I used `ALLSELECTED()` instead of `ALL()` so that the running total continues to respect the selections made using dashboard slicers and filters.

### Purpose

This measure calculates cumulative sales over time and is displayed in the Running Total Sales line chart.

---

## 3. RANKX-Based Item Ranking

### Measure

`Item Sales Rank`

### Copilot Initial Suggestion

Copilot suggested using the `RANKX()` function to rank items according to their total sales, with the highest-selling item receiving rank 1.

### Final DAX

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

### What I Corrected

I used `ALLSELECTED()` so that the ranking responds to the filters and slicers selected by the dashboard user.

The ranking uses:

- `DESC` — highest sales receives the highest position
- `DENSE` — tied items receive the same rank without gaps

### Purpose

This measure ranks products according to their sales performance and is used in the Top Items by Sales visual.

---

## 4. Average Selling Price

### Measure

`Average Selling Price`

### Copilot Initial Suggestion

Copilot initially suggested calculating the average selling price using the `unit_price` column.

### Final DAX

```DAX
Average Selling Price =
DIVIDE(
    [Total Sales],
    [Total Quantity],
    0
)
```

### What I Corrected

I changed the calculation to divide total sales by total quantity.

This produces a volume-weighted average selling price rather than simply averaging the individual unit prices.

### Purpose

This measure provides an overall view of the average amount generated per item sold. It is displayed as a KPI card in the dashboard.

---

# Summary of Measures

| Measure               | Main DAX Function              | Purpose                                |
| --------------------- | ------------------------------ | -------------------------------------- |
| MoM Growth %          | `DATEADD()`                  | Measures month-over-month sales growth |
| Running Total Sales   | `CALCULATE()` + `FILTER()` | Calculates cumulative sales            |
| Item Sales Rank       | `RANKX()`                    | Ranks items by sales                   |
| Average Selling Price | `DIVIDE()`                   | Calculates sales per quantity          |

---

# Validation

Each measure was reviewed after creation and tested using the Power BI report visuals.

The measures were checked for:

- Correct relationship with the date dimension
- Correct response to slicers
- Correct sales aggregation
- Correct ranking order
- Division-by-zero handling
- Appropriate formatting for percentage and currency values

The final measures were then used meaningfully in the dashboard visuals.
