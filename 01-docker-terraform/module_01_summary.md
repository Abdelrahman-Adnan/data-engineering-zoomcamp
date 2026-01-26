# Module 1: Docker, SQL & Terraform

This is my notes and walkthrough for Module 1 of the Data Engineering Zoomcamp. If you're new to data engineering, this should help you understand the basics.

## What is Data Engineering?

Data Engineering is basically about building systems that collect, store, and analyze data at scale. Think of it as the plumbing that makes data flow from point A to point B so analysts and data scientists can do their thing.

A **data pipeline** is just a service that takes data in, does something with it, and outputs more data. Simple example: read a CSV file, clean it up, store it in a database.

---

## Part 1: Docker Basics

### Why Docker?

Docker lets you package your application and all its dependencies into a "container". This solves the classic "it works on my machine" problem.

Main benefits:
- **Reproducibility** - same environment everywhere
- **Isolation** - apps run independently, won't mess with your system
- **Portability** - works on any machine with Docker installed

Containers are different from virtual machines - they're much lighter because they share the host OS kernel.

### Getting Started with Docker

First, check if Docker is installed:
```bash
docker --version
```

Run your first container:
```bash
docker run hello-world
```

Try running Ubuntu:
```bash
docker run -it ubuntu
```

The `-it` flag means interactive mode with a terminal. Without it, the container just starts and exits.

### Important: Containers are Stateless

This tripped me up at first. Any changes you make inside a container are **lost** when the container stops. For example:

```bash
docker run -it ubuntu
apt update && apt install python3
exit
# Run it again
docker run -it ubuntu
python3  # Error! Python is not installed
```

This is actually a feature, not a bug. It means you can always start fresh.

### Managing Containers

See all containers (including stopped ones):
```bash
docker ps -a
```

Clean up old containers:
```bash
docker rm $(docker ps -aq)
```

Better approach - use `--rm` to auto-delete when container stops:
```bash
docker run -it --rm ubuntu
```

### Using Different Base Images

You can use pre-built images with software already installed:

```bash
# Python image - starts Python interpreter
docker run -it --rm python:3.13

# If you want bash instead of Python:
docker run -it --rm --entrypoint=bash python:3.13-slim
```

### Volumes - Persisting Data

Since containers are stateless, we need volumes to save data. There are two types:

**Named volumes** (Docker manages them):
```bash
docker run -it -v my_data:/app/data ubuntu
```

**Bind mounts** (map to a folder on your computer):
```bash
docker run -it -v $(pwd)/my_folder:/app/data ubuntu
```

---

## Part 2: Creating a Dockerfile

A Dockerfile is a recipe for building your own Docker image.

### Simple Example

Create a file called `pipeline.py`:
```python
import sys
import pandas as pd

print(sys.argv)
day = sys.argv[1]
print(f'Job finished for day = {day}')
```

Create a `Dockerfile`:
```dockerfile
FROM python:3.13-slim

RUN pip install pandas pyarrow

WORKDIR /app
COPY pipeline.py pipeline.py

ENTRYPOINT ["python", "pipeline.py"]
```

What each line does:
- `FROM` - base image to build on
- `RUN` - execute commands during build
- `WORKDIR` - set the working directory
- `COPY` - copy files from your machine to the image
- `ENTRYPOINT` - the command that runs when container starts

Build and run:
```bash
docker build -t test:pandas .
docker run -it test:pandas some_argument
```

---

## Part 3: Running PostgreSQL with Docker

Now let's do some real data engineering. We'll run Postgres in a container.

```bash
docker run -it --rm \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:17
```

Breaking this down:
- `-e` sets environment variables (username, password, database name)
- `-v` creates a named volume so data persists
- `-p 5432:5432` maps the container port to your machine

### Connecting to Postgres

Install pgcli (a nice command-line client):
```bash
pip install pgcli
# or with uv:
uv add --dev pgcli
```

Connect:
```bash
pgcli -h localhost -p 5432 -u root -d ny_taxi
```

Try some SQL:
```sql
\dt                              -- list tables
CREATE TABLE test (id INTEGER);
INSERT INTO test VALUES (1);
SELECT * FROM test;
\q                               -- quit
```

