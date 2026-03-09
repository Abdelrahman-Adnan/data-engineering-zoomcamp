# Module 6 Summary - Batch Processing with Apache Spark

#DataEngineeringZoomcamp #Spark #PySpark #BatchProcessing #BigData #DataEngineering

---

## Part 1: Introduction to Batch Processing 🎯

### What is Batch Processing?

Think of it like doing laundry. You have two choices:

1. **Stream Processing** 🚿 — Wash each piece of clothing the moment it gets dirty (one at a time, immediately)
2. **Batch Processing** 🧺 — Collect dirty clothes all week and wash them in one big load on Sunday

In data engineering, **batch processing** means collecting data over a period of time and then processing it all at once, as a single "batch."

```
📦 Batch Processing Flow:

    Events happen over time...
    ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐
    │ 9am │ │10am │ │11am │ │12pm │ │ 1pm │   ← Data arrives throughout the day
    └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘
       │       │       │       │       │
       └───────┴───────┴───┬───┴───────┘
                           │
                           ▼
                   ┌───────────────┐
                   │  BATCH JOB    │   ← Process everything at once
                   │  (runs daily) │      (e.g., every night at 2 AM)
                   └───────┬───────┘
                           │
                           ▼
                   ┌───────────────┐
                   │    Results    │   ← Output: reports, tables, files
                   └───────────────┘
```

### Real-World Examples of Batch Processing 🌍

| Use Case | What Happens | Schedule |
|----------|-------------|----------|
| 🏦 Bank statements | Process all daily transactions | Nightly |
| 📊 Sales dashboards | Aggregate yesterday's sales data | Every morning |
| 🚕 Taxi analytics | Calculate trip stats for a month | Monthly |
| 📧 Email reports | Generate weekly user engagement stats | Weekly |
| 🧾 Payroll | Process employee hours and salaries | Bi-weekly |

### Batch vs. Stream Processing: When to Use What?

| Feature | Batch Processing 🧺 | Stream Processing 🚿 |
|---------|---------------------|----------------------|
| **Latency** | Minutes to hours | Milliseconds to seconds |
| **Data volume** | Very large (TBs/PBs) | Small increments |
| **Complexity** | Simpler to build | More complex |
| **Use case** | Reports, analytics, ML training | Fraud detection, live dashboards |
| **Tools** | Spark, Hadoop, Hive | Kafka, Flink, Spark Streaming |
| **Cost** | Cheaper (runs periodically) | More expensive (always running) |
| **Error handling** | Easy to re-run | Hard to reprocess |

> 💡 **Key insight:** Most data engineering work (roughly 80%) is batch processing. It's simpler, cheaper, and good enough for the vast majority of use cases. Only use streaming when you truly need real-time results.

### Why Apache Spark? 🔥

Apache Spark is the **most popular engine** for batch processing. Here's why:

```
Traditional Processing (Single Machine):
┌─────────────────────────────┐
│         One computer        │
│   Processing 100GB of data  │
│   ⏱️ Takes: 4 hours         │
└─────────────────────────────┘

Spark (Distributed Processing):
┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│ Machine 1│ │ Machine 2│ │ Machine 3│ │ Machine 4│
│   25GB   │ │   25GB   │ │   25GB   │ │   25GB   │
│  1 hour  │ │  1 hour  │ │  1 hour  │ │  1 hour  │
└──────────┘ └──────────┘ └──────────┘ └──────────┘
                     ⏱️ Total: ~1 hour
```

**Key advantages of Spark:**

| Feature | Why It Matters |
|---------|---------------|
| ⚡ **Speed** | Processes data in-memory (100x faster than MapReduce) |
| 📈 **Scalability** | Scales from 1 laptop to 1000s of machines |
| 🐍 **PySpark** | Write Spark jobs in Python (also supports Scala, Java, R) |
| 📦 **Unified engine** | Batch, streaming, SQL, ML — all in one framework |
| 🆓 **Open source** | Free to use, massive community |
| 🏢 **Industry standard** | Used by Netflix, Uber, Airbnb, and thousands more |

