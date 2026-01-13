# Hive Collections

This repository contains **structured learning notes on Apache Hive**, organized into **concepts** and **hands-on commands** for data engineering.

The goal of this repo is to provide:
- Clear understanding of Hive fundamentals
- One-page conceptual overview
- Practical HiveQL commands for daily usage
- A strong foundation for Hadoop, Spark, and data warehouse projects

---

# What is Apache Hive?

Apache Hive is a **data warehouse system** built on top of Hadoop that enables querying large datasets stored in HDFS using **SQL-like syntax (HiveQL)**.

Hive abstracts complex distributed processing and allows users to perform analytics without writing low-level MapReduce code.

---

# Repository Structure

```text
hive-collections/
│
├── README.md          # Overview of Hive and repository structure
├── hive-fundaments.md   # One-page Hive concepts & architecture
└── hive-commands.md   # HiveQL commands and hands-on examples
casestudies/
|
├── business/
        ├── README.md
        ├── e-commerce_business_problem.md
├── healthcare/
        ├── README.md
        ├── healthcare_data_analytics.md
        ├── healthcare_test_queries.md
├── telecom/
        ├── README.md
        ├── telecom_data_analytics.md
        └── telecommunication_test_queries.md
