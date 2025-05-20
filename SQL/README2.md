# UK Online Sales Analysis
## 1.	Which individual products and product categories generate the highest revenue and quantity sold across the entire dataset?

## Top 10 Best Performing item (Revenue)
```sql
SELECT
  *
FROM (SELECT 
    sales.StockCode,
    product.Description,
    product.Category,
    ROUND(SUM(Revenue),2) as total_revenue,
    SUM(sales.Quantity) as total_quantity,
    RANK() OVER(ORDER BY SUM(Revenue) desc) as rank_revenue,
    RANK() OVER(ORDER BY SUM(Quantity) desc) as rank_quantity
  FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2` as sales
    JOIN `awesome-nimbus-448511-d2.portfolio_project.cleaned_product_category` as product 
    ON sales.StockCode = product.StockCode
  GROUP BY sales.StockCode, product.Description, product.Category
)
WHERE
rank_revenue < 11 
ORDER BY rank_revenue asc
;
```
## Top 10 Best Performing item (Revenue)

```sql
SELECT
  *
FROM (SELECT 
    sales.StockCode,
    product.Description,
    product.Category,
    ROUND(SUM(Revenue),2) as total_revenue,
    SUM(sales.Quantity) as total_quantity,
    RANK() OVER(ORDER BY SUM(Revenue) desc) as rank_revenue,
    RANK() OVER(ORDER BY SUM(Quantity) desc) as rank_quantity
  FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2` as sales
    JOIN `awesome-nimbus-448511-d2.portfolio_project.cleaned_product_category` as product 
    ON sales.StockCode = product.StockCode
  GROUP BY sales.StockCode, product.Description, product.Category
)
WHERE
rank_quantity < 11 
ORDER BY rank_quantity asc
;
```
## Categorizing products into 5 performance-based groups using sales quantity and revenue quartiles:
### 1. Best Performer: High revenue & moderate to high quantity
### 2. High Volume, Low Income: High quantity, low revenue
### 3. Expensive Products: High revenue, low quantity
### 4. Solid Performer: Moderate-to-high in both quantity and revenue
### 5. Under Performer: Low in both dimensions
```sql
-- Step 1: Prepare sales data with ranking and quartiles

WITH product_stats AS (
  SELECT 
    sales.StockCode,
    product.Description,
    product.Category,
    ROUND(SUM(Revenue),2) as total_revenue,
    SUM(sales.Quantity) as total_quantity,
    ntile(4) OVER(Order BY SUM(Revenue) desc) as revenue_quartile,
    ntile(4) OVER(Order BY SUM(Quantity) desc) as quantity_quartile
  FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2` as sales
    JOIN `awesome-nimbus-448511-d2.portfolio_project.cleaned_product_category` as product 
    ON sales.StockCode = product.StockCode
  GROUP BY sales.StockCode, product.Description, product.Category
),

-- Step 2: Categorize based on performance
product_categories as (
    SELECT
      *,
      CASE
      WHEN revenue_quartile = 1 AND quantity_quartile IN (1,2) THEN "Best Perfomer"
      WHEN revenue_quartile IN (3,4) AND quantity_quartile IN (1,2)THEN "High Volume, Low Income"
      WHEN revenue_quartile IN (1,2) AND quantity_quartile IN (3,4) THEN "High Margin, Low Reach" 
      WHEN revenue_quartile <= 2 AND quantity_quartile <=2 THEN "Consistent Performer"
      ELSE "Low Performer"
      END AS Performance_category
    FROM product_stats),
total_performance as (
SELECT
Performance_category,
ROUND(SUM(product_categories.total_revenue),2)  as total_r,
SUM(product_categories.total_quantity)  as total_q
FROM product_categories
GROUP BY Performance_category
)

-- Step 3: Summary of the categories

SELECT 
  *,
  SUM(total_r) OVER() as grand_total_revenue,
  SUM(total_q) OVER() as grand_total_quantity,
  ROUND(total_r * 100/ SUM(total_r) OVER(),2 ) as percanatge_r,
  ROUND(total_q * 100/ SUM(total_q) OVER(),2) as percantage_q
