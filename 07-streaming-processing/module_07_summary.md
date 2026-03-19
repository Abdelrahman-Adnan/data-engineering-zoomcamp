# Module 7 Summary - Stream Processing with Redpanda, Kafka, and PyFlink

#DataEngineeringZoomcamp #Streaming #Kafka #Redpanda #PyFlink #DataEngineering

---

## Part 1: Stream Processing Fundamentals 🌊

### What is stream processing?

Stream processing means handling data continuously as events arrive, instead of waiting for a full batch.

Think of it like this:

- **Batch** = process all orders at end of day
- **Streaming** = process each order immediately when it arrives

```
Batch mindset:
[events all day] -> [run job at night] -> [results]

Streaming mindset:
[event] -> [process] -> [result]
[event] -> [process] -> [result]
[event] -> [process] -> [result]
```

### When should you choose streaming?

From real-world community discussions and workshop Q&A, the key question is:

> "Will something important happen in real time on the other side?"

Use streaming when:
- You need near real-time action (fraud detection, alerting, dynamic pricing)
- Delay of minutes can cause measurable business impact
- Continuous event flow is core to your product

Use micro-batch or batch when:
- Dashboards are checked by humans (not machines)
- Hourly or daily freshness is enough
- You want simpler operations and lower on-call overhead

### Core terms you should know

| Term | Beginner-friendly meaning |
|------|---------------------------|
| Event | One record happening at a point in time |
| Topic | A named event stream in Kafka/Redpanda |
| Producer | App that sends events to a topic |
| Consumer | App that reads events from a topic |
| Offset | Position of a message in a topic partition |
| Partition | A shard of a topic for parallelism |
| Event time | Time when event happened |
| Processing time | Time when system processes event |
| Watermark | Flink hint for how late events can arrive |

### Why Module 7 matters in modern data engineering

Streaming sits between ingestion and real-time analytics:

```
Source systems -> Kafka/Redpanda -> Flink -> PostgreSQL/Warehouse -> BI/Apps
```

This module teaches exactly that flow with a practical stack:
- Redpanda as Kafka-compatible broker
- Python producer/consumer
- PyFlink for stream transformations and windows
- PostgreSQL as sink for materialized results

### A practical mental model (for interviews + real projects)

Use this 5-layer model to understand any streaming system:

1. **Source layer** – where events originate (app backend, IoT, CDC, logs)
2. **Transport layer** – event bus (Kafka/Redpanda topics)
3. **Compute layer** – stream processor (Flink jobs)
4. **Serving layer** – sink store for querying (PostgreSQL, warehouse, Elasticsearch)
5. **Consumption layer** – dashboard, alerting, ML feature serving, APIs

If you can clearly explain these 5 layers, you can describe most real-time pipelines.

### Latency vs throughput (the key trade-off)

In streaming, we constantly balance:

- **Lower latency** (faster updates)
- **Higher throughput** (process more events/second)

Typical pattern:
- Smaller windows / shorter trigger intervals -> fresher data, higher compute cost
- Larger windows / longer triggers -> cheaper, less noisy, but less real-time

### Beginner checklist for deciding “batch or stream”

Before choosing streaming, ask:

- What is the maximum acceptable delay (SLA)?
- Who uses this output: humans, or automated systems?
- What happens if data is delayed by 5-15 minutes?
- Is the team ready to operate a 24/7 pipeline?
- Do we need replay/backfill guarantees?

If these answers are unclear, start with micro-batch first and evolve later.

### Scenario: Food delivery app notifications 🍔

Let’s make this concrete with a beginner scenario.

Suppose you run a food delivery app and you want to show users live order status:

- Order accepted
- Driver assigned
- Driver near you
- Delivered

If this status updates every 30 minutes (batch), users have a bad experience.
Streaming is a better fit because each event should appear almost immediately.

Pipeline in plain words:

1. App backend emits events (order status changes)
2. Events are sent to Kafka/Redpanda topic `order-status-events`
3. Flink enriches events (e.g., add city, ETA category)
4. Results are stored in a serving DB
5. Mobile app reads updated status in near real time

This is the “why” behind learning Module 7.

---

## Part 2: Kafka and Redpanda Architecture for Beginners 🧱

### Redpanda vs Kafka (practical view)

