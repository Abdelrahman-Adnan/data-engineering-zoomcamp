# Module 4 Summary - Analytics Engineering with dbt

#DataEngineeringZoomcamp #dbt #AnalyticsEngineering #DataModeling

---

## Part 1: Introduction to Analytics Engineering & dbt Fundamentals 🎯

### What is Analytics Engineering?

### The Evolution of Data Roles

Traditionally, there were two main roles in data:

| Role | Focus | Skills |
|------|-------|--------|
| **Data Engineer** | Building pipelines, infrastructure, data movement | Python, Spark, Airflow, cloud services |
| **Data Analyst** | Creating reports, dashboards, insights | SQL, Excel, BI tools |

But there was a gap! Who transforms the raw data into clean, analysis-ready tables? Enter the **Analytics Engineer**.

### What Does an Analytics Engineer Do?

An Analytics Engineer sits between Data Engineering and Data Analytics:

```
┌─────────────────┐     ┌──────────────────────┐     ┌─────────────────┐
│  Data Engineer  │ ──► │  Analytics Engineer  │ ──► │   Data Analyst  │
│                 │     │                      │     │                 │
│  • Pipelines    │     │  • Transform data    │     │  • Dashboards   │
│  • Infrastructure│    │  • Data modeling     │     │  • Reports      │
│  • Data movement│     │  • Quality tests     │     │  • Insights     │
└─────────────────┘     │  • Documentation     │     └─────────────────┘
                        └──────────────────────┘
```

**Key responsibilities:**
- 📊 Transform raw data into clean, modeled datasets
- 🧪 Write tests to ensure data quality
- 📝 Document everything so others can understand
- 🔗 Build the "T" in ELT (Extract, Load, Transform)

### The Kitchen Analogy 🍳

Think of a data warehouse like a restaurant:

| Restaurant | Data Warehouse | Who accesses it |
|------------|----------------|-----------------|
| **Pantry** (raw ingredients) | Staging area (raw data) | Data Engineers |
| **Kitchen** (cooking happens) | Processing area (transformations) | Analytics Engineers |
| **Dining Hall** (served dishes) | Presentation area (final tables) | Business users, Analysts |

Raw ingredients (data) come in, get processed (transformed), and are served as polished dishes (analytics-ready tables).

---

## What is dbt? 🛠️

**dbt** stands for **data build tool**. It's the most popular tool for analytics engineering.

### The Problems dbt Solves

Before dbt, data transformation was messy:
- ❌ SQL scripts scattered everywhere with no organization
- ❌ No version control (changes got lost)
- ❌ No testing (errors discovered too late)
- ❌ No documentation (nobody knew what anything meant)
- ❌ No environments (changes went straight to production!)

**dbt brings software engineering best practices to analytics:**
- ✅ **Version control** - Your SQL lives in Git
- ✅ **Modularity** - Reusable pieces instead of copy-paste
- ✅ **Testing** - Automated data quality checks
- ✅ **Documentation** - Generated from your code
- ✅ **Environments** - Separate dev and prod

### How dbt Works

dbt follows a simple principle: **write SQL, dbt handles the rest**.

```
┌─────────────────────────────────────────────────────────────┐
│                     Your dbt Project                        │
│                                                             │
│   ┌───────────────┐    ┌───────────────┐    ┌────────────┐ │
│   │  models/*.sql │───►│   dbt compile │───►│ SQL Queries│ │
│   │  (your logic) │    │   dbt run     │    │ (executed) │ │
│   └───────────────┘    └───────────────┘    └────────────┘ │
│                              │                              │
│                              ▼                              │
│                    ┌──────────────────┐                     │
│                    │  Data Warehouse  │                     │
│                    │  (views/tables)  │                     │
│                    └──────────────────┘                     │
└─────────────────────────────────────────────────────────────┘
```

1. You write SQL files (called "models")
2. dbt compiles them (adds warehouse-specific syntax)
3. dbt runs them against your data warehouse
4. Views/tables are created automatically!

### dbt Core vs dbt Cloud

| Feature | dbt Core | dbt Cloud |
|---------|----------|-----------|
| **Cost** | Free (open source) | Free tier + paid plans |
| **Where it runs** | Your machine/server | Cloud-hosted |
| **Setup** | Manual installation | Browser-based IDE |
| **Scheduling** | Need external tool | Built-in scheduler |
| **Best for** | Local development, cost savings | Teams, ease of use |

