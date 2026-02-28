# Module 5 Homework Solutions - Data Platforms with Bruin

## Overview

This homework focuses on using Bruin to build a complete data pipeline from ingestion to reporting, covering pipeline structure, materialization strategies, variables, dependencies, quality checks, and lineage.

**Key Topics:**
- Bruin project structure and configuration
- Materialization strategies for incremental processing
- Pipeline variables and overrides
- Running assets with dependencies
- Data quality checks
- Lineage visualization

---

## Setup

### Prerequisites

1. Install Bruin CLI: `curl -LsSf https://getbruin.com/install/cli | sh`
2. Initialize the zoomcamp template: `bruin init zoomcamp my-pipeline`
3. Configure your `.bruin.yml` with a DuckDB connection
4. Follow the tutorial in the [main module README](https://github.com/DataTalksClub/data-engineering-zoomcamp/blob/main/05-data-platforms)

After completing the setup, you should have a working NYC taxi data pipeline.

---

## Quiz Questions & Answers

### Question 1. Bruin Pipeline Structure

**Question:** In a Bruin project, what are the required files/directories?

- `bruin.yml` and `assets/`
- `.bruin.yml` and `pipeline.yml` (assets can be anywhere)
- `.bruin.yml` and `pipeline/` with `pipeline.yml` and `assets/`
- `pipeline.yml` and `assets/` only

**Answer:** `.bruin.yml` and `pipeline.yml` (assets can be anywhere)

A Bruin project requires:
- `.bruin.yml` - The main configuration file at the project root that defines connections and environments
- `pipeline.yml` - Defines the pipeline configuration including variables and settings

Assets can be placed anywhere in the directory structure, but by convention they're typically in an `assets/` folder. The flexibility allows you to organize your assets in a way that makes sense for your project.

---

### Question 2. Materialization Strategies

**Question:** You're building a pipeline that processes NYC taxi data organized by month based on `pickup_datetime`. Which incremental strategy is best for processing a specific interval period by deleting and inserting data for that time period?

- `append` - always add new rows
- `replace` - truncate and rebuild entirely
- `time_interval` - incremental based on a time column
- `view` - create a virtual table only

**Answer:** `time_interval` - incremental based on a time column

The `time_interval` strategy is ideal for time-series data like taxi trips:
- It deletes existing data for a specific time period (e.g., a month)
- Then inserts fresh data for that same period
- This avoids duplicates while being efficient (no full table rebuild)

The other strategies:
- `append` would cause duplicates if you reprocess data
- `replace` would rebuild the entire table unnecessarily
- `view` doesn't persist data at all

---

### Question 3. Pipeline Variables

**Question:** You have the following variable defined in `pipeline.yml`:

```yaml
variables:
  taxi_types:
    type: array
    items:
      type: string
    default: ["yellow", "green"]
```

How do you override this when running the pipeline to only process yellow taxis?

- `bruin run --taxi-types yellow`
- `bruin run --var taxi_types=yellow`
- `bruin run --var 'taxi_types=["yellow"]'`
- `bruin run --set taxi_types=["yellow"]`

**Answer:** `bruin run --var 'taxi_types=["yellow"]'`

When overriding array variables in Bruin:
- Use the `--var` flag followed by the variable name and value
- Since `taxi_types` is an array type, you must pass it as a JSON array: `["yellow"]`
- Quote the entire value to prevent shell interpretation of special characters like brackets

Using `--var taxi_types=yellow` without the array format would fail because the variable expects an array, not a simple string.

---

### Question 4. Running with Dependencies

**Question:** You've modified the `ingestion/trips.py` asset and want to run it plus all downstream assets. Which command should you use?

- `bruin run ingestion.trips --all`
- `bruin run ingestion/trips.py --downstream`
- `bruin run pipeline/trips.py --recursive`
- `bruin run --select ingestion.trips+`

**Answer:** `bruin run ingestion/trips.py --downstream`

In Bruin, the `--downstream` flag is used to run an asset along with all its downstream dependencies. This is useful when:
- You've modified a source asset and want to propagate changes
- You need to rebuild all dependent models after updating upstream data

Note: Unlike dbt which uses the `+` suffix syntax (`model+`), Bruin uses explicit `--downstream` and `--upstream` flags for clearer intent.

---

### Question 5. Quality Checks

**Question:** You want to ensure the `pickup_datetime` column in your trips table never has NULL values. Which quality check should you add to your asset definition?

- `name: unique`
- `name: not_null`
- `name: positive`
- `name: accepted_values, value: [not_null]`

**Answer:** `name: not_null`

The `not_null` quality check ensures a column never has NULL values. In your asset definition, you would add:

```yaml
columns:
  - name: pickup_datetime
    checks:
      - name: not_null
```

The other checks serve different purposes:
- `unique` - ensures no duplicate values
- `positive` - ensures numeric values are greater than 0
- `accepted_values` - validates that values are within a predefined set

---

### Question 6. Lineage and Dependencies

**Question:** After building your pipeline, you want to visualize the dependency graph between assets. Which Bruin command should you use?

- `bruin graph`
- `bruin dependencies`
- `bruin lineage`
- `bruin show`

**Answer:** `bruin lineage`

The `bruin lineage` command visualizes the dependency graph between assets in your pipeline. It shows:
- Which assets depend on other assets
- The flow of data through your pipeline
- Upstream and downstream relationships

This is useful for:
- Understanding pipeline structure
- Impact analysis before making changes
- Debugging data flow issues

---

### Question 7. First-Time Run

**Question:** You're running a Bruin pipeline for the first time on a new DuckDB database. What flag should you use to ensure tables are created from scratch?

- `--create`
- `--init`
- `--full-refresh`
- `--truncate`

**Answer:** `--full-refresh`

The `--full-refresh` flag tells Bruin to:
- Drop existing tables (if any)
- Recreate tables from scratch
- Ignore any incremental logic and process all data

This is essential for:
- First-time pipeline runs on a new database
- Rebuilding tables after schema changes
- Resetting data after testing

Example usage:
```bash
bruin run --full-refresh
```

---

## Summary

| Question | Answer |
|----------|--------|
| Q1. Pipeline Structure | `.bruin.yml` and `pipeline.yml` (assets can be anywhere) |
| Q2. Materialization Strategies | `time_interval` |
| Q3. Pipeline Variables | `bruin run --var 'taxi_types=["yellow"]'` |
| Q4. Running with Dependencies | `bruin run ingestion/trips.py --downstream` |
| Q5. Quality Checks | `name: not_null` |
| Q6. Lineage and Dependencies | `bruin lineage` |
| Q7. First-Time Run | `--full-refresh` |

---

## Submission

Form for submitting: https://courses.datatalks.club/de-zoomcamp-2026/homework/hw5