---

## Part 4: Data Ingestion with Python

We're going to load the NYC Taxi dataset into Postgres.

### Setting Up

Install dependencies:
```bash
pip install pandas sqlalchemy psycopg2-binary jupyter
```

Or with uv:
```bash
uv add pandas sqlalchemy psycopg2-binary
uv add --dev jupyter
```

### The Dataset

We use the NYC Taxi trip data. Download it:
```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz
```

### Loading Data into Postgres

Here's the basic approach:

```python
import pandas as pd
from sqlalchemy import create_engine

# Create connection
engine = create_engine('postgresql://root:root@localhost:5432/ny_taxi')

# Read CSV in chunks (it's a big file)
df_iter = pd.read_csv('yellow_tripdata_2021-01.csv.gz', 
                       iterator=True, 
                       chunksize=100000)

# Create table from first chunk
first_chunk = next(df_iter)
first_chunk.head(0).to_sql(name='yellow_taxi_data', con=engine, if_exists='replace')

# Insert first chunk
first_chunk.to_sql(name='yellow_taxi_data', con=engine, if_exists='append')

# Insert remaining chunks
for chunk in df_iter:
    chunk.to_sql(name='yellow_taxi_data', con=engine, if_exists='append')
    print(f'Inserted {len(chunk)} rows')
```

The key things here:
- `chunksize` prevents loading the whole file into memory
- `if_exists='replace'` creates the table (first time)
- `if_exists='append'` adds rows (subsequent chunks)

---

## Part 5: Docker Compose

Running multiple `docker run` commands is annoying. Docker Compose lets you define everything in one file.

Create `docker-compose.yaml`:
```yaml
services:
  pgdatabase:
    image: postgres:17
    environment:
      POSTGRES_USER: "root"
      POSTGRES_PASSWORD: "root"
      POSTGRES_DB: "ny_taxi"
    volumes:
      - "ny_taxi_postgres_data:/var/lib/postgresql/data"
    ports:
      - "5432:5432"

  pgadmin:
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: "admin@admin.com"
      PGADMIN_DEFAULT_PASSWORD: "root"
    volumes:
      - "pgadmin_data:/var/lib/pgadmin"
    ports:
      - "8080:80"

volumes:
  ny_taxi_postgres_data:
  pgadmin_data:
```

Now just run:
```bash
docker-compose up      # start everything
docker-compose up -d   # start in background
docker-compose down    # stop everything
docker-compose down -v # stop and remove volumes
```

Docker Compose automatically creates a network so containers can talk to each other using their service names (e.g., `pgdatabase` instead of `localhost`).

### Connecting to Postgres from pgAdmin

1. Open `http://localhost:8080` in browser
2. Login with the email/password from docker-compose
3. Right-click Servers > Create > Server
4. Name it whatever you want
5. Under Connection tab:
   - Host: `pgdatabase` (the service name, not localhost!)
   - Port: `5432`
   - Username: `root`
   - Password: `root`

---

## Part 6: SQL Refresher

Quick review of SQL queries we'll use a lot.

### JOINs

There are two ways to write an INNER JOIN:

```sql
-- Implicit join (old style)
SELECT t.*, z."Zone"
FROM yellow_taxi_data t, zones z
WHERE t."PULocationID" = z."LocationID";

-- Explicit join (preferred)
SELECT t.*, z."Zone"
FROM yellow_taxi_data t
JOIN zones z ON t."PULocationID" = z."LocationID";
```

For multiple joins:
```sql
SELECT 
    t.total_amount,
    zpu."Zone" AS pickup_zone,
    zdo."Zone" AS dropoff_zone
FROM yellow_taxi_data t
JOIN zones zpu ON t."PULocationID" = zpu."LocationID"
JOIN zones zdo ON t."DOLocationID" = zdo."LocationID";
```

### GROUP BY and Aggregations

Count trips per day:
```sql
SELECT 
    CAST(tpep_dropoff_datetime AS DATE) AS day,
    COUNT(1) AS trip_count
FROM yellow_taxi_data
GROUP BY CAST(tpep_dropoff_datetime AS DATE)
ORDER BY day;
```

