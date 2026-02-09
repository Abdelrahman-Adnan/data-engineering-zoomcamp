# Module 3 Summary - Data Warehousing & BigQuery

#DataEngineeringZoomcamp #BigQuery #DataWarehouse #GCP

---

## Part 1: Understanding Data Warehouses & OLAP vs OLTP 🏢

### Why Do We Need Data Warehouses? 🤔

Imagine you run an online store. Your website has a database that handles:
- Customer sign-ups
- Product orders
- Payment processing
- Inventory updates

This database needs to be FAST because customers are waiting. Every millisecond counts!

Now, your boss asks: "What were our top-selling products last year by region, and how did that compare to the year before?"

Running that query on your production database would:
1. Slow down your website (bad for customers!)
2. Take forever because the database isn't designed for such complex analysis
3. Potentially crash things if the query is too heavy

**This is exactly why data warehouses exist!** They're a separate place to store your data, specifically designed for answering complex analytical questions without affecting your live applications.

### OLTP vs OLAP - The Two Worlds of Databases

These acronyms sound scary, but they're simple concepts:

**OLTP = Online Transaction Processing** (Your everyday app databases)
**OLAP = Online Analytical Processing** (Data warehouses for analysis)

| Aspect | OLTP (Transactional) | OLAP (Analytical) |
|--------|---------------------|-------------------|
| **What it's for** | Running your app - orders, logins, updates | Answering business questions - reports, dashboards |
| **Type of queries** | Simple: "Get user #123's info" | Complex: "Show sales trends by region for 5 years" |
| **Speed** | Super fast for small operations | Can take minutes for huge analyses |
| **Data freshness** | Real-time, always up-to-date | Usually updated daily/hourly (batch) |
| **How data is organized** | Normalized (split into many tables, no duplicates) | Denormalized (fewer tables, some duplication OK) |
| **Data size** | Gigabytes (current data) | Terabytes/Petabytes (years of history) |
| **Who uses it** | Your application, customers | Data analysts, managers, executives |
| **Examples** | MySQL for your website, PostgreSQL for your app | BigQuery, Snowflake, Amazon Redshift |

#### Real-World Example 🛒

**OLTP scenario (your app database):**
```sql
-- A customer places an order - needs to be FAST
INSERT INTO orders (customer_id, product_id, quantity, price) 
VALUES (123, 456, 2, 29.99);
```

**OLAP scenario (data warehouse):**
```sql
-- Your CEO wants to know Q4 performance - can take a minute, that's fine
SELECT 
    region,
    product_category,
    SUM(revenue) as total_revenue,
    COUNT(DISTINCT customer_id) as unique_customers
FROM sales_data
WHERE order_date BETWEEN '2023-10-01' AND '2023-12-31'
GROUP BY region, product_category
ORDER BY total_revenue DESC;
```

💡 **Key insight:** OLTP is like a cashier at a store - fast, handles one customer at a time. OLAP is like the accounting department - takes time to analyze all the receipts and produce reports.

### What Exactly is a Data Warehouse? 🏗️

A data warehouse is a **centralized repository** where you collect data from ALL your different systems and store it in a way that's optimized for analysis.

**Think of it like this:**

Imagine a company with multiple departments:
- Sales team uses Salesforce
- Marketing uses HubSpot  
- Website runs on PostgreSQL
- Inventory managed in SAP

Each system has its own database. But your CEO wants a report combining data from ALL of them. This is where a data warehouse comes in!

```
┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│  Salesforce │   │   HubSpot   │   │  PostgreSQL │
│   (Sales)   │   │ (Marketing) │   │  (Website)  │
└──────┬──────┘   └──────┬──────┘   └──────┬──────┘
       │                 │                 │
       │    ETL/ELT      │                 │
       │   (Extract,     │                 │
       │   Transform,    │                 │
       │    Load)        │                 │
       ▼                 ▼                 ▼
┌──────────────────────────────────────────────────┐
│              DATA WAREHOUSE (BigQuery)            │
│                                                  │
│   All your data, cleaned, organized, ready       │
│   for analysis!                                  │
└──────────────────────────────────────────────────┘
                        │
                        ▼
         ┌──────────────────────────┐
         │  Reports, Dashboards,    │
         │  Machine Learning, etc.  │
         └──────────────────────────┘
```

