# Module 2 Summary - Workflow Orchestration with Kestra

---

## Part 1: Introduction to Workflow Orchestration & Kestra Fundamentals

### What is Workflow Orchestration?

Think of a music orchestra with various instruments that need to work together. The conductor helps them play in harmony. Similarly, a **workflow orchestrator** coordinates multiple tools and platforms to work together.

A workflow orchestrator typically:
- **Runs workflows** containing predefined steps
- **Monitors and logs errors** with additional handling when they occur
- **Automatically triggers workflows** based on schedules or events

In data engineering, we often need to move data from one place to another with modifications. The orchestrator manages these steps while providing visibility into the process.

### What is Kestra?

**Kestra** is an open-source, event-driven, infinitely-scalable orchestration platform. Key features include:

| Feature | Description |
|---------|-------------|
| **Flow Code (YAML)** | Build workflows with code, no-code, or AI Copilot |
| **1000+ Plugins** | Integrate with virtually any tool |
| **Multi-language Support** | Use Python, SQL, or any programming language |
| **Flexible Triggers** | Schedule-based or event-based execution |

### Core Kestra Concepts

1. **Flow** - A container for tasks and orchestration logic (defined in YAML)
2. **Tasks** - Individual steps within a flow
3. **Inputs** - Dynamic values passed at runtime
4. **Outputs** - Data passed between tasks and flows
5. **Triggers** - Mechanisms that automatically start flow execution
6. **Execution** - A single run of a flow with a specific state
7. **Variables** - Key-value pairs for reusable values across tasks
8. **Plugin Defaults** - Default values applied to tasks of a given type
9. **Concurrency** - Control how many executions can run simultaneously

### Installing Kestra

Kestra runs via Docker Compose with two main services:
- Kestra server container
- PostgreSQL database container

```bash
cd 02-workflow-orchestration
docker compose up -d
```

Access the UI at: `http://localhost:8080`

### Running Python Code in Kestra

Kestra can execute Python code either:
- From a dedicated file
- Written directly inside the workflow YAML

This allows you to pick the right tools for your pipelines without limitations.

---

## Part 2: Building ETL & ELT Data Pipelines

### ETL Pipeline (Local Postgres)

**ETL = Extract → Transform → Load**

The local pipeline workflow:
1. **Extract** CSV data from GitHub (partitioned by year and month)
2. **Transform** data using Python
3. **Load** data into PostgreSQL database

Key steps in the flow:
- Create tables
- Load data to monthly staging table
- Merge data to final destination table

**Dataset Source:** NYC Taxi and Limousine Commission (TLC) Trip Record Data available in CSV format from the DataTalksClub GitHub repository.

### Scheduling and Backfills

Kestra provides powerful scheduling capabilities:
- **Schedule Trigger** - Run pipelines at specific times (e.g., daily at 9 AM UTC)
- **Backfill** - Process historical data by running workflows for past dates

Example: Backfill green taxi data for year 2019.

### ELT Pipeline (Google Cloud Platform)

**ELT = Extract → Load → Transform**

When working with large datasets in the cloud, ELT is often preferred:

| Step | Description |
|------|-------------|
| **Extract** | Get dataset from source (GitHub) |
| **Load** | Upload to data lake (Google Cloud Storage) |
| **Transform** | Create tables in data warehouse (BigQuery) using data from GCS |

**Advantage:** Leverage cloud's performance for transforming large datasets much faster than local machines.

### GCP Setup for Kestra

Required KV Store values:
- `GCP_PROJECT_ID` - Your Google Cloud project
- `GCP_LOCATION` - Region for resources
- `GCP_BUCKET_NAME` - GCS bucket name
- `GCP_DATASET` - BigQuery dataset name
- `GCP_CREDS` - Service account credentials (keep secure!)

### GCP Pipeline Flow

1. Extract CSV from GitHub
2. Upload to Google Cloud Storage (data lake)
3. Create external table in BigQuery from GCS
4. Create partitioned table in BigQuery
5. Schedule with timezone support (e.g., `America/New_York`)

---

## Part 3: AI Integration & Best Practices

### Using AI for Data Engineering

AI tools help data engineers by:
- **Generating workflows faster** - Describe tasks in natural language
- **Avoiding errors** - Get syntax-correct code following best practices

**Key Insight:** AI is only as good as the context you provide.

### Context Engineering with LLMs

**Problem:** Generic AI assistants (like ChatGPT without context) may produce:
- Outdated plugin syntax
- Incorrect property names
- Hallucinated features that don't exist

**Why?** LLMs are trained on data up to a knowledge cutoff date and don't know about software updates.

**Solution:** Provide proper context to AI!

### Kestra AI Copilot

Kestra's built-in AI Copilot is designed specifically for generating Kestra flows with:
- Full context about latest plugins
- Correct workflow syntax
- Current best practices

**Setup Requirements:**
1. Get Gemini API key from Google AI Studio
2. Configure in docker-compose.yml with `GEMINI_API_KEY`
3. Access via sparkle icon (✨) in Kestra UI

### Retrieval Augmented Generation (RAG)

RAG is a technique that:
1. **Retrieves** relevant information from data sources
2. **Augments** the AI prompt with this context
3. **Generates** responses grounded in real data

**RAG Process in Kestra:**
1. Ingest documents (documentation, release notes)
2. Create embeddings (vector representations)
3. Store embeddings in KV Store or vector database
4. Query with context at runtime
5. Generate accurate, context-aware responses

**RAG Best Practices:**
- Keep documents updated regularly
- Chunk large documents appropriately
- Test retrieval quality

### Deployment & Production

For production deployment:
- Deploy Kestra on Google Cloud
- Sync workflows from Git repository
- Use **Secrets** and **KV Store** for sensitive data
- Never commit API keys to Git

### Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| Port conflict with pgAdmin | Change Kestra port to 18080 |
| CSV column mismatch in BigQuery | Rerun entire execution including re-download |
| Container issues | Stop, remove, and restart containers |

**Recommended Docker Images:**
- `kestra/kestra:v1.1` (stable version)
- `postgres:18`

### Additional Resources

- [Kestra Documentation](https://kestra.io/docs)
- [Blueprints Library](https://kestra.io/blueprints) - Pre-built workflow examples
- [600+ Plugins](https://kestra.io/plugins)
- [Kestra Slack Community](http://kestra.io/slack)

---

## Key Takeaways

1. **Workflow orchestration** is essential for managing complex data pipelines
2. **Kestra** provides a flexible, scalable solution with YAML-based flows
3. **ETL** is ideal for local processing; **ELT** leverages cloud computing power
4. **Scheduling and backfills** enable automated and historical data processing
5. **AI Copilot** accelerates workflow development with proper context
6. **RAG** eliminates AI hallucinations by grounding responses in real data