In this module, Redpanda is used as a **Kafka-compatible broker**.
That means Kafka client libraries work without code changes.

Why this is useful:
- Faster local setup
- Familiar Kafka protocol
- Same producer/consumer concepts

### How data moves through the broker

```
Producer -> Topic (partitioned log) -> Consumer group
```

More detailed:

```
┌──────────┐       ┌─────────────────────────────┐       ┌──────────────┐
│ Producer │ ----> │ Topic: green-trips          │ ----> │ Consumer A   │
└──────────┘       │  partition-0                │       └──────────────┘
                   │  partition-1                │       ┌──────────────┐
                   │  partition-2                │ ----> │ Consumer B   │
                   └─────────────────────────────┘       └──────────────┘
```

### Important local endpoints in the workshop setup

- Redpanda broker: `localhost:9092`
- Flink UI: `http://localhost:8081`
- PostgreSQL: `localhost:5432`

### Offsets and consumer behavior (super important)

Offsets define where a consumer starts reading.

Common options:
- `earliest`: read from beginning of topic
- `latest`: read only new events

In homework and debugging, `earliest` is usually needed so you can replay all events.

### Single partition vs multiple partitions

Community notes repeatedly highlight this:

- If topic has **1 partition**, only one consumer subtask gets data
- If Flink parallelism is higher than partitions, extra subtasks are idle
- Idle subtasks may hold back global watermark progression in event-time pipelines

Practical rule:
- For `green-trips` (1 partition), set `env.set_parallelism(1)` in Flink jobs

### Topic design basics (beginner-friendly)

When creating topics, think about:

- **Name**: make it descriptive and stable (`green-trips`)
- **Keying**: choose a field that controls partitioning (e.g., `PULocationID`)
- **Retention**: how long events should stay replayable
- **Partitions**: enough for parallelism, not too many for local development

In this homework, single partition is intentionally simple for learning.

### Consumer groups in plain English

Consumer groups let multiple consumers share topic work.

- Same `group_id` -> they split partitions
- Different `group_id` -> each group gets a full copy of the stream

Example:

- `group_id=analytics-job` reads for analytics
- `group_id=fraud-detector` reads same topic independently for alerts

### Delivery semantics (high-level)

You will often hear these terms:

- **At-most-once**: may lose messages, no duplicates
- **At-least-once**: no loss, but duplicates possible
- **Exactly-once**: no loss + no duplicates (harder, more operational complexity)

Beginner strategy:
- Start with at-least-once + idempotent sink logic
- Move to stronger guarantees when required by business rules

### Scenario: Why partitions matter in practice

Imagine your pipeline gets **100,000 events/minute**.

- With `1` partition -> only one consumer instance can read that partition at a time
- With `10` partitions -> up to ten consumers in same group can read in parallel

For homework we keep `1` partition to make concepts easy.
In production, partitions are scaled based on throughput + consumer parallelism goals.

---

## Part 3: Producer and Consumer Pipeline in Python 🐍

### Step A: Create the topic

```bash
docker exec -it workshop-redpanda-1 rpk topic create green-trips
```

### Step B: Build the producer

Goal:
- Read Green Taxi parquet data
- Keep only required columns
- Convert datetime to string for JSON
- Send row-by-row to topic

Core producer pattern:

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

df = pd.read_parquet(url, columns=columns)
df["lpep_pickup_datetime"] = df["lpep_pickup_datetime"].astype(str)
df["lpep_dropoff_datetime"] = df["lpep_dropoff_datetime"].astype(str)

producer = KafkaProducer(
    bootstrap_servers=["localhost:9092"],
    value_serializer=lambda v: json.dumps(v).encode("utf-8")
)

records = df.to_dict(orient="records")

t0 = time()
for row in records:
    producer.send("green-trips", value=row)
producer.flush()
t1 = time()

