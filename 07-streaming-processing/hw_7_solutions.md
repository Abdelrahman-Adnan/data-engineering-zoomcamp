# Module 7 Homework Solutions - Streaming with Redpanda and PyFlink

## Overview

This homework focuses on streaming fundamentals using Redpanda (Kafka-compatible) and PyFlink.

We work with `green_tripdata_2025-10.parquet` and cover:
- Sending events to a Kafka topic
- Consuming and filtering streaming records
- Building windowed aggregations with PyFlink
- Writing streaming results into PostgreSQL

---

## Setup

Use the same infrastructure from the module workshop:

```bash
cd 07-streaming/workshop/
docker compose build
docker compose up -d
```

This brings up:
- Redpanda on `localhost:9092`
- Flink JobManager on `http://localhost:8081`
- Flink TaskManager
- PostgreSQL on `localhost:5432` (`postgres/postgres`)

If you have old containers or volumes, clean and restart:

```bash
docker compose down -v
docker compose build
docker compose up -d
```

---

## Questions & Answers

### Question 1. Redpanda version

**Question:** What version of Redpanda are you running?

Command used:

```bash
docker exec -it workshop-redpanda-1 rpk version
```

**Answer:** `v25.3.9`

---

### Question 2. Sending data to Redpanda

**Question:** How long did it take to send all records and flush the producer?

Options:
- 10 seconds
- 60 seconds
- 120 seconds
- 300 seconds

**Answer:** `10 seconds`

I created the topic and produced the Green Taxi records with a Python producer. Datetime fields were converted to strings before JSON serialization.

```bash
docker exec -it workshop-redpanda-1 rpk topic create green-trips
```

```python
import json
from time import time

import pandas as pd
from kafka import KafkaProducer

url = "https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2025-10.parquet"
columns = [
    "lpep_pickup_datetime",
    "lpep_dropoff_datetime",
    "PULocationID",
    "DOLocationID",
    "passenger_count",
    "trip_distance",
    "tip_amount",
    "total_amount",
]

# Read only required columns
df = pd.read_parquet(url, columns=columns)

# Convert datetime columns to strings for JSON
df["lpep_pickup_datetime"] = df["lpep_pickup_datetime"].astype(str)
df["lpep_dropoff_datetime"] = df["lpep_dropoff_datetime"].astype(str)

producer = KafkaProducer(
    bootstrap_servers=["localhost:9092"],
    value_serializer=lambda v: json.dumps(v).encode("utf-8"),
)

topic_name = "green-trips"
records = df.to_dict(orient="records")

t0 = time()

for row in records:
    producer.send(topic_name, value=row)

producer.flush()

t1 = time()
print(f"took {(t1 - t0):.2f} seconds")
```

Measured runtime was closest to **10 seconds**.

---

### Question 3. Consumer - trip distance

**Question:** How many trips have `trip_distance > 5`?

Options:
- 6506
- 7506
- 8506
- 9506

**Answer:** `8506`

I consumed all messages from the beginning of the topic (`auto_offset_reset='earliest'`) and counted rows where `trip_distance > 5.0`.

```python
import json
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    "green-trips",
    bootstrap_servers=["localhost:9092"],
    auto_offset_reset="earliest",
    group_id="hw7-trip-distance-counter",
    value_deserializer=lambda m: json.loads(m.decode("utf-8")),
    consumer_timeout_ms=5000,
)

count = 0
for message in consumer:
    ride = message.value
    if float(ride.get("trip_distance", 0.0)) > 5.0:
        count += 1

print(count)
```

Output: **8506**

---

## Part 2: PyFlink

For Questions 4-6, I adapted the workshop job pattern to this homework dataset using:
- topic: `green-trips`
- timestamp field from string datetime: `lpep_pickup_datetime`
- watermark: 5 seconds
- `env.set_parallelism(1)` because the topic has a single partition

---

### Question 4. Tumbling window - pickup location

**Question:** Which `PULocationID` had the most trips in a single 5-minute window?

Options:
- 42
- 74
- 75
- 166

**Answer:** `74`

Approach:
1. Read stream from `green-trips` topic
2. Convert pickup datetime string into event-time timestamp
3. Use 5-minute tumbling window grouped by `PULocationID`
4. Write window counts to PostgreSQL table
5. Query top row by `num_trips`

Result query:

```sql
SELECT PULocationID, num_trips
FROM trip_counts
ORDER BY num_trips DESC
LIMIT 3;
```

Top `PULocationID` was **74**.

---

### Question 5. Session window - longest streak

**Question:** How many trips were in the longest session?

Options:
- 12
- 31
- 51
- 81

**Answer:** `81`

Approach:
1. Use event-time session window with 5-minute inactivity gap
2. Group by `PULocationID`
3. Count events per session and write to PostgreSQL
4. Sort by `num_trips` descending

Example query:

```sql
SELECT PULocationID, num_trips
FROM trip_sessions
ORDER BY num_trips DESC
LIMIT 5;
```

The largest session contained **81 trips**.

---

### Question 6. Tumbling window - largest tip

**Question:** Which hour had the highest total `tip_amount`?

Options:
- 2025-10-01 18:00:00
- 2025-10-16 18:00:00
- 2025-10-22 08:00:00
- 2025-10-30 16:00:00

**Answer:** `2025-10-16 18:00:00`

Approach:
1. Build a 1-hour tumbling window on event time
2. Aggregate with `SUM(tip_amount)` across all locations
3. Write to PostgreSQL and pick the maximum

Validation query:

```sql
SELECT window_start, total_tips
FROM hourly_tips
ORDER BY total_tips DESC
LIMIT 1;
```

Top hour by total tips: **2025-10-16 18:00:00**.

---

## Final Answers Summary

| Question | Answer |
|----------|--------|
| Q1. Redpanda version | `v25.3.9` |
| Q2. Producer send time | `10 seconds` |
| Q3. Trips with distance > 5 | `8506` |
| Q4. Top pickup location in 5-min window | `74` |
| Q5. Longest session size | `81` |
| Q6. Hour with highest total tips | `2025-10-16 18:00:00` |

---

## Submission

Form for submitting: https://courses.datatalks.club/de-zoomcamp-2026/homework/hw7
