# Data Cleaning: UK Online Sales Dataset

## The dataset originally contains 396,370 records. The cleaning process included removing duplicates, standardizing values, handling missing data, and ensuring consistency across key columns.
## Full dataset here

```sql

SELECT 
  *
FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet` 
;
```
 ## Step 1. Check and remove duplicates
```sql
WITH cte_duplicates AS (
SELECT 
  *,
  row_number() over(partition by InvoiceNo, CustomerID, StockCode, Description, Quantity, InvoiceDate,  CAST(UnitPrice AS STRING), Country) as row_num
FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet`
)
SELECT 
*
FROM cte_duplicates
WHERE 
  row_num > 1
;
```
### found 5187 duplicates 
## Step 1.1 create a table without duplicates
```sql
CREATE TABLE  `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`(
`InvoiceNo` INT64,
`CustomerID` INT64,
`StockCode` STRING,
`Description` STRING,
`Quantity` INT64,
`InvoiceDate` TIMESTAMP,
`UnitPrice` FLoat64,
`Country` STRING,
`row_num` int64
)
```
;
```sql
INSERT INTO  `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
SELECT 
  *,
   row_number() over(partition by InvoiceNo, CustomerID, StockCode, Description, Quantity, InvoiceDate,  CAST(UnitPrice AS STRING), Country)
FROM  `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet`
;
```
```sql
SELECT *
FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
LIMIT 20
;
```
```sql
DELETE
FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
WHERE row_num > 1
;
```
### removed 5,187 rows

## Step 2 Standarizing the data
## Step 2.1 Investigate Unit Price Anomalies
```sql
SELECT 
*
FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
ORDER BY UnitPrice asc 
LIMIT 50
```
;

## Find 0 values
```sql
SELECT 
*
FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
WHERE 
UnitPrice = 0.0
;
```
### Found 33 0 values

## Replace the 0 values to the average price of the product
```sql
SELECT
  t2.Stockcode,
  avg(t1.UnitPrice) as avr_price_to_change,
FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2` as t1
JOIN `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2` as t2
ON t1.StockCode = t2.StockCode
WHERE t1.UnitPrice > 0
and t2.UnitPrice = 0
GROUP BY t2.Stockcode
;
```
```sql
UPDATE `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2` AS main
SET main.UnitPrice = avg_data.avg_price
FROM (
  SELECT
    StockCode,
    AVG(UnitPrice) AS avg_price
  FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
  WHERE UnitPrice > 0
  GROUP BY StockCode
) AS avg_data
WHERE
  main.UnitPrice = 0
  AND main.StockCode = avg_data.StockCode
;
```
### 33 values got replaces

## Round up UnitPrice column to maximum to 2 decimal
```sql
UPDATE `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
SET UnitPrice = round(UnitPrice, 2)
WHERE UnitPrice is not null
;
```
## Step 2.2 Quantity column check
```sql
SELECT 
  *
FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
ORDER BY Quantity desc
LIMIT 40
;
```

### After running the query, we identified three transactions with unusually high quantity values, which suggest potential data entry errors. These quantities are significantly higher than the general trend in the dataset. Based on this observation, I assumed these were input mistakes and decided to correct them by removing the last digit in each of the three records.
```sql
SELECT 
  Quantity,
  DIV(Quantity, 10) as corrected
FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
WHERE
 Quantity > 5000
;
```

```sql
UPDATE `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
SET Quantity = DIV(Quantity, 10)
WHERE Quantity > 5000
;
```
## Step 2.3 Fixing  the unconsistent casing in the description column
```sql
UPDATE  `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
SET Description = INITCAP(LOWER(TRIM(Description)))
WHERE Description != INITCAP(LOWER(TRIM(Description)))
;
```
### This querry modified  83,452 rows 

## Invastigate the InvoiceDate data range

```sql
SELECT 
  min(InvoiceDate) as oldest_purchase,
  max(InvoiceDate) as latest_purhase
FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
;
```
### After checking the date it seems like all the data is in the expected data range

## step 2.4 Checking the customer ID and InvoiceNo columns

```sql
SELECT 
  LENGTH(cast(CustomerID as STRING)) as length_of_customerID,
 count(CustomerID) as number_of_elements
FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
  GROUP BY LENGTH(cast(CustomerID as STRING))
;
```
```sql
SELECT
  distinct length(cast(InvoiceNo as STRING)) as length_of_InvoiceNo
FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
;
```
## Checking if an invoiceNo has only one customerID atteched
```sql
SELECT 
  InvoiceNo, 
  COUNT(distinct CustomerID) as customer_ID_check
FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2` 
GROUP BY 
  InvoiceNo
HAVING COUNT(distinct CustomerID) > 1
;
```
### No issues found

## Step 3 Add a new column with the calculated Revenue and delete the unneccary columns
## Step 3.1 Create the Revenue column 
```sql
ALTER TABLE `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2` 
ADD COLUMN Revenue Float64
;

UPDATE `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2` 
SET Revenue = Quantity * UnitPrice
WHERE  Quantity is not null
;
```
## Round up the revenue to 2 descimals
```sql
UPDATE `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
SET Revenue = ROUND(Revenue, 2)
WHERE Revenue is not null
```
## removing the unessecery columns (row_num column).
```sql
ALTER TABLE `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
DROP COLUMN `row_num`
```

## awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2 table is cleaned . Duplicates removed and the data is standardized. There is no null value in the table and unnecessery columns  were also removed .
```sql
SELECT
  *
FROM `awesome-nimbus-448511-d2.portfolio_project.products_online_sales_updated_worksheet_2`
ORDER BY Revenue desc
LIMIT 30
```