### The Spark Ecosystem 🌐

```
┌─────────────────────────────────────────────────────────────┐
│                     APACHE SPARK                             │
│                                                              │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌──────────┐│
│  │ Spark SQL │  │ Spark     │  │  MLlib    │  │ GraphX   ││
│  │           │  │ Streaming │  │           │  │          ││
│  │ DataFrames│  │           │  │ Machine   │  │ Graph    ││
│  │ & SQL     │  │ Real-time │  │ Learning  │  │ Analysis ││
│  └───────────┘  └───────────┘  └───────────┘  └──────────┘│
│                                                              │
│  ┌──────────────────────────────────────────────────────────┐│
│  │                    SPARK CORE                             ││
│  │          (Task scheduling, memory management,             ││
│  │           fault recovery, storage access)                 ││
│  └──────────────────────────────────────────────────────────┘│
│                                                              │
│  APIs: Python (PySpark) | Scala | Java | R                   │
└─────────────────────────────────────────────────────────────┘
```

In this module we focus on **Spark SQL & DataFrames** using **PySpark** — the most common combo for data engineering batch work.

---

## Part 2: Setting Up Spark & PySpark 🚀

### What You Need

Before Spark can run, you need two things:

1. **Java** — Spark is built on the JVM (Java Virtual Machine)
2. **PySpark** — The Python API for Spark

```
How the pieces connect:

    ┌──────────────┐
    │  Your Python │    You write Python code
    │    Script    │
    └──────┬───────┘
           │
           ▼
    ┌──────────────┐
    │   PySpark    │    Translates Python → Spark commands
    │  (Python     │
    │   library)   │
    └──────┬───────┘
           │
           ▼
    ┌──────────────┐
    │  Spark Core  │    The engine that does the work
    │   (JVM)      │
    └──────┬───────┘
           │
           ▼
    ┌──────────────┐
    │    Java 17   │    The runtime that Spark needs
    │   (JDK)      │
    └──────────────┘
```

### Step-by-Step Installation 🛠️

#### Option A: Linux (Ubuntu/Debian)

```bash
# Step 1: Install Java 17
sudo apt update
sudo apt install openjdk-17-jdk -y

# Verify Java
java -version
# Expected: openjdk version "17.x.x"

# Step 2: Install PySpark
pip install pyspark

# Verify PySpark
python -c "import pyspark; print(pyspark.__version__)"
# Expected: 4.0.1
```

#### Option B: macOS

```bash
# Step 1: Install Java 17
brew install openjdk@17

# Step 2: Install PySpark
pip install pyspark
# or with uv:
uv add pyspark
```

#### Option C: Windows (via WSL)

Use Windows Subsystem for Linux and follow the Linux instructions above.

> 🎉 **Great news for Spark 4.x users:** PySpark 4.x bundles its own Spark distribution! No more manual Spark binary downloads or setting `SPARK_HOME`. Just install PySpark and you're good to go.

### Your First Spark Session ✨

Every PySpark program starts by creating a **SparkSession** — it's the entry point to all Spark functionality:

```python
from pyspark.sql import SparkSession

# Create a Spark session
spark = SparkSession.builder \
    .master("local[*]") \
    .appName("MyFirstSparkApp") \
    .getOrCreate()

# Check it's working
print(f"Spark version: {spark.version}")
# Output: 4.0.1
```

**Let's break this down:**

| Part | What It Does |
|------|-------------|
| `SparkSession.builder` | Starts building a session configuration |
| `.master("local[*]")` | Run locally using ALL available CPU cores |
| `.appName("MyFirstSparkApp")` | Give this app a name (shows in Spark UI) |
| `.getOrCreate()` | Create a new session or reuse an existing one |

**Understanding `master()` settings:**

| Value | Meaning |
|-------|---------|
| `local` | Use 1 CPU core |
| `local[4]` | Use exactly 4 cores |
| `local[*]` | Use ALL available cores |
| `spark://host:7077` | Connect to a real Spark cluster |