FROM total_performance
;
```

## Which product categories are most common in the low perfomer and high margin low income categories

### Low perfomer products categories
```sql
WITH product_stats AS (
  SELECT 
    sales.StockCode,
    product.Description,
    product.Category,
    ROUND(SUM(Revenue),2) as total_revenue,
    SUM(sales.Quantity) as total_quantity,
    ntile(4) OVER(Order BY SUM(Revenue) desc) as revenue_quartile,
    ntile(4) OVER(Order BY SUM(Quantity) desc) as quantity_quartile
  FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2` as sales
    JOIN `awesome-nimbus-448511-d2.portfolio_project.cleaned_product_category` as product 
    ON sales.StockCode = product.StockCode
  GROUP BY sales.StockCode, product.Description, product.Category
)
SELECT
CAtegory,
COUNT(Category) as num_of_cat
from product_stats
WHERE revenue_quartile in(3,4)
AND quantity_quartile in(3,4)
group by Category
ORDER BY num_of_cat desc
;
```
### High volume, low income products Categories

```sql
WITH product_stats AS (
  SELECT 
    sales.StockCode,
    product.Description,
    product.Category,
    ROUND(SUM(Revenue),2) as total_revenue,
    SUM(sales.Quantity) as total_quantity,
    ntile(4) OVER(Order BY SUM(Revenue) desc) as revenue_quartile,
    ntile(4) OVER(Order BY SUM(Quantity) desc) as quantity_quartile
  FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2` as sales
    JOIN `awesome-nimbus-448511-d2.portfolio_project.cleaned_product_category` as product 
    ON sales.StockCode = product.StockCode
  GROUP BY sales.StockCode, product.Description, product.Category
)
SELECT
CAtegory,
COUNT(Category) as num_of_cat
from product_stats
WHERE revenue_quartile in(3,4)
AND quantity_quartile in(1,2)
group by Category
ORDER BY num_of_cat desc
;

```


## 2.	Are there noticeable monthly or seasonal patterns in customer purchasing behavior throughout the year?


## monthly revenue  and quantity made 
```sql
SELECT 
  EXTRACT(YEAR from InvoiceDate) as year,
  EXTRACT(MONTH from InvoiceDate) as month,
  FORMAT_TIMESTAMP ("%B",InvoiceDate ) as month_name,
  ROUND(SUM(Revenue),0) as total_revenue,
  SUM(Quantity) as total_quantity,
FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
GROUP BY year, month, month_name
ORDER BY year, month asc
;
```

## Seanonal breakdown by revenue and quantity

```sql
SELECT
 CASE
  WHEN month in (12,1,2) THEN "Winter"
  WHEN month in(3,4,5) THEN "Spring"
  WHEN month in (6,7,8) THEN "Summer"
  WHEN month in (9,10,11) THEN "Fall"
  END as seasons,
  SUM(total_revenue) as seasonal_revenue,
  SUM(total_quantity) as seasonal_quantity
FROM (SELECT 
      EXTRACT(MONTH from InvoiceDate) as month,
      ROUND(SUM(Revenue),0) as total_revenue,
      SUM(Quantity) as total_quantity
    FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
    GROUP BY  month
    ORDER BY  month asc
    )
GROUP BY seasons
;
```

## Average revenue per day 
### Note: No sales data exists for Saturdays in the dataset. This is unusual for online sales and may indicate missing data or business closure.

```sql
SELECT 
  FORMAT_DATE("%A", DATE(InvoiceDate)) as days,
  ROUND(SUM(Revenue),1) as total_income,
  SUM(Quantity) as total_quanity,
   ROUND(SUM(Revenue),1) / SUM(Quantity) as avg_item_spent
FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
GROUP BY days
ORDER BY total_income desc
;
```

 
## Top 5 perfroming products in each season
```sql
SELECT
  *
