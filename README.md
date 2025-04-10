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
### Available Products and Assiocated Revenues 
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

### Total Sales by Month Jan(1) to Jun(6)
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

I changed the product category from Coffee to Tea, and then to Bakery, Drinking Chocolate and finally Coffee beans. They all followed the same pattern. The total revenue for each product the last two months were each roughly double the size of the revenue in the first three months. April saw a rise in sales, but didn't total match the performances in the last two months. I also looked at the products with the lowest revenue and saw the same pattern. 

  

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

Although sales are pretty consistent day to day, the weekdays tend to do better than the weekends. Monday and Friday have the most sales out of any other days of the week. Saturday and Sunday on the other hand are the slowest days of the week as far as sales are concerned. 

Interesting note, looking at the spreadsheet of this data set we see that the max value for transaction_qty is 8. This quantity is assiocated with sales of bagged coffee, labeled "Coffee beans" in the dataset. Running the following code, I was able to see all of the transactions of bagged coffee that were greater than 1:
```sql
SELECT *
FROM my-project-1178-441803.Coffee_Shop_Sales.Coffee_Shop_Sales_Expanded
WHERE product_category = 'Coffee beans'
  AND transaction_qty > 1;
```
The results showed that transactions of 8 bags of coffee occuried twice a month at the same Hell's Kitchen location, excluding February and March and were the exact same every time. Also Premium Beans, always Civet Cat.  It also showed that transaction of 2 bags of coffee were being purchased from a Lower Manhattan location every month, also excluding February and March. The transactions of 8 bags of coffee twice a month suggests that another coffee shop or cafe were purchasing large orders of beans to use at their own business. 

Next, I looked more specifically at the different product types that the coffee shops offered. Together Coffee and Tea products make up around two thirds of the total revenue. Due to this, we will only be looking at Coffee and Tea products. Running the following code (which uses a familiar syntax at this point) , I was able to make two different charts displaying the different coffee and tea products and their assiocated revenues.

### Coffee Products
```sql
SELECT 
  product_type,
  ROUND(SUM(transaction_total), 2) AS total_revenue,
  COUNT(*) AS total_transactions,
  ROUND(
    (SUM(transaction_total) / (
      SELECT SUM(transaction_total)
      FROM my-project-1178-441803.Coffee_Shop_Sales.Coffee_Shop_Sales_Expanded
      WHERE product_category = 'Coffee'
    )) * 100, 1
  ) AS percent_of_total_revenue
FROM my-project-1178-441803.Coffee_Shop_Sales.Coffee_Shop_Sales_Expanded
WHERE product_category = 'Coffee'
GROUP BY product_type
ORDER BY total_revenue DESC;

```
|product_type	|total_revenue	|total_transactions	|percent_of_total_revenue|
|-------------|--------------|-------------------|------------------------|
|Barista Espresso	|91406.2|	16403|	33.9|
|Gourmet brewed coffee	|70034.6	|16912	|25.9|
|Premium brewed coffee	|38781.15	|8135	|14.4|
|Organic brewed coffee	|37746.5|	8489	|14.0|
|Drip coffee	|31984.0	|8477	|11.8|
### Tea Products
```sql
SELECT 
  product_type,
  ROUND(SUM(transaction_total), 2) AS total_revenue,
  COUNT(*) AS total_transactions,
  ROUND(
    (SUM(transaction_total) / (
      SELECT SUM(transaction_total)
      FROM my-project-1178-441803.Coffee_Shop_Sales.Coffee_Shop_Sales_Expanded
      WHERE product_category = 'Tea'
    )) * 100, 1
  ) AS percent_of_total_revenue
FROM my-project-1178-441803.Coffee_Shop_Sales.Coffee_Shop_Sales_Expanded
WHERE product_category = 'Tea'
GROUP BY product_type
ORDER BY total_revenue DESC;
```
|product_type	|total_revenue	|total_transactions|	percent_of_total_revenue|
|-------------|--------------|------------------|-------------------------|
|Brewed Chai tea|	77081.95	|17183	|39.2|
|Brewed Black tea	|47932.0	|11350|	24.4|
|Brewed herbal tea|	47539.5|	11245	|24.2|
|Brewed Green tea	|23852.5|	5671	|12.1|

