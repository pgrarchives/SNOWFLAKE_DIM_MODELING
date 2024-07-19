
# Snowflake Dimensional Modeling Project

This project demonstrates the process of setting up a dimensional model in Snowflake, starting from data ingestion from an S3 bucket to creating and linking various dimension and fact tables. 

## Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Project Steps](#project-steps)
    - [Step 1: Create a Stage](#step-1-create-a-stage)
    - [Step 2: Create a File Format](#step-2-create-a-file-format)
    - [Step 3: Create Dimension Tables](#step-3-create-dimension-tables)
    - [Step 4: Build Dimension Tables](#step-4-build-dimension-tables)
    - [Step 5: Build Fact Table](#step-5-build-fact-table)
    - [Step 6: Querying the Data](#step-7-querying-the-data)
4. [Schema Diagram](#schema-diagram)
5. [Conclusion](#conclusion)

## Introduction
This project involves creating a dimensional model in Snowflake, a cloud-based data warehousing service. The model includes various dimension tables and a fact table, which are linked together to form a star schema. 

## Prerequisites
- Snowflake account
- S3 bucket with the necessary CSV files
![Screenshot (28)](https://github.com/user-attachments/assets/397b8ee2-38bd-4a13-9037-220e4adc2ff9)

- Basic understanding of SQL and data warehousing concepts

## Project Steps

### Step 1: Create a Stage
First, create a stage in Snowflake linked to your S3 bucket to facilitate data loading.

```sql
CREATE STAGE my_stage 
URL='s3://mybucket/'
CREDENTIALS=(AWS_KEY_ID='your_key_id' AWS_SECRET_KEY='your_secret_key');
```
### Step 2: Create a File Format
Next, define a file format for the CSV files stored in your S3 bucket.

```sql
CREATE FILE FORMAT my_csv_format 
TYPE = 'CSV' 
FIELD_DELIMITER = ',' 
SKIP_HEADER = 1;
FIELD_OPTIONALLY_ENCLOSED_BY='"'; -- If the CSV file has a header row, skip it
```
![Screenshot 2024-07-13 163316](https://github.com/user-attachments/assets/1421b219-9d83-48c7-976d-98c09d452047)

### Step 3: Create Tables in Snowflake
```sql
Aisles Table:
CREATE TABLE aisles (
        aisle_id INTEGER PRIMARY KEY,
        aisle VARCHAR
    );
COPY INTO aisles (aisle_id, aisle)
FROM @my_stage/aisles.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');

Departments Table:
CREATE TABLE departments (
        department_id INTEGER PRIMARY KEY,
        department VARCHAR
    );
COPY INTO departments (department_id, department)
FROM @my_stage/departments.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');

Products Table:
CREATE OR REPLACE TABLE products (
        product_id INTEGER PRIMARY KEY,
        product_name VARCHAR,
        aisle_id INTEGER,
        department_id INTEGER
    );
COPY INTO products (product_id, product_name, aisle_id, department_id)
FROM @my_stage/products.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');

Orders Table:
CREATE OR REPLACE TABLE orders (
        order_id INTEGER PRIMARY KEY,
        user_id INTEGER,
        eval_set STRING,
        order_number INTEGER,
        order_dow INTEGER,
        order_hour_of_day INTEGER,
        days_since_prior_order INTEGER
    );
COPY INTO orders (order_id, user_id, eval_set, order_number, order_dow, order_hour_of_day, days_since_prior_order)
FROM @my_stage/orders.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');

Order Products Table:
CREATE OR REPLACE TABLE order_products (
        order_id INTEGER,
        product_id INTEGER,
        add_to_cart_order INTEGER,
        reordered INTEGER,
        PRIMARY KEY (order_id, product_id)
    );
COPY INTO order_products (order_id, product_id, add_to_cart_order, reordered)
FROM @my_stage/order_products.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');
```

### Step 4: Build Dimension Tables
Create the dimension tables as per your schema design.

```sql
CREATE OR REPLACE TABLE dim_users AS (
  SELECT
    user_id
  FROM
    orders
);

CREATE OR REPLACE TABLE dim_products AS (
  SELECT
    product_id,
    product_name
  FROM
    products
);

CREATE OR REPLACE TABLE dim_aisles AS (
  SELECT
    aisle_id,
    aisle
  FROM
    aisles
);

CREATE OR REPLACE TABLE dim_departments AS (
  SELECT
    department_id,
    department
  FROM
    departments
);

CREATE OR REPLACE TABLE dim_orders AS (
  SELECT
    order_id,
    order_number,
    order_dow,
    order_hour_of_day,
    days_since_prior_order
  FROM
    orders
);
```

### Step 5: Build Fact Table
Create the fact table as per your schema design.

```sql
CREATE TABLE fact_order_products AS (
  SELECT
    op.order_id,
    op.product_id,
    o.user_id,
    p.department_id,
    p.aisle_id,
    op.add_to_cart_order,
    op.reordered
  FROM
    order_products op
  JOIN
    orders o ON op.order_id = o.order_id
  JOIN
    products p ON op.product_id = p.product_id
);
```
![Screenshot 2024-07-13 163635](https://github.com/user-attachments/assets/5c77a0a9-d74e-4221-ad42-16233c931b80)

### Step 6: Querying the Data
Finally, query the data for analytics.

```sql
-- Query to calculate the total number of products ordered per department:
SELECT
  d.department,
  COUNT(*) AS total_products_ordered
FROM
  fact_order_products fop
JOIN
  dim_departments d ON fop.department_id = d.department_id
GROUP BY
  d.department;

-- Query to find the top 5 aisles with the highest number of reordered products:
SELECT
  a.aisle,
  COUNT(*) AS total_reordered
FROM
  fact_order_products fop
JOIN
  dim_aisles a ON fop.aisle_id = a.aisle_id
WHERE
  fop.reordered = TRUE
GROUP BY
  a.aisle
ORDER BY
  total_reordered DESC
LIMIT 5;

-- Query to calculate the average number of products added to the cart per order by day of the week:
SELECT
  o.order_dow,
  AVG(fop.add_to_cart_order) AS avg_products_per_order
FROM
  fact_order_products fop
JOIN
  dim_orders o ON fop.order_id = o.order_id
GROUP BY
  o.order_dow;

-- Query to identify the top 10 users with the highest number of unique products ordered:
SELECT
  u.user_id,
  COUNT(DISTINCT fop.product_id) AS unique_products_ordered
FROM
  fact_order_products fop
JOIN
  dim_users u ON fop.user_id = u.user_id
GROUP BY
  u.user_id
ORDER BY
  unique_products_ordered DESC
LIMIT 10;
```

## Schema Diagram
Below is the schema diagram illustrating the relationship between the fact table and the dimension tables:
![Screenshot 2024-07-19 125619](https://github.com/user-attachments/assets/3b8da981-3d0a-4124-ae40-e203d5f1bc53)


## Conclusion
This project demonstrates how to set up a dimensional model in Snowflake, starting from data ingestion from an S3 bucket to creating and linking various dimension and fact tables. This setup allows for efficient querying and data analysis.