> 💡 **Beginner tip:** Always use `local[*]` for local development. When you deploy to a cluster, you'd change this to the cluster URL.

### Stopping a Session 🛑

Always stop your Spark session when you're done to free up resources:

```python
spark.stop()
```

---

## Part 3: Working with DataFrames & Parquet 📊

### What is a DataFrame?

If you've used **pandas**, you already know what a DataFrame is — it's a table with rows and columns. Spark DataFrames work the same way, but distributed across multiple machines:

```
Pandas DataFrame (lives on ONE machine):
┌─────────────┬──────────┬──────────┐
│ pickup_time │ distance │   fare   │
├─────────────┼──────────┼──────────┤
│ 2025-11-01  │    3.5   │  $15.00  │
│ 2025-11-01  │    1.2   │   $8.50  │
│ 2025-11-02  │    7.8   │  $32.00  │
│     ...     │   ...    │   ...    │  ← All data on one machine
└─────────────┴──────────┴──────────┘

Spark DataFrame (lives across MULTIPLE machines):
┌───────────────┐  ┌───────────────┐  ┌───────────────┐
│  Partition 1  │  │  Partition 2  │  │  Partition 3  │
│  (Machine A)  │  │  (Machine B)  │  │  (Machine C)  │
├───────────────┤  ├───────────────┤  ├───────────────┤
│ 2025-11-01... │  │ 2025-11-10... │  │ 2025-11-20... │
│ 2025-11-02... │  │ 2025-11-11... │  │ 2025-11-21... │
│     ...       │  │     ...       │  │     ...       │
└───────────────┘  └───────────────┘  └───────────────┘
                 ↕ Processed in parallel! ↕
```

**Key differences from pandas:**

| Feature | Pandas | Spark DataFrame |
|---------|--------|-----------------|
| **Data size** | GBs (limited by RAM) | TBs/PBs (distributed) |
| **Execution** | Immediate (eager) | Lazy (waits until action) |
| **Location** | Single machine | Distributed across cluster |
| **Mutability** | Mutable (change in place) | Immutable (create new DF) |
| **API** | `df.column` | `df.select("column")` |

### What is Parquet? 📄

**Parquet** is the go-to file format for big data. Think of it as "CSV on steroids":

```
CSV File (Row-based):
┌──────────────────────────────────────┐
│ name,   age, city                    │  ← Row 1: all columns together
│ Alice,  30,  NYC                     │  ← Row 2: all columns together
│ Bob,    25,  LA                      │  ← Row 3: all columns together
└──────────────────────────────────────┘
❌ To read just "age", must scan entire file

Parquet File (Column-based):
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│    name      │ │     age      │ │    city      │
├──────────────┤ ├──────────────┤ ├──────────────┤
│ Alice        │ │     30       │ │    NYC       │
│ Bob          │ │     25       │ │    LA        │
└──────────────┘ └──────────────┘ └──────────────┘
✅ To read just "age", only reads that column!
```

**Why Parquet is better for big data:**

| Feature | CSV | Parquet |
|---------|-----|---------|
| 📏 **Size** | Large (no compression) | Much smaller (compressed) |
| ⚡ **Read speed** | Slow (reads all columns) | Fast (reads only needed columns) |
| 📋 **Schema** | No schema info | Schema embedded in file |
| 🔢 **Data types** | Everything is text | Proper types (int, timestamp, etc.) |
| 🔀 **Splittable** | Not easily | Yes (parallel reads) |

### Reading Data into Spark 📥

```python
# Read a Parquet file
df = spark.read.parquet("yellow_tripdata_2025-11.parquet")

# Read a CSV file
df_csv = spark.read \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .csv("taxi_zone_lookup.csv")
```

**Common read options:**

