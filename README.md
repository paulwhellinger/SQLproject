# Coffee Shop SQL Project

In this project, I will analyze a dataset of a Coffee shop's sales to uncover sales patterns and customer behaviors. The dataset includes 
transaction dates, timestamps, locations, and product details.

Using SQL, I aim to explore various aspects such as sales trends over time, peak sales periods, product popularity, and the impact of location
on sales performance. This project demonstrates the use of SQL for practical business analysis by transforming raw transactional data into 
actionable insights that can inform sales strategy, optimize operations, and support data-driven decision-making.

## Prepare

This dataset, created by Maven Analytics, comes from a fictitious coffee company based in New York City, with three locations across the city. It covers entries from January 1st, 2023, to June 30th, 2023. The dataset is designed as a practice tool for data analysts to apply and refine their analysis skills. Since it is fictional, I won’t be focusing on evaluating whether the data fully meets the ROCCCs criteria (Reliable, Original, Clear, Consistent, and Cited). However, I will still assess whether the data is usable in its current form or if it needs to be cleaned.
There is no unit given for the retail price of the products. We will assume it is in USD. 
 
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
Once again no data was displayed after running the query, so we know that we don't have any invalid or out-of-range values for these columns.
We currently have data for single transactions that show the quantity of the product purchased, the unit price of the product, but there isn't a column that shows the total amount made in the transaction. To aid in our analysis later we added a column named transaction_total that will give the total amount in the transaction. We also added a column that gives the month of the transction. Finally, we added a column that gives the day of the week the transaction took place. We used the following code to make these columns:
#### Formulas Used:

- `=D2*H2` to calculate total sales.
- `=MONTH(B2)` to extract the month from the transaction date.
- `=TEXT(B2, "dddd")` to extract day of week from transaction date. 

The new spreadsheet was saved and uploaded into BigQuery as Coffee_Shop_Sales_Expanded.
The data seems validated enough to continue on with our analysis. 

## Anaylze 

In the  first part of the data analysis, we took a look at the dataset using Excel to get some basic facts about the data. Looking at the spreadsheet
and reversing the orders we can see that there is a total number of 149456 transactions, which span from Jan 1st 2023 to Jun 30th 2023. There also are
87 different product ids. 

First I checked to see if there were any missing product IDs. We know from the spreadsheet that the max number of product IDs is 87. Running the following
code will show the number of disctinct IDs in the dataset.
```sql
SELECT COUNT(DISTINCT product_id) AS total_product_ids
FROM my-project-1178-441803.Coffee_Shop_Sales.Coffee_Shop_Sales_Expanded;
```
Running the code will produce the following results:
|Row | total_product_ids|
|----|------------------|
|1   |80                |

This means that there are 7 missing product IDs or rather there were 7 products that were not bought during the dataset's timeframe. Using the following code, we looked at the distinct product ids and the number of times each one them we purchased.
```sql
SELECT product_id, COUNT(*) AS total_sales
FROM my-project-1178-441803.Coffee_Shop_Sales.Coffee_Shop_Sales_Expanded
GROUP BY product_id
ORDER BY product_id;
```
Looking at the outputed data, we see that product ids 62, 66, 67, 68, 80, 85 and 86 were not purchased at any location between the first 6 months of 2023. This means that we also
don't have the descriptions of these products, so we'll be unable to gather further insight into why they might not have been purchased. 

Using the following SQL code, we will find out more details about the variety of products that these coffeeshops offer. We want to find out all the distinct product categories there are. In addition, we'll find the total amounts made for each product and the percentage of total sales they made up. 
```sql
SELECT 
  product_category, 
  ROUND(SUM(transaction_total), 1) AS total_revenue,
  ROUND((SUM(transaction_total) / (SELECT SUM(transaction_total) FROM my-project-1178-441803.Coffee_Shop_Sales.Coffee_Shop_Sales_Expanded)) * 100, 1) AS percentage_of_total_sales
FROM my-project-1178-441803.Coffee_Shop_Sales.Coffee_Shop_Sales_Expanded
GROUP BY product_category
ORDER BY product_category;
```
### Output
|product_category|	total_revenue     |	percentage_of_total_sales|
|----------------|-------------------|--------------------------|
|Bakery|	82315.6|	11.8|
|Branded	|13607.0|	1.9|
|Coffee	|269952.5	|38.6|
|Coffee beans|	40085.3	|5.7|
|Drinking Chocolate|	72416.0|	10.4|
|Flavours|	8408.8|	1.2|
|Loose Tea|	11213.6|	1.6|
|Packaged Chocolate|	4407.6|	0.6|
|Tea	|196406.0	|28.1|

