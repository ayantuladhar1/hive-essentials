# Solution
Solution Statement
In this healthcare analytics scenario, we first design an optimization strategy because, without optimization, analytical queries would scan the entire dataset, leading to poor performance as medical records, patient visits, and prescription data grow rapidly over time. As the healthcare organization processes millions of visit and prescription records across multiple regions and dates, inefficient queries increase compute time and cost. To support frequent reporting on patient visits, regional healthcare trends, prescription analysis, and demographic insights, Hive partitioning and bucketing are used to optimize query performance and data organization.

# Optimization Statement

|Requirement|	Hive Feature Used|	Reason|
|-----------|------------------|--------|
|Filter patient visits by date|	Partitioning| Avoid full table scans by pruning irrelevant date partitions|
|Filter reports by region|	Partitioning |Query only required regional partitions|
|Faster joins between visits & prescriptions|	Bucketing|	Reduces shuffle and optimizes join performance|
|Patient-level & visit-level analytics|	Bucketing on join keys|	Even data distribution and efficient aggregations|

# Hive Table Design
Partitioning Strategy:
* Partition by: visit_date, region (Medical Visits)
* Partition by: prescription_date, region (Prescriptions)
* Most healthcare reports are time-based, and region is a frequent filter for operational and regulatory reporting.

Bucketing Strategy:
* Bucket by: patient_id (Medical Visits)
* Bucket by: visit_id (Prescriptions)
* Buckets: 8
* Bucketing improves GROUP BY, JOIN, and aggregation queries, especially when analyzing patient histories and prescription trends.

# Step 1:
* Create a file "medical_visits.csv" in HDFS and save it under path '/data/medical_visits.csv'
* Create a file "prescriptions.csv" in HDFS and save it under path '/data/prescriptions.csv'
* It will help Hive to load the files and save it under the tables later
  
# Step 2:
* We need to create 4 tables in Hive, first will be medical_visit_staging and medical_visit and second will be prescriptions_staging and prescriptions
* In Hive when we create a table and store as ORC or Parquet, it expects the data to be in ORC or Parquet format, which makes it inconvenience for us to directly load the data into the table as our data is in csv format.
* We will use the staging tables to load the data as csv
```sql
	CREATE TABLE medical_visit_staging (
	    visit_id INT,
	    patient_id INT,
	    region STRING,
	    visit_date DATE,
	    diagnosis STRING,
	    treatment STRING
	)
	ROW FORMAT DELIMITED
	FIELDS TERMINATED BY ','
	STORED AS TEXTFILE;
	
	CREATE TABLE prescriptions_staging (
	    prescriptions_staging INT,
	    visit_id INT,
	    patient_age INT,
	    patient_gender STRING,
	    drug_name STRING,
	    dosage STRING,
	    region STRING,
	    prescription_date DATE
	)
	ROW FORMAT DELIMITED
	FIELDS TERMINATED BY ','
	STORED AS TEXTFILE;
```
# Step 3:
* Load the Data in Hive Table medical_visit_staging.
```sql
LOAD DATA INPATH '/data/medical_visits.csv' INTO TABLE medical_visit_staging; 
```
* Load the Data in Hive Table prescriptions_staging.
```sql
LOAD DATA INPATH '/data/prescriptions.csv' INTO TABLE prescriptions_staging; 
```	
# Step 4:
	• We will create medical_visit and prescriptions tables where we will Partition visit_date, region and bucket patient_id for medical_visit and Partition prescription_date, region and bucket visit_id for prescriptions where both of the tables would be stored as Parquet
	CREATE TABLE medical_visit (
	    visit_id INT,
	    patient_id INT,
	    diagnosis STRING,
	    treatment STRING)
	PARTITIONED BY (
	    visit_date DATE,
	    region STRING
	)
	CLUSTERED BY (patient_id)
	INTO 8 BUCKETS
	STORED AS PARQUET;
	
	CREATE TABLE prescriptions (
	    prescription_id INT,
	    visit_id INT,
	    patient_age INT,
	    patient_gender STRING,
	    drug_name STRING,
	    dosage STRING)
	PARTITIONED BY (
	    prescription_date DATE,
	    region STRING)
	CLUSTERED BY (visit_id)
	INTO 8 BUCKETS
	STORED AS PARQUET;
	
# Step 5:
	• Set the following properties in Hive for dynamic partition
	SET hive.exec.dynamic.partition = true;
	SET hive.exec.dynamic.partition.mode = nonstrict;
	
# Step 6:
	• We will insert the data we loaded earlier from medical_visit_staging table to  medical_visit table and prescriptions_staging table to prescriptions table.
	INSERT INTO TABLE  medical_visit
	PARTITION (visit_date, region)
	SELECT
	    visit_id,
	    patient_id,
	    diagnosis,
	    treatment,
	    visit_date,
	    region
	FROM sales_data_staging;
	
	
	
	INSERT INTO TABLE  prescriptions
	PARTITION (prescription_date , region)
	SELECT
	   prescription_id,
	   visit_id,
	   patient_age,
	   patient_gender,
	   drug_name,
	   dosage, region,
	   prescription_date
	FROM sales_data_staging;
	
	
# Step 7:
	• We will verify the Partition.
	SHOW PARTITIONS medical_visit;
	

	• SHOW PARTITIONS prescriptions;
	
# Step 8:
	• Test Query
	• SELECT * FROM medical_visit WHERE visit_date = '2023-09-01' AND region = 'North'; 
	
	
<img width="911" height="5232" alt="image" src="https://github.com/user-attachments/assets/189eca0c-baff-4a63-a354-d9179c3ee92f" />
