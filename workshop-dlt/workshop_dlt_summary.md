# Workshop Summary - dlt (Data Load Tool): AI-Assisted Data Ingestion

#DataEngineeringZoomcamp #dlt #DataIngestion #DuckDB #RESTAPIs #Pipelines

---

## Part 1: Introduction to dlt (Data Load Tool) 🎯

### What is dlt?

Imagine you're a librarian. Every day, new books arrive from different publishers in different formats:
- Some come in **boxes** (JSON from APIs)
- Some come in **envelopes** (CSV files)
- Some come as **digital files** (database exports)

Your job is to:
1. **Receive** all these books (Extract)
2. **Catalog** them properly (Normalize)
3. **Put them on shelves** in the right section (Load)

**dlt** (data load tool) does exactly this for data! It's an open-source Python library that makes it easy to load data from any source into any destination.

```
┌─────────────────────────────────────────────────────────────────┐
│                        What dlt Does                             │
│                                                                  │
│   📥 EXTRACT          🔄 NORMALIZE          📦 LOAD             │
│   Get data from       Convert to clean      Put it into your     │
│   APIs, files,        structured tables     data warehouse       │
│   databases                                                      │
│                                                                  │
│   ┌─────────┐        ┌─────────────┐       ┌──────────────┐    │
│   │ REST API│  ───►  │ Flatten &   │ ───►  │   DuckDB     │    │
│   │ CSV File│        │ Type Infer  │       │   BigQuery   │    │
│   │ Database│        │ Deduplicate │       │   Postgres   │    │
│   └─────────┘        └─────────────┘       └──────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### Why dlt? The Problem It Solves

Before dlt, loading data from an API was painful:

```
❌ The Old Way (Manual & Error-Prone)
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   1. Write HTTP requests manually                               │
│   2. Handle pagination yourself                                 │
│   3. Parse nested JSON manually                                 │
│   4. Figure out data types                                      │
│   5. Create database tables by hand                             │
│   6. Handle errors, retries, duplicates                         │
│   7. Write INSERT statements                                    │
│                                                                  │
│   = 200+ lines of fragile code for ONE API!                     │
└─────────────────────────────────────────────────────────────────┘
```

With dlt:

```
✅ The dlt Way (Simple & Robust)
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   1. Define your API source (URL, pagination)                   │
│   2. Create a pipeline (source → destination)                   │
│   3. Run it!                                                    │
│                                                                  │
│   = ~30 lines of clean, maintainable code!                      │
└─────────────────────────────────────────────────────────────────┘
```

### dlt vs Other Tools

| Feature | dlt | Manual Python | Fivetran | Airbyte |
|---------|-----|---------------|----------|---------|
| **Cost** | Free (open source) | Free but time-consuming | $$$ SaaS | Free (self-host) |
| **Setup** | `pip install dlt` | N/A | Cloud signup | Docker + Kubernetes |
| **Custom APIs** | Easy (REST API source) | Lots of code | Limited | Connector needed |
| **AI-Assisted** | Yes (MCP Server) | No | No | No |
| **Schema Inference** | Automatic | Manual | Automatic | Automatic |
| **Pagination** | Built-in | Manual | Built-in | Built-in |
| **Learning Curve** | Low (Python) | Medium | Low (UI) | Medium |

### The dlt Pipeline Concept

A **pipeline** in dlt has three components:

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   SOURCE     │     │   PIPELINE   │     │ DESTINATION  │
│              │     │              │     │              │
│  Where data  │ ──► │  How to move │ ──► │  Where data  │
│  comes FROM  │     │  the data    │     │  goes TO     │
│              │     │              │     │              │
│  Examples:   │     │  • Name      │     │  Examples:   │
│  • REST API  │     │  • Settings  │     │  • DuckDB    │
│  • Database  │     │  • Schema    │     │  • BigQuery  │
│  • CSV files │     │              │     │  • Postgres  │
└──────────────┘     └──────────────┘     └──────────────┘
```

In Python, this looks like:

```python
import dlt

# 1. Define the SOURCE (where data comes from)
source = my_api_source()

# 2. Create the PIPELINE (how to move it)
pipeline = dlt.pipeline(
    pipeline_name="my_pipeline",
    destination="duckdb",           # Where to put it
    dataset_name="my_dataset"       # Schema/dataset name
)

# 3. RUN it!
load_info = pipeline.run(source)
print(load_info)
```

### What is DuckDB?

**DuckDB** is the destination we use in this workshop. Think of it as SQLite's big brother for analytics:

| Feature | DuckDB | SQLite | PostgreSQL |
|---------|--------|--------|------------|
| **Type** | Analytical (OLAP) | General (OLTP) | General (OLTP) |
| **Setup** | Zero config, single file | Zero config, single file | Server required |
| **Best for** | Analytics, large scans | Mobile apps, small data | Production apps |
| **Speed** | Very fast for analytics | Slow for big data | Good all-around |
| **Install** | `pip install duckdb` | Built into Python | Separate install |

> 🎓 **Beginner tip:** DuckDB stores everything in a single `.duckdb` file on your computer. No server, no Docker, no setup!

### What is AI-Assisted Development?

This workshop introduces a new way of building pipelines — using AI to write the code for you:

```
Traditional Development:
┌──────────┐     ┌──────────────────┐     ┌──────────┐
│   You    │ ──► │  Read docs for   │ ──► │  Write   │
│  (human) │     │  hours & hours   │     │  code    │
└──────────┘     └──────────────────┘     └──────────┘

AI-Assisted Development:
┌──────────┐     ┌──────────────────┐     ┌──────────┐
│   You    │ ──► │  Describe what   │ ──► │ AI writes│
│  (human) │     │  you want        │     │  code    │
└──────────┘     └──────────────────┘     └──────────┘
                  "Build a REST API          ▼
                   source for NYC        ┌──────────┐
                   taxi data..."         │ You debug│
                                         │ & refine │
                                         └──────────┘
```

The **dlt MCP Server** gives the AI access to:
- dlt documentation
- Code examples
- Your pipeline metadata
- Schema information

This means the AI can generate accurate dlt code because it understands the library!

---

## Part 2: Setting Up Your dlt Environment 🚀

### Prerequisites Checklist

Before starting, make sure you have:

| Requirement | How to Check | Install Command |
|-------------|-------------|-----------------|
| Python 3.11+ | `python --version` | [python.org](https://python.org) |
| pip | `pip --version` | Comes with Python |
| uv (recommended) | `uv --version` | `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| Agentic IDE | — | Cursor, VS Code + Copilot, or Claude Code |

### Step 1: Install uv (Recommended)

**uv** is a fast Python package manager (like pip but 10-100x faster):

```bash
# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# Verify installation
uv --version
```

> 💡 **Why uv?** The dlt MCP server uses `uv run` to manage dependencies automatically without polluting your global Python environment.

### Step 2: Create Your Project Folder

```bash
# Create and enter the project folder
mkdir taxi-pipeline
cd taxi-pipeline
```

### Step 3: Set Up the dlt MCP Server

The MCP (Model Context Protocol) server is what gives your AI assistant knowledge about dlt. Choose your IDE:

#### For VS Code + GitHub Copilot

Create a file at `.vscode/mcp.json` in your project:

```json
{
  "servers": {
    "dlt": {
      "command": "uv",
      "args": [
        "run",
        "--with",
        "dlt[duckdb]",
        "--with",
        "dlt-mcp[search]",
        "python",
        "-m",
        "dlt_mcp"
      ]
    }
  }
}
```

#### For Cursor

Go to **Settings → Tools & MCP → New MCP Server** and add:

```json
{
  "mcpServers": {
    "dlt": {
      "command": "uv",
      "args": [
        "run",
        "--with",
        "dlt[duckdb]",
        "--with",
        "dlt-mcp[search]",
        "python",
        "-m",
        "dlt_mcp"
      ]
    }
  }
}
```

#### For Claude Code

```bash
claude mcp add dlt -- uv run --with "dlt[duckdb]" --with "dlt-mcp[search]" python -m dlt_mcp
```

### Step 4: Install dlt

```bash
pip install "dlt[workspace]"
```

**What this installs:**
- `dlt` — The core data load tool library
- `workspace` extras — Scaffolding tools, dashboard, and project initialization

Verify it worked:

```bash
python -c "import dlt; print(dlt.__version__)"
```

### Step 5: Initialize the Project

```bash
dlt init dlthub:taxi_pipeline duckdb
```

**What this command does:**

| Part | Meaning |
|------|---------|
| `dlt init` | Initialize a new dlt project |
| `dlthub:taxi_pipeline` | Use the taxi_pipeline template from dlt workspace |
| `duckdb` | Set DuckDB as the destination |

**What it creates:**

```
taxi-pipeline/
├── .dlt/
│   ├── config.toml         # Pipeline configuration
│   └── secrets.toml        # API keys, credentials (none needed here)
├── taxi_pipeline.py        # Main pipeline file (YOU edit this)
├── requirements.txt        # Python dependencies
└── .cursorrules/           # AI assistant rules (if using Cursor)
```

> ⚠️ **Important:** Since the NYC taxi API has no scaffold, there's **no YAML file with API metadata**. You must provide the API details yourself in your prompt to the AI assistant.

### Understanding the Project Files

#### `.dlt/config.toml` — Pipeline Settings

```toml
[runtime]
log_level = "WARNING"

[normalize]
loader_file_format = "parquet"
```

This file configures how dlt behaves. For beginners, the defaults are fine.

#### `.dlt/secrets.toml` — Credentials

```toml
# No secrets needed for this workshop!
# The NYC taxi API is public and doesn't require authentication.
```

For APIs that need API keys, you'd add them here:

```toml
[sources.my_api]
api_key = "your-secret-key-here"
```

### Common Setup Issues & Solutions

| Problem | Cause | Solution |
|---------|-------|----------|
| `command not found: dlt` | dlt not in PATH | Run `pip install "dlt[workspace]"` again |
| `ModuleNotFoundError: dlt` | Wrong Python environment | Activate your venv: `source venv/bin/activate` |
| `uv: command not found` | uv not installed | Install: `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| MCP server not connecting | Config file in wrong location | Check file path matches your IDE |
| `dlt init` fails | Network issue or wrong template | Check internet connection; try `pip install --upgrade dlt` |

---

## Part 3: Building REST API Pipelines with dlt 🔧

### Understanding REST APIs (Quick Refresher)

A **REST API** is like a waiter in a restaurant:

```
You (Client)              Waiter (API)              Kitchen (Server)
┌──────────┐             ┌──────────┐              ┌──────────┐
│ "I'd like│  ─Request─► │ Takes    │  ─Fetches──► │ Prepares │
│  some    │             │ your     │              │ data     │
│  data"   │  ◄Response─ │ order    │  ◄Returns──  │          │
└──────────┘             └──────────┘              └──────────┘
```

**Key concepts:**
- **Endpoint** = The URL path (like a menu item)
- **Request** = What you ask for (GET, POST, etc.)
- **Response** = What you get back (usually JSON)
- **Pagination** = When there's too much data to send at once

### What is Pagination?

When an API has lots of data, it sends it in **pages** — just like a book:

```
Page 1 (records 1-1000)     Page 2 (records 1001-2000)    Page 3 (empty = STOP!)
┌─────────────────────┐    ┌─────────────────────┐       ┌─────────────────────┐
│ Record 1            │    │ Record 1001         │       │                     │
│ Record 2            │    │ Record 1002         │       │   (no records)      │
│ ...                 │    │ ...                 │       │                     │
│ Record 1000         │    │ Record 2000         │       │   ← Stop here!     │
└─────────────────────┘    └─────────────────────┘       └─────────────────────┘
        ▼                          ▼                              ▼
   ?page=1                    ?page=2                        ?page=3
```

For our NYC taxi API:
- Each page returns **1,000 records**
- You request pages by adding `?page=1`, `?page=2`, etc.
- When you get an **empty page** (no records), you stop

### The NYC Taxi API Details

| Property | Value |
|----------|-------|
| **Base URL** | `https://us-central1-dlthub-analytics.cloudfunctions.net/data_engineering_zoomcamp_api` |
| **Format** | JSON |
| **Page Size** | 1,000 records per page |
| **Pagination** | Page number based (`?page=1`, `?page=2`, ...) |
| **Stop Condition** | Empty page returned |
| **Authentication** | None required |

### Building the Pipeline: Step by Step

#### Step 1: Import dlt and the REST API Source

```python
import dlt
from dlt.sources.rest_api import rest_api_source
```

**What these do:**
- `dlt` — The main library
- `rest_api_source` — A built-in helper that handles REST API extraction, including pagination, automatically

#### Step 2: Define the Source

```python
def taxi_source():
    source = rest_api_source(
        {
            "client": {
                "base_url": "https://us-central1-dlthub-analytics.cloudfunctions.net",
            },
            "resources": [
                {
                    "name": "data_engineering_zoomcamp_api",
                    "endpoint": {
                        "path": "data_engineering_zoomcamp_api",
                        "paginator": {
                            "type": "page_number",
                            "base_page": 1,
                            "total_path": None,
                            "page_param": "page",
                        },
                    },
                },
            ],
        }
    )
    return source
```

**Breaking down each part:**

| Part | What It Does |
|------|--------------|
| `"client"` | Global settings for all API calls |
| `"base_url"` | The root URL of the API |
| `"resources"` | List of endpoints to extract data from |
| `"name"` | What to call the resulting table |
| `"endpoint"` | Specific endpoint configuration |
| `"path"` | URL path appended to base_url |
| `"paginator"` | How to handle pagination |
| `"type": "page_number"` | Use page numbers (?page=1, ?page=2) |
| `"base_page": 1` | Start from page 1 |
| `"total_path": None` | API doesn't tell us total pages |
| `"page_param": "page"` | URL parameter name for the page number |

#### Step 3: Create and Run the Pipeline

```python
if __name__ == "__main__":
    pipeline = dlt.pipeline(
        pipeline_name="taxi_pipeline",
        destination="duckdb",
        dataset_name="taxi_pipeline_dataset",
    )

    load_info = pipeline.run(taxi_source())
    print(load_info)
```

**What happens when you run this:**

```
Step 1: EXTRACT
├── Request page 1 → Get 1,000 records
├── Request page 2 → Get more records (if any)
├── Request page 3 → Empty page → STOP
│
Step 2: NORMALIZE
├── Flatten nested JSON
├── Infer column types (string, int, datetime, float)
├── Create target schema
│
Step 3: LOAD
├── Create table in DuckDB (if not exists)
├── Insert all records
└── Report: "Loaded X rows into table Y"
```

### The Complete Pipeline File

Here's the full `taxi_pipeline.py` ready to run:

```python
"""
NYC Yellow Taxi Data Pipeline
Loads trip data from a custom paginated API into DuckDB using dlt.
"""

import dlt
from dlt.sources.rest_api import rest_api_source


def taxi_source():
    """
    REST API source for NYC Yellow Taxi trip data.
    Handles pagination automatically — stops when empty page returned.
    """
    source = rest_api_source(
        {
            "client": {
                "base_url": "https://us-central1-dlthub-analytics.cloudfunctions.net",
            },
            "resources": [
                {
                    "name": "data_engineering_zoomcamp_api",
                    "endpoint": {
                        "path": "data_engineering_zoomcamp_api",
                        "paginator": {
                            "type": "page_number",
                            "base_page": 1,
                            "total_path": None,
                            "page_param": "page",
                        },
                    },
                },
            ],
        }
    )
    return source


if __name__ == "__main__":
    # Create the pipeline
    pipeline = dlt.pipeline(
        pipeline_name="taxi_pipeline",
        destination="duckdb",
        dataset_name="taxi_pipeline_dataset",
    )

    # Run it!
    load_info = pipeline.run(taxi_source())
    print(load_info)
```

### Understanding dlt REST API Paginator Types

dlt supports several pagination strategies:

| Paginator Type | How It Works | When to Use |
|----------------|-------------|-------------|
| `page_number` | Increments `?page=1`, `?page=2`, ... | Our taxi API! |
| `offset` | Uses `?offset=0`, `?offset=1000`, ... | APIs with offset pagination |
| `cursor` | Uses a token from response for next page | APIs with cursor-based pagination |
| `header_link` | Follows `Link` header URLs | GitHub API, etc. |
| `json_link` | Follows a `next` URL in the JSON body | Many modern APIs |

### Running the Pipeline

```bash
python taxi_pipeline.py
```

**Expected output:**

```
Pipeline taxi_pipeline load step completed in 3.45 seconds
1 load package(s) were loaded to destination duckdb and target dataset taxi_pipeline_dataset
The duckdb destination used duckdb:///path/to/taxi_pipeline.duckdb location to store data
Load package 1234567890.123456 is LOADED and target dataset taxi_pipeline_dataset
```

### Debugging Common Pipeline Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `ConnectionError` | Can't reach the API | Check your internet connection |
| `JSONDecodeError` | API returning unexpected format | Check the endpoint URL is correct |
| `SchemaCorruptedException` | Schema mismatch vs previous run | Delete `.dlt/` folder and rerun |
| `PipelineStepFailed` | Error during extract/normalize/load | Read the full error — usually has specifics |
| Infinite loop | Paginator never gets empty page | Verify `total_path: None` and stop condition |

> 💡 **Pro tip:** If the AI agent gets stuck, paste the full error message into the chat. The dlt MCP server helps the AI understand dlt-specific errors!

---

## Part 4: Loading Data into DuckDB 📦

### How dlt Loads Data

When your pipeline runs, dlt performs three stages internally:

```
┌─────────────────────────────────────────────────────────────────┐
│                     dlt Pipeline Stages                         │
│                                                                  │
│  Stage 1: EXTRACT                                               │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  • Connect to API                                        │   │
│  │  • Fetch pages: page 1 → page 2 → ... → empty → STOP   │   │
│  │  • Store raw JSON responses locally                      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                           ▼                                     │
│  Stage 2: NORMALIZE                                             │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  • Flatten nested JSON into tabular format              │   │
│  │  • Infer data types (string, int, float, datetime)      │   │
│  │  • Handle nested objects → create child tables          │   │
│  │  • Generate schema                                       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                           ▼                                     │
│  Stage 3: LOAD                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  • Create tables in DuckDB (if not exists)              │   │
│  │  • Insert normalized data                                │   │
│  │  • Update pipeline state (metadata)                      │   │
│  │  • Report success/failure                                │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### What dlt Creates in DuckDB

After running the pipeline, dlt creates several tables:

```
taxi_pipeline_dataset/                     ← Your dataset
├── _data_engineering_zoomcamp_api         ← YOUR DATA (the taxi trips!)
├── _dlt_loads                             ← Metadata: when loads happened
├── _dlt_pipeline_state                    ← Metadata: pipeline state
└── _dlt_version                           ← Metadata: dlt version info
```

| Table | What It Contains |
|-------|-----------------|
| `_data_engineering_zoomcamp_api` | The actual NYC taxi trip records |
| `_dlt_loads` | Load history (when, how many rows, status) |
| `_dlt_pipeline_state` | Pipeline state for incremental loading |
| `_dlt_version` | Which version of dlt created this |

> 🎓 **Beginner tip:** Tables starting with `_dlt_` are metadata tables created by dlt. Your actual data is in `_data_engineering_zoomcamp_api`.

### Schema Inference: How dlt Figures Out Column Types

One of dlt's best features is **automatic schema inference**. It looks at your JSON data and figures out the column types:

```
JSON Input:                          DuckDB Table:
{                                    ┌─────────────────────────────────┐
  "vendor_name": "VTS",              │ vendor_name     │ VARCHAR       │
  "trip_pickup_date_time":           │ trip_pickup_...  │ TIMESTAMP     │
    "2009-06-15 17:26:21",           │ trip_dropoff_... │ TIMESTAMP     │
  "passenger_count": 2,              │ passenger_count  │ BIGINT        │
  "trip_distance": 1.37,             │ trip_distance    │ DOUBLE        │
  "payment_type": "Credit",          │ payment_type     │ VARCHAR       │
  "tip_amt": 2.00,                   │ tip_amt          │ DOUBLE        │
  "total_amt": 11.75                 │ total_amt        │ DOUBLE        │
}                                    └─────────────────────────────────┘
```

### Connecting to DuckDB After Loading

You can query your data directly using Python:

```python
import duckdb

# Connect to the DuckDB file created by dlt
conn = duckdb.connect("taxi_pipeline.duckdb")

# See all tables
print(conn.sql("SHOW TABLES").fetchall())

# Query your data
result = conn.sql("""
    SELECT COUNT(*) as total_trips 
    FROM taxi_pipeline_dataset._data_engineering_zoomcamp_api
""")
print(result.fetchone())
```

### Important DuckDB Query Tips

When querying data loaded by dlt into DuckDB:

```sql
-- ✅ Correct: Use dataset_name.table_name
SELECT * FROM taxi_pipeline_dataset._data_engineering_zoomcamp_api LIMIT 5;

-- ❌ Wrong: Missing dataset name
SELECT * FROM _data_engineering_zoomcamp_api;  -- May fail!

-- 💡 Set the schema first to avoid typing it every time
SET SCHEMA 'taxi_pipeline_dataset';
SELECT * FROM _data_engineering_zoomcamp_api LIMIT 5;
```

### Understanding the Loaded Data

The NYC taxi dataset typically contains these columns:

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| `vendor_name` | VARCHAR | Taxi company | "VTS", "CMT" |
| `trip_pickup_date_time` | TIMESTAMP | When trip started | 2009-06-15 17:26:21 |
| `trip_dropoff_date_time` | TIMESTAMP | When trip ended | 2009-06-15 17:34:56 |
| `passenger_count` | BIGINT | Number of passengers | 2 |
| `trip_distance` | DOUBLE | Miles traveled | 1.37 |
| `start_lon` / `start_lat` | DOUBLE | Pickup coordinates | -73.99, 40.73 |
| `end_lon` / `end_lat` | DOUBLE | Dropoff coordinates | -73.98, 40.75 |
| `rate_code` | VARCHAR | Rate type | "Standard" |
| `payment_type` | VARCHAR | How they paid | "Credit", "Cash" |
| `fare_amt` | DOUBLE | Metered fare | 6.50 |
| `surcharge` | DOUBLE | Extra charges | 0.50 |
| `tip_amt` | DOUBLE | Tip amount | 2.00 |
| `tolls_amt` | DOUBLE | Toll fees | 0.00 |
| `total_amt` | DOUBLE | Total charged | 11.75 |

### Load Modes in dlt

dlt supports different ways to load data:

| Mode | Behavior | Use When |
|------|----------|----------|
| `replace` (default) | Drop and recreate table | First run, full refresh |
| `append` | Add new rows to existing table | Incremental loads |
| `merge` | Update existing + insert new | Deduplication needed |

```python
# Replace mode (default) — recreate table each time
pipeline.run(source, write_disposition="replace")

# Append mode — add new rows
pipeline.run(source, write_disposition="append")

# Merge mode — update + insert (needs primary key)
pipeline.run(source, write_disposition="merge")
```

---

## Part 5: Exploring Data with dlt Tools 🔍

### Three Ways to Explore Your Pipeline Data

After your pipeline runs successfully, dlt offers three powerful ways to investigate your data:

```
┌─────────────────────────────────────────────────────────────────┐
│              Three Ways to Explore Your Data                     │
│                                                                  │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐          │
│  │    dlt      │   │    dlt      │   │   marimo    │          │
│  │  Dashboard  │   │  MCP Server │   │  Notebook   │          │
│  │             │   │             │   │             │          │
│  │  Web UI     │   │  Chat with  │   │ Interactive │          │
│  │  Point &    │   │  your data  │   │ Python      │          │
│  │  Click      │   │  via AI     │   │ Notebook    │          │
│  └─────────────┘   └─────────────┘   └─────────────┘          │
│                                                                  │
│  Best for:         Best for:         Best for:                  │
│  Quick overview    Ad-hoc questions  Custom analysis            │
│  Schema viewing    Natural language  Visualizations             │
│  Load history      Metadata queries  Shareable reports          │
└─────────────────────────────────────────────────────────────────┘
```

### Method 1: dlt Dashboard

The **dlt Dashboard** is a web-based UI for inspecting your pipeline:

```bash
dlt pipeline taxi_pipeline show
```

This opens a browser with:

```
┌─────────────────────────────────────────────────────────┐
│  dlt Dashboard — taxi_pipeline                          │
│─────────────────────────────────────────────────────────│
│                                                         │
│  📊 Overview                                           │
│  ├── Pipeline: taxi_pipeline                           │
│  ├── Destination: duckdb                               │
│  ├── Dataset: taxi_pipeline_dataset                    │
│  └── Last load: 2 minutes ago                          │
│                                                         │
│  📋 Tables                                             │
│  ├── _data_engineering_zoomcamp_api (1,000 rows)      │
│  ├── _dlt_loads                                        │
│  └── _dlt_pipeline_state                               │
│                                                         │
│  🔍 Query Editor                                       │
│  ┌─────────────────────────────────────────────────┐  │
│  │ SELECT * FROM _data_engineering_zoomcamp_api    │  │
│  │ LIMIT 10                                        │  │
│  └─────────────────────────────────────────────────┘  │
│  [Run Query]                                           │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**What you can do in the dashboard:**
- View all tables and their schemas
- Browse sample data
- Run SQL queries directly
- Check load history and metadata
- See column types and row counts

### Method 2: dlt MCP Server (Chat with Your Data)

With the MCP server configured, you can ask your AI assistant questions about the pipeline:

```
You: "How many rows are in the taxi dataset?"
AI: "The _data_engineering_zoomcamp_api table has 1,000 rows."

You: "What columns does the table have?"
AI: "The table has 18 columns: vendor_name, trip_pickup_date_time, ..."

You: "What's the average trip distance?"
AI: "The average trip distance is 2.83 miles."
```

The MCP server gives the AI access to:
- Pipeline metadata (tables, schemas, row counts)
- dlt documentation
- Code examples for common tasks

### Method 3: marimo Notebooks

**marimo** is a modern Python notebook (alternative to Jupyter) that works great with dlt:

```bash
# Install marimo
pip install marimo

# Create a new notebook
marimo edit taxi_analysis.py
```

Example marimo notebook code:

```python
import marimo as mo
import duckdb

# Connect to your pipeline's DuckDB
conn = duckdb.connect("taxi_pipeline.duckdb")

# Query and display
df = conn.sql("""
    SELECT payment_type, COUNT(*) as trips, AVG(total_amt) as avg_total
    FROM taxi_pipeline_dataset._data_engineering_zoomcamp_api
    GROUP BY payment_type
""").df()

mo.ui.table(df)
```

### Other Useful dlt Commands

| Command | What It Does |
|---------|--------------|
| `dlt pipeline taxi_pipeline show` | Open the dashboard |
| `dlt pipeline taxi_pipeline info` | Show pipeline status |
| `dlt pipeline taxi_pipeline schema` | Display the data schema |
| `dlt pipeline taxi_pipeline trace` | Show last run trace |
| `dlt pipeline taxi_pipeline drop` | Delete pipeline state |
| `dlt pipeline list` | List all pipelines |

### Querying the Data (SQL Examples)

Here are useful queries for exploring the loaded taxi data:

#### Basic Data Exploration

```sql
-- How many trips total?
SELECT COUNT(*) AS total_trips
FROM taxi_pipeline_dataset._data_engineering_zoomcamp_api;

-- Preview the first 5 rows
SELECT * 
FROM taxi_pipeline_dataset._data_engineering_zoomcamp_api
LIMIT 5;

-- What columns do we have?
DESCRIBE taxi_pipeline_dataset._data_engineering_zoomcamp_api;
```

#### Date Range Analysis

```sql
-- What's the date range of the dataset?
SELECT 
    MIN(trip_pickup_date_time) AS earliest_trip,
    MAX(trip_pickup_date_time) AS latest_trip
FROM taxi_pipeline_dataset._data_engineering_zoomcamp_api;
```

#### Payment Analysis

```sql
-- Payment type breakdown
SELECT 
    payment_type,
    COUNT(*) AS trip_count,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 2) AS percentage
FROM taxi_pipeline_dataset._data_engineering_zoomcamp_api
GROUP BY payment_type
ORDER BY trip_count DESC;
```

#### Revenue Analysis

```sql
-- Total revenue and tips
SELECT 
    SUM(total_amt) AS total_revenue,
    SUM(tip_amt) AS total_tips,
    AVG(total_amt) AS avg_fare,
    AVG(tip_amt) AS avg_tip
FROM taxi_pipeline_dataset._data_engineering_zoomcamp_api;
```

---

## Part 6: Data Investigation and Best Practices 📊

### Answering Questions from Your Pipeline Data

The final step in any data pipeline is **using the data** to answer questions. This is where all the setup pays off. Let's walk through the analytical process:

```
┌─────────────────────────────────────────────────────────────────┐
│              Data Investigation Workflow                         │
│                                                                  │
│  1. UNDERSTAND    → What question are we answering?             │
│  2. IDENTIFY      → Which columns/tables have the answer?      │
│  3. QUERY         → Write SQL to extract the answer             │
│  4. VALIDATE      → Does the result make sense?                │
│  5. INTERPRET     → What does this mean in context?            │
└─────────────────────────────────────────────────────────────────┘
```

### Investigation 1: Dataset Date Range

**Question:** What period does our data cover?

```sql
SELECT 
    MIN(trip_pickup_date_time) AS start_date,
    MAX(trip_pickup_date_time) AS end_date,
    DATEDIFF('day', 
        MIN(trip_pickup_date_time), 
        MAX(trip_pickup_date_time)
    ) AS days_covered
FROM taxi_pipeline_dataset._data_engineering_zoomcamp_api;
```

**Result:** The data covers **June 1, 2009 to July 1, 2009** — exactly one month of NYC taxi trips.

**Context:** This is historical data from summer 2009. NYC had approximately 13,000 licensed yellow taxis at that time, generating millions of trips per month. Our dataset is a sample of 1,000 trips from this period.

### Investigation 2: Payment Method Distribution

**Question:** How do riders pay for their trips?

```sql
SELECT 
    payment_type,
    COUNT(*) AS trips,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM taxi_pipeline_dataset._data_engineering_zoomcamp_api), 2) AS pct
FROM taxi_pipeline_dataset._data_engineering_zoomcamp_api
GROUP BY payment_type
ORDER BY trips DESC;
```

**Result:** Approximately **26.66%** of trips were paid with credit card.

**Context:** In 2009, credit card usage in NYC taxis was still relatively low. The Taxi & Limousine Commission (TLC) had only recently mandated credit card readers in all cabs (2007). By 2024, credit card usage has grown to over 70% of all taxi trips!

### Investigation 3: Revenue Analysis

**Question:** How much total revenue was generated?

```sql
SELECT 
    COUNT(*) AS total_trips,
    ROUND(SUM(total_amt), 2) AS total_revenue,
    ROUND(AVG(total_amt), 2) AS avg_trip_cost,
    ROUND(SUM(tip_amt), 2) AS total_tips,
    ROUND(AVG(tip_amt), 2) AS avg_tip
FROM taxi_pipeline_dataset._data_engineering_zoomcamp_api;
```

**Result:** Total amount is approximately **$10,063.41** across all trips.

### Best Practices for dlt Pipelines

#### 1. Always Name Your Pipeline

```python
# ✅ Good: Descriptive name
pipeline = dlt.pipeline(
    pipeline_name="taxi_pipeline",       # Clear, specific
    destination="duckdb",
    dataset_name="taxi_pipeline_dataset"  # Separate from pipeline name
)

# ❌ Bad: Default or vague name
pipeline = dlt.pipeline(destination="duckdb")
```

#### 2. Handle Pagination Correctly

```python
# ✅ Good: Explicit pagination config
"paginator": {
    "type": "page_number",
    "base_page": 1,
    "total_path": None,      # API doesn't tell us total pages
    "page_param": "page",    # URL parameter name
}

# ❌ Bad: No pagination = only get first page
"paginator": None
```

#### 3. Use the Right Write Disposition

```python
# First run or full refresh → replace
pipeline.run(source, write_disposition="replace")

# Subsequent runs with new data → append
pipeline.run(source, write_disposition="append")
```

#### 4. Check Your Data After Loading

Always verify your pipeline worked:

```python
# Quick verification after loading
import duckdb
conn = duckdb.connect("taxi_pipeline.duckdb")

row_count = conn.sql("""
    SELECT COUNT(*) FROM taxi_pipeline_dataset._data_engineering_zoomcamp_api
""").fetchone()[0]

print(f"Loaded {row_count} rows")
assert row_count > 0, "No data loaded!"
```

#### 5. Use the dlt Dashboard for Debugging

```bash
# Always check the dashboard after a run
dlt pipeline taxi_pipeline show
```

Look for:
- Row counts (did all data load?)
- Schema changes (unexpected new columns?)
- Load status (any failures?)

### The Complete dlt Workflow Summary

```
┌─────────────────────────────────────────────────────────────────┐
│              Complete dlt Workflow                               │
│                                                                  │
│  ① SETUP                                                        │
│     pip install "dlt[workspace]"                                │
│     dlt init dlthub:taxi_pipeline duckdb                        │
│     Configure MCP server                                        │
│                                                                  │
│  ② BUILD                                                        │
│     Define source (REST API with pagination)                    │
│     Create pipeline (name, destination, dataset)                │
│     Prompt AI agent for help                                    │
│                                                                  │
│  ③ RUN                                                          │
│     python taxi_pipeline.py                                     │
│     Check output for success                                    │
│                                                                  │
│  ④ EXPLORE                                                      │
│     dlt pipeline taxi_pipeline show  (Dashboard)                │
│     Ask AI via MCP server           (Chat)                      │
│     marimo edit notebook.py          (Notebook)                 │
│                                                                  │
│  ⑤ ANALYZE                                                      │
│     Write SQL queries                                           │
│     Answer business questions                                   │
│     Build visualizations                                        │
│                                                                  │
│  ⑥ ITERATE                                                      │
│     Add more endpoints                                          │
│     Set up incremental loading                                  │
│     Schedule regular runs                                       │
└─────────────────────────────────────────────────────────────────┘
```

### Key Takeaways

| Concept | What You Learned |
|---------|-----------------|
| **dlt** | An open-source Python library for data loading |
| **REST API Source** | Built-in dlt helper for extracting from APIs |
| **Pagination** | How APIs send data in pages, and how dlt handles it |
| **DuckDB** | A local analytical database — no server needed |
| **Pipeline** | Source → Normalize → Load workflow |
| **MCP Server** | Gives AI assistants knowledge about dlt |
| **Dashboard** | Web UI for inspecting loaded data |
| **marimo** | Modern Python notebook for data analysis |
| **AI-Assisted Dev** | Using AI to generate and debug pipeline code |

### Resources

| Resource | Link |
|----------|------|
| dlt Documentation | [dlthub.com/docs](https://dlthub.com/docs) |
| dlt Dashboard Docs | [dlthub.com/docs/general-usage/dashboard](https://dlthub.com/docs/general-usage/dashboard) |
| marimo + dlt Guide | [dlthub.com/docs/general-usage/dataset-access/marimo](https://dlthub.com/docs/general-usage/dataset-access/marimo) |
| dlt REST API Source | [dlthub.com/docs/dlt-ecosystem/verified-sources/rest_api](https://dlthub.com/docs/dlt-ecosystem/verified-sources/rest_api) |
| Workshop README | [GitHub](https://github.com/DataTalksClub/data-engineering-zoomcamp/blob/main/cohorts/2026/workshops/dlt/README.md) |
| DuckDB Documentation | [duckdb.org/docs](https://duckdb.org/docs/) |

---

*Workshop by [dltHub](https://dlthub.com) for the Data Engineering Zoomcamp 2026*
