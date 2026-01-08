# Patient Visit Analytics (Partition Optimized)
* Visits by date range
* Business question:
  * “How many patient visits happened last week / last month?”

```sql
SELECT visit_date, COUNT(*) AS total_visits
FROM medical_visit
WHERE visit_date BETWEEN '2023-09-01' AND '2023-09-05'
GROUP BY visit_date;
```

# Why optimized:
* Partition pruning on visit_date → scans only relevant partitions.=
* Visits by region
  * Business question:
  * “Which region has the highest number of visits?”

```sql
SELECT region, COUNT(*) AS total_visits
FROM medical_visit
WHERE visit_date = '2023-09-01'
GROUP BY region;
```

# Why optimized:
* Partitions on visit_date + region.


# Prescription Analytics (Partition + Bucketing)
* Top prescribed drugs by region
  * Business question:
  * “What are the most prescribed drugs in each region?”

```sql
SELECT region, drug_name, COUNT(*) AS prescription_count
FROM prescriptions
WHERE prescription_date BETWEEN '2023-09-01' AND '2023-09-30'
GROUP BY region, drug_name
ORDER BY prescription_count DESC;
```

# Why optimized:
* Partition pruning on prescription_date and region
* Bucketing helps GROUP BY
* Drug trends over time
	* Business question:
	* “How has a specific drug’s usage changed over time?”

```sql
SELECT prescription_date, COUNT(*) AS usage_count
FROM prescriptions
WHERE drug_name = 'Lisinopril'
GROUP BY prescription_date
ORDER BY prescription_date;
```

# Why optimized:
* Partition pruning by date.


# Join Queries (Bucketing Advantage)
* Visits + prescriptions analysis
	* Business question:
	* “Which diagnoses lead to the most prescriptions?”

```sql
SELECT mv.diagnosis, COUNT(p.prescription_id) AS total_prescriptions
FROM medical_visit mv
JOIN prescriptions p
ON mv.visit_id = p.visit_id
GROUP BY mv.diagnosis;
```

# Why optimized:
* Bucketing on visit_id
* Reduced shuffle during join

* Patient-level analysis
	* Business question:
	* “What prescriptions are linked to patient visits?”

```sql
SELECT mv.patient_id, mv.diagnosis, p.drug_name
FROM medical_visit mv
JOIN prescriptions p
ON mv.visit_id = p.visit_id;
```

# Why optimized:
* Bucketed joins → faster execution.


# Patient Demographics Analytics
* Prescriptions by age group
	* Business question:
	* “Which age group receives the most prescriptions?”

```sql
SELECT patient_age, COUNT(*) AS total_prescriptions
FROM prescriptions
GROUP BY patient_age
ORDER BY total_prescriptions DESC;
```

* Gender-based analysis
	* Business question:
	* “Prescription distribution by gender?”

```sql
SELECT patient_gender, COUNT(*) AS total_prescriptions
FROM prescriptions
GROUP BY patient_gender;
```


# Healthcare Trend & Reporting Queries
* Most common diagnoses

```sql
SELECT diagnosis, COUNT(*) AS cases
FROM medical_visit
GROUP BY diagnosis
ORDER BY cases DESC;
```

* Region-wise healthcare load
```sql
SELECT region, COUNT(*) AS total_visits
FROM medical_visit
GROUP BY region;
```


# Performance-Focused Queries (to explain optimization)
* Partition pruning proof

```sql
EXPLAIN
SELECT *
FROM medical_visit
WHERE visit_date = '2023-09-01'
AND region = 'North';
```
* You can show that only specific partitions are scanned.