| Option | What It Does | When to Use |
|--------|-------------|-------------|
| `header` | First row contains column names | CSV files |
| `inferSchema` | Auto-detect data types | When you don't specify schema |
| `mode` | How to handle malformed data | `PERMISSIVE`, `DROPMALFORMED`, `FAILFAST` |

### Exploring Your Data 🔍

Once the data is loaded, here are the most useful commands:

```python
# See the schema (column names and types)
df.printSchema()
# Output:
# root
#  |-- VendorID: integer
#  |-- tpep_pickup_datetime: timestamp
#  |-- tpep_dropoff_datetime: timestamp
#  |-- passenger_count: integer
#  |-- trip_distance: double
#  |-- fare_amount: double
#  ...

# See the first few rows
df.show(5)

# Count the total number of rows
print(f"Total rows: {df.count()}")

# Get column names
print(df.columns)

# Statistical summary
df.describe().show()
```

### Writing Data (Saving Results) 💾

```python
# Write to Parquet (most common)
df.write.parquet("output/data.parquet", mode="overwrite")

# Write to CSV
df.write.option("header", "true").csv("output/data.csv")
```

**Write modes:**

| Mode | Behavior |
|------|----------|
| `overwrite` | Delete existing data and write new |
| `append` | Add to existing data |
| `ignore` | Skip if data already exists |
| `error` (default) | Throw error if data exists |

### Repartitioning: Controlling Parallelism 🔀

**Repartitioning** controls how many files Spark creates when writing data. It's crucial for performance:

```python
# Repartition to exactly 4 partitions
df_repartitioned = df.repartition(4)

# Save — this creates exactly 4 .parquet files
df_repartitioned.write.parquet("output/", mode="overwrite")
```

**Why repartition?**

```
Too few partitions (1 file):
┌──────────────────────────────────┐
│        100GB in ONE file         │  ← Only 1 core can read this
│        ⏱️ Very slow reads         │
└──────────────────────────────────┘

Too many partitions (10,000 files):
┌──┐┌──┐┌──┐┌──┐┌──┐┌──┐   ...    │  ← Overhead from managing 10K files
│10││10││10││10││10││10│          │     (lots of small files = slow)
└──┘└──┘└──┘└──┘└──┘└──┘          │

Just right (4-8 files for local work):
┌─────────┐┌─────────┐┌─────────┐┌─────────┐
│  ~25GB  ││  ~25GB  ││  ~25GB  ││  ~25GB  │  ← Each core gets 1 partition
│ Core 1  ││ Core 2  ││ Core 3  ││ Core 4  │
└─────────┘└─────────┘└─────────┘└─────────┘
```

**Rules of thumb:**

| Scenario | Recommended Partitions |
|----------|----------------------|
| Local development | 4-8 (match your CPU cores) |
| Small cluster | 2-4× number of cores |
| Large cluster | Target ~128 MB per partition |

---

## Part 4: Transformations & Actions — How Spark Executes 🧠

### Lazy Evaluation: Spark's Secret Weapon

One of the most important things to understand about Spark is **lazy evaluation**. Spark doesn't actually DO anything until it absolutely has to:

```
Your code:                          What Spark does:

df = spark.read.parquet(...)        → "I'll remember to read this" (no action yet)
df2 = df.filter(fare > 0)          → "I'll remember to filter" (still no action)
df3 = df2.select("fare", "tip")    → "I'll remember to select" (still waiting...)
df3.count()                         → "OK, NOW I have to do something!"
                                       Spark reads → filters → selects → counts
                                       ALL AT ONCE, optimized!
```

This is powerful because Spark can **optimize the entire plan** before executing it.

### Transformations vs. Actions

Every Spark operation falls into one of two categories:

| | Transformations 🔄 | Actions ⚡ |
|--|-------------------|-----------|
| **What** | Define WHAT to do | Actually DO it |
| **When run** | Never (just a plan) | Immediately |
| **Returns** | New DataFrame | A result (number, list, etc.) |
| **Examples** | `filter`, `select`, `groupBy`, `join`, `withColumn` | `count`, `show`, `collect`, `write`, `first` |