💡 **For this course:** You can use either! Local setup uses DuckDB + dbt Core (free). Cloud setup uses BigQuery + dbt Cloud.

---

## Part 2: dbt Project Structure & Building Models 📁

### Why Model Data? 📐

Raw data is messy and hard to query. Dimensional modeling organizes data into a structure that's:
- Easy to understand
- Fast to query
- Flexible for different analyses

### Fact Tables vs Dimension Tables

This is the core of dimensional modeling (also called "star schema"):

**Fact Tables (`fct_`)**
- Contain **measurements** or **events**
- One row per thing that happened
- Usually have many rows (millions/billions)
- Contain numeric values you want to analyze

**Examples:**
- `fct_trips` - one row per taxi trip
- `fct_sales` - one row per sale
- `fct_orders` - one row per order

```sql
-- Example fact table
CREATE TABLE fct_trips AS
SELECT
    trip_id,           -- unique identifier
    pickup_datetime,   -- when it happened
    dropoff_datetime,
    pickup_zone_id,    -- foreign keys to dimensions
    dropoff_zone_id,
    fare_amount,       -- numeric measures
    tip_amount,
    total_amount
FROM transformed_trips;
```

**Dimension Tables (`dim_`)**
- Contain **attributes** or **descriptive information**
- One row per entity
- Usually fewer rows
- Provide context for fact tables

**Examples:**
- `dim_zones` - one row per taxi zone
- `dim_customers` - one row per customer
- `dim_products` - one row per product

```sql
-- Example dimension table
CREATE TABLE dim_zones AS
SELECT
    location_id,       -- primary key
    borough,           -- descriptive attributes
    zone_name,
    service_zone
FROM zone_lookup;
```

### The Star Schema ⭐

When you join facts and dimensions, you get a star shape:

```
                    ┌──────────────┐
                    │  dim_zones   │
                    │  (pickup)    │
                    └───────┬──────┘
                            │
┌──────────────┐    ┌───────┴──────┐    ┌──────────────┐
│  dim_vendors │────│  fct_trips   │────│  dim_zones   │
│              │    │  (center)    │    │  (dropoff)   │
└──────────────┘    └───────┬──────┘    └──────────────┘
                            │
                    ┌───────┴──────┐
                    │ dim_payment  │
                    │    types     │
                    └──────────────┘
```

**Why it's powerful:**
```sql
-- Easy to answer business questions!
SELECT 
    z.borough,
    COUNT(*) as trip_count,
    SUM(f.total_amount) as total_revenue
FROM fct_trips f
JOIN dim_zones z ON f.pickup_zone_id = z.location_id
GROUP BY z.borough
ORDER BY total_revenue DESC;
```

---

### dbt Project Structure

A dbt project has a specific folder structure. Understanding this helps you navigate any project:

```
taxi_rides_ny/
├── dbt_project.yml      # Project configuration (most important!)
├── profiles.yml         # Database connection (often in ~/.dbt/)
├── packages.yml         # External packages to install
│
├── models/              # ⭐ YOUR SQL MODELS LIVE HERE
│   ├── staging/         # Raw data, minimally cleaned
│   ├── intermediate/    # Complex transformations
│   └── marts/           # Final, business-ready tables
│
├── seeds/               # CSV files to load as tables
├── macros/              # Reusable SQL functions
├── tests/               # Custom test files
├── snapshots/           # Track data changes over time
└── analysis/            # Ad-hoc queries (not built)
```

### The `dbt_project.yml` File

This is the **most important file** - dbt looks for it first:

```yaml
name: 'taxi_rides_ny'
version: '1.0.0'
profile: 'taxi_rides_ny'  # Must match profiles.yml!

# Default configurations
models:
  taxi_rides_ny:
    staging:
      materialized: view  # Staging models become views
    marts:
      materialized: table # Mart models become tables
```

### The Three Model Layers

dbt recommends organizing models into three layers:

**1. Staging Layer (`staging/`)**

**Purpose:** Clean copy of raw data with minimal transformations

**What happens here:**
- Rename columns (snake_case, clear names)
- Cast data types
- Filter obviously bad data
- Keep 1:1 with source (same rows, similar columns)

