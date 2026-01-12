# Telecommunication Analytics – Hive SQL Queries

* Call Analytics Queries
* Total call duration by region and date
```sql
SELECT
    region,
    call_date,
    SUM(call_duration) AS total_call_duration
FROM call_data
GROUP BY region, call_date;
```

* Average call duration per customer
```sql
SELECT
    customer_id,
    AVG(call_duration) AS avg_call_duration
FROM call_data
GROUP BY customer_id;
```

* Customers with long call durations
```sql
SELECT
    customer_id,
    SUM(call_duration) AS total_call_duration
FROM call_data
GROUP BY customer_id
HAVING SUM(call_duration) > 100;
```
 
# Data Usage Analytics Queries
* Top data users per region
```sql
SELECT
    region,
    customer_id,
    SUM(data_used) AS total_data_used
FROM data_usage
GROUP BY region, customer_id
ORDER BY total_data_used DESC;
```
 
* Total data consumption per customer
```sql
SELECT
    customer_id,
    SUM(data_used) AS total_data_used
FROM data_usage
GROUP BY customer_id;
```

* Daily data usage trends
```sql
SELECT
    usage_date,
    SUM(data_used) AS daily_data_usage
FROM data_usage
GROUP BY usage_date
ORDER BY usage_date;
```

# SMS Analytics Queries
* Total SMS sent per customer
```sql
SELECT
    customer_id,
    SUM(sms_count) AS total_sms
FROM sms_data
GROUP BY customer_id;
```
 
* Regions with highest SMS activity
```sql
SELECT
    region,
    SUM(sms_count) AS total_sms
FROM sms_data
GROUP BY region
ORDER BY total_sms DESC;
```
 
* Daily SMS usage trends
```sql
SELECT
    sms_date,
    SUM(sms_count) AS daily_sms
FROM sms_data
GROUP BY sms_date
ORDER BY sms_date;
```

# Customer-Centric (Cross-Table) Queries
* 360° customer telecom usage profile
```sql
SELECT
    c.customer_id,
    SUM(c.call_duration) AS total_call_duration,
    SUM(d.data_used) AS total_data_used,
    SUM(s.sms_count) AS total_sms
FROM call_data c
JOIN data_usage d
    ON c.customer_id = d.customer_id
JOIN sms_data s
    ON c.customer_id = s.customer_id
GROUP BY c.customer_id;
```

* Heavy data users with low call activity
```sql
SELECT
    d.customer_id,
    SUM(d.data_used) AS total_data_used,
    SUM(c.call_duration) AS total_call_duration
FROM data_usage d
JOIN call_data c
    ON d.customer_id = c.customer_id
GROUP BY d.customer_id
HAVING SUM(d.data_used) > 50
   AND SUM(c.call_duration) < 20;
```

# Regional & Time-Based Queries (Partition-Aware)
* Telecom usage for a specific region and date
```sql
SELECT
    customer_id,
    SUM(call_duration) AS total_calls
FROM call_data
WHERE region = 'North'
  AND call_date = '2023-08-01'
GROUP BY customer_id;
```
* Partition pruning happens here

* Monthly call usage per region
```sql
SELECT
    region,
    SUBSTR(call_date, 1, 7) AS month,
    SUM(call_duration) AS total_call_duration
FROM call_data
GROUP BY region, SUBSTR(call_date, 1, 7);
```

# Ranking & Business Intelligence Queries
* Rank customers by total data usage
```sql
SELECT
    customer_id,
    SUM(data_used) AS total_data_used,
    RANK() OVER (ORDER BY SUM(data_used) DESC) AS usage_rank
FROM data_usage
GROUP BY customer_id;
```

* Top 5 customers by combined telecom usage
```sql
SELECT
    customer_id,
    total_usage
FROM (
    SELECT
        customer_id,
        SUM(call_duration) + SUM(data_used) + SUM(sms_count) AS total_usage
    FROM call_data c
    JOIN data_usage d USING (customer_id)
    JOIN sms_data s USING (customer_id)
    GROUP BY customer_id
) t
ORDER BY total_usage DESC
LIMIT 5;
```

# Why These Queries Are Optimized
* Partition pruning → date & region filters
* Bucketing on customer_id → faster joins
* Columnar storage (ORC/Parquet) → minimal IO
* Aggregation-heavy workloads → Hive strength