print(f"took {(t1 - t0):.2f} seconds")
```

### Producer code walkthrough (line by line)

What each section of code does:

1. `import json` and `KafkaProducer`
    - We need JSON to serialize Python dictionaries
    - `KafkaProducer` sends messages to broker

2. `pd.read_parquet(url, columns=columns)`
    - Reads only required columns (faster and memory-efficient)
    - Avoids loading unnecessary fields

3. `astype(str)` on datetime columns
    - JSON serializer cannot safely encode pandas datetime objects directly
    - Converting to string prevents runtime serialization errors

4. `value_serializer=lambda v: json.dumps(v).encode("utf-8")`
    - Converts each Python dictionary into JSON bytes
    - Kafka transmits bytes, not Python objects

5. `records = df.to_dict(orient="records")`
    - Converts DataFrame rows into list of dictionaries
    - Each dictionary becomes one event message

6. `producer.send(...)` in a loop
    - Queues records into producer buffer
    - Send is asynchronous for better throughput

7. `producer.flush()`
    - Forces all buffered records to be sent before script exits
    - Without flush, some events may remain unsent

8. timing with `t0` and `t1`
    - Measures end-to-end publish duration for homework answer

### Scenario: What if producer crashes mid-send?

If producer stops before `flush()` completes:

- Some events are already in topic
- Some events are still in local buffer and are lost

This is why durable producer settings (`acks='all'`, retries) are used in production.

### Why convert datetime to string?

Raw Python datetime objects are not JSON-serializable by default.
String conversion avoids serialization errors and keeps payloads simple.

### Step C: Build a consumer

Consumer pattern for full replay and counting conditions:

```python
import json
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    "green-trips",
    bootstrap_servers=["localhost:9092"],
    auto_offset_reset="earliest",
    group_id="green-trips-debug",
    value_deserializer=lambda m: json.loads(m.decode("utf-8")),
    consumer_timeout_ms=5000
)

count = 0
for message in consumer:
    trip = message.value
    if float(trip.get("trip_distance", 0.0)) > 5.0:
        count += 1

print(count)
```

### Consumer code walkthrough (line by line)

1. `KafkaConsumer("green-trips", ...)`
    - Subscribes to one topic

2. `auto_offset_reset="earliest"`
    - If group has no committed offsets, start from oldest message
    - Perfect for replaying full dataset during learning/homework

3. `group_id="green-trips-debug"`
    - Identifies this consumer group in broker
    - Offsets are tracked per group

4. `value_deserializer=...json.loads(...)`
    - Converts bytes back into Python dictionary

5. `consumer_timeout_ms=5000`
    - Stops polling loop if no new messages for 5 seconds
    - Prevents script from hanging forever after stream is consumed

6. `if float(trip.get("trip_distance", 0.0)) > 5.0`
    - Defensive extraction of field
    - Counts only long trips required by question

### Scenario: Why your count might change unexpectedly

If you publish the same file twice without deleting/recreating topic, consumer reading from earliest will process both copies.

Result: count roughly doubles.

So for clean homework reruns:

1. delete topic
2. recreate topic
3. rerun producer
4. rerun consumer

### Beginner mistakes to avoid

- Forgetting `producer.flush()` and wondering where messages went
- Using `latest` when you intended to read old data
- Re-sending same dataset many times and accidentally duplicating topic data
- Forgetting to recreate topic before repeated homework runs

Helpful cleanup command:

```bash
docker exec -it workshop-redpanda-1 rpk topic delete green-trips
```

Then recreate topic and rerun producer.

### Production-hardening tips for producer/consumer

Even in beginner projects, it helps to know these settings:

**Producer considerations:**
- Set `acks='all'` for stronger durability
- Add retries + backoff for temporary broker/network issues
- Use message keys for consistent partition routing
- Log send failures and count dropped messages

**Consumer considerations:**
- Use explicit `group_id` naming convention per use case
- Track lag and throughput metrics
- Handle malformed JSON defensively (`try/except`)
- Commit offsets carefully (auto vs manual depends on your reliability needs)

### Recommended folder structure for local streaming projects

```text
streaming-project/
    src/
        producers/
            producer_green.py
        consumers/
            consumer_trip_distance.py
        flink_jobs/
            q4_tumbling.py
            q5_session.py
            q6_hourly_tips.py
    sql/
        create_tables.sql
    docker-compose.yml
    README.md
