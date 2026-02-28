# Module 5 Summary - Data Platforms with Bruin

#DataEngineeringZoomcamp #Bruin #DataPlatforms #ELT #DataQuality

---

## Part 1: Introduction to Data Platforms 🎯

### What is a Data Platform?

Imagine you're running a restaurant. You need to:
- **Get ingredients** (buy from suppliers)
- **Store ingredients** (in the pantry/fridge)
- **Prepare meals** (cook in the kitchen)
- **Serve customers** (in the dining area)
- **Check quality** (taste test before serving)

A **data platform** works the same way for data! It's a unified system that manages the entire data lifecycle:

| Restaurant Analogy | Data Platform | What It Does |
|-------------------|---------------|--------------|
| 🚚 Delivery truck | **Data Ingestion** | Get data from sources (APIs, files, databases) |
| 🧊 Refrigerator | **Data Storage** | Store raw data in a data warehouse |
| 👨‍🍳 Kitchen | **Data Transformation** | Clean, model, and aggregate data |
| ⏰ Kitchen timer | **Data Orchestration** | Schedule jobs and manage dependencies |
| 👅 Taste test | **Data Quality** | Validate data before serving to analysts |
| 📋 Recipe book | **Metadata Management** | Track lineage and documentation |

### The Problem: Too Many Tools! 😵

Before data platforms, teams had to manage many separate tools:

```
❌ The Old Way (Complex & Fragmented)
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Fivetran     +    dbt    +    Airflow    +    Great          │
│   (ingestion)      (transform)   (orchestrate)   Expectations   │
│                                                   (quality)      │
│                                                                  │
│   = 4 different tools, 4 different configs, 4 things to learn! │
└─────────────────────────────────────────────────────────────────┘
```

This creates problems:
- 💸 **Expensive** - Multiple SaaS subscriptions
- 🤯 **Complex** - Each tool has its own syntax and config
- 🔗 **Integration headaches** - Making tools work together
- 📚 **Steep learning curve** - Learn 4+ tools

### The Solution: Unified Data Platform ✨

A data platform combines everything in one place:

```
✅ The New Way (Simple & Unified)
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│                          BRUIN                                   │
│                                                                  │
│    ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐       │
│    │ Ingest  │ → │Transform│ → │Orchestr │ → │ Quality │       │
│    └─────────┘   └─────────┘   └─────────┘   └─────────┘       │
│                                                                  │
│         = 1 tool, 1 config, 1 thing to learn!                   │
└─────────────────────────────────────────────────────────────────┘
```

### The Modern Data Stack Explained

Here's how data flows through a modern data platform:

```
┌─────────────────┐     ┌──────────────────────┐     ┌─────────────────┐
│  DATA SOURCES   │     │    DATA PLATFORM     │     │    CONSUMERS    │
│                 │     │       (Bruin)        │     │                 │
│  • APIs         │     │                      │     │  • Dashboards   │
│  • Databases    │ ──► │  1. INGEST (load)    │ ──► │  • Reports      │
│  • Files (CSV)  │     │  2. TRANSFORM (clean)│     │  • ML Models    │
│  • SaaS tools   │     │  3. ORCHESTRATE      │     │  • Data Science │
└─────────────────┘     │  4. QUALITY CHECK    │     └─────────────────┘
                        │  5. DOCUMENT (lineage)│
                        └──────────────────────┘
```

**Step by step:**
1. **Ingest**: Pull data from APIs, databases, files into your warehouse
2. **Transform**: Clean messy data, join tables, calculate metrics
3. **Orchestrate**: Run jobs on schedule, handle dependencies
4. **Quality Check**: Validate data before it reaches analysts
5. **Document**: Track where data came from (lineage)

### Why Bruin? 🐻

**Bruin** is an open-source data platform that puts everything under one roof:

| Feature | What It Means for You |
|---------|----------------------|
| ✅ **Single CLI** | Just one command-line tool, no Docker/Kubernetes needed |
| ✅ **Multi-language** | Write in SQL, Python, or YAML - your choice! |
| ✅ **Built-in Quality** | Add data checks without installing extra tools |
| ✅ **Auto Lineage** | See how tables connect without manual work |
| ✅ **Dev/Prod Envs** | Test safely before deploying to production |
| ✅ **Local or Cloud** | Run locally with DuckDB or scale to BigQuery |

**Who uses Bruin?**
- Small teams who want simplicity
- Companies avoiding vendor lock-in
- Data engineers who prefer code over UI
- Anyone learning data engineering!

### ELT vs ETL: Quick Refresher 🔄

Bruin follows the **ELT** pattern:

| | ETL (Old Way) | ELT (Modern Way) |
|--|---------------|------------------|
| **Order** | Extract → Transform → Load | Extract → Load → Transform |
| **Where transform happens** | Before loading (separate system) | After loading (in warehouse) |
| **Why** | Storage was expensive | Storage is cheap, compute is fast |
| **Tools** | Informatica, SSIS | dbt, Bruin, Dataform |

```
ELT Flow:
┌─────────┐    ┌─────────┐    ┌─────────────────────────────────┐
│ Extract │ ─► │  Load   │ ─► │         Transform               │
│ (API)   │    │ (raw)   │    │  (clean, join, aggregate)       │
└─────────┘    └─────────┘    │  ↓                              │
                              │  This happens IN the warehouse! │
                              └─────────────────────────────────┘
```

---

## Part 2: Getting Started with Bruin 🚀

### Prerequisites

Before we start, make sure you have:
- ✅ A terminal (Mac/Linux/WSL)
- ✅ Basic knowledge of SQL
- ✅ A code editor (VS Code recommended)

No Docker, no Kubernetes, no complex setup! Bruin runs as a single binary.

### Step 1: Installation

Install the Bruin CLI with one command:

```bash
# macOS / Linux
curl -LsSf https://getbruin.com/install/cli | sh
```

**What this does:** Downloads and installs the `bruin` command to your system.

Verify it worked:

```bash
bruin --version
# Output: bruin version 0.x.x
```

> 💡 **Tip:** If you get "command not found", restart your terminal or add Bruin to your PATH.

### Step 2: Create Your First Project

Initialize a new project using the zoomcamp template:

```bash
bruin init zoomcamp my-taxi-pipeline
cd my-taxi-pipeline
```

**What this does:**
- Creates a new folder called `my-taxi-pipeline`
- Sets up all the config files you need
- Includes example assets for NYC taxi data

### Step 3: Understand the Project Structure

Open the folder and you'll see:

```
my-taxi-pipeline/
├── .bruin.yml              # 🔧 MAIN CONFIG: Where your databases are
├── pipeline/
│   ├── pipeline.yml        # ⚙️ PIPELINE CONFIG: When and how to run
│   └── assets/
│       ├── ingestion/      # 📥 Layer 1: Download raw data
│       │   └── trips.py
│       ├── staging/        # 🧹 Layer 2: Clean the data
│       │   └── trips.sql
│       └── reports/        # 📊 Layer 3: Analytics tables
│           └── daily.sql
```

Think of it like a factory:
- **Raw materials come in** (ingestion)
- **Get processed** (staging)
- **Finished products go out** (reports)

### Configuration File Deep Dive

#### File 1: `.bruin.yml` - Where is your data?

This file tells Bruin HOW to connect to databases:

```yaml
# .bruin.yml - Main configuration file
# Lives at the ROOT of your project

environments:
  default:                        # Name of this environment
    connections:
      duckdb:                     # Connection type
        name: duckdb-default      # Name to reference in assets
        path: ./data/warehouse.duckdb   # Where the database file lives
```

**Breaking it down:**

| Part | What it means |
|------|---------------|
| `environments:` | You can have multiple (dev, staging, prod) |
| `default:` | The environment name |
| `connections:` | List of databases you connect to |
| `duckdb:` | Type of database (could be `bigquery`, `postgres`, etc.) |
| `name:` | An alias you'll use in your code |
| `path:` | Where DuckDB stores the database file |