As far as coffee products are concerned, we can see a noticable increase in both number of transactions and revenue with a raise of product quality. Assuming that drip coffee has the worst quality and Gourmet as the best (excluding espresso as its a different type of drink), we see out of the different coffees that Gourmet has the has amount of transactions and revenue.
We will now take the analysis further by looking at the different product details for both Coffee and Tea products. Using the following SQL query will create two tables giving the product type, product detail, total revenue, corresponding unit prices and total quantities sold. 
### Coffee Products Expanded
```sql
SELECT 
  product_detail,
  product_type,
  ROUND(SUM(transaction_total), 2) AS total_revenue,
  unit_price,
  SUM(transaction_qty) AS total_qty_sold
FROM my-project-1178-441803.Coffee_Shop_Sales.Coffee_Shop_Sales_Expanded
WHERE product_category = 'Coffee'
GROUP BY product_detail, product_type, unit_price
ORDER BY total_revenue DESC;
```
| product_detail               | product_type            | total_revenue | unit_price | total_qty_sold |
|------------------------------|-------------------------|---------------|------------|----------------|
| Latte Rg                     | Barista Espresso        | 19112.25      | 4.25       | 4497           |
| Cappuccino Lg                 | Barista Espresso        | 17641.75      | 4.25       | 4151           |
| Latte                         | Barista Espresso        | 17257.50      | 3.75       | 4602           |
| Jamaican Coffee River Lg      | Premium brewed coffee   | 16481.25      | 3.75       | 4395           |
| Cappuccino                    | Barista Espresso        | 15997.50      | 3.75       | 4266           |
| Brazilian Lg                  | Organic brewed coffee   | 15109.50      | 3.50       | 4317           |
| Ethiopia Lg                   | Gourmet brewed coffee   | 14794.50      | 3.50       | 4227           |
| Ethiopia Rg                   | Gourmet brewed coffee   | 13179.00      | 3.00       | 4393           |
| Brazilian Rg                  | Organic brewed coffee   | 13155.00      | 3.00       | 4385           |
| Columbian Medium Roast Lg     | Gourmet brewed coffee   | 12585.00      | 3.00       | 4195           |
| Espresso shot                 | Barista Espresso        | 12495.00      | 3.00       | 4165           |
| Jamaican Coffee River Rg      | Premium brewed coffee   | 12455.80      | 3.10       | 4018           |
| Our Old Time Diner Blend Lg   | Drip coffee             | 11991.00      | 3.00       | 3997           |
| Columbian Medium Roast Rg     | Gourmet brewed coffee   | 11367.50      | 2.50       | 4547           |
| Our Old Time Diner Blend Rg   | Drip coffee             | 11025.00      | 2.50       | 4410           |
| Jamaican Coffee River Sm      | Premium brewed coffee   | 9844.10       | 2.45       | 4018           |
| Ethiopia Sm                   | Gourmet brewed coffee   | 9752.60       | 2.20       | 4433           |
| Brazilian Sm                  | Organic brewed coffee   | 9482.00       | 2.20       | 4310           |
| Our Old Time Diner Blend Sm   | Drip coffee             | 8968.00       | 2.00       | 4484           |
| Columbian Medium Roast Sm     | Gourmet brewed coffee   | 8356.00       | 2.00       | 4178           |
| Ouro Brasileiro shot          | Barista Espresso        | 6840.00       | 3.00       | 2280           |
| Ouro Brasileiro shot          | Barista Espresso        | 2062.20       | 2.10       | 982            |
### Tea Products Expanded
```sql
SELECT 
  product_detail,
  product_type,
  ROUND(SUM(transaction_total), 2) AS total_revenue,
  unit_price,
  SUM(transaction_qty) AS total_qty_sold
FROM my-project-1178-441803.Coffee_Shop_Sales.Coffee_Shop_Sales_Expanded
WHERE product_category = 'Tea'
GROUP BY product_detail, product_type, unit_price
ORDER BY total_revenue DESC;
```
| product_detail                | product_type           | total_revenue | unit_price | total_qty_sold |
|-------------------------------|------------------------|---------------|------------|----------------|
| Morning Sunrise Chai Lg        | Brewed Chai tea        | 17384.00      | 4.00       | 4346           |
| Spicy Eye Opener Chai Lg       | Brewed Chai tea        | 13652.40      | 3.10       | 4404           |
| Peppermint Lg                  | Brewed herbal tea      | 13050.00      | 3.00       | 4350           |
| English Breakfast Lg           | Brewed Black tea       | 12927.00      | 3.00       | 4309           |
| Earl Grey Lg                   | Brewed Black tea       | 12735.00      | 3.00       | 4245           |
| Serenity Green Tea Lg          | Brewed Green tea       | 12660.00      | 3.00       | 4220           |
| Traditional Blend Chai Lg      | Brewed Chai tea        | 12522.00      | 3.00       | 4174           |
| Lemon Grass Lg                 | Brewed herbal tea      | 12267.00      | 3.00       | 4089           |
| Earl Grey Rg                   | Brewed Black tea       | 11770.00      | 2.50       | 4708           |
| Morning Sunrise Chai Rg        | Brewed Chai tea        | 11607.50      | 2.50       | 4643           |
| Peppermint Rg                  | Brewed herbal tea      | 11410.00      | 2.50       | 4564           |
| Traditional Blend Chai Rg      | Brewed Chai tea        | 11280.00      | 2.50       | 4512           |
| Serenity Green Tea Rg          | Brewed Green tea       | 11192.50      | 2.50       | 4477           |
| Lemon Grass Rg                 | Brewed herbal tea      | 10812.50      | 2.50       | 4325           |
| Spicy Eye Opener Chai Rg       | Brewed Chai tea        | 10636.05      | 2.55       | 4171           |
| English Breakfast Rg           | Brewed Black tea       | 10500.00      | 2.50       | 4200           |

