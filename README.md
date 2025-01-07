# Pizza Sales and Revenue Analysis

## Project Title

### Pizza Sales and Revenue Analysis: Insights into Orders, Revenue, and Trends

## Project Description

This project explores a dataset containing information about pizza orders, revenue, and customer preferences to extract valuable insights for decision-making. The analysis leverages SQL queries to clean data, calculate revenue, and understand customer behavior, contributing to better inventory management, pricing strategies, and sales optimization.

## Objectives 

* Revenue Analysis: Calculate total revenue, revenue by pizza size, and revenue contributions of different pizza categories.

* Order Trends: Understand order distributions by date, time, and day of the week.

* Top Performers: Identify the most popular pizza sizes, types, and categories based on quantity and revenue.

* Customer Insights: Analyze cumulative revenue over time and the percentage contribution of pizza types to total revenue.

* Performance Breakdown: Highlight top-performing pizzas in each category and provide insights into peak ordering times.

## Key Features

1. Data Exploration: Includes schema inspection and raw data review for tables like ORDERS, ORDER_DETAILS, PIZZAS, and PIZZA_TYPES.

2. Revenue Metrics: Total revenue calculation, revenue by pizza size, and cumulative revenue analysis.

3. Trend Analysis: Daily and monthly order counts, distribution by time of day, and seasonal patterns.

4. Top Products: Identification of the highest-priced pizza, most common pizza size, and top 5 pizza types by quantity and revenue.

5. Advanced Analysis: Rank-based insights and partitioned revenue evaluation for different pizza categories.

## SQL Queries Overview

* Schema Inspection: Use DESCRIBE to understand table structures.

* Revenue Calculations: Compute revenue by joining relevant tables and applying aggregate functions.

* Order Trends: Analyze data grouped by date, time, and day.

* Top Performers: Use ranking and sorting to highlight top-selling pizzas and their contributions.

* Insights by Time: Group orders by time intervals (morning, afternoon, evening) and assess peak periods.

## Analysis With SQL(MYSQL)
### CS1 : Extract only the table information
```sql
DESCRIBE ORDER_DETAILS;
DESCRIBE ORDERS;
DESCRIBE pizza_types;
DESCRIBE pizzas;

SELECT * FROM ORDERS;
SELECT * FROM ORDER_DETAILS;
SELECT * FROM PIZZAS;
SELECT * FROM PIZZA_TYPES;
```
### CS2 : Total Revenue from All Orders
```sql
WITH CTE AS
(SELECT 
    O.PIZZA_ID, O.QUANTITY, P.PRICE, QUANTITY * PRICE AS REVENUE
FROM
    ORDER_DETAILS AS O
        LEFT JOIN
    PIZZAS AS P ON P.PIZZA_ID = O.PIZZA_ID)

SELECT 
    ROUND(SUM(REVENUE), 2)
FROM
    CTE;
```
### CS3 : WHERE QUANTITY IS MORE THAN 1
```sql
WITH CTE AS
(SELECT 
    O.PIZZA_ID, O.QUANTITY, P.PRICE, QUANTITY * PRICE AS REVENUE
FROM
    ORDER_DETAILS AS O
        LEFT JOIN
    PIZZAS AS P ON P.PIZZA_ID = O.PIZZA_ID)

SELECT 
    *
FROM
    CTE
WHERE
    PRICE <> REVENUE;
```
### CS4 : REVENUE BY PIZZA SIZE
```sql
SELECT DISTINCT SIZE FROM PIZZAS;
SELECT 
    P.SIZE, ROUND(SUM(O.QUANTITY * P.PRICE), 1) AS REVENUE
FROM
    PIZZAS AS P
        LEFT JOIN
    ORDER_DETAILS AS O ON P.PIZZA_ID = O.PIZZA_ID
GROUP BY 1;
```
### CS5 : COUNT ORDERS BY DAY
```sql
 SELECT COUNT(DISTINCT DATE) FROM ORDERS;
 
SELECT 
    DATE, COUNT(QUANTITY) AS TOTAL_ORDERS
FROM
    ORDERS AS O
        LEFT JOIN
    ORDER_DETAILS AS OD ON O.ORDER_ID = OD.ORDER_ID
GROUP BY DATE;
```
 ### CS6: TOP 5 SALES