```

This structure keeps responsibilities clear and reduces beginner confusion.

---

## Part 4: PyFlink Basics — Event Time, Watermarks, Windows ⏱️

### Why Flink after Kafka?

Kafka stores/streams events. Flink computes continuous transformations over those events.

Kafka answers: "Can we move events reliably?"
Flink answers: "Can we compute useful stateful analytics in real time?"

### Event time vs processing time

In taxi data, the business timestamp is pickup datetime.
We should compute windows on **event time**, not machine processing time.

### Defining event time in source DDL

In this homework, timestamps arrive as strings. So we derive a proper timestamp column:

```sql
lpep_pickup_datetime VARCHAR,
event_timestamp AS TO_TIMESTAMP(lpep_pickup_datetime, 'yyyy-MM-dd HH:mm:ss'),
WATERMARK FOR event_timestamp AS event_timestamp - INTERVAL '5' SECOND
```

### What is a watermark (easy intuition)?

Watermark is Flink's estimate of how far event time has advanced.

- It allows slightly late events (here: 5 seconds)
- Events later than watermark tolerance may be considered late
- Without good watermark progression, windows may never close

### Scenario: late taxi event example

Suppose your 5-minute window is `10:00 - 10:05` and watermark delay is 5 seconds.

- Most events for that window arrive around real time
- One event with event-time `10:04:58` arrives at processing time `10:06:30`

This event is too late relative to watermark and might be dropped (or routed differently if configured).

That’s why choosing watermark tolerance is a business decision:
- too small -> you lose valid late events
- too large -> results finalize slower

### Window types you need for this module

#### 1) Tumbling window
- Fixed size
- Non-overlapping
- Example: every 5 minutes or 1 hour

#### 2) Session window
- Dynamic size
- Groups events separated by less than a gap
- Closes when inactivity exceeds gap (here: 5 minutes)

Visual intuition:

```
Tumbling (5 min):
[10:00-10:05] [10:05-10:10] [10:10-10:15]

Session (gap=5 min):
events at 10:00,10:02,10:06 -> one session
next event at 10:20 -> new session
```

### Why `PRIMARY KEY ... NOT ENFORCED` in JDBC sink?

In Flink SQL with JDBC upserts, this key tells connector how to update rows.
It is not physically enforced by Flink, but required for changelog/upsert semantics.

### Watermark timeline walkthrough (step-by-step)

Suppose your events arrive in this order (event time):

- `10:00:05`
- `10:00:12`
- `10:00:09` (late but still acceptable)
- `10:05:01`

With watermark `event_timestamp - 5 seconds`:

1. After seeing `10:00:12`, watermark is near `10:00:07`
2. Event `10:00:09` still accepted (not too late)
3. Once watermark passes end of a window, that window can close and emit results

This is why properly chosen watermark tolerance is critical.

### Code meaning: source DDL computed columns

In this DDL:

```sql
event_timestamp AS TO_TIMESTAMP(lpep_pickup_datetime, 'yyyy-MM-dd HH:mm:ss'),
WATERMARK FOR event_timestamp AS event_timestamp - INTERVAL '5' SECOND
```

- `event_timestamp AS ...` creates a derived timestamp column from string
- `WATERMARK FOR ...` tells Flink how to track event-time progress
- Together they enable event-time windows (`TUMBLE`, `SESSION`) to work correctly

### State and checkpointing (what Flink keeps in memory/disk)

Windowed jobs are stateful. Flink stores intermediate counters/state until windows close.

- If job crashes, checkpointed state enables recovery
- Without checkpoints, recovery may reprocess data differently

Beginner rule of thumb:
- Learn windows first
- Then learn checkpointing and state backends for reliability

---

## Part 5: End-to-End Flink Jobs (Questions 4-6 Pattern) 🛠️

### Common job skeleton

All three Flink jobs follow the same structure:

1. Create source table from Kafka (`green-trips`)
2. Create sink table in PostgreSQL
3. Execute `INSERT INTO ... SELECT ...` with window aggregation
4. Query sink table in Postgres to validate result

### Source table template

```sql
CREATE TABLE green_trips (
    PULocationID INT,
    tip_amount DOUBLE,
    lpep_pickup_datetime VARCHAR,
    event_timestamp AS TO_TIMESTAMP(lpep_pickup_datetime, 'yyyy-MM-dd HH:mm:ss'),
    WATERMARK FOR event_timestamp AS event_timestamp - INTERVAL '5' SECOND
) WITH (
    'connector' = 'kafka',
    'topic' = 'green-trips',
    'properties.bootstrap.servers' = 'redpanda:29092',
    'scan.startup.mode' = 'earliest-offset',
    'format' = 'json'
)
```

### Q4-style job: 5-minute tumbling by pickup zone

Core query pattern:

```sql
INSERT INTO trip_counts
SELECT
    window_start,
    PULocationID,
    COUNT(*) AS num_trips