> 🎓 **Beginner tip:** DuckDB is a local database - no server needed! Great for learning.

#### File 2: `pipeline.yml` - How should things run?

This file defines WHEN and HOW the pipeline runs:

```yaml
# pipeline/pipeline.yml - Pipeline configuration
# Lives inside the pipeline folder

name: taxi-pipeline             # Human-readable name
schedule: daily                 # When to run (daily, hourly, etc.)

variables:                      # Variables you can use in assets
  taxi_types:
    type: array                 # It's a list
    items:
      type: string              # List of strings
    default: ["yellow", "green"] # Default values
```

**Breaking it down:**

| Part | What it means |
|------|---------------|
| `name:` | What to call this pipeline |
| `schedule:` | How often to run automatically |
| `variables:` | Custom parameters for your pipeline |

### Step 4: Run Your First Pipeline

Now let's see it in action!

```bash
# Run the entire pipeline
bruin run
```

**What happens:**
1. Bruin reads your config files
2. Looks at all assets in the pipeline
3. Figures out the order (dependencies)
4. Runs each asset in order
5. Reports success or failure

**Example output:**
```
Running pipeline: taxi-pipeline
[1/3] ingestion.trips ✓ (2.3s)
[2/3] staging.trips ✓ (0.8s)
[3/3] reports.daily_summary ✓ (0.4s)
Pipeline completed successfully!
```

### Common Beginner Issues & Solutions

| Problem | Solution |
|---------|----------|
| "command not found: bruin" | Restart terminal, or run `source ~/.bashrc` |
| "connection failed" | Check your `.bruin.yml` has correct paths |
| "asset not found" | Make sure file extension is `.sql`, `.py`, or `.yaml` |
| "dependency cycle" | You have circular dependencies, check your `depends:` |

---

## Part 3: Understanding Assets 📦

### What is an Asset?

An **asset** is a single file that creates or updates something in your database. Think of it like a recipe:
- The file contains instructions (SQL, Python, or YAML)
- When you run it, you get a table or view
- It can depend on other assets (ingredients from other recipes)

```
┌─────────────────────────────────────────────────────────────────┐
│                        ASSET FILE                                │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  /* @bruin                                               │   │
│   │  name: staging.trips     ← What table to create          │   │
│   │  type: duckdb.sql        ← What database/language        │   │
│   │  depends:                                                 │   │
│   │    - ingestion.trips     ← Run this first!               │   │
│   │  @bruin */                                                │   │
│   │                                                           │   │
│   │  SELECT * FROM raw.trips  ← The actual code              │   │
│   │  WHERE fare > 0                                           │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   When you run this, it creates: staging.trips table            │
└─────────────────────────────────────────────────────────────────┘
```

### Asset Types Explained

Bruin supports three types of assets:

| Type | File Extension | Best For | Example Use Case |
|------|----------------|----------|------------------|
| **SQL** | `.sql` | Transformations, joins, aggregations | Clean data, create reports |
| **Python** | `.py` | API calls, complex logic, ML | Download files, call APIs |
| **YAML** | `.yaml` | Pre-built connectors (ingestors) | Pull from Salesforce, Stripe |

### SQL Assets: The Most Common Type

SQL assets are the bread and butter of data transformation. Here's a complete example:

```sql
/* @bruin

name: staging.trips
type: duckdb.sql

description: Clean and transform raw taxi trip data

depends:
  - ingestion.trips

materialization:
  type: table
  strategy: time_interval
  incremental_key: pickup_datetime

columns:
  - name: pickup_datetime
    description: When the trip started
    checks:
      - name: not_null
  - name: fare_amount
    description: The metered fare
    checks:
      - name: positive

@bruin */

-- Everything below is regular SQL!
SELECT
    pickup_datetime,
    dropoff_datetime,
    passenger_count,
    trip_distance,
    fare_amount
FROM raw.taxi_trips
WHERE pickup_datetime >= '{{ start_date }}'
  AND pickup_datetime < '{{ end_date }}'
```

**Let's break down each part:**

| Section | What It Does |
|---------|--------------|
| `/* @bruin ... @bruin */` | The header block (metadata) |
| `name: staging.trips` | Creates a table called `trips` in `staging` schema |
| `type: duckdb.sql` | This is SQL that runs on DuckDB |
| `description:` | Human-readable explanation |
| `depends:` | List of assets that must run first |
| `materialization:` | How to store the data (table, view, etc.) |
| `columns:` | Schema info and quality checks |
| `{{ start_date }}` | A variable (replaced at runtime) |

### Python Assets: For Custom Logic

When SQL isn't enough, use Python:

```python
""" @bruin

name: ingestion.trips
type: python

description: Download NYC taxi data from the internet

depends: []

@bruin """

# Everything below is regular Python!
import httpx
import duckdb

def main():
    """
    Bruin automatically calls this main() function.
    """
    # Step 1: Define where to get data
    url = "https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2024-01.parquet"
    
    # Step 2: Connect to DuckDB
    conn = duckdb.connect("data/warehouse.duckdb")
    
    # Step 3: Load data directly from URL (DuckDB can read parquet from URLs!)
    conn.execute(f"""
        CREATE OR REPLACE TABLE raw.trips AS 
        SELECT * FROM read_parquet('{url}')
    """)
    
    print(f"Loaded data from {url}")
```

**Key things to remember:**
- ✅ Use triple quotes `"""` for the header (not `/* */`)
- ✅ Always define a `main()` function - Bruin calls this
- ✅ `depends: []` means this runs first (no dependencies)

### YAML Ingestors: Pre-Built Connectors

For common data sources, Bruin has pre-built ingestors:

```yaml
# ingestion/chess_games.yaml

name: ingestion.chess_games
type: ingestr

parameters:
  source_connection: chess.com      # Where to get data
  source_table: games               # What table/endpoint
  destination: duckdb-default.raw.chess_games  # Where to put it
```

**Why use YAML ingestors?**
- 🚀 No code needed - just configuration
- 🔌 Pre-built connectors for 50+ data sources
- 🔄 Handles pagination, rate limits, auth automatically

### The Asset Header Explained

Every asset needs a header. Here's a complete reference:

```
/* @bruin                         ← Start marker

name: schema.table_name           ← REQUIRED: What to create
type: duckdb.sql                  ← REQUIRED: Database.language

description: What this asset does ← Optional but recommended

depends:                          ← What runs first
  - other.asset1
  - other.asset2

materialization:                  ← How to store results
  type: table                     ← table, view, or nothing
  strategy: replace               ← How to update (more on this later)

columns:                          ← Schema and quality checks
  - name: column_name
    description: What it is
    checks:
      - name: not_null

@bruin */                         ← End marker
```

### How Dependencies Work

Dependencies control the order of execution:

```
                    ┌─────────────────┐
                    │ ingestion.trips │  ← Runs FIRST (no depends)
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  staging.trips  │  ← Runs SECOND
                    │                 │     depends: [ingestion.trips]
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
    ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
    │reports.daily │ │reports.weekly│ │reports.zones │
    └──────────────┘ └──────────────┘ └──────────────┘
                    ← All run THIRD (in parallel!)
                      depends: [staging.trips]
```

Bruin automatically:
1. Builds a dependency graph
2. Runs assets in the correct order
3. Runs independent assets in parallel for speed!

---

## Part 4: Materialization Strategies 🔄

### What is Materialization?

**Materialization** is HOW Bruin stores the result of your SQL query. Think of it like cooking:
- **View** = A recipe card (instructions only, cook each time you want it)
- **Table** = Pre-made meals in the freezer (ready to serve immediately)

Each strategy has trade-offs:

| Strategy | Speed | Storage | Freshness |
|----------|-------|---------|-----------|
| View | Slow (computed each time) | None | Always fresh |
| Table | Fast (pre-computed) | Uses space | Stale until refreshed |

### The Five Materialization Strategies

#### 1. `view` - Virtual Table (No Storage)

```sql
/* @bruin
materialization:
  type: view
@bruin */
```

