# Coffee Shop SQL Project

In this project, I will analyze a dataset of a Coffee shop's sales to uncover sales patterns and customer behaviors. The dataset includes 
transaction dates, timestamps, locations, and product details.

Using SQL, I aim to explore various aspects such as sales trends over time, peak sales periods, product popularity, and the impact of location
on sales performance. This project demonstrates the use of SQL for practical business analysis by transforming raw transactional data into 
actionable insights that can inform sales strategy, optimize operations, and support data-driven decision-making.

## Prepare 

The dataset is contained in a single .xlsx file that I converted to a .csv file. The .csv file was then uplaoded to BigQuery for analysis using 
SQL.

I first wanted to check the data for any Null values.

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
