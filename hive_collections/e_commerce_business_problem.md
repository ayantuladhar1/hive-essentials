# Solution Statement
In this scenario we would first create a optimization strategy as without optimization queries scan the entire table, as data grows performance degrades and cost increases due to compute time. As our analytics team frequently runs queries with respect to millions of sales records a day. 

# Optimization Statement

|Requirement|	Hive Feature Used|	Reason|
|-----------|------------------|--------|
|Filter by date|	Partitioning|	Avoid full table scans|
|Filter by country/region|	Partitioning	|Query only relevant partitions|
|Faster joins & aggregations|	Bucketing|	Reduces shuffle & improves joins|
|Product-level analytics|	Bucketing on product_id|	Even data distribution|

# Hive Table Design
Partitioning Strategy:
* Partition by: sale_date, country
* As Most reports are date-based and Country is a frequent filter
Bucketing Strategy:
* Bucket by: product_id
* Buckets: 8
* As it  Improves GROUP BY, JOIN queries
# Step 1:
	• Create a file "sales_data.csv" in HDFS and save it under path '/data/sales_data.csv'
	• It will help Hive to load the file and save it under the table later
# Step 2:
	• We need to create 2 tables in Hive, first will be sales_data_staging and second will be sales_data
	• In Hive when we create a table and store as ORC or Parquet, it expects the data to be in ORC or Parquet format, which makes it inconvenience for us to directly load the data into the table as our data is in csv format.
	• We will use the staging table to load the data as csv
  ```sql
	CREATE TABLE sales_data_staging (
	    sale_id INT,
	    product_id INT,
	    product_category STRING,
	    customer_id INT,
	    sale_amount FLOAT,
	    sale_date STRING,
	    country STRING,
	    region STRING )
	ROW FORMAT DELIMITED
	FIELDS TERMINATED BY ','
	STORED AS TEXTFILE;
```
Step 3:
	•  Load the Data in Hive Table sales_data_staging.
	LOAD DATA INPATH '/data/sales_data.csv' INTO TABLE sales_data_staging; 
	

Step 4:
	• We will create sales_data where we will Partition sale_date, country and bucket product_id, stored as Parquet
	CREATE TABLE sales_data (
	    sale_id INT,
	    product_id INT,
	    product_category STRING,
	    customer_id INT,
	    sale_amount FLOAT,
	    region STRING
	)
	PARTITIONED BY (
	    sale_date DATE,
	    country STRING
	)
	CLUSTERED BY (product_id)
	INTO 8 BUCKETS
	STORED AS PARQUET;
	
Step 5:
	• Set the following properties in Hive for dynamic partition
	SET hive.exec.dynamic.partition = true;
	SET hive.exec.dynamic.partition.mode = nonstrict;
	
Step 6:
	• We will insert the data we loaded earlier from sales_data_staging table to sales_data table.
	INSERT INTO TABLE sales_data
	PARTITION (sale_date, country)
	SELECT
	    sale_id,
	    product_id,
	    product_category,
	    customer_id,
	    sale_amount,
	    region,
	    sale_date,
	    country
	FROM sales_data_staging;
	
	

Step 7:
	• We will verify the Partition.
	SHOW PARTITIONS sales_data;
	

Step 8:
	• Test Query
	SELECT SUM(sale_amount)
	FROM sales_data
	WHERE sale_date = '2023-08-03'
	AND country = 'IN';
	
	
	
<img width="950" height="3589" alt="image" src="https://github.com/user-attachments/assets/732721fd-59ea-4215-9f4c-731cf5058c3e" />