**How it works:**
- Creates a "saved query" - no data is stored
- Every time you query it, the SQL runs fresh
- Zero storage cost

**When to use:**
- ✅ Small datasets
- ✅ You need real-time data
- ✅ Testing/development

**When NOT to use:**
- ❌ Large datasets (slow!)
- ❌ Complex queries (recalculated every time)

#### 2. `table` (default) - Full Table

```sql
/* @bruin
materialization:
  type: table
@bruin */
```

**How it works:**
- Runs the query and stores results
- Replaces entire table each run
- Fast to query (data is pre-computed)

**When to use:**
- ✅ Most common choice
- ✅ Small to medium tables
- ✅ You can afford full rebuilds

#### 3. `append` - Add New Rows Only

```sql
/* @bruin
materialization:
  type: table
  strategy: append
@bruin */
```

**How it works:**
```
Before: [row1, row2, row3]
Run:    INSERT [row4, row5]
After:  [row1, row2, row3, row4, row5]
```

**When to use:**
- ✅ Event logs (clicks, transactions)
- ✅ Data that never changes
- ✅ You only add new records

**Warning:** If you run twice with the same data, you'll get duplicates!

#### 4. `replace` - Truncate and Rebuild

```sql
/* @bruin
materialization:
  type: table
  strategy: replace
@bruin */
```

**How it works:**
```
Before: [old_row1, old_row2, old_row3]
Run:    DELETE ALL, then INSERT
After:  [new_row1, new_row2]
```

**When to use:**
- ✅ Full refreshes
- ✅ Small reference tables
- ✅ When you want a clean slate

#### 5. `time_interval` - Smart Incremental (Best for Time-Series!) ⭐

```sql
/* @bruin
materialization:
  type: table
  strategy: time_interval
  incremental_key: pickup_datetime
@bruin */
```

**How it works:**
```
Say you're processing January 15, 2024:

Step 1: DELETE WHERE pickup_datetime >= '2024-01-15' 
                  AND pickup_datetime < '2024-01-16'
        (Remove old data for that day)

Step 2: INSERT new data for January 15, 2024
        (Add fresh data)
```

**Visual explanation:**
```
Before running for Jan 15:
┌─────────────────────────────────────────────────┐
│ Jan 13 │ Jan 14 │ Jan 15 (old) │ Jan 16 │      │
└─────────────────────────────────────────────────┘

After running for Jan 15:
┌─────────────────────────────────────────────────┐
│ Jan 13 │ Jan 14 │ Jan 15 (NEW) │ Jan 16 │      │
└─────────────────────────────────────────────────┘
       unchanged      replaced      unchanged
```

**When to use:**
- ✅ Time-series data (taxi trips, transactions, logs)
- ✅ Daily/hourly processing
- ✅ Large tables where full rebuild is too slow

**Why it's the best for taxi data:**
- Only processes one day/hour at a time
- No duplicates (deletes before inserting)
- Much faster than full rebuild

### Complete Example: Incremental Taxi Trips

```sql
/* @bruin

name: staging.taxi_trips
type: duckdb.sql
description: Clean taxi trip data, processed incrementally by day

materialization:
  type: table
  strategy: time_interval
  incremental_key: pickup_datetime

@bruin */

SELECT
    pickup_datetime,
    dropoff_datetime,
    passenger_count,
    trip_distance,
    fare_amount,
    tip_amount,
    total_amount
FROM raw.trips
WHERE pickup_datetime >= '{{ start_date }}'
  AND pickup_datetime < '{{ end_date }}'
  AND fare_amount > 0  -- Filter bad data
```

**How Bruin uses `{{ start_date }}` and `{{ end_date }}`:**
- These are replaced at runtime
- For daily runs: `start_date = '2024-01-15'`, `end_date = '2024-01-16'`
- You can override them manually too!

### Strategy Comparison Cheat Sheet

```
┌──────────────────┬───────────────────────────────────────────────────┐
│    Strategy      │                   Behavior                        │
├──────────────────┼───────────────────────────────────────────────────┤
│    view          │  No storage - computed on every query             │
├──────────────────┼───────────────────────────────────────────────────┤
│    table         │  DROP + CREATE - full rebuild each time           │
├──────────────────┼───────────────────────────────────────────────────┤
│    append        │  INSERT only - adds rows, never deletes           │
├──────────────────┼───────────────────────────────────────────────────┤
│    replace       │  TRUNCATE + INSERT - deletes all, then inserts    │
├──────────────────┼───────────────────────────────────────────────────┤
│  time_interval   │  DELETE time range + INSERT - smart incremental   │
└──────────────────┴───────────────────────────────────────────────────┘
```

---

## Part 5: Pipeline Variables 📊

### What Are Variables?

Variables are like blanks in a form that get filled in when the pipeline runs. They let you write flexible code that works with different values.

```
Without variables (hardcoded):
SELECT * FROM trips WHERE pickup_date = '2024-01-15'
                                         ^^^^^^^^^^
                                         This is stuck!

With variables (flexible):
SELECT * FROM trips WHERE pickup_date = '{{ start_date }}'
                                         ^^^^^^^^^^^^^^^
                                         Filled in at runtime!
```

### Built-in Variables (Automatic!)

Bruin provides these variables automatically for every run:

| Variable | What It Contains | Example Value |
|----------|------------------|---------------|
| `{{ start_date }}` | Start of processing window (date only) | `2024-01-15` |
| `{{ end_date }}` | End of processing window (date only) | `2024-01-16` |
| `{{ start_datetime }}` | Start with time | `2024-01-15T00:00:00` |
| `{{ end_datetime }}` | End with time | `2024-01-16T00:00:00` |
| `{{ pipeline_name }}` | Name of the pipeline | `taxi-pipeline` |
| `{{ run_id }}` | Unique ID for this run | `abc123def456` |

**Why are these useful?**
- Process data day-by-day without changing code
- Track which run created which data
- Backfill historical data by changing dates

**Example: Filter by date range**

```sql
/* @bruin
name: staging.trips
@bruin */

SELECT *
FROM raw.trips
WHERE pickup_datetime >= '{{ start_date }}'   -- e.g., '2024-01-15'
  AND pickup_datetime < '{{ end_date }}'      -- e.g., '2024-01-16'
```

### Custom Variables

You can define your own variables in `pipeline.yml`:

```yaml
# pipeline/pipeline.yml

name: taxi-pipeline
schedule: daily

variables:
  # Array variable (list of values)
  taxi_types:
    type: array
    items:
      type: string
    default: ["yellow", "green"]
  
  # Number variable
  min_fare:
    type: float
    default: 2.50
  
  # String variable
  output_schema:
    type: string
    default: "staging"
  
  # Boolean variable
  include_cancelled:
    type: boolean
    default: false
```

**Using custom variables in SQL:**

```sql
/* @bruin
name: staging.trips
@bruin */

SELECT *
FROM raw.trips
WHERE fare_amount >= {{ min_fare }}              -- Replaced with 2.50
  AND taxi_type IN ({{ taxi_types | join(',') }}) -- Replaced with 'yellow','green'
  {% if include_cancelled == false %}
  AND trip_status != 'cancelled'                  -- Conditional logic!
  {% endif %}
```

### Overriding Variables at Runtime

You can change variable values when running the pipeline:

```bash
# Override a single variable
bruin run --var 'min_fare=5.00'

# Override an array variable (must use JSON format!)
bruin run --var 'taxi_types=["yellow"]'

# Override multiple variables
bruin run --var 'min_fare=5.00' --var 'taxi_types=["yellow"]'

# Override date range (process a specific day)
bruin run --start-date 2024-01-15 --end-date 2024-01-16
```

**Common use cases for overriding:**
- 🔄 **Backfill**: Process historical data with different dates
- 🧪 **Testing**: Run with a subset of data
- 🐛 **Debugging**: Process only yellow taxis to isolate issues

### Variable Types Reference