FROM (SELECT
  StockCode,
  Description,
  seasons,
  CAST(SUM(Revenue) as INT64) as total,
  RANK() OVER (partition by seasons order by SUM(Revenue) desc) as rank_product
    FROM (SELECT
      StockCode,
      Description,
      Case
      WHEN month in (12,1,2) THEN "Winter"
      WHEN month in(3,4,5) THEN "Spring"
      WHEN month in (6,7,8) THEN "Summer"
      WHEN month in (9,10,11) THEN "Fall"
      END as seasons,
      Revenue
    FROM (SELECT
          sales.StockCode,
          product.Description,
          Revenue,
          EXTRACT(MONTH from InvoiceDate) as month
        FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2` as sales
        JOIN `awesome-nimbus-448511-d2.portfolio_project.cleaned_product_category` as product
        ON sales.StockCode = product.StockCode
       
    )
    )
 GROUP BY Seasons, StockCode, Description
)
WHERE rank_product < 6
;
```
## Top 5 perfroming category in each season

```sql
SELECT
  *
FROM (SELECT
  Category,
  seasons,
  CAST(SUM(Revenue) as INT64) as total,
  RANK() OVER (partition by seasons order by SUM(Revenue) desc) as rank_product
    FROM (SELECT
      Category,
      Case
      WHEN month in (12,1,2) THEN "Winter"
      WHEN month in(3,4,5) THEN "Spring"
      WHEN month in (6,7,8) THEN "Summer"
      WHEN month in (9,10,11) THEN "Fall"
      END as seasons,
      Revenue
    FROM (SELECT
          product.Category,
          Revenue,
          EXTRACT(MONTH from InvoiceDate) as month
        FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2` as sales
        JOIN `awesome-nimbus-448511-d2.portfolio_project.cleaned_product_category` as product
        ON sales.StockCode = product.StockCode
       
    )
    )
 GROUP BY Seasons,  Category
)
WHERE rank_product < 6
;
```

## 3.	Which countries or regions contribute the most to total revenue, and how does revenue distribution vary across locations?

## Best performing regions by revenue 
```sql
SELECT
  region,
  SUM(Revenue) as total_revenue,
  SUM(Quantity) as total_quantity
FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2` as s
LEFT JOIN `awesome-nimbus-448511-d2.portfolio_project.continents` as c
ON s.Country = c.name
GROUP BY region
ORDER BY total_revenue desc
;
```
## the countries generating the biggest revenue. compare to the grand total revenue

```sql
WITH country_revenue as (
  SELECT
    Country,
    ROUND(SUM(REvenue),0) as total_revenue_country
  FROM`awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
  GROUP BY Country)
SELECT
  *,
  SUM(country_revenue.total_revenue_country) OVER() as grand_total_revenue,
  ROUND(total_revenue_country * 100 / SUM(country_revenue.total_revenue_country) OVER(),2) as percentage_of_total
FROM country_revenue
ORDER BY percentage_of_total desc

;
```
##top selling categories in the top 6 best revenue generated countries
```sql
--Step 1. Get top 6 countries by revenue 

WITH cte AS (
SELECT
  country,
  p.Category,
  Revenue
FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2` as s 
JOIN `awesome-nimbus-448511-d2.portfolio_project.cleaned_product_category` as p
ON s.StockCode = p.StockCode
WHERE Country IN(SELECT
  Country
    FROM (SELECT
    Country,
    ROUND(SUM(Revenue),0) as total_revenue
    FROM`awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
    GROUP BY Country
    ORDER BY total_revenue desc
    LIMIT 6
))),

--Step 2. group the countries and the category and then rank those by the total revenue made

cte2 as(
SELECT
  country,
  Category,
  CAST(SUM(Revenue) as INT64) as total_revenue,
  RANK() OVER (PARTITION BY Country ORDER BY SUM(Revenue) desc) as ranking
FROM cte
GROUP BY Country, Category
)

--Step 3. Filter to the top 5 selling categories in each country 

SELECT 
  *
FROM cte2
WHERE ranking < 5
ORDER BY country, ranking asc 
;
```

##Europe without the UK Revenue breakdown
```sql
with cte as(
SELECT 
Country,
SUM(revenue) as revenue_by_country,
region
FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2` as t1
INNER JOIN `awesome-nimbus-448511-d2.portfolio_project.continents`  as t2
ON t1.Country = t2.name 
WHERE region = "Europe"
GROUP BY Country, region
)
SELECT
  Country,
  revenue_by_country,
  SUM(revenue_by_country) OVER() as grand_total,
  revenue_by_country*100 / SUM(revenue_by_country) OVER() as percentage
FROM cte
ORDER BY percentage desc
```