Both of these charts show the same relationship between unit_price and total_qty_sold. The higher the unit_price, the higher the total_qty_sold and thus the higher the total_revenue. 
The customer's clearly enjoyed higher quality products. This relationship seems to be stronger when it comes to coffee products but it also mostly holds true for the tea products.

## Share
The following figures were created using Tableau 
![Sheet 11 (1)](https://github.com/user-attachments/assets/673cccde-8060-422b-82a8-c36da50811fb)
#### Figure One: Transaction Timeline
![Dashboard 1](https://github.com/user-attachments/assets/dfd8d290-8c91-463c-8f3f-30ae0f074422)
#### Figure Two: Transaction Totals by Weekday
![Sheet 1](https://github.com/user-attachments/assets/71388892-4927-4cbd-a4cf-377021781dc6)
#### Figure Three: Transaction Totals vs Unit Price (excludes some data points from bagged coffee)

### Findings:
- Fig 1: The amount of transactions and total revenue of all products increased greatly from the first three months of the year to the next three months. The lowest earning month was February and the highest was June. 
-Fig 2: The weekdays performed marginally better than the weekend. The highest preforming days were Monday and Friday and the lowest preforming days were Saturday and Sunday. This is only marginal though. Monday, the highest grossing day, had about 14.55% of total sales while Sunday, the lowest, made up about 13.87% of total sales.
-Fig 3: As unit price increased, total revenue also increased. Total amounts sold varied but there was a clear positive relationship between unit price of the coffees and the total revenue from those products. 

## Act

- The findings show that coffee products, particularly espresso drinks and higher quality coffee drinks, were the highest drivers of revenue. When it came to coffee products there was also a correlation between unit prices and revenue; the higher the price, the higher the revenue. These coffee shop seem to be servering a clientele that doesn't mind, and if anything wishes, to pay more for higher quality. Expanding espresso drinks or more premium coffee options could drive further revenue gains. 

- Weekdays were slightly busier than weekend days. Also, the summer early summer months were much busier than the beginning of the year. Both of these facts will be important to note when determining appriorate levels of staffing for each day/month.  

- Although rare, there were great revenue seen by the purchase of bagged coffee on some frequent occasions. If more local cafes or coffee shops were interested in buying large quantities of coffee beans, a shift in business practice towards catering to these clientele could be very profitable.  
 