| Type | Example Definition | Example Value |
|------|-------------------|---------------|
| `string` | `type: string, default: "hello"` | `"hello"` |
| `integer` | `type: integer, default: 10` | `10` |
| `float` | `type: float, default: 2.50` | `2.50` |
| `boolean` | `type: boolean, default: true` | `true` |
| `array` | `type: array, items: {type: string}` | `["a", "b"]` |

### Practical Example: Parameterized Pipeline

Here's a real-world example putting it all together:

**pipeline.yml:**
```yaml
name: nyc-taxi-pipeline
schedule: daily

variables:
  taxi_types:
    type: array
    items:
      type: string
    default: ["yellow", "green"]
  
  min_fare:
    type: float
    default: 2.50
```

**staging/trips.sql:**
```sql
/* @bruin
name: staging.trips
type: duckdb.sql
materialization:
  type: table
  strategy: time_interval
  incremental_key: pickup_datetime
@bruin */

SELECT
    pickup_datetime,
    dropoff_datetime,
    taxi_type,
    fare_amount
FROM raw.taxi_trips
WHERE 
    -- Only process data in our date window
    pickup_datetime >= '{{ start_date }}'
    AND pickup_datetime < '{{ end_date }}'
    
    -- Filter by configured parameters
    AND fare_amount >= {{ min_fare }}
    AND taxi_type IN ('{{ taxi_types | join("','") }}')
```

**Run commands:**
```bash
# Normal run (uses defaults)
bruin run
# → Processes yellow & green taxis, fare >= $2.50

# Process only yellow taxis
bruin run --var 'taxi_types=["yellow"]'

# Process expensive fares only
bruin run --var 'min_fare=50.00'

# Backfill a specific week
bruin run --start-date 2024-01-01 --end-date 2024-01-08
```

---

## Part 6: Data Quality Checks ✅

### Why Data Quality Matters

Bad data leads to bad decisions. Imagine:
- 📊 A dashboard showing "Total Revenue: -$50,000" (negative fares!)
- 📈 A report claiming "1 million trips yesterday" (duplicates!)
- 🤖 An ML model trained on NULL values (garbage in, garbage out!)

Data quality checks catch these problems BEFORE they reach analysts.

### The "Garbage In, Garbage Out" Problem

```
Without Quality Checks:
┌─────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ Raw Data│ →  │ Transform   │ →  │   Reports   │ →  │ 💩 Bad      │
│ (messy) │    │ (no checks) │    │ (broken)    │    │ Decisions   │
└─────────┘    └─────────────┘    └─────────────┘    └─────────────┘

With Quality Checks:
┌─────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ Raw Data│ →  │ Transform   │ →  │   Reports   │ →  │ ✅ Good     │
│ (messy) │    │ + CHECKS ✓  │    │ (reliable)  │    │ Decisions   │
└─────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                    ↓
              If check fails:
              STOP pipeline!
              Alert the team!
```

### Built-in Quality Checks

Bruin provides these checks out of the box:

| Check | What It Validates | Example Failure |
|-------|-------------------|-----------------|
| `not_null` | Column has no NULL values | `pickup_datetime` is NULL |
| `unique` | All values are unique (no duplicates) | Two trips have same ID |
| `positive` | Numbers are greater than zero | `fare_amount = -5.00` |
| `non_negative` | Numbers are zero or greater | `passenger_count = -1` |
| `accepted_values` | Values are in allowed list | `payment_type = 99` (unknown) |
| `pattern` | Text matches a regex pattern | `email = "not-an-email"` |

### Adding Quality Checks to Assets

Checks are defined in the asset header:

```sql
/* @bruin

name: staging.trips
type: duckdb.sql

columns:
  - name: trip_id
    description: Unique identifier for each trip
    checks:
      - name: not_null     # Must exist
      - name: unique       # No duplicates
  
  - name: pickup_datetime
    description: When the trip started
    checks:
      - name: not_null     # Every trip needs a start time
  
  - name: fare_amount
    description: The metered fare in dollars
    checks:
      - name: not_null
      - name: positive     # No free or negative fares
  
  - name: passenger_count
    description: Number of passengers
    checks:
      - name: non_negative # Can be 0 (driver alone) but not negative
  
  - name: payment_type
    description: How the customer paid
    checks:
      - name: accepted_values
        value: [1, 2, 3, 4, 5]  # 1=Credit, 2=Cash, etc.

@bruin */

SELECT
    trip_id,
    pickup_datetime,
    fare_amount,
    passenger_count,
    payment_type
FROM raw.trips
```

### What Happens When a Check Fails?

When Bruin detects bad data:

```
$ bruin run

Running pipeline: taxi-pipeline
[1/3] ingestion.trips ✓ (2.3s)
[2/3] staging.trips
      ⚠️ Quality check failed: "positive" on column "fare_amount"
      Found 47 rows with fare_amount <= 0
      
      Sample failing rows:
      | trip_id | fare_amount |
      |---------|-------------|
      | abc123  | -5.50       |
      | def456  | 0.00        |
      | ghi789  | -12.00      |
      
      ❌ Pipeline stopped!

Exit code: 1
```

**The pipeline STOPS immediately!** This prevents bad data from reaching downstream assets.

### Quality Check Examples by Data Type

#### Text/String Columns

```yaml
columns:
  - name: email
    checks:
      - name: not_null
      - name: pattern
        value: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
  
  - name: status
    checks:
      - name: accepted_values
        value: ["pending", "completed", "cancelled"]
```

#### Numeric Columns

```yaml
columns:
  - name: amount
    checks:
      - name: positive         # > 0
  
  - name: quantity
    checks:
      - name: non_negative     # >= 0
  
  - name: percentage
    checks:
      - name: accepted_values
        value: [0, 25, 50, 75, 100]  # Only these values allowed
```

#### Date/Time Columns

```yaml
columns:
  - name: pickup_datetime
    checks:
      - name: not_null         # Every trip needs a timestamp
  
  - name: dropoff_datetime
    checks:
      - name: not_null
```

#### ID Columns

```yaml
columns:
  - name: trip_id
    checks:
      - name: not_null
      - name: unique           # Primary key - must be unique!
```

### Combining Multiple Checks

You can add multiple checks to one column:

```yaml
columns:
  - name: fare_amount
    description: Total trip fare
    checks:
      - name: not_null      # Check 1: Must exist
      - name: positive      # Check 2: Must be > 0
```

All checks must pass! If ANY check fails, the pipeline stops.

### Best Practices for Data Quality

| ✅ Do | ❌ Don't |
|------|---------|
| Add `not_null` to required columns | Leave key columns unchecked |
| Use `unique` for primary keys | Assume source data is clean |
| Add `accepted_values` for categorical data | Use overly strict checks in dev |
| Start with basic checks, add more over time | Add 50 checks on day one |
| Investigate failures, don't just remove checks | Ignore failing checks |

### The Quality Check Workflow

```
1. Define checks in asset header
   ↓
2. Run pipeline: bruin run
   ↓
3. If checks pass → continue to next asset
   If checks fail → STOP and show errors
   ↓
4. Fix the data or adjust the checks
   ↓
5. Re-run pipeline
```

---

## Part 7: CLI Commands 🛠️

### The Bruin CLI

Everything in Bruin happens through the command line. No web UI needed for development!

```bash
bruin <command> [options] [arguments]
```

### Essential Commands Overview

| Command | Purpose | When to Use |
|---------|---------|-------------|
| `bruin init` | Create a new project | Starting fresh |
| `bruin run` | Execute assets | Running your pipeline |
| `bruin validate` | Check configuration | Before running |
| `bruin lineage` | View dependencies | Understanding flow |

### Command 1: `bruin init` - Create Projects

Create a new Bruin project from a template:

```bash
# Syntax
bruin init <template> <project-name>

# Examples
bruin init zoomcamp my-taxi-pipeline    # Use the zoomcamp template
bruin init default my-project           # Use the default template
```