FROM TABLE(
    TUMBLE(TABLE green_trips, DESCRIPTOR(event_timestamp), INTERVAL '5' MINUTE)
)
GROUP BY window_start, PULocationID
```

**What this query does:**
- Splits stream into fixed 5-minute buckets
- Groups records by both `window_start` and pickup zone
- Counts trips per zone per window
- Writes one row per zone/window to sink table

### Q5-style job: session window by pickup zone

```sql
INSERT INTO trip_sessions
SELECT
    window_start,
    window_end,
    PULocationID,
    COUNT(*) AS num_trips
FROM TABLE(
    SESSION(TABLE green_trips, DESCRIPTOR(event_timestamp), INTERVAL '5' MINUTE)
)
GROUP BY window_start, window_end, PULocationID
```

**What this query does:**
- Builds dynamic sessions where consecutive events are less than 5 minutes apart
- Ends session when inactivity exceeds 5 minutes
- Counts number of trips in each session per zone

### Q6-style job: 1-hour tumbling sum of tips

```sql
INSERT INTO hourly_tips
SELECT
    window_start,
    SUM(tip_amount) AS total_tips
FROM TABLE(
    TUMBLE(TABLE green_trips, DESCRIPTOR(event_timestamp), INTERVAL '1' HOUR)
)
GROUP BY window_start
```

**What this query does:**
- Creates 1-hour fixed windows
- Sums all tip amounts across all zones
- Helps find peak tipping hour

### Operational commands you use repeatedly

Submit job:

```bash
docker exec -it workshop-jobmanager-1 flink run -py /opt/src/job/your_job.py
```

Watch jobs and cancel if needed:
- Flink UI: `http://localhost:8081`

Check results in PostgreSQL:

```sql
SELECT * FROM trip_counts ORDER BY num_trips DESC LIMIT 3;
SELECT * FROM trip_sessions ORDER BY num_trips DESC LIMIT 5;
SELECT * FROM hourly_tips ORDER BY total_tips DESC LIMIT 3;
```

### Complete minimal PyFlink job skeleton

Use this as a reusable template for all three window jobs:

```python
from pyflink.datastream import StreamExecutionEnvironment
from pyflink.table import EnvironmentSettings, StreamTableEnvironment


def main():
    env = StreamExecutionEnvironment.get_execution_environment()
    env.set_parallelism(1)

    settings = EnvironmentSettings.new_instance().in_streaming_mode().build()
    t_env = StreamTableEnvironment.create(env, environment_settings=settings)

    t_env.execute_sql("""
        CREATE TABLE green_trips (
            PULocationID INT,
            tip_amount DOUBLE,
            lpep_pickup_datetime VARCHAR,
            event_timestamp AS TO_TIMESTAMP(lpep_pickup_datetime, 'yyyy-MM-dd HH:mm:ss'),
            WATERMARK FOR event_timestamp AS event_timestamp - INTERVAL '5' SECOND
        ) WITH (
            'connector' = 'kafka',
            'topic' = 'green-trips',
            'properties.bootstrap.servers' = 'redpanda:29092',
            'properties.group.id' = 'streaming-job-group',
            'scan.startup.mode' = 'earliest-offset',
            'format' = 'json'
        )
    """)

    # Replace sink + INSERT query per question
    t_env.execute_sql("""
        INSERT INTO your_sink_table
        SELECT ...
    """).wait()


if __name__ == '__main__':
    main()
```

### Skeleton code walkthrough (what each block is for)

1. `StreamExecutionEnvironment.get_execution_environment()`
    - Creates runtime context for streaming execution

2. `env.set_parallelism(1)`
    - Controls number of parallel task instances
    - Set to `1` here because source topic has one partition

3. `StreamTableEnvironment.create(...)`
    - Enables SQL DDL + SQL queries in Flink

4. `CREATE TABLE green_trips ... WITH ('connector'='kafka', ...)`
    - Registers Kafka topic as a queryable streaming table

5. `INSERT INTO sink SELECT ...`
    - Starts continuous streaming pipeline from source to sink

