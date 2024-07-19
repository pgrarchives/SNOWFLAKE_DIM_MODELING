
# Snowflake Dimensional Modeling Project

Welcome to the Snowflake Dimensional Modeling Project! This project demonstrates the process of setting up a dimensional model in Snowflake, starting from data ingestion from an S3 bucket to creating and linking various dimension and fact tables. 

## Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Project Steps](#project-steps)
    - [Step 1: Create a Stage](#step-1-create-a-stage)
    - [Step 2: Create a File Format](#step-2-create-a-file-format)
    - [Step 3: Create Dimension Tables](#step-3-create-dimension-tables)
    - [Step 4: Load Data into Dimension Tables](#step-4-load-data-into-dimension-tables)
    - [Step 5: Create the Fact Table](#step-5-create-the-fact-table)
    - [Step 6: Load Data into the Fact Table](#step-6-load-data-into-the-fact-table)
    - [Step 7: Querying the Data](#step-7-querying-the-data)
4. [Schema Diagram](#schema-diagram)
5. [Conclusion](#conclusion)

## Introduction
This project involves creating a dimensional model in Snowflake, a cloud-based data warehousing service. The model includes various dimension tables and a fact table, which are linked together to form a star schema. 

## Prerequisites
- Snowflake account
- S3 bucket with the necessary CSV files
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
```

### Step 3: Create Dimension Tables
Create the dimension tables as per your schema design.

```sql
CREATE TABLE dim_orders (
    order_id INTEGER,
    order_number INTEGER,
    order_dow INTEGER,
    order_hour_of_day INTEGER,
    days_since_prior_order INTEGER
    -- add any other columns if necessary
);

CREATE TABLE dim_products (
    product_id INTEGER,
    product_name STRING
    -- add any other columns if necessary
);

CREATE TABLE dim_user (
    user_id INTEGER,
    -- add any other columns if necessary
);

CREATE TABLE dim_department (
    department_id INTEGER,
    department STRING
    -- add any other columns if necessary
);

CREATE TABLE dim_aisles (
    aisle_id INTEGER,
    aisle STRING
    -- add any other columns if necessary
);
```

### Step 4: Load Data into Dimension Tables
Load data from the stage into the dimension tables.

```sql
COPY INTO dim_orders
FROM @my_stage/orders.csv
FILE_FORMAT = (FORMAT_NAME = 'my_csv_format');

COPY INTO dim_products
FROM @my_stage/products.csv
FILE_FORMAT = (FORMAT_NAME = 'my_csv_format');

COPY INTO dim_user
FROM @my_stage/users.csv
FILE_FORMAT = (FORMAT_NAME = 'my_csv_format');

COPY INTO dim_department
FROM @my_stage/departments.csv
FILE_FORMAT = (FORMAT_NAME = 'my_csv_format');

COPY INTO dim_aisles
FROM @my_stage/aisles.csv
FILE_FORMAT = (FORMAT_NAME = 'my_csv_format');
```

### Step 5: Create the Fact Table
Create the fact table that links all dimension tables.

```sql
CREATE TABLE fact_order_products (
    order_id INTEGER,
    product_id INTEGER,
    user_id INTEGER,
    aisle_id INTEGER,
    department_id INTEGER,
    add_to_cart_order INTEGER,
    reordered BOOLEAN
    -- add any other columns if necessary
);
```

### Step 6: Load Data into the Fact Table
Load data from the stage into the fact table.

```sql
COPY INTO fact_order_products
FROM @my_stage/order_products.csv
FILE_FORMAT = (FORMAT_NAME = 'my_csv_format');
```

### Step 7: Querying the Data
Finally, query the data to verify the setup.

```sql
SELECT * FROM fact_order_products
JOIN dim_orders ON fact_order_products.order_id = dim_orders.order_id
JOIN dim_products ON fact_order_products.product_id = dim_products.product_id
JOIN dim_user ON fact_order_products.user_id = dim_user.user_id
JOIN dim_department ON fact_order_products.department_id = dim_department.department_id
JOIN dim_aisles ON fact_order_products.aisle_id = dim_aisles.aisle_id;
```

## Schema Diagram
Below is the schema diagram illustrating the relationship between the fact table and the dimension tables:

![Schema Diagram](path/to/your/schema_diagram.png)

## Conclusion
This project demonstrates how to set up a dimensional model in Snowflake, starting from data ingestion from an S3 bucket to creating and linking various dimension and fact tables. This setup allows for efficient querying and data analysis.