```python
# 🔄 Transformations (lazy — nothing happens yet)
df_filtered = df.filter(df.fare_amount > 0)          # Just a plan
df_selected = df_filtered.select("fare_amount")       # Still just a plan
df_grouped = df_selected.groupBy("fare_amount").count()  # Yep, still planning

# ⚡ Action (triggers execution of ALL the above!)
result = df_grouped.show()  # NOW everything runs!
```

### Essential DataFrame Operations 🔧

Here are the operations you'll use most often:

#### Selecting Columns

```python
# Select specific columns
df.select("tpep_pickup_datetime", "fare_amount", "tip_amount").show(5)

# Select with alias (rename)
from pyspark.sql import functions as F
df.select(
    F.col("tpep_pickup_datetime").alias("pickup_time"),
    F.col("fare_amount").alias("fare")
).show(5)
```

#### Filtering Rows

```python
from pyspark.sql import functions as F

# Simple filter
df.filter(df.fare_amount > 10).show(5)

# Multiple conditions (use & for AND, | for OR)
df.filter(
    (df.fare_amount > 10) & (df.trip_distance > 2)
).show(5)

# Date filtering
df.filter(
    F.to_date("tpep_pickup_datetime") == "2025-11-15"
).show(5)
```

#### Adding New Columns

```python
from pyspark.sql import functions as F

# Add a calculated column
df_with_duration = df.withColumn(
    "duration_hours",
    (F.unix_timestamp("tpep_dropoff_datetime") - F.unix_timestamp("tpep_pickup_datetime")) / 3600
)

# Add a constant column
df_with_label = df.withColumn("source", F.lit("yellow"))

# Add a conditional column
df_with_category = df.withColumn(
    "trip_type",
    F.when(df.trip_distance > 10, "long")
     .when(df.trip_distance > 3, "medium")
     .otherwise("short")
)
```

#### Aggregations

```python
from pyspark.sql import functions as F

# Basic aggregations
df.select(
    F.count("*").alias("total_trips"),
    F.avg("fare_amount").alias("avg_fare"),
    F.sum("fare_amount").alias("total_revenue"),
    F.max("trip_distance").alias("max_distance"),
    F.min("fare_amount").alias("min_fare")
).show()

# Group by and aggregate
df.groupBy("VendorID").agg(
    F.count("*").alias("trip_count"),
    F.avg("fare_amount").alias("avg_fare")
).show()
```

#### Sorting

```python
# Sort ascending (default)
df.orderBy("fare_amount").show(5)

# Sort descending
df.orderBy(F.desc("fare_amount")).show(5)

# Sort by multiple columns
df.orderBy(F.desc("fare_amount"), "trip_distance").show(5)
```

### PySpark Functions Cheat Sheet 📝

The `pyspark.sql.functions` module (commonly imported as `F`) provides all built-in functions:

| Function | What It Does | Example |
|----------|-------------|---------|
| `F.col("name")` | Reference a column | `F.col("fare")` |
| `F.lit(value)` | Create a column with a constant | `F.lit("yellow")` |
| `F.to_date(col)` | Convert timestamp → date | `F.to_date("pickup_dt")` |
| `F.unix_timestamp(col)` | Convert to seconds since epoch | `F.unix_timestamp("pickup_dt")` |
| `F.when(cond, val)` | Conditional logic (like IF) | `F.when(df.x > 5, "high")` |
| `F.count(col)` | Count values | `F.count("*")` |
| `F.sum(col)` | Sum of values | `F.sum("fare")` |
| `F.avg(col)` | Average | `F.avg("fare")` |
| `F.max(col)` / `F.min(col)` | Max / Min value | `F.max("distance")` |
| `F.round(col, n)` | Round to n decimals | `F.round("fare", 2)` |
| `F.year(col)` / `F.month(col)` | Extract date parts | `F.year("pickup_dt")` |

---

## Part 5: Spark SQL & Joins 🔗