```sql
-- models/staging/stg_green_tripdata.sql
{{ config(materialized='view') }}

with tripdata as (
    select * 
    from {{ source('staging', 'green_tripdata') }}
    where vendorid is not null  -- filter bad data
)

select
    -- Rename and cast columns
    cast(vendorid as integer) as vendor_id,
    cast(lpep_pickup_datetime as timestamp) as pickup_datetime,
    cast(lpep_dropoff_datetime as timestamp) as dropoff_datetime,
    cast(pulocationid as integer) as pickup_location_id,
    cast(dolocationid as integer) as dropoff_location_id,
    cast(passenger_count as integer) as passenger_count,
    cast(trip_distance as numeric) as trip_distance,
    cast(fare_amount as numeric) as fare_amount,
    cast(total_amount as numeric) as total_amount
from tripdata
```

**2. Intermediate Layer (`intermediate/`)**

**Purpose:** Complex transformations, joins, business logic

**What happens here:**
- Combine multiple staging models
- Apply business rules
- Heavy data manipulation
- NOT exposed to end users

```sql
-- models/intermediate/int_trips_unioned.sql
with green_trips as (
    select *, 'Green' as service_type
    from {{ ref('stg_green_tripdata') }}
),

yellow_trips as (
    select *, 'Yellow' as service_type
    from {{ ref('stg_yellow_tripdata') }}
)

select * from green_trips
union all
select * from yellow_trips
```

**3. Marts Layer (`marts/`)**

**Purpose:** Final, business-ready tables for end users

**What happens here:**
- Final fact and dimension tables
- Ready for dashboards and reports
- Only these should be exposed to BI tools!

```sql
-- models/marts/fct_trips.sql
{{ config(materialized='table') }}

select
    t.trip_id,
    t.service_type,
    t.pickup_datetime,
    t.dropoff_datetime,
    t.pickup_location_id,
    t.dropoff_location_id,
    z_pickup.zone as pickup_zone,
    z_dropoff.zone as dropoff_zone,
    t.passenger_count,
    t.trip_distance,
    t.fare_amount,
    t.total_amount
from {{ ref('int_trips_unioned') }} t
left join {{ ref('dim_zones') }} z_pickup 
    on t.pickup_location_id = z_pickup.location_id
left join {{ ref('dim_zones') }} z_dropoff 
    on t.dropoff_location_id = z_dropoff.location_id
```

---

### Sources and the `source()` Function 📥

### What are Sources?

Sources tell dbt where your raw data lives in the warehouse. They're defined in YAML files:

```yaml
# models/staging/sources.yml
version: 2

sources:
  - name: staging           # Logical name (you choose)
    database: my_project    # Your GCP project or database
    schema: nytaxi          # BigQuery dataset or schema
    tables:
      - name: green_tripdata
      - name: yellow_tripdata
```

### Using the `source()` Function

Instead of hardcoding table names, use `source()`:

```sql
-- ❌ Bad - hardcoded path
SELECT * FROM my_project.nytaxi.green_tripdata

-- ✅ Good - using source()
SELECT * FROM {{ source('staging', 'green_tripdata') }}
```

**Benefits:**
- Change database/schema in one place (YAML file)
- dbt tracks dependencies automatically
- Can add freshness tests on sources

---

### The `ref()` Function - Building Dependencies 🔗

This is **the most important dbt function!**

### `source()` vs `ref()`

| Function | Use When | Example |
|----------|----------|---------|
| `source()` | Reading raw/external data | `{{ source('staging', 'green_tripdata') }}` |
| `ref()` | Reading another dbt model | `{{ ref('stg_green_tripdata') }}` |

### How `ref()` Works

```sql
-- models/marts/fct_trips.sql
select *
from {{ ref('int_trips_unioned') }}  -- References the int_trips_unioned model
```

**What `ref()` does:**
1. ✅ Resolves to the correct schema/table name
2. ✅ Builds the dependency graph automatically
3. ✅ Ensures models run in the correct order

### The DAG (Directed Acyclic Graph)

dbt builds a **dependency graph** from your `ref()` calls:

```
┌──────────────────┐     ┌──────────────────┐
│ stg_green_trips  │     │ stg_yellow_trips │
└────────┬─────────┘     └────────┬─────────┘
         │                        │
         └──────────┬─────────────┘
                    │
                    ▼
         ┌──────────────────┐
         │ int_trips_unioned│
         └────────┬─────────┘
                  │
                  ▼
         ┌──────────────────┐
         │    fct_trips     │
         └──────────────────┘
```

When you run `dbt build`, models run in dependency order automatically!