**Key characteristics of a data warehouse:**
- 📊 **Subject-oriented** - Organized by business topics (sales, customers, products)
- 🔗 **Integrated** - Data from multiple sources combined together
- 📅 **Time-variant** - Keeps historical data (years worth!)
- 🔒 **Non-volatile** - Data doesn't change once loaded (it's a historical record)

### Modern Cloud Data Warehouses ☁️

Traditional data warehouses (like Oracle, Teradata) required:
- Buying expensive hardware
- Hiring DBAs to manage servers
- Months of setup time
- Huge upfront costs

**Modern cloud data warehouses** (BigQuery, Snowflake, Redshift) changed everything:
- ✅ No servers to manage (serverless)
- ✅ Pay only for what you use
- ✅ Scales automatically
- ✅ Set up in minutes
- ✅ Access from anywhere

---

## Part 2: BigQuery Deep Dive 🔍

### What is BigQuery? 

BigQuery is Google's **data warehouse in the cloud**. It's one of the most popular choices for storing and analyzing large amounts of data because it's:

1. **Serverless** - You don't manage any servers. No installing software, no worrying about disk space, no maintenance. Google handles everything.

2. **Fully managed** - Google takes care of security, backups, scaling, and updates.

3. **Petabyte-scale** - Can handle absolutely massive datasets (1 petabyte = 1,000 terabytes = 1,000,000 gigabytes!)

4. **SQL-based** - You just write SQL queries. No need to learn a new programming language!

### Why BigQuery is Great for Beginners 🌟

- ☁️ **No setup headaches** - Create a project, load data, start querying. That's it!
- 💰 **Free tier** - 1TB of queries and 10GB storage free per month
- 📊 **Familiar SQL** - If you know basic SQL, you can use BigQuery
- 🔗 **Works with everything** - Google Sheets, Data Studio, Python, R, etc.
- 🤖 **Built-in ML** - Train machine learning models using just SQL!

### How BigQuery Works Under the Hood 🔧

Understanding the architecture helps you write better queries and save money. Don't worry, I'll keep it simple!

#### The Secret: Separation of Storage and Compute

Traditional databases store data and process queries on the same machine. BigQuery does something clever - it separates them:

```
┌─────────────────────────────────────────────────────────┐
│                    YOUR SQL QUERY                        │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                DREMEL (Compute Engine)                   │
│                                                         │
│   Your query gets broken into tiny pieces and           │
│   thousands of workers process them in parallel         │
└─────────────────────────┬───────────────────────────────┘
                          │
                          │  Jupiter Network (super fast!)
                          │  1 Terabyte per second
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                 COLOSSUS (Storage)                       │
│                                                         │
│   Your data lives here in COLUMNAR format               │
│   (organized by columns, not rows)                      │
└─────────────────────────────────────────────────────────┘
```

#### What Does "Columnar Storage" Mean? 📋

This is SUPER important for understanding BigQuery performance!

**Traditional databases (row-oriented):**
Stores data like this:
```
Row 1: [John, 25, New York, $50000]
Row 2: [Jane, 30, Chicago, $60000]
Row 3: [Bob, 35, Miami, $55000]
```

To find all salaries, it reads EVERY row, even though you only need one column.

**BigQuery (column-oriented):**
Stores data like this:
```
Names column:    [John, Jane, Bob]
Ages column:     [25, 30, 35]
Cities column:   [New York, Chicago, Miami]
Salaries column: [$50000, $60000, $55000]
```

To find all salaries, it ONLY reads the salary column! Much faster and cheaper!