To get a better look at the data, we first looked at the total monthly transactions. We used the following SQL query to showcase the amounts made in transactions every month:

```sql
SELECT 
  transaction_month,
  ROUND(SUM(transaction_total), 2) AS monthly_total,
  ROUND((SUM(transaction_total) / (SELECT SUM(transaction_total) 
                                    FROM my-project-1178-441803.Coffee_Shop_Sales.Coffee_Shop_Sales_Expanded)) * 100, 2) AS percentage_of_total_sales
FROM my-project-1178-441803.Coffee_Shop_Sales.Coffee_Shop_Sales_Expanded
GROUP BY transaction_month
ORDER BY transaction_month;
```

### Total sales by month
|transaction_month|	monthly_total|	percentage_of_total_sales|
|-----------------|--------------|--------------------------|
|1|	81677.74	|11.69|
|2	|76145.19	|10.9|
|3|	98834.68	|14.14|
|4	|118941.08	|17.02|
|5|	156727.76	|22.43|
|6	|166485.88	|23.82|

The overall sales were much higher in the last three months of the year compared to the next three. Next we'll take a more detailed look specifically at just coffee sales each month and see how they compare to each other. 
```sql
SELECT 
  transaction_month,
  ROUND(SUM(transaction_total), 1) AS total_revenue,
  ROUND((SUM(transaction_total) / (SELECT SUM(transaction_total) 
                                    FROM my-project-1178-441803.Coffee_Shop_Sales.Coffee_Shop_Sales_Expanded 
                                    WHERE product_category = 'Coffee')) * 100, 1) AS percentage_of_total_sales
FROM my-project-1178-441803.Coffee_Shop_Sales.Coffee_Shop_Sales_Expanded
WHERE product_category = 'Coffee'
GROUP BY transaction_month
ORDER BY transaction_month;
```
### Monthly coffee sales excluding bagged coffee
|transaction_month|	total_revenue|	percentage_of_total_sales|
|-----------------|--------------|--------------------------|
|1|	31256.9	|11.6|
|2|	29269.0|	10.8|
|3|	38303.6	|14.2|
|4|45971.2	|17.0|
|5|	60362.8	|22.4|
|6	|64789.0	|24.0|

I changed the product category from Coffee to Tea, and then to Bakery, Drinking Chocolate and finally Coffee beans. They all followed the same pattern. The total revenue for each product the last two months were each roughly double the size of the revenue in the first three months. April saw a rise in sales, but didn't total match the performances in the last two months,  

Next I looked at which days of the week did the best. We'll use a similar code syntax as we've been using in order to see the percentages of total sales that each day made.
```sql
SELECT 
  day_of_week,
  ROUND(SUM(transaction_total), 2) AS total_revenue,
  ROUND((SUM(transaction_total) / (SELECT SUM(transaction_total) 
                                   FROM my-project-1178-441803.Coffee_Shop_Sales.Coffee_Shop_Sales_Expanded)) * 100, 2) AS percentage_of_total_sales
FROM my-project-1178-441803.Coffee_Shop_Sales.Coffee_Shop_Sales_Expanded
GROUP BY day_of_week
ORDER BY total_revenue DESC;
```
### 

|day_of_week|	total_revenue	|percentage_of_total_sales|
|-----------|---------------|-------------------------|
|Monday|	101677.28	|14.55|
|Friday	|101373.0	|14.51|
|Thursday	|100767.78	|14.42|
|Wednesday	|100313.54	|14.35|
|Tuesday|	99455.94	|14.23|
|Sunday|	98330.31|	14.07|
|Saturday	|96894.48	|13.87|

Although the sales are pretty consistent day to day, the weekdays tend to do better than the weekends. Monday and Friday have the most sales out of any other days of the week. Saturday and Sunday on the other hand are the slowest days of the week as far as sales are concerned. 