6. `.wait()`
    - Keeps job attached and running until completion/termination

### SQL validation pattern after each job

Always validate in this order:

1. **Row count sanity**
2. **Top-N by metric**
3. **Time range sanity**
4. **Null checks on key columns**

Example:

```sql
SELECT COUNT(*) FROM trip_counts;
SELECT * FROM trip_counts ORDER BY num_trips DESC LIMIT 10;
SELECT MIN(window_start), MAX(window_start) FROM trip_counts;
SELECT COUNT(*) FROM trip_counts WHERE window_start IS NULL OR PULocationID IS NULL;
```

---

## Part 6: Troubleshooting, Best Practices, and Learning Path 🚦

### Common issues and quick fixes

| Problem | Likely reason | Fix |
|---------|---------------|-----|
| No rows appear in sink | Watermark not advancing | Set `env.set_parallelism(1)` for single-partition topic |
| Consumer reads nothing | Started at `latest` | Use `auto_offset_reset='earliest'` |
| Duplicated counts | Topic contains old messages from prior runs | Delete/recreate topic and rerun producer |
| Flink job runs but no output | Wrong timestamp parsing | Verify `TO_TIMESTAMP` format matches payload |
| JDBC write errors | Sink schema mismatch | Align SQL types and primary key fields |

### Performance and reliability tips for beginners

- Keep schemas explicit; avoid guessing types in production
- Add idempotent sink strategy where possible
- Version your producer payload schema early
- Monitor lag and checkpoint health in real deployments
- Start local, then move to managed Kafka/Flink once flow is stable

### Suggested learning progression after Module 7

1. Add schema evolution with Avro + Schema Registry
2. Add dead-letter strategy for malformed events
3. Move from local docker to cloud Kafka + managed Flink
4. Add exactly-once semantics and checkpoint tuning
5. Build a real-time dashboard over PostgreSQL sink

### Debug playbook (copy/paste friendly)

When results look wrong, use this sequence:

1. Confirm topic exists

```bash
docker exec -it workshop-redpanda-1 rpk topic list
```

2. Confirm topic has data

```bash
docker exec -it workshop-redpanda-1 rpk topic consume green-trips -n 5
```

3. Confirm Flink job is running
- Open `http://localhost:8081` and check job state

4. Confirm sink table exists and receives rows

```sql
SELECT COUNT(*) FROM trip_counts;
```

5. If stuck, restart in clean order
- stop job
- recreate topic
- rerun producer
- rerun Flink job

### Scenario-based troubleshooting examples

#### Scenario A: Flink job is running but Postgres table is empty

Possible causes:
- wrong topic name in source DDL
- wrong bootstrap server (`localhost:9092` vs `redpanda:29092` inside containers)
- watermark not progressing due to parallelism mismatch

Checklist:
1. verify topic has data
2. verify Flink source connector config
3. set `env.set_parallelism(1)`
4. check Flink UI for backpressure/errors

#### Scenario B: SQL results look too high

Likely duplicate events from repeated producer runs.

Fix:
- recreate topic for clean run
- rerun producer once
- rerun Flink job from earliest offsets

#### Scenario C: Timestamp parsing error

Cause:
- input format in `TO_TIMESTAMP` doesn’t match actual message string format

Fix:
- print one consumed message payload
- match exact pattern in Flink SQL format string

### What “good” looks like in a beginner streaming project

- Reproducible local setup (`docker compose up -d`)
- Clear topic naming and schema assumptions
- Windowed job that can be rerun from earliest offsets
- Validation SQL queries documented
- Troubleshooting notes for common failures

If your project has these five, you are already beyond beginner level.

### Community notes mindset (how to learn faster)

From Module 7 community notes and workshop discussions, the recurring theme is:

- Keep first pipeline simple and observable
- Prefer correctness over fancy architecture
- Understand event time deeply before optimizing
- Only use streaming when it solves a real latency problem

### Final recap

By the end of Module 7, you should be able to:

- Explain stream processing fundamentals clearly
- Produce and consume Kafka/Redpanda events in Python
- Build event-time Flink jobs with tumbling and session windows
- Materialize streaming aggregates into PostgreSQL
- Debug watermark/offset/parallelism issues with confidence

That is a complete beginner-to-practical foundation for real-time data engineering pipelines.