💡 **This is why `SELECT *` is expensive in BigQuery** - it has to read EVERY column. Always specify only the columns you need!

#### The Dremel Execution Engine 🚀

When you run a query, here's what happens:

1. **Root Server** receives your query
2. Query is broken into smaller pieces
3. **Mixers** distribute work to thousands of **Leaf Nodes**
4. Each Leaf Node processes a small chunk of data in parallel
5. Results flow back up through Mixers to Root
6. You get your final result!

```
                    ┌──────────┐
                    │   ROOT   │  ← Your query comes here
                    └────┬─────┘
                         │
           ┌─────────────┼─────────────┐
           ▼             ▼             ▼
      ┌────────┐    ┌────────┐    ┌────────┐
      │ MIXER  │    │ MIXER  │    │ MIXER  │
      └───┬────┘    └───┬────┘    └───┬────┘
          │             │             │
    ┌─────┼─────┐ ┌─────┼─────┐ ┌─────┼─────┐
    ▼     ▼     ▼ ▼     ▼     ▼ ▼     ▼     ▼
   [L]   [L]   [L][L]   [L]   [L][L]   [L]   [L]
   
   L = Leaf nodes (thousands of them!)
```

**Why this matters:** A query that would take hours on your laptop can run in seconds because thousands of machines work on it simultaneously!

### External Tables vs Native Tables 📦

You have two ways to work with data in BigQuery:

#### Option 1: External Tables (Data stays in GCS)

Your data remains in Google Cloud Storage, BigQuery just reads it when you query.

```sql
-- Create external table pointing to files in GCS bucket
CREATE OR REPLACE EXTERNAL TABLE `my-project.my_dataset.taxi_external`
OPTIONS (
  format = 'PARQUET',
  uris = ['gs://my-bucket/taxi_data/*.parquet']
);
```

**When to use External Tables:**
- ✅ You want to save on storage costs (GCS is cheaper than BigQuery storage)
- ✅ One-time or occasional analysis
- ✅ Data is updated frequently in source system
- ✅ Quick exploration before committing to load

**Downsides:**
- ❌ Slower queries (data needs to be read from GCS each time)
- ❌ No cost estimation before running queries
- ❌ Can't partition or cluster (limited optimization)

#### Option 2: Native Tables (Data loaded into BigQuery)

Data is copied into BigQuery's own storage (Colossus).

```sql
-- Create native table from external table
CREATE OR REPLACE TABLE `my-project.my_dataset.taxi_native` AS
SELECT * FROM `my-project.my_dataset.taxi_external`;
```

**When to use Native Tables:**
- ✅ Frequently queried data
- ✅ Need best query performance
- ✅ Want to use partitioning and clustering
- ✅ Need accurate cost estimates before running queries

**Downsides:**
- ❌ Higher storage costs
- ❌ Data duplication (exists in both GCS and BigQuery)

💡 **Pro tip:** Start with external tables for exploration, then load into native tables once you know what data you actually need!

### Understanding BigQuery Costs 💰

BigQuery has two main pricing models:

#### On-Demand Pricing (Pay per query)
- **$5 per TB** of data scanned
- Good for: Occasional users, unpredictable workloads
- You pay for how much data your queries read

#### Flat-Rate Pricing (Monthly commitment)
- **~$2,000/month** for 100 "slots" (compute units)
- Good for: Heavy users, predictable workloads
- Unlimited queries within your slot capacity

#### How to Estimate Query Cost 🧮

Before running a query, BigQuery shows you how much data it will scan:

```
┌────────────────────────────────────────────────┐
│  Query Editor                                  │
│  ─────────────────────────────────────────────│
│  SELECT * FROM my_table WHERE date = '2024-01'│
│                                                │
│  [This query will process 2.5 GB when run]    │ ← Check this!
└────────────────────────────────────────────────┘
```

