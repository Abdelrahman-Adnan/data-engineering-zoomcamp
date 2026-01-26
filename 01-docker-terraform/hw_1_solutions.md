# Module 1 Homework Solutions

## Question 1: Understanding Docker images

**Question:** What's the version of `pip` in the `python:3.13` image?

**Answer:** 25.3

To verify this, I ran:
```bash
docker run --rm python:3.13 pip --version
```

Output: `pip 25.3 from /usr/local/lib/python3.13/site-packages/pip (python 3.13)`

---

## Question 2: Understanding Docker networking and docker-compose

**Question:** Given the docker-compose.yaml configuration, what hostname and port should pgadmin use to connect to the postgres database?

**Answer:** db:5432

In Docker Compose, containers communicate using service names as hostnames. The postgres service is named "db" in the configuration, and internally it listens on port 5432 (the standard PostgreSQL port). The port mapping `5433:5432` only affects external access from the host machine. From inside the Docker network, pgadmin connects to `db:5432`.

---

## Question 3: Counting short trips

**Question:** For trips in November 2025, how many trips had a `trip_distance` of less than or equal to 1 mile?

**Answer:** 8,007

SQL query used:
```sql
SELECT COUNT(*) FROM green_taxi_data t
WHERE DATE(t.lpep_pickup_datetime) >= '2025-11-01' 
AND DATE(t.lpep_pickup_datetime) < '2025-12-01'
AND t.trip_distance <= 1.0;
```

---

## Question 4: Longest trip for each day

**Question:** Which pickup day had the longest trip distance (excluding trips >= 100 miles)?

**Answer:** 2025-11-14

The longest trip was 88.03 miles on November 14th, 2025.

SQL query:
```sql
SELECT DATE(t.lpep_pickup_datetime), t.trip_distance 
FROM green_taxi_data t
WHERE t.trip_distance < 100.0
ORDER BY t.trip_distance DESC
LIMIT 1;
```

---

## Question 5: Biggest pickup zone

**Question:** Which pickup zone had the largest `total_amount` sum on November 18th, 2025?

**Answer:** East Harlem North

Total amount sum: $9,281.92

SQL query:
```sql
SELECT z."Zone" as zone, SUM(t.total_amount) as total_amount_sum
FROM green_taxi_data t
JOIN taxi_zone_lookup z ON t."PULocationID" = z."LocationID"
WHERE DATE(t.lpep_pickup_datetime) = '2025-11-18'
GROUP BY zone
ORDER BY total_amount_sum DESC
LIMIT 1;
```

---

## Question 6: Largest tip

**Question:** For passengers picked up in "East Harlem North" during November 2025, which dropoff zone had the largest tip?

**Answer:** Yorkville West

Largest tip amount: $81.89

SQL query:
```sql
SELECT zdo."Zone" AS dropoff_zone, t.tip_amount
FROM green_taxi_data t
JOIN taxi_zone_lookup zpu ON t."PULocationID" = zpu."LocationID"
JOIN taxi_zone_lookup zdo ON t."DOLocationID" = zdo."LocationID"
WHERE zpu."Zone" = 'East Harlem North'
  AND DATE(t.lpep_pickup_datetime) >= '2025-11-01' 
  AND DATE(t.lpep_pickup_datetime) <= '2025-11-30'
ORDER BY t.tip_amount DESC
LIMIT 1;
```

---

## Question 7: Terraform Workflow

**Question:** Which sequence describes the workflow for:
1. Downloading provider plugins and setting up backend
2. Generating proposed changes and auto-executing the plan
3. Removing all resources managed by terraform

**Answer:** terraform init, terraform apply -auto-approve, terraform destroy

Explanation:
- `terraform init` - Initializes the working directory and downloads provider plugins
- `terraform apply -auto-approve` - Creates/updates resources without manual approval
- `terraform destroy` - Removes all managed infrastructure resources
