# Coffee Shop SQL Project

In this project, I will analyze a dataset of a Coffee shop's sales to uncover sales patterns and customer behaviors. The dataset includes 
transaction dates, timestamps, locations, and product details.

Using SQL, I aim to explore various aspects such as sales trends over time, peak sales periods, product popularity, and the impact of location
on sales performance. This project demonstrates the use of SQL for practical business analysis by transforming raw transactional data into 
actionable insights that can inform sales strategy, optimize operations, and support data-driven decision-making.

## Prepare

This dataset is from a fictious coffee company based in NY city, which has 3 locations throughout the city. The entries are from Jan 1st 2023 to Jun 30th 2023. This is a
fictious dataset that is meant as an example for data analysist to practice their analysis processes on. Due to this fact, I won't look too vigously into whether or not the data
ROCCCs (is it reliable, origional, clear, consist, and citied). The dataset was created by Maven Analytics. We will still check the data to make sure whether or not the data
is useable in its current state or if it needs to be cleaned.

 
## Process 

The dataset is contained in a single .xlsx file that I converted to a .csv file. The .csv file was then uplaoded to BigQuery for analysis using 
SQL.

I first wanted to check the data for any NULL values.

```sql
SELECT
  COUNTIF(transaction_id IS NULL) AS null_transaction_id,
  COUNTIF(transaction_date IS NULL) AS null_transaction_date,
  COUNTIF(transaction_time IS NULL) AS null_transaction_time,
  COUNTIF(transaction_qty IS NULL) AS null_transaction_qty,
  COUNTIF(store_id IS NULL) AS null_store_id,
  COUNTIF(store_location IS NULL) AS null_store_location,
  COUNTIF(product_id IS NULL) AS null_product_id,
  COUNTIF(unit_price IS NULL) AS null_unit_price,
  COUNTIF(product_category IS NULL) AS null_product_category,
  COUNTIF(product_type IS NULL) AS null_product_type,
  COUNTIF(product_detail IS NULL) AS null_product_detail
FROM `my-project-1178-441803.Coffee_Shop_Sales.Coffee_Shop_Sales`;
```

| null_transaction_id | null_transaction_date | null_transaction_time | null_transaction_qty | null_store_id | null_store_location | null_product_id | null_unit_price | null_product_category | null_product_type | null_product_detail |
|---------------------|-----------------------|-----------------------|----------------------|---------------|---------------------|-----------------|-----------------|-----------------------|-------------------|---------------------|
| 0                   | 0                     | 0                     | 0                    | 0             | 0                   | 0               | 0               | 0                     | 0                 | 0                   |

As we can see from the chart there are no NULL values in the dataset.
Now let's check to see if there are any duplicate entries. 

```sql
SELECT transaction_id, COUNT(*)
FROM `my-project-1178-441803.Coffee_Shop_Sales.Coffee_Shop_Sales`
GROUP BY transaction_id
HAVING COUNT(*) > 1;
```
After running the query there was no data displayed meaning there were no duplicate entries.
Next let's check for any invalid or out-of-range values. We'll specifically look at the transaction quanty and unit prices. Any negative 
quantities in these columns would be considered invalid. 
```sql
SELECT *
FROM my-project-1178-441803.Coffee_Shop_Sales.Coffee_Shop_Sales
WHERE transaction_qty <= 0 OR unit_price <= 0;
``` 
Once again no data was displayed after running the query, so we know that we don't have any invalid or out-of-range values for these columns
The data seems validated enough to continue on with our analysis of it. 

## 