Multiple aggregations:
```sql
SELECT 
    CAST(tpep_dropoff_datetime AS DATE) AS day,
    COUNT(1) AS trips,
    MAX(total_amount) AS max_amount,
    SUM(total_amount) AS total_revenue
FROM yellow_taxi_data
GROUP BY 1
ORDER BY trips DESC;
```

### Data Quality Checks

Find NULL values:
```sql
SELECT COUNT(*) FROM yellow_taxi_data
WHERE "PULocationID" IS NULL;
```

Find values not in lookup table:
```sql
SELECT * FROM yellow_taxi_data
WHERE "PULocationID" NOT IN (SELECT "LocationID" FROM zones);
```

---

## Part 7: Terraform & GCP

Terraform is Infrastructure as Code (IaC). Instead of clicking around in a cloud console, you write config files describing what you want, and Terraform creates it.

### Why Terraform?

- Version control your infrastructure
- Reproducible environments
- Easy to replicate across dev/staging/production
- Works with AWS, GCP, Azure, and many more

### GCP Setup

1. Create a Google Cloud account (free tier gives you $300 credits)
2. Create a new project
3. Create a service account:
   - Go to IAM & Admin > Service Accounts
   - Create new service account
   - Give it these roles: Storage Admin, BigQuery Admin
4. Download the JSON key file
5. Set the environment variable:
```bash
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/your/key.json"
```

### Terraform Basics

Main files:
- `main.tf` - main configuration
- `variables.tf` - variable definitions

Basic `main.tf` example:
```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "5.6.0"
    }
  }
}

provider "google" {
  project = "your-project-id"
  region  = "us-central1"
}

resource "google_storage_bucket" "data_lake" {
  name          = "your-unique-bucket-name"
  location      = "US"
  force_destroy = true
}

resource "google_bigquery_dataset" "dataset" {
  dataset_id = "trips_data"
  location   = "US"
}
```

### Terraform Commands

The workflow is always:

```bash
# 1. Initialize (download providers)
terraform init

# 2. Preview changes
terraform plan

# 3. Apply changes
terraform apply

# 4. When you're done, destroy resources
terraform destroy
```

For auto-approving (skips confirmation):
```bash
terraform apply -auto-approve
terraform destroy -auto-approve
```

### Common Terraform Flags

- `-auto-approve` - don't ask for confirmation
- `-var="name=value"` - pass variables
- `-var-file="file.tfvars"` - use a variables file

---

## Useful Tips

### Docker Cleanup Commands

```bash
# Remove all stopped containers
docker container prune

# Remove unused images
docker image prune

# Remove unused volumes
docker volume prune

# Nuclear option - remove everything unused
docker system prune -a
```

### Checking Ports

If a port is already in use:
```bash
# Find what's using port 5432
lsof -i :5432
# or
netstat -tulpn | grep 5432
```

### Docker Networking

When containers need to talk to each other:
- In Docker Compose: use service names as hostnames
- Manual setup: create a network with `docker network create`

```bash
docker network create my_network
docker run --network=my_network --name=container1 ...
docker run --network=my_network --name=container2 ...
# container2 can reach container1 using hostname "container1"
```

---

## Summary

What we covered:
1. **Docker** - containerization for reproducible environments
2. **PostgreSQL** - relational database running in Docker
3. **Data Ingestion** - loading data with Python/pandas/SQLAlchemy
4. **Docker Compose** - orchestrating multiple containers
5. **SQL** - querying and aggregating data
6. **Terraform** - infrastructure as code for GCP

The main takeaway: these tools help you build reproducible, scalable data pipelines. Docker ensures your code runs the same everywhere, and Terraform ensures your infrastructure is consistent and version-controlled.

---

## Resources

- [Docker Documentation](https://docs.docker.com/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Terraform Documentation](https://www.terraform.io/docs)
- [Data Engineering Zoomcamp GitHub](https://github.com/DataTalksClub/data-engineering-zoomcamp)
