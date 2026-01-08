# Solution Statement
In this telecommunication analytics scenario, we first design an optimization strategy because, without optimization, analytical queries would scan the entire dataset, leading to performance degradation as call records, data usage logs, and SMS data grow rapidly over time. The telecom company processes millions of call, data, and messaging records daily across multiple regions and dates, and inefficient queries increase compute time and infrastructure cost.
To support frequent reporting on call duration trends, heavy data users, regional usage analysis, and customer-level insights, Hive partitioning and bucketing are applied. These optimizations enable efficient data pruning, reduce shuffle during joins, and improve query performance for large-scale telecom analytics workloads.

# Optimization Statement

|Requirement|	Hive Feature Used|	Reason|
|-----------|--------------------|--------|
|Filter call records by date|	Partitioning|	Avoid full table scans by pruning irrelevant date partitions|
|Filter usage and SMS reports by region|	Partitioning|	Query only required regional partitions|
|Identify heavy data users|	Bucketing|	Improves aggregation performance on customer-level metrics|
|Faster joins across call, data, and SMS datasets|	Bucketing on customer_id|	Reduces shuffle and optimizes join execution|
|Customer-centric telecom analytics|	Bucketing|	Ensures even data distribution and efficient GROUP BY operations|

# Hive Table Design
Partitioning Strategy:
* Call Data
  * Partition by: call_date, region
* Data Usage
  * Partition by: usage_date, region
* SMS Data
  * Partition by: sms_date, region
* Most telecom reports are time-based (daily, weekly, monthly usage) and region-based, making date and region ideal partition keys for effective partition pruning and reduced IO.

Bucketing Strategy:
* Bucket by: customer_id (across all three tables)
* Buckets: 8
* Bucketing on customer_id improves:
  * Customer-level aggregations (total call duration, total data usage, SMS counts)
  * Join performance across call, data, and SMS datasets
  * Load balancing by evenly distributing records across buckets

# Step 1:
* Create a file "call_data.csv" in HDFS and save it under path '/data/call_data.csv'
* Create a file "data_usage.csv" in HDFS and save it under path '/data/data_usage.csv'
* Create a file "sms_data.csv" in HDFS and save it under path '/data/sms_data.csv'
* It will help Hive to load the files and save it under the tables later

# Step 2:
* We need to create 6 tables in Hive, first  three will be call_data_staging, data_usage_staging, sms_data and second three will be call_data, data_usage and sms_data respectively.
* In Hive when we create a table and store as ORC or Parquet, it expects the data to be in ORC or Parquet format, which makes it inconvenience for us to directly load the data into the table as our data is in csv format.
* We will use the staging tables to load the data as csv
```sql
	CREATE TABLE call_data_staging (
	  call_id INT,
	  customer_id INT,
	  call_duration FLOAT,
	  region STRING,
	  call_date DATE
	)
	ROW FORMAT DELIMITED
	FIELDS TERMINATED BY ','
	STORED AS TEXTFILE;
	
	CREATE TABLE data_usage_staging (
	  usage_id INT,
	  customer_id INT,
	  data_used FLOAT,
	  region STRING,
	  usage_date DATE
	)
	ROW FORMAT DELIMITED
	FIELDS TERMINATED BY ','
	STORED AS TEXTFILE;

	CREATE TABLE sms_data_staging (
	  sms_id INT,
	  customer_id INT,
	  sms_count INT,
	  region STRING,
	  sms_date DATE
	)
	ROW FORMAT DELIMITED
	FIELDS TERMINATED BY ','
	STORED AS TEXTFILE;
```

# Step 3:
* Load the Data in Hive Table call_data_staging.
```sql
LOAD DATA INPATH '/data/call_data.csv' INTO TABLE call_data_staging; 
```
* Load the Data in Hive Table data_usage_staging.
```sql
LOAD DATA INPATH '/data/data_usage.csv' INTO TABLE data_usage_staging; 
```
* Load the Data in Hive Table sms_data_staging.
```sql
LOAD DATA INPATH '/data/sms_data.csv' INTO TABLE sms_data_staging; 
```

# Step 4:
* We will create call_data, data_usage and sms_data tables where we will Partition region, call_date and bucket customer_id for call_data table, Partition region, usage_date and bucket customer_id for data_usage table and Partition region, sms_date and bucket customer_id for sms_data table, where all the tables would be stored as Parquet.
```sql
	CREATE TABLE call_data (
	  call_id INT,
	  customer_id INT,
	  call_duration FLOAT
	) 
	PARTITIONED BY (
	  region STRING,
	  call_date DATE
	)
	CLUSTERED BY (customer_id)
	INTO 8 BUCKETS
	STORED AS PARQUET;
	
	CREATE TABLE data_usage (
	  usage_id INT,
	  customer_id INT,
	  data_used FLOAT
	)
	PARTITIONED BY (
	  region STRING,
	  usage_date DATE
	)
	CLUSTERED BY (customer_id)
	INTO 8 BUCKETS
	STORED AS PARQUET;
	
	CREATE TABLE sms_data (
	  sms_id INT,
	  customer_id INT,
	  sms_count INT
	)
	PARTITIONED BY (
	  region STRING,
	  sms_date DATE
	)
	CLUSTERED BY (customer_id)
	INTO 8 BUCKETS
	STORED AS PARQUET;
```

# Step 5:
* Set the following properties in Hive for dynamic partition
```sql
SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode = nonstrict;
```

# Step 6:
* We will insert the data we loaded earlier from call_data_staging table to  call_data table, data_usage_staging table tp data_usage table and  sms_data_staging table to sms_data table.
```sql
	INSERT INTO table call_data partition (region, call_date) 
	SELECT 
	  call_id, 
	  customer_id, 
	  call_duration, 
	  region, 
	  call_date 
	FROM 
	  call_data_staging;
```
	
```sql
	INSERT INTO data_usage PARTITION (region, usage_date) 
	SELECT 
	  usage_id, 
	  customer_id, 
	  data_used, 
	  region, 
	  usage_date 
	FROM 
	  data_usage_staging;
```
	
```sql
	INSERT INTO TABLE sms_data PARTION (region, sms_date) 
	SELECT 
	  sms_id, 
	  customer_id, 
	  sms_count, 
	  region, 
	  sms_date 
	FROM 
	  sms_data_staging;
```	
	
# Step 7:
* We will verify the Partition.
```sql
SHOW PARTITIONS call_data;
```
```sql
SHOW PARTITIONS data_usage;
```
```sql
SHOW PARTITIONS data_usage;
```

# Step 8:
* Test Query
```sql
SELECT * FROM call_data WHERE region = 'North' AND call_date = '2023-08-05';
```
