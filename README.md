# Automated Data Pipelines with Snowflake Tasks

This repository explains the core concepts and SQL examples used to build automated, scheduled data pipelines in Snowflake using **Tasks**.  
Snowflake Tasks allow you to schedule SQL statements, orchestrate dependencies, and create scalable, reliable data workflows.

---

## 💡 Core Concepts: Automation and Dependencies

### 1. Scheduled Tasks
A **scheduled task** is the entry point of an automated workflow.  
It runs according to a defined schedule, typically using a CRON expression.

### 2. Dependent Tasks (DAGs)
For more complex workflows, tasks can be chained into a **Directed Acyclic Graph (DAG)**.  
Child tasks execute **only after** their parent task completes successfully, ensuring proper sequencing of data loads and transformations.

---

## 🛠️ Task Creation SQL

## A. Parent Scheduled Task – `WEATHERTASK`

This task executes on a schedule and loads JSON weather data from an external stage (`@weather_stage`) into the `WEATHER` table.

### Key Components

| Component    | Purpose |
|-------------|---------|
| `WAREHOUSE` | Compute resource used to run the task |
| `SCHEDULE`  | CRON expression defining execution frequency |
| `COPY INTO` | Performs the actual data load |
| `t.$1:field` | Extracts JSON fields from the VARIANT column |

### SQL
```sql
CREATE TASK DEMO.DEMO_SCHEMA.WEATHERTASK
  WAREHOUSE = COMPUTE_WH
  SCHEDULE = 'USING CRON 0 0 * * * UTC'
AS
COPY INTO DEMO.DEMO_SCHEMA.WEATHER
FROM (
    SELECT
        t.$1:city:findname,
        t.$1:city:coord:lat,
        t.$1:city:coord:lon,
        t.$1:clouds:all,
        t.$1:main:humidity,
        t.$1:main:pressure,
        t.$1:main:temp,
        t.$1:time,
        t.$1:weather[0]:main
    FROM @DEMO.DEMO_SCHEMA.weather_stage t
);