**What it creates:**
```
my-taxi-pipeline/
├── .bruin.yml          # ← Connection config
├── pipeline/
│   ├── pipeline.yml    # ← Pipeline config
│   └── assets/         # ← Your code goes here
└── data/               # ← DuckDB database location
```

### Command 2: `bruin run` - Execute Pipelines

This is the command you'll use most often!

#### Run Everything

```bash
# Run all assets in the pipeline
bruin run
```

**What happens:**
1. Bruin scans for all assets
2. Builds dependency graph
3. Runs assets in correct order
4. Runs quality checks
5. Reports results

#### Run a Specific Asset

```bash
# Run just one asset
bruin run staging/trips.sql

# Or use the asset name
bruin run staging.trips
```

#### Run with Dependencies

```bash
# Run an asset AND everything that depends on it (downstream)
bruin run ingestion/trips.py --downstream
```

**Visual explanation:**
```
                        ingestion/trips.py  ← YOU RAN THIS
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
        staging/trips   staging/zones   staging/fares
              │               │               │
              └───────────────┼───────────────┘
                              ▼
                       reports/daily    ← THESE ALL RUN TOO!
```

```bash
# Run an asset AND everything it depends on (upstream)
bruin run reports/daily.sql --upstream
```

#### Full Refresh (Start from Scratch)

```bash
# Ignore incremental logic, rebuild everything
bruin run --full-refresh
```

**When to use `--full-refresh`:**
- ✅ First time running on a new database
- ✅ Schema changed (added/removed columns)
- ✅ Need to reprocess all historical data
- ✅ Something went wrong, want a clean slate

#### Specify Date Range

```bash
# Process a specific date
bruin run --start-date 2024-01-15 --end-date 2024-01-16

# Process an entire month
bruin run --start-date 2024-01-01 --end-date 2024-02-01

# Backfill historical data
bruin run --start-date 2023-01-01 --end-date 2024-01-01 --full-refresh
```

#### Override Variables

```bash
# Change a variable value
bruin run --var 'min_fare=10.00'

# Override array variable (JSON format!)
bruin run --var 'taxi_types=["yellow"]'

# Multiple overrides
bruin run --var 'min_fare=10.00' --var 'taxi_types=["yellow"]'
```

### Command 3: `bruin validate` - Check Configuration

Before running, validate your config:

```bash
# Validate the entire project
bruin validate

# Validate a specific pipeline
bruin validate pipeline/
```

**What it checks:**
- ✅ YAML syntax is correct
- ✅ Asset headers are valid
- ✅ Dependencies exist
- ✅ No circular dependencies
- ✅ Connection names are defined

**Example output (success):**
```
$ bruin validate
Validating project...
✓ .bruin.yml is valid
✓ pipeline/pipeline.yml is valid
✓ Found 5 assets
✓ No circular dependencies
✓ All dependencies exist

Validation passed!
```

**Example output (failure):**
```
$ bruin validate
Validating project...
✓ .bruin.yml is valid
✗ pipeline/assets/staging/trips.sql
  Error: Unknown dependency "ingestion.trip" (did you mean "ingestion.trips"?)

Validation failed!
```

### Command 4: `bruin lineage` - Visualize Dependencies

See how your assets connect:

```bash
# Show lineage for the entire pipeline
bruin lineage

# Show lineage for a specific asset
bruin lineage staging/trips.sql
```

**Example output:**
```
$ bruin lineage

Pipeline: taxi-pipeline

ingestion.trips (python)
└── staging.trips (sql)
    ├── reports.daily_summary (sql)
    └── reports.zone_stats (sql)

ingestion.zones (yaml)
└── staging.zones (sql)
    └── reports.zone_stats (sql)
```

**Why lineage matters:**
- 🔍 Understand data flow
- 🐛 Debug issues (find which asset caused a problem)
- 📊 Impact analysis (what breaks if I change this?)

### Command Cheat Sheet

```bash
# ─────────────────────────────────────────────────────────
# PROJECT SETUP
# ─────────────────────────────────────────────────────────
bruin init zoomcamp my-proj     # Create new project
bruin validate                  # Check configuration

# ─────────────────────────────────────────────────────────
# RUNNING PIPELINES
# ─────────────────────────────────────────────────────────
bruin run                       # Run everything
bruin run asset.sql             # Run one asset
bruin run asset.sql --downstream  # Include downstream
bruin run asset.sql --upstream    # Include upstream
bruin run --full-refresh        # Rebuild from scratch

# ─────────────────────────────────────────────────────────
# DATE RANGES
# ─────────────────────────────────────────────────────────
bruin run --start-date 2024-01-15 --end-date 2024-01-16

# ─────────────────────────────────────────────────────────
# VARIABLES
# ─────────────────────────────────────────────────────────
bruin run --var 'key=value'
bruin run --var 'array=["a","b"]'

# ─────────────────────────────────────────────────────────
# INSPECTION
# ─────────────────────────────────────────────────────────
bruin lineage                   # Show dependency graph
bruin lineage asset.sql         # Show for specific asset
```

### Common Errors and Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| `asset not found` | Typo in asset name | Check filename and path |
| `connection failed` | Wrong database config | Check `.bruin.yml` |
| `dependency cycle` | A depends on B, B depends on A | Remove circular reference |
| `variable not defined` | Using `{{ var }}` without defining it | Add to `pipeline.yml` |

---

## Part 8: Building an NYC Taxi Pipeline 🚕

### The Goal

We're going to build a complete data pipeline that:
1. Downloads NYC taxi trip data from the internet
2. Cleans and transforms the raw data
3. Creates summary reports for analysis

### The Three-Layer Architecture

Think of it like a factory assembly line:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        NYC TAXI PIPELINE                                 │
│                                                                          │
│  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐  │
│  │   📥 INGESTION   │    │    🧹 STAGING    │    │   📊 REPORTS     │  │
│  │                  │    │                  │    │                  │  │
│  │  Download raw    │ ─► │  Clean & filter  │ ─► │  Aggregate for   │  │
│  │  data from web   │    │  bad records     │    │  dashboards      │  │
│  │                  │    │                  │    │                  │  │
│  │  Python          │    │  SQL             │    │  SQL             │  │
│  └──────────────────┘    └──────────────────┘    └──────────────────┘  │
│                                                                          │
│     ⬇️ Creates:              ⬇️ Creates:           ⬇️ Creates:          │
│     raw.trips                staging.trips         reports.daily       │
│     raw.zones                staging.zones         reports.monthly     │
└─────────────────────────────────────────────────────────────────────────┘
```

**Why three layers?**

| Layer | Purpose | Language | Tables Created |
|-------|---------|----------|----------------|
| **Ingestion** | Get data into the warehouse | Python/YAML | `raw.*` |
| **Staging** | Clean and standardize | SQL | `staging.*` |
| **Reports** | Business metrics | SQL | `reports.*` |

### Layer 1: Ingestion - Getting the Data

The ingestion layer downloads raw data and loads it into DuckDB.

#### trips.py - Download Taxi Trip Data

```python
""" @bruin

name: ingestion.trips
type: python
description: Download NYC Yellow Taxi trip data from NYC Open Data

# No dependencies - this runs first!
depends: []

@bruin """

import duckdb
from pathlib import Path

def main():
    """
    Download taxi trip data and load into DuckDB.
    Bruin automatically calls this function.
    """
    
    # Step 1: Data source URL (NYC publishes this monthly)
    url = "https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2024-01.parquet"
    
    # Step 2: Ensure the data directory exists
    Path("data").mkdir(exist_ok=True)
    
    # Step 3: Connect to DuckDB
    conn = duckdb.connect("data/warehouse.duckdb")
    
    # Step 4: Create the raw schema if it doesn't exist
    conn.execute("CREATE SCHEMA IF NOT EXISTS raw")
    
    # Step 5: Load data directly from URL (DuckDB is amazing!)
    conn.execute(f"""
        CREATE OR REPLACE TABLE raw.trips AS 
        SELECT * FROM read_parquet('{url}')
    """)
    
    # Step 6: Verify it worked
    count = conn.execute("SELECT COUNT(*) FROM raw.trips").fetchone()[0]
    print(f"✓ Loaded {count:,} trips into raw.trips")
