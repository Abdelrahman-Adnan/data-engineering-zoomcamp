# Module 3 Homework Solutions - Data Warehousing & BigQuery

## Overview

This homework focuses on working with BigQuery and Google Cloud Storage using the Yellow Taxi Trip Records for January 2024 - June 2024.

**Dataset Source:**
- Yellow Taxi Trip Records (Parquet files): `https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page`

---

## Setup

### Loading Data to GCS

Data was loaded to GCS bucket using DLT (Data Load Tool) to upload the 6 parquet files for January-June 2024.

### BigQuery Setup

Create an external table using the Yellow Taxi Trip Records:
```sql
CREATE OR REPLACE EXTERNAL TABLE `[PROJECT_ID].[DATABASE].yellow_taxi_external`
OPTIONS (
  format = 'PARQUET',
  uris = [
    'gs://[BUCKET_NAME]/rides_dataset/rides/yellow_tripdata_2024_*.parquet'
  ]
);
```

Create a regular table in BQ using the Yellow Taxi Trip Records:
```sql
CREATE OR REPLACE TABLE `[PROJECT_ID].[DATABASE].yellow_taxi_2024`
AS 
SELECT * FROM `[PROJECT_ID].[DATABASE].yellow_taxi_external`;
```

---

## Quiz Questions & Answers

### Question 1. Counting records
**What is count of records for the 2024 Yellow Taxi Data?**

- 65,623
- 840,402
- 20,332,093
- 85,431,289

**Answer:** `20,332,093`

SQL query used:
```sql
SELECT COUNT(*) FROM `[PROJECT_ID].[DATABASE].yellow_taxi_external`;
```

---

### Question 2. Data read estimation
**Write a query to count the distinct number of PULocationIDs for the entire dataset on both the tables. What is the estimated amount of data that will be read when this query is executed on the External Table and the Table?**

- 18.82 MB for the External Table and 47.60 MB for the Materialized Table
- 0 MB for the External Table and 155.12 MB for the Materialized Table
- 2.14 GB for the External Table and 0MB for the Materialized Table
- 0 MB for the External Table and 0MB for the Materialized Table

**Answer:** `0 MB for the External Table and 155.12 MB for the Materialized Table`

SQL queries used:
```sql
-- External Table
SELECT COUNT(DISTINCT pu_location_id) AS distinct_pu_locations
FROM `[PROJECT_ID].[DATABASE].yellow_taxi_external`;

-- Materialized Table
SELECT COUNT(DISTINCT pu_location_id) AS distinct_pu_locations
FROM `[PROJECT_ID].[DATABASE].yellow_taxi_2024`;
```

---

### Question 3. Understanding columnar storage
**Write a query to retrieve the PULocationID from the table (not the external table) in BigQuery. Now write a query to retrieve the PULocationID and DOLocationID on the same table. Why are the estimated number of Bytes different?**

- BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed.
- BigQuery duplicates data across multiple storage partitions, so selecting two columns instead of one requires scanning the table twice, doubling the estimated bytes processed.
- BigQuery automatically caches the first queried column, so adding a second column increases processing time but does not affect the estimated bytes scanned.
- When selecting multiple columns, BigQuery performs an implicit join operation between them, increasing the estimated bytes processed

**Answer:** `BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed.`

SQL queries used:
```sql
-- Query for PULocationID only
SELECT pu_location_id
FROM `[PROJECT_ID].[DATABASE].yellow_taxi_2024`
LIMIT 10;

-- Query for PULocationID and DOLocationID
SELECT pu_location_id, do_location_id
FROM `[PROJECT_ID].[DATABASE].yellow_taxi_2024`
LIMIT 10;
```

---

### Question 4. Counting zero fare trips
**How many records have a fare_amount of 0?**

- 128,210
- 546,578
- 20,188,016
- 8,333

**Answer:** `8,333`

