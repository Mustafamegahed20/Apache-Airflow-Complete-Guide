# 🚀 Apache Airflow Complete Guide — Zero to Pro (2026)

> A comprehensive, beginner-friendly guide to mastering Apache Airflow 3.x with practical examples, architecture deep-dives, and production-ready patterns.

---

## 📑 Table of Contents

1. [What is Apache Airflow?](#1-what-is-apache-airflow)
2. [What Airflow is NOT](#2-what-airflow-is-not)
3. [Why Not Just a Python Script?](#3-why-not-just-a-python-script)
4. [Core Components of Airflow](#4-core-components-of-airflow)
5. [Airflow Architecture (Deep Dive)](#5-airflow-architecture-deep-dive)
6. [Core Concepts: DAG, Task, Operator](#6-core-concepts-dag-task-operator)
7. [Installation & Setup (Docker)](#7-installation--setup-docker)
8. [Airflow UI Overview](#8-airflow-ui-overview)
9. [Chapter 1 — Your First DAG (Linear DAG & Parsing)](#9-chapter-1--your-first-dag)
10. [Chapter 2 — DAG Versioning](#10-chapter-2--dag-versioning)
11. [Chapter 3 — Operators & DAG Syncing](#11-chapter-3--operators--dag-syncing)
12. [Chapter 4 — XCOMs (Cross-Communication)](#12-chapter-4--xcoms-cross-communication)
13. [Chapter 5 — Manual XCOMs with Kwargs](#13-chapter-5--manual-xcoms-with-kwargs)
14. [Chapter 6 — Parallel Tasks](#14-chapter-6--parallel-tasks)
15. [Chapter 7 — Conditional Branches](#15-chapter-7--conditional-branches)
16. [Chapter 8 — Scheduling Presets](#16-chapter-8--scheduling-presets)
17. [Chapter 9 — Cron Syntax](#17-chapter-9--cron-syntax)
18. [Chapter 10 — Delta Trigger](#18-chapter-10--delta-trigger)
19. [Chapter 11 — Incremental Load & Jinja Templating](#19-chapter-11--incremental-load--jinja-templating)
20. [Chapter 12 — Special Schedules (Event-Based)](#20-chapter-12--special-schedules-event-based)
21. [Chapter 13 — Assets in Airflow](#21-chapter-13--assets-in-airflow)
22. [Chapter 14 — DAG Orchestration (Inherited DAGs)](#22-chapter-14--dag-orchestration-inherited-dags)
23. [Shutting Down & Restarting Airflow](#23-shutting-down--restarting-airflow)
24. [Quick Reference Cheat Sheet](#24-quick-reference-cheat-sheet)

---

## 1. What is Apache Airflow?

**Apache Airflow is an open-source framework used as an orchestrator.**

The keyword here is **orchestrator**. Airflow doesn't process your data — it tells *other systems* when and how to process it.

### The Orchestration Analogy

Imagine you're a music conductor (orchestrator). You have musicians playing different instruments — flute, guitar, piano. Your job is to ensure:
- The flute plays **after** the guitar
- Guitar and drums play **in parallel**
- Piano plays only **after** both finish

You're not playing any instrument yourself — you're **coordinating the flow**. That's exactly what Airflow does for data pipelines.

### Real-World Scenario

As a data engineer, you typically need to:

```
[SQL Database] ──→ [Python Script] ──→ [S3 / Data Lake]
[API Source]   ──→ (Extract Data)  ──→ (Store Raw Data)
                                          │
                                          ▼
                                   [PySpark Processing]
                                          │
                                          ▼
                                   [Data Warehouse]
                                   (Redshift/Snowflake)
```

Two fundamental challenges arise:
1. **Sequencing** — You must extract data *before* processing it. Tasks need a defined order.
2. **Repetition** — This workflow must repeat daily, hourly, or on a schedule automatically.

**Airflow solves both problems.** It orchestrates workflows, manages dependencies, handles scheduling, and provides monitoring through a web UI.

### Key Benefits

- **Pythonic** — Airflow is a Python library. If you know Python, you can use Airflow.
- **Web UI** — Rich visual interface for monitoring DAGs, task states, logs, and more.
- **Open Source** — Free to use, massive community, extensive provider ecosystem.
- **Scalable** — From local development to thousands of DAGs in production.

---

## 2. What Airflow is NOT

Understanding what Airflow is **not** is just as important as knowing what it is. This prevents architectural mistakes.

### ❌ Airflow is NOT a Data Processing Framework

Airflow doesn't process data itself. That's the job of frameworks like **Apache Spark**, **Pandas**, **dbt**, etc. Airflow *tells* these tools when to run — it doesn't replace them.

> **Common confusion**: Airflow has a distributed architecture for processing DAGs (not data). People sometimes assume they should process large datasets directly inside Airflow tasks. **Don't do this.**

### ❌ Airflow is NOT a Real-Time Processing Framework

Airflow handles:
- ✅ Daily schedules
- ✅ Hourly schedules
- ✅ Every 15-30 minutes (acceptable)
- ❌ Real-time / streaming data (microseconds, continuous streams)

For real-time data (IoT sensors, streaming events), use tools like **Apache Kafka**, **Apache Flink**, or **Spark Streaming**.

### ❌ Airflow is NOT an ETL Framework

ETL tools like **Azure Data Factory** or **AWS Glue** can Extract, Transform, AND Load data on their own. Airflow **only orchestrates**:

| Capability | ADF / Glue | Airflow |
|---|---|---|
| Extract data | ✅ Built-in | ❌ Delegates to scripts |
| Transform data | ✅ Built-in | ❌ Delegates to Spark, etc. |
| Load data | ✅ Built-in | ❌ Delegates to scripts |
| Orchestrate | ✅ | ✅ **Primary purpose** |

Airflow says: *"Run this Python script to extract. Now invoke this Spark job to transform. Now trigger this load script."* It delegates — it doesn't execute the heavy lifting.

> **Why use Airflow then?** Because it's open source, handles parallel DAG execution at scale, provides superior monitoring, and is the industry standard for workflow orchestration. Even **Microsoft Fabric** introduced managed Airflow jobs — that tells you something.

---

## 3. Why Not Just a Python Script?

A valid question: *"I know Python. I can call functions sequentially. Why learn Airflow?"*

### Simple Flows — Python Works Fine

```
Task 1  →  Task 2  →  Task 3
```

Yes, you can chain Python functions for this. No argument.

### Complex Flows — Python Gets Painful

```
         ┌─→ Task 2 ─┐
Task 1 ──┤            ├──→ Decision ──┬─→ Task 5
         └─→ Task 3 ─┘               └─→ Task 6
```

Now you need:
- **Parallel execution** (Task 2 and Task 3 simultaneously)
- **Dependency management** (Task 4 only runs if BOTH 2 and 3 succeed)
- **Conditional branching** (Decision node picks Task 5 OR Task 6)
- **State management** (passing data between tasks)
- **Retry logic**, **alerting**, **logging**, **scheduling**...

Building all this from scratch in Python is possible but **extremely complex to maintain**. Airflow gives you all of this out of the box.

### How Airflow Executes Tasks

Using the complex flow above, here's how Airflow processes it:

1. **Task 1** runs first (no dependencies)
2. **Task 2** and **Task 3** run **in parallel** (both depend only on Task 1)
3. If **any parallel task fails**, downstream tasks are **skipped**
4. If both succeed → **Decision node** evaluates a condition
5. Based on the condition, either **Task 5** or **Task 6** runs; the other is **skipped**

---

## 4. Core Components of Airflow

Understanding these components is essential before diving into architecture. Think of these as the building blocks.

### 4.1 Metadata Database

The **backbone** of Airflow (especially in Airflow 3.x). It stores everything about your DAGs:

- DAG definitions and structure
- Schedule information
- DAG run statuses (success, failed, running)
- Task instance states
- XCom values (inter-task communication)

**Database used**: PostgreSQL (recommended and default in Docker setup).

> In Airflow 3.x, the metadata database is even more critical. **All communication between components goes through the database via the API server.**

### 4.2 DAG File Processor

The component that **continuously scans** your `dags/` folder:

1. Reads your Python DAG files
2. Parses the DAG structure
3. **Serializes** the DAG (converts to JSON-like representation)
4. Stores the serialized DAG in the metadata database (via the API server)

```
Your Code (Python) → DAG File Processor → Serialized JSON → Metadata DB
```

The scanning interval is configurable (default: 30 seconds in Airflow 3.x). You can change it in `airflow.cfg`:
```ini
dag_bag_import_timeout = 30
```

### 4.3 API Server (Web Server)

The **central hub** of Airflow. It serves two purposes:

1. **Frontend**: Provides the web UI (built in React) at `localhost:8080`
2. **Backend**: Exposes REST APIs that ALL other components use to communicate

In Airflow 3.x, **every component communicates with the metadata database through the API server**. No direct database access is allowed.

```
User (UI) ──→ API Server ──→ Metadata DB
DAG Processor ──→ API Server ──→ Metadata DB
Scheduler ──→ API Server ──→ Metadata DB
Worker ──→ API Server ──→ Metadata DB
```

### 4.4 Scheduler

Determines **WHAT** and **WHEN** tasks need to execute.

- Reads DAG definitions from the database
- Creates DAG runs based on schedule
- Places tasks into the **execution queue**

Remember two words: **What** and **When**.

### 4.5 Executor

Determines **HOW** and **WHERE** tasks should run. Think of it as a **contractor** — it doesn't do the work itself but assigns work to workers.

**Types of Executors:**

| Executor | Description |
|---|---|
| `LocalExecutor` | Runs tasks in parallel on a single machine (using multiple processes) |
| `CeleryExecutor` | Distributes tasks across multiple worker machines via Celery |
| `KubernetesExecutor` | Spins up a new Kubernetes pod for each task |

> **Important**: Despite its name, the Executor does NOT execute tasks. It creates an execution strategy and places tasks in the queue.

### 4.6 Workers

The components that **actually execute** your task code. Workers pick tasks from the queue and run them.

```
Scheduler → Execution Queue → Executor (strategy) → Redis Queue → Workers (actual execution)
```

### 4.7 Queue

A prioritization layer for tasks. When multiple DAGs with many tasks run simultaneously, the queue ensures workers pick tasks in the correct priority order.

**Two types of queues:**
1. **Execution Queue** — Between Scheduler and Executor
2. **Redis Queue** (or Celery queue) — Between Executor and Workers

### 4.8 Triggerer

Handles **deferred tasks** — tasks that are waiting for an external event (like an API response or a long-running Spark job).

**Analogy**: A senior developer sends an email and waits for a reply. Instead of sitting idle, they hand the "monitoring" task to a junior developer and pick up new work. When the reply arrives, the junior notifies the senior.

**Flow:**
1. Worker encounters a `defer` function in a task
2. Task is handed to the Triggerer
3. Worker is freed to pick up new tasks
4. Triggerer monitors the deferred condition
5. When condition is met, Triggerer updates the task state in the database
6. Scheduler picks it up and sends the remaining work back to a Worker

---

## 5. Airflow Architecture (Deep Dive)

This is the **most important topic** for interviews and real-world understanding. Based on the latest Airflow 3.x architecture.

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        AIRFLOW SYSTEM                           │
│                                                                 │
│  ┌──────────┐     ┌───────────────┐     ┌──────────────────┐   │
│  │  User /  │     │  DAG File     │     │   Metadata DB    │   │
│  │Developer │     │  Processor    │     │   (PostgreSQL)   │   │
│  │          │     │               │     │                  │   │
│  │ Writes   │     │ Scans dags/   │     │ Stores:          │   │
│  │ DAG code │     │ folder every  │     │ - Serialized DAGs│   │
│  │ in dags/ │     │ 30 seconds    │     │ - DAG runs       │   │
│  └────┬─────┘     └───────┬───────┘     │ - Task states    │   │
│       │                   │             │ - XCom values    │   │
│       ▼                   │             └────────▲─────────┘   │
│  ┌──────────┐             │                      │             │
│  │  dags/   │─────────────┘              ┌───────┴────────┐   │
│  │  folder  │      Parses & stores       │   API Server   │   │
│  │          │      via API ──────────────▶│  (FastAPI)     │   │
│  └──────────┘                            │                │   │
│                                          │ Central hub:   │   │
│  ┌────────────┐    Reads DAGs via API    │ - Serves UI    │   │
│  │ Scheduler  │◄─────────────────────────│ - REST APIs    │   │
│  │            │                          │ - All comms    │   │
│  │ What/When  │                          └───────▲────────┘   │
│  └─────┬──────┘                                  │             │
│        │ Places tasks                            │             │
│        ▼                                         │             │
│  ┌──────────────┐                                │             │
│  │  Execution   │                                │             │
│  │    Queue     │                                │             │
│  └──────┬───────┘                                │             │
│         │                                        │             │
│         ▼                                        │             │
│  ┌────────────┐    Strategy applied              │             │
│  │  Executor  │─────────────────┐                │             │
│  │  How/Where │                 │                │             │
│  └────────────┘                 ▼                │             │
│                          ┌──────────────┐        │             │
│                          │  Redis Queue │        │             │
│                          │  (Priority)  │        │             │
│                          └──────┬───────┘        │             │
│                                 │                │             │
│                                 ▼                │             │
│                          ┌────────────┐          │             │
│                          │  Workers   │──────────┘             │
│                          │  (Execute) │  Writes results        │
│                          └──────┬─────┘  via API               │
│                                 │                              │
│                          ┌──────▼─────┐                        │
│                          │  Triggerer │ (for deferred tasks)   │
│                          └────────────┘                        │
└─────────────────────────────────────────────────────────────────┘
```

### Key Points About Airflow 3.x Architecture

1. **All components talk to the Metadata DB through the API Server** — no direct database connections.
2. Some components use **in-place APIs** (internal calls), while the UI uses **explicit APIs** (user-triggered).
3. In production, each component typically runs on a **separate machine** (multi-node architecture).
4. Locally (for learning), everything runs on a **single machine** via Docker containers.

### Single-Node vs Multi-Node

| Aspect | Single-Node (Local) | Multi-Node (Production) |
|---|---|---|
| All components | Same machine | Separate machines/clusters |
| Workers | Limited by local CPU | Scale to hundreds |
| Database | Local PostgreSQL container | Managed database service |
| Use case | Development & learning | Enterprise deployments |

---

## 6. Core Concepts: DAG, Task, Operator

### DAG — Directed Acyclic Graph

A **DAG** is a workflow — a collection of tasks with defined dependencies and execution order.

- **Directed** — Each task has a defined direction/order (Task A → Task B)
- **Acyclic** — No cycles allowed. A downstream task **cannot** loop back to an upstream task.
- **Graph** — Consists of nodes (tasks) and edges (dependencies)

```
✅ Valid DAG:     Task A → Task B → Task C

❌ NOT a DAG:     Task A → Task B → Task C → Task A  (cycle!)
```

> **Why no cycles?** A cycle would create an infinite recursive loop — Task C triggers Task A, which triggers B, then C, then A again... forever.

### Task

A **single unit of work** within a DAG. Examples:
- Making an API call
- Running a SQL query
- Executing a bash command
- Running a Python function

**Pro Tip**: Keep tasks **granular**. A task with 50 lines of code is hard to debug when it fails. Multiple small tasks make debugging fast and reruns efficient.

### Operator

An operator is a **pre-built template** that defines the *category* of a task.

| Operator | Purpose |
|---|---|
| `PythonOperator` / `@task.python` | Execute a Python function |
| `BashOperator` / `@task.bash` | Execute a bash command |
| `SQLOperator` | Execute SQL queries |
| `EmailOperator` | Send emails |
| `DummyOperator` | Placeholder/no-op task |

Operators save time by providing built-in connection management, error handling, and logging. For example, the `SQLOperator` automatically creates a connection, runs the query, and closes the connection.

---

## 7. Installation & Setup (Docker)

### Prerequisites

#### 1. Enable Virtualization
- Open **Task Manager** → **Performance** tab
- Check that **Virtualization** is **Enabled**
- If not, enable it in your BIOS settings

#### 2. Enable WSL2 (Windows Subsystem for Linux)
1. Search **"Windows Features"** in the Start menu
2. Enable:
   - ✅ **Windows Subsystem for Linux**
   - ✅ **Virtual Machine Platform**
3. Click OK and **restart your computer**

#### 3. Install WSL2 (if not auto-installed)
```powershell
wsl --install
```

Verify installation:
```powershell
wsl --list -v
```
You should see **Docker Desktop** running.

#### 4. Install Docker Desktop
1. Go to [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
2. Download for your OS (Windows/Mac/Linux)
3. Install and **restart your computer**
4. Open Docker Desktop → Settings → Resources → Confirm **WSL2 backend** is enabled

### Project Setup

#### Step 1: Create Project Structure
```
airflow-tutorial/
├── dags/           # Your DAG files go here
├── logs/           # Airflow logs (auto-populated)
├── plugins/        # Custom plugins
├── config/         # Configuration files
├── .env            # Environment variables
└── docker-compose.yml
```

Create the folders:
```bash
mkdir airflow-tutorial
cd airflow-tutorial
mkdir dags logs plugins config
```

#### Step 2: Download docker-compose.yml

Download the official Airflow Docker Compose file:
```bash
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/stable/docker-compose.yaml'
```

#### Step 3: Modify docker-compose.yml

**Change 1**: Add port mapping for PostgreSQL (to access the database directly):

Find the `postgres` service and add:
```yaml
postgres:
    image: postgres:13
    environment:
      POSTGRES_USER: airflow
      POSTGRES_PASSWORD: airflow
      POSTGRES_DB: airflow
    ports:
      - "5432:5432"    # ← ADD THIS LINE
```

**Change 2**: Disable example DAGs (they clutter your view with 80+ sample DAGs):

Find this line:
```yaml
AIRFLOW__CORE__LOAD_EXAMPLES: 'true'
```
Change to:
```yaml
AIRFLOW__CORE__LOAD_EXAMPLES: 'false'
```

#### Step 4: Create .env file

Create a `.env` file in the project root:
```
AIRFLOW_UID=50000
```

#### Step 5: Create Virtual Environment (Optional but Recommended)

This is for **local development comfort** (IDE autocomplete, no import warnings). Your DAGs still run inside Docker.

```bash
pip install uv          # Modern Python package manager
uv init                 # Initialize project
uv sync                 # Create virtual environment
uv add apache-airflow   # Install Airflow library
```

Activate the virtual environment:
```bash
# Windows
.venv\Scripts\activate

# Mac/Linux
source .venv/bin/activate
```

#### Step 6: Start Airflow

```bash
docker compose up -d
```

This pulls all required Docker images and starts the containers:
- `postgres` — Metadata database
- `airflow-init` — One-time initialization (exits after setup)
- `airflow-apiserver` — API Server / Web UI
- `airflow-scheduler` — Scheduler
- `airflow-worker` — Worker
- `airflow-triggerer` — Triggerer
- `airflow-dag-processor` — DAG File Processor
- `redis` — Message broker/queue

Wait for all containers to be healthy (check Docker Desktop or run `docker ps`).

#### Step 7: Access Airflow UI

Open your browser: **http://localhost:8080**

- **Username**: `airflow`
- **Password**: `airflow`

### How DAGs Sync to Docker

Your local `dags/` folder is **bind-mounted** to `/opt/airflow/dags/` inside the Docker containers. Any file you save locally is immediately available inside Docker.

```
Local Machine                    Docker Container
dags/my_dag.py  ──bind mount──▶  /opt/airflow/dags/my_dag.py
```

The DAG File Processor scans `/opt/airflow/dags/` every 30 seconds (configurable) and parses new/changed files.

---

## 8. Airflow UI Overview

### Main Sections

| Section | Purpose |
|---|---|
| **DAGs** | Your primary workspace — view all DAGs, trigger runs, see status |
| **DAG Runs** | History of all DAG executions with statuses |
| **Task Instances** | Individual task execution history |
| **Assets** | New in Airflow 3.x — data-driven dependencies |
| **Browse** | Audit logs, XComs, required actions |
| **Admin** | Variables, Pools, Connections, Providers, Plugins |
| **Security** | Users, Roles, Permissions |

### Key Settings

- **View Toggle**: Switch between **Grid View** and **Graph View** for DAGs
- **Dark Mode**: Available in user settings
- **Time Zone**: Configured per user
- **Default Graph View**: Enable in settings for visual DAG representation

### Admin → Connections

This is where you configure connections to external systems:
- PostgreSQL, MySQL, SQL Server
- AWS S3, Redshift, Athena
- Azure Blob Storage, Synapse
- Snowflake, BigQuery
- And many more...

---

## 9. Chapter 1 — Your First DAG

### File: `dags/1_first_dag.py`

```python
from airflow.sdk import dag, task

@dag
def first_dag():
    """My very first Airflow DAG"""

    @task.python
    def first_task():
        print("This is the first task")

    @task.python
    def second_task():
        print("This is the second task")

    @task.python
    def third_task():
        print("DAG complete")

    # Define dependencies
    first = first_task()
    second = second_task()
    third = third_task()

    first >> second >> third

# Register the DAG
first_dag()
```

### Key Concepts

1. **`@dag` decorator** — Marks a function as a DAG. The function name becomes the DAG ID (unless you specify `dag_id=`).
2. **`@task.python` decorator** — Marks a function as a Python task. Default if you just use `@task`.
3. **`>>` operator** — Defines task dependencies (bitshift operator). `first >> second` means "run second after first."
4. **DAG registration** — Calling `first_dag()` at the bottom registers the DAG with Airflow.

### Viewing Your DAG

1. Save the file in `dags/`
2. Wait for parsing (check `logs/dag_processor/` for confirmation)
3. Go to Airflow UI → DAGs → Click your DAG
4. Switch to **Graph View** to see the visual flow
5. Enable the DAG (toggle the pause switch)
6. Click **Trigger** → **Trigger** to run manually

### Checking Logs

Click on any task → **Logs** tab to see the output (e.g., your `print()` statements).

### Verifying DAG Parsing

If your DAG doesn't appear in the UI:
1. Check `logs/dag_processor/` for parsing logs
2. Wait at least 30 seconds for the parser to scan
3. Check for syntax errors in your code
4. Click the **"Reparse DAG"** option in the DAG menu (three dots)

---

## 10. Chapter 2 — DAG Versioning

Airflow 3.x automatically tracks DAG versions. Every significant code change creates a new version.

### File: `dags/2_version_dag.py`

```python
from airflow.sdk import dag, task

@dag
def version_dag():

    @task.python
    def first_task():
        print("This is the first task")

    @task.python
    def second_task():
        print("This is the second task")

    @task.python
    def third_task():
        print("This is the third task")

    first = first_task()
    second = second_task()
    third = third_task()

    first >> second >> third

version_dag()
```

### How Versioning Works

- Initial deployment → **V1**
- Add a new task or make significant changes → **V2**
- Each version is tracked and viewable in the UI
- You can view the code and state of **any previous version**
- You can even trigger a DAG run using a previous version

### Viewing Versions

In the DAG detail view, look for the version dropdown (e.g., `V1`, `V2`). You can switch between versions to compare code and review run history for each version.

> **Note**: Minor changes (like updating a print statement) may not always create a new version. Significant structural changes (adding/removing tasks, changing dependencies) reliably trigger version increments.

---

## 11. Chapter 3 — Operators & DAG Syncing

### Using Both `@task` Decorator and Traditional Operators

### File: `dags/3_operators.py`

```python
from airflow.sdk import dag, task
from airflow.operators.bash import BashOperator

@dag
def operators_dag():

    @task.python
    def first_task():
        print("Python task executed")

    @task.bash
    def second_task():
        """Bash task using decorator"""
        return "echo 'Bash task via decorator'"

    # Traditional Bash Operator (old-school method)
    bash_task_old = BashOperator(
        task_id="bash_task_old_school",
        bash_command="echo 'Bash task via operator'"
    )

    first = first_task()
    second = second_task()

    first >> second >> bash_task_old

operators_dag()
```

### Two Ways to Define Bash Tasks

| Method | Syntax | Notes |
|---|---|---|
| **Decorator (recommended)** | `@task.bash` + `return "command"` | Modern, readable, consistent |
| **Traditional Operator** | `BashOperator(task_id=..., bash_command=...)` | Classic approach, still works |

> **Recommendation**: Use the `@task` decorator method. It's cleaner, more readable, and recommended by Airflow 3.x documentation.

### Verifying DAG Sync

To confirm your files are properly synced to Docker:

```bash
# Enter any Airflow container
docker exec -it <container_name> bash

# Navigate to DAGs folder
cd /opt/airflow/dags

# List your DAG files
ls
```

---

## 12. Chapter 4 — XCOMs (Cross-Communication)

XCOMs (Cross-Communications) allow tasks to **share data** with each other. This is how you pass output from one task as input to the next.

### Automatic XCOMs (Return Values)

### File: `dags/4_xcoms_auto.py`

```python
from airflow.sdk import dag, task

@dag
def xcoms_dag_auto():

    @task.python
    def first_task():
        """Extract data and return it"""
        print("Extracting data - First task")
        fetched_data = {"data": [1, 2, 3, 4, 5]}
        return fetched_data  # Auto-pushed to XCom as 'return_value'

    @task.python
    def second_task(data: dict):
        """Transform data received from first task"""
        print("Transforming data")
        fetch_data = data["data"]
        transform_data = fetch_data * 2  # Duplicate the list
        return {"transform_data": transform_data}

    @task.python
    def third_task(data: dict):
        """Load transformed data"""
        transformed_data = data["transform_data"]
        load_data = {"loaded": transformed_data}
        print(f"Loading data: {load_data}")
        return load_data

    # Automatic dependency detection!
    # Airflow sees that second_task takes first_task's output
    first = first_task()
    second = second_task(first)
    third = third_task(second)

xcoms_dag_auto()
```

### How Automatic XCOMs Work

1. Whatever you **`return`** from a task is automatically stored in XCom under the key `return_value`
2. When you pass one task's output as an argument to another, Airflow **automatically** creates the dependency
3. You don't need to manually define `>>` dependencies — Airflow infers them from the data flow

### Viewing XCOMs in the UI

Click on any task → **XCom** tab:
- **Key**: `return_value` (automatically assigned)
- **Value**: The data your task returned

---

## 13. Chapter 5 — Manual XCOMs with Kwargs

For more control over XCom keys and values, use **manual push/pull** via the `kwargs` context.

### File: `dags/5_xcoms_kwargs.py`

```python
from airflow.sdk import dag, task

@dag
def xcoms_dag_kwargs():

    @task.python
    def first_task(**kwargs):
        print("Extracting data - First task")
        fetched_data = {"data": [1, 2, 3, 4, 5]}

        # Extract task instance from kwargs
        ti = kwargs["ti"]

        # Manually push to XCom with custom key
        ti.xcom_push(key="return_result", value=fetched_data)

    @task.python
    def second_task(**kwargs):
        ti = kwargs["ti"]

        # Pull from XCom using task_id and key
        fetched_data = ti.xcom_pull(
            task_ids="first_task",
            key="return_result"
        )

        data = fetched_data["data"]
        transform_data = data * 2
        ti.xcom_push(key="return_result", value={"transform_data": transform_data})

    @task.python
    def third_task(**kwargs):
        ti = kwargs["ti"]

        transformed_data = ti.xcom_pull(
            task_ids="second_task",
            key="return_result"
        )

        load_data = transformed_data["transform_data"]
        print(f"Loading data: {load_data}")
        ti.xcom_push(key="return_result", value={"loaded_data": load_data})

    # Must define dependencies manually (no auto-detection with kwargs)
    first = first_task()
    second = second_task()
    third = third_task()

    first >> second >> third

xcoms_dag_kwargs()
```

### Key Differences: Auto vs Manual XCOMs

| Feature | Auto XCOMs | Manual XCOMs (kwargs) |
|---|---|---|
| Push mechanism | `return` value | `ti.xcom_push(key, value)` |
| Pull mechanism | Function argument | `ti.xcom_pull(task_ids, key)` |
| Key name | Always `return_value` | Custom key names |
| Auto dependencies | ✅ Yes | ❌ Must define manually |
| Control level | Low | High |
| Best for | Simple DAGs | Complex DAGs, custom keys |

### Understanding `kwargs`

- `kwargs` (keyword arguments) is a **dictionary** containing context variables for the current task execution
- Key variable: `ti` (Task Instance) — provides `xcom_push()` and `xcom_pull()` methods
- Other useful context variables: `logical_date`, `data_interval_start`, `data_interval_end`, etc.

---

## 14. Chapter 6 — Parallel Tasks

### File: `dags/6_parallel_tasks.py`

```python
from airflow.sdk import dag, task

@dag
def parallel_dag():

    @task.python
    def extract_task(**kwargs):
        """Extract data from multiple sources"""
        print("Extracting data")
        extracted_data = {
            "api_extracted_data": [1, 2, 3],
            "db_extracted_data": [4, 5, 6],
            "s3_extracted_data": [7, 8, 9]
        }
        ti = kwargs["ti"]
        ti.xcom_push(key="return_value", value=extracted_data)

    @task.python
    def transform_api_task(**kwargs):
        ti = kwargs["ti"]
        data = ti.xcom_pull(task_ids="extract_task", key="return_value")
        api_data = data["api_extracted_data"]
        transformed = [x * 10 for x in api_data]
        print(f"Transforming API data: {transformed}")
        ti.xcom_push(key="return_value", value={"api_data": transformed})

    @task.python
    def transform_db_task(**kwargs):
        ti = kwargs["ti"]
        data = ti.xcom_pull(task_ids="extract_task", key="return_value")
        db_data = data["db_extracted_data"]
        transformed = [x * 10 for x in db_data]
        print(f"Transforming DB data: {transformed}")
        ti.xcom_push(key="return_value", value={"db_data": transformed})

    @task.python
    def transform_s3_task(**kwargs):
        ti = kwargs["ti"]
        data = ti.xcom_pull(task_ids="extract_task", key="return_value")
        s3_data = data["s3_extracted_data"]
        transformed = [x * 10 for x in s3_data]
        print(f"Transforming S3 data: {transformed}")
        ti.xcom_push(key="return_value", value={"s3_data": transformed})

    @task.bash
    def load_task(**kwargs):
        ti = kwargs["ti"]
        api = ti.xcom_pull(task_ids="transform_api_task", key="return_value")
        db = ti.xcom_pull(task_ids="transform_db_task", key="return_value")
        s3 = ti.xcom_pull(task_ids="transform_s3_task", key="return_value")
        return f"echo 'Loaded: {api}, {db}, {s3}'"

    # Define dependencies
    extract = extract_task()
    t_api = transform_api_task()
    t_db = transform_db_task()
    t_s3 = transform_s3_task()
    load = load_task()

    # Parallel execution: extract → [api, db, s3] → load
    extract >> [t_api, t_db, t_s3] >> load

parallel_dag()
```

### Defining Parallel Dependencies

The key syntax for parallel tasks is using a **list**:

```python
extract >> [t_api, t_db, t_s3] >> load
```

This means:
1. `extract` runs first
2. `t_api`, `t_db`, `t_s3` run **in parallel** after extract
3. `load` runs only after **ALL three** parallel tasks succeed

### Visual Representation

```
                    ┌─→ transform_api ─┐
extract_task ───────┤─→ transform_db  ─┼──→ load_task
                    └─→ transform_s3  ─┘
```

> **Pro Tip**: Using manual `xcom_push/pull` prevents messy dependency arrows in the DAG visualization. Automatic XCOMs would create direct connections from each transform back to extract, cluttering the graph.

---

## 15. Chapter 7 — Conditional Branches

Branches allow your DAG to take different paths based on runtime conditions.

### File: `dags/7_branches.py`

```python
from airflow.sdk import dag, task

@dag
def branch_dag():

    @task.python
    def extract_task(**kwargs):
        extracted_data = {
            "api_extracted_data": [1, 2, 3],
            "db_extracted_data": [4, 5, 6],
            "s3_extracted_data": [7, 8, 9],
            "weekend_flag": False  # ← Condition flag
        }
        ti = kwargs["ti"]
        ti.xcom_push(key="return_value", value=extracted_data)

    @task.python
    def transform_api_task(**kwargs):
        ti = kwargs["ti"]
        data = ti.xcom_pull(task_ids="extract_task", key="return_value")
        print(f"Transforming API data: {data['api_extracted_data']}")
        ti.xcom_push(key="return_value", value=data)

    @task.python
    def transform_db_task(**kwargs):
        ti = kwargs["ti"]
        data = ti.xcom_pull(task_ids="extract_task", key="return_value")
        print(f"Transforming DB data: {data['db_extracted_data']}")
        ti.xcom_push(key="return_value", value=data)

    @task.python
    def transform_s3_task(**kwargs):
        ti = kwargs["ti"]
        data = ti.xcom_pull(task_ids="extract_task", key="return_value")
        print(f"Transforming S3 data: {data['s3_extracted_data']}")
        ti.xcom_push(key="return_value", value=data)

    # ─── BRANCH (Decider Node) ───
    @task.branch
    def decide_task(**kwargs):
        """Returns the task_id of the next task to run"""
        ti = kwargs["ti"]
        data = ti.xcom_pull(task_ids="extract_task", key="return_value")
        weekend_flag = data.get("weekend_flag", False)

        if weekend_flag:
            return "no_load_task"  # Must match function name / task_id
        else:
            return "load_task"

    @task.bash
    def load_task(**kwargs):
        return "echo 'Loading data to destination...'"

    @task.bash
    def no_load_task(**kwargs):
        return "echo 'Weekend - skipping load'"

    # Dependencies
    extract = extract_task()
    t_api = transform_api_task()
    t_db = transform_db_task()
    t_s3 = transform_s3_task()
    decide = decide_task()
    load = load_task()
    no_load = no_load_task()

    extract >> [t_api, t_db, t_s3] >> decide >> [load, no_load]

branch_dag()
```

### How Branching Works

1. Use `@task.branch` decorator on the decision function
2. The function must **return the task_id** (function name) of the next task to execute
3. The returned task runs; **all other branch tasks are skipped**
4. The list after the branch `>> [load, no_load]` defines the possible paths — only one executes

### Critical Rule

> The **return value** of a branch task must **exactly match** a downstream task's function name (which is also its `task_id`). Mismatches will cause failures.

### Visual Result

```
                    ┌─→ transform_api ─┐
extract_task ───────┤─→ transform_db  ─┼──→ decide_task ──┬─→ load_task ✅
                    └─→ transform_s3  ─┘                  └─→ no_load   ⊘ (skipped)
```

---

## 16. Chapter 8 — Scheduling Presets

### Scheduling Parameters

| Parameter | Required | Description |
|---|---|---|
| `start_date` | ✅ Yes | When the DAG becomes eligible to run |
| `end_date` | Optional | When to stop scheduling |
| `schedule` | ✅ Yes | Frequency/timing of runs |
| `catchup` | Optional | Whether to backfill missed intervals (default: `False` in 3.x) |
| `is_paused_upon_creation` | Optional | Auto-enable DAG on deployment (default: `True`) |

### File: `dags/8_schedule_preset.py`

```python
from airflow.sdk import dag, task
from pendulum import datetime

@dag(
    start_date=datetime(year=2026, month=1, day=26, tz="America/Halifax"),
    schedule="@daily",
    catchup=True,
    is_paused_upon_creation=False,
)
def first_schedule_dag():

    @task.python
    def my_task():
        print("Scheduled task executed!")

    my_task()

first_schedule_dag()
```

### Available Presets

| Preset | Equivalent Cron | Description |
|---|---|---|
| `@once` | — | Run once and only once |
| `@hourly` | `0 * * * *` | Every hour at minute 0 |
| `@daily` | `0 0 * * *` | Every day at midnight |
| `@weekly` | `0 0 * * 0` | Every Sunday at midnight |
| `@monthly` | `0 0 1 * *` | First day of each month at midnight |
| `@yearly` | `0 0 1 1 *` | January 1st at midnight |

### Using Pendulum for Dates

Always use **Pendulum** for dates in Airflow — it handles timezone conversions automatically:

```python
from pendulum import datetime

start_date = datetime(year=2026, month=1, day=26, tz="America/New_York")
```

### Catchup Behavior

- `catchup=True` — Airflow backfills all missed intervals between `start_date` and now
- `catchup=False` — Only runs from the current interval onward
- **Default in Airflow 3.x**: `False`

---

## 17. Chapter 9 — Cron Syntax

Cron expressions give you precise control over scheduling.

### Cron Format

```
┌───────── Minute (0-59)
│ ┌─────── Hour (0-23)
│ │ ┌───── Day of Month (1-31)
│ │ │ ┌─── Month (1-12)
│ │ │ │ ┌─ Day of Week (MON-FRI or 0-6)
│ │ │ │ │
* * * * *
```

### Examples

| Cron Expression | Meaning |
|---|---|
| `0 16 * * MON-FRI` | 4:00 PM every weekday |
| `30 9 * * *` | 9:30 AM every day |
| `0 0 1 * *` | Midnight on the 1st of every month |
| `0 */2 * * *` | Every 2 hours |
| `15 10 * * MON` | 10:15 AM every Monday |

### File: `dags/9_schedule_cron.py`

```python
from airflow.sdk import dag, task
from pendulum import datetime
from airflow.timetables.trigger import CronTriggerTimetable

@dag(
    schedule=CronTriggerTimetable(
        "0 16 * * MON-FRI",
        timezone="America/Halifax"
    ),
    start_date=datetime(year=2026, month=1, day=26, tz="America/Halifax"),
    end_date=datetime(year=2026, month=1, day=31, tz="America/Halifax"),
    catchup=True,
)
def cron_schedule_dag():

    @task.python
    def my_task():
        print("Cron-scheduled task executed!")

    my_task()

cron_schedule_dag()
```

> **Pro Tip**: Use tools like [crontab.guru](https://crontab.guru) to build and verify cron expressions.

---

## 18. Chapter 10 — Delta Trigger

### The Problem Cron Can't Solve

What if you need to run a DAG **every 3 days**? With cron, you'd manually list dates for each month, and it breaks across month boundaries (e.g., Jan 31 → Feb 3, not Feb 1).

### Solution: Delta Trigger

```python
from airflow.sdk import dag, task
from pendulum import datetime, duration
from airflow.timetables.trigger import DeltaTriggerTimetable

@dag(
    schedule=DeltaTriggerTimetable(
        duration(days=3)
    ),
    start_date=datetime(year=2026, month=1, day=26, tz="America/Halifax"),
    catchup=True,
)
def delta_schedule_dag():

    @task.python
    def my_task():
        print("Delta-scheduled task executed!")

    my_task()

delta_schedule_dag()
```

### How Delta Differs from Cron

| Feature | Cron | Delta |
|---|---|---|
| Fixed calendar times | ✅ | ❌ |
| Fixed intervals (e.g., every 3 days) | ❌ (complex) | ✅ |
| Cross-month boundary handling | Manual | Automatic |
| Use case | "Run at 4 PM daily" | "Run every 72 hours" |

---

## 19. Chapter 11 — Incremental Load & Jinja Templating

### What is Incremental Load?

Instead of loading ALL data every run (full load), you only load **new data since the last run**.

```
Full Load:     Load EVERYTHING every time (wasteful)
Incremental:   Load only data from interval_start to interval_end (efficient)
```

### Interval-Based Scheduling

For incremental loads, Airflow uses **interval-based triggers** instead of trigger-based:

```
Timeline:
|──── Jan 1 ────|──── Jan 2 ────|──── Jan 3 ────|
     start_date   ← interval →    ← interval →

DAG run 1: Runs on Jan 2, processes data from Jan 1 → Jan 2
DAG run 2: Runs on Jan 3, processes data from Jan 2 → Jan 3
```

> **Critical**: The DAG does NOT run on `start_date`. It runs at the **end of the first interval**, because it needs a complete interval to process.

### Context Variables

| Variable | Description |
|---|---|
| `data_interval_start` | Start of the current interval |
| `data_interval_end` | End of the current interval |
| `logical_date` | The logical execution date |

### File: `dags/11_incremental_load.py`

```python
from airflow.sdk import dag, task
from pendulum import datetime
from airflow.timetables.interval import CronDataIntervalTimetable

@dag(
    schedule=CronDataIntervalTimetable(
        "0 0 * * *",  # Daily at midnight
        timezone="America/Halifax"
    ),
    start_date=datetime(year=2026, month=1, day=26, tz="America/Halifax"),
    end_date=datetime(year=2026, month=1, day=29, tz="America/Halifax"),
    catchup=True,
)
def incremental_load_dag():

    # ─── Python: Access dates via kwargs ───
    @task.python
    def incremental_data_fetch(**kwargs):
        interval_start = kwargs["data_interval_start"]
        interval_end = kwargs["data_interval_end"]
        print(f"Fetching data from {interval_start} to {interval_end}")
        # In production: pass these dates to your SQL query or API call
        # e.g., SELECT * FROM orders WHERE created_at BETWEEN '{start}' AND '{end}'

    # ─── Bash: Access dates via Jinja templates ───
    @task.bash
    def incremental_data_process():
        return "echo 'Processing data from {{ data_interval_start }} to {{ data_interval_end }}'"

    fetch = incremental_data_fetch()
    process = incremental_data_process()

    fetch >> process

incremental_load_dag()
```

### Jinja Templating in Bash Operators

In `@task.bash`, you can use **Jinja2 template variables** directly with `{{ variable_name }}`:

```python
@task.bash
def my_bash_task():
    return "echo 'Interval: {{ data_interval_start }} to {{ data_interval_end }}'"
```

These are automatically rendered by Airflow at execution time. No `kwargs` needed!

### Python vs Bash — Accessing Context

| Method | Python Tasks | Bash Tasks |
|---|---|---|
| Access mechanism | `kwargs["data_interval_start"]` | `{{ data_interval_start }}` |
| Template type | Python dictionary | Jinja2 template |
| Requires kwargs | ✅ Yes | ❌ No |

---

## 20. Chapter 12 — Special Schedules (Event-Based)

For irregular dates that don't follow any pattern (birthdays, company events, holidays).

### File: `dags/12_special_dates.py`

```python
from airflow.sdk import dag, task
from pendulum import datetime
from airflow.timetables.events import EventsTimetable

special_dates = [
    datetime(2026, 1, 1),    # New Year
    datetime(2026, 2, 14),   # Valentine's Day
    datetime(2026, 6, 1),    # Some event
    datetime(2026, 12, 31),  # New Year's Eve
]

@dag(
    schedule=EventsTimetable(event_dates=special_dates),
    start_date=datetime(year=2026, month=1, day=1),
    end_date=datetime(year=2026, month=12, day=31),
    catchup=True,
)
def special_dates_dag():

    @task.python
    def event_task(**kwargs):
        logical_date = kwargs["logical_date"]
        print(f"Running special event task for: {logical_date}")

    event_task()

special_dates_dag()
```

### When to Use

- Company-specific holidays
- Quarterly reporting dates
- Regulatory compliance deadlines
- Any dates that don't follow a regular pattern

---

## 21. Chapter 13 — Assets in Airflow

Assets (new in Airflow 3.x) enable **data-driven dependencies** instead of task-driven dependencies.

### Task Approach vs Asset Approach

| Aspect | Task Approach | Asset Approach |
|---|---|---|
| Trigger condition | Previous task completed | Data was produced/updated |
| Focus | Task execution status | Data availability |
| Cross-DAG deps | Complex (trigger operators) | Native (asset dependencies) |
| Data validation | Manual | Built-in support |

### Why Assets?

Consider this scenario:
- **DAG A** extracts data from an API → writes to `/data/extract.txt`
- **DAG B** processes data from `/data/extract.txt`

**Task approach problem**: If DAG A fails but another process populates the file, DAG B won't run because it depends on DAG A's task status.

**Asset approach**: DAG B depends on the *file being updated*, not on DAG A's task status. Any process that updates the file triggers DAG B.

### Creating a Producer Asset

### File: `dags/13_asset.py`

```python
from airflow.sdk import dag, task, Asset
import os

@Asset(
    schedule="@daily",
    uri="file:///opt/airflow/logs/data/data_extract.txt",
    name="fetch_data"
)
def fetch_data(self):
    """Producer asset — writes data to a file"""
    # Create directory if it doesn't exist
    os.makedirs(os.path.dirname(self.uri.replace("file://", "")), exist_ok=True)

    file_path = self.uri.replace("file://", "")
    with open(file_path, "w") as f:
        f.write("Data fetched successfully")

    print(f"Data written to {file_path}")
```

### Creating a Consumer (Dependent) Asset

### File: `dags/14_asset_dependent.py`

```python
from airflow.sdk import dag, task, Asset
from dags.asset_13 import fetch_data  # Import the producer asset
import os

@Asset(
    schedule=fetch_data,  # ← Triggered when fetch_data is materialized!
    uri="file:///opt/airflow/logs/data/data_processed.txt",
    name="process_data"
)
def process_data(self):
    """Consumer asset — triggered when fetch_data completes"""
    file_path = self.uri.replace("file://", "")
    os.makedirs(os.path.dirname(file_path), exist_ok=True)

    with open(file_path, "w") as f:
        f.write("Data processed successfully")

    print(f"Processed data written to {file_path}")
```

### How Assets Work Under the Hood

An Asset is essentially a **DAG with a single task**, plus metadata about what data it produces:

```
┌───────────────── Asset "fetch_data" ─────────────────┐
│                                                       │
│  ┌─────────────┐                ┌─────────────────┐  │
│  │ DAG (auto)  │───────────────▶│ data_extract.txt │  │
│  │ Single task │  materializes  │ (URI target)     │  │
│  └─────────────┘                └─────────────────┘  │
└───────────────────────────────────────────────────────┘
         │
         │ When materialized, triggers:
         ▼
┌───────────────── Asset "process_data" ───────────────┐
│  ┌─────────────┐                ┌──────────────────┐ │
│  │ DAG (auto)  │───────────────▶│ data_processed   │ │
│  │ Single task │                │ (URI target)     │ │
│  └─────────────┘                └──────────────────┘ │
└──────────────────────────────────────────────────────┘
```

### Viewing Assets

Go to the Airflow UI → **Assets** tab to see all registered assets, their URIs, and dependency relationships.

### Materializing an Asset

"Materializing" = running the asset. Go to **DAGs** → find the asset's auto-generated DAG → **Trigger**.

When the producer is materialized, the consumer is **automatically triggered**.

---

## 22. Chapter 14 — DAG Orchestration (Inherited DAGs)

For orchestrating multiple DAGs from a parent DAG — useful when you need DAG A to complete before DAG B starts.

### Child DAGs

### File: `dags/dag_orchestrate_1.py`

```python
from airflow.sdk import dag, task
import os

@dag
def first_orchestrator_dag():

    @task.python
    def task_one():
        print("First orchestrator - Task 1")

    @task.python
    def task_two():
        print("First orchestrator - Task 2")

    @task.python
    def task_three():
        # Write a file to confirm execution
        os.makedirs("/opt/airflow/logs/data", exist_ok=True)
        with open("/opt/airflow/logs/data/output_1.txt", "w") as f:
            f.write("Output from DAG 1")
        print("First orchestrator - Task 3 (wrote file)")

    t1 = task_one()
    t2 = task_two()
    t3 = task_three()
    t1 >> t2 >> t3

first_orchestrator_dag()
```

### File: `dags/dag_orchestrate_2.py`

```python
from airflow.sdk import dag, task
import os

@dag
def second_orchestrator_dag():

    @task.python
    def task_one():
        print("Second orchestrator - Task 1")

    @task.python
    def task_two():
        os.makedirs("/opt/airflow/logs/data", exist_ok=True)
        with open("/opt/airflow/logs/data/output_2.txt", "w") as f:
            f.write("Output from DAG 2")
        print("Second orchestrator - Task 2 (wrote file)")

    t1 = task_one()
    t2 = task_two()
    t1 >> t2

second_orchestrator_dag()
```

### Parent DAG (Orchestrator)

### File: `dags/dag_orchestrate_parent.py`

```python
from airflow.sdk import dag, task
from airflow.operators.trigger_dagrun import TriggerDagRunOperator

@dag
def dag_orchestrate_parent():

    trigger_first = TriggerDagRunOperator(
        task_id="trigger_first_dag",
        trigger_dag_id="first_orchestrator_dag",
        wait_for_completion=True,  # Wait until child finishes
    )

    trigger_second = TriggerDagRunOperator(
        task_id="trigger_second_dag",
        trigger_dag_id="second_orchestrator_dag",
        wait_for_completion=True,
    )

    trigger_first >> trigger_second

dag_orchestrate_parent()
```

### Key: TriggerDagRunOperator

| Parameter | Description |
|---|---|
| `task_id` | Name of this task in the parent DAG |
| `trigger_dag_id` | The DAG ID of the child DAG to trigger |
| `wait_for_completion` | If `True`, parent waits for child to finish before proceeding. **Recommended for sequential orchestration.** |

### Important Notes

1. **Child DAGs must be enabled** (unpaused) for the trigger to work. Otherwise, the run will be queued but not executed.
2. `wait_for_completion=True` makes execution **synchronous** — safer for sequential pipelines but slower.
3. Without `wait_for_completion`, child DAGs run **asynchronously** — faster but may cause race conditions.

---

## 23. Shutting Down & Restarting Airflow

### Stop All Containers
```bash
docker compose down
```
This stops and removes all containers. Your data in `dags/`, `logs/`, etc. is preserved (it's on your local machine).

### Restart
```bash
docker compose up -d
```
Containers restart within seconds (images are already downloaded).

### Check Status
```bash
docker ps
```

---

## 24. Quick Reference Cheat Sheet

### Imports

```python
from airflow.sdk import dag, task, Asset
from pendulum import datetime, duration
from airflow.timetables.trigger import CronTriggerTimetable, DeltaTriggerTimetable
from airflow.timetables.interval import CronDataIntervalTimetable
from airflow.timetables.events import EventsTimetable
from airflow.operators.bash import BashOperator
from airflow.operators.trigger_dagrun import TriggerDagRunOperator
```

### DAG Decorator Parameters

```python
@dag(
    dag_id="my_dag",                              # Optional (defaults to function name)
    schedule="@daily",                             # Preset, cron, timetable, or asset
    start_date=datetime(2026, 1, 1, tz="UTC"),     # Required
    end_date=datetime(2026, 12, 31, tz="UTC"),     # Optional
    catchup=False,                                 # Backfill missed intervals
    is_paused_upon_creation=True,                  # Auto-enable on deploy
)
```

### Task Types

```python
@task.python       # Python function
@task.bash         # Bash command (return the command string)
@task.branch       # Branch/decision node (return task_id)
```

### Dependencies

```python
# Sequential
a >> b >> c

# Parallel
a >> [b, c, d] >> e

# Branch
decide >> [option_a, option_b]
```

### XCom Push/Pull

```python
# Push
ti = kwargs["ti"]
ti.xcom_push(key="my_key", value={"data": [1, 2, 3]})

# Pull
data = ti.xcom_pull(task_ids="source_task", key="my_key")
```

### Docker Commands

```bash
docker compose up -d       # Start Airflow
docker compose down        # Stop Airflow
docker ps                  # Check running containers
docker exec -it <name> bash  # Enter a container
```

### Useful Airflow CLI (inside container)

```bash
airflow assets list        # List all assets
airflow dags list          # List all DAGs
airflow tasks list <dag_id>  # List tasks in a DAG
```

---

## 📚 Additional Resources

- [Official Apache Airflow Documentation](https://airflow.apache.org/docs/)
- [Airflow Docker Compose](https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html)
- [Cron Expression Tool](https://crontab.guru/)
- [Manning: Data Pipelines with Apache Airflow (2nd Edition)](https://www.manning.com/) — Latest architecture reference
- [Astronomer](https://www.astronomer.io/) — Managed Airflow platform & learning resources

---

## 📝 License

This guide is free to use for learning purposes. Share it with your fellow data engineers!

---

> *"It's not just about learning the code — it's about understanding the WHY behind every concept. When you have deep knowledge, interviews and certifications become a very small thing."*