```

**What's happening:**
1. DuckDB can read Parquet files directly from URLs (no download needed!)
2. We create a `raw` schema to organize raw data
3. `CREATE OR REPLACE` ensures we start fresh each time

#### zones.yaml - Load Zone Reference Data (Using Ingestor)

For simpler data sources, use YAML ingestors:

```yaml
# ingestion/zones.yaml

name: ingestion.zones
type: ingestr

description: Load NYC taxi zone lookup table

parameters:
  source: csv
  source_uri: "https://d37ci6vzurychx.cloudfront.net/misc/taxi+_zone_lookup.csv"
  destination: duckdb-default.raw.zones
```

**Why use YAML here?**
- No code needed for simple CSV/Parquet loads
- Built-in error handling
- Declarative = easier to understand

### Layer 2: Staging - Cleaning the Data

Now we clean the raw data and prepare it for analysis.

#### staging/trips.sql - Clean Trip Data

```sql
/* @bruin

name: staging.trips
type: duckdb.sql
description: Clean and transform raw taxi trip data

depends:
  - ingestion.trips    # Wait for raw data to load first!

materialization:
  type: table
  strategy: time_interval
  incremental_key: pickup_datetime

columns:
  - name: trip_id
    description: Unique trip identifier
    checks:
      - name: unique
  - name: pickup_datetime
    description: Trip start time
    checks:
      - name: not_null
  - name: fare_amount
    description: Metered fare in USD
    checks:
      - name: positive

@bruin */

-- Generate a unique ID for each trip
-- Clean up column names
-- Filter out bad records

WITH cleaned AS (
    SELECT
        -- Create unique ID from multiple fields
        md5(
            CAST(tpep_pickup_datetime AS VARCHAR) || 
            CAST(tpep_dropoff_datetime AS VARCHAR) ||
            CAST(PULocationID AS VARCHAR) ||
            CAST(DOLocationID AS VARCHAR)
        ) AS trip_id,
        
        -- Rename columns to be clearer
        tpep_pickup_datetime AS pickup_datetime,
        tpep_dropoff_datetime AS dropoff_datetime,
        PULocationID AS pickup_location_id,
        DOLocationID AS dropoff_location_id,
        
        -- Keep as-is
        passenger_count,
        trip_distance,
        fare_amount,
        tip_amount,
        total_amount,
        payment_type
        
    FROM raw.trips
    
    -- Only process data in our time window (for incremental loads)
    WHERE tpep_pickup_datetime >= '{{ start_date }}'
      AND tpep_pickup_datetime < '{{ end_date }}'
)

SELECT *
FROM cleaned
WHERE 
    -- Filter out bad data
    fare_amount > 0           -- No free rides
    AND trip_distance > 0     -- Actual movement
    AND passenger_count > 0   -- Has passengers
```

**Key concepts:**
1. **depends**: Waits for `ingestion.trips` to complete
2. **time_interval**: Only processes data in the date range
3. **quality checks**: Validates data after transformation
4. **filtering**: Removes bad records (negative fares, zero distance)

#### staging/zones.sql - Clean Zone Data

```sql
/* @bruin

name: staging.zones
type: duckdb.sql
description: Standardize zone lookup data

depends:
  - ingestion.zones

materialization:
  type: table

columns:
  - name: zone_id
    checks:
      - name: not_null
      - name: unique

@bruin */

SELECT
    LocationID AS zone_id,
    Borough AS borough,
    Zone AS zone_name,
    service_zone
FROM raw.zones
WHERE LocationID IS NOT NULL
```

### Layer 3: Reports - Analytics-Ready Tables

Finally, create summary tables for dashboards and analysis.

#### reports/daily_summary.sql - Daily Metrics

```sql
/* @bruin

name: reports.daily_summary
type: duckdb.sql
description: Daily aggregated taxi metrics

depends:
  - staging.trips    # Needs clean trip data

materialization:
  type: table

columns:
  - name: trip_date
    description: Date of trips
    checks:
      - name: unique  # One row per date!

@bruin */

SELECT
    -- Group by date
    CAST(pickup_datetime AS DATE) AS trip_date,
    
    -- Count metrics
    COUNT(*) AS total_trips,
    SUM(passenger_count) AS total_passengers,
    
    -- Revenue metrics
    SUM(fare_amount) AS total_fares,
    SUM(tip_amount) AS total_tips,
    SUM(total_amount) AS total_revenue,
    
    -- Average metrics
    ROUND(AVG(fare_amount), 2) AS avg_fare,
    ROUND(AVG(trip_distance), 2) AS avg_distance,
    ROUND(AVG(tip_amount / NULLIF(fare_amount, 0)) * 100, 1) AS avg_tip_percentage,
    
    -- Metadata
    CURRENT_TIMESTAMP AS last_updated

FROM staging.trips
GROUP BY CAST(pickup_datetime AS DATE)
ORDER BY trip_date
```

#### reports/zone_stats.sql - Zone-Level Analysis

```sql
/* @bruin

name: reports.zone_stats
type: duckdb.sql
description: Trip statistics by pickup zone

depends:
  - staging.trips
  - staging.zones    # Needs zone names!

materialization:
  type: table

@bruin */

SELECT
    z.zone_name,
    z.borough,
    COUNT(*) AS total_trips,
    ROUND(AVG(t.fare_amount), 2) AS avg_fare,
    ROUND(SUM(t.total_amount), 2) AS total_revenue
FROM staging.trips t
LEFT JOIN staging.zones z 
    ON t.pickup_location_id = z.zone_id
GROUP BY z.zone_name, z.borough
ORDER BY total_trips DESC
```

### Running the Complete Pipeline

```bash
# First run: create everything from scratch
bruin run --full-refresh

# Output:
# Running pipeline: taxi-pipeline
# [1/5] ingestion.trips ✓ (15.2s)
# [2/5] ingestion.zones ✓ (1.1s)
# [3/5] staging.trips ✓ (3.4s)
# [4/5] staging.zones ✓ (0.2s)
# [5/5] reports.daily_summary ✓ (0.8s)
# [5/5] reports.zone_stats ✓ (0.5s)
# 
# Pipeline completed successfully!
```

### Verify It Worked

Query your data:

```bash
# Connect to DuckDB
duckdb data/warehouse.duckdb

# Check the reports
SELECT * FROM reports.daily_summary LIMIT 5;
```

---

## Part 9: Deploying to Bruin Cloud ☁️

### Why Deploy to the Cloud?

Running `bruin run` manually is great for development, but in production you need:
- ⏰ **Automated scheduling** - Run daily without human intervention
- 📧 **Alerts** - Get notified when something fails
- 📊 **Monitoring** - See run history and performance
- 🔐 **Security** - Secure credential management

### Local vs Cloud Comparison

| Feature | Local (Your Laptop) | Bruin Cloud |
|---------|---------------------|-------------|
| Run manually | `bruin run` | Automatic schedule |
| Always on? | No (laptop sleeps) | Yes (24/7) |
| View history | Terminal output | Web dashboard |
| Alerts | None | Email/Slack |
| Cost | Free | Free tier available |

### Deployment Steps (Beginner-Friendly)

#### Step 1: Create a Bruin Cloud Account

1. Go to [getbruin.com](https://getbruin.com)
2. Click "Sign Up"
3. Use your GitHub account (recommended)

#### Step 2: Push Your Pipeline to GitHub

Your pipeline code needs to be in a Git repository:

```bash
# If not already a git repo
cd my-taxi-pipeline
git init
git add .
git commit -m "Initial taxi pipeline"