**Cost calculation:**
- 2.5 GB = 0.0025 TB
- 0.0025 TB × $5 = $0.0125 (about 1 cent)

But if you run that query 100 times a day... costs add up!

### Cost Optimization Tips 💡

1. **NEVER use `SELECT *`** unless you absolutely need every column
   ```sql
   -- ❌ Bad - reads ALL columns
   SELECT * FROM taxi_data;
   
   -- ✅ Good - reads only what you need
   SELECT pickup_time, dropoff_time, fare_amount FROM taxi_data;
   ```

2. **Use partitioned tables** (covered in Part 3)

3. **Preview before running** - Always check the estimated bytes

4. **Use LIMIT wisely** - It doesn't reduce data scanned! The filtering happens AFTER reading.
   ```sql
   -- ❌ Still scans the whole table!
   SELECT * FROM huge_table LIMIT 10;
   
   -- ✅ Better - add a WHERE clause first
   SELECT * FROM huge_table WHERE date = CURRENT_DATE() LIMIT 10;
   ```

5. **Cache results** - BigQuery caches query results for 24 hours (free!)

### BigQuery Caching 🗄️

When you run the same query twice:
- First run: Scans data, costs money
- Second run: Returns cached result, FREE!

Cache is invalidated when:
- Underlying table data changes
- 24 hours pass
- You disable caching in query settings

---

## Part 3: Partitioning & Clustering for Performance 🚀

This is where BigQuery optimization gets really powerful! These two techniques can reduce your query costs by 90% or more.

### The Problem We're Solving 🤔

Imagine you have a table with 5 years of taxi trip data - about 500 million rows. Every time you query:

```sql
SELECT * FROM taxi_trips WHERE pickup_date = '2024-01-15';
```

Without optimization, BigQuery scans ALL 500 million rows just to find trips from one day. That's:
- Slow (lots of data to read)
- Expensive (you pay for all data scanned)
- Wasteful (you only needed 0.05% of the data!)

**Partitioning and clustering solve this problem!**

### Partitioning: Dividing Your Table into Sections 📁

Think of partitioning like organizing a filing cabinet. Instead of one giant drawer with all documents, you have:
- Drawer for January
- Drawer for February
- Drawer for March
- ...and so on

When you need something from March, you ONLY open the March drawer!

#### How Partitioning Works in BigQuery

```sql
-- Create a table partitioned by date
CREATE OR REPLACE TABLE `project.dataset.taxi_partitioned`
PARTITION BY DATE(pickup_datetime) AS
SELECT * FROM `project.dataset.taxi_external`;
```

Now your table looks like this internally:

```
taxi_partitioned/
├── 2024-01-01/    (all trips from Jan 1)
├── 2024-01-02/    (all trips from Jan 2)
├── 2024-01-03/    (all trips from Jan 3)
│   ... 
├── 2024-06-30/    (all trips from Jun 30)
└── [metadata]
```

When you query with a date filter:
```sql
SELECT * FROM taxi_partitioned 
WHERE DATE(pickup_datetime) = '2024-03-15';
```

BigQuery ONLY reads the 2024-03-15 partition! The other 180+ partitions are never touched.

#### Types of Partitioning

**1. Time-based partitioning (most common)**
```sql
-- Partition by day (default)
PARTITION BY DATE(pickup_datetime)

-- Partition by month (for less granular data)
PARTITION BY DATE_TRUNC(pickup_datetime, MONTH)

-- Partition by year
PARTITION BY DATE_TRUNC(pickup_datetime, YEAR)
```

**2. Integer range partitioning**
```sql
-- Partition by customer ID ranges
PARTITION BY RANGE_BUCKET(customer_id, GENERATE_ARRAY(0, 1000000, 10000))
```

**3. Ingestion time partitioning**
```sql
-- Partition by when data was loaded
PARTITION BY _PARTITIONDATE
```

#### Partitioning Rules to Remember ⚠️