---

### Seeds - Loading CSV Files 🌱

Seeds let you load small CSV files into your warehouse as tables.

### When to Use Seeds

✅ **Good use cases:**
- Lookup tables (zone names, country codes)
- Static mappings (vendor ID → vendor name)
- Small reference data that rarely changes

❌ **Not good for:**
- Large datasets (use proper data loading)
- Frequently changing data

### How to Use Seeds

1. **Put CSV files in the `seeds/` folder:**

```
seeds/
└── taxi_zone_lookup.csv
```

```csv
locationid,borough,zone,service_zone
1,EWR,Newark Airport,EWR
2,Queens,Jamaica Bay,Boro Zone
3,Bronx,Allerton/Pelham Gardens,Boro Zone
...
```

2. **Run `dbt seed`:**

```bash
dbt seed
```

3. **Reference in models using `ref()`:**

```sql
-- models/marts/dim_zones.sql
select
    locationid as location_id,
    borough,
    zone,
    service_zone
from {{ ref('taxi_zone_lookup') }}
```

---

## Part 3: Testing, Documentation & Deployment 🚀

### Macros - Reusable SQL Functions 🔧

Macros are like functions in Python - write once, use everywhere.

### Why Use Macros?

Without macros, you repeat code:

```sql
-- ❌ Repeated everywhere
CASE 
    WHEN payment_type = 1 THEN 'Credit card'
    WHEN payment_type = 2 THEN 'Cash'
    WHEN payment_type = 3 THEN 'No charge'
    WHEN payment_type = 4 THEN 'Dispute'
    WHEN payment_type = 5 THEN 'Unknown'
    ELSE 'Unknown'
END as payment_type_description
```

With macros, write it once:

```sql
-- macros/get_payment_type_description.sql
{% macro get_payment_type_description(payment_type) %}
    CASE {{ payment_type }}
        WHEN 1 THEN 'Credit card'
        WHEN 2 THEN 'Cash'
        WHEN 3 THEN 'No charge'
        WHEN 4 THEN 'Dispute'
        WHEN 5 THEN 'Unknown'
        ELSE 'Unknown'
    END
{% endmacro %}
```

Use it in any model:

```sql
-- models/staging/stg_green_tripdata.sql
select
    payment_type,
    {{ get_payment_type_description('payment_type') }} as payment_type_description
from {{ source('staging', 'green_tripdata') }}
```

### Jinja Templating

dbt uses **Jinja** - a Python templating language. You'll recognize it by `{{ }}` and `{% %}`:

| Syntax | Purpose | Example |
|--------|---------|---------|
| `{{ }}` | Output expression | `{{ ref('my_model') }}` |
| `{% %}` | Logic/control flow | `{% if is_incremental() %}` |
| `{# #}` | Comments | `{# This is a comment #}` |

---

### dbt Packages - Community Libraries 📦

Packages let you use macros and models built by others.

### Popular Packages

| Package | What it Does |
|---------|--------------|
| **dbt_utils** | Common SQL helpers (surrogate keys, pivot, etc.) |
| **dbt_codegen** | Auto-generate YAML and SQL |
| **dbt_expectations** | Great Expectations-style tests |
| **dbt_audit_helper** | Compare model outputs when refactoring |

### Installing Packages

1. **Create `packages.yml`:**

```yaml
packages:
  - package: dbt-labs/dbt_utils
    version: 1.1.1
```

2. **Run `dbt deps`:**

```bash
dbt deps
```

3. **Use the macros:**

```sql
-- Using dbt_utils to generate surrogate keys
select
    {{ dbt_utils.generate_surrogate_key(['vendorid', 'pickup_datetime']) }} as trip_id,
    *
from {{ source('staging', 'green_tripdata') }}
```

---

### Testing in dbt 🧪

Tests ensure your data meets expectations. dbt has several test types:

**1. Generic Tests (Most Common)**

Built-in tests you apply in YAML:

```yaml
# models/staging/schema.yml
version: 2

models:
  - name: stg_green_tripdata
    columns:
      - name: trip_id
        tests:
          - unique       # No duplicate values
          - not_null     # No null values
      
      - name: payment_type
        tests:
          - accepted_values:
              values: [1, 2, 3, 4, 5, 6]  # Only these values allowed
      
      - name: pickup_location_id
        tests:
          - relationships:  # Referential integrity
              to: ref('dim_zones')
              field: location_id
```