SQL query used:
```sql
SELECT COUNT(*) AS zero_fare_count
FROM `[PROJECT_ID].[DATABASE].yellow_taxi_2024`
WHERE fare_amount = 0;
```

---

### Question 5. Partitioning and clustering
**What is the best strategy to make an optimized table in Big Query if your query will always filter based on tpep_dropoff_datetime and order the results by VendorID (Create a new table with this strategy)?**

- Partition by tpep_dropoff_datetime and Cluster on VendorID
- Cluster on by tpep_dropoff_datetime and Cluster on VendorID
- Cluster on tpep_dropoff_datetime Partition by VendorID
- Partition by tpep_dropoff_datetime and Partition by VendorID

**Answer:** `Partition by tpep_dropoff_datetime and Cluster on VendorID`

**Explanation:**
- **Partitioning:** BigQuery partitions are used to filter large datasets efficiently. Since queries always filter on `tpep_dropoff_datetime`, partitioning by this column reduces the number of bytes scanned, making queries faster and cheaper.
- **Clustering:** Clustering organizes the data within each partition based on the values of one or more columns. Because queries order results by `VendorID`, clustering on `VendorID` improves sorting and aggregation performance.

SQL query to create the optimized table:
```sql
CREATE OR REPLACE TABLE `[PROJECT_ID].[DATABASE].yellow_taxi_optimized`
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY vendor_id AS
SELECT *
FROM `[PROJECT_ID].[DATABASE].yellow_taxi_2024`;
```

---

### Question 6. Partition benefits
**Write a query to retrieve the distinct VendorIDs between tpep_dropoff_datetime 2024-03-01 and 2024-03-15 (inclusive). Use the materialized table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 5 and note the estimated bytes processed. What are these values?**

- 12.47 MB for non-partitioned table and 326.42 MB for the partitioned table
- 310.24 MB for non-partitioned table and 26.84 MB for the partitioned table
- 5.87 MB for non-partitioned table and 0 MB for the partitioned table
- 310.31 MB for non-partitioned table and 285.64 MB for the partitioned table

**Answer:** `310.24 MB for non-partitioned table and 26.84 MB for the partitioned table`

SQL queries used:
```sql
-- Query on materialized (non-partitioned) table
SELECT DISTINCT vendor_id
FROM `[PROJECT_ID].[DATABASE].yellow_taxi_2024`
WHERE DATE(tpep_dropoff_datetime) BETWEEN '2024-03-01' AND '2024-03-15';

-- Query on partitioned table
SELECT DISTINCT vendor_id
FROM `[PROJECT_ID].[DATABASE].yellow_taxi_optimized`
WHERE DATE(tpep_dropoff_datetime) BETWEEN '2024-03-01' AND '2024-03-15';
```

---

### Question 7. External table storage
**Where is the data stored in the External Table you created?**

- Big Query
- Container Registry
- GCP Bucket
- Big Table

**Answer:** `GCP Bucket`

**Explanation:**
- When you create an External Table in BigQuery, BigQuery does not copy the data into its own storage.
- Instead, it references the data where it lives, in this case in Parquet files on GCS.
- Queries on the External Table read the data directly from GCS each time they run.

---

### Question 8. Clustering best practices
**It is best practice in Big Query to always cluster your data:**

- True
- False

**Answer:** `False`

**Explanation:**
Clustering is best practice only when your queries benefit from it, not always. Clustering adds overhead and may not be beneficial for small tables or when queries don't filter/order by the clustered columns.

---

### Question 9. Understanding table scans (No Points)
**Write a `SELECT count(*)` query FROM the materialized table you created. How many bytes does it estimate will be read? Why?**

SQL query:
```sql
SELECT COUNT(*) FROM `[PROJECT_ID].[DATABASE].yellow_taxi_2024`;
```

**Answer:** It shows `0 B` because BigQuery returned the result from cached query results. BigQuery caches COUNT(*) results as metadata, so it doesn't need to scan any actual data. To see the actual bytes processed, cached results must be disabled.

---