```sql
 	WITH CTE AS (SELECT
        DATE, 
        COUNT(ORDER_ID) AS DAILY_SALES,
        RANK() OVER (ORDER BY COUNT(ORDER_ID) DESC) AS RN
        FROM 
        ORDERS
        GROUP BY 1)
        
        SELECT * 
        FROM CTE
        WHERE RN <= 5;
 ```
### CS7: COUNT OF ORDERS IN DECEMBER
```sql
SELECT 
    SUM(TOTAL_ORDERS) AS G
FROM
    (SELECT 
        DATE, COUNT(ORDER_ID) AS TOTAL_ORDERS
    FROM
        ORDERS
    WHERE
        STR_TO_DATE(DATE, '%Y-%m-%d') BETWEEN '2015-12-01' AND '2015-12-30'
    GROUP BY 1) X;
```
### CS8 : MONTHLY PIZZA SALES
```sql
 WITH CT AS (SELECT 
    DATE, COUNT(ORDER_ID) AS SALES
FROM
    ORDERS
GROUP BY 1)
SELECT 
    EXTRACT(MONTH FROM DATE) AS MONTH, SUM(SALES) AS TOTAL_SALES
FROM
    CT
GROUP BY MONTH
ORDER BY 2 DESC;
 
 WITH CTE AS (
    SELECT 
    STR_TO_DATE(date, '%Y-%m-%d') AS DAYS, ORDER_ID, TIME
FROM
    ORDERS) 
	SELECT 
    EXTRACT(MONTH FROM DAYS) AS MONTH,
    COUNT(ORDER_ID) AS TOTAL_ORDERS
FROM
    CTE
GROUP BY 1
ORDER BY 2 DESC;
```
### CS9 : ORDER BY TIME OF A DAY (MORNING, AFTERNOON, EVENING)
```sql
WITH cte AS
(SELECT 
	Hour(STR_TO_DATE(TIME, '%H:%i:%s')) as Day_Time, ORDER_ID  FROM ORDERS)
SELECT 
CASE
	 WHEN Day_Time BETWEEN 0 AND 11 THEN 'MORNING'
	 WHEN Day_Time BETWEEN 12 AND 17 THEN 'AFTERNOON'
	 WHEN Day_Time BETWEEN 18 AND 23 THEN 'EVENING'
 END AS DAY, COUNT(ORDER_ID) AS TOTAL_ORDERS
FROM cte
GROUP BY DAY;
```
### CS10: IDENTIFY THE HIGHEST-PRICED PIZZA.
```sql
SELECT 
    PT.NAME, P.PRICE
FROM
    PIZZA_TYPES AS PT
        LEFT JOIN
    PIZZAS AS P ON PT.PIZZA_TYPE_ID = P.PIZZA_TYPE_ID
ORDER BY P.PRICE DESC
LIMIT 1;
```
### CS11: IDENTIFY THE MOST COMMON PIZZA SIZE ORDERED.
```sql
SELECT 
    P.SIZE, COUNT(*) AS PIZZA_ORDERED
FROM
    PIZZAS AS P
        LEFT JOIN
    order_details AS OD ON P.PIZZA_ID = OD.PIZZA_ID
GROUP BY SIZE
ORDER BY 2 DESC;
```
### CS12: LIST THE TOP 5 MOST ORDERED PIZZA TYPES ALONG WITH THEIR QUANTITIES.
```sql
SELECT 
    PT.NAME, SUM(QUANTITY) AS ORDERED_QUANTITY
FROM
    PIZZA_TYPES AS PT
        LEFT JOIN
    PIZZAS AS P ON PT.PIZZA_TYPE_ID = P.PIZZA_TYPE_ID
        LEFT JOIN
    order_details AS OD ON P.PIZZA_ID = OD.PIZZA_ID
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
```    
 ### CS13: Join the necessary tables to find the total quantity of each pizza category ordered.
 ```sql
SELECT 
    PT.CATEGORY, SUM(QUANTITY) AS TOTAL_QUANTITY_ORDERED
FROM
    PIZZA_TYPES AS PT
        LEFT JOIN
    PIZZAS AS P ON PT.PIZZA_TYPE_ID = P.PIZZA_TYPE_ID
        LEFT JOIN
    order_details AS OD ON P.PIZZA_ID = OD.PIZZA_ID
GROUP BY 1
ORDER BY 2 DESC;
```
### CS14: Determine the distribution of orders by hour of the day
```sql
SELECT 
    HOUR(TIME) AS HOURS, COUNT(ORDER_ID) AS ORDERS
FROM
    ORDERS
GROUP BY 1
ORDER BY 1 ASC;
```
### CS15: Determine the top 3 most ordered pizza types based on revenue.
```sql
SELECT 
    PT.NAME, SUM(QUANTITY * PRICE) AS REVENUE
FROM
    PIZZA_TYPES AS PT
        LEFT JOIN
    PIZZAS AS P ON PT.PIZZA_TYPE_ID = P.PIZZA_TYPE_ID
        LEFT JOIN
    order_details AS OD ON P.PIZZA_ID = OD.PIZZA_ID
GROUP BY 1
ORDER BY 2 DESC
LIMIT 3;
```
### CS16: Calculate the percentage contribution of each pizza type to total revenue.
```sql
SELECT 
PT.CATEGORY, 
CONCAT(ROUND((SUM(OD.QUANTITY*P.PRICE))/
(SELECT 
   SUM(OD.QUANTITY*P.PRICE) AS TOTAL_REVENUE 
FROM
    PIZZA_TYPES AS PT
        LEFT JOIN
    PIZZAS AS P ON PT.PIZZA_TYPE_ID = P.PIZZA_TYPE_ID
        LEFT JOIN
    order_details AS OD ON P.PIZZA_ID = OD.PIZZA_ID)*100,2),'%') AS PERCENTAGE
 FROM
    PIZZA_TYPES AS PT
        LEFT JOIN
    PIZZAS AS P ON PT.PIZZA_TYPE_ID = P.PIZZA_TYPE_ID
        LEFT JOIN
    order_details AS OD ON P.PIZZA_ID = OD.PIZZA_ID
    GROUP BY 1
    ORDER BY PERCENTAGE DESC;
```    
### CS17: Analyze the cumulative revenue generated over time.
```sql
SELECT 
    O.DATE, ROUND(SUM(QUANTITY * PRICE), 2) AS REV
FROM
    PIZZAS AS P
        LEFT JOIN
    order_details AS OD ON P.PIZZA_ID = OD.PIZZA_ID
        LEFT JOIN
    ORDERS AS O ON O.ORDER_ID = OD.ORDER_ID
GROUP BY 1
ORDER BY 1;
-- 
SELECT
 DATE,REV, SUM(REV) OVER(ORDER BY DATE) AS CUM_DATE
 FROM
(SELECT 
    O.DATE, ROUND(SUM(QUANTITY * PRICE), 2) AS REV
FROM
    PIZZAS AS P
        LEFT JOIN
    order_details AS OD ON P.PIZZA_ID = OD.PIZZA_ID
        LEFT JOIN
    ORDERS AS O ON O.ORDER_ID = OD.ORDER_ID
GROUP BY 1)T 
    WHERE DATE IS NOT NULL;
 ```   
  ### CS18: Determine the top 3 most ordered pizza types based on revenue for each pizza category.
 ```sql
 SELECT CATEGORY ,NAME, REVENUE, RNK 
 FROM
 (SELECT  
	 CATEGORY,NAME,REVENUE,
	 RANK() OVER (PARTITION BY CATEGORY ORDER BY REVENUE DESC) AS RNK
 FROM
 (SELECT 
    PT.CATEGORY,
    PT.NAME,
    ROUND(SUM(P.PRICE * OD.QUANTITY), 2) AS REVENUE
FROM
    PIZZA_TYPES AS PT
        LEFT JOIN
    PIZZAS AS P ON PT.PIZZA_TYPE_ID = P.PIZZA_TYPE_ID
        LEFT JOIN
    order_details AS OD ON P.PIZZA_ID = OD.PIZZA_ID
GROUP BY 1 , 2)X) AS R
    WHERE RNK<=3
  ```  