### Spark SQL: Write SQL Instead of Python

If you're more comfortable with SQL, Spark lets you run full SQL queries. First, register your DataFrame as a temporary "view" (like a temporary table):

```python
# Register DataFrame as a SQL view
df.createOrReplaceTempView("trips")

# Now you can write SQL!
result = spark.sql("""
    SELECT
        COUNT(*) AS total_trips,
        AVG(fare_amount) AS avg_fare,
        MAX(trip_distance) AS max_distance
    FROM trips
    WHERE fare_amount > 0
""")

result.show()
```

**When to use SQL vs. DataFrame API?**

| SQL 📝 | DataFrame API 🐍 |
|--------|------------------|
| Quick exploration | Complex transformations |
| Already know SQL | Need Python logic |
| Simple joins & aggregations | Chaining many operations |
| Sharing with SQL-speaking colleagues | Building reusable functions |

> 💡 **Under the hood**, both approaches generate the exact same execution plan — there's zero performance difference!

### Joining DataFrames 🔗

Joins are how you combine data from multiple sources. In the taxi dataset, we join trips with the zone lookup to get human-readable location names:

```python
# Load zone data
df_zones = spark.read \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .csv("taxi_zone_lookup.csv")

# Join trips with zones on pickup location
df_with_zones = df_trips.join(
    df_zones,
    df_trips.PULocationID == df_zones.LocationID,
    "inner"
)

df_with_zones.select("tpep_pickup_datetime", "Zone", "fare_amount").show(5)
```

**Or using Spark SQL:**

```python
df_zones.createOrReplaceTempView("zones")
df_trips.createOrReplaceTempView("trips")

spark.sql("""
    SELECT
        t.tpep_pickup_datetime,
        z.Zone,
        t.fare_amount
    FROM trips t
    JOIN zones z ON t.PULocationID = z.LocationID
    LIMIT 5
""").show(truncate=False)
```

### Types of Joins

```
INNER JOIN (default):                 LEFT JOIN:
Only matching rows                    All from left + matches from right

 Table A    Table B                   Table A    Table B
┌───┬───┐  ┌───┬───┐                ┌───┬───┐  ┌───┬───┐
│ 1 │ a │  │ 1 │ x │                │ 1 │ a │  │ 1 │ x │
│ 2 │ b │  │ 3 │ y │                │ 2 │ b │  │ 3 │ y │
│ 3 │ c │  └───┴───┘                │ 3 │ c │  └───┴───┘
└───┴───┘                           └───┴───┘

Result:                              Result:
┌───┬───┬───┐                        ┌───┬───┬──────┐
│ 1 │ a │ x │                        │ 1 │ a │  x   │
│ 3 │ c │ y │                        │ 2 │ b │ NULL │  ← No match
└───┴───┴───┘                        │ 3 │ c │  y   │
                                     └───┴───┴──────┘
```

| Join Type | PySpark Keyword | What It Returns |
|-----------|----------------|-----------------|
| Inner | `"inner"` | Only rows that match in BOTH tables |
| Left | `"left"` or `"left_outer"` | All rows from left + matches from right |
| Right | `"right"` or `"right_outer"` | All rows from right + matches from left |
| Full Outer | `"full"` or `"outer"` | All rows from both tables |
| Cross | `"cross"` | Every combination (cartesian product) — use carefully! |

### Practical Example: Finding the Least Frequent Pickup Zone 🗺️

This is a classic join + aggregation pattern. We want to know which zone has the fewest taxi pickups:

```python
# Using DataFrame API
from pyspark.sql import functions as F

result = df_trips.join(
    df_zones,
    df_trips.PULocationID == df_zones.LocationID,
    "inner"
).groupBy("Zone") \
 .agg(F.count("*").alias("pickup_count")) \
 .orderBy("pickup_count") \
 .limit(5)

result.show(truncate=False)
```

