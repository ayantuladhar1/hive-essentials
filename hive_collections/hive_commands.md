# Starting Hive CLI
```sql
hive
```
* Using Beeline
```sql
beeline
```
* Beeline is preferred in production because it connects to HiveServer2 securely.

# Database Commands (DDL)
* Create Database:
```sql
CREATE DATABASE retail_db;
```
* Create Database (if not exits):
```sql
CREATE DATABASE IF NOT EXISTS retail_db;
```
* Use Database:
```sql
USE retail_db;
```
* Show Databases:
```sql
SHOW DATABASES;
```
* Drop Database:
```sql
DROP DATABASE retail_db;
```
* Note: Use CASCADE if table exists.

# Table Commands (DDL)
```sql
Create Managed Table:
CREATE TABLE customers (
  customer_id INT,
  name STRING,
  country STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ',';
```
* Data stored in Hive warehouse directory
* Dropping table deletes data

* Create External Table (Best Practice):
```sql
CREATE EXTERNAL TABLE customers_ext (
  customer_id INT,
  name STRING,
  country STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LOCATION '/data/customers';
```
* Hive manages metadata only
* Data stays in HDFS

* Show Tables:
```sql
SHOW TABLES;
```
* Describe Table:
```sql
DESCRIBE customers;
```
* Detailed Metadata:
```sql
DESCRIBE FORMATTED customers;
```
* Drop Table:
```sql
DROP TABLE customers;
```
# Load Data in Hive:
* Load Data into Hive:
```sql
LOAD DATA INPATH '/data/customers.csv'
INTO TABLE customers;
```
* Overwrite Existing Data:
```sql
LOAD DATA INPATH '/data/customers.csv'
OVERWRITE INTO TABLE customers;
```
* LOAD DATA moves the file
* External Tables do not require load

# Select Queries (DML)
* Basic Select:
```sql
SELECT * FROM customers;
```
* Select Specific Columns:
```sql
SELECT name, country FROM customers;
```
* Where Clause:
```sql
SELECT * FROM customers
WHERE country = 'USA';
```
# Aggregations
```sql
SELECT country, COUNT(*) AS total_customers
FROM customers
GROUP BY country;
```
# Sorting and Limiting
* Order By (global sort):
```sql
SELECT * FROM customers
ORDER BY name;
```
* Sort By (per reducer):
```sql
SELECT * FROM customers
SORT BY name;
```
* Limit:
```sql
SELECT * FROM customers
LIMIT 10;
```
# Partitioning Commands
* Create Partitioned Table:
```sql
CREATE TABLE sales (
  order_id INT,
  amount DOUBLE
)
PARTITIONED BY (sale_date STRING);
```
* Static Partition Insert:
```sql
INSERT INTO TABLE sales
PARTITION (sale_date='2025-01-01')
SELECT order_id, amount FROM staging_sales;
```
* Dynamic Partition Insert:
```sql
SET hive.exec.dynamic.partition=true;
SET hive.exec.dynamic.partition.mode=nonstrict;
INSERT INTO TABLE sales
PARTITION (sale_date)
SELECT order_id, amount, sale_date
FROM staging_sales;
```
# Bucketing Commands
```sql
CREATE TABLE users (
  id INT,
  name STRING
)
CLUSTERED BY (id)
INTO 4 BUCKETS
STORED AS PARQUET;
```
* Bucketing improves join performance

# File Formats in Hive
* Parquet Table:
```sql
CREATE TABLE sales_parquet (
  order_id INT,
  amount DOUBLE
)
STORED AS PARQUET;
```
* ORC Table:
```sql
CREATE TABLE sales_orc (
  order_id INT,
  amount DOUBLE
)
STORED AS ORC;
```
# Insert Data Between Tables
```sql
INSERT INTO TABLE sales_parquet
SELECT * FROM sales;
```
# JOINS in Hive
```sql
SELECT o.order_id, c.name
FROM orders o
JOIN customers c
ON o.customer_id = c.customer_id;
```
# Hive Functions
* String Functions:
```sql
SELECT UPPER(name), LENGTH(name) FROM customers;
```
* Date Functions:
```sql
SELECT current_date;
```
* Aggregate Functions:
```sql
SELECT MAX(amount), MIN(amount) FROM sales;
```
# Common Optimization Commands
```sql
SET hive.exec.parallel=true;
SET hive.cbo.enable=true;
SET hive.compute.query.using.stats=true;
```
* Helps query planner choose better execution plans

# Useful Hive Settings
```sql
SET hive.execution.engine=spark;
```
* Check current settings:
```sql
SET;
```
