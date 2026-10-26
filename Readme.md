## 📐 Data Cleansing Challenge & SQL Solution
The raw dataset contained critical formatting anomalies that prevented traditional mathematical computing:
1. **Corrupt Date Attributes:** Stored as generic strings in `DD/MM/YYYY` format, which SQLite cannot sort or group.
2. **Text-Formatted Currency:** The `unit_price` field included text characters (`$`), rendering mathematical multiplication impossible.

### The Engineered SQL Script
The solution uses advanced string manipulation (`SUBSTR`, `REPLACE`) inside an optimized **Common Table Expression (CTE)** framework to funnel 10,000+ individual transactions down into **48 clean, chronological monthly summary records**:

```sql
WITH CleanedData AS (
    SELECT 
        -- Rebuilds DD/MM/YYYY text into standard YYYY-MM-DD format
        SUBSTR(date, 7, 4) || '-' || SUBSTR(date, 4, 2) || '-' || SUBSTR(date, 1, 2) AS clean_date,
        
        -- Removes the '\$' sign and cleans hidden spaces from the price
        CAST(REPLACE(TRIM(unit_price), '\$', '') AS REAL) AS numeric_price,
        
        -- Keeps quantity as a clean integer
        CAST(TRIM(quantity) AS INTEGER) AS numeric_qty
    FROM Walmart
),
CalculatedSales AS (
    SELECT 
        -- Groups cleaned strings into a unified Year-Month calendar track
        SUBSTR(clean_date, 1, 7) AS sales_month,
        SUM(numeric_price * numeric_qty) AS current_month_sales
    FROM CleanedData
    WHERE clean_date IS NOT NULL AND clean_date LIKE '20%'
    GROUP BY 1
)
SELECT 
    sales_month,
    ROUND(current_month_sales, 2) AS monthly_revenue,
    
    -- Window Function: Fetches the previous month's revenue to compare
    ROUND(LAG(current_month_sales) OVER (ORDER BY sales_month), 2) AS previous_month_revenue,
    
    -- Calculates the percentage change month-over-month
    ROUND(
        ((current_month_sales - LAG(current_month_sales) OVER (ORDER BY sales_month)) 
         / LAG(current_month_sales) OVER (ORDER BY sales_month)) * 100, 2
    ) AS mom_growth_percentage
FROM CalculatedSales
ORDER BY sales_month;
```

---

## 📊 Excel Dashboard Deliverables
The aggregated 48-row analytical grid was connected directly to Microsoft Excel to form an executive reporting system:
* **Dynamic KPI Callout Blocks:** Dark-themed structural cards computing **Total Revenue**, **Average Monthly Volume**, and **Latest Performance Variations** dynamically.
* **Continuous Timeline Visuals:** A continuous financial trend graph built to replace raw transactional noise with actionable macro-level tracking.