# SQL Data Analysis for A Food Delivery Company

## Table of Contents
- [Project Overview](#project-overview)
- [Results/ Findings](#results-findings)
- 
### Project Overview
This data analysis project focuses on extracting insights from a food delivery company’s dataset. It involves database setup, data import, handling missing values, and addressing key business challenges through advanced SQL queries.


### Data Sources
Sales Data: The primary dataset used for this analysis is the "car_sales.csv" file, which contains detailed information about each car sold.

### Tools
- PostgreSQL - Data Cleaning, Data Analysis, Creating reports

### Data Cleaning/Preparation
In the initial data preparation phase, the following tasks were performed:
1. Database setup and data import.
2. Data cleaning.
   
![Image 1](https://github.com/user-attachments/assets/a3dbfab8-fd25-4b6e-8763-f5d4c99085a7)


### 1. Database Setup
```sql
CREATE DATABASE zomato_db;
```

### 2. Dropping Existing Tables
```sql
DROP TABLE IF EXISTS deliveries;
DROP TABLE IF EXISTS Orders;
DROP TABLE IF EXISTS customers;
DROP TABLE IF EXISTS restaurants;
DROP TABLE IF EXISTS riders;

-- 2. Creating Tables
CREATE TABLE restaurants (
    restaurant_id SERIAL PRIMARY KEY,
    restaurant_name VARCHAR(100) NOT NULL,
    city VARCHAR(50),
    opening_hours VARCHAR(50)
);

CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    reg_date DATE
);

CREATE TABLE riders (
    rider_id SERIAL PRIMARY KEY,
    rider_name VARCHAR(100) NOT NULL,
    sign_up DATE
);

CREATE TABLE Orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    restaurant_id INT,
    order_item VARCHAR(255),
    order_date DATE NOT NULL,
    order_time TIME NOT NULL,
    order_status VARCHAR(20) DEFAULT 'Pending',
    total_amount DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    FOREIGN KEY (restaurant_id) REFERENCES restaurants(restaurant_id)
);

CREATE TABLE deliveries (
    delivery_id SERIAL PRIMARY KEY,
    order_id INT,
    delivery_status VARCHAR(20) DEFAULT 'Pending',
    delivery_time TIME,
    rider_id INT,
    FOREIGN KEY (order_id) REFERENCES Orders(order_id),
    FOREIGN KEY (rider_id) REFERENCES riders(rider_id)
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
### Results/ Findings
The analysis results were summarized as follows:
1. Total sales have grown by 23.59% compared to the previous year.
2. The average selling price has decreased by 0.79%, reflecting a $0.22K reduction from last year.
3. Car sales volume has increased by 2.62K units, marking a 24.57% rise over the previous year.
4. SUVs are the highly preferred body style, while pale white remains the most popular car color.
5. Chevrolet had the highest sales among all car brands
6. The Austin dealership recorded the highest sales, whereas Middletown had the lowest performance. 

### Recommendations
Based on the analysis, the following actions are recommended:
- Increase inventory for high-demand SUV models and pale white color to meet customer preferences and boost sales.
- Introduce targeted promotions and special incentives to improve sales performance in Middletown.
- Enhance pricing strategies by offering value-added services (e.g., free maintenance, extended warranties) instead of reducing prices to maintain profitability.
- Leverage Chevrolet’s popularity by strengthening sales efforts through discounts, financing options, and promotional campaigns.
- Align dealership stock with regional demand trends, ensuring more inventory is available in high-demand areas.