# Create repo on GitHub, then:
git remote add origin https://github.com/YOUR_USERNAME/my-taxi-pipeline.git
git push -u origin main
```

#### Step 3: Connect GitHub to Bruin Cloud

1. In Bruin Cloud dashboard, click "New Project"
2. Select "Connect GitHub Repository"
3. Choose your `my-taxi-pipeline` repo
4. Grant permissions

#### Step 4: Configure Cloud Connections

Your local `.bruin.yml` has secrets (database passwords). For production:

**Option A: Use Environment Variables**

In Bruin Cloud UI:
1. Go to Settings → Environment Variables
2. Add your secrets:
   - `BQ_PROJECT`: your-gcp-project
   - `BQ_CREDENTIALS`: (paste JSON key)

Update `.bruin.yml` to use variables:
```yaml
environments:
  production:
    connections:
      bigquery:
        name: bigquery-prod
        project: ${BQ_PROJECT}
        credentials: ${BQ_CREDENTIALS}
```

**Option B: Configure in Bruin Cloud UI**

The web interface lets you add connections securely without storing secrets in code.

#### Step 5: Deploy!

Once connected, Bruin Cloud will:
1. Detect new commits to your repo
2. Automatically deploy changes
3. Run pipelines on your defined schedule

### Cloud Dashboard Features

```
┌─────────────────────────────────────────────────────────────────────┐
│                      BRUIN CLOUD DASHBOARD                          │
│                                                                      │
│  ┌─────────────────────────────────────┐  ┌──────────────────────┐  │
│  │           Pipeline Runs              │  │    Alerts           │  │
│  │                                      │  │                     │  │
│  │  ✓ Jan 15, 2024 08:00 - Success     │  │  ⚠️ 1 warning       │  │
│  │  ✓ Jan 14, 2024 08:00 - Success     │  │     yesterday       │  │
│  │  ✗ Jan 13, 2024 08:00 - FAILED      │  │                     │  │
│  │  ✓ Jan 12, 2024 08:00 - Success     │  │  📧 Email: ON       │  │
│  │                                      │  │  💬 Slack: ON       │  │
│  └─────────────────────────────────────┘  └──────────────────────┘  │
│                                                                      │
│  ┌─────────────────────────────────────┐  ┌──────────────────────┐  │
│  │           Lineage View               │  │    Quick Stats      │  │
│  │                                      │  │                     │  │
│  │   ingestion → staging → reports     │  │  Runs today: 3      │  │
│  │                                      │  │  Success rate: 97%  │  │
│  └─────────────────────────────────────┘  └──────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

### Environments: Dev vs Production

Best practice: separate configurations for different environments:

```yaml
# .bruin.yml

environments:
  # Development: local DuckDB for testing
  default:
    connections:
      duckdb:
        name: duckdb-local
        path: ./data/dev.duckdb
  
  # Production: BigQuery for real data
  production:
    connections:
      bigquery:
        name: bigquery-prod
        project: ${BQ_PROJECT}
        location: US
        credentials: ${BQ_CREDENTIALS}
```

**Running locally (dev):**
```bash
bruin run  # Uses 'default' environment
```

**Running in cloud (prod):**
```bash
bruin run --environment production
```

---

## Part 10: Using Bruin MCP with AI Agents 🤖

### What is Bruin MCP?

**MCP** stands for **Model Context Protocol**. It's a way for AI assistants (like GitHub Copilot, Cursor, or Claude) to "talk" to Bruin directly!

Think of it like teaching an AI assistant how to use Bruin:

```
Without MCP:
┌─────────────────┐    ┌─────────────────┐
│   AI Assistant  │    │   Your Code     │
│                 │ ✗  │                 │
│  "I can help    │    │  bruin project  │
│   write code    │    │                 │
│   but I don't   │    │                 │
│   know Bruin"   │    │                 │
└─────────────────┘    └─────────────────┘

With MCP:
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   AI Assistant  │───▶│    Bruin MCP    │───▶│   Your Code     │
│                 │    │    (bridge)     │    │                 │
│  "I can read,   │    │                 │    │  bruin project  │
│   write, and    │    │  Translates     │    │                 │
│   run Bruin!"   │    │  AI → Bruin     │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Why Use AI with Bruin?

| Without AI | With AI + MCP |
|------------|---------------|
| Read documentation | Ask "how do I do X?" |
| Write SQL manually | "Create a staging table that cleans trips" |
| Debug by reading logs | "Why did my pipeline fail?" |
| Remember syntax | AI knows the syntax |

### Setup in VS Code

#### Step 1: Install the Bruin Extension

1. Open VS Code
2. Go to Extensions (Ctrl+Shift+X)
3. Search "Bruin"
4. Install the official Bruin extension

#### Step 2: Enable MCP

1. Open VS Code Settings (Ctrl+,)
2. Search for "Bruin MCP"
3. Check "Enable Bruin MCP"

#### Step 3: Start Using AI

Now you can chat with AI about your pipeline!

### Example Conversations

Here's what you can do with AI + Bruin MCP:

#### 1. Create Assets with Natural Language

```
You: "Create a staging table that cleans the taxi trips data. 
      Remove rows with negative fares and add a quality check for not_null 
      on pickup_datetime."

AI: I'll create that staging asset for you.

[Creates file: staging/trips.sql]
/* @bruin
name: staging.trips
type: duckdb.sql
depends:
  - ingestion.trips
columns:
  - name: pickup_datetime
    checks:
      - name: not_null
@bruin */

SELECT *
FROM raw.trips
WHERE fare_amount > 0
```

#### 2. Debug Pipeline Failures

```
You: "My pipeline failed. What went wrong?"

AI: I checked the last run. The quality check on 'staging.trips' failed:

    ❌ "positive" check on column "fare_amount"
    Found 23 rows with values <= 0
    
    Here are some examples:
    - Row 1: fare_amount = -5.50
    - Row 2: fare_amount = 0.00
    
    This is happening because the source data contains these records.
    
    To fix this, you could either:
    1. Filter them out in your WHERE clause (recommended)
    2. Remove the quality check (not recommended)
```

#### 3. Understand Your Pipeline

```
You: "Show me the lineage for daily_summary"

AI: Here's the dependency graph for reports.daily_summary:

    ingestion.trips (python)
         │
         ▼
    staging.trips (sql)
         │
         ▼
    reports.daily_summary (sql)
    
    This means:
    1. First, trips are downloaded (ingestion.trips)
    2. Then cleaned (staging.trips)  
    3. Finally aggregated into daily metrics (reports.daily_summary)
```

#### 4. Query Data Conversationally

```
You: "What was the total revenue last week?"

AI: I ran this query against your DuckDB database:

    SELECT SUM(total_revenue) as total
    FROM reports.daily_summary
    WHERE trip_date >= '2024-01-08'
      AND trip_date < '2024-01-15'
    
    Result: $14,523,897.50
```

#### 5. Generate Boilerplate

```
You: "I need a Python asset that downloads data from an API"

AI: Here's a template for an API ingestion asset:

[Creates file: ingestion/api_data.py]
""" @bruin
name: ingestion.api_data
type: python
description: Download data from external API
depends: []
@bruin """

import httpx
import duckdb

def main():
    # Configure your API
    url = "https://api.example.com/data"
    headers = {"Authorization": "Bearer YOUR_TOKEN"}
    
    # Fetch data
    response = httpx.get(url, headers=headers)
    data = response.json()
    
    # Load into DuckDB
    conn = duckdb.connect("data/warehouse.duckdb")
    # ... continue from here
```

### Tips for Effective AI Collaboration

| ✅ Do | ❌ Don't |
|------|---------|
| Be specific about what you want | Give vague instructions |
| Mention column names, table names | Expect AI to guess your schema |
| Ask follow-up questions | Accept first answer without review |
| Review generated code | Blindly copy-paste |
| Use AI for boilerplate | Replace understanding with AI |

---

## Quick Reference 📋

### Project Files at a Glance

```
my-pipeline/
├── .bruin.yml          ← "WHERE are my databases?"
├── pipeline/
│   ├── pipeline.yml    ← "WHEN and HOW to run?"
│   └── assets/
│       ├── *.sql       ← SQL transformations
│       ├── *.py        ← Python scripts
│       └── *.yaml      ← Pre-built ingestors
└── data/
    └── warehouse.duckdb ← Local database file
