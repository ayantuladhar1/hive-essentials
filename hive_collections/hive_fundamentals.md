# What is Hive?
Apache Hive is a data warehouse tool built on top of Hadoop that allows you to query large datasets stored in HDFS using SQL-like language called HiveQL.
Hive converts SQL queries into distributed processing jobs that run on Hadoop execution engines such as MapReduce, Tez or Spark.
Hive does store data itself, it only manages metadata.

# Why Hive is Important?
Before Hive, working with Hadoop Required:
* Writing complex MapReduce programs
* Deep Java knowledge
* Long development cycles
Hive simplified Big Data analytics by:
* Providing SQL abstraction
* Allowing analysts to query petabyte-scala data
* Integrating seamlessly with the Hadoop ecosystem

# When to use Hive?
Hive is best suited for:
* Batch data processing
* Data warehousing
* Analytical queries
* Reporting and aggregations
Hive is not designed for:
* Real-time queries
* Transactional workloads (OLTP)
* Low-latency applications

# Hive vs Traditional Databases

Feature	RDBMS	Hive
Data size	GBs	TBs/PBs
Query latency	Low	High
Schema	Schema-on-write	Schema-on-read
Use case	Transactions	Analytics

# Hive Architecture Overview
<img width="1037" height="526" alt="image" src="https://github.com/user-attachments/assets/c48105b1-f8d9-4588-bfb0-536fad90622d" />

Hive follows a layered architecture:
Major Components:
* Client: Interfaces such as CLI or Beeline used to submit queries
* Driver: Manages query lifecycle (parsing, compiling, executing)
* Compiler: Converts HiveQL into execution plans
* Metastore: Stores metadata
* Execution Engine: Executes queries using MapReduce, Tez or Spark

# Hive Metastore (Key Component)
The Hive Metastore stores:
* Database names
* Table schemas
* Column data types
* Partition information
* File format details
* HDFS file locations

Note: Metadata is stored in an RDBMS, actual data remains in HDFS

# Schema-on-Read(Core Concept)
Hive uses schema-on-read, meaning:
* Data is stored first
* Schema is applied only when data is read
This provides:
* Flexibility in data ingestion
* Support for evolving schemas
* Faster ingestion pipelines

# Hive Data Organization  
Hive organizes data logically as:  
Database 
└── Table
      └── Partition
           └── Bucket
                └── Files
Each level helps improve query performance and manageability
# Partitioning in Hive
* It splits a table into separate directories in HDFS based on the value of one or more columns.
* Think of it as folder-level segregation of data.
* Example:
```sql
 CREATE TABLE sales (
   order_id INT,
   product STRING,
   amount DOUBLE
 )
 PARTITIONED BY (year INT);
```
* HDFS Layout
```swift
 /warehouse/sales/year=2023/
 /warehouse/sales/year=2024/
 /warehouse/sales/year=2025/
```
* Example
* When you run:
```sql
 SELECT * FROM sales WHERE year = 2024;
```
* Hive only scans:
```swift
 /year=2024/
```
* Instead of entire table

* Advantages includes:
  * Huge performance boost
  * Less data scanned
  * Faster query execution

* When to use
  * Columns with low cardinality
  * Date,year,country,region

* Avoid partitioning on
  * High-cardinality fields like user_id, order_id

# Bucketing in Hive
* Bucketing divides data into a file-level distribution inside a table or partition.
* Example
```sql
Bucket customers by customer_id into 4 buckets:
CREATE TABLE customers (
  customer_id INT,
  name STRING,
  city STRING
)
CLUSTERED BY (customer_id) INTO 4 BUCKETS;
```
* Hive calculates:
```python
hash(customer_id) % 4
```
* And places the row into one of 4 files.

* Why it matters
  * Faster JOINs
  * Faster GROUP BY
  * Enables map-side joins

* Advantages
  * Optimized joins
  * Even data distribution
  * Better performance on aggregations

* Requirements
```python
SET hive.enforce.bucketing=true;
```
* Data must be inserted using:
```sql
INSERT INTO TABLE customers SELECT …
```
* Direct file copy won't respect bucketing

# Managed vs External Tables(Conceptual)
Managed Tables
* Hive manages both metadata and data
* Dropping the table deletes underlying data
External Tables
* Hive manages only metadata
* Data remains in HDFS when table is dropped
Best Practice: External tables are preferred in production environments.

# Hive File Formats(Overview)
Hive supports multiple file formats:
* Text/CSV - raw ingestion
* JSON - semi-structured data
* Parquet/ORC - optimized analytics
* Avro - schema evolution support

# Hive's Role in Data Engineering
In typical data pipeline:
* Raw data lands in HDFS
* Hive external tables read raw data
* Transformations are applied
* Optimized formats (Parquet/ORC) are created
* Data is queried using Hive or Spark
Hive acts as the data warehouse layer in Hadoop-based architectures.

# Advantages of Hive
* SQL-based querying
* Handles massive datasets
* Integrates with Spark and Hadoop
* Cost-effective storage
* Scalable and fault tolerant

# Limitations of Hive
* High query latency
* Not suitable for real-time use cases
* Performance depends on data modeling

# Summary:
Apache Hive provides a powerful SQL abstraction over Hadoop, enabling scalable data warehousing and analytics on big data. It plays a critical role in modern Data Engineering pipelines.