**The four built-in tests:**
| Test | What it Checks |
|------|----------------|
| `unique` | No duplicate values in column |
| `not_null` | No NULL values in column |
| `accepted_values` | Values must be in specified list |
| `relationships` | Values must exist in another table |

**2. Singular Tests**

Custom SQL tests in the `tests/` folder:

```sql
-- tests/assert_positive_fare_amount.sql
-- Test FAILS if any rows are returned

select
    trip_id,
    fare_amount
from {{ ref('fct_trips') }}
where fare_amount < 0  -- Find negative fares (bad data!)
```

**3. Source Freshness Tests**

Check if your source data is up to date:

```yaml
sources:
  - name: staging
    tables:
      - name: green_tripdata
        freshness:
          warn_after: {count: 24, period: hour}
          error_after: {count: 48, period: hour}
        loaded_at_field: pickup_datetime
```

### Running Tests

```bash
# Run all tests
dbt test

# Run tests for specific model
dbt test --select stg_green_tripdata

# Run tests and models together
dbt build
```

---

### Documentation 📝

dbt generates beautiful documentation automatically!

### Adding Descriptions

In your schema YAML:

```yaml
version: 2

models:
  - name: fct_trips
    description: >
      Fact table containing all taxi trips (yellow and green).
      One row per trip with fare details and zone information.
    
    columns:
      - name: trip_id
        description: Unique identifier for each trip (surrogate key)
      
      - name: service_type
        description: Type of taxi service - 'Yellow' or 'Green'
      
      - name: total_amount
        description: Total trip cost including fare, tips, taxes, and fees
```

### Generating Docs

```bash
# Generate documentation
dbt docs generate

# Serve locally (opens browser)
dbt docs serve
```

This creates an interactive website with:
- Model descriptions
- Column definitions
- Dependency graph (visual DAG)
- Source information

---

### Essential dbt Commands 💻

### The Big Four

| Command | What it Does |
|---------|--------------|
| `dbt run` | Build all models (create views/tables) |
| `dbt test` | Run all tests |
| `dbt build` | Run + test together (recommended!) |
| `dbt compile` | Generate SQL without executing |

### Other Useful Commands

```bash
# Check connection
dbt debug

# Load seed files
dbt seed

# Install packages
dbt deps

# Generate docs
dbt docs generate

# Retry failed models
dbt retry
```

### Selecting Specific Models

Use `--select` (or `-s`) to run specific models:

```bash
# Single model
dbt run --select stg_green_tripdata

# Model and all upstream dependencies
dbt run --select +fct_trips

# Model and all downstream models
dbt run --select stg_green_tripdata+

# Both directions
dbt run --select +fct_trips+

# All models in a folder
dbt run --select staging.*

# Multiple models
dbt run --select stg_green_tripdata stg_yellow_tripdata
```

### Target Environments

```bash
# Development (default)
dbt run

# Production
dbt run --target prod
```

---

### Materializations - Views vs Tables 📊

Materialization controls how dbt persists your models in the warehouse.

### Types of Materializations

| Type | What it Creates | Use Case |
|------|-----------------|----------|
| **view** | SQL view (query stored, runs on access) | Staging models, frequently changing logic |
| **table** | Physical table (data stored) | Final marts, large datasets, performance |
| **incremental** | Appends new data only | Very large tables, event data |
| **ephemeral** | Not created (CTE in downstream) | Helper models, intermediate steps |

### Setting Materializations

**In the model file:**
```sql
{{ config(materialized='table') }}

select * from {{ ref('stg_trips') }}
```

**In dbt_project.yml (project-wide):**
```yaml
models:
  my_project:
    staging:
      materialized: view
    marts:
      materialized: table
```

### View vs Table Decision

```
┌─────────────────────────────────────────────────────────────┐
│                 Should I use view or table?                  │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
              ┌──────────────────────────┐
              │ Is the query expensive?  │
              └──────────────────────────┘
                     │            │
                    Yes          No
                     │            │
                     ▼            ▼
               ┌─────────┐  ┌─────────┐
               │  TABLE  │  │  VIEW   │
               └─────────┘  └─────────┘
```

**Use VIEW when:**
- Staging models (simple transformations)
- Logic changes frequently
- Storage cost is a concern

**Use TABLE when:**
- Final marts queried often
- Complex joins/aggregations
- Query performance matters

---

### Putting It All Together - The NYC Taxi Project 🚕

