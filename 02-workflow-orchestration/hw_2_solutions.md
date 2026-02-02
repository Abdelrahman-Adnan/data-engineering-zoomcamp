# Module 2 Homework Solutions - Workflow Orchestration

## Overview

This homework extends the existing Kestra flows to include NYC taxi data for the year 2021.

**Dataset Source:**
- Green taxi data: `https://github.com/DataTalksClub/nyc-tlc-data/releases/tag/green/download`
- wget-able prefix: `https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/`

---

## Assignment Task

We needed to extend the existing flows to include data for the year 2021. Two approaches were available:

1. **Backfill Method**: Use Kestra's backfill functionality in the scheduled flow for the period `2021-01-01` to `2021-07-31` for both `yellow` and `green` taxi data.

2. **Manual/Loop Method**: Run the flow manually for each month, or use `ForEach` task with `Subflow` to loop over Year-Month and taxi-type combinations.

---

## Quiz Questions & Answers

### Question 1
**Within the execution for `Yellow` Taxi data for the year `2020` and month `12`: what is the uncompressed file size (i.e. the output file `yellow_tripdata_2020-12.csv` of the `extract` task)?**

- 128.3 MiB
- 134.5 MiB
- 364.7 MiB
- 692.6 MiB

**Answer:** `128.3 MiB`

---

### Question 2
**What is the rendered value of the variable `file` when the inputs `taxi` is set to `green`, `year` is set to `2020`, and `month` is set to `04` during execution?**

- `{{inputs.taxi}}_tripdata_{{inputs.year}}-{{inputs.month}}.csv`
- `green_tripdata_2020-04.csv`
- `green_tripdata_04_2020.csv`
- `green_tripdata_2020.csv`

**Answer:** `green_tripdata_2020-04.csv`

---

### Question 3
**How many rows are there for the `Yellow` Taxi data for all CSV files in the year 2020?**

- 13,537,299
- 24,648,499
- 18,324,219
- 29,430,127

**Answer:** `24,648,499`

---

### Question 4
**How many rows are there for the `Green` Taxi data for all CSV files in the year 2020?**

- 5,327,301
- 936,199
- 1,734,051
- 1,342,034

**Answer:** `1,734,051`

---

### Question 5
**How many rows are there for the `Yellow` Taxi data for the March 2021 CSV file?**

- 1,428,092
- 706,911
- 1,925,152
- 2,561,031

**Answer:** `1,925,152`

---

### Question 6
**How would you configure the timezone to New York in a Schedule trigger?**

- Add a `timezone` property set to `EST` in the `Schedule` trigger configuration
- Add a `timezone` property set to `America/New_York` in the `Schedule` trigger configuration
- Add a `timezone` property set to `UTC-5` in the `Schedule` trigger configuration
- Add a `location` property set to `New_York` in the `Schedule` trigger configuration

**Answer:** Add a `timezone` property set to `America/New_York` in the `Schedule` trigger configuration

---

## Submission

Form for submitting: https://courses.datatalks.club/de-zoomcamp-2026/homework/hw2