```

### File Reference

| File | What It Configures | Key Settings |
|------|-------------------|--------------|
| `.bruin.yml` | Database connections | `environments`, `connections` |
| `pipeline.yml` | Pipeline behavior | `name`, `schedule`, `variables` |
| `*.sql` | SQL transformations | `name`, `type`, `depends`, `columns` |
| `*.py` | Python scripts | `name`, `type`, `depends` + `main()` |
| `*.yaml` | Ingestors | `name`, `type: ingestr`, `parameters` |

### Command Cheat Sheet

```bash
# ═══════════════════════════════════════════════════════════
# 🚀 GETTING STARTED
# ═══════════════════════════════════════════════════════════

# Install Bruin
curl -LsSf https://getbruin.com/install/cli | sh

# Create new project
bruin init zoomcamp my-pipeline
cd my-pipeline

# ═══════════════════════════════════════════════════════════
# ✅ VALIDATE (before running)
# ═══════════════════════════════════════════════════════════

bruin validate                    # Check all configs

# ═══════════════════════════════════════════════════════════
# ▶️ RUN PIPELINES
# ═══════════════════════════════════════════════════════════

bruin run                         # Run entire pipeline
bruin run staging.trips           # Run one asset
bruin run staging.trips --downstream   # Include dependents
bruin run staging.trips --upstream     # Include dependencies
bruin run --full-refresh          # Rebuild everything

# ═══════════════════════════════════════════════════════════
# 📅 DATE RANGES
# ═══════════════════════════════════════════════════════════

bruin run --start-date 2024-01-15 --end-date 2024-01-16

# ═══════════════════════════════════════════════════════════
# 🔧 VARIABLES
# ═══════════════════════════════════════════════════════════

bruin run --var 'key=value'
bruin run --var 'array=["a","b"]'

# ═══════════════════════════════════════════════════════════
# 🔍 INSPECT
# ═══════════════════════════════════════════════════════════

bruin lineage                     # View dependency graph
```

### Asset Header Template

Use this as a starting point for new assets:

```sql
/* @bruin

name: schema.table_name           # Required
type: duckdb.sql                  # Required

description: What this asset does # Optional but recommended

depends:                          # What runs first
  - other.asset

materialization:                  # How to store
  type: table
  strategy: time_interval         # append, replace, time_interval
  incremental_key: date_column

columns:                          # Quality checks
  - name: column_name
    description: What it is
    checks:
      - name: not_null
      - name: unique

@bruin */

-- Your SQL here
SELECT * FROM source_table
```

### Materialization Quick Guide

| Strategy | When Data Changes | What Happens |
|----------|-------------------|--------------|
| `view` | Every query | SQL runs fresh each time |
| `table` (default) | Each run | DROP + CREATE (full rebuild) |
| `append` | Add only | INSERT new rows |
| `replace` | Full refresh | TRUNCATE + INSERT |
| `time_interval` | Incremental | DELETE time range + INSERT |

**Rule of thumb:**
- Use `time_interval` for time-series data (trips, transactions)
- Use `table` for small reference tables
- Use `view` for real-time calculations

### Quality Checks Quick Guide

| Check | Use When | Catches |
|-------|----------|---------|
| `not_null` | Required columns | Missing data |
| `unique` | IDs, keys | Duplicates |
| `positive` | Amounts, counts | Zero or negative values |
| `accepted_values` | Categories | Invalid codes |

### Common Troubleshooting

| Problem | Likely Cause | Solution |
|---------|--------------|----------|
| "command not found: bruin" | Not in PATH | Restart terminal |
| "connection failed" | Wrong config | Check `.bruin.yml` paths |
| "asset not found" | Wrong name | Check `depends:` spelling |
| "dependency cycle" | A → B → A | Remove circular dependency |
| Pipeline runs but no data | Incremental on empty table | Use `--full-refresh` |
| Quality check failed | Bad data in source | Fix data or adjust check |

### Helpful SQL Patterns

```sql
-- Filter by date range (for incremental)
WHERE pickup_datetime >= '{{ start_date }}'
  AND pickup_datetime < '{{ end_date }}'

-- Handle NULLs in division
AVG(tip / NULLIF(fare, 0))

-- Generate unique ID from multiple columns
md5(CAST(col1 AS VARCHAR) || CAST(col2 AS VARCHAR))

-- Conditional filtering with Jinja
{% if include_cancelled == false %}
AND status != 'cancelled'
{% endif %}
```

---

## Key Takeaways 🎓

### What You Learned

1. **Data Platforms** unify ingestion, transformation, orchestration, and quality in one tool
2. **Bruin** is an open-source data platform with a simple CLI
3. **Assets** are files (SQL/Python/YAML) that create tables/views
4. **Materialization strategies** control how data is stored and updated
5. **Variables** make pipelines flexible and reusable
6. **Quality checks** catch bad data before it reaches analysts
7. **Lineage** shows how data flows through your pipeline

### Core Concepts Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                     BRUIN MENTAL MODEL                          │
│                                                                  │
│   .bruin.yml      →  WHERE (database connections)               │
│   pipeline.yml    →  WHEN (schedule) + HOW (variables)          │
│   assets/*.sql    →  WHAT (transformations)                     │
│   columns.checks  →  VALIDATION (quality gates)                 │
│   depends:        →  ORDER (run this first)                     │
│   materialization →  STORAGE (table, view, incremental)         │
└─────────────────────────────────────────────────────────────────┘
```

### When to Use Each Feature

| Situation | Feature to Use |
|-----------|----------------|
| Download data from API | Python asset |
| Transform with SQL | SQL asset |
| Import from SaaS tool | YAML ingestor |
| Process daily data | `time_interval` strategy |
| Rebuild entire table | `--full-refresh` |
| Validate data exists | `not_null` check |
| Prevent duplicates | `unique` check |
| Test one change | `bruin run asset.sql` |
| Push changes through | `--downstream` flag |

---

## Resources 📚

### Official Documentation

- [Bruin Documentation](https://getbruin.com/docs) - Complete reference
- [Bruin GitHub Repository](https://github.com/bruin-data/bruin) - Source code & issues
- [Bruin MCP Guide](https://getbruin.com/docs/bruin/getting-started/bruin-mcp) - AI integration
- [Bruin Cloud](https://getbruin.com/) - Managed deployment

### Video Tutorials

| Video | Topics Covered |
|-------|----------------|
| 5.1 - Introduction | What is Bruin, modern data stack |
| 5.2 - Getting Started | Installation, VS Code extension |
| 5.3 - NYC Taxi Pipeline | Full pipeline walkthrough |
| 5.4 - Bruin MCP | Using AI to build pipelines |
| 5.5 - Cloud Deployment | Production deployment |

### Practice Exercises

1. **Basic**: Create a pipeline that downloads a CSV and creates a staging table
2. **Intermediate**: Add quality checks and incremental loading
3. **Advanced**: Deploy to Bruin Cloud with scheduled runs

### Related Technologies

| Tool | Relationship to Bruin |
|------|----------------------|
| **dbt** | Similar transform layer, but Bruin adds ingestion |
| **Airflow** | Bruin has built-in orchestration |
| **Great Expectations** | Bruin has built-in quality checks |
| **DuckDB** | Great for local development with Bruin |
| **BigQuery** | Common production destination |

### Community

- [Data Engineering Zoomcamp](https://github.com/DataTalksClub/data-engineering-zoomcamp) - This course!
- [DataTalks.Club Slack](https://datatalks.club/slack.html) - Ask questions
- [Bruin Discord](https://discord.gg/bruin) - Bruin-specific help

---

## Next Steps 🚀

After completing this module, you can:

1. ✅ Build local pipelines with DuckDB
2. ✅ Add quality checks to catch bad data
3. ✅ Use incremental loading for efficiency
4. ✅ Deploy to Bruin Cloud for production

**Ready for the next module?** Continue to Module 6: Stream Processing!