In this module, we build a complete dbt project for NYC taxi data:

### What We Build

```
┌──────────────────────────────────────────────────────────────┐
│                      RAW DATA                                 │
│  green_tripdata (GCS/BigQuery) │ yellow_tripdata (GCS/BigQuery)│
└───────────────────┬─────────────────────┬────────────────────┘
                    │                     │
                    ▼                     ▼
┌──────────────────────────────────────────────────────────────┐
│                    STAGING LAYER                              │
│      stg_green_tripdata    │    stg_yellow_tripdata          │
│      (cleaned, renamed)    │    (cleaned, renamed)           │
└───────────────────┬─────────────────────┬────────────────────┘
                    │                     │
                    └──────────┬──────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────┐
│                  INTERMEDIATE LAYER                           │
│                   int_trips_unioned                           │
│            (green + yellow combined)                          │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────┐
│                      MARTS LAYER                              │
│  ┌─────────────┐  ┌───────────────┐  ┌─────────────────────┐ │
│  │ dim_zones   │  │   fct_trips   │  │fct_monthly_zone_rev │ │
│  │ (dimension) │  │    (fact)     │  │     (report)        │ │
│  └─────────────┘  └───────────────┘  └─────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```

### The Models We Create

| Model | Type | Description |
|-------|------|-------------|
| `stg_green_tripdata` | Staging | Cleaned green taxi data |
| `stg_yellow_tripdata` | Staging | Cleaned yellow taxi data |
| `int_trips_unioned` | Intermediate | Combined yellow + green trips |
| `dim_zones` | Dimension | Zone lookup table |
| `fct_trips` | Fact | One row per trip |
| `fct_monthly_zone_revenue` | Report | Monthly revenue by zone |

---

### Setup Options 🔧

### Option 1: Local Setup (DuckDB + dbt Core)

**Pros:** Free, no cloud account needed
**Cons:** Limited to your machine's power

```bash
# 1. Install dbt with DuckDB adapter
pip install dbt-duckdb

# 2. Clone the project
git clone https://github.com/DataTalksClub/data-engineering-zoomcamp
cd data-engineering-zoomcamp/04-analytics-engineering/taxi_rides_ny

# 3. Create profiles.yml in ~/.dbt/
# 4. Run dbt debug to test connection
dbt debug

# 5. Build the project
dbt build --target prod
```

### Option 2: Cloud Setup (BigQuery + dbt Cloud)

**Pros:** Powerful, team collaboration, scheduler
**Cons:** Requires GCP account (free tier available)

1. Create dbt Cloud account (free)
2. Connect to your BigQuery project
3. Clone the repo in dbt Cloud IDE
4. Run `dbt build --target prod`

---

### Troubleshooting Common Issues 🔍

### "Profile not found"
- Check `dbt_project.yml` profile name matches `profiles.yml`
- Ensure `profiles.yml` is in `~/.dbt/`

### "Source not found"
- Verify database/schema names in `sources.yml`
- Check your data is actually loaded in the warehouse

### "Model depends on model that was not found"
- Check for typos in `ref()` calls
- Ensure referenced model exists

### DuckDB Out of Memory
- Add memory settings to profiles.yml:
```yaml
settings:
  memory_limit: '2GB'
```

---

### Key Takeaways 🎓

1. **Analytics Engineering** bridges data engineering and data analysis

2. **dbt** brings software engineering best practices to SQL transformations

3. **Dimensional modeling** organizes data into facts (events) and dimensions (attributes)

4. **Three layers** - staging (raw copy), intermediate (transformations), marts (final)

5. **`ref()` and `source()`** are your main functions for building dependencies

6. **Testing** ensures data quality - use unique, not_null, accepted_values, relationships

7. **Documentation** is auto-generated from YAML descriptions

8. **`dbt build`** runs and tests everything in dependency order

---

### Additional Resources 📚

- [dbt Documentation](https://docs.getdbt.com/)
- [dbt Fundamentals Course](https://learn.getdbt.com/courses/dbt-fundamentals) (free)
- [SQL Refresher for Window Functions](https://github.com/DataTalksClub/data-engineering-zoomcamp/blob/main/04-analytics-engineering/refreshers/SQL.md)
- [dbt Community Slack](https://community.getdbt.com/)

---

## Submission

Homework form: https://courses.datatalks.club/de-zoomcamp-2026/homework/hw4