| Rule | Details |
|------|---------|
| **Max partitions** | 4,000 per table |
| **Min partition size** | Aim for at least 1GB per partition |
| **One column only** | Can only partition on ONE column |
| **Column types** | DATE, TIMESTAMP, DATETIME, or INTEGER |

💡 **When NOT to use partitioning:**
- If you'd have < 1GB per partition (use clustering instead)
- If you'd exceed 4,000 partitions
- If you rarely filter on the partition column

### Clustering: Organizing Data Within Partitions 🗂️

If partitioning is like having separate drawers in a filing cabinet, clustering is like organizing the folders WITHIN each drawer alphabetically.

#### How Clustering Works

```sql
-- Create table with partitioning AND clustering
CREATE OR REPLACE TABLE `project.dataset.taxi_optimized`
PARTITION BY DATE(pickup_datetime)
CLUSTER BY vendor_id, payment_type AS
SELECT * FROM `project.dataset.taxi_external`;
```

Now within each date partition, data is sorted by vendor_id, then by payment_type:

```
taxi_optimized/
├── 2024-01-15/
│   ├── vendor_id=1, payment_type=1, ...
│   ├── vendor_id=1, payment_type=2, ...
│   ├── vendor_id=2, payment_type=1, ...
│   └── vendor_id=2, payment_type=2, ...
├── 2024-01-16/
│   └── (similarly organized)
```

When you query:
```sql
SELECT * FROM taxi_optimized 
WHERE DATE(pickup_datetime) = '2024-01-15'
  AND vendor_id = 1;
```

BigQuery:
1. Goes directly to the 2024-01-15 partition (thanks to partitioning)
2. Reads only the vendor_id=1 blocks (thanks to clustering)

**Even more data skipped = even faster and cheaper!**

#### Clustering Rules 📏

| Rule | Details |
|------|---------|
| **Max columns** | Up to 4 clustering columns |
| **Order matters** | Put most filtered column first |
| **No cost for re-clustering** | BigQuery automatically re-clusters as data is added |
| **Works with partitioning** | Best used together! |
| **Minimum table size** | Most effective for tables > 1GB |

**Good clustering column candidates:**
- Columns you frequently filter on (WHERE clause)
- Columns you frequently group by (GROUP BY clause)
- High-cardinality columns (many distinct values)

### Partitioning vs Clustering: When to Use What? 🤷

