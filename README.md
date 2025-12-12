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
#B. Child Dependent Task – BIKETASK

This task executes only after WEATHERTASK succeeds, forming a two-step pipeline.

###Key Components
Component	Purpose
AFTER	Defines parent–child dependency
t.$1, t.$2...	Positional references for CSV files
CREATE TASK DEMO.DEMO_SCHEMA.BIKETASK
  WAREHOUSE = COMPUTE_WH
  AFTER DEMO.DEMO_SCHEMA.WEATHERTASK
AS
COPY INTO DEMO.DEMO_SCHEMA.BIKE
FROM (
    SELECT
        t.$1,
        t.$2,
        t.$3,
        t.$4,
        t.$5,
        t.$6,
        t.$7,
        t.$8,
        t.$9,
        t.$10,
        t.$11,
        t.$12,
        t.$13
    FROM @DEMO_SCHEMA.BIKE_STAGE t
);
#⚙️ Task Management Commands

Use these commands to control and monitor your pipeline.

###Action	Command	Description
Resume Task	ALTER TASK DEMO.DEMO_SCHEMA.BIKETASK RESUME;	Reactivates a suspended task
Show All Tasks	SHOW TASKS;	Lists tasks and their status
Manual Trigger	EXECUTE TASK DEMO.DEMO_SCHEMA.WEATHERTASK;	Immediately runs the task
View History	SELECT * FROM TABLE(INFORMATION_SCHEMA.TASK_HISTORY()) ORDER BY SCHEDULED_TIME;	Shows execution history

#📂 Repository Structure
/sql
  ├── weather_task.sql
  ├── bike_task.sql
  └── task_management.sql

/diagrams
  └── pipeline_dag.mmd

README.md

#📘 Prerequisites

Snowflake Account

Appropriate roles (e.g., SYSADMIN, ACCOUNTADMIN)

Warehouse (COMPUTE_WH)

Stages:

@weather_stage

@BIKE_STAGE

#🔄 Pipeline Execution Flow

Snowflake triggers WEATHERTASK based on its CRON schedule

If successful, Snowflake automatically triggers BIKETASK

Both tasks record execution history

Pipeline continues daily without manual intervention

#📈 Future Enhancements

Add error notification via Snowflake Alerts

Add more task layers (3+ step DAG)

Add dbt integration

Add S3 to Snowflake ETL patterns

#📞 Contact

If you'd like help extending this pipeline or automating more Snowflake workflows, feel free to reach out.


---

