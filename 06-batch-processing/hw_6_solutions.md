# Module 6 Homework Solutions - Batch Processing with Apache Spark

## Overview

This homework puts into practice what we learned about batch processing using Apache Spark and PySpark. We work with NYC Yellow Taxi trip data from November 2025, performing tasks like setting up Spark, reading/writing Parquet files, filtering, calculating trip durations, and joining datasets.

**Key Topics:**
- Installing and configuring PySpark 4.x
- Creating a local Spark session
- Reading and writing Parquet files
- Repartitioning DataFrames for parallel processing
- Filtering and aggregating data with PySpark
- Using Spark SQL for joins
- Spark UI for monitoring

---

## Setup

### Prerequisites

1. **Java 17** installed (required by Spark 4.x)
2. **Python 3.10+** with a package manager (`pip`, `uv`, or `conda`)
3. **PySpark 4.x** — now bundles its own Spark distribution, so no manual Spark download or `SPARK_HOME` setup is needed

### Installation (Quick Start)

```bash
# Install Java 17 (Ubuntu/Debian)
sudo apt install openjdk-17-jdk

# Or on macOS with Homebrew
brew install openjdk@17

# Install PySpark
pip install pyspark
```

### Dataset

Download the Yellow Taxi trip data for November 2025:

```bash
wget https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2025-11.parquet
```

For Question 6, also download the taxi zone lookup:

```bash
wget https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv
```

---

## Questions & Answers

### Question 1. Install Spark and PySpark

**Question:** Install Spark, run PySpark, create a local Spark session, and execute `spark.version`. What's the output?

**Answer:** `4.0.1`

With PySpark 4.x, setup is significantly simpler than older versions — there's no longer a need to manually download Spark binaries or configure `SPARK_HOME`. After installing Java and PySpark, we can verify everything by creating a session and checking the version:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .master("local[*]") \
    .appName("test") \
    .getOrCreate()

print(spark.version)
# Output: 4.0.1
```

The `local[*]` master setting tells Spark to run locally using all available CPU cores on the machine.

---

### Question 2. Yellow November 2025

**Question:** Read the November 2025 Yellow data into a Spark DataFrame. Repartition the DataFrame to 4 partitions and save it to Parquet. What is the average size of the Parquet files created (in MB)?

- 6MB
- 25MB
- 75MB
- 100MB

**Answer:** `25MB`

Repartitioning controls how data is split across files when writing, which directly impacts parallelism during reads. With 4 partitions, Spark writes 4 separate `.parquet` files:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .master("local[*]") \
    .appName("Q2_Repartition") \
    .getOrCreate()

# Read the original parquet file
df = spark.read.parquet("yellow_tripdata_2025-11.parquet")

# Repartition to 4 partitions and write out
output_path = "data/pq/yellow/2025/11/"
df.repartition(4).write.parquet(output_path, mode="overwrite")
```

Check the output file sizes:

```bash
ls -lh data/pq/yellow/2025/11/*.parquet
```

Each of the 4 generated `.parquet` files is approximately **25 MB** in size.

---

### Question 3. Count Records

**Question:** How many taxi trips were there on the 15th of November? Consider only trips that started on the 15th of November.

- 62,610
- 102,340
- 162,604
- 225,768

**Answer:** `162,604`

Since the `tpep_pickup_datetime` column contains full timestamps (date + time), we cast it to a date before filtering to ensure we capture all trips regardless of what time of day they started:

```python
from pyspark.sql import functions as F

# Load the repartitioned data
df_trips = spark.read.parquet("data/pq/yellow/2025/11/")

# Filter for trips starting on November 15, 2025
trips_nov_15 = df_trips.filter(
    F.to_date(df_trips.tpep_pickup_datetime) == "2025-11-15"
).count()

print(f"Total trips on November 15, 2025: {trips_nov_15}")
# Output: 162604
```

`F.to_date()` extracts only the date portion from the timestamp, so a trip at `2025-11-15 23:59:00` is correctly included.

---

### Question 4. Longest Trip

**Question:** What is the length of the longest trip in the dataset in hours?

- 22.7
- 58.2
- 90.6
- 134.5

**Answer:** `90.6`

To calculate trip duration, we convert both pickup and dropoff timestamps to Unix timestamps (seconds since epoch), take the difference, and convert to hours by dividing by 3600:

```python
from pyspark.sql import functions as F

# Calculate trip duration in hours
df_with_duration = df_trips.withColumn(
    "duration_hours",
    (F.unix_timestamp("tpep_dropoff_datetime") - F.unix_timestamp("tpep_pickup_datetime")) / 3600
)

# Find the maximum duration
longest_trip = df_with_duration.select(F.max("duration_hours")).first()[0]
print(f"Longest trip duration: {longest_trip:.1f} hours")
# Output: 90.6 hours
```

This approach is more reliable than subtracting timestamps directly because `unix_timestamp` gives a clean numeric value in seconds, making the arithmetic straightforward.

---

### Question 5. User Interface

**Question:** Spark's User Interface which shows the application's dashboard runs on which local port?

- 80
- 443
- 4040
- 8080

**Answer:** `4040`

When a `SparkSession` is initialized, Spark automatically launches a Web UI for monitoring and inspecting the application. This dashboard shows Jobs, Stages, Tasks, storage usage, and the execution DAG.

By default, the UI is hosted on port **4040**. If that port is already in use (e.g., by another active Spark context), Spark increments the port to 4041, 4042, and so on.

You can verify the active UI URL programmatically:

```python
print(spark.sparkContext.uiWebUrl)
# Output: http://localhost:4040 (or similar)
```

---

### Question 6. Least Frequent Pickup Location Zone

**Question:** Load the zone lookup data into a temp view in Spark. Using the zone lookup data and the Yellow November 2025 data, what is the name of the LEAST frequent pickup location zone?

- Governor's Island/Ellis Island/Liberty Island
- Arden Heights
- Rikers Island
- Jamaica Bay

**Answer:** `Governor's Island/Ellis Island/Liberty Island`

To find the least frequent pickup zone, we join the trips data with the zone lookup table on `PULocationID = LocationID`, group by zone name, count occurrences, and sort ascending:

```python
# Load the zone lookup CSV
df_zones = spark.read \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .csv("taxi_zone_lookup.csv")

# Register both DataFrames as temporary SQL views
df_zones.createOrReplaceTempView("zones")
df_trips.createOrReplaceTempView("trips")

# Find the least frequent pickup zone
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

The query returns **Governor's Island/Ellis Island/Liberty Island** as the zone with the fewest pickups. This makes sense — it's a tourist destination primarily accessed by ferry, not taxi.

---

## Summary

| Question | Answer |
|----------|--------|
| Q1. Install Spark and PySpark | `4.0.1` |
| Q2. Yellow November 2025 (avg Parquet size) | `25MB` |
| Q3. Count Records (Nov 15) | `162,604` |
| Q4. Longest Trip | `90.6 hours` |
| Q5. User Interface Port | `4040` |
| Q6. Least Frequent Pickup Zone | `Governor's Island/Ellis Island/Liberty Island` |

---

## Submission

Form for submitting: https://courses.datatalks.club/de-zoomcamp-2026/homework/hw6