| Scenario | Recommendation |
|----------|---------------|
| Always filter by date | Partition by date |
| Filter by date AND other columns | Partition by date, cluster by other columns |
| Filter by multiple non-date columns | Cluster by those columns |
| Need to know query cost upfront | Must use partitioning (clustering doesn't show estimates) |
| Less than 1GB per potential partition | Use clustering instead |
| Would have > 4,000 partitions | Use clustering instead |
| Data is rarely filtered | Maybe neither - analyze your query patterns first |

### Real-World Performance Comparison 📊

I ran tests on the NYC taxi dataset (about 20 million rows). Here are the results:

#### Test 1: Filtering by Date Range

```sql
SELECT DISTINCT vendor_id 
FROM [table] 
WHERE DATE(pickup_datetime) BETWEEN '2024-03-01' AND '2024-03-15';
```

| Table Type | Data Scanned | Cost |
|------------|--------------|------|
| Non-partitioned | 310 MB | $0.00155 |
| Partitioned by date | 27 MB | $0.000135 |
| **Savings** | **91% less!** | **91% cheaper!** |

#### Test 2: Filtering by Date AND Vendor

```sql
SELECT COUNT(*) 
FROM [table] 
WHERE DATE(pickup_datetime) BETWEEN '2024-06-01' AND '2024-06-30'
  AND vendor_id = 1;
```

| Table Type | Data Scanned |
|------------|--------------|
| Partitioned only | 1.1 GB |
| Partitioned + Clustered | 865 MB |
| **Additional savings** | **21% less!** |

**Combined savings: Over 90% reduction in costs!** 💰

### Step-by-Step: Creating an Optimized Table

Here's the full workflow:

```sql
-- Step 1: Create external table pointing to your data in GCS
CREATE OR REPLACE EXTERNAL TABLE `my-project.dataset.taxi_external`
OPTIONS (
  format = 'PARQUET',
  uris = ['gs://my-bucket/yellow_taxi_2024/*.parquet']
);

-- Step 2: Check how many records we have
SELECT COUNT(*) FROM `my-project.dataset.taxi_external`;
-- Result: 20,332,093 records

-- Step 3: Create optimized table with partitioning and clustering
CREATE OR REPLACE TABLE `my-project.dataset.taxi_optimized`
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID AS
SELECT * FROM `my-project.dataset.taxi_external`;

-- Step 4: Verify partitions were created
SELECT 
  table_name, 
  partition_id, 
  total_rows
FROM `dataset.INFORMATION_SCHEMA.PARTITIONS`
WHERE table_name = 'taxi_optimized'
ORDER BY partition_id;
```

### Best Practices Cheat Sheet ✅

#### For Reducing Costs 💵
- ❌ Never use `SELECT *`
- ✅ Only query columns you need
- ✅ Use partitioned tables
- ✅ Add clustering for frequently filtered columns
- ✅ Check estimated bytes before running
- ✅ Use table previews instead of SELECT for quick looks

#### For Better Performance ⚡
- ✅ Filter early - apply WHERE before JOINs
- ✅ Put largest table first in JOINs
- ✅ Use ORDER BY at the end of query
- ✅ Consider approximate functions (APPROX_COUNT_DISTINCT) when exact precision isn't needed
- ✅ Avoid JavaScript UDFs when possible
- ✅ Don't over-partition (keep partitions > 1GB)

---

## Quick Reference: Common SQL Patterns for Beginners 📝

Here are the most useful BigQuery SQL commands you'll need:

### Basic Queries

```sql
-- Count all records in a table
SELECT COUNT(*) FROM `project.dataset.table`;

-- Count distinct values in a column
SELECT COUNT(DISTINCT vendor_id) FROM `project.dataset.taxi`;

-- Get first 10 rows (but remember - this still scans the whole table!)
SELECT * FROM `project.dataset.taxi` LIMIT 10;

-- Better way to preview - use table preview in BigQuery console instead!
```

### Filtering Data

```sql
-- Filter by exact value
SELECT * FROM `project.dataset.taxi`
WHERE vendor_id = 1;

-- Filter by date range
SELECT * FROM `project.dataset.taxi`
WHERE DATE(pickup_datetime) BETWEEN '2024-01-01' AND '2024-01-31';

-- Filter with multiple conditions
SELECT * FROM `project.dataset.taxi`
WHERE vendor_id = 1 
  AND fare_amount > 10
  AND DATE(pickup_datetime) = '2024-03-15';
```

### Aggregations

```sql
-- Sum, average, min, max
SELECT 
  SUM(fare_amount) as total_fares,
  AVG(fare_amount) as avg_fare,
  MIN(fare_amount) as min_fare,
  MAX(fare_amount) as max_fare,
  COUNT(*) as trip_count
FROM `project.dataset.taxi`;

-- Group by
SELECT 
  vendor_id,
  COUNT(*) as trips,
  AVG(fare_amount) as avg_fare
FROM `project.dataset.taxi`
GROUP BY vendor_id;
```

### Creating Tables

```sql
-- Create external table from GCS
CREATE OR REPLACE EXTERNAL TABLE `project.dataset.taxi_external`
OPTIONS (
  format = 'PARQUET',
  uris = ['gs://bucket-name/folder/*.parquet']
);

-- Create native table from external
CREATE OR REPLACE TABLE `project.dataset.taxi_native` AS
SELECT * FROM `project.dataset.taxi_external`;

-- Create partitioned + clustered table
CREATE OR REPLACE TABLE `project.dataset.taxi_optimized`
PARTITION BY DATE(pickup_datetime)
CLUSTER BY vendor_id
AS SELECT * FROM `project.dataset.taxi_external`;
```

### Checking Table Info

```sql
-- View partition information
SELECT 
  table_name, 
  partition_id, 
  total_rows
FROM `dataset.INFORMATION_SCHEMA.PARTITIONS`
WHERE table_name = 'your_table_name'
ORDER BY partition_id;

-- Check table schema
SELECT column_name, data_type 
FROM `dataset.INFORMATION_SCHEMA.COLUMNS`
WHERE table_name = 'your_table_name';
```

---

## Glossary for Beginners 📚

| Term | Simple Explanation |
|------|-------------------|
| **Data Warehouse** | A big database designed for analyzing historical data, not for running apps |
| **OLTP** | Databases for running applications (fast, small transactions) |
| **OLAP** | Databases for analysis (complex queries, lots of data) |
| **BigQuery** | Google's cloud data warehouse service |
| **GCS** | Google Cloud Storage - where you store files in the cloud |
| **External Table** | A table that reads data from GCS without copying it |
| **Native Table** | A table with data stored in BigQuery itself |
| **Partitioning** | Splitting a table into smaller pieces by date or number |
| **Clustering** | Sorting data within partitions by specific columns |
| **Columnar Storage** | Storing data by column instead of row (faster for analytics) |
| **Slot** | A unit of compute power in BigQuery |
| **Data Scanned** | How much data BigQuery reads to answer your query (you pay for this!) |

---

## Common Mistakes to Avoid ⚠️

1. **Using `SELECT *` everywhere** - Always specify columns you need!

2. **Thinking LIMIT reduces cost** - It doesn't! BigQuery scans first, limits after.

3. **Not using partitions** - Always partition time-series data by date.

4. **Wrong partition column** - Partition by columns you ALWAYS filter on.

5. **Too many partitions** - Keep it under 4,000, aim for >1GB per partition.

6. **Ignoring the query validator** - Always check estimated bytes before running!

7. **Not using clustering with partitioning** - They work best together!

---

## Resources for Learning More 📖

- 📊 [BigQuery Official Documentation](https://cloud.google.com/bigquery/docs)
- 🎥 [DE Zoomcamp Video: Data Warehouse and BigQuery](https://youtu.be/jrHljAoD6nM)
- 🎥 [DE Zoomcamp Video: Partitioning vs Clustering](https://youtu.be/-CqXf7vhhDs)
- 🎥 [DE Zoomcamp Video: Best Practices](https://youtu.be/k81mLJVX08w)
- 🎥 [DE Zoomcamp Video: Internals of BigQuery](https://youtu.be/eduHi1inM4s)
- 📝 [Course SQL Examples](https://github.com/DataTalksClub/data-engineering-zoomcamp/blob/main/03-data-warehouse/big_query.sql)
- 📑 [Course Slides](https://docs.google.com/presentation/d/1a3ZoBAXFk8-EhUsd7rAZd-5p_HpltkzSeujjRGB2TAI/edit)

---

## Summary: Key Takeaways 🎯

1. **Data warehouses** are for analysis, not running apps - that's why they exist!

2. **BigQuery** is serverless - no servers to manage, just write SQL.

3. **Columnar storage** = only reads columns you request = faster + cheaper.

4. **External tables** = data in GCS, slower but flexible.

5. **Native tables** = data in BigQuery, faster but costs more storage.

6. **Partitioning** = split table by date, only scan relevant dates.

7. **Clustering** = sort data within partitions, skip irrelevant blocks.

8. **Always check estimated bytes** before running queries!

9. **Never use `SELECT *`** - specify only the columns you need.

10. **Combine partitioning + clustering** for maximum optimization!

---

#DataEngineeringZoomcamp #BigQuery #DataWarehouse #GCP #SQL #CloudComputing