```python
# Same thing using Spark SQL
spark.sql("""
    SELECT
        z.Zone,
        COUNT(1) AS pickup_count
    FROM trips t
    JOIN zones z ON t.PULocationID = z.LocationID
    GROUP BY z.Zone
    ORDER BY pickup_count ASC
    LIMIT 5
""").show(truncate=False)
```

Both approaches give the same result — **Governor's Island/Ellis Island/Liberty Island** is the least frequent pickup zone (it's mainly ferry-accessible, so almost no one takes a taxi from there 🚢).

---

## Part 6: Spark Internals & Monitoring 🔍

### How Spark Executes Your Code

When you trigger an action (like `.count()` or `.show()`), Spark goes through several stages:

```
Your Code
    │
    ▼
┌─────────────────────┐
│  1. LOGICAL PLAN    │   "What you asked for"
│     (Unresolved)    │   (column names, operations)
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. ANALYZED PLAN   │   "Validated against schema"
│     (Resolved)      │   (check columns exist, types match)
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. OPTIMIZED PLAN  │   "Spark's optimizer (Catalyst) improves it"
│     (Catalyst)      │   (push filters down, reorder joins, etc.)
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. PHYSICAL PLAN   │   "Actual execution steps"
│     (How to do it)  │   (which algorithms, which partitions)
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. EXECUTION       │   "Run it across the cluster!"
│     (Tasks on cores)│
└─────────────────────┘
```

You can see the execution plan using `.explain()`:

```python
df.filter(df.fare_amount > 10).groupBy("VendorID").count().explain()
```

### Spark Architecture: Driver & Executors 🏗️

```
┌───────────────────────────────────────────────────────┐
│                    SPARK APPLICATION                    │
│                                                        │
│   ┌─────────────┐                                     │
│   │   DRIVER    │  ← Your program runs here            │
│   │  (Master)   │  ← Creates the execution plan        │
│   │             │  ← Distributes work to executors      │
│   └──────┬──────┘                                     │
│          │                                             │
│    ┌─────┼─────────────────┐                          │
│    │     │                 │                           │
│    ▼     ▼                 ▼                           │
│ ┌──────┐ ┌──────┐    ┌──────┐                        │
│ │EXEC 1│ │EXEC 2│    │EXEC 3│  ← Workers that         │
│ │      │ │      │    │      │    process data          │
│ │Task 1│ │Task 2│    │Task 3│                          │
│ │Task 4│ │Task 5│    │Task 6│  ← Each runs tasks       │
│ └──────┘ └──────┘    └──────┘    in parallel           │
│                                                        │
└───────────────────────────────────────────────────────┘
```

**In local mode** (`local[*]`), the driver AND executors all run on your one machine. In cluster mode, executors run on separate machines.

| Component | Role |
|-----------|------|
| 🧠 **Driver** | The "brain" — coordinates everything, builds the plan |
| 💪 **Executor** | The "muscle" — processes data partitions |
| 📋 **Task** | A unit of work on one partition |
| 📊 **Job** | All the work needed for one action (e.g., `.count()`) |
| 🔢 **Stage** | A set of tasks that can run without shuffling data |

### The Spark UI: Your Dashboard 📺

When Spark is running, it serves a web UI at **`http://localhost:4040`** that shows:

```
┌─────────────────────────────────────────────────────────┐
│                  SPARK UI (port 4040)                     │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  📋 Jobs Tab                                              │
│  └── See all jobs, their stages, and status              │
│                                                           │
│  🔢 Stages Tab                                            │
│  └── Detailed view of each stage's tasks                 │
│                                                           │
│  💾 Storage Tab                                           │
│  └── Cached DataFrames and their memory usage            │
│                                                           │
│  🌐 Environment Tab                                       │
│  └── Spark configuration and system properties           │
│                                                           │
│  🔧 Executors Tab                                         │
│  └── Memory and disk usage per executor                  │
│                                                           │
│  📊 SQL Tab                                               │
│  └── Execution plans and timings for SQL queries         │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

Access it in your browser:

```python
# Get the UI URL programmatically
print(spark.sparkContext.uiWebUrl)
# Output: http://localhost:4040
```

> 💡 **Tip:** If port 4040 is busy (maybe you have two Spark sessions), Spark will use 4041, 4042, etc.

### Performance Tips for Beginners ⚡

| Tip | Why It Helps |
|-----|-------------|
| 🗂️ **Use Parquet, not CSV** | Columnar format = faster reads, smaller files |
| 🔀 **Repartition wisely** | Too few = underutilized cores; too many = overhead |
| 🎯 **Select only needed columns** | Less data to move = faster processing |
| 🔽 **Filter early** | Reduce data volume as soon as possible |
| 💾 **Cache repeated DataFrames** | `df.cache()` keeps it in memory for re-use |
| 📊 **Check the Spark UI** | Identify bottlenecks (data skew, slow stages) |
| 🐍 **Avoid Python UDFs when possible** | Use built-in `F.*` functions (they're way faster) |
| 📏 **Know your data size** | `df.count()` and `df.rdd.getNumPartitions()` |

### Common Beginner Mistakes ❌

| Mistake | Problem | Fix |
|---------|---------|-----|
| Calling `.collect()` on big data | Brings ALL data to driver (OOM crash!) | Use `.show()`, `.take()`, or write to file |
| Not stopping the session | Resources stay locked | Always call `spark.stop()` |
| Too many small files | Slow reads, metadata overhead | Repartition to fewer, larger files |
| Using Python loops on DataFrames | Defeats the purpose of distributed computing | Use built-in Spark operations |
| Ignoring the Spark UI | Miss performance issues | Check it regularly at port 4040 |

### Quick Reference: Complete Workflow 🗺️

Here's a typical PySpark batch processing workflow from start to finish:

```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F

# 1️⃣ Create session
spark = SparkSession.builder \
    .master("local[*]") \
    .appName("TaxiAnalysis") \
    .getOrCreate()

# 2️⃣ Read data
df = spark.read.parquet("yellow_tripdata_2025-11.parquet")

# 3️⃣ Explore
df.printSchema()
print(f"Total rows: {df.count()}")

# 4️⃣ Transform
df_clean = df.filter(df.fare_amount > 0) \
    .withColumn("duration_hours",
        (F.unix_timestamp("tpep_dropoff_datetime") - 
         F.unix_timestamp("tpep_pickup_datetime")) / 3600
    ) \
    .select("tpep_pickup_datetime", "fare_amount", "trip_distance", "duration_hours")

# 5️⃣ Analyze
df_clean.groupBy(F.to_date("tpep_pickup_datetime").alias("date")) \
    .agg(
        F.count("*").alias("trip_count"),
        F.avg("fare_amount").alias("avg_fare"),
        F.avg("duration_hours").alias("avg_duration")
    ) \
    .orderBy("date") \
    .show(10)

# 6️⃣ Save results
df_clean.repartition(4).write.parquet("output/clean_trips", mode="overwrite")

# 7️⃣ Clean up
spark.stop()
```

---

## Recap: Key Takeaways 🎓

| Concept | What to Remember |
|---------|-----------------|
| **Batch processing** | Process data in large chunks on a schedule |
| **Apache Spark** | Distributed engine for processing big data fast |
| **PySpark** | Python API for Spark — write Python, Spark distributes it |
| **SparkSession** | Entry point — always create one first |
| **DataFrames** | Like pandas tables, but distributed across machines |
| **Parquet** | Columnar file format — always prefer over CSV |
| **Repartitioning** | Controls how many files/partitions Spark uses |
| **Lazy evaluation** | Transformations are planned, actions execute them |
| **Spark SQL** | Write SQL queries directly on DataFrames |
| **Joins** | Combine datasets (trips + zones) for richer analysis |
| **Spark UI** | Dashboard at port 4040 for monitoring |

> 🚀 **Next steps:** Now that you understand batch processing with Spark, the next module covers streaming — where data is processed in real-time instead of in batches!
