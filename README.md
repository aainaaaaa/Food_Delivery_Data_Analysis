# SQL Data Analysis for A Food Delivery Company

## Table of Contents
- [Project Overview](#project-overview)
- [Analysis & Reports](#analysis--reports)
  
### Project Overview
This data analysis project focuses on extracting insights from a food delivery company’s dataset. It involves database setup, data import, handling missing values, and addressing key business challenges through advanced SQL queries.


### Data Sources
The dataset used for this analysis are:
- `customers.csv` – Contains information about each customer.
- `restaurants.csv` – Stores details about the restaurants.
- `orders.csv` – Records detailed information on customer orders.
- `riders.csv` – Includes key details about the riders.
- `deliveries.csv` – Captures information about each delivery.

### Tools
- PostgreSQL - Data Cleaning, Data Analysis

### Data Cleaning/Preparation
In the initial data preparation phase, the following tasks were performed:
1. Database setup
2. Dropping & Creating tables
3. Data import
4. Data Cleaning
   
![Image 1](https://github.com/user-attachments/assets/a3dbfab8-fd25-4b6e-8763-f5d4c99085a7)


### 1. Database Setup
```sql
CREATE DATABASE zomato_db;
```

### 2. Dropping Existing & Creating Tables
```sql
DROP TABLE IF EXISTS deliveries;
DROP TABLE IF EXISTS Orders;
DROP TABLE IF EXISTS customers;
DROP TABLE IF EXISTS restaurants;
DROP TABLE IF EXISTS riders;

-- Creating Tables
CREATE TABLE customers
	(
	customer_id INT PRIMARY KEY,
	customer_name VARCHAR(25),
	reg_date DATE
	);

CREATE TABLE restaurants
	(
	restaurant_id INT PRIMARY KEY,
	restaurant_name VARCHAR(55),
	city VARCHAR(15),
	opening_hours VARCHAR(55)
	);

CREATE TABLE orders
	(
	order_id INT PRIMARY KEY,
	customer_id INT, -- coming from customers table
	restaurant_id INT, -- coming from restaurants table
	order_item VARCHAR(55),
	order_date DATE,
	order_time TIME,
	order_status VARCHAR(55),
	total_amount FLOAT
	);

-- adding FK CONSTRAINT
ALTER TABLE orders
ADD CONSTRAINT fk_customers
FOREIGN KEY (customer_id)
REFERENCES customers(customer_id);

-- adding FK CONSTRAINT
ALTER TABLE orders
ADD CONSTRAINT fk_restaurants
FOREIGN KEY (restaurant_id)
REFERENCES restaurants(restaurant_id);


CREATE TABLE riders
	(
	rider_id INT PRIMARY KEY,
	rider_name VARCHAR (55),
	sign_up DATE
	);

CREATE TABLE deliveries
	(
	delivery_id INT PRIMARY KEY,
	order_id INT, -- coming from the orders table
	delivery_status	VARCHAR(35),
	delivery_time TIME,
	rider_id INT, -- coming from the riders table
	CONSTRAINT fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id),
	CONSTRAINT fk_riders FOREIGN KEY (rider_id) REFERENCES riders(rider_id)
	);
```

### 3. Data Import
Importing datasets in the following order (*refer to ERD diagram*)
- `customers.csv`
- `restaurants.csv`
- `orders.csv`
- `riders.csv`
- `deliveries.csv`

### 4. Data Cleaning and Handling Null Values
```sql
-- Checking for null value
SELECT COUNT(*)
FROM customers
WHERE
	customer_name IS NULL
	OR
	reg_date IS NULL

SELECT COUNT(*)
FROM restaurants 
WHERE
	restaurant_name IS NULL
	OR
	city IS NULL
	OR
	opening_hours IS NULL

SELECT COUNT(*)
FROM orders 
WHERE
	order_item IS NULL
	OR
	order_date IS NULL
	OR
	order_time IS NULL
	OR
	order_status IS NULL
	OR
	total_amount IS NULL

SELECT COUNT(*)
FROM riders 
WHERE
	rider_name IS NULL
	OR
	sign_up IS NULL

SELECT COUNT(*)
FROM deliveries 
WHERE
	delivery_status IS NULL
	OR
	delivery_time IS NULL

-- Deleting null values
DELETE
FROM orders 
WHERE
	order_item IS NULL
	OR
	order_date IS NULL
	OR
	order_time IS NULL
	OR
	order_status IS NULL
	OR
	total_amount IS NULL
```
### Analysis & Reports
```sql
-- Q.1
-- The top 5 most frequently ordered dishes by a customer called "Arjun Mehta" in the last 2 year.

SELECT 
	dishes, 
	total_order
FROM
(SELECT 
	c.customer_name,
	o.order_item as dishes, 
	COUNT(*) as total_order,
	DENSE_RANK() OVER(ORDER BY COUNT(*) DESC) as rank
FROM orders as o
JOIN customers as c  
ON c.customer_id = o.customer_id
WHERE 
	o.order_date >= CURRENT_DATE - INTERVAL '2 Year'
	AND
	c.customer_name = 'Arjun Mehta'
GROUP BY 1,2
ORDER BY 3 DESC) as t1
WHERE rank <= 5


-- 2. Popular Time Slots
-- The time slots during which the most orders are placed. based on 2-hour intervals.

-- METHOD 1
--assume 00:59:59AM = 0 to 01:59:59AM =1 is a 2-hour interval

SELECT 
	CASE
		WHEN EXTRACT (HOUR FROM order_time) BETWEEN 0 AND 1 THEN '00:00 - 02:00'
		WHEN EXTRACT (HOUR FROM order_time) BETWEEN 2 AND 3 THEN '02:00 - 04:00'
		WHEN EXTRACT (HOUR FROM order_time) BETWEEN 4 AND 5 THEN '04:00 - 06:00'
		WHEN EXTRACT (HOUR FROM order_time) BETWEEN 6 AND 7 THEN '06:00 - 08:00'
		WHEN EXTRACT (HOUR FROM order_time) BETWEEN 8 AND 9 THEN '08:00 - 10:00'
		WHEN EXTRACT (HOUR FROM order_time) BETWEEN 10 AND 11 THEN '10:00 - 12:00'
		WHEN EXTRACT (HOUR FROM order_time) BETWEEN 12 AND 13 THEN '12:00 - 14:00'
		WHEN EXTRACT (HOUR FROM order_time) BETWEEN 14 AND 15 THEN '14:00 - 16:00'
		WHEN EXTRACT (HOUR FROM order_time) BETWEEN 16 AND 17 THEN '16:00 - 18:00'
		WHEN EXTRACT (HOUR FROM order_time) BETWEEN 18 AND 19 THEN '18:00 - 20:00'
		WHEN EXTRACT (HOUR FROM order_time) BETWEEN 20 AND 21 THEN '20:00 - 22:00'
		WHEN EXTRACT (HOUR FROM order_time) BETWEEN 22 AND 23 THEN '22:00 - 00:00'
		END as time_slot,
		COUNT(order_id) as number_of_order
FROM orders
GROUP BY 1
ORDER BY 2 DESC

-- METHOD 2
-- if extracting hour from 23:55PM/2 = 11 (FLOOR), then 11*2 = 22 --> start_time
-- if extracting hour from 23:55PM/2 = 11 (FLOOR), then 11*2+2 = 24 --> end_time

SELECT
	FLOOR(EXTRACT(HOUR FROM order_time)/2)*2 as start_time,
	FLOOR(EXTRACT(HOUR FROM order_time)/2)*2 + 2 as end_time,
	COUNT(*) as frequency_of_order
FROM orders
GROUP BY 1, 2
ORDER BY 3 DESC

-- 3. Order Value Analysis
-- The average order value per customer that placed more than 750 orders.
SELECT 
	c.customer_name, 
	AVG(o.total_amount) as average_order_value, 
	COUNT(o.order_id) as frequency_of_order
FROM customers as c
	JOIN orders as o
	ON c.customer_id = o.customer_id
GROUP BY 1
HAVING COUNT(o.order_id) > 750


-- 4. High-Value Customers
-- The customers who have spent more than 100K in total on food orders.
SELECT c.customer_name, SUM(o.total_amount) as total_food_order
FROM customers as c
JOIN orders as o
ON c.customer_id = o.customer_id
GROUP BY 1
HAVING SUM(o.total_amount) > 100000
ORDER BY 2 DESC
SELECT *
FROM deliveries

-- 5. Orders Without Delivery
-- The orders that were placed but not delivered. 
-- Return each restaurant name, city and number of orders not delivered 

-- METHOD 1
-- left joining restaurants and deliveries table to orders table 
-- filtering and counting the order_id with null delivery_id

SELECT 
	r.restaurant_name, 
	r.city, 
	COUNT(o.order_id) as order_not_delivered
FROM orders as o
LEFT JOIN restaurants as r
ON r.restaurant_id = o.restaurant_id
LEFT JOIN deliveries as d
ON d.order_id = o.order_id
WHERE d.delivery_id IS NULL
GROUP BY 1, 2
ORDER BY 3 DESC

-- METHOD 2
-- left joining restaurants table to orders table
-- selecting order_id from orders table that was not listed in order_id from deliveries table
-- tl;dr: there was 10,000 rows of order_id in orders table and 9750 rows of order_id in deliveries table
-- tl;dr: hence, order_id from orders table that was not listed in order_id from deliveries table = 10,000 - 9,750

SELECT 
	r.restaurant_name, 
	r.city, 
	COUNT(o.order_id) as order_not_delivered
FROM orders as o
LEFT JOIN restaurants as r
ON r.restaurant_id = o.restaurant_id
WHERE o.order_id NOT IN (SELECT order_id FROM deliveries)
GROUP BY 1, 2
ORDER BY 3 DESC


-- Q. 6
-- Restaurant Revenue Ranking: 
-- Based on total revenue in the past 2 yeasr and rank within their city.
WITH ranking_table
AS
(	SELECT 
		r.city,
		r.restaurant_name,
		SUM(o.total_amount) as total_revenue,
		RANK() OVER(PARTITION BY r.city ORDER BY SUM(o.total_amount) DESC)
	FROM orders as o
	JOIN restaurants as r
	ON r.restaurant_id = o.restaurant_id
	WHERE o.order_date >= CURRENT_DATE - INTERVAL '2 years'
	GROUP BY 1, 2 
)
SELECT *
FROM ranking_table
WHERE rank = 1


-- Q. 7
-- Most Popular Dish by City: 
WITH most_popular_dish
AS
(
	SELECT
		r.city,
		o.order_item as dish,
		COUNT(o.order_id) as order_frequency,
		RANK() OVER(PARTITION BY r.city ORDER BY COUNT(o.order_id) DESC)
	FROM orders as o
	JOIN restaurants as r
	ON r.restaurant_id = o.restaurant_id
	GROUP BY 1,2
)

SELECT *
FROM most_popular_dish
WHERE rank = 1


-- Q.8 Customer Churn: 
-- Find customers who haven’t placed an order in 2024 but did in 2023.
-- Find customers who placed orders in 2023 & find customers who haven't placed any orders in 2024, then compare
SELECT 
	DISTINCT o.customer_id, 
	c.customer_name
FROM orders as o
JOIN customers as c
ON c.customer_id = o.customer_id
WHERE 
	EXTRACT(YEAR FROM o.order_date) = 2023
	AND
	o.customer_id NOT IN (
						SELECT DISTINCT o.customer_id
						FROM orders as o
						WHERE EXTRACT(YEAR FROM o.order_date) = 2024
						)


-- Q.9 Cancellation Rate Comparison: 
-- Calculate and compare the order cancellation rate for each restaurant between the 2023 and 2024.
WITH 
cancellation_rate_2023
AS
(
	SELECT 
		r.restaurant_name, 
		COUNT(o.order_id) as order_frequency,
		COUNT(
			CASE WHEN d.delivery_id IS NULL THEN 1
			END
			  ) as cancellation_frequency
	FROM orders as o
	LEFT JOIN restaurants as r
	ON r.restaurant_id = o.restaurant_id
	LEFT JOIN deliveries as d
	ON d.order_id = o.order_id
	WHERE EXTRACT(YEAR FROM o.order_date) = 2023
	GROUP BY 1
),

cancellation_rate_2024
AS
(
	SELECT 
		r.restaurant_name, 
		COUNT(o.order_id) as order_frequency,
		COUNT(
			CASE WHEN d.delivery_id IS NULL THEN 1
			END
			  ) as cancellation_frequency
	FROM orders as o
	LEFT JOIN restaurants as r
	ON r.restaurant_id = o.restaurant_id
	LEFT JOIN deliveries as d
	ON d.order_id = o.order_id
	WHERE EXTRACT(YEAR FROM o.order_date) = 2024
	GROUP BY 1
),
ratio_2023
AS
(
	SELECT 
		restaurant_name, 
		order_frequency,
		cancellation_frequency,
		ROUND(cancellation_frequency::numeric/order_frequency::numeric * 100,2) as cancel_ratio
	FROM cancellation_rate_2023
),

ratio_2024
AS
(
	SELECT 
		restaurant_name, 
		order_frequency,
		cancellation_frequency,
		ROUND(cancellation_frequency::numeric/order_frequency::numeric * 100,2) as cancel_ratio
	FROM cancellation_rate_2024
)

SELECT
	ratio_2024.restaurant_name,
	ratio_2024.cancel_ratio as cancel_ratio_2024,
	ratio_2023.cancel_ratio as cancel_ratio_2023
FROM ratio_2024
JOIN ratio_2023
ON ratio_2024.restaurant_name = ratio_2023.restaurant_name


-- Q.10 Rider Average Delivery Time: 
-- Determine each rider's average delivery time.
SELECT 
	d.rider_id,
	o.order_id,
	o.order_time,
	d.delivery_time,
	d.delivery_time - o.order_time as interval,
	EXTRACT(EPOCH FROM (d.delivery_time - o.order_time +
	CASE WHEN d.delivery_time < o.order_time
	THEN INTERVAL '1 day'
	ELSE INTERVAL '0 day' END))/60 as time_taken
FROM orders as o 
JOIN deliveries as d
ON d.order_id = o.order_id
WHERE d.delivery_status = 'Delivered'
ORDER BY 1
```
