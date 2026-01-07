# Business Use Case: Telecommunication Data Analytics with Hive Partitioning and Bucketing  
Problem Statement
* A telecommunication company processes vast amounts of data related to customer calls, SMS usage, and data consumption. This data is used to analyze telecom usage trends, such as:
* Call Durations by region and date.
* Top Data Users in specific regions.
* SMS usage trends based on customer demographics.

Given the large scale of the dataset (millions of records), querying the data for business insights is slow without proper optimization. The company wants to:
* Optimize queries to retrieve data based on call date and region for reporting purposes.
* Speed up queries to identify heavy data users.
* Use partitioning and bucketing to manage the dataset efficiently in Hive, improving query performance and storage optimization.

Sample Data
Call Data Table (call_data)
* Columns:
  * call_id: INT
  * customer_id: INT
  * call_duration: FLOAT
  * region: STRING
  * call_date: DATE

Sample Data:
1, 101, 15.5, North, 2023-08-01
2, 102, 20.2, South, 2023-08-02
3, 103, 5.7, East, 2023-08-03
4, 104, 12.4, West, 2023-08-04
5, 105, 25.0, North, 2023-08-05

Data Usage Table (data_usage)
* Columns:
  * usage_id: INT
  * customer_id: INT
  * data_used: FLOAT (in GB)
  * region: STRING
  * usage_date: DATE

Sample Data:
1, 101, 2.5, North, 2023-08-01
2, 102, 3.0, South, 2023-08-02
3, 103, 1.2, East, 2023-08-03
4, 104, 5.5, West, 2023-08-04
5, 105, 10.0, North, 2023-08-05

SMS Data Table (sms_data)
* Columns:
  * sms_id: INT
  * customer_id: INT
  * sms_count: INT
  * region: STRING
  * sms_date: DATE

Sample Data:
1, 101, 5, North, 2023-08-01
2, 102, 10, South, 2023-08-02
3, 103, 8, East, 2023-08-03
4, 104, 7, West, 2023-08-04
5, 105, 15, North, 2023-08-05
