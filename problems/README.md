# Business Use Case: E-Commerce Sales Analytics with Hive Partitioning and Bucketing  
Problem Statement
 * An e-commerce company processes massive amounts of sales data from customers across different regions and countries. The data is queried frequently for analytics purposes, such as:
 * Daily Sales Reports by country and region.
 * Product Sales Trends to identify the top-selling products in specific regions.
 * Customer Purchase Patterns for customer segmentation and targeted marketing.

The dataset is large (millions of records), and querying it without optimization leads to slow performance. The company wants to:  
 * Optimize queries for specific date ranges (daily sales reports).
 * Speed up queries for product categories.
 * Use partitioning and bucketing to manage this large dataset efficiently in Hive.

Sales Table (sales_data)  
  * Columns:
   * sale_id: INT
   * product_id: INT
   * product_category: STRING
   * customer_id: INT
   * sale_amount: FLOAT
   * sale_date: DATE
   * country: STRING
   * region: STRING

Sample Data:  
|sale_id|product_id|product_category|customer_id|sale_amount|sale_date|country|region|
|-------|---------|-----------------|-----------|-----------|---------|-------|-------|
|1| 101| Electronics| 2001|500.00| 2023-08-01| US| North America|
|2| 102| Electronics| 2002| 300.00|2023-08-01| US| North America|
|3| 103| Furniture| 2003| 700.00| 2023-08-02| UK|Europe|
|4| 104| Furniture| 2004| 800.00|2023-08-03| UK| Europe|
|5| 105| Clothing| 2005| 200.00| 2023-08-03| IN| Asia|
|6| 106| Clothing| 2006| 600.00| 2023-08-03| IN| Asia|
