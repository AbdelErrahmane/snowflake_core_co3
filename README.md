# Account & User Management — Snowflake COF-C03 Quick Revision

## Dropping an Account

### Grace period
- An account can be dropped with a **grace period from 3 to 90 days**
- During the grace period:
  - the account can be restored
  - you **cannot create another account with the same name**

### Best practice
Before dropping an account:
- **rename the account first**
- then drop it

Reason:
- avoids name conflict during the grace period

### Important restriction
Using `ORGADMIN`:
- you **cannot drop the account you are currently connected to**

So:
- log in to **another ORGADMIN-enabled account**
- then drop the target account

If your organization has only one account:
- you need to **contact Snowflake Support**

### Syntax
`DROP ACCOUNT my_account GRACE_PERIOD_IN_DAYS = 14;`

### Undrop concept
- an account can be restored during the grace period
- after the grace period, permanent deletion applies

---

## Before Dropping an Account with Listings / Shares

If the account provides listings, reader accounts, or shares:

You must first:
1. delete the listings
2. drop the shares associated with those listings

---

## Account URL Formats

### Standard URL formats
- Account name:
  `https://<orgname>-<account_name>.snowflakecomputing.com`

- Connection name:
  `https://<orgname>-<connectionname>.snowflakecomputing.com`

- Legacy account locator:
  `https://<accountlocator>.<region>.<cloud>.snowflakecomputing.com`

---

## Okta URL Important Rule

Okta does **not support underscores** in URLs.

If the Snowflake account name contains `_`:
- convert `_` to `-` in the **Okta account URL**

Example:
- Snowflake account name may be:
  `my_account`
- Okta URL must use:
  `my-account`

### Important certification idea
- avoid underscores in account names when possible
- Okta compatibility is a key reason

---

## Private Connectivity URL

When using private connectivity such as AWS PrivateLink:

- append `privatelink` to the account URL

Format:
`https://<orgname>-<account_name>.privatelink.snowflakecomputing.com`

### Exam keyword
- **privatelink** must appear in the URL

---

## Account Naming Best Practices

- do **not** start with `_`
- do **not** end with `_`
- avoid `_` in general because Okta does not support it in URLs
- account name should not exceed **63 characters**

---

## Organization and Account Uniqueness

- **Organization name** must be unique in Snowflake
- **Account name** must be unique inside the organization

### Useful command
`SELECT CURRENT_ORGANIZATION_NAME() || '-' || CURRENT_ACCOUNT_NAME();`

Used to display:
- current organization name
- current account name

---

## Parameter Management

Snowflake provides 3 main parameter levels:

### 1. Account parameters
- affect the entire account

### 2. Session parameters
- affect users and their sessions

### 3. Object parameters
- affect objects such as:
  - warehouses
  - databases
  - schemas
  - tables

### Show account parameters
`SHOW PARAMETERS IN ACCOUNT`

or with pattern:
`SHOW PARAMETERS LIKE '<pattern>' IN ACCOUNT`

### Set an account parameter
`ALTER ACCOUNT SET <param> = <value>;`

### Reset an account parameter
`ALTER ACCOUNT UNSET <param>;`

---

## ACCOUNTADMIN Best Practices

- strictly control assignment of `ACCOUNTADMIN`
- recommended: assign it to **at least two users**
- `ACCOUNTADMIN` should **not** be a user’s default role
- use a lower admin role or custom role as default role instead

### Certification idea
`ACCOUNTADMIN` is powerful and should be tightly controlled

---

## User Creation and Modification

### Role commonly used
- `USERADMIN` can create and manage users

### Required privileges for a custom role
If using a custom role instead of USERADMIN, it needs:
- `CREATE USER`
- `ALTER USER`

---

## User Locking

After **5 failed login attempts**:
- the user is locked for **15 minutes**

To unlock immediately:
`ALTER USER janesmith SET MINS_TO_UNLOCK = 0;`

### Certification point
- failed login attempts can temporarily lock the user
- admin can manually unlock before 15 minutes ends

---

## User Session Parameters

### Show session parameters for a user
`SHOW PARAMETERS FOR USER <name>;`

or:
`SHOW PARAMETERS LIKE '<pattern>' FOR USER <name>;`

### Change session parameters for a user
`ALTER USER <name> SET <session_param> = <value>;`

---

## Viewing User Information

Use:
- `DESCRIBE USER`
- `SHOW USERS`

---

## What Happens When a User Is Dropped

When a user is dropped:

### Becomes inaccessible
- folders
- worksheets
- dashboards owned by that user

These do **not transfer automatically** unless sharing is enabled.

### Not dropped automatically
Objects created by that user are **not dropped**, because they are owned by the **active role** used when the objects were created.

That means:
- another user with the same role
- or a higher role in the hierarchy

can:
- manage the objects
- transfer ownership

---

## Certification Memory Sentences

### Drop account
You cannot drop the account you are currently connected to with ORGADMIN.

### Grace period
Dropped accounts have a grace period of 3 to 90 days, and the same account name cannot be reused during that period.

### Account naming
Avoid underscores in account names because Okta does not support them in account URLs.

### Parameters
Snowflake parameters exist at account, session, and object level.

### User lock
After 5 failed attempts, a user is locked for 15 minutes unless manually unlocked.

### Dropped user
Dropping a user does not drop tables or views they created, because object ownership is tied to the active role, not the user identity.

**#####################################################################################""**

# Data Governance — Data Quality Monitoring (Snowflake COF-C03 Quick Revision)

## Feature Requirement
- Data Quality Monitoring requires **Enterprise Edition or higher**
- Data profiling uses the **user’s default warehouse** (can be changed in Snowsight)

---

## Core Concepts

### Data Metric Function (DMF)
- Measures a **data attribute**
- Examples:
  - number of NULL values
  - table freshness
  - duplicate rows
- Returns a value only → **does not decide pass/fail**

### Expectation
- Combined with a DMF to create a **data quality check**
- DMF result is compared to expectation
- If condition fails → **expectation violation**

### Anomaly Detection
- Uses historical DMF results
- Detects abnormal patterns automatically
- Currently detects anomalies in:
  - data volume
  - data freshness

### DMF Schedule
- Defines how often a DMF runs
- Default frequency = **once per hour**

---

## Supported Objects for DMF
You can associate DMFs with:
- Table (including temporary and transient)
- View
- Materialized view
- Dynamic table
- Event table
- External table
- Apache Iceberg table

---

## DMF Limitations
- Maximum **10,000 DMF associations per account**
- Cannot grant DMF privileges to a **data share**
- Cannot set DMF on **shared objects**
- Cannot set DMF on **object tags**
- Not supported in **reader accounts**
- Not available in **trial accounts**

---

## Cortex Data Quality (AI-based checks)
- Uses AI to **suggest quality checks automatically**
- Snowflake runs checks periodically after acceptance

Manual setup steps:
1. Open Snowsight
2. Go to Catalog → Database Explorer
3. Select object
4. Data Quality tab → Monitoring

---

## Anomaly Detection Training Rules
- Frequent DMF schedule → requires **2 weeks of history**
- Algorithm may train on **up to 60 days of data**
- Infrequent schedule → requires at least **2 data points**
- Sensitivity levels:
  - Low
  - Medium (default)
  - High

- Can disable anomaly detection using **ALTER statement**

---

## Where Results Are Stored
Event table:
SNOWFLAKE.LOCAL.DATA_QUALITY_MONITORING_RESULTS_RAW

View:
SNOWFLAKE.LOCAL.DATA_QUALITY_MONITORING_RESULTS

---

## Viewing Quality Monitoring in Snowsight
- DMFs grouped under **Quality Dimensions**
- Categories:
  - System DMFs (by category)
  - Custom DMFs

---

## Notifications Workflow

### Supported notification types
- Email
- External systems (e.g., Slack via webhook)

### Setup steps
1. Create notification integration
2. Grant privileges to database owner:
   - MANAGE DATA QUALITY (account level)
   - USAGE on notification integration
3. Configure database settings

Webhook setup:
- Create secret for webhook URL
- Create webhook notification integration

Disable notification example concept:
ALTER VIEW v2 MODIFY DATA METRIC FUNCTION ... SET DATA_QUALITY_NOTIFICATION = FALSE;

---

## Monitoring DMF Associations
Use:
DATA_METRIC_FUNCTION_REFERENCES

Important column:
- data_quality_notification_status

---

## Remediation of Data Quality Issues

Use system function:
SYSTEM$DATA_METRIC_SCAN

Purpose:
- Returns **records that failed a quality check**

Supported system DMFs as argument:
- ACCEPTED_VALUES
- BLANK_COUNT
- BLANK_PERCENT
- DUPLICATE_COUNT
- NULL_COUNT
- NULL_PERCENT

Limitations:
- Cannot use **custom DMFs**
- Policies (masking / row access) may affect returned results

---

## Certification Memory Sentences

- DMF measures data quality metrics but **expectation defines pass/fail**
- Default DMF schedule = **1 hour**
- Data Quality Monitoring requires **Enterprise Edition**
- Maximum **10,000 DMF associations per account**
- Results stored in **SNOWFLAKE.LOCAL event tables**
- SYSTEM$DATA_METRIC_SCAN helps identify problematic rows


===============================================

# Object Tagging — Snowflake Certification Summary (COF-C03)

## Definition
- A **tag is a schema-level object** used to classify or label Snowflake objects.
- Stored as **key-value pair**:
  - Tag name = key
  - Tag value = string

Example idea:
- tag = cost_center  
- value = sales

---

## Important Limits (Exam Focus)
- Maximum **50 tags across all columns in one table**
- Tag name must be **unique within a schema**
- Tag value is **always a string**

---

## Tag Assignment Concepts

- One object → can have **multiple tags**
- One tag → can be applied to **multiple objects**
- Tag value → can be duplicated across objects

Example concept:
- Many tables can have:
  - tag = cost_center
  - value = sales

---

## Governance Benefits

### Centralized classification
- Define tag once → reuse across many objects

### Data protection
- Tags can be combined with **masking policies**
- Enables **tag-based masking**

### Cost monitoring
- Tags help group resources by:
  - department
  - project
  - cost center

---

## Enterprise Edition Required Features
Requires **Enterprise Edition or higher**:

- Tag propagation
- Tag-based masking policies

---

## Tag Propagation (Very Important)

Tag propagation = automatic assignment of tag to related objects

### Propagation modes
- Object dependency propagation  
  Example:
  - View created from tagged table → tag propagated

- Data movement propagation  
  Example:
  - Data copied into another table → tag propagated

- Both modes can be enabled

### Conflict handling
- Controlled using **ON_CONFLICT property**

### Monitoring propagation
- Telemetry stored in **event tables**

---

## Supported Objects for Tagging

- Columns
- Tables (permanent / temporary / transient)
- Dynamic tables
- External tables
- Iceberg tables
- Views
- Secure views
- Materialized views

Important concept:
- Creating a **dynamic table** counts as:
  - object dependency  
  - data movement

---

## Tag Inheritance and Replication

### Cloning behavior
- Tags are **preserved in cloned objects**

### Replication behavior
- Tags and tag assignments **replicate from source account**
- Cannot modify tag assignments in **target account**
- Must modify in source → then replicate

Replication failure cases (Exam trap):
- Target account edition lower than Enterprise
- Object references a tag located in another database (dangling reference)

---

## Tag Metadata and Monitoring

Useful views and functions:

- ACCOUNT_USAGE.TAG_REFERENCES
- ACCOUNT_USAGE.TAG_REFERENCES_WITH_LINEAGE
- INFORMATION_SCHEMA.TAG_REFERENCES
- INFORMATION_SCHEMA.TAG_REFERENCES_ALL_COLUMNS

Purpose:
- Understand **how and where tags are applied**

---

## Tag Limitations

- No future grants supported on tags
- Cannot set tag on shared objects in some contexts
- Some restrictions exist with **Snowflake Native Apps**

---

## Certification Memory Sentences

- Tag = schema object used for **classification and governance**
- Max **50 tags per table columns combined**
- Tag value is always **string**
- Tag propagation requires **Enterprise Edition**
- Tags can drive **masking policies and cost tracking**
- Tags replicate but **cannot be modified in secondary account**


# ===========



# Virtual Warehouses — Snowflake Certification Summary (COF-C03)

## Definition
- A **virtual warehouse (WH)** = cluster of compute resources (CPU, memory, temp storage).
- Used to execute:
  - SELECT queries
  - DML (INSERT / UPDATE / DELETE)
  - Data load → COPY INTO table
  - Data unload → COPY INTO location

---

## Warehouse Types (Exam)
- **Standard warehouse**
- **Snowpark-optimized warehouse** (for heavy Python / Snowpark workloads)

---

## Billing Concept (Very Important)
- **Per-second billing**
- **Minimum charge = 60 seconds each time warehouse starts**
- Credits depend on:
  - warehouse size
  - number of clusters
  - running time

👉 No benefit stopping warehouse before first 60 seconds.

---

## Warehouse Size Impact

### On data loading
- Performance depends more on:
  - number of files
  - file size  
NOT mainly warehouse size.

### On query processing
- Larger warehouse → more compute → faster queries (especially complex queries)

---

## Auto Management Defaults (Exam Trap)
- **Auto-suspend = ON (default)**
- **Auto-resume = ON (default)**

Warehouse suspends after inactivity → resumes automatically when query arrives.

---

## Warehouse and Sessions
- Session starts **without warehouse by default**
- Queries cannot run until warehouse is set.

Solutions:
- Assign **default warehouse to user**
- Or specify warehouse in connection parameters.

---

## Default Warehouse for Notebooks (Important)
- Snowflake auto-creates:

SYSTEM$STREAMLIT_NOTEBOOK_WH

Characteristics:
- Multi-cluster X-Small
- Max clusters = 10
- Timeout = 60 seconds
- OWNER = ACCOUNTADMIN

Best practice:
- Use it **only for notebook workloads**
- Separate Python notebook compute from SQL workloads.

---

## Query Concurrency Concept
- Warehouse reserves compute for each query.
- If not enough resources → **query queued**.

Useful parameters:
- STATEMENT_QUEUED_TIMEOUT_IN_SECONDS
- STATEMENT_TIMEOUT_IN_SECONDS

---

## Multi-Cluster Warehouse (Enterprise Feature)

Purpose:
- Scale compute automatically for concurrency peaks.

### Scaling modes

**Maximized mode**
- Min clusters = Max clusters
- All clusters always running

**Auto-scale mode (Recommended)**
- Snowflake automatically starts / stops clusters

Scaling policy options:
- Standard → minimize queuing (spend more credits)
- Economy → minimize credits (more query queuing)

---

## Cluster Limits (Important Concept)
- XSMALL / SMALL / MEDIUM:
  - Default max clusters = 10
  - Upper max limit = 300

Larger sizes → smaller max limits.

---

## Scale Up vs Scale Out (Exam Question)

- **Scale up** → resize warehouse (more power per cluster)
- **Scale out** → add clusters (more concurrency)

Enterprise Edition required for scaling out.

Important:
- Resizing running warehouse **does NOT affect running queries**
- New resources used only for queued / new queries.

---

## Credit Consumption Formula
Max hourly credits =
warehouse size × max cluster count

---

## Warehouse Caching Concept
- Larger warehouse → larger cache
- Cache lost when warehouse suspends

Trade-off:
- Suspend → save credits
- Keep running → reuse cache → better performance

---

## Query Performance Factors
More important than row count:
- table size
- joins count
- filters (predicates)

---

## Creating Warehouse — Key Decision Factors
Most important for cost & performance:

1. Warehouse size
2. Auto vs manual suspend/resume

Guidelines:
- Small environments → XS / S / M
- Production analytics → L / XL / 2XL+

---

## Snowsight Warehouse Usage
Some UI pages need warehouse to run queries.

Example:
- Task history
- Data preview

Recommendation:
- X-Small usually enough.

Important:
- Some commands (ex: SHOW TABLES) **do not consume credits**.

---

## Generation 2 Warehouses (New Concept)

Gen2 = faster hardware + smarter optimizations:
- better DELETE / UPDATE / MERGE
- improved table scans

Options to enable:

GENERATION = '2'

or

RESOURCE_CONSTRAINT = STANDARD_GEN_2

Gen2 default for new orgs in many regions after mid-2025.

---

## Certification Memory Sentences

- Warehouse = compute layer in Snowflake
- Billing = per-second with **60-sec minimum**
- Auto-suspend & auto-resume = default ON
- Scale up = resize warehouse
- Scale out = add clusters (Enterprise)
- Multi-cluster reduces query queuing
- Cache lost when warehouse suspends
- Default notebook warehouse exists automatically
- Gen2 warehouses = next generation performance


# ===================
# Query Acceleration Service (QAS) — Snowflake Certification Summary (COF-C03)

## Definition
- **Query Acceleration Service (QAS)** = serverless compute feature that accelerates parts of queries.
- Helps reduce impact of **outlier queries** (queries using unusually high resources).
- Improves overall warehouse performance.

---

## SQL operations that QAS can accelerate (Exam)
- SELECT
- INSERT
- CREATE TABLE AS SELECT (CTAS)
- COPY INTO <table>

---

## How to Enable QAS
Use warehouse parameter:

ENABLE_QUERY_ACCELERATION = TRUE

Example concept:
CREATE WAREHOUSE my_wh ENABLE_QUERY_ACCELERATION = TRUE;

Important:
- QAS may **increase credit consumption rate**.

---

## How QAS Works (Concept)
- Offloads eligible parts of query plan to **serverless compute resources**
- Works mainly on **large table scans**
- Parallel processing → faster execution

---

## Eligible Query Patterns (Very Important)
Queries typically eligible when they include:

1. Large scans + aggregation or selective filter
2. Large scans inserting or copying many rows

Eligibility depends on:
- warehouse size
- query plan
- partition count

---

## Common Reasons Queries Are NOT Eligible
- Not enough partitions
- Filters not selective
- LIMIT clause (sometimes)
- Non-deterministic functions (e.g., RANDOM, SEQ)

---

## How to Identify Eligible Queries

### View
QUERY_ACCELERATION_ELIGIBLE

### Function
SYSTEM$ESTIMATE_QUERY_ACCELERATION(query_id)

Example concept:
Returns JSON indicating:
- eligible
- ineligibleReason

Example reason:
NO_LARGE_ENOUGH_SCAN

---

## Scale Factor (Cost Control Concept)
- Defines **maximum additional compute QAS can use**
- Multiplier based on warehouse size

Example:
Medium warehouse = 4 credits/hour

Scale factor = 5 →
QAS can consume up to **20 extra credits/hour**

Important:
- Applies to entire warehouse
- For multi-cluster WH → consider increasing scale factor

---

## Monitoring QAS Usage

### Query Profile Stats
- Partitions scanned by service
- Scans selected for acceleration

### ACCOUNT_USAGE View Columns
- QUERY_ACCELERATION_BYTES_SCANNED
- QUERY_ACCELERATION_PARTITIONS_SCANNED
- QUERY_ACCELERATION_UPPER_LIMIT_SCALE_FACTOR

---

## QAS Cost Model (Exam)
- Serverless billing
- Charged per-second
- Separate from warehouse compute credits

---

## Monitoring Warehouse Load

Query load states:
- Running
- Queued (Provisioning)
- Blocked
- Queued

Query load calculation:
Total query execution time ÷ interval duration

---

## Troubleshooting Slow Queries

If query load high:
- Add clusters (multi-cluster WH)
- Move workload to separate warehouse

If query load low but query slow:
- Resize warehouse
- Restart query after resize

---

## Peak Workload Optimization
- Analyze workload patterns over last 2 weeks
- Possible optimizations:
  - Separate peak workloads to another warehouse
  - Adjust multi-cluster configuration
  - Reduce warehouse size (cost saving)
  - Decrease MIN_CLUSTER_COUNT

---

## Certification Memory Sentences

- QAS = serverless acceleration for large scan queries
- Enable using ENABLE_QUERY_ACCELERATION parameter
- Helps with outlier query performance
- Uses scale factor to control cost
- Eligible queries = large scans with aggregation or heavy inserts
- Monitor via QUERY_HISTORY and query profile
- Billed per-second like other serverless features


**======**
# Snowpark-Optimized Warehouses & Interactive Warehouses — Certification Summary

## Snowpark-Optimized Warehouses — Definition
- A **Snowpark-optimized warehouse** is a special type of virtual warehouse designed to provide **higher memory per node and configurable CPU architecture**.
- Optimized for **single-node compute workloads with large memory requirements**.

---

## When to Use Snowpark-Optimized Warehouses (Exam Concept)
Recommended for:
- Snowpark workloads requiring **large memory**
- Machine Learning model training in stored procedures
- Heavy Python workloads using UDF / UDTF
- Workloads dependent on specific CPU architecture

Important:
- Non-Snowpark workloads usually **do not benefit** from Snowpark-optimized warehouses.

---

## Configuration Options
- Default memory configuration = **MEMORY_16X**
- Provides **16× more memory per node** compared to standard warehouses
- Additional memory sizes available using:
  - RESOURCE_CONSTRAINT property

Example concept:
CREATE WAREHOUSE wh_sp
RESOURCE_CONSTRAINT = MEMORY_32X;

---

## High-Memory Preview Feature
- MEMORY_64X and MEMORY_64X_x86 → up to **1 TB memory per node**
- Available as preview
- Supported **only on AWS**

---

## Interactive Warehouses & Interactive Tables — Overview
Feature available only in **selected regions (AWS / Azure / GCP)**.

These introduce new optimized objects:
- Interactive warehouse
- Interactive table

Goal:
- Provide **very low-latency query performance**.

---

## Interactive Warehouse — Definition
- Specialized warehouse optimized for **interactive workloads**
- Designed for:
  - real-time dashboards
  - APIs
  - high-concurrency analytics

Key behavior:
- Always running → **no auto-suspend**
- Cannot query standard Snowflake tables

---

## Interactive Table — Definition
- Specialized table optimized for **fast simple queries**
- Best performance when used with interactive warehouses

Typical query pattern:
- SELECT with selective WHERE
- Small GROUP BY
- Avoid large joins or complex subqueries

---

## Use Cases (Exam)
- Real-time analytics dashboards
- Data-driven APIs
- Applications requiring consistent low-latency responses

---

## Limitations of Interactive Warehouses (Very Important)
- Do not support long-running queries
- Always running (no auto-suspend)
- Cannot access standard tables
- Multi-cluster mode → no auto-scale
  - MIN_CLUSTER_COUNT = MAX_CLUSTER_COUNT required
- Cannot run CALL stored procedures
- Cannot use pipe operator ->>
- Not supported in replication / failover groups

---

## Limitations of Interactive Tables
- UPDATE and DELETE not supported
- Only INSERT OVERWRITE allowed
- No Fail-safe (Time Travel still supported)
- Query insights not available
- Cannot be source of materialized view
- Cannot modify structure using ALTER TABLE ADD/DROP COLUMN

---

## Certification Memory Sentences
- Snowpark-optimized warehouses = high-memory single-node compute
- Default memory = MEMORY_16X
- Interactive warehouses = always-on low-latency compute
- Interactive tables optimized for simple fast SELECT queries
- Interactive workloads → dashboards, APIs, real-time analytics
- Interactive warehouses cannot query standard tables


# Micro-partitions & Data Clustering — Certification Summary

## Micro-partitions — Definition
- Snowflake automatically divides table data into **micro-partitions**.
- Each micro-partition contains **50 MB to 500 MB of uncompressed data**.
- Data is always **stored compressed and columnar**.

Key idea (exam):
👉 Micro-partitioning is **automatic** (no manual partition definition required).

---

## Metadata Stored in Micro-partitions
Snowflake stores metadata such as:
- Minimum and maximum values per column
- Number of distinct values
- Other optimization statistics

👉 This metadata enables **query pruning and performance optimization**.

---

## Benefits of Micro-partitioning
- Automatic partition management
- Fine-grained pruning → faster queries
- Columnar storage → scan only required columns
- Independent column compression
- Small partition size → avoids data skew
- Efficient DML operations

Certification memory:
👉 Micro-partitions enable **automatic partitioning + pruning + compression.**

---

## Impact on DML Operations
- DELETE / UPDATE / MERGE use partition metadata for efficiency
- Some operations can be **metadata-only** (e.g., deleting all rows)

Important:
👉 Dropping a column does **not rewrite micro-partitions**.  
Data remains in storage.

---

## Query Pruning (Very Important Exam Concept)
Snowflake uses metadata to:
1. Eliminate unnecessary micro-partitions
2. Scan only required columns

Example idea:
👉 If a filter accesses 10% of data → ideally only 10% of partitions scanned.

---

## Data Clustering — Definition
- Clustering organizes micro-partitions based on column value ranges.
- Improves query performance by reducing scanned partitions.

Snowflake query process:
1. Prune unnecessary micro-partitions
2. Then prune by column inside partitions

---

## Clustering Metadata Maintained
Snowflake tracks:
- Total number of micro-partitions
- Overlapping partitions
- Clustering depth

---

## Clustering Depth (Exam Topic)
- Measures overlap between micro-partitions
- Range:
  - 0 → empty table
  - 1 → ideal clustering
  - Higher value → poor clustering

👉 Lower clustering depth = better performance.

---

## Monitoring Clustering
Use system functions:
- SYSTEM$CLUSTERING_DEPTH
- SYSTEM$CLUSTERING_INFORMATION

---

## Clustering Keys & Clustered Tables
- You can define clustering keys on columns or expressions.
- Snowflake automatically reorganizes partitions.

Supported on:
- Tables
- Materialized views

---

## When to Use Clustering Keys
Recommended when:
- Very fast query response is required
- Performance benefit justifies compute cost

Not recommended for:
- Small tables
- Low query filtering workloads

---

## Certification Memory Sentences
- Micro-partitioning is automatic and columnar.
- Each micro-partition = 50–500 MB (uncompressed).
- Query pruning uses partition metadata.
- Clustering reduces partition overlap.
- Lower clustering depth = better clustering.
- Clustering keys improve performance but increase cost.

## ===========

# Clustering Key — Certification Summary

## What is a Clustering Key
- A clustering key is a **subset of columns or expressions** used to co-locate related rows in the same micro-partitions.
- Useful when:
  - Natural clustering degraded due to heavy DML
  - Data load ordering was not optimal
  - Query performance becomes slower

Certification memory:
👉 Clustering key improves **micro-partition pruning for very large tables.**

---

## Indicators That Clustering Is Needed
- Queries are slower than expected
- Performance degraded over time
- Clustering depth is high

---

## Benefits of Clustering Keys
For very large tables:
- Improved query scan efficiency (better pruning)
- Better column compression (especially correlated columns)
- Automatic maintenance after key definition

Important:
👉 Snowflake performs **automatic reclustering**.

---

## When Queries Benefit from Clustering
- Queries filter on clustering columns
- Queries sort on clustering columns:
  - ORDER BY
  - GROUP BY
  - Some joins

---

## When to Use Clustering (Best Conditions)
Clustering is recommended when:

- Table is very large (typically TB scale)
- Queries are selective (read small % of rows)
- Queries sort data
- Many queries use the same filtering columns

Snowflake recommendation:
👉 Always test queries before enabling clustering.

---

## Strategy for Choosing Clustering Keys
Best practices:

1. Choose columns used in **selective filters**
   - Example: date column in fact tables
2. Then choose columns used in **join predicates**

---

## Cardinality Rule (Very Important)
Good clustering key must have:
- Enough distinct values → enable pruning
- Not too many distinct values → allow grouping

High-cardinality tip:
👉 Use an **expression** (e.g., truncated date) instead of raw column.

---

## Multi-Column Clustering Key Rule
- Recommended maximum: **3–4 columns**
- Column order matters:
  👉 Order from **lowest cardinality → highest cardinality**

---

## Reclustering Concept
- DML operations reduce clustering quality
- Snowflake automatically reclusters data
- Reclustering reorganizes rows into new micro-partitions

Important exam idea:
👉 Reclustering is a **DML operation consuming credits.**

---

## Storage Impact of Reclustering
- New micro-partitions are created
- Old partitions remain for:
  - Time Travel
  - Fail-safe

Retention impact:
👉 Storage cost increases until retention period expires.

---

## Monitoring Clustering
Use system functions:

- SYSTEM$CLUSTERING_INFORMATION
- SYSTEM$CLUSTERING_DEPTH

Works even if table has no clustering key.

---

## Creating a Clustering Key
Defined at table creation:

CREATE TABLE t1 (...) CLUSTER BY (col1, col2);

Supported:
- Base columns
- Expressions
- VARIANT path expressions

Not supported types:
- GEOGRAPHY
- VARIANT (direct)
- OBJECT
- ARRAY

---

## Modifying Clustering Key
Change key:

ALTER TABLE t1 CLUSTER BY (col1, col2);

Drop key:

ALTER TABLE t1 DROP CLUSTERING KEY;

---

## VARCHAR Clustering Note (Exam Trap)
- Only first **5 bytes** used for clustering
- If prefix identical → use substring expression

Example idea:
👉 CLUSTER BY (SUBSTR(col, 5))

---

## Certification Memory Sentences
- Clustering key improves pruning for large tables.
- Use clustering when queries filter or sort on same columns.
- Reclustering is automatic but consumes credits.
- Lower clustering depth = better clustering.
- Keep clustering key ≤ 4 columns.
- Order clustering columns by increasing cardinality.


# ======================
# Automatic Clustering — Certification Summary

## Definition
Automatic Clustering is a **serverless Snowflake service** that automatically manages reclustering of clustered tables when beneficial.

Certification memory:
👉 Snowflake decides **when reclustering is needed.**

---

## Key Behavior (Exam Important)
- Reclustering does NOT start immediately after defining clustering key.
- Snowflake reclusters **only if performance benefit exists.**
- Automatic Clustering runs **in background.**

---

## Benefits of Automatic Clustering

### Ease of maintenance
- No need to monitor clustering depth manually
- No need to schedule reclustering jobs
- No need to assign warehouses

👉 Snowflake manages everything automatically.

### Full control
- You can suspend or resume reclustering anytime.

Suspend:
ALTER TABLE t1 SUSPEND RECLUSTER;

Resume:
ALTER TABLE t1 RESUME RECLUSTER;

While suspended:
👉 No reclustering → no credit consumption.

### Non-blocking DML
- Reclustering does NOT block:
  - INSERT
  - UPDATE
  - DELETE
  - MERGE

### Optimal efficiency
- Snowflake dynamically allocates internal resources
- Serverless compute ensures efficient clustering

---

## How to Enable Automatic Clustering
- Simply define a clustering key on table.

Important exam trap:
👉 Tables created using CLONE start with **automatic clustering suspended** even if source table was enabled.

---

## Monitoring Automatic Clustering Status
Use:

SHOW TABLES;

Important columns:
- AUTO_CLUSTERING_ON → shows status
- CLUSTER_BY → shows clustering key

Also:
- TABLES view → CLUSTERING_KEY column

---

## Cost Model (Very Important)
Automatic Clustering generates:

### Compute cost
- Serverless compute used for:
  - Initial clustering
  - Maintenance after DML

👉 More frequent table changes → higher cost.

### Storage cost
- Usually minimal
- Can increase Fail-safe storage due to new micro-partitions

---

## Warehouse Requirement (Exam Concept)
- Automatic Clustering **does NOT use virtual warehouse**
- Uses Snowflake internal serverless compute

👉 Credits still billed based on actual usage.

---

## Estimating Automatic Clustering Cost
Use system function:

SYSTEM$ESTIMATE_AUTOMATIC_CLUSTERING_COSTS

Used to:
- Predict cost before enabling clustering
- Estimate cost when changing clustering key

---

## Certification Memory Sentences
- Automatic Clustering is serverless and background.
- No warehouse required for reclustering.
- Reclustering only happens if beneficial.
- Clone tables start with clustering suspended.
- More DML = higher clustering cost.
- Can suspend/resume reclustering anytime.

# Temporary and Transient Tables — Certification Summary

## Temporary Tables

### Definition
Temporary tables store **non-permanent session data** (for ETL steps, intermediate results, testing).

Certification memory:
👉 Temporary table exists **only during the session.**

After session ends:
- Table is automatically dropped
- Data is permanently deleted
- No Time Travel
- No Fail-safe
- Not recoverable

---

### Visibility
- Visible only inside the session that created the table
- Not accessible by other users or sessions

---

### Storage Cost
- Temporary table storage is billed while session is active
- Long sessions → unexpected storage cost

Best practice:
👉 Explicitly DROP temporary tables when finished.

---

### Naming Behavior (Exam Trap)
You can create:
- Temporary table and permanent table with same name in same schema

Important:
👉 Temporary table **takes precedence** in session  
→ Permanent table becomes hidden.

This may cause:
- Confusing DDL behavior
- Unexpected query results

---

### Create Temporary Table

CREATE TEMP TABLE my_temp_table (id NUMBER);

---

## Transient Tables

### Definition
Transient tables are **semi-permanent tables**
- Persist until explicitly dropped
- Accessible to authorized users
- Designed for short-term stored data

Certification memory:
👉 Transient table = permanent table **without Fail-safe.**

---

### Key Differences vs Permanent Tables
- No Fail-safe period
- Lower storage cost (no Fail-safe storage)
- Still supports Time Travel (depending retention settings)

---

### Storage Cost
- Storage billed like permanent tables
- No Fail-safe storage cost

---

### Zero-Copy Clone Behavior (Important)
If transient table is cloned from permanent table:
- Initially shares micro-partitions
- No extra storage used

After DML:
- New micro-partitions created for clone

Important exam concept:
👉 Shared partitions delay Fail-safe entry of original table bytes.

---

## Transient Databases and Schemas
- All tables inside transient schema are automatically transient
- All schemas inside transient database are transient

Example:

CREATE TRANSIENT TABLE my_transient_table (id NUMBER);

---

## Key Certification Comparison

Temporary table:
- Session-based
- Auto-deleted
- Not visible to others
- No recovery

Transient table:
- Persist until dropped
- Visible with privileges
- No Fail-safe
- Lower protection level

Permanent table:
- Persist until dropped
- Time Travel + Fail-safe
- Highest data protection

# ======================
# External Tables — Certification Summary

## Definition  
An **external table** allows Snowflake to query data stored **outside Snowflake** (in cloud storage) as if it were a normal table.

Certification memory:  
👉 External table = **schema-on-read access to data in external stage.**

Important:  
- Data is **not stored or managed by Snowflake**
- External stage is located in cloud storage (S3, Azure Blob, GCS)

---

## Key Characteristics (Exam Focus)

- External tables are **read-only**
- No DML operations allowed (no INSERT / UPDATE / DELETE)
- Can be used in **SELECT and JOIN**
- Can create **views** on external tables
- Performance usually slower than native tables

Performance improvement:  
👉 Use **materialized view** on external table (Enterprise Edition).

---

## Supported File Formats
External tables support formats supported by:

👉 COPY INTO command  
Exception:  
- XML is **not supported**

---

## Schema on Read Concept (Very Important)

External table structure is derived **when querying data**, not at load time.

All external tables include:

- `VALUE` → VARIANT column representing row data  
- `METADATA$FILENAME` → file name + path  
- `METADATA$FILE_ROW_NUMBER` → row number in file  

---

## Virtual Columns

You can define **virtual columns** using:

- VALUE column  
- Metadata pseudocolumns  

Purpose:
- Schema validation
- Strong typing
- Better query performance

---

## File Size Recommendations (Exam Tip)

To improve parallel scanning:

- Parquet files → **256–512 MB**
- Parquet row groups → **16–256 MB**
- Other formats → **16–256 MB**

---

## Partitioned External Tables (Important)

Partitioning improves performance by scanning only relevant files.

Partition columns:
- Derived from file path or filename
- Stored in external table metadata

Two approaches:

### Automatic partitions
- Defined using expressions on METADATA$FILENAME
- Added when table metadata is refreshed

### Manual partitions
- User controls which partitions are added or removed

---

## Metadata Refresh (Very Important)

External table metadata must be refreshed to detect new or removed files.

Refresh methods:

- Automatic refresh using **cloud event notifications**
- Manual refresh:

ALTER EXTERNAL TABLE my_ext_table REFRESH;

Refresh synchronizes:
- New files added
- Modified files updated
- Deleted files removed

---

## Billing Concepts

Costs include:

- Cloud services cost for manual refresh
- Snowpipe-like cost for automatic refresh notifications

You can monitor usage via:

- PIPE_USAGE_HISTORY view or function

---

## Query Behavior

External tables are queried like normal tables.

Important behaviors:
- Invalid UTF-8 records are skipped
- To avoid missing rows:
  - Use `REPLACE_INVALID_CHARACTERS = TRUE`

For Parquet optimization:
- Set `BINARY_AS_TEXT = FALSE`

---

## Result Cache (Exam Trap)

Query results on external tables persist **24 hours**.

Cache invalidated when:

- External table definition changes
- Metadata refresh occurs
- File set in storage changes

---

## Key Certification Comparison

External table:
- Data stored outside Snowflake
- Schema on read
- Read-only
- Requires metadata refresh

Internal table:
- Data stored inside Snowflake
- Schema on write
- Full DML supported
- Better performance


# ======================================

# Hybrid Tables — Certification Summary

## Definition  
A **hybrid table** is a Snowflake table type optimized for **low-latency transactional workloads** using a **row-based storage engine with indexing**.

Certification memory:  
👉 Hybrid table = **transactional + analytical workloads in Snowflake (Unistore concept).**

Availability:  
- Supported only in **AWS and Azure commercial regions**

---

## Key Characteristics (Exam Focus)

- Uses **row-store architecture** (not columnar like standard tables)
- Supports **index-based random reads and writes**
- Provides **row-level locking** for high concurrency
- Enforces:
  - **Primary key constraints**
  - **Unique constraints**
  - **Referential integrity constraints**
- Designed for **operational / transactional queries**

Important exam idea:  
👉 Hybrid tables enable **OLTP-like workloads inside Snowflake.**

---

## Typical Use Cases

Hybrid tables are useful for:

- Application metadata storage  
- High-concurrency ingestion state tracking  
- API-based serving of small datasets  
- Lightweight transactional systems  
- Operational workloads with frequent updates

Examples of benefiting queries:

- Random point lookup queries  
- Short running SELECT queries  
- High-concurrency INSERT / UPDATE / MERGE operations  

---

## Architecture Benefits

- Can **join hybrid tables with standard Snowflake tables**
- Supports **atomic transactions across hybrid and standard tables**
- Enables **mixed operational + analytical workloads**
- Snowflake governance and security features apply automatically
- No need for external transactional database

Certification concept:  
👉 Hybrid tables help implement **Unistore architecture** (unified OLTP + OLAP).

---

## Performance Characteristics

Hybrid tables:

- Faster for **short operational queries**
- Provide **low latency and high throughput**
- Better for **random access patterns**

Standard tables:

- Better for **large analytical queries**
- Use **columnar storage and high compression**

Important exam trap:  
👉 Hybrid tables usually have **larger storage footprint** than standard tables.

---

## When to Use Hybrid Tables

Choose hybrid tables when:

- You need **high-concurrency writes**
- Queries access **small subsets of rows**
- Workload requires **transactional consistency**
- You need **indexed access patterns**

Avoid hybrid tables when:

- Running heavy analytical workloads
- Querying large scans and aggregations
- Storage efficiency is critical

---

## Certification Memory Sentence

Hybrid Table =  
Row-based Snowflake table designed for **low-latency transactional workloads with indexing and high concurrency.**

# ==============================
# Search Optimization Service — Certification Summary

## Definition  
The **Search Optimization Service** is a Snowflake performance feature that improves the speed of **selective lookup and search queries** by creating a special search structure.

Certification memory:  
👉 Search Optimization = **faster point lookup queries using a search access path.**

Availability:  
- Requires **Enterprise Edition (or higher)**

---

## What It Improves (Exam Focus)

This service significantly improves performance for:

- Selective point lookup queries (returning few rows)
- Text search operations:
  - SEARCH function  
  - SEARCH_IP function  
  - LIKE / ILIKE / RLIKE / substring search  
- Queries on **semi-structured data**
  - VARIANT  
  - OBJECT  
  - ARRAY  
- Queries on **structured complex types**
  - ARRAY  
  - MAP  
- Some **geospatial queries** on GEOGRAPHY columns

Important certification idea:  
👉 Best for queries that search **specific values or small subsets of data.**

---

## How It Works

The service creates a persistent structure called a:

### Search Access Path

This structure:

- Tracks which column values exist in each micro-partition
- Allows Snowflake to **skip unnecessary micro-partitions**
- Reduces scan cost and query time

Certification concept:  
👉 Similar goal as clustering → **reduce data scanned**, but optimized for search predicates.

---

## When to Use

Use search optimization when:

- Queries frequently search specific values
- Workloads include text pattern matching
- Tables contain semi-structured data queries
- Fast lookup performance is critical

Avoid using it when:

- Queries perform full table scans
- Analytical aggregations dominate workload

---

## Other Performance Optimization Techniques (Exam Comparison)

Snowflake performance tuning options include:

- Query Acceleration Service (QAS)
- Materialized Views
- Table Clustering
- Warehouse scaling

Certification trap:  
👉 Search Optimization is **not the only performance method.**

---

## Certification Memory Sentence

Search Optimization Service =  
Enterprise feature that creates a **search access path** to accelerate selective lookup and text search queries.

# ====================================

# Views — Certification Summary

## Definition  
A **View** is a database object that stores a SQL query definition and allows users to query the result **as if it were a table**.

Certification memory:  
👉 View = **stored query, not stored data (except materialized views).**

---

## Types of Views (Must Know for Exam)

### Non-materialized View (Standard View)
- Stores only the **query definition**
- Query runs every time the view is used  
- No data stored  
- Slower performance compared to materialized views  

👉 Key exam concept  
Standard view = **logical layer only**

---

### Materialized View
- Stores **precomputed query results**
- Improves query performance  
- Requires:
  - Storage space  
  - Maintenance cost (credits)

👉 Behaves similarly to a **table copy of subset data**

Important certification idea:  
Materialized views are used for **performance optimization**.

---

### Secure Views
- Can be defined for:
  - Standard views  
  - Materialized views  
- Improve **data privacy and secure data sharing**
- May have slight performance overhead

---

### Recursive Views (Only Standard Views)
- A view can reference itself  
- Implemented using **recursive CTE logic**

---

## Advantages of Views (Exam Focus)

### Modular SQL Design
- Simplifies complex queries  
- Enables layered data modeling  

### Data Security
- Can expose **only subset of table columns or rows**
- Useful for controlled data access

### Performance Improvement (Materialized Views)
- Faster scanning vs base table  
- Can be clustered  
- Multiple materialized views can exist on same table

---

## Limitations (Very Important for Certification)

- View definition **cannot be modified**
  → must recreate view  

- Views are **read-only**
  → cannot run INSERT / UPDATE / DELETE directly  

- Changes in base table structure are **not automatically reflected** in views  

---

## Certification Memory Sentence

View = logical query object.  
Materialized View = stored result for performance.  
Secure View = privacy-focused version of a view.


# ==========================

# Secure Views — Certification Summary

## Definition  
A **Secure View** is a special type of view designed to **protect sensitive data and metadata exposure**.

Certification memory:  
👉 Secure View = **privacy-focused view that hides internal details and query optimizations.**

---

## Why Use Secure Views (Important Exam Concept)

Secure views help prevent:

- Exposure of **underlying table data through query optimizations**
- Visibility of the **view definition to unauthorized users**
- Exposure of **data distribution information (scan size, partitions, etc.)**

👉 Key idea  
Secure views improve **data governance and secure data sharing.**

---

## When to Use Secure Views

Use secure views when:

- The view is created specifically for **data privacy or controlled data sharing**
- You want to **restrict access to sensitive rows or columns**
- You need secure sharing across accounts

Do NOT use secure views when:

- The goal is only **query simplification or convenience**
- Performance is the main priority (secure views can be slower)

---

## How to Create a Secure View

Use the `SECURE` keyword:

CREATE SECURE VIEW my_view AS  
SELECT ... ;

Also supported for materialized views:

CREATE SECURE MATERIALIZED VIEW my_mv AS  
SELECT ... ;

---

## Visibility of Secure View Definition

Unauthorized users **cannot see secure view definition** using:

- SHOW VIEWS  
- SHOW MATERIALIZED VIEWS  
- GET_DDL  
- INFORMATION_SCHEMA views  

Authorized access possible via:

- ACCOUNTADMIN role  
- SNOWFLAKE.OBJECT_VIEWER database role  
- IMPORTED PRIVILEGES on shared SNOWFLAKE database  

👉 Certification tip  
Secure views hide metadata even in **Query Profile.**

---

## How to Check if a View is Secure

Use:

Information Schema:

SELECT table_name, is_secure  
FROM database.information_schema.views;

Account Usage:

SELECT table_name, is_secure  
FROM snowflake.account_usage.views;

---

## Secure Views and Access Control

Secure views can implement **row-level security logic** using:

- CURRENT_ROLE()
- CURRENT_USER()
- CURRENT_ACCOUNT()

Useful for:

- Secure Data Sharing  
- Multi-tenant architectures  

---

## Best Practices (Exam Focus)

- Avoid exposing **sequence-generated surrogate keys**  
  → users may infer hidden data volume  

- Secure views hide **scanned data size and partition info**,  
  but users might still infer data patterns from performance  

- Use CURRENT_ACCOUNT() to control **cross-account sharing access**

---

## Certification Memory Sentence

Secure View = privacy-protected view  
→ hides definition, metadata, and optimization details.

## ======


# Snowflake Materialized Views — COF-C03 Quick Revision

## 1\. Definition

A **Materialized View (MV)** is a **pre-computed and stored query result** used to **improve query performance**.

Key points:

* **Enterprise Edition feature**
* Results are **stored physically**
* **Automatically maintained by Snowflake**
* Used for **repeated expensive queries**

---

# 2\. Why Use Materialized Views

Use them when:

* Queries are **frequent**
* Queries are **resource-intensive**
* Base table changes **infrequently**
* Results are **smaller than the base table**
* Queries include **heavy aggregations**
* Queries analyze **semi-structured data**
* Queries run on **external tables**

**Goal:** Improve query performance.

---

# 3\. When NOT to Use Them

Use a **regular view** instead when:

* Data changes **frequently**
* Queries are **not executed often**
* Queries are **simple**

Rule:

> Materialized View = better performance but higher cost

---

# 4\. Materialized View vs Regular View

|Feature|Regular View|Materialized View|
|-|-|-|
|Stores results|No|Yes|
|Performance|Normal|Faster|
|Maintenance|None|Automatic|
|Cost|Low|Higher|

---

# 5\. Materialized View vs Cached Query Result

|Feature|Cached Result|Materialized View|
|-|-|-|
|Fastest|Yes|No|
|Flexible|No|Yes|
|Requires same query|Yes|No|

Cached results only work if:

* same query
* data has not changed
* deterministic functions are used

---

# 6\. Query Optimizer

Important exam point:

The **Snowflake optimizer can automatically use a materialized view**, even if the query references the **base table**.

You do **not** need to explicitly query the MV.

You can verify usage with:

* `EXPLAIN`
* **Query Profile**

---

# 7\. Commands to Know

Create materialized view

```sql

CREATE MATERIALIZED VIEW mv_name AS

SELECT ...

```

Modify materialized view

```sql

ALTER MATERIALIZED VIEW mv_name ...

```

Drop materialized view

```sql

DROP MATERIALIZED VIEW mv_name

```

Describe materialized view

```sql

DESCRIBE MATERIALIZED VIEW mv_name

```

List materialized views

```sql

SHOW MATERIALIZED VIEWS

```

Suspend or resume maintenance

```sql

ALTER MATERIALIZED VIEW mv_name SUSPEND

ALTER MATERIALIZED VIEW mv_name RESUME

```

---

# 8\. Maintenance

Materialized views are **automatically maintained**.

Snowflake refreshes them when:

* `INSERT`
* `UPDATE`
* `DELETE`

occur on the **base table**.

Maintenance runs **in the background**.

---

# 9\. Cost

Materialized views increase costs because of:

### Storage

Materialized views store query results.

### Compute

Snowflake consumes **credits** for automatic refresh.

Cost depends on:

* number of materialized views
* frequency of base table updates
* clustering
* data volume

Important concept:

> Materialized views improve performance but increase cost.

---

# 10\. Major Limitations

Materialized views:

* Only **one base table**
* **No joins**

Cannot reference:

* another materialized view
* regular view
* dynamic table
* hybrid table
* UDTF

Not supported in MV definitions:

* `JOIN`
* `WINDOW FUNCTIONS`
* `HAVING`
* `ORDER BY`
* `LIMIT`
* nested subqueries
* `UDF`
* `EXCEPT`
* `INTERSECT`
* `MINUS`

---

# 11\. Supported Aggregate Functions

Examples supported:

* `COUNT`
* `COUNT_IF`
* `SUM`
* `AVG`
* `MIN`
* `MAX`
* `STDDEV`
* `VARIANCE`
* `APPROX_COUNT_DISTINCT`

Rules:

* No **nested aggregates**
* No **window aggregation (`OVER`)**

---

# 12\. DML Restrictions

You **cannot run DML on a materialized view**.

Not allowed:

* `INSERT`
* `UPDATE`
* `DELETE`
* `MERGE`
* `COPY`
* `TRUNCATE`

You must modify the **base table** instead.

---

# 13\. Base Table Changes

|Change|Effect on Materialized View|
|-|-|
|Add column|MV unchanged|
|Drop column|MV becomes invalid|
|Modify column|MV suspended|
|Drop table|MV unusable|

Often requires **recreating the MV**.

---

# 14\. Best Practices

* Use MVs for **expensive and frequent queries**
* Avoid `SELECT *`
* Filter rows and columns
* Batch DML operations
* Monitor maintenance costs
* Start with **few materialized views**

---

# 15\. Clustering

Materialized views **can be clustered**.

Pros:

* Faster queries

Cons:

* Higher maintenance cost

---

# 16\. Time Travel

Materialized views:

* **cannot query historical data**
* **can be cloned** through database or schema cloning

---

# 17\. The 10 Exam Facts to Memorize

1. Materialized views are **Enterprise Edition features**
2. They **store precomputed query results**
3. They **improve query performance**
4. They are **automatically maintained**
5. They **consume storage and compute credits**
6. Best for **frequent expensive queries**
7. The **optimizer can automatically use them**
8. Only **one base table allowed**
9. **No joins**
10. **No DML allowed on the materialized view**



# Snowflake Semantic Views

## 1\. Definition

A **Semantic View** is a **schema-level object that stores business meaning and definitions for data**.

It allows you to define:

* **Business metrics**
* **Business entities**
* **Relationships between data**

Semantic Views create a **business-friendly layer on top of physical tables**.

Important:

* Semantic Views are considered **metadata**
* They provide **consistent business definitions across applications**

---

# 2\. Purpose of Semantic Views

Semantic Views solve the mismatch between:

* **Business language**
* **Database schema names**

Example:

Database column:

```
amt\\_ttl\\_pre\\_dsc
```

Business meaning:

```
Gross Revenue
```

Semantic Views map **technical fields → business concepts**.

---

# 3\. Benefits

### For AI Applications

Semantic Views improve **AI query accuracy**.

Example:

**Cortex Analyst** reads semantic view definitions and generates SQL automatically.

---

### For Business Intelligence (BI)

Business users benefit from:

* consistent metrics
* reusable dimensions
* standardized calculations

Example:

“Net Revenue by Region” always uses the **same formula**.

---

### For Data Engineers / Analysts

Semantic Views:

* centralize business logic
* reduce duplicated calculations
* simplify complex schemas

---

# 4\. Logical vs Physical Objects

Two types of objects exist:

### Physical Objects

Actual database objects

Examples:

* tables
* columns
* schemas

---

### Logical Objects

Objects defined inside the **semantic view**

Examples:

* logical tables
* metrics
* dimensions
* facts

Semantic Views translate **physical data → logical business model**.

---

# 5\. Logical Tables

A **logical table** represents a **business entity**.

Examples:

* Customers
* Orders
* Products
* Suppliers

Logical tables usually correspond to **physical database tables**.

Relationships between logical tables are defined using **joins on shared keys**.

---

# 6\. Facts

Facts represent **row-level business events**.

They answer:

> How much?  
> How many?

Examples:

* sales amount
* quantity purchased
* transaction cost

Facts exist at the **most granular level of data**.

They are mainly used to **build metrics and dimensions**.

---

# 7\. Metrics

Metrics are **aggregated measures of business performance**.

They use functions such as:

* `SUM`
* `AVG`
* `COUNT`

Examples:

* Total Revenue
* Profit Margin
* Total Orders

Metrics are **KPIs used in dashboards and reports**.

---

# 8\. Dimensions

Dimensions provide **context for analyzing metrics**.

They answer:

* Who
* What
* Where
* When

Examples:

* Customer
* Product Category
* Region
* Purchase Date

Dimensions allow users to:

* filter
* group
* analyze data

---

# 9\. Semantic View Components

|Component|Purpose|
|-|-|
|Facts|Row-level business data|
|Metrics|Aggregated business KPIs|
|Dimensions|Categories for analysis|

Metrics and dimensions are the **primary elements used in analysis**.

---

# 10\. Creating Semantic Views

You can create semantic views using:

### SQL

Example:

```sql
CREATE SEMANTIC VIEW ...
```

---

### Snowsight Wizard

Snowsight provides a **visual interface** to:

* create semantic views
* upload YAML definitions

---

### YAML Specification

Semantic views can also be defined using **YAML configuration files**.

---

# 11\. Querying Semantic Views

Semantic views can be queried using **SQL SELECT statements**.

Example:

```sql
SELECT metric\\_name, dimension\\_name
FROM semantic\\_view\\_name
```

They behave like **logical analytical models**.

---

# 12\. Integration with Cortex Analyst

Semantic views integrate with **Cortex Analyst**.

Cortex Analyst can:

* read semantic definitions
* convert natural language queries into SQL

Example:

User asks:

> "Net revenue by region"

Cortex Analyst generates SQL using the **semantic definitions**.

---

# 13\. Sharing Semantic Views

Semantic Views can be shared using:

* **Private listings**
* **Snowflake Marketplace**
* **Organizational listings**

This allows **standardized data models across organizations**.

---

# 14\. Interfaces to Manage Semantic Views

Three main interfaces:

|Interface|Purpose|
|-|-|
|SQL|Create and manage views|
|Snowsight|Visual creation wizard|
|Cortex Analyst API|AI-based queries|

---

# 15\. Limitation

Current limitation:

* **Semantic Views do NOT support replication**

---

# 16\. Typical Workflow

### Step 1 — Design Business Model

Define:

* business entities
* metrics
* dimensions

Example:

* customers
* orders
* revenue

---

### Step 2 — Map to Physical Tables

Identify:

* which tables store the data
* how tables are joined

Best practice:

Use a **star schema**.

---

### Step 3 — Define Metrics

Example:

```
Net Revenue = SUM(gross\\_revenue \\* (1 - discount))
```

---

### Step 4 — Create Semantic View

Using:

* SQL
* Snowsight
* YAML specification

---

### Step 5 — Query the Semantic View

Using:

* SQL queries
* Cortex Analyst natural language queries

---

# 17\. Key Concepts to Remember for the Exam

1. Semantic Views store **business definitions**
2. They are **schema-level objects**
3. They are considered **metadata**
4. They map **business concepts to physical data**
5. They define **facts, metrics, and dimensions**
6. They improve **AI and BI query consistency**
7. They can be queried using **SELECT**
8. They integrate with **Cortex Analyst**
9. They can be shared via **Snowflake Marketplace**
10. They **do not support replication**

---

# 18\. Simple Summary

Semantic Views provide a **business-friendly abstraction layer** over database tables.

They allow organizations to:

* define consistent metrics
* standardize business logic
* simplify analytics
* improve AI query accuracy


# Authentication Policies — Certification Summary

## Definition  
An **Authentication Policy** controls **how users and clients authenticate to Snowflake.**  

👉 Centralizes login security rules.

---

## What Authentication Policies Control (Exam Focus)

They can define:

- MFA enrollment requirement  
- MFA enforcement  
- Allowed authentication methods  
- Allowed SAML security integrations  
- Allowed client types  
- Minimum client versions  
- Programmatic Access Token (PAT) settings  

👉 Example authentication methods:
- PASSWORD  
- SAML  
- OAUTH  
- KEY_PAIR  
- PROGRAMMATIC_ACCESS_TOKEN  

👉 Example client types:
- SNOWFLAKE_UI  
- SNOWFLAKE_CLI  
- SNOWSQL  
- DRIVERS  

---

## Where Policies Can Be Applied

Authentication policies can be set at:

- Account level  
- User level  
- All SERVICE users  

⭐ Exam rule  
👉 **User-level policy overrides account-level policy.**

---

## Main Use Cases

Use authentication policies to:

- Enforce MFA  
- Restrict login methods  
- Restrict login client types  
- Allow specific Identity Providers (IdP)  
- Enforce minimum client versions  
- Control login experience  

---

## Authentication Methods vs Security Integrations

Policies can define:

- Allowed authentication methods  
- Allowed SAML integrations  

⭐ Exam rule  
👉 Methods and integrations must be **consistent** or policy creation fails.

Example conflict:

- Allow only OAUTH  
- Specify only SAML integration  

---

## CLIENT_TYPES Limitation (Very Important)

Authentication policies can restrict client access.

Example:

- Allow only Snowsight  

But:

⭐ Exam rule  
👉 CLIENT_TYPES is **best-effort only**  
👉 It does NOT block REST API access  
👉 It should NOT be used as the only security control.

---

## MFA Enforcement

Policies can require:

- MFA enrollment  
- MFA for password users  
- MFA for SSO users  

⭐ Exam rule  
👉 Snowflake is deprecating single-factor password login.

Important rule:

If:

```
MFA_ENROLLMENT = REQUIRED
```

Then:

```
CLIENT_TYPES must include SNOWFLAKE_UI
```

👉 Users enroll MFA only via Snowsight.

---

## Security Policy Evaluation Order (Exam Trap)

Order of evaluation:

1. Network policies  
2. Authentication policies  
3. Password policies  
4. Session policies  

⭐ Exam rule  
👉 If blocked by network policy → authentication policy not evaluated.

---

## Identifier-First Login

Login flow:

- User enters username/email  
- Snowflake detects user  
- Shows only allowed login methods  

Benefits:

- Cleaner login experience  
- Easier multi-IdP control  

⭐ Exam rule  
👉 Identifier-first login shows only authentication methods allowed by policy.

---

## Creating an Authentication Policy

Main command:

```
CREATE AUTHENTICATION POLICY my_policy
  CLIENT_TYPES = ('SNOWFLAKE_UI')
  AUTHENTICATION_METHODS = ('SAML','PASSWORD');
```

Default behavior if not specified:

- All client types allowed  
- All methods allowed  
- All integrations allowed  

---

## Applying an Authentication Policy

Account level:

```
ALTER ACCOUNT SET AUTHENTICATION POLICY my_policy;
```

User level:

```
ALTER USER user1 SET AUTHENTICATION POLICY my_policy;
```

Service users:

```
ALTER ACCOUNT SET AUTHENTICATION POLICY my_policy
FOR ALL SERVICE USERS;
```

---

## Required Privileges

- CREATE AUTHENTICATION POLICY (schema)
- APPLY AUTHENTICATION POLICY (account/user)
- OWNERSHIP (full control)

Also need privileges on:

- Parent database  
- Parent schema  

---

## Key Commands to Know

Create:
```
CREATE AUTHENTICATION POLICY
```

Alter:
```
ALTER AUTHENTICATION POLICY
```

Drop:
```
DROP AUTHENTICATION POLICY
```

Describe:
```
DESCRIBE AUTHENTICATION POLICY
```

Show:
```
SHOW AUTHENTICATION POLICIES
```

---

## MFA Hardening Example

Require MFA for password users:

```
MFA_POLICY =
(ENFORCE_MFA_ON_EXTERNAL_AUTHENTICATION = NONE)
```

Require MFA for password + SSO:

```
MFA_POLICY =
(ENFORCE_MFA_ON_EXTERNAL_AUTHENTICATION = ALL)
```

⭐ Exam rule  
👉 ALL → also enforces MFA for SSO.

---

## Tracking Policy Usage

Use:

```
POLICY_REFERENCES()
```

👉 Shows which users or account use the policy.

---

## Preventing Admin Lockout (Best Practice)

Create a less restrictive policy for admins.

Reason:

- User-level policy overrides account-level  
- Admin keeps emergency access  

⭐ Exam best practice.

---

## Replication

Authentication policies support:

- Replication groups  
- Failover groups  

⭐ Exam rule  
👉 Authentication policies are replicable.

---

## Simple Final Memory (Exam Sentence)

Authentication Policy = controls:

- how users login  
- which client they use  
- which IdP they use  
- whether MFA is required  

Key rules:

- User policy overrides account policy  
- Network policy evaluated first  
- CLIENT_TYPES is not full security  
- MFA enrollment requires Snowsight
-
-
  =========================================================================================



# Snowflake Multi-Factor Authentication (MFA)

# 

# 1\. Definition

**Multi-factor authentication (MFA)** adds a **second factor of authentication** in addition to a password.

Login flow:

* user enters **password**
* user confirms identity with a **second factor**

Main purpose:

> MFA reduces the security risks of password-based authentication.

---

# 2\. Who MFA Applies To

MFA is intended for:

* **human users**
* users who authenticate with a **password**

MFA is **not for service users**.

Service users must use another authentication method.

Exam point:

> MFA is mainly for \*password-based human users\*, not service users.

---

# 3\. Important Global Change

Snowflake is rolling out mandatory MFA for **all password sign-ins**.

This means:

* single-factor password sign-in is being deprecated
* password-only login is being removed progressively

Exam point:

> Snowflake is deprecating \*single-factor password authentication\*.

---

# 4\. MFA Enrollment and the 2024\_08 Bundle

MFA behavior depends on whether the account existed before or after the **2024\_08 behavior change bundle**.

## Accounts created before 2024\_08 bundle

* MFA is **not automatically required**
* admins must configure it if they want all human users to use MFA

## Accounts created after 2024\_08 bundle

* all **human password users** must enroll in MFA by default
* this does **not** apply to service users

Exam point:

> Newer accounts require MFA by default for \*human password users\*.

---

# 5\. Making MFA Optional

Admins can temporarily opt out of mandatory MFA for human users by creating an authentication policy with:

```sql
MFA\\_ENROLLMENT = OPTIONAL
```

However:

* password users in **Snowsight** still must use MFA
* this opt-out is temporary

Exam point:

> `MFA\\_ENROLLMENT = OPTIONAL` can reduce MFA enforcement temporarily, but not permanently.

---

# 6\. MFA for SSO Users

By default, Snowflake does **not** require Snowflake MFA for users authenticating with **SSO**.

Why?

* Snowflake expects the **IdP** to enforce MFA or another strong authentication method

If you want stronger security, create an authentication policy that also requires Snowflake MFA for SSO users.

Example:

```sql
CREATE AUTHENTICATION POLICY ACCOUNTADMIN\\_DOUBLE\\_MFA
  AUTHENTICATION\\_METHODS = ('PASSWORD', 'SAML')
  SECURITY\\_INTEGRATIONS = ('<SAML SECURITY INTEGRATIONS>')
  MFA\\_ENROLLMENT = 'REQUIRED'
  MFA\\_POLICY = (ENFORCE\\_MFA\\_ON\\_EXTERNAL\\_AUTHENTICATION = 'ALL');
```

Exam point:

> SSO users are \*not required\* to use Snowflake MFA by default unless a policy enforces it.

---

# 7\. Supported MFA Methods

Snowflake supports these MFA methods:

* **Passkey**
* **Authenticator app (TOTP)**
* **Duo**

---

# 8\. Recommended MFA Method

Snowflake recommends:

* **Passkeys**

Why:

* better security
* better usability

Important note:

* **Duo is not replicated** like passkeys and TOTP

Exam point:

> \*Passkeys are recommended\*.
>
> \*Duo is not replicated\*.

---

# 9\. Restricting MFA Methods with Authentication Policies

Admins can control which MFA methods users are allowed to use.

Example:

```sql
CREATE AUTHENTICATION POLICY mfa\\_policy
  MFA\\_ENROLLMENT = REQUIRED
  MFA\\_POLICY = (ALLOWED\\_METHODS = ('PASSKEY', 'TOTP'));
```

This means:

* passkeys allowed
* TOTP allowed
* Duo blocked

If a user already configured a method that becomes prohibited:

* they authenticate once with the existing method
* then Snowflake prompts them to configure a new allowed method

Exam point:

> Authentication policies can restrict which MFA methods are allowed.

---

# 10\. Viewing and Removing MFA Methods

To list MFA methods for a user:

```sql
SHOW MFA METHODS FOR USER joe;
```

To remove a method:

```sql
ALTER USER joe REMOVE MFA METHOD TOTP-48A7;
```

Exam point:

> `SHOW MFA METHODS` lists MFA methods.
>
> `ALTER USER ... REMOVE MFA METHOD` removes a configured MFA method.

---

# 11\. Recovering a Locked-Out User

If a user loses access to their MFA method, an admin can help in two main ways:

* prompt the user to enroll a new MFA method
* temporarily bypass MFA

---

# 12\. Prompting a User to Add a New MFA Method

Admin command:

```sql
ALTER USER joe ENROLL MFA;
```

Behavior:

* if the user has a verified email, Snowflake emails them instructions
* otherwise, Snowflake returns a URL that the admin can share

Exam point:

> `ALTER USER ... ENROLL MFA` is used to help a locked-out user set up a new MFA method.

---

# 13\. Temporarily Bypassing MFA

Admin command:

```sql
ALTER USER joe SET MINS\\_TO\\_BYPASS\\_MFA = 30;
```

Meaning:

* user can sign in without MFA for 30 minutes

Exam point:

> `MINS\\_TO\\_BYPASS\\_MFA` temporarily disables MFA for recovery purposes.

---

# 14\. Break Glass Access

**Break glass access** means emergency login using alternative authentication when normal methods fail.

Example:

* IdP outage
* standard login unavailable

Best practice:

* create a dedicated Snowflake user
* store its password in a key vault
* store one or more **OTPs** with it

This allows emergency access while still meeting MFA requirements.

Exam point:

> Break glass access uses a dedicated user and emergency MFA options such as OTPs.

---

# 15\. One-Time Passcodes (OTPs)

Admins can generate OTPs for a user.

Example:

```sql
ALTER USER breakglass\\_user ADD MFA METHOD OTP COUNT = 5;
```

Important:

* OTPs are **one-time use**
* once used, an OTP is invalidated

Exam point:

> OTPs can be generated for emergency access and are invalid after one use.

---

# 16\. Invalidating OTPs

Ways to invalidate OTPs:

## Invalidate all OTPs

Generate new OTPs; old OTPs become invalid.

## Invalidate a specific OTP for another user

```sql
ALTER USER joe REMOVE MFA METHOD OTP\\_2;
```

## Invalidate your own OTP

Do it through **Snowsight**.

Exam point:

> Generating new OTPs invalidates previously generated OTPs.

---

# 17\. Replication of Break Glass MFA

When break glass users are replicated:

* **OTPs are not replicated**

Reason:

* to prevent the same OTP from working in both source and target accounts

Alternatives in target account:

* generate OTPs directly there
* use **TOTP** or **passkey**, which can be replicated

Exam point:

> OTPs are \*not replicated\* across accounts.

---

# 18\. Where MFA Is Supported

MFA is designed primarily for the **web interface**, but it is also supported by:

* Snowflake CLI
* SnowSQL
* JDBC
* Node.js
* ODBC
* Python Connector

Important note:

* MFA using landline or phone callback does **not** support drivers like JDBC and ODBC

---

# 19\. MFA Token Caching

MFA token caching reduces how often the user is prompted for MFA.

Key facts:

* cached MFA token is valid for **up to 4 hours**
* stored in the client OS keystore
* optional feature

Benefits:

* fewer MFA prompts
* useful when many connections happen in a short period

Exam point:

> MFA token caching reduces repeated prompts and tokens are valid for \*up to 4 hours\*.

---

# 20\. When MFA Token Cache Becomes Invalid

Cached MFA token becomes invalid if:

* `ALLOW\\_CLIENT\\_MFA\\_CACHING` is `FALSE`
* authentication method changes
* username or password changes
* credentials are invalid
* token expires
* token is not cryptographically valid
* account name changes

Exam point:

> MFA token caching depends on stable credentials, auth method, and account context.

---

# 21\. Enabling MFA Token Caching

Enable it at account level:

```sql
ALTER ACCOUNT SET ALLOW\\_CLIENT\\_MFA\\_CACHING = TRUE;
```

Client connection should use:

```text
authenticator = username\\_password\\_mfa
```

Disable it:

```sql
ALTER ACCOUNT UNSET ALLOW\\_CLIENT\\_MFA\\_CACHING;
```

Exam point:

> `ALLOW\\_CLIENT\\_MFA\\_CACHING` enables or disables MFA token caching.

---

# 22\. Tracking MFA Token Caching Usage

Admins can find users authenticating with MFA token caching using:

```sql
SELECT EVENT\\_TIMESTAMP,
       USER\\_NAME,
       IS\\_SUCCESS
  FROM SNOWFLAKE.ACCOUNT\\_USAGE.LOGIN\\_HISTORY
  WHERE SECOND\\_AUTHENTICATION\\_FACTOR = 'MFA\\_TOKEN';
```

Exam point:

> `LOGIN\\_HISTORY` can be queried to track MFA token-based logins.

---

# 23\. Driver and Client Behavior

MFA is supported in:

* Snowflake CLI
* SnowSQL
* JDBC
* Node.js
* ODBC
* Python

Default behavior for enrolled users:

* **Duo Push** is commonly the default mechanism

Alternative:

* use a **Duo-generated passcode**

For the exam, you do **not** need to memorize every driver parameter.

What matters most:

> Snowflake supports MFA across major clients and drivers.

---

# 24\. Most Important Commands

Show MFA methods:

```sql
SHOW MFA METHODS FOR USER joe;
```

Remove MFA method:

```sql
ALTER USER joe REMOVE MFA METHOD TOTP-48A7;
```

Prompt user to enroll MFA again:

```sql
ALTER USER joe ENROLL MFA;
```

Temporarily bypass MFA:

```sql
ALTER USER joe SET MINS\\_TO\\_BYPASS\\_MFA = 30;
```

Generate OTPs:

```sql
ALTER USER breakglass\\_user ADD MFA METHOD OTP COUNT = 5;
```

Enable MFA token caching:

```sql
ALTER ACCOUNT SET ALLOW\\_CLIENT\\_MFA\\_CACHING = TRUE;
```

Disable MFA token caching:

```sql
ALTER ACCOUNT UNSET ALLOW\\_CLIENT\\_MFA\\_CACHING;
```

---

# 25\. Best Practices

* require MFA for all password-based human users
* prefer **passkeys**
* use authentication policies to control allowed MFA methods
* keep break glass access for emergencies
* keep a backup MFA method to avoid lockout
* be careful when enabling MFA token caching
* do not use service users with password-based MFA

---

# 26\. Simple Summary

Snowflake MFA:

* adds a second factor to password authentication
* is intended for human password users
* is becoming mandatory for all password sign-ins
* supports passkeys, TOTP, and Duo
* can be controlled by authentication policies
* supports recovery through re-enrollment or temporary bypass
* supports emergency access with OTPs
* supports optional MFA token caching

---

# 27\. The 15 Exam Facts to Memorize

1. MFA adds a second factor to password authentication
2. MFA is intended for human password users, not service users
3. Snowflake is deprecating single-factor password sign-ins
4. Newer accounts require MFA by default for human password users
5. SSO users are not required to use Snowflake MFA by default
6. Authentication policies can enforce MFA for SSO users
7. Supported MFA methods are passkey, TOTP, and Duo
8. Passkeys are the recommended MFA method
9. Duo is not replicated
10. `SHOW MFA METHODS` lists a user’s MFA methods
11. `ALTER USER ... REMOVE MFA METHOD` removes one MFA method
12. `ALTER USER ... ENROLL MFA` helps recover a locked-out user
13. `ALTER USER ... SET MINS\\_TO\\_BYPASS\\_MFA` temporarily bypasses MFA
14. OTPs are one-time use and are not replicated
15. MFA token caching is optional and valid for up to 4 hours

---

# ========================================================================================

# Snowflake Federated Authentication and SSO

# 1\. Definition

**Federated authentication** means user authentication is handled by an **external Identity Provider (IdP)** instead of Snowflake directly.

With this model:

* the **IdP authenticates the user**
* Snowflake receives that authentication result
* the user gets **single sign-on (SSO)** access to Snowflake

Main idea:

> Authentication is separated from access.

---

# 2\. Main Components in a Federated Environment

A federated environment has two main components:

## Service Provider (SP)

In Snowflake federated authentication:

* **Snowflake is the Service Provider (SP)**

## Identity Provider (IdP)

The IdP is the external system that:

* creates and maintains user credentials
* authenticates users
* sends the authentication result to Snowflake

Examples:

* Okta
* Microsoft Entra ID
* other SAML 2.0-compliant IdPs

Exam point:

> In Snowflake federated authentication, **Snowflake = SP** and **external system = IdP**.

---

# 3\. Protocol Used

Snowflake federated authentication is based on:

* **SAML 2.0**

Snowflake supports most:

* **SAML 2.0-compliant IdPs**

Exam point:

> Federated authentication in Snowflake commonly uses **SAML 2.0**.

---

# 4\. Supported Identity Providers

Snowflake has native support for:

* **Okta**
* **Microsoft Entra ID**

Snowflake also supports most SAML 2.0-compliant IdPs, such as:

* Google G Suite
* OneLogin
* Ping Identity PingOne
* Microsoft Entra ID

Important:

> If the IdP is not Okta or Entra ID, you must usually define a **custom Snowflake application** in the IdP.

Exam point:

> Native support is especially highlighted for **Okta** and **Microsoft Entra ID**.

---

# 5\. Multiple Identity Providers

Snowflake can be configured so that:

* different users authenticate with different IdPs

Important note:

* only some Snowflake drivers support multiple IdPs

Supported drivers include:

* JDBC
* ODBC
* Python

Exam point:

> Snowflake supports **multiple IdPs**, but only a subset of drivers support that feature.

---

# 6\. Supported SSO Workflows

Federated authentication supports these workflows:

* **login**
* **logout**
* **session timeout**

The behavior depends on whether the action starts from:

* Snowflake
* the IdP

---

# 7\. Login Workflows

There are two main login flows:

## Snowflake-Initiated Login

Flow:

1. user goes to Snowflake
2. user selects IdP login
3. user authenticates with IdP
4. IdP sends **SAML response** to Snowflake
5. Snowflake opens the session

Important note:

* Snowflake can be configured to redirect the user directly to the IdP without showing the Snowflake sign-in page

## IdP-Initiated Login

Flow:

1. user goes to the IdP
2. user signs in at the IdP
3. user selects the Snowflake application
4. IdP sends **SAML response** to Snowflake
5. Snowflake session starts

Exam point:

> Snowflake supports both **SP-initiated login** and **IdP-initiated login**.

---

# 8\. What Happens at Login

Successful login requires:

* user authenticated by IdP
* IdP sends a **SAML response/assertion**
* Snowflake trusts the IdP and creates the session

Exam point:

> The IdP authenticates the user and sends a **SAML response** to Snowflake.

---

# 9\. Logout Workflows

There are two logout models:

## Standard Logout

* user must log out of Snowflake and the IdP separately

## Global Logout

* logging out from the IdP can also log the user out from all Snowflake sessions

Important:

* **all IdPs support standard logout**
* **global logout depends on the IdP**

Exam point:

> All IdPs support **standard logout**, but **global logout is IdP-dependent**.

---

# 10\. Snowflake-Initiated Logout

If the user logs out from Snowflake:

* only that Snowflake session ends
* other Snowflake sessions remain open
* IdP session remains open

Important:

> Snowflake does **not** support global logout when logout is initiated from Snowflake.

Exam point:

> Snowflake-initiated logout only ends the **current Snowflake session**.

---

# 11\. IdP-Initiated Logout

If logout starts from the IdP, behavior depends on IdP capabilities.

## Microsoft Entra ID

* supports **standard logout**
* supports **global logout**

If global logout is enabled:

* logging out from Entra ID can log the user out of all Snowflake sessions

## Okta

* supports **standard logout only**

If user logs out of Okta:

* active Snowflake sessions stay open
* but new Snowflake sessions require re-authentication through Okta

## Custom IdPs

* standard logout supported
* global logout varies by provider

Exam point:

> **Entra ID** can support global logout.
>
> **Okta** supports standard logout only.

---

# 12\. Browser Closing Does Not Always End IdP Session

For web-based IdPs like Okta:

* closing the browser tab/window does **not necessarily** end the IdP session

If the IdP session is still active:

* user may still be able to access Snowflake until the IdP session times out

Exam point:

> Closing the browser does not always terminate the IdP session.

---

# 13\. Timeout Workflows

Two timeout types matter:

* **Snowflake session timeout**
* **IdP session timeout**

---

# 14\. Snowflake Session Timeout

If a Snowflake session expires due to inactivity:

* Snowflake UI is disabled
* user is prompted to authenticate again through the IdP

Options:

* re-authenticate and continue
* cancel the expired session
* go back to the IdP and relaunch Snowflake, which creates a new session

Exam point:

> If the **Snowflake session** times out, user must re-authenticate through the IdP.

---

# 15\. IdP Session Timeout

If the IdP session times out:

* current Snowflake sessions remain open
* existing Snowflake sessions are not interrupted
* but new Snowflake sessions require logging in to the IdP again

Exam point:

> **IdP timeout does not close existing Snowflake sessions**, but new sessions require IdP login.

---

# 16\. SSO with Private Connectivity

Snowflake supports SSO with private connectivity on:

* AWS
* Azure
* GCP

Important rule:

> For a given Snowflake account, SSO works with only **one account URL at a time**.

That means:

* either the **public account URL**
* or the **private connectivity URL**

Not both at the same time for SSO.

Exam point:

> SSO can use only **one account URL per account** at a time.

---

# 17\. Order for Private Connectivity + SSO

If using SSO with private connectivity:

> Configure **private connectivity first**, then configure SSO.

This applies to:

* AWS PrivateLink
* Azure Private Link
* GCP Private Service Connect

---

# 18\. GCP Private Connectivity Note

For GCP:

* contact Snowflake Support
* use `SYSTEM$GET_PRIVATELINK_CONFIG` to determine the correct URL

This is more implementation detail than a core exam point, but useful to recognize.

---

# 19\. Replication of SSO Configuration

Snowflake supports replication and failover/failback of:

* **SAML2 security integration**

This can be replicated from:

* source account
* target account

Exam point:

> SAML2 security integrations used for SSO can be replicated.

---

# 20\. Simple Summary of Federated Authentication

Federated authentication means:

* Snowflake does not authenticate the user directly
* an **IdP** authenticates the user
* the IdP sends a **SAML response**
* Snowflake grants access through **SSO**

Key facts:

* Snowflake is the **Service Provider**
* IdP is the **Identity Provider**
* SAML 2.0 is the core protocol
* login can be initiated from Snowflake or the IdP
* logout behavior depends on the IdP
* session timeout behavior differs between Snowflake and IdP
* SSO supports private connectivity
* SAML2 integrations can be replicated

---

# 21\. The 15 Exam Facts to Memorize

1. Federated authentication separates authentication from access
2. In Snowflake federated authentication, **Snowflake is the Service Provider (SP)**
3. The external authentication system is the **Identity Provider (IdP)**
4. Snowflake federated authentication commonly uses **SAML 2.0**
5. Snowflake has native support for **Okta** and **Microsoft Entra ID**
6. Other SAML 2.0-compliant IdPs are also supported
7. If using a non-native IdP, you usually create a **custom application** in the IdP
8. Snowflake supports **multiple identity providers**
9. Only some drivers support multiple IdPs: **JDBC, ODBC, Python**
10. Snowflake supports both **Snowflake-initiated** and **IdP-initiated** login
11. On successful login, the IdP sends a **SAML response** to Snowflake
12. Snowflake-initiated logout ends only the current Snowflake session
13. **Entra ID** can support global logout; **Okta** supports standard logout only
14. If the IdP session times out, existing Snowflake sessions remain open
15. SAML2 security integrations for SSO can be **replicated**

---

# 22\. Ultra-Short Exam Summary

**Federated Authentication / SSO in Snowflake**

* Snowflake = **SP**
* External IdP = **authenticates user**
* Protocol = **SAML 2.0**
* Native IdPs = **Okta, Entra ID**
* Login = Snowflake-initiated or IdP-initiated
* Logout = standard for all, global only for some IdPs
* Snowflake timeout requires re-authentication
* IdP timeout does not close current Snowflake sessions
* One SSO account URL at a time
* SAML2 integrations can be replicated

---





# =======================================================

=======================================================

# Snowflake Programmatic Access Tokens (PATs)

## 1\. Definition

A **Programmatic Access Token (PAT)** is a **time-limited credential** that can be used to authenticate to Snowflake **without using a password**.

You can use a PAT for:

* Snowflake REST APIs
* Snowflake SQL API
* Snowflake drivers
* Snowflake CLI / SnowSQL
* third-party tools like Tableau and Power BI
* Snowflake APIs and libraries

Main idea:

> A PAT is a Snowflake-generated credential for programmatic authentication.

---

# 2\. Who Can Use PATs

PATs can be generated for:

* **human users** → `TYPE=PERSON`
* **service users** → `TYPE=SERVICE`

Exam point:

> PATs support both PERSON and SERVICE users.

---

# 3\. Main Use Cases

PATs can be used in two main ways:

## As a password replacement

Use the PAT in place of the password in:

* drivers
* CLI tools
* third-party apps
* SDKs / libraries

## As a Bearer token

Use the PAT in HTTP headers to authenticate to Snowflake endpoints.

Exam point:

> PATs can be used as a password substitute or as a Bearer token.

---

# 4\. Authentication Policy Requirement

If a user is governed by an authentication policy that restricts authentication methods, then:

* the policy must include `PROGRAMMATIC\_ACCESS\_TOKEN`

Example:

```sql
ALTER AUTHENTICATION POLICY my\_auth\_policy
  SET AUTHENTICATION\_METHODS = ('OAUTH', 'PASSWORD', 'PROGRAMMATIC\_ACCESS\_TOKEN');
```

Exam point:

> If authentication methods are restricted, PATs work only if `PROGRAMMATIC\_ACCESS\_TOKEN` is allowed.

---

# 5\. Network Policy Requirement

By default, PAT usage is tied to **network policy requirements**.

## Service users

For `TYPE=SERVICE` users:

* user must be subject to a **network policy** to generate and use a PAT

## Human users

For `TYPE=PERSON` users:

* can generate a PAT without a network policy
* but must be subject to a network policy to authenticate with the PAT

Important:

> Users cannot bypass the network policy itself.

Exam point:

> By default, PATs are strongly tied to network policies, especially for service users.

---

# 6\. PAT Network Policy Modes

Authentication policies can change PAT network policy behavior using:

```sql
PAT\_POLICY = ( NETWORK\_POLICY\_EVALUATION = ... )
```

Supported values:

## `ENFORCED\_REQUIRED` (default)

* user must be subject to a network policy
* if a network policy exists, it is enforced

## `ENFORCED\_NOT\_REQUIRED`

* user does not need to be subject to a network policy
* if a network policy exists, it is enforced

## `NOT\_ENFORCED`

* user does not need to be subject to a network policy
* even if a network policy exists, it is not enforced during PAT authentication

Exam point:

> Default PAT behavior is `ENFORCED\_REQUIRED`.

---

# 7\. PAT Expiration Defaults

By default:

* **default expiration** = **15 days**
* **maximum expiration** = **365 days**

Admins can change both values using an authentication policy with `PAT\_POLICY`.

Exam point:

> Default PAT expiration is 15 days and maximum is 365 days.

---

# 8\. Setting PAT Maximum Expiration

Admins can set the maximum allowed PAT lifetime with:

```sql
PAT\_POLICY = (
  MAX\_EXPIRY\_IN\_DAYS = 100
)
```

Important:

* if an existing PAT exceeds the new maximum, authentication with that PAT fails

Exam point:

> Reducing the PAT maximum expiry can invalidate existing longer-lived tokens.

---

# 9\. Setting PAT Default Expiration

Admins can set the default PAT lifetime with:

```sql
PAT\_POLICY = (
  DEFAULT\_EXPIRY\_IN\_DAYS = 5
)
```

Exam point:

> PAT default expiration can be changed with authentication policy.

---

# 10\. Role Restriction for Service Users

By default, when generating a PAT for a **service user**:

* you must specify a **role restriction**

That role is used for:

* privilege evaluation
* object creation

This restriction can be removed using:

```sql
PAT\_POLICY = (
  REQUIRE\_ROLE\_RESTRICTION\_FOR\_SERVICE\_USERS = FALSE
)
```

Important:

* changing it back to `TRUE` invalidates service-user PATs created without role restriction

Exam point:

> Service-user PATs normally require a role restriction.

---

# 11\. PAT Privileges

## For human users

A human user does **not** need special privileges to manage their **own** PATs.

## For other users or service users

To manage PATs for another user or a service user, the role must have:

* `OWNERSHIP`
* or `MODIFY PROGRAMMATIC AUTHENTICATION METHODS` on that user

Example:

```sql
GRANT MODIFY PROGRAMMATIC AUTHENTICATION METHODS ON USER my\_service\_user
  TO ROLE my\_service\_owner\_role;
```

Exam point:

> Managing PATs for another user requires `OWNERSHIP` or `MODIFY PROGRAMMATIC AUTHENTICATION METHODS`.

---

# 12\. Generating a PAT

PATs can be generated in:

* **Snowsight**
* **SQL**

Important details:

* token name uses letters, numbers, underscores
* token secret is shown **only once**
* after the dialog closes, you cannot see the secret again

Exam point:

> The PAT secret is visible only at creation time.

---

# 13\. Role Scope When Generating a PAT

When generating a PAT, you can restrict it to:

* **one specific role** (recommended)
* or use broader role behavior depending on the interface

Important:

* for a **service user**, role restriction is normally required
* if the restricted role is revoked or dropped, the PAT stops working
* **secondary roles are not used**

Exam point:

> PATs restricted to a role stop working if that role is revoked or dropped.

---

# 14\. Using a PAT as a Password

A PAT can be used in place of a password.

Example in Python:

```python
conn = snowflake.connector.connect(
    user=USER,
    password=<programmatic\_access\_token>,
    account=ACCOUNT,
    warehouse=WAREHOUSE,
    database=DATABASE,
    schema=SCHEMA
)
```

This also applies to tools like:

* Tableau
* Power BI

Exam point:

> PATs can replace passwords in drivers and third-party tools.

---

# 15\. Using a PAT to Authenticate to Endpoints

Use the PAT in the HTTP header:

```http
Authorization: Bearer <token\_secret>
```

Optional header:

```http
X-Snowflake-Authorization-Token-Type: PROGRAMMATIC\_ACCESS\_TOKEN
```

Example:

```bash
curl --location "https://myorganization-myaccount.snowflakecomputing.com/api/v2/databases" \\
  --header "Authorization: Bearer <token\_secret>"
```

Exam point:

> PAT endpoint authentication uses `Authorization: Bearer <token\_secret>`.

---

# 16\. PAT\_INVALID Error

A request can fail with `PAT\_INVALID` if:

* user associated with the PAT is not found
* validation fails
* role associated with the PAT is not found
* user does not match the token

You do not need to memorize every cause, just know it indicates token validation/authentication failure.

---

# 17\. Managing PATs

You can:

* list PATs
* rename PATs
* rotate PATs
* revoke PATs
* re-enable disabled PATs

Important rule:

> You cannot modify, rename, rotate, or revoke a PAT in a session authenticated with a PAT.

Exam point:

> A PAT cannot manage itself in a PAT-authenticated session.

---

# 18\. Listing PATs

Admins can list PATs for a user.

Important notes:

* expired PATs disappear after **7 days**
* there is no command to list **all PATs in the account**
* admins can query the `CREDENTIALS` view for account-wide visibility

Exam point:

> There is no command to list all PATs in the account; use `CREDENTIALS` view.

---

# 19\. Rotating a PAT

Rotation:

* returns a **new token secret**
* keeps the same name
* extends expiration time
* expires the old secret

Important:

* copy/download the new secret immediately
* if desired, old secret can expire immediately

Exam point:

> Rotating a PAT creates a new secret and invalidates the old one.

---

# 20\. Revoking a PAT

Revoking a PAT:

* disables it permanently
* the secret cannot be recovered afterward

Exam point:

> You cannot recover a PAT after revoking it.

---

# 21\. Disabled PATs

If login access for a user is disabled, then the user’s PATs are also disabled.

Important:

* if the user is later re-enabled, the PATs remain disabled
* admin must explicitly re-enable them

Example:

```sql
ALTER USER example\_user MODIFY PROGRAMMATIC ACCESS TOKEN example\_token SET DISABLED = FALSE;
```

Exam point:

> Re-enabling a user does not automatically re-enable their disabled PATs.

---

# 22\. Decode PAT Secret

If you have the token secret and need information about it, use:

```sql
SELECT SYSTEM$DECODE\_PAT('abC...Y5Z');
```

This can return info like:

* state
* PAT name
* user name

Exam point:

> `SYSTEM$DECODE\_PAT` helps identify a PAT from its secret.

---

# 23\. Leaked PATs

Snowflake participates in the **GitHub secret scanning partner program**.

If a PAT is leaked in a public GitHub repository:

* Snowflake is notified
* the PAT is automatically disabled
* Snowflake notifies the account admin and associated user

Exam point:

> Publicly leaked PATs on GitHub can be automatically disabled by Snowflake.

---

# 24\. Investigating a Leaked PAT

To investigate a leaked PAT:

* identify sessions where the PAT was used
* inspect queries run in those sessions

Snowflake provides account usage views such as:

* `LOGIN\_HISTORY`
* `SESSIONS`
* `QUERY\_HISTORY`

Exam point:

> PAT usage can be audited through account usage views.

---

# 25\. Identifying PAT Login Sessions

You can identify PAT-authenticated sessions by joining:

* `LOGIN\_HISTORY`
* `CREDENTIALS`

Key columns:

* `first\_authentication\_factor`
* `first\_authentication\_factor\_id`
* `credential\_id`

Exam point:

> PAT usage can be tracked through `LOGIN\_HISTORY` and `CREDENTIALS`.

---

# 26\. Best Practices

Best practices for PATs:

* store PATs securely in a secrets manager
* do not expose PATs in code
* restrict PATs to a specific role when possible
* rotate PATs regularly
* reduce max expiration to encourage shorter token lifetimes
* use network policies
* review PAT usage regularly

Exam point:

> PATs should be short-lived, securely stored, role-restricted, and regularly rotated.

---

# 27\. Key Limitations

Important limitations:

* PAT secret is shown **only once**
* cannot change PAT expiration after creation
* cannot change/remove role restriction after creation
* max **15 PATs per user**
* disabled tokens count toward the limit
* expired tokens do not count toward the limit
* cannot recover a revoked PAT
* cannot modify/manage a PAT in a PAT-authenticated session

Exam point:

> Major PAT limits include secret visible once, max 15 per user, and expiration/role restriction cannot be changed after creation.

---

# 28\. Very Important Special Notes

## Third-party tools

If a third-party application uses PATs, a network policy may need to allow that application’s IP ranges.

## Secondary roles

When PAT is role-restricted:

* **secondary roles are not used**

Exam point:

> Secondary roles are not used for role-restricted PAT authentication.

---

# 29\. Simple Summary

A **Programmatic Access Token (PAT)** is a Snowflake-generated, time-limited credential for programmatic access.

Main facts:

* works for both human and service users
* can replace passwords or be used as a Bearer token
* usually requires network policy support
* must be allowed in authentication policy
* default expiry = 15 days
* maximum expiry = 365 days
* service-user PATs usually require role restriction
* PAT secrets are visible only once
* PATs can be rotated, revoked, and audited

---

# 30\. The 20 Exam Facts to Memorize

1. A PAT is a time-limited Snowflake credential for programmatic authentication
2. PATs can be used by both `TYPE=PERSON` and `TYPE=SERVICE` users
3. PATs can replace passwords in drivers and tools
4. PATs can also be used as Bearer tokens in HTTP requests
5. PAT authentication may require `PROGRAMMATIC\_ACCESS\_TOKEN` in authentication policy
6. By default, PATs are tied to network policy requirements
7. For service users, a network policy is required by default
8. For human users, PAT generation can happen without network policy, but PAT authentication requires network policy by default
9. `NETWORK\_POLICY\_EVALUATION` controls how network policy applies to PATs
10. Default PAT expiration is **15 days**
11. Maximum PAT expiration is **365 days**
12. Admins can change PAT default and max expiry using authentication policy
13. Service-user PATs normally require a role restriction
14. If a restricted role is revoked or dropped, the PAT stops working
15. PAT secret is shown only once at creation time
16. PATs cannot be modified, rotated, or revoked in a PAT-authenticated session
17. Rotating a PAT creates a new secret and invalidates the old one
18. Revoked PATs cannot be recovered
19. Publicly leaked PATs on GitHub can be automatically disabled by Snowflake
20. Each user can have a maximum of **15 PATs**

---

# 31\. Ultra-Short Exam Summary

**Programmatic Access Tokens (PATs)**

* Snowflake-generated credential
* used for APIs, drivers, CLI, BI tools
* supports PERSON and SERVICE users
* can act as password or Bearer token
* default expiry = 15 days
* max expiry = 365 days
* often requires network policy
* must be allowed in authentication policy
* service-user PATs usually require a role
* secret shown once only
* can rotate/revoke, but not inside a PAT-authenticated session

---

# =========================================================

# Snowflake OAuth

## 1\. Definition

**OAuth** in Snowflake allows clients to authenticate and access Snowflake using **OAuth 2.0 access tokens** instead of passwords.

Snowflake enables OAuth through **security integrations**.

Main idea:

> OAuth lets applications obtain access tokens to connect to Snowflake securely.

---

# 2\. How OAuth Is Configured in Snowflake

OAuth is configured using a **Security Integration**.

A security integration acts as the interface between:

* Snowflake
* client applications
* external OAuth / identity services

It allows clients to:

* redirect users to an authorization page
* obtain **access tokens**
* optionally obtain **refresh tokens**

Exam point:

> OAuth in Snowflake is configured through a **security integration**.

---

# 3\. OAuth Options in Snowflake

Snowflake supports two OAuth models:

* **Snowflake OAuth**
* **External OAuth**

Exam point:

> The two main OAuth options are **Snowflake OAuth** and **External OAuth**.

---

# 4\. Snowflake OAuth vs External OAuth

## Snowflake OAuth

* Snowflake is the OAuth server
* uses OAuth 2.0 **authorization code grant**
* browser access is required
* client app must be modified
* suitable when Snowflake handles OAuth directly

## External OAuth

* external authorization server / IdP handles OAuth
* supports any OAuth flow the external server supports
* browser access is not required
* client app must still be modified
* best fit for programmatic clients

Exam point:

> **Snowflake OAuth** uses the **authorization code grant flow**.
>
> **External OAuth** is generally the **best fit for programmatic clients**.

---

# 5\. Driver Property for OAuth

For both Snowflake OAuth and External OAuth, clients use:

```text
authenticator = oauth
```

Exam point:

> For OAuth connections, the authenticator parameter is `oauth`.

---

# 6\. Security Integration Syntax

## Snowflake OAuth

```sql
CREATE SECURITY INTEGRATION ... TYPE = OAUTH ...
```

## External OAuth

```sql
CREATE SECURITY INTEGRATION ... TYPE = EXTERNAL_OAUTH ...
```

Exam point:

> `TYPE = OAUTH` is for Snowflake OAuth.
>
> `TYPE = EXTERNAL_OAUTH` is for External OAuth.

---

# 7\. OAuth Flow Differences

## Snowflake OAuth

Uses:

* **OAuth 2.0 code grant flow**

## External OAuth

Uses:

* any OAuth flow supported by the external OAuth server and client

Exam point:

> Snowflake OAuth specifically uses the **authorization code flow**.

---

# 8\. Browser Requirement

## Snowflake OAuth

* requires browser access

## External OAuth

* browser access is not required

Exam point:

> Snowflake OAuth requires a browser; External OAuth may not.

---

# 9\. Programmatic Clients

For programmatic clients:

* **External OAuth** is the better fit

Reason:

* does not require a browser
* supports broader OAuth flow options

Exam point:

> **External OAuth** is best suited for **programmatic clients**.

---

# 10\. Auditing OAuth Logins

Snowflake provides login history to audit OAuth-based authentication.

Useful objects:

* `LOGIN_HISTORY`
* `LOGIN_HISTORY_BY_USER`
* `LOGIN_HISTORY` view

When OAuth is used successfully or unsuccessfully:

* `FIRST_AUTHENTICATION_FACTOR = OAUTH_ACCESS_TOKEN`

Exam point:

> In login history, OAuth logins appear as `OAUTH_ACCESS_TOKEN`.

---

# 11\. Private Connectivity Support

Snowflake supports **External OAuth** with private connectivity.

This means External OAuth can be used with private connectivity to the Snowflake service.

Exam point:

> **External OAuth supports private connectivity**.

---

# 12\. Snowflake OAuth and Private Connectivity

Snowflake OAuth can be used with private connectivity in some partner tools.

## Tableau Desktop

Starting with Tableau 2020.4:

* embedded OAuth client supports Snowflake private connectivity URL
* no extra configuration needed after upgrade

## Tableau Cloud

Starting with Tableau 2020.4:

* can optionally use embedded OAuth client
* requires a new **Custom Client** security integration

Exam point:

> Tableau can work with **Snowflake OAuth + private connectivity** starting from Tableau 2020.4.

---

# 13\. Looker Limitation

Snowflake OAuth with **Looker** currently requires access to the **public Internet**.

Therefore:

* you cannot use Snowflake OAuth + Looker with private connectivity

Exam point:

> **Looker + Snowflake OAuth** does **not** work with private connectivity.

---

# 14\. Determining the Private Connectivity URL

To find the correct account URL for private connectivity, use:

```sql
SYSTEM$GET_PRIVATELINK_CONFIG
```

Exam point:

> Use `SYSTEM$GET_PRIVATELINK_CONFIG` to determine the private connectivity account URL.

---

# 15\. Supported Clients, Drivers, and Connectors

OAuth can be used with supported Snowflake clients, drivers, and connectors, including:

* Snowflake CLI
* SnowSQL
* Python
* Go
* JDBC
* ODBC
* Spark Connector
* .NET Driver
* Node.js Driver

You do not need to memorize the full list word-for-word.

What matters:

> Many Snowflake clients and drivers support OAuth authentication.

---

# 16\. Required Connection Parameters

When using OAuth in clients/drivers:

* set `authenticator = oauth`
* set `token = oauth_access_token`

Important note:

* if the token is passed as a URL query parameter, it must be **URL-encoded**
* if passed in a Properties object, no modification is needed

Exam point:

> OAuth client connections require `authenticator = oauth` and the OAuth token.

---

# 17\. Client Redirect Support

Snowflake supports **Client Redirect** with:

* Snowflake OAuth
* External OAuth

This also works with supported Snowflake clients.

Exam point:

> Client Redirect is supported with both Snowflake OAuth and External OAuth.

---

# 18\. Replication Support

Snowflake supports **replication and failover/failback** for both:

* Snowflake OAuth security integrations
* External OAuth security integrations

This works from:

* source account
* target account

Exam point:

> Both Snowflake OAuth and External OAuth integrations support replication and failover/failback.

---

# 19\. Simple Summary

Snowflake OAuth allows applications to authenticate with **OAuth 2.0 tokens**.

Two models exist:

* **Snowflake OAuth**
* **External OAuth**

Main differences:

* Snowflake OAuth uses the **authorization code flow**
* External OAuth is better for **programmatic clients**
* both use `authenticator = oauth`
* both are configured through **security integrations**
* both support replication
* External OAuth supports private connectivity

---

# 20\. The 15 Exam Facts to Memorize

1. Snowflake supports OAuth 2.0 authentication and authorization
2. OAuth is configured using a **security integration**
3. The two OAuth models are **Snowflake OAuth** and **External OAuth**
4. Snowflake OAuth uses `TYPE = OAUTH`
5. External OAuth uses `TYPE = EXTERNAL_OAUTH`
6. Snowflake OAuth uses the **authorization code grant flow**
7. External OAuth can use any flow supported by the external OAuth server
8. Snowflake OAuth requires browser access
9. External OAuth does not require browser access
10. External OAuth is the better fit for programmatic clients
11. OAuth client connections use `authenticator = oauth`
12. In login history, OAuth appears as `FIRST_AUTHENTICATION_FACTOR = OAUTH_ACCESS_TOKEN`
13. External OAuth supports private connectivity
14. Looker + Snowflake OAuth does not work with private connectivity
15. Both Snowflake OAuth and External OAuth integrations support replication and failover/failback

---

# 21\. Ultra-Short Exam Summary

**Snowflake OAuth**

* OAuth 2.0 based
* configured with **security integrations**
* two types: **Snowflake OAuth** and **External OAuth**
* `authenticator = oauth`
* Snowflake OAuth = code grant + browser required
* External OAuth = best for programmatic clients
* OAuth logins show as `OAUTH_ACCESS_TOKEN`
* External OAuth supports private connectivity
* both OAuth integration types support replication

---



**=======================================================================**

**======================================================================**



# Snowflake Workload Identity Federation (WIF)

## 1\. Definition

**Workload Identity Federation (WIF)** is a **service-to-service authentication method** that lets workloads authenticate to Snowflake using their cloud provider’s native identity system.

Examples of native identity systems:

* **AWS IAM roles**
* **Microsoft Entra ID**
* **Google Cloud service accounts**
* **OIDC providers** for Kubernetes or custom platforms

Main idea:

> WIF lets workloads authenticate to Snowflake \*without storing long-lived secrets\*.

---

# 2\. Why WIF Is Important

WIF removes the need to manage long-lived credentials such as:

* passwords
* API keys
* key pairs
* programmatic access tokens

Instead, workloads use:

* native cloud identity
* short-lived attestations or tokens
* automatic driver support

Exam point:

> WIF is \*secretless authentication\* for service-to-service access.

---

# 3\. Main Benefits of WIF

## Security

* no long-lived secrets to store or rotate
* short-lived credentials are used

## Lower operational complexity

* easier than managing passwords, PATs, or key pairs
* often simpler than External OAuth for services

## Cost-effective

* reuses existing cloud identity systems

## Auditing and monitoring

* cloud-native monitoring tools can be used
* Snowflake can audit WIF logins through account usage views

Exam point:

> WIF improves security and reduces secret-management overhead.

---

# 4\. Who WIF Is For

WIF is intended for:

* in-house cloud services
* internal and external service integrations
* multi-tenant SaaS workloads
* containers and applications
* service-to-service authentication

It is **not** for human interactive login.

Exam point:

**> WIF is for service-to-service workloads, not human users.**

---

# 5\. Basic WIF Workflow

The basic workflow is:

1. workload uses a native identity provider
2. provider issues an **attestation** or token
3. Snowflake service user is configured with matching identity properties
4. Snowflake driver sends the attestation to Snowflake
5. Snowflake validates it and authenticates the workload

Exam point:

> WIF works by matching the workload identity token to a Snowflake \*service user\*.

---

# 6\. Snowflake User Type for WIF

WIF requires a **Snowflake service user**.

Use:

```sql
TYPE = SERVICE
```

Exam point:

> WIF is configured on users with `TYPE = SERVICE`.

---

# 7\. Required Privileges to Configure WIF

To configure WIF for a service user, your role needs one of:

* `OWNERSHIP` on the service user
* `MODIFY PROGRAMMATIC AUTHENTICATION METHODS` on the service user

Exam point:

> WIF configuration requires `OWNERSHIP` or `MODIFY PROGRAMMATIC AUTHENTICATION METHODS` on the service user.

---

# 8\. Supported Drivers

WIF is supported by Snowflake drivers, including:

* Go
* JDBC
* .NET
* Node.js
* ODBC
* PHP PDO
* Python

You do **not** need to memorize all minimum versions.

What matters most:

> WIF is supported across major Snowflake drivers.

---

# 9\. How Drivers Use WIF

The Snowflake driver sends the workload’s attestation or token to Snowflake.

Typical connection settings include:

```python
authenticator='WORKLOAD_IDENTITY'
```

and a provider such as:

```python
workload_identity_provider='AWS'
```

or

```python
workload_identity_provider='AZURE'
```

or

```python
workload_identity_provider='GCP'
```

or

```python
workload_identity_provider='OIDC'
```

Exam point:

> Drivers authenticate with WIF using `authenticator='WORKLOAD_IDENTITY'`.

---

# 10\. Supported WIF Providers

Snowflake WIF supports identities from:

* **AWS**
* **Azure / Microsoft Entra ID**
* **Google Cloud**
* **OIDC providers**
* Kubernetes OIDC issuers
* custom OIDC providers

Exam point:

> WIF supports cloud-native identity systems and OIDC providers.

---

# 11\. AWS WIF

For AWS:

* workload uses an **IAM role**
* Snowflake service user stores the AWS identity info

Example Snowflake configuration:

```sql
CREATE USER <username>
  WORKLOAD_IDENTITY = (
    TYPE = AWS
    ARN = '<amazon_resource_identifier>'
  )
  TYPE = SERVICE
  DEFAULT_ROLE = PUBLIC;
```

Key mapping:

* `TYPE = AWS`
* `ARN = IAM role/user identifier`

Exam point:

> AWS WIF uses the workload’s \*IAM ARN\*.

---

# 12\. Azure WIF

For Azure:

* workload uses a **managed identity**
* Snowflake service user stores:

  * issuer URL
  * subject = managed identity object ID

Example:

```sql
CREATE USER <username>
  WORKLOAD_IDENTITY = (
    TYPE = AZURE
    ISSUER = 'https://login.microsoftonline.com/<tenant_id>/v2.0'
    SUBJECT = '<managed_identity_object_id>'
  )
  TYPE = SERVICE
  DEFAULT_ROLE = PUBLIC;
```

Exam point:

> Azure WIF uses `ISSUER` and `SUBJECT` for the managed identity.

---

# 13\. GCP WIF

For Google Cloud:

* workload uses a **service account**
* Snowflake service user stores:

  * `TYPE = GCP`
  * `SUBJECT = uniqueId of the service account`

Example:

```sql
CREATE USER <username>
  WORKLOAD_IDENTITY = (
    TYPE = GCP
    SUBJECT = '<unique_id_of_service_account>'
  )
  TYPE = SERVICE
  DEFAULT_ROLE = PUBLIC;
```

Exam point:

> GCP WIF uses the service account’s \*unique ID\* as the subject.

---

# 14\. OIDC WIF

Snowflake also supports **OIDC-based WIF**.

This is used for:

* EKS
* AKS
* GKE
* custom OIDC providers

Common Snowflake pattern:

```sql
CREATE USER my_service
  WORKLOAD_IDENTITY = (
    TYPE = OIDC
    ISSUER = '<issuer_url>'
    SUBJECT = '<subject>'
  )
  TYPE = SERVICE;
```

Exam point:

> OIDC WIF uses `TYPE = OIDC` with `ISSUER` and `SUBJECT`.

---

# 15\. Kubernetes WIF (EKS, AKS, GKE)

For Kubernetes OIDC workloads:

* projected service account token is used
* token must include audience:

```text
snowflakecomputing.com
```

The workload then authenticates through a Snowflake driver with:

```python
authenticator='WORKLOAD_IDENTITY'
workload_identity_provider='OIDC'
token_file_path='<service_account_token_path>'
```

Exam point:

> Kubernetes WIF tokens must include audience `snowflakecomputing.com`.

---

# 16\. Custom OIDC Provider

Snowflake supports custom OIDC providers if:

* OpenID configuration is publicly accessible
* `jwks_uri` is publicly accessible
* provider can issue valid ID tokens

If the audience is not `snowflakecomputing.com`, specify it in:

```sql
OIDC_AUDIENCE_LIST = ('<custom_audience>')
```

Exam point:

> For custom OIDC, `OIDC_AUDIENCE_LIST` is needed if the audience is not `snowflakecomputing.com`.

---

# 17\. Minimizing the Number of Snowflake Identities

Snowflake recommends **not** creating a separate Snowflake user for every workload at large scale.

Better approach:

* consolidate workloads behind a smaller number of service users
* reduce identity sprawl
* simplify lifecycle management

Exam point:

> Best practice is to minimize Snowflake identity sprawl by consolidating workloads where possible.

---

# 18\. Impersonation for Multiple Workloads

Some cloud providers support **impersonation** so multiple workloads can authenticate using one shared Snowflake identity.

Currently supported for:

* **AWS**
* **GCP**

Not supported for:

* **Azure**

Example use case:

* multiple workloads impersonate a shared cloud identity
* Snowflake maps that shared identity to one service user

Exam point:

> Impersonation can reduce one-to-one workload-to-user mapping, but Azure does not support it.

---

# 19\. Identity Chain

For impersonation, drivers may use an **identity chain**.

Example in Python:

```python
workload_identity_impersonation_path=[
  'service_account_a@my-project.iam.gserviceaccount.com',
  'service_account_b@my-project.iam.gserviceaccount.com',
  'service_account_d@my-project.iam.gserviceaccount.com'
]
```

Important rule:

* Snowflake service user should correspond to the **final identity** in the chain

Exam point:

> With impersonation, Snowflake matches the \*final identity in the chain\*.

---

# 20\. GitHub / GitLab OIDC Consolidation

GitHub and GitLab OIDC tokens may have different `sub` claims by default.

To avoid creating many Snowflake users:

* customize the `sub` claim
* make it the same across environments
* configure Snowflake service user `SUBJECT` to match it

Exam point:

> Customizing the OIDC `sub` claim lets multiple GitHub/GitLab environments share one Snowflake service user.

---

# 21\. Hardening WIF with Authentication Policies

Authentication policies can restrict WIF usage.

They can control:

* which providers are allowed
* which issuers are allowed

Example:

```sql
CREATE AUTHENTICATION POLICY workload_policy
  WORKLOAD_IDENTITY_POLICY = (
    ALLOWED_PROVIDERS = (AZURE)
    ALLOWED_AZURE_ISSUERS = (
      'https://login.microsoftonline.com/<tenant_id>/v2.0'
    )
  );
```

Exam point:

> Authentication policies can restrict WIF by provider and issuer.

---

# 22\. Auditing WIF

WIF usage can be audited through:

* cloud provider logs

  * AWS CloudTrail
  * Azure Monitor

* Snowflake account usage views

  * `LOGIN_HISTORY`
  * `CREDENTIALS`

Exam point:

> WIF can be audited both in the cloud provider and in Snowflake.

---

# 23\. Viewing WIF Settings

To inspect WIF settings for a service user:

```sql
SHOW USER WORKLOAD IDENTITY AUTHENTICATION METHODS FOR USER my_custom_service;
```

Exam point:

> `SHOW USER WORKLOAD IDENTITY AUTHENTICATION METHODS` displays WIF configuration for a service user.

---

# 24\. Important Limitation

Azure workloads cannot be located in:

* Azure sovereign clouds

  * Azure China
  * Azure US Gov

This limitation is about the workload location, not the Snowflake account region.

Exam point:

> Azure WIF does not support Azure sovereign clouds.

---

# 25\. Simple Summary

**Workload Identity Federation (WIF)** is Snowflake’s **secretless service-to-service authentication** method.

Main points:

* uses cloud-native identity or OIDC
* avoids long-lived secrets
* requires `TYPE = SERVICE`
* supported by major drivers
* works with AWS, Azure, GCP, and OIDC
* Kubernetes uses OIDC tokens with audience `snowflakecomputing.com`
* authentication policies can restrict allowed providers/issuers
* best practice is to reduce identity sprawl

---

# 26\. The 20 Exam Facts to Memorize

1. WIF is a **service-to-service** authentication method
2. WIF uses native cloud identity or OIDC instead of long-lived secrets
3. WIF removes the need for passwords, key pairs, PATs, and API keys
4. WIF uses short-lived attestations/tokens
5. WIF is configured on users with `TYPE = SERVICE`
6. To configure WIF, you need `OWNERSHIP` or `MODIFY PROGRAMMATIC AUTHENTICATION METHODS`
7. Drivers authenticate using `authenticator='WORKLOAD_IDENTITY'`
8. WIF is supported by major Snowflake drivers
9. AWS WIF maps Snowflake user to an IAM `ARN`
10. Azure WIF uses `ISSUER` and `SUBJECT`
11. GCP WIF uses the service account `uniqueId` as `SUBJECT`
12. OIDC WIF uses `TYPE = OIDC`
13. EKS, AKS, and GKE use OIDC-based WIF
14. Kubernetes service account tokens must include audience `snowflakecomputing.com`
15. Custom OIDC providers must expose public OpenID configuration and `jwks_uri`
16. `OIDC_AUDIENCE_LIST` is needed when the token audience is not `snowflakecomputing.com`
17. Snowflake recommends minimizing identity sprawl
18. Impersonation is supported for AWS and GCP, not Azure
19. Authentication policies can restrict allowed WIF providers and issuers
20. Azure sovereign clouds are not supported for Azure workloads using WIF

---

# 27\. Ultra-Short Exam Summary

**Workload Identity Federation (WIF)**

* secretless service-to-service auth
* uses AWS / Azure / GCP / OIDC identities
* requires `TYPE = SERVICE`
* driver uses `authenticator='WORKLOAD_IDENTITY'`
* avoids long-lived credentials
* Kubernetes uses OIDC token with audience `snowflakecomputing.com`
* auth policies can restrict provider and issuer
* best practice: reduce identity sprawl

---

# ===================================

# Snowflake External API Authentication and Secrets

## 1\. Definition

**External API authentication** in Snowflake is used to authenticate to a **service hosted outside Snowflake**.

Because the external service requires authentication, Snowflake supports storing and using credentials securely through a **secret**.

Main idea:

> Snowflake uses \*secrets\* to securely store credentials for calling external services.

---

# 2\. Supported Authentication Methods

Snowflake supports these authentication methods for external API authentication:

* **Basic authentication**
* **OAuth 2.0 code grant flow**
* **OAuth 2.0 client credentials flow**

Exam point:

> External API authentication supports \*Basic Auth\*, \*OAuth code grant\*, and \*OAuth client credentials\*.

---

# 3\. Basic Authentication

Basic authentication uses:

* **username**
* **password**

These credentials are sent in the API request header and encoded with **Base64**.

This follows:

* **RFC 7617**

Exam point:

> Basic authentication in external API auth uses Base64-encoded username/password.

---

# 4\. OAuth for External API Authentication

Snowflake also supports **OAuth 2.0** for external API authentication.

Supported OAuth flows:

* **authorization code grant**
* **client credentials grant**

This follows:

* **RFC 6749**

Exam point:

> External API authentication supports OAuth 2.0 code grant and client credentials flows.

---

# 5\. What a Secret Is

A **secret** is a **schema-level object** in Snowflake that stores sensitive information securely.

A secret can store:

* usernames and passwords
* OAuth credentials
* other sensitive authentication information

Main properties:

* schema-level object
* encrypted
* protected with RBAC
* used by Snowflake components, not exposed directly

Exam point:

> A \*secret\* is a schema-level object that securely stores sensitive information.

---

# 6\. Why Secrets Matter

Secrets provide:

* centralized credential management
* secure storage of sensitive information
* role-based access control
* separation of duties

This means:

* connector users do not need to know the credentials
* only the secret name is used
* sensitive values remain hidden

Exam point:

> Secrets allow secure credential management without exposing raw credentials to users.

---

# 7\. Encryption of Secrets

Secrets are:

* encrypted using the **Snowflake key encryption hierarchy**

Important idea:

* the stored value is encrypted
* even if users describe the secret, the raw sensitive values are not exposed

Exam point:

> Secrets are encrypted using Snowflake’s key hierarchy.

---

# 8\. Who Can Read Secret Contents

After a secret is created:

* only dedicated Snowflake components can read the sensitive information

Examples:

* integrations
* external functions

Users cannot directly see the sensitive data stored inside the secret.

Exam point:

> Secret contents are not exposed to normal users; only specific Snowflake components can read them.

---

# 9\. Secrets and External Functions

An **external function** may need a secret to:

* read stored credentials
* pass them into an API authorization header
* call an external service

Important:

* the binding of the secret to the external function happens during connector installation or setup
* the password or token itself is not exposed to users

Exam point:

> External functions can use secrets to authenticate to external APIs securely.

---

# 10\. Secrets and Security Integrations

For OAuth-based external API authentication:

* a **security integration** enables Snowflake to connect to the external service

So the main objects are:

* **secret** → stores credentials
* **security integration** → enables OAuth connection flow

Exam point:

> OAuth-based external API authentication uses both \*secrets\* and \*security integrations\*.

---

# 11\. Separation of Duties (SoD)

Snowflake secrets support **separation of duties**.

This means:

* one role can manage secrets
* another role can manage connectors or integrations
* users of the connector do not need access to the secret value

Exam point:

> Secrets support separation of duties by hiding credentials from connector users.

---

# 12\. Commands for Managing Secrets

Main commands:

```sql
CREATE SECRET
```

```sql
ALTER SECRET
```

```sql
DESCRIBE SECRET
```

```sql
DROP SECRET
```

```sql
SHOW SECRETS
```

Exam point:

> Main secret commands are `CREATE`, `ALTER`, `DESCRIBE`, `DROP`, and `SHOW SECRET`.

---

# 13\. Secret Privileges

Snowflake supports these main privileges for secrets:

* **CREATE**
* **USAGE**
* **OWNERSHIP**

Meaning:

## CREATE

* create a new secret in a schema

## USAGE

* use a secret
* required to describe/show/use the secret

## OWNERSHIP

* full control
* required for most modifications

Exam point:

> Secret privilege model mainly uses \*CREATE\*, \*USAGE\*, and \*OWNERSHIP\*.

---

# 14\. Privilege Requirements by Operation

## CREATE SECRET

Requires:

* `USAGE` on parent database
* `USAGE` on parent schema
* `CREATE SECRET` in the schema

## ALTER SECRET

Requires:

* `OWNERSHIP` on the secret

## DROP SECRET

Requires:

* `OWNERSHIP` on the secret

## DESCRIBE SECRET

Requires:

* `USAGE` on the secret

## SHOW SECRETS

Requires:

* `USAGE` on the secret

## USE SECRET

Requires:

* `USAGE` on the secret

Exam point:

> `OWNERSHIP` is needed to alter or drop a secret; `USAGE` is needed to use/describe/show it.

---

# 15\. USE SECRET and External Functions

`USAGE` on a secret is important when using external functions.

It is required for:

* the role creating the external function
* the role calling the external function at query runtime

Exam point:

> `USAGE` on the secret is required when the secret is used with an external function.

---

# 16\. Secrets Are Schema-Level Objects

Important classification:

> A secret is a \*schema-level object\*.

That means:

* it lives inside a schema
* operations on it require privileges on parent database and schema

Exam point:

> Secrets are schema-level objects, so schema/database privileges matter.

---

# 17\. Main Use Cases

Secrets are commonly used with:

* Snowflake connectors
* external functions
* external access integrations
* external API authentication to services outside Snowflake

Exam point:

> Secrets are mainly used when Snowflake needs to authenticate to external services.

---

# 18\. Replication

Snowflake supports:

* **replication of secrets** using account replication

Exam point:

> Secrets can be replicated.

---

# 19\. Simple Summary

Snowflake **external API authentication** is used when Snowflake needs to authenticate to an external service.

Main pieces:

* **Basic Auth** or **OAuth**
* credentials stored in a **secret**
* secret protected by **RBAC**
* secret encrypted with Snowflake key hierarchy
* secrets often used with **external functions**, **connectors**, and **integrations**

---

# 20\. The 15 Exam Facts to Memorize

1. External API authentication is used to authenticate to services outside Snowflake
2. Snowflake supports **Basic authentication**
3. Snowflake supports **OAuth code grant flow**
4. Snowflake supports **OAuth client credentials flow**
5. Basic authentication uses Base64-encoded username/password
6. A **secret** is a schema-level object
7. Secrets store sensitive authentication information securely
8. Secrets are protected by RBAC
9. Secrets are encrypted using the Snowflake key encryption hierarchy
10. Secret values are not exposed through normal inspection
11. External functions can use secrets to authenticate external API requests
12. OAuth-based external API auth uses a **security integration**
13. Main secret commands are `CREATE`, `ALTER`, `DESCRIBE`, `DROP`, `SHOW`
14. `USAGE` is required to use/describe/show a secret
15. Secrets can be replicated with account replication

---

# 21\. Ultra-Short Exam Summary

**External API Authentication and Secrets**

* used for calling services outside Snowflake
* auth methods: **Basic**, **OAuth code grant**, **OAuth client credentials**
* credentials stored in **secrets**
* secret = **schema-level object**
* encrypted + RBAC protected
* used by external functions/connectors/integrations
* `OWNERSHIP` to alter/drop, `USAGE` to use/describe/show
* secrets support replication

---





# ===================================

# Snowflake Network Security — COF-C03 Quick Revision

## 1\. Malicious IP Protection

### Definition

**Malicious IP Protection** is a Snowflake service that blocks login/network access attempts coming from IP addresses on a curated threat-intelligence list.

Main goal:

> Automatically block known bad IP addresses to reduce unauthorized access risk.

### IP Categories

Snowflake categorizes malicious IPs into:

* `ANONYMOUS_VPN`
* `ANONYMOUS_PROXIES`
* `MALICIOUS_BEHAVIOR`
* `TOR_EXITS`

By default:

* **all categories are blocked**

### Risk Levels

Blocked IPs can appear as:

* **LOW risk**
* **HIGH risk**

Important:

* you can **opt out only for low-risk categories**
* you **cannot opt out of high-risk categories**

### View Blocked Logins

Use:

```sql
SELECT *
  FROM SNOWFLAKE.ACCOUNT_USAGE.LOGIN_HISTORY
  WHERE NOT is_success AND login_details IS NOT NULL
  ORDER BY event_timestamp DESC;
```

Look at:

* `is_success`
* `login_details`
* `ip_address`

### Opt Out Example

```sql
USE ROLE ACCOUNTADMIN;
SELECT SYSTEM$OPT_OUT_MALICIOUS_IP_PROTECTION_BY_CATEGORY('MALICIOUS_BEHAVIOR');
```

Re-enable all blocking:

```sql
USE ROLE ACCOUNTADMIN;
SELECT SYSTEM$OPT_OUT_MALICIOUS_IP_PROTECTION_BY_CATEGORY('');
```

Exam points:

* Malicious IP Protection blocks known bad IPs automatically
* All categories are blocked by default
* Only **low-risk** categories can be opted out
* Blocked attempts appear in `LOGIN_HISTORY`

---

# 2\. Network Policies

## Definition

A **network policy** controls **inbound access** to:

* the Snowflake service
* internal stages

It allows or blocks requests based on their origin.

Main idea:

> Network policies control who can connect to Snowflake based on network origin.

---

## 3\. Network Rules vs Network Policies

### Network Rule

A **network rule** is a **schema-level object** that groups network identifiers into a logical unit.

A network rule does **not** say allow or block by itself.

### Network Policy

A **network policy** uses network rules in:

* `ALLOWED_NETWORK_RULE_LIST`
* `BLOCKED_NETWORK_RULE_LIST`

So:

> Network rule = list of identifiers  
> Network policy = decision to allow/block them

Exam point:

* **Network rules are schema-level objects**
* **Network policies enforce access**

---

## 4\. Workflow for Inbound Traffic Control

General workflow:

1. create network rules
2. create network policies using those rules
3. activate the network policy on:

   * account
   * user
   * security integration

Important:

> A network policy has no effect until it is activated.

---

## 5\. Supported Network Identifiers

### Incoming traffic

Supported identifiers include:

* `IPV4`
* `AWSVPCEID`
* `AZURELINKID`
* `GCPPSCID`

### Outgoing traffic

For egress rules, supported identifiers include:

* domains and ports

Exam point:

* `IPV4` is for IP ranges
* private connectivity identifiers include AWS VPCE IDs, Azure LinkIDs, GCP PSC IDs

---

## 6\. Network Rule Modes

Network rules use a `MODE` to indicate traffic direction.

### `INGRESS`

Used for incoming traffic to Snowflake.

### `INTERNAL_STAGE`

Used for AWS internal stage access only.

### `EGRESS`

Used for traffic leaving Snowflake.

Exam point:

* `INGRESS` = incoming access
* `EGRESS` = outgoing access
* `INTERNAL_STAGE` = AWS internal stage only

---

## 7\. Allowed vs Blocked Lists

Important behavior:

* if an identifier is in the **allowed list**, only those allowed identifiers can connect
* blocked list is used for explicit deny rules

If the same identifier appears in both allowed and blocked lists:

> **blocked wins**

This applies to both:

* `ALLOWED_IP_LIST` / `BLOCKED_IP_LIST`
* `ALLOWED_NETWORK_RULE_LIST` / `BLOCKED_NETWORK_RULE_LIST`

Exam point:

* **Blocked list takes precedence over allowed list**

---

## 8\. Private Connectivity Precedence

Private connectivity network rules take precedence over IPv4 rules.

Types:

* `AWSVPCEID`
* `AZURELINKID`

If incoming request uses private connectivity and matching private endpoint rule exists:

* IPv4 rules are ignored for that traffic

Exam point:

> Private endpoint identifiers take precedence over IPv4 rules.

---

## 9\. Network Policy Precedence

You can apply network policies at:

* account level
* user level
* security integration level

Order of precedence:

1. **Security integration**
2. **User**
3. **Account**

Most specific wins.

Exam point:

> Security integration policy overrides user policy, which overrides account policy.

---

## 10\. Security Policy Precedence

Among security policies, evaluation order is:

1. **Network policies**
2. **Authentication policies**
3. **Password policies**
4. **Session policies**

Exam point:

> Network policies are evaluated before authentication policies.

---

## 11\. Account-Level Network Policy

Set on the account:

```sql
ALTER ACCOUNT SET NETWORK_POLICY = my_policy;
```

Effect:

* applies to all users unless overridden
* users already connected but not allowed by policy cannot continue querying

Important:

* only one network policy can be active on an account at a time

Exam point:

* account-level policy is the most general policy

---

## 12\. User-Level Network Policy

Set on a user:

```sql
ALTER USER joe SET NETWORK_POLICY = my_policy;
```

Effect:

* applies specifically to that user
* overrides account-level policy

Exam point:

* user-level policy is more specific than account-level policy

---

## 13\. Security Integration-Level Network Policy

Supported by:

* SCIM
* Snowflake OAuth
* External OAuth

Example:

```sql
CREATE SECURITY INTEGRATION oauth_kp_int
  TYPE = oauth
  ENABLED = true
  OAUTH_CLIENT = custom
  OAUTH_CLIENT_TYPE = 'CONFIDENTIAL'
  OAUTH_REDIRECT_URI = 'https://example.com'
  NETWORK_POLICY = mypolicy;
```

Important:

* integration-level policy is the **most specific**
* does **not** restrict internal stage access

Exam point:

* integration-level network policy is the highest-precedence network policy

---

## 14\. Internal Stage Protection

### AWS

You can protect internal stages on AWS using network rules.

Prerequisite:

```sql
ALTER ACCOUNT SET ENFORCE_NETWORK_RULES_FOR_INTERNAL_STAGES = true;
```

Important:

* internal stage restrictions do not work until this parameter is enabled

### Azure

You cannot use a network rule to restrict internal stage access directly on Azure.

Exam point:

* `ENFORCE_NETWORK_RULES_FOR_INTERNAL_STAGES` is required for AWS internal stage protection
* Azure does not support internal stage restriction with network rules the same way

---

## 15\. Internal Stage Modes

To protect only AWS internal stage:

* `MODE = INTERNAL_STAGE`
* `TYPE = AWSVPCEID`

Important:

* `INTERNAL_STAGE` is AWS-specific
* to restrict by VPCE ID for internal stage, use a separate rule

Exam point:

* `INTERNAL_STAGE` requires `TYPE = AWSVPCEID`

---

## 16\. Best Practices for Network Rules

Snowflake recommends:

* create **small, targeted** network rules
* avoid large monolithic lists
* add comments
* narrow scope whenever possible
* prefer network rules over old `ALLOWED_IP_LIST` and `BLOCKED_IP_LIST`

Exam point:

> Best practice is to use **network rules**, not legacy IP list parameters.

---

## 17\. Legacy IP Lists

Older network policies can still use:

* `ALLOWED_IP_LIST`
* `BLOCKED_IP_LIST`

But:

> all new network policies should use **network rules**

Exam point:

* legacy IP lists still work, but network rules are the recommended approach

---

## 18\. Creating a Network Rule

Example:

```sql
CREATE NETWORK RULE cloud_network
  TYPE = IPV4
  MODE = INGRESS
  VALUE_LIST = ('47.88.25.32/27');
```

To create a network rule, need:

* `CREATE NETWORK RULE` on schema

Exam point:

* network rule is created with `TYPE`, `MODE`, and `VALUE_LIST`

---

## 19\. Modifying a Network Rule

You can modify:

* identifiers
* comment

You cannot modify:

* type
* mode
* name
* schema

Exam point:

* network rule `TYPE` and `MODE` cannot be changed after creation

---

## 20\. Creating a Network Policy

Example:

```sql
CREATE NETWORK POLICY public_network_policy
  ALLOWED_NETWORK_RULE_LIST = ('allow_access_rule')
  BLOCKED_NETWORK_RULE_LIST = ('block_access_rule');
```

Only security admins or higher, or role with global create privilege, can create network policies.

Important:

* an empty network policy blocks all IPv4 access

Exam point:

* network policies use allowed and blocked network rule lists
* empty policy can lock you out

---

## 21\. Modifying a Network Policy

Examples:

Replace blocked rules:

```sql
ALTER NETWORK POLICY my_policy
  SET BLOCKED_NETWORK_RULE_LIST = ('other_network');
```

Add allowed rule:

```sql
ALTER NETWORK POLICY my_policy
  ADD ALLOWED_NETWORK_RULE_LIST = ('new_rule');
```

Remove blocked rule:

```sql
ALTER NETWORK POLICY my_policy
  REMOVE BLOCKED_NETWORK_RULE_LIST = ('other_network');
```

Exam point:

* `SET` replaces list
* `ADD` appends
* `REMOVE` deletes entries

---

## 22\. Identifying Active Network Policies

You can identify active network policies using:

* Snowsight
* `POLICY_REFERENCES`
* `NETWORK_POLICIES` view
* `SHOW PARAMETERS LIKE 'network_policy' IN ACCOUNT`

Exam point:

* `POLICY_REFERENCES` is useful for seeing where a policy is applied

---

## 23\. Network Rule Replication

Network rules are schema-level objects, so they replicate with their database.

Network policies and their assignments also support:

* replication
* failover/failback

Exam point:

* network rules and network policies support replication

---

# 24\. Snowflake-Managed Network Rules

Snowflake provides built-in managed network rules in:

```text
SNOWFLAKE.NETWORK_SECURITY
```

These are maintained automatically by Snowflake.

Examples include partner applications such as:

* dbt platform
* Microsoft Power BI
* Qlik
* Tableau
* GitHub Actions

Main benefit:

> Snowflake keeps the partner egress IP ranges up to date automatically.

Exam point:

* Snowflake-managed rules are in `SNOWFLAKE.NETWORK_SECURITY`

---

## 25\. Viewing Snowflake-Managed Rules

List managed rules:

```sql
SHOW NETWORK RULES IN SNOWFLAKE.NETWORK_SECURITY;
```

Query their details:

```sql
SELECT *
  FROM SNOWFLAKE.ACCOUNT_USAGE.NETWORK_RULES
  WHERE DATABASE = 'SNOWFLAKE' AND SCHEMA = 'NETWORK_SECURITY';
```

Exam point:

* managed network rules can be referenced directly in a network policy

---

## 26\. Snowflake-Managed Egress Rule

Snowflake provides a built-in egress rule:

```text
SNOWFLAKE.EXTERNAL_ACCESS.PYPI_RULE
```

Use case:

* allow external access to **PyPI**
* useful for package installation in Snowpark Container / notebooks

Exam point:

* `SNOWFLAKE.EXTERNAL_ACCESS.PYPI_RULE` is a built-in Snowflake-managed egress network rule

---

# 27\. Snowflake Egress IP Addresses

Snowflake can generate egress IP address ranges for use in your external firewall.

Supported use cases include:

* external access from UDFs and procedures
* Snowpark Container Services external access
* Openflow on Snowpark Container Services

Use:

```sql
SYSTEM$GET_SNOWFLAKE_EGRESS_IP_RANGES()
```

Exam point:

* use `SYSTEM$GET_SNOWFLAKE_EGRESS_IP_RANGES` to retrieve Snowflake egress IP ranges

---

## 28\. Egress IP Range Lifecycle

Generated egress IP ranges have:

* an **effective** time
* an **expiration** time

Important:

* egress IPs expire
* firewall rules must be refreshed regularly

Exam point:

* Snowflake egress IP ranges are not permanent and must be refreshed

---

## 29\. Automating Egress IP Refresh

Recommended process:

1. call `SYSTEM$GET_SNOWFLAKE_EGRESS_IP_RANGES`
2. compare new vs existing ranges
3. update firewall if changed
4. schedule regular refreshes
5. log updates and failures

Typical automation tools:

* Python / Bash / PowerShell
* AWS / Azure / GCP APIs or CLIs
* Terraform / Ansible / CloudFormation

Exam point:

* Snowflake recommends automating egress IP refresh because ranges expire

---

# 30\. Ultra-Short Exam Summary

## Malicious IP Protection

* blocks known bad IPs automatically
* categories include VPN, proxies, malware behavior, Tor exits
* all categories blocked by default
* only low-risk categories can be opted out

## Network Policies

* control inbound access to Snowflake
* use network rules
* blocked list overrides allowed list
* precedence: security integration > user > account
* network policies are checked before auth policies

## Network Rules

* schema-level objects
* group identifiers such as IPv4, VPCE IDs, LinkIDs, PSC IDs, domains
* modes: `INGRESS`, `INTERNAL_STAGE`, `EGRESS`

## Internal Stage

* AWS supported with `ENFORCE_NETWORK_RULES_FOR_INTERNAL_STAGES = true`
* Azure internal stage restrictions differ

## Managed Rules

* Snowflake-managed rules exist in `SNOWFLAKE.NETWORK_SECURITY`
* built-in egress rule: `SNOWFLAKE.EXTERNAL_ACCESS.PYPI_RULE`

## Egress IPs

* generated using `SYSTEM$GET_SNOWFLAKE_EGRESS_IP_RANGES`
* expire and should be refreshed automatically

---

# 31\. The 20 Exam Facts to Memorize

1. Malicious IP Protection blocks known bad IP addresses automatically
2. Malicious IP categories include `ANONYMOUS_VPN`, `ANONYMOUS_PROXIES`, `MALICIOUS_BEHAVIOR`, `TOR_EXITS`
3. Only low-risk malicious IP categories can be opted out
4. Network policies control inbound access to Snowflake service and internal stages
5. Network rules are schema-level objects
6. Network rules group identifiers; they do not themselves allow or block traffic
7. Network policies use `ALLOWED_NETWORK_RULE_LIST` and `BLOCKED_NETWORK_RULE_LIST`
8. Blocked rules override allowed rules
9. Private endpoint identifiers take precedence over IPv4 rules
10. Network policy precedence is: security integration > user > account
11. Network policies are evaluated before authentication policies
12. `INGRESS` is for incoming traffic
13. `EGRESS` is for outgoing traffic
14. `INTERNAL_STAGE` is for AWS internal stages
15. `ENFORCE_NETWORK_RULES_FOR_INTERNAL_STAGES = true` is required to protect AWS internal stages
16. New network policies should use network rules, not legacy IP list parameters
17. Snowflake-managed partner rules are in `SNOWFLAKE.NETWORK_SECURITY`
18. `SNOWFLAKE.EXTERNAL_ACCESS.PYPI_RULE` is a built-in egress rule
19. Snowflake egress IP ranges are retrieved with `SYSTEM$GET_SNOWFLAKE_EGRESS_IP_RANGES`
20. Snowflake egress IP ranges expire and should be refreshed automatically

---

========================================================================

#  The Private connectivity for inbound network traffic

\#

**Private connectivity requires Business Critical edition or higher.**

Exam point:

> Private connectivity is a Business Critical (or higher) feature.

---

# 2\. Definition

Snowflake connections can use either:

* the **public Internet**
* or **private connectivity** through private IP addresses in the cloud platform

Private connectivity improves security by routing traffic privately between your environment and Snowflake.

Main idea:

> Private connectivity hardens security by avoiding public Internet routing.

---

# 3\. Supported Cloud Private Connectivity Services

Private connectivity depends on the cloud provider hosting the Snowflake account:

* **AWS PrivateLink**
* **Azure Private Link**
* **Google Cloud Private Service Connect (PSC)**

Exam point:

> Snowflake private connectivity uses the cloud provider’s native private endpoint service.

---

# 4\. Main Benefit

Using private connectivity:

* reduces exposure to the public Internet
* hardens inbound network security
* uses private endpoints between customer network and Snowflake network

Exam point:

> Private connectivity is used to improve the security posture of inbound access to Snowflake.

---

# 5\. Features That Support Private Connectivity

Private connectivity can be used for access to:

* **Snowflake Service**
* **Snowsight**
* **Streamlit in Snowflake**
* **internal stages**
* **Snowpark Container Services**
* **Snowflake Intelligence**

Exam point:

> Private connectivity supports multiple Snowflake features, not just the core Snowflake service.

---

# 6\. Private Connectivity to the Snowflake Service

Private connectivity to the Snowflake Service means:

* traffic goes from your **VPC/VNet**
* to the **Snowflake VPC/VNet**
* through a **private IP path**

Cloud-specific technologies:

* AWS PrivateLink
* Azure Private Link
* Google Private Service Connect

Exam point:

> Private connectivity to the Snowflake Service is private routing from customer network to Snowflake network.

---

# 7\. Private Connectivity to Snowsight

Snowsight can be accessed over private connectivity after configuration.

Main idea:

* configure private connectivity for Snowsight
* users can then sign in privately

Exam point:

> Snowsight supports private connectivity after specific configuration.

---

# 8\. Private Connectivity to Streamlit in Snowflake

Streamlit in Snowflake also supports private connectivity using:

* AWS PrivateLink
* Azure Private Link
* Google Private Service Connect

Exam point:

> Streamlit in Snowflake supports private connectivity.

---

# 9\. Private Connectivity to Internal Stages

Snowflake internal stages can be accessed through private connectivity.

Supported through cloud-specific private endpoint options:

* AWS VPC interface endpoints
* Azure private endpoints
* Google Private Service Connect endpoints

Exam point:

> Internal stages support private connectivity.

---

# 10\. Private Connectivity to Snowpark Container Services

Snowpark Container Services supports private connectivity for inbound connections.

Exam point:

> Snowpark Container Services supports private connectivity.

---

# 11\. Private Connectivity to Snowflake Intelligence

Snowflake Intelligence also supports private connectivity.

Exam point:

> Snowflake Intelligence supports private connectivity.

---

# 12\. Simple Summary

Snowflake private connectivity:

* requires **Business Critical or higher**
* routes traffic over **private cloud networking**
* avoids using the public Internet
* uses:

  * AWS PrivateLink
  * Azure Private Link
  * Google Private Service Connect

* supports:

  * Snowflake Service
  * Snowsight
  * Streamlit in Snowflake
  * internal stages
  * Snowpark Container Services
  * Snowflake Intelligence

---

# 13\. The 10 Exam Facts to Memorize

1. Private connectivity requires **Business Critical or higher**
2. It routes traffic privately instead of over the public Internet
3. AWS uses **PrivateLink**
4. Azure uses **Private Link**
5. GCP uses **Private Service Connect**
6. The Snowflake Service supports private connectivity
7. Snowsight supports private connectivity
8. Internal stages support private connectivity
9. Snowpark Container Services supports private connectivity
10. Snowflake Intelligence supports private connectivity

---

# 14\. Ultra-Short Exam Summary

**Snowflake Private Connectivity**

* **Business Critical+** feature
* private route instead of public Internet
* AWS = **PrivateLink**
* Azure = **Private Link**
* GCP = **Private Service Connect**
* supported for Snowflake Service, Snowsight, Streamlit, internal stages, Snowpark Container Services, and Snowflake Intelligence

---

===============================================================================================

# Snowflake Outbound Private Connectivity



## 1\. Edition Requirement

**Outbound private connectivity requires Business Critical edition (or higher).**

Exam point:

> Private connectivity (inbound or outbound) is a **Business Critical feature**.

---

# 2\. Definition

Outbound private connectivity is used when **Snowflake sends traffic to external cloud services**.

Instead of using the public Internet, Snowflake connects through:

* **private cloud endpoints**
* using the cloud provider’s private networking service

Main idea:

> Outbound private connectivity secures traffic from Snowflake to external resources.

---

# 3\. Why Use Outbound Private Connectivity

Benefits:

* improves security posture
* avoids public Internet exposure
* enables private access to cloud services from Snowflake

Exam point:

> Used when Snowflake features generate outbound network traffic.

---

# 4\. Snowflake Features That Support Outbound Private Connectivity

Outbound private connectivity is supported by:

* External network locations (external access integrations)
* External functions
* External stages
* External tables
* Apache Iceberg catalog integrations
* Apache Iceberg external volumes
* Apache Iceberg REST catalog integrations
* Snowpipe automation

Exam point:

> Many data-integration features support outbound private connectivity.

---

# 5\. Cost Model

You pay for:

* each private endpoint
* amount of data processed

Billing service types:

* **OUTBOUND\_PRIVATELINK\_ENDPOINT**
* **OUTBOUND\_PRIVATELINK\_DATA\_PROCESSED**

Exam point:

> Outbound private connectivity has **endpoint + data usage cost**.

---

# 6\. Basic Workflow

Typical setup steps:

1. Configure prerequisite feature settings
2. Provision private endpoint in Snowflake
3. Authorize the endpoint
4. Retrieve endpoint URL
5. Configure Snowflake feature to use endpoint
6. Deprovision unused endpoints

Exam point:

> Configuration usually involves endpoint creation + authorization + integration.

---

# 7\. Scaling Limits

Important limitations:

* Maximum **5 private endpoints per Snowflake account**
* Recently deleted endpoints (within 7 days) still count
* Only **one endpoint per AWS service or Azure subresource**

Exam point:

> Outbound private connectivity has strict endpoint limits.

---

# 8\. Cloud-Specific Behavior

AWS:

* limitation is **per service**
* example: cannot create multiple endpoints to different S3 buckets

Azure:

* limitation is **per subresource**
* multiple endpoints possible only if subresources differ

Exam point:

> Endpoint duplication rules differ by cloud provider.

---

# 9\. Government Region Support

Supported:

* Azure Government regions
* AWS GovCloud regions

Condition:

* both Snowflake region and target service must be in the same government cloud

Exam point:

> Outbound private connectivity is supported in government cloud regions.

---

# 10\. External Network Access with Snowpark

Outbound private connectivity can be used for:

* Snowpark UDFs / stored procedures
* Snowpark Container Services external network calls

Exam point:

> Snowpark workloads can securely call external services using private connectivity.

---

# 11\. External Stages and External Tables

If an external stage is configured with private connectivity:

* all access to that stage
* and related external tables

uses private connectivity.

Exam point:

> External tables inherit private connectivity from the external stage.

---

# 12\. Apache Iceberg Integration Support

Outbound private connectivity supports:

* Iceberg REST catalog integrations
* Iceberg external volumes

Exam point:

> Apache Iceberg integrations can use private connectivity.

---

# 13\. Snowpipe Automation

Snowpipe automation can use outbound private connectivity when:

* accessing external stages
* especially in Azure environments

Exam point:

> Snowpipe can ingest data securely through private endpoints.

---

# 14\. Ultra-Short Exam Summary

**Outbound Private Connectivity**

* Business Critical feature
* Secures Snowflake → external service traffic
* Uses private endpoints (AWS PrivateLink / Azure Private Link / GCP PSC)
* Supported by external functions, stages, tables, Snowpipe, Iceberg
* Cost = endpoint + data processed
* Limit = **5 endpoints per account**

---

# ===========================================================================

# Snowflake Trust Center — 

# 1\. Definition

**Trust Center** is a Snowflake security feature used to:

* evaluate security risks
* monitor account security posture
* provide remediation recommendations

It uses **scanners** to analyze configuration and activity.

Exam point:

> Trust Center helps identify and reduce security risks in Snowflake accounts.

---

# 2\. Key Concepts

### Scanners

* Background processes that check security risks
* Based on:

  * account configuration
  * authentication risks
  * anomalous activity

Scanners generate **findings**.

---

### Findings Types

#### Violations

* Persistent configuration issues
* Continue appearing until fixed

Examples:

* MFA not enforced
* too many ACCOUNTADMIN users

Exam point:

> Violations = ongoing misconfiguration

---

#### Detections

* One-time security events
* Represent something that happened at a specific time

Examples:

* login from unknown IP
* large data transfer

Exam point:

> Detections = event-based security alerts

---

# 3\. Trust Center Tabs

### Overview tab

* High-level security posture summary
* shows scanner findings

### Violations tab

* configuration issues
* shows severity (low → critical)
* can:

  * resolve
  * reopen
  * add justification

### Detections tab

* event-based findings
* cannot remediate directly
* used for investigation

### Organization tab

* org-level violation insights
* requires org account access

---

# 4\. Common Trust Center Use Cases

* enforce MFA for human users
* detect over-privileged roles
* limit ACCOUNTADMIN / SECURITYADMIN users
* find inactive users
* detect anomalous access

Exam point:

> Trust Center helps with governance + authentication risk monitoring.

---

# 5\. Required Roles

To use Trust Center:

Application roles:

* **SNOWFLAKE.TRUST\_CENTER\_VIEWER**
* **SNOWFLAKE.TRUST\_CENTER\_ADMIN**

Notes:

* ACCOUNTADMIN grants these roles
* Organization account uses GLOBALORGADMIN

Exam point:

> Trust Center access is controlled by application roles.

---

# 6\. Scanner Packages

Trust Center groups scanners into **scanner packages**.

Main packages:

### Security Essentials (default enabled)

Checks:

* MFA enforcement
* trusted IP network policy
* event table configuration

Characteristics:

* runs automatically
* no serverless cost on scheduled runs

Exam point:

> Security Essentials package = enabled by default.

---

### CIS Benchmarks package

* evaluates account against **CIS security best practices**
* runs daily by default
* schedule can be changed

Exam point:

> CIS package = optional advanced security insights.

---

### Threat Intelligence package

Detects:

* risky users
* authentication failures
* abnormal login patterns
* suspicious activity

Includes:

* schedule-based scanners
* event-driven scanners

Exam point:

> Threat Intelligence focuses on behavior-based risk detection.

---

# 7\. Schedule-Based vs Event-Driven Scanners

### Schedule-based scanners

* run periodically
* detect configuration state

### Event-driven scanners

* triggered by events
* detect security incidents faster

Exam point:

> Event-driven scanners can detect events that scheduled scans may miss.

---

# 8\. Organization vs Account Findings

* Organization view → violations summary
* Account view → full violation + detection details

Important:

* detections cannot be viewed at organization level

---

# 9\. Notifications and Lifecycle

* violations can be marked **resolved**
* resolved violations stop notifications
* detections cannot be lifecycle-managed

Exam point:

> Only violations have lifecycle management.

---

# 10\. Limitations

* Not supported for reader accounts

---

# 11\. Ultra-Short Exam Summary

**Trust Center**

* security monitoring tool
* uses scanners
* findings = violations + detections
* Security Essentials enabled by default
* CIS + Threat Intelligence optional
* roles required: TRUST\_CENTER\_VIEWER / ADMIN

---



# ===============================================
# Snowflake Sessions & Session Policies

## 1. Edition Requirement

**Session policies require Enterprise Edition (or higher).**

Exam point:

> Session policy = **Enterprise feature**.

---

# 2. What is a Snowflake Session

A **session starts** when:

- user connects to Snowflake  
- authentication succeeds  

A session is:

- independent of IdP session  
- maintained while user activity continues  

If Snowflake session expires but IdP session is active →  
**silent authentication is possible (no credentials needed).**

Exam point:

> Snowflake session ≠ Identity Provider session.

---

# 3. Idle Session Timeout

- Session ends after inactivity  
- Default timeout = **240 minutes (4 hours)**  
- Maximum default timeout = **4 hours**

Applies to:

- Snowsight  
- Snowflake CLI  
- SnowSQL  
- Drivers / connectors  
- Third-party clients  

Exam point:

> Default session timeout = **4 hours**.

---

# 4. Session Expiration Causes (Snowsight)

Session can end due to:

- idle timeout  
- browser restart  
- authentication cookie expiration (~24h)  
- network policy restriction  
- network disconnection  

Important:

- running queries may be terminated after session ends  

---

# 5. Monitoring Sessions

You can monitor sessions using:

- **ACCOUNT_USAGE.SESSIONS view**
- Snowsight → Sessions tab  

Information available:

- session ID  
- user  
- start time  
- client driver  
- IP address  
- authentication method  

Exam point:

> Session monitoring uses **SESSIONS view**.

---

# 6. Session Policies

A **session policy** controls:

- idle timeout duration  
- allowed secondary roles  

Minimum configurable timeout:

- **5 minutes**

Default if no policy:

- **240 minutes**

Exam point:

> Session policy overrides default timeout.

---

# 7. Account vs User Session Policy

Session policy can be applied to:

- account  
- specific user  

Precedence rule:

> **User-level session policy overrides account policy.**

---

# 8. Session Policy Parameters

### Idle timeout parameters

- **SESSION_IDLE_TIMEOUT_MINS**
  → programmatic clients  

- **SESSION_UI_IDLE_TIMEOUT_MINS**
  → Snowsight  

Exam point:

> Separate timeout settings for UI vs programmatic access.

---

# 9. Secondary Roles Control

Users can activate secondary roles in a session:

```sql
USE SECONDARY ROLES;
```

Security admins can restrict this using session policy:

```
ALLOWED_SECONDARY_ROLES
```

Examples:

- empty list → disables secondary roles  
- specific roles → limits privilege scope  

Important:

- enforcement applies immediately  
- affects existing sessions  

Exam point:

> Session policies help control **privilege scope during sessions**.

---

# 10. CLIENT_SESSION_KEEP_ALIVE Option

If enabled:

- session remains active indefinitely  
- requires active connection  

Risks:

- many open sessions  
- potential performance impact  

Exam point:

> Avoid CLIENT_SESSION_KEEP_ALIVE unless necessary.

---

# 11. Heartbeat Frequency

Parameter:

```
CLIENT_SESSION_KEEP_ALIVE_HEARTBEAT_FREQUENCY
```

Defines:

- how often client refreshes session token  

---

# 12. Worksheet Behavior

- opening or creating worksheet resets idle timeout  
- session reused across worksheets  

---

# 13. Limitation

- **Future grants not supported on session policies**

Workaround:

- grant **APPLY SESSION POLICY** privilege to custom role  

---

# 14. Ultra-Short Exam Summary

**Session**

- starts after successful authentication  
- default timeout = **4 hours**  
- can expire due to inactivity or browser/session issues  

**Session Policy**

- Enterprise feature  
- controls timeout + secondary roles  
- user policy overrides account policy  

---

# =========================================================================
# Snowflake Sessions & Session Policies

## 1. Edition Requirement

**Session policies require Enterprise Edition (or higher).**

Exam point:

> Session policy = **Enterprise feature**.

---

# 2. What is a Snowflake Session

A **session starts** when:

- user connects to Snowflake  
- authentication succeeds  

A session is:

- independent of IdP session  
- maintained while user activity continues  

If Snowflake session expires but IdP session is active →  
**silent authentication is possible (no credentials needed).**

Exam point:

> Snowflake session ≠ Identity Provider session.

---

# 3. Idle Session Timeout

- Session ends after inactivity  
- Default timeout = **240 minutes (4 hours)**  
- Maximum default timeout = **4 hours**

Applies to:

- Snowsight  
- Snowflake CLI  
- SnowSQL  
- Drivers / connectors  
- Third-party clients  

Exam point:

> Default session timeout = **4 hours**.

---

# 4. Session Expiration Causes (Snowsight)

Session can end due to:

- idle timeout  
- browser restart  
- authentication cookie expiration (~24h)  
- network policy restriction  
- network disconnection  

Important:

- running queries may be terminated after session ends  

---

# 5. Monitoring Sessions

You can monitor sessions using:

- **ACCOUNT_USAGE.SESSIONS view**
- Snowsight → Sessions tab  

Information available:

- session ID  
- user  
- start time  
- client driver  
- IP address  
- authentication method  

Exam point:

> Session monitoring uses **SESSIONS view**.

---

# 6. Session Policies

A **session policy** controls:

- idle timeout duration  
- allowed secondary roles  

Minimum configurable timeout:

- **5 minutes**

Default if no policy:

- **240 minutes**

Exam point:

> Session policy overrides default timeout.

---

# 7. Account vs User Session Policy

Session policy can be applied to:

- account  
- specific user  

Precedence rule:

> **User-level session policy overrides account policy.**

---

# 8. Session Policy Parameters

### Idle timeout parameters

- **SESSION_IDLE_TIMEOUT_MINS**
  → programmatic clients  

- **SESSION_UI_IDLE_TIMEOUT_MINS**
  → Snowsight  

Exam point:

> Separate timeout settings for UI vs programmatic access.

---

# 9. Secondary Roles Control

Users can activate secondary roles in a session:

```sql
USE SECONDARY ROLES;
```

Security admins can restrict this using session policy:

```
ALLOWED_SECONDARY_ROLES
```

Examples:

- empty list → disables secondary roles  
- specific roles → limits privilege scope  

Important:

- enforcement applies immediately  
- affects existing sessions  

Exam point:

> Session policies help control **privilege scope during sessions**.

---

# 10. CLIENT_SESSION_KEEP_ALIVE Option

If enabled:

- session remains active indefinitely  
- requires active connection  

Risks:

- many open sessions  
- potential performance impact  

Exam point:

> Avoid CLIENT_SESSION_KEEP_ALIVE unless necessary.

---

# 11. Heartbeat Frequency

Parameter:

```
CLIENT_SESSION_KEEP_ALIVE_HEARTBEAT_FREQUENCY
```

Defines:

- how often client refreshes session token  

---

# 12. Worksheet Behavior

- opening or creating worksheet resets idle timeout  
- session reused across worksheets  

---

# 13. Limitation

- **Future grants not supported on session policies**

Workaround:

- grant **APPLY SESSION POLICY** privilege to custom role  

---

# 14. Ultra-Short Exam Summary

**Session**

- starts after successful authentication  
- default timeout = **4 hours**  
- can expire due to inactivity or browser/session issues  

**Session Policy**

- Enterprise feature  
- controls timeout + secondary roles  
- user policy overrides account policy  

---

# ==================
# Snowflake SCIM Support 

## 1. What is SCIM in Snowflake

Snowflake supports **SCIM 2.0 (System for Cross-domain Identity Management).**

Purpose:

- integrate Snowflake with Identity Providers (IdP)
- automatically provision users and groups (roles)

Snowflake acts as:

> **Service Provider**

Identity Provider examples:

- Okta  
- Microsoft Entra ID (Azure AD)  
- Custom IdP  

Exam point:

> SCIM enables **automatic user & role provisioning from IdP to Snowflake**.

---

# 2. SCIM Provisioning Concept

Provisioning means:

- creating users in Snowflake from IdP  
- creating roles (groups) in Snowflake from IdP  

Mapping:

- **User → one-to-one mapping**
- **Group → Snowflake role**

Important:

> Changes in IdP sync to Snowflake  
> Changes directly in Snowflake do **NOT sync back to IdP**

Exam trap:

> Snowflake is **not source of truth** → IdP is.

---

# 3. Required SCIM Roles in Snowflake

Each IdP uses a specific provisioning role.

| Identity Provider | SCIM Role |
|------------------|----------|
| Okta | `okta_provisioner` |
| Microsoft Entra ID | `aad_provisioner` |
| Custom IdP | `generic_scim_provisioner` |

Critical rule:

> SCIM role must **own imported users & roles**  
Otherwise → updates from IdP will NOT sync.

Exam point:

> Ownership required for synchronization.

---

# 4. SCIM API Authentication

SCIM API requests use:

- **REST API**
- OAuth **Bearer token** in HTTP header  

Token validity:

- **6 months**

If expired:

Use function:

```sql
SYSTEM$GENERATE_SCIM_ACCESS_TOKEN
```

Exam point:

> SCIM uses **OAuth bearer token authentication**.

---

# 5. SCIM API Workflow

1. IdP sends REST request via SCIM client  
2. Snowflake validates OAuth token  
3. Snowflake performs action:

- create user  
- update user  
- create role  
- update role  

---

# 6. Auditing SCIM Requests

You can monitor provisioning activity using:

```
REST_EVENT_HISTORY
```

Example use:

- check recent provisioning updates  
- verify active users sync  

Exam point:

> SCIM audit uses **REST_EVENT_HISTORY table function**.

---

# 7. SCIM Use Cases

Main scenarios:

### User lifecycle management

- create users automatically  
- disable users centrally  
- update attributes  

### Group / Role lifecycle management

- manage Snowflake roles from IdP  
- assign role memberships centrally  

### Security audit

- verify IdP provisioning events  
- detect provisioning issues  

---

# 8. Replication Support

Snowflake supports:

- replication  
- failover / failback  

For:

> **SCIM security integrations**

---

# 9. Ultra-Short Exam Summary

SCIM =

- identity provisioning standard  
- users & roles managed from IdP  
- OAuth bearer token authentication  
- token valid 6 months  
- ownership required for sync  
- audit via REST_EVENT_HISTORY  

---
# ======================
# Snowflake Access Control

## 1. Snowflake Access Control Model

Snowflake uses a **hybrid access control framework**:

- **DAC (Discretionary Access Control)**  
  → Each object has an **owner** who can grant access.

- **RBAC (Role-Based Access Control)** ⭐ (main model)  
  → Privileges are granted to **roles**, then roles to users.

- **UBAC (User-Based Access Control)**  
  → Privileges can be granted **directly to users**  
  → Considered only when `USE SECONDARY ROLES = ALL`.

Exam key:

> Snowflake primarily uses **RBAC**.

---

## 2. Core Access Control Concepts

### Securable Object
Entity that access can be granted on.

Examples:

- database  
- schema  
- table  
- view  
- stage  
- warehouse  

Default rule:

> Access is **denied unless granted**.

---

### Role
Container of privileges.

- granted privileges on objects  
- assigned to users or other roles  

---

### Privilege
Permission to perform an action.

Examples:

- SELECT  
- USAGE  
- CREATE  
- OWNERSHIP  

---

### User
Identity in Snowflake:

- human user  
- service account  

Users receive privileges through **roles** (recommended).

---

## 3. Ownership Concept (DAC)

- Each object has **one owner role**
- Owner has **all privileges**
- Owner can **grant or revoke access**

Ownership transfer:

```sql
GRANT OWNERSHIP
```

Exam point:

> Object ownership controls access delegation.

---

## 4. Managed Access Schema (Important Exam Topic)

Normal schema:

- Object owner can grant privileges

Managed access schema:

- Only **schema owner** or role with `MANAGE GRANTS`
- Object owners **cannot grant access**

Exam key:

> Managed access centralizes privilege management.

---

## 5. Role Hierarchy & Privilege Inheritance

Roles can be granted to roles.

Example:

```
Role3 → Role2 → Role1 → User
```

Inheritance:

- Role1 inherits privileges of Role2 & Role3  
- User inherits all privileges  

Important:

> Role owner does NOT inherit privileges automatically.

---

## 6. Types of Roles

### Account Roles
- Can access objects across account
- Can be activated in session

### Database Roles
- Scope limited to one database
- Must be granted to account role
- Cannot be activated directly

### Instance Roles
- Control access to class instances

### Application Roles
- Used in Snowflake Native Apps

### Service Roles
- Provide access to service endpoints

---

## 7. System-Defined Roles (Very Important for Exam)

### GLOBALORGADMIN
- Organization-level administration  
- Exists only in organization account  

### ACCOUNTADMIN ⭐
- Highest role  
- Includes SYSADMIN + SECURITYADMIN  
- Should be limited

### SECURITYADMIN
- Manage grants globally  
- Create/manage users & roles  
- Has `MANAGE GRANTS`

### USERADMIN
- Create users and roles  

### SYSADMIN
- Create warehouses & databases  
- Recommended top role in custom hierarchy  

### PUBLIC
- Granted to all users and roles  
- Objects owned by PUBLIC accessible to all  

Exam trap:

> No “super-user” bypass → privileges always required.

---

## 8. Custom Roles (Best Practice)

- Create business-specific roles  
- Assign hierarchy under **SYSADMIN**

Recommended structure:

```
Custom Roles → SYSADMIN → ACCOUNTADMIN
```

Benefit:

- Clear separation of duties  
- Secure privilege management  

---

## 9. Privilege Management Commands

Grant privileges:

```sql
GRANT <privilege> TO ROLE
GRANT <privilege> TO USER
```

Revoke privileges:

```sql
REVOKE <privilege> FROM ROLE
REVOKE <privilege> FROM USER
```

Future grants:

- Define privileges on **future objects**

Example concept:

> Grant SELECT on all future tables in schema.

---

## 10. Active Roles in a Session

Each session has:

- **One primary role**
- **Optional secondary roles**

Primary role determined by:

1. Role specified in connection  
2. Default user role  
3. Otherwise → PUBLIC  

Change roles:

```sql
USE ROLE
USE SECONDARY ROLES
```

Check roles:

```sql
CURRENT_ROLE()
CURRENT_SECONDARY_ROLES()
```

---

## 11. Authorization Rules

Important exam rules:

- **CREATE object authorization → primary role only**
- Other actions → primary + secondary roles

Example:

- SELECT privilege may come from secondary role  
- Ownership privilege in hierarchy allows DDL  

---

## 12. Key Exam Summary (Ultra-Short)

Snowflake access control:

- Hybrid model: DAC + RBAC + UBAC  
- RBAC is main approach  
- Objects have single owner  
- Managed schema centralizes grants  
- Role hierarchy enables privilege inheritance  
- Primary role required for CREATE  
- No super-user bypass in Snowflake  

---

# =========================

# End-to-End Encryption (E2EE) in Snowflake 

## 1. What is End-to-End Encryption (E2EE)?

**End-to-End Encryption = Full protection of data**

- Protects data **at rest and in transit**
- Prevents **third parties from reading data**
- Reduces the **attack surface**

Snowflake security guarantees:

- Data always encrypted in cloud storage  
- TLS encryption used for network communication  

Exam key:

> Snowflake encrypts all customer data automatically.

---

## 2. E2EE Architecture Components

Typical data flow involves:

- Customer corporate network  
- Staging area (internal or external)  
- Snowflake secure VPC / VNet  

Types of stages:

### Internal Stage (Snowflake-managed)
- Files automatically encrypted on client before upload  
- Also encrypted again inside Snowflake  

### External Stage (Customer-managed)
- Example: Amazon S3, Azure Blob, GCS  
- Client-side encryption is **optional but recommended**

Exam trap:

> Internal stages always encrypt automatically.

---

## 3. End-to-End Encryption Data Flow

Typical workflow:

1. User uploads files to a stage  
2. Files encrypted (client-side or Snowflake encryption)  
3. Data loaded into Snowflake tables  
4. Stored in proprietary encrypted format  
5. Data decrypted temporarily during processing  
6. Re-encrypted after processing  
7. Results can be unloaded to stages  
8. Files decrypted on client side after download  

Key point:

> Snowflake decrypts data only during processing.

---

## 4. Encryption States in Snowflake

### Encryption in Transit
- Uses **TLS (HTTPS)**

### Encryption at Rest
- All data stored encrypted in cloud storage

### Encryption During Processing
- Temporary decryption in compute layer  
- Re-encrypted after transformation  

---

## 5. Client-Side Encryption (External Stages)

Definition:

> Data encrypted by client **before upload to cloud storage**

Benefits:

- Cloud storage provider cannot read data  
- ISP or third parties cannot read data  

Protocol steps:

1. Customer creates **secret master key**
2. Client generates **random file encryption key**
3. File encrypted with random key  
4. Random key encrypted with master key  
5. Both uploaded to cloud storage  

During download:

- Client decrypts random key using master key  
- Then decrypts file  

Important:

> Decryption happens only on client side.

---

## 6. Using Client-Side Encryption in Snowflake

To load encrypted data:

- Create **named stage**
- Specify `MASTER_KEY`

Example concept:

```sql
CREATE STAGE my_stage
ENCRYPTION=(MASTER_KEY='Base64_AES_key');
```

Key facts:

- AES 128-bit or 256-bit keys supported  
- Master key stored encrypted in Snowflake metadata  
- Only Snowflake processing layer can access it  

---

## 7. Named Stage Benefits (Exam Point)

Named stages allow:

- Secure credential storage  
- Key reuse  
- Granting access without exposing secrets  

Example workflow:

- Create table  
- Load encrypted data  
- Run analytics  
- Unload encrypted results  

---

## 8. Internal vs External Stage Encryption (Exam Comparison)

| Feature | Internal Stage | External Stage |
|--------|---------------|---------------|
| Automatic encryption | Yes | No (optional client-side) |
| Managed by Snowflake | Yes | Customer |
| Client-side encryption | Not needed | Recommended |

---

## 9. Key COF-C03 Exam Summary

- Snowflake uses **end-to-end encryption**
- Data encrypted:
  - at rest  
  - in transit  
  - during staging  
- Internal stages auto-encrypt  
- External stages support client-side encryption  
- Named stages can store encryption keys  
- TLS used for network security  

Ultra-short memory tip:

> Snowflake always encrypts data — client-side encryption adds extra security for external storage.

---

# ==========
# Differential Privacy in Snowflake 

## 1. What is Differential Privacy?

**Differential privacy = strong protection of sensitive data**

- Prevents identification of individuals in datasets  
- Allows analysis of **statistics, trends, and group behavior**
- Protects entities such as:
  - people  
  - companies  
  - locations  

Exam key:

> Differential privacy enables data sharing while reducing re-identification risk.

---

## 2. Why Use Differential Privacy?

It helps when:

- Sharing sensitive datasets across teams or organizations  
- Combining datasets (joins increase privacy risk)  
- Adding new fields or exposing detailed data  

Unique advantages:

- Protects against targeted privacy attacks  
- Quantifies trade-off between **privacy and data usefulness**
- Removes need for heavy masking or redaction  

---

## 3. How Snowflake Implements Differential Privacy

Snowflake enforces privacy using two main mechanisms:

### A. Noisy Aggregates

- Queries must return **aggregated results**
- Row-level queries (e.g. `SELECT *`) are blocked  
- Snowflake adds **random noise** to results  

Purpose of noise:

- Hide whether a specific individual is included  
- Prevent attacks like:
  - differencing  
  - thin-slicing  

Noise level depends on:

- number of rows  
- type of aggregate  
- transformations  

Important exam fact:

> Noise is added only once per final aggregation.

Special case:

- `SELECT COUNT(*)` returns exact result  
- Number of rows is considered public  

---

### B. Privacy Loss Control

Each query consumes **privacy loss**

- Privacy loss = measurable exposure risk  
- Snowflake tracks cumulative privacy loss  

Key concept: **Privacy Budget**

- Sets maximum acceptable privacy loss  
- Queries fail when budget limit is reached  
- Budget refreshes periodically  

Exam memory tip:

> Privacy budget limits how much sensitive insight a user can extract.

---

## 4. Privacy Policy

Differential privacy uses a **privacy policy object**

- Schema-level object  
- Associates privacy budgets with users or groups  
- Applied to tables or views → makes them **privacy-protected**

Snowflake then:

- Checks privacy budget before query execution  
- Adds noise based on query sensitivity  

---

## 5. Differential Privacy Workflow

### Data Provider Tasks

- Prepare dataset structure  
- Create privacy policy  
- Assign policy to tables/views  
- Define privacy domains for columns  
- Grant analyst access  
- Monitor privacy budgets  

### Analyst Tasks

- Review privacy domains  
- Run aggregated queries  
- Interpret results using noise interval  
- Adjust query scope to improve accuracy  

---

## 6. Supported Data Types

Allowed in privacy-protected tables:

- Numeric  
- String  
- Boolean  
- Date & time  

Limitations:

- Binary types not supported  
- Only `TIMESTAMP_NTZ` supported  

---

## 7. Practical vs Theoretical Privacy

Academic differential privacy:

- Very strict settings  
- Strong protection but lower data utility  

Snowflake default:

- Balanced privacy and usability  
- Customizable for business needs  

---

## 8. Key COF-C03 Exam Summary

- Differential privacy protects individual identity  
- Only aggregated queries allowed  
- Snowflake adds controlled random noise  
- Privacy loss accumulates per query  
- Privacy budget limits data exposure  
- Privacy policies enforce protection  

Ultra-short memory tip:

> Differential privacy = noisy aggregates + privacy budget control.

---

# ============

# Synthetic Data in Snowflake 

## 1. What is Synthetic Data?

Synthetic data = **artificial data generated from real data**

- Created using the stored procedure **GENERATE_SYNTHETIC_DATA**
- Keeps **statistical characteristics** of the original dataset  
- Does **NOT contain real rows or direct references**

Main use:

> Share or test sensitive data safely.

Feature requirement:

- **Enterprise Edition or higher**

---

## 2. Why Use Synthetic Data?

### Key Benefits

**Statistical consistency**
- Same column names and data types  
- Similar value distributions and correlations  
- Helps engineers understand production data patterns  

**Production validation**
- Enables safe workload testing  
- Improves reliability of production pipelines  

Exam tip:

> Synthetic data = realistic test data without exposing confidential information.

---

## 3. How Synthetic Data Generation Works

- Snowflake analyzes the **distribution of source data**
- Generates new artificial values with similar statistical properties  
- Output tables:
  - Same number of columns  
  - Same column types  
  - Same or fewer rows  

Synthetic tables appear in:

- **Data lineage graph**

---

## 4. Generated Data Behavior

### Non-join-key columns

**Statistical types**  
(number, boolean, date, time, timestamp)

- Generated with similar numeric patterns  

**Categorical strings**  
(few unique values)

- Generated using real values from source  

**Non-categorical strings**  
(many unique values)

- Redacted unless replacement option is used  

Definition:

- Few unique values → less than 50% of rows  
- Many unique values → more than 50% of rows  

---

## 5. Join Key Consistency

Important exam concept.

To enable joins on synthetic data:

- Designate columns as **join keys**

Snowflake then:

- Generates consistent synthetic values  
- Maintains join relationships  

Consistency rules:

- Same join key arguments must be used in all tables  
- Consistency across multiple runs requires:
  - **consistency_secret parameter**

Without secret:

- Consistency only within one execution  

---

## 6. Privacy Enhancements

Optional configuration:

- **similarity_filter = TRUE**

Effect:

- Removes rows too similar to original data  
- Uses distance-based privacy metrics  

Requirement:

- Non-string columns must not contain NULL values  

---

## 7. Input Table Requirements

Supported:

- Tables and views  
- Up to **5 input tables per call**

Limits:

- Minimum 20 distinct rows  
- Maximum 100 columns  
- Maximum 14 million rows  

Not supported:

- External tables  
- Apache Iceberg tables  
- Streams  

---

## 8. Supported Data Types

Supported:

- Numeric  
- Boolean  
- Date & time (except TIMESTAMP_TZ)  
- String  

Special rule:

- Highly unique string values may be **redacted**

---

## 9. Access Control Requirements

To generate synthetic data, role needs:

- USAGE on warehouse  
- SELECT on input tables  
- USAGE on databases and schemas  
- CREATE TABLE on output schema  
- OWNERSHIP on output tables  

Procedure availability:

- Accessible through **SNOWFLAKE.CORE_VIEWER role**
- Granted to **PUBLIC**

---

## 10. Operational Recommendations

Best practices:

- Use **medium Snowpark-optimized warehouse**
- Avoid running other queries simultaneously  
- Accept **Anaconda terms** before using feature  

---

## 11. Key COF-C03 Exam Summary

- Synthetic data = artificial dataset with similar statistics  
- Generated using **GENERATE_SYNTHETIC_DATA**
- Used for testing and safe data sharing  
- Join keys maintain relational consistency  
- Privacy filter removes overly similar rows  
- Enterprise Edition feature  

Ultra-short memory tip:

> Synthetic data = safe testing data with realistic distributions.

---

## ===============

# Data Sharing & Collaboration in Snowflake 

## 1. Why Share Data in Snowflake?

Snowflake enables **secure, real-time data sharing without copying data.**

### Benefits for Data Providers

- Control who can access shared datasets  
- Avoid data duplication and synchronization issues  
- Track consumer usage  
- Optionally monetize data  
- Automatically replicate data across regions  

### Benefits for Data Consumers

- Query shared data directly in Snowflake  
- Join shared datasets with their own data  
- Reduce ETL and transformation effort  

Exam tip:

> Snowflake data sharing = no data movement + secure controlled access.

---

## 2. Main Data Sharing Options

Snowflake provides three main sharing mechanisms:

### A. Listings

- Share data across **regions and clouds**
- Can be **private or public**
- Can include metadata (title, description, examples)
- Available on **Snowflake Marketplace**

Capabilities:

- Automatic cross-region fulfillment  
- Optional monetization  
- Public data offering  
- Consumer usage metrics  

Best use case:

> Broad distribution of datasets globally.

---

### B. Direct Share

- Share data with **specific accounts in same region**
- No automatic replication  
- No monetization or marketplace visibility  

Advantages:

- Simple and fast sharing  
- No data copy required  

Best use case:

> Private collaboration in one region.

---

### C. Data Exchange

- Share data with **a controlled group of accounts**
- Members can be:
  - providers  
  - consumers  
  - both  

Requires:

- Provisioning of exchange environment  

Best use case:

> Organization-level or consortium collaboration.

---

## 3. Sharing with Non-Snowflake Users

Snowflake supports **Reader Accounts**

- Enables external users to access shared data  
- No full Snowflake account required  

---

## 4. Collaboration with Data Clean Rooms

For **controlled analytics collaboration**, use:

### Snowflake Data Clean Rooms

- Provider defines allowed analyses  
- Consumer gets insights **without direct raw data access**

Purpose:

- Secure data collaboration  
- Privacy-preserving analytics  

Exam memory tip:

> Clean Room = controlled query access to shared data.

---

## 5. Comparison Summary (Exam Focus)

| Feature | Listing | Direct Share |
|--------|--------|-------------|
| Cross-region sharing | Yes | No |
| Cross-cloud sharing | Yes | No |
| Monetization option | Yes | No |
| Public offering | Yes | No |
| Usage metrics | Yes | No |

---

## 6. Key COF-C03 Exam Summary

- Snowflake sharing avoids data duplication  
- Listings enable global marketplace sharing  
- Direct shares are regional and private  
- Data Exchange supports controlled group sharing  
- Reader Accounts allow sharing with non-Snowflake users  
- Data Clean Rooms enable secure collaborative analytics  

Ultra-short memory tip:

> Listings = global sharing  
> Direct share = regional sharing  
> Clean Room = controlled collaboration.

---

## ====
# Secure Data Sharing in Snowflake 

## 1. What is Secure Data Sharing?

**Secure Data Sharing = sharing live data between Snowflake accounts without copying it.**

- Providers share selected database objects  
- Consumers access data in **read-only mode**
- Data stays in provider account  

Supported shared objects (exam focus examples):

- Databases and tables  
- Dynamic / external / Iceberg tables  
- Views (including secure views)  
- UDFs and ML models  
- Semantic views  
- Cortex Search services  

Key exam tip:

> Shared data is never physically copied.

---

## 2. How Secure Data Sharing Works

Architecture:

1. Provider creates a **share object**
2. Grants privileges on database objects to the share  
3. Adds consumer accounts to the share  
4. Consumer creates a **read-only database from the share**

Important properties:

- Near-instant access  
- Real-time updates  
- Storage cost = **zero for consumer**
- Consumer only pays **compute cost**

Exam memory tip:

> Storage billing stays with provider.

---

## 3. What is a Share?

A **share** is a Snowflake object that defines:

- Which objects are shared  
- Which accounts can access them  

Privileges can be granted:

- Directly to the share  
- Through a database role  

Provider capabilities:

- Add/remove consumers anytime  
- Add new objects instantly  
- Revoke access at any moment  

---

## 4. Provider vs Consumer Roles

### Provider

- Creates shares  
- Controls access granularity  
- Can share with many accounts  

Best practice:

- Use listings or data exchange for large distribution  

### Consumer

- Creates one imported database per share  
- Queries data like any normal database  
- Can consume multiple shares  

---

## 5. Read-Only Nature of Shared Data

Critical exam concept:

- Imported objects **cannot be modified**
- No insert / update / delete  
- No schema changes  

Consumers can only:

- Query data  
- Join with local datasets  

---

## 6. Sharing Options in Snowflake

Secure sharing can be delivered through:

- Listings (global distribution)  
- Direct shares (same region)  
- Data exchange (group collaboration)  
- Data clean rooms (controlled analytics)

---

## 7. Usage Metrics for Providers

Providers can monitor:

- Consumer activity  
- Listing usage statistics  
- Account-level access metrics  

This is available for:

- Marketplace listings  
- Private listings  
- Data exchange listings  

---

## 8. Reader Accounts (Third-Party Access)

Reader account = **managed Snowflake account for external users**

Characteristics:

- Created and owned by provider  
- Consumer does not need Snowflake license  
- Can query shared data only  

Limitations:

- Cannot load or modify data  
- Can consume shares only from provider  

Exam tip:

> Reader accounts enable data sharing with non-Snowflake customers.

---

## 9. Key COF-C03 Exam Summary

- Secure sharing = live read-only data access  
- No data movement or duplication  
- Consumers pay only compute cost  
- Share object controls access  
- One imported database per share  
- Reader accounts enable third-party sharing  

Ultra-short memory tip:

> Secure sharing = real-time, zero-copy, read-only data collaboration.

---

## ======================

# Reader Accounts in Snowflake 
## 1. What is a Reader Account?

A **reader account** is a special Snowflake account created and managed by a data provider to share data with users who are **not Snowflake customers**.

Key characteristics:

- Created and owned by the provider account  
- Used only to **query shared data**  
- Consumer does not need a Snowflake license  
- Provider pays all compute (credit) costs  

Exam tip:

> Reader accounts allow data sharing with external users without requiring them to purchase Snowflake.

---

## 2. Initial Configuration Workflow (Bootstrap)

After creating a reader account, the administrator must configure it.

### Step-by-step configuration

1. Log in as **ACCOUNTADMIN**
2. (Optional) Create **custom roles**
3. Create **users**
4. (Optional) Create **resource monitors**
5. Create **virtual warehouses**
6. Create **databases from shares**
7. Grant required privileges
8. Invite users and reset passwords

Important:

- Tasks must be performed **inside the reader account**
- Minimum requirement → warehouse + shared database

---

## 3. Roles and User Setup

Reader accounts include system roles:

- SYSADMIN  
- SECURITYADMIN  
- PUBLIC  

Optional best practice:

- Create custom roles for fine-grained access  
- Grant SECURITYADMIN to another user  
- Grant SYSADMIN to another user  

This enables delegation of administration tasks.

---

## 4. Virtual Warehouses and Credit Control

Warehouses are required to query shared data.

Key considerations:

- Credits are billed to the **provider account**
- Configure **auto-suspend** to save cost  
- Choose warehouse size carefully  

To control cost:

- Create **resource monitors**  
- Can limit credits per warehouse or per account  

Exam tip:

> Without resource monitors, reader accounts can consume unlimited credits.

---

## 5. Accessing Shared Data

Reader accounts contain **no data by default**.

To consume shared data:

```
CREATE DATABASE shared_db FROM SHARE provider_account.share_name;
```

Then grant minimum privileges:

- IMPORTED PRIVILEGES on shared database  
- USAGE on warehouse  

Example:

```
GRANT IMPORTED PRIVILEGES ON DATABASE shared_db TO ROLE PUBLIC;
GRANT USAGE ON WAREHOUSE query_wh TO ROLE PUBLIC;
```

---

## 6. Restrictions in Reader Accounts

Reader accounts are primarily **read-only analytics environments**.

Not allowed:

- INSERT / UPDATE / DELETE / MERGE  
- Loading new data  
- Creating shares or stages  
- Creating masking or row access policies  
- COPY INTO table  

Allowed examples:

- Query shared data  
- Create materialized views  
- Unload data using connection credentials  

Exam memory tip:

> Reader account = query-only environment.

---

## 7. Managing Reader Accounts

Reader accounts are managed using:

### Snowsight UI

Admin → Accounts → Reader accounts  

Tasks available:

- Create account  
- View accounts  
- Drop account  

### SQL DDL

Reader accounts are first-class objects:

```
CREATE MANAGED ACCOUNT
DROP MANAGED ACCOUNT
SHOW MANAGED ACCOUNTS
```

Example:

```
CREATE MANAGED ACCOUNT reader_acct1
ADMIN_NAME = admin_user
ADMIN_PASSWORD = 'Password123'
TYPE = READER;
```

Limits:

- Default maximum = **20 reader accounts**
- Dropped accounts remain in retention for **7 days**

---

## 8. Provider Responsibilities

Because reader accounts have no Snowflake contract:

- Provider handles user support  
- Provider opens Snowflake Support tickets if needed  

Also remember:

- Provider is responsible for all compute costs  
- Reader account exists in same **region and edition** as provider  

---

## 9. High Availability with Client Redirect (Business Critical)

For disaster recovery:

- Create reader accounts in multiple regions  
- Configure **Client Redirect**
- Redirect users to secondary region during outage  

Exam tip:

> Client Redirect requires Business Critical edition.

---

## 10. COF-C03 Exam Summary

- Reader account = external read-only Snowflake account  
- Created and managed by provider  
- Used to query shared data only  
- Provider pays compute cost  
- Requires warehouse + imported database  
- Limited DML and data loading capabilities  
- Max 20 reader accounts by default  

Ultra-short memory tip:

> Reader account = managed external analytics access with provider-paid compute.

---

## =====
# Snowflake Stages – COF-C03 Quick Revision

## What is a Stage in Snowflake
A **stage** is a location used to store data files for:

- Loading data into Snowflake tables
- Unloading data from tables into files

Stages are required before using the **COPY command**.

---

## Types of Stages

### Internal Stage
- Files stored **inside Snowflake storage**
- Managed automatically by Snowflake
- Good for small or temporary data loads

Default encryption: **SNOWFLAKE_FULL**

---

### External Stage
- Files stored **outside Snowflake** (cloud storage)

Supported providers:
- Amazon S3
- Azure Blob Storage
- Google Cloud Storage

Requires:
- Storage integration (recommended)
OR
- Credentials (less secure)

---

## Important Stage Commands

- `CREATE STAGE`
- `ALTER STAGE`
- `DROP STAGE`
- `SHOW STAGES`
- `DESCRIBE STAGE`

Variant:
- `CREATE OR ALTER STAGE`
- `CREATE STAGE … CLONE`

---

## File Formats in Stage

A stage can define file format:

Supported types:
- CSV
- JSON
- PARQUET
- AVRO
- ORC
- XML

Key concept:
- You can use **FORMAT_NAME** or **TYPE**
- Not both at the same time

Default file format: **CSV**

---

## Directory Table (Important for Exam)

A stage can include a **directory table** that stores metadata about files.

Options:
- ENABLE = TRUE
- AUTO_REFRESH = TRUE

Benefits:
- Track files automatically
- Improve data ingestion automation

---

## Temporary Stage

- Created using `TEMP` or `TEMPORARY`
- Deleted at end of session

Important behavior:
- Internal stage → files deleted permanently
- External stage → only stage object deleted

---

## Security Best Practice

For external stages:

Use **STORAGE INTEGRATION** instead of credentials.

Why:
- More secure
- Easier access management
- No need to store cloud keys in SQL

---

## Encryption Concepts

Internal stage:
- SNOWFLAKE_FULL → client + server encryption
- SNOWFLAKE_SSE → server only

External stage:
- Encryption depends on cloud provider
- Example: AWS_SSE_KMS, AZURE_CSE

---

## Important Exam Notes

- If URL is not specified → Snowflake creates **internal stage**
- Recreating stage can break:
  - External tables
  - Pipes
  - Directory tables
- COPY options should be defined in `COPY INTO`, not in CREATE STAGE

---

## Privileges Required

To create a stage:
- USAGE on storage integration (if external)
- CREATE STAGE on schema

To modify stage:
- OWNERSHIP privilege

---

## Key Concept Summary

Stage = data landing zone  
Internal = Snowflake storage  
External = Cloud storage  
Used with COPY INTO  
Storage integration = best practice  
Directory table = file metadata tracking

## ====

# Snowflake File Format – COF-C03 Quick Revision

## What is a File Format in Snowflake

A **file format** is a named object that defines how Snowflake:

- Reads staged files (data loading)
- Writes files (data unloading)

Used mainly with:
- COPY INTO <table>
- COPY INTO <location>
- Stages
- External tables

---

## Why File Format is Important

It defines:
- Field delimiter
- Record delimiter
- Compression
- Null handling
- Encoding
- Data structure (CSV / JSON / etc.)

Without correct file format → load errors or wrong data.

---

## Main Command

CREATE FILE FORMAT

Variants:
- CREATE OR ALTER FILE FORMAT
- DROP FILE FORMAT
- SHOW FILE FORMATS
- DESCRIBE FILE FORMAT

---

## Supported File Format Types

### CSV
- Most common format
- Any delimiter possible (not only comma)

Used for:
- Loading and unloading

---

### JSON
- Semi-structured format
- Supports multiline JSON
- Snowflake unloads JSON as **NDJSON**

---

### PARQUET
- Columnar binary format
- High performance
- Used for load and unload

---

### AVRO
- Binary format
- Load only (cannot unload)

---

### ORC
- Binary format
- Load only

---

### XML
- Text structured format
- Load only

---

### CUSTOM (preview)
- Used for unstructured data

---

## Temporary File Format

You can create:

TEMP FILE FORMAT

Behavior:
- Exists only during session
- Automatically dropped

---

## Important File Format Options (Exam Focus)

### Compression
Default = AUTO

Examples:
- GZIP
- ZSTD
- NONE

---

### Delimiters

Field delimiter → separates columns  
Record delimiter → separates rows  

Default:
- FIELD_DELIMITER = comma
- RECORD_DELIMITER = newline

---

### Header Handling

SKIP_HEADER = number of lines to skip  
PARSE_HEADER = use first row as column names  

---

### NULL Handling

NULL_IF = ('NULL', '')

Used to convert string values into SQL NULL.

---

### Encoding

Default encoding = UTF-8

Snowflake always stores data internally in UTF-8.

---

### Error Handling

ERROR_ON_COLUMN_COUNT_MISMATCH

TRUE → load fails  
FALSE → load continues with NULL values

---

### Space Handling

TRIM_SPACE = TRUE removes unwanted spaces during load.

---

## Important Exam Notes

- Default file format type = CSV
- TYPE cannot be changed using ALTER
- CREATE OR REPLACE breaks external table linkage
- File format conflicts in SQL → error
- TEMP file format hides permanent file format with same name

---

## Privileges Required

To create file format:
- CREATE FILE FORMAT on schema

To modify:
- OWNERSHIP privilege

---

## Key Concept Summary

File format = data interpretation rules  
Used with COPY command  
Supports structured and semi-structured formats  
Controls delimiters, compression, nulls, encoding  
Critical for successful data loading

## ======

# Snowflake External Volume – COF-C03 Quick Revision

## What is an External Volume

An **external volume** is a Snowflake object that defines  
a **cloud storage location used by Apache Iceberg tables**.

It connects Snowflake to external storage such as:

- Amazon S3
- Google Cloud Storage
- Microsoft Azure
- S3-compatible storage

External volumes store:
- Iceberg metadata files
- Table data files (Parquet)

---

## Why External Volume is Important

It enables:

- Iceberg table storage outside Snowflake
- Data lake architecture
- Hybrid storage model
- Multi-cloud support

External volume is **required for Iceberg tables**.

---

## Main Command

CREATE EXTERNAL VOLUME

Other commands:

- ALTER EXTERNAL VOLUME  
- DROP EXTERNAL VOLUME  
- SHOW EXTERNAL VOLUMES  
- DESCRIBE EXTERNAL VOLUME  

---

## Key Concept

External Volume =  
Named connection to external cloud storage  
used by Iceberg tables.

---

## Required Parameter

External volume must define:

STORAGE_LOCATIONS

Each storage location includes:

- Name
- Cloud provider
- Base storage URL
- Authentication configuration

Only **one storage location is active at a time**.

---

## Supported Cloud Providers

### Amazon S3

Uses:

- IAM Role ARN
- Bucket URL
- Optional encryption (SSE-S3 or SSE-KMS)

---

### Google Cloud Storage

Uses:

- Bucket URL
- Optional KMS encryption

---

### Microsoft Azure

Uses:

- Azure tenant ID
- Storage account URL

---

### S3-Compatible Storage

Uses:

- Endpoint URL
- Access key credentials

---

## Optional Parameter

ALLOW_WRITES = TRUE | FALSE

TRUE → Snowflake can write Iceberg metadata  
FALSE → Read-only access

Default = TRUE

Important for:
- Writable Iceberg tables
- Delta-based Iceberg tables

---

## Encryption Options

External storage encryption types include:

- AWS_SSE_S3
- AWS_SSE_KMS
- GCS_SSE_KMS
- NONE

Encryption is managed by cloud provider.

---

## Privileges Required

To create external volume:

- CREATE EXTERNAL VOLUME privilege on ACCOUNT

By default:
- Only ACCOUNTADMIN has this privilege

---

## Important Exam Notes

- External volume is mainly used with Iceberg tables  
- You cannot drop external volume if Iceberg tables depend on it  
- CREATE OR REPLACE recreates the object atomically  
- OR REPLACE and IF NOT EXISTS cannot be used together  
- Storage location must contain Iceberg metadata and Parquet data  

---

## Key Concept Summary

External volume =  
Bridge between Snowflake and external data lake storage.

Used only for:
- Apache Iceberg tables
- Hybrid storage architectures

### =======
# Snowflake Pipe / Snowpipe – COF-C03 Quick Revision

## What is a Pipe in Snowflake

A **pipe** is a Snowflake object that contains a `COPY INTO <table>` statement.

It is used by:

- **Snowpipe** to automatically or manually load files into a table
- **Snowpipe Streaming** to ingest streaming data directly into tables

---

## Why Pipe is Important

A pipe defines **how data is ingested** into a table.

It enables:

- Continuous data loading
- Automated ingestion from cloud storage events
- Low-latency streaming ingestion

---

## Main Command

CREATE PIPE

Other commands:

- ALTER PIPE
- DROP PIPE
- SHOW PIPES
- DESCRIBE PIPE

---

## Key Concept

Pipe = saved ingestion definition using `COPY INTO <table>`

---

## Required Parameter

A pipe must contain:

- A unique pipe name
- A `COPY INTO <table>` statement

Two possible data sources:

### 1. Staged files
Example concept:

`COPY INTO mytable FROM @mystage`

### 2. Streaming source
Example concept:

`COPY INTO mytable FROM (SELECT ... FROM TABLE(DATA_SOURCE(TYPE => 'STREAMING')))`

---

## AUTO_INGEST Parameter

### AUTO_INGEST = TRUE
- Enables automatic loading
- Triggered by cloud event notifications
- Common with Snowpipe

### AUTO_INGEST = FALSE
- Automatic loading disabled
- Files must be submitted using Snowpipe REST API

Exam tip:

> AUTO_INGEST is one of the most important Snowpipe concepts.

---

## Cloud Integrations for AUTO_INGEST

### Amazon S3
Uses:
- `AWS_SNS_TOPIC`

### Google Cloud Storage
Uses:
- `INTEGRATION`

### Microsoft Azure
Uses:
- `INTEGRATION`

---

## Snowpipe vs Snowpipe Streaming

### Snowpipe
- Loads files from stages
- Usually event-driven
- Uses COPY INTO from `@stage`

### Snowpipe Streaming
- Loads records directly from streaming source
- No stage required
- Uses `DATA_SOURCE(TYPE => 'STREAMING')`

Exam memory tip:

> Snowpipe = file ingestion  
> Snowpipe Streaming = row streaming ingestion

---

## Important Supported Features

Pipe supports:
- Simple transformations in SELECT
- Column mapping
- Metadata columns
- MATCH_BY_COLUMN_NAME
- INCLUDE_METADATA

---

## Important Limitations

Not supported in pipe COPY options:

- `FILES = (...)`
- `ON_ERROR = ABORT_STATEMENT`
- `SIZE_LIMIT`
- `PURGE`
- `FORCE`
- `VALIDATION_MODE`

Important note:

- Pipe definition is **not dynamic**
- If stage or table changes, create a new pipe

---

## Recommended Best Practice

Do not use these time functions in Snowpipe copy statement:

- CURRENT_TIMESTAMP
- CURRENT_DATE
- CURRENT_TIME
- SYSDATE
- SYSTIMESTAMP

Recommended instead:

- `METADATA$START_SCAN_TIME`

---

## Privileges Required

To create a pipe, role needs at least:

- CREATE PIPE on schema
- SELECT and INSERT on target table
- READ on internal stage
- USAGE on external stage
- USAGE on integration if required

---

## Important Exam Notes

- Pipe stores a COPY INTO statement
- AUTO_INGEST enables event-driven loading
- Snowpipe works with staged files
- Snowpipe Streaming works with streaming source
- Recreating a pipe replaces the object atomically
- OR REPLACE and IF NOT EXISTS cannot be used together

---

## Key Concept Summary

Pipe = ingestion object  
Snowpipe = automated file ingestion  
Snowpipe Streaming = direct streaming ingestion  
AUTO_INGEST = automatic load from events  
Pipe always loads data into a table

## ==========

# Snowflake — COPY INTO <table> (Course Summary)

## Purpose

`COPY INTO <table>` loads data files into an existing Snowflake table.

Files must be located in:

- Internal stage (Snowflake storage)
- External stage (S3 / Azure / GCS)
- Direct external cloud storage location

Snowflake cannot load data from archival storage (e.g., Glacier, Azure Archive).

---

## Basic Syntax

```sql
COPY INTO table_name
FROM @stage_or_location
FILE_FORMAT = (TYPE = CSV | JSON | PARQUET | AVRO | ORC | XML);
```

Optional parameters:

- FILES → load specific files  
- PATTERN → load files using regex  
- COPY options → control loading behavior  

---

## Types of Stages

### Internal Stage

- Named stage → `@stage_name`
- Table stage → `@%table_name`
- User stage → `@~`

### External Stage

- References cloud storage
- Uses STORAGE INTEGRATION or credentials

### Direct External Location

Example:

```sql
FROM 's3://bucket/path'
```

---

## Supported File Formats

- CSV  
- JSON  
- AVRO  
- ORC  
- PARQUET  
- XML  

Example:

```sql
FILE_FORMAT = (TYPE = JSON);
```

Or use named format:

```sql
FILE_FORMAT = (FORMAT_NAME = my_format);
```

---

## Loading with Transformation

```sql
COPY INTO table_name (col1, col2)
FROM (
   SELECT $1, $2
   FROM @stage
);
```

Allows:

- Column selection  
- Column reordering  
- Casting  

---

## Important COPY Options

### ON_ERROR

Controls error handling:

- ABORT_STATEMENT → stop load (default bulk)
- CONTINUE → skip bad rows
- SKIP_FILE → skip entire file

---

### FORCE

```sql
FORCE = TRUE;
```

Reload files even if already loaded (can create duplicates).

---

### PURGE

```sql
PURGE = TRUE;
```

Deletes staged files after successful load.

---

### MATCH_BY_COLUMN_NAME

Loads semi-structured data by matching column names.

Values:

- CASE_SENSITIVE  
- CASE_INSENSITIVE  

Column order does not matter.

---

### VALIDATION_MODE

Validates files without loading data.

```sql
VALIDATION_MODE = RETURN_ERRORS;
VALIDATION_MODE = RETURN_10_ROWS;
```

---

### SIZE_LIMIT

Limits total data size per COPY execution.

Useful for batch ingestion.

---

## Pattern Loading

```sql
PATTERN = '.*sales.*[.]csv';
```

Used to filter files in large stages.

---

## Metadata Capture

```sql
INCLUDE_METADATA = (
   filename = METADATA$FILENAME
);
```

Stores file metadata in table columns.

---

## Iceberg Loading Modes

- FULL_INGEST → scans and rewrites files  
- ADD_FILES_COPY → fast copy + register  

Used for Snowflake-managed Iceberg tables.

---

## COPY Command Output

After execution Snowflake returns:

- FILE  
- STATUS  
- ROWS_PARSED  
- ROWS_LOADED  
- ERRORS_SEEN  
- FIRST_ERROR  

---

## Key Exam Points

- COPY loads data from stages  
- Storage integration is recommended  
- MATCH_BY_COLUMN_NAME ignores column order  
- FORCE reloads files  
- PURGE deletes staged files  
- VALIDATION_MODE checks data without loading  
- ON_ERROR controls load failure behavior  

## ======
# Apache Iceberg Tables in Snowflake — Certification Focus

## Definition
Apache Iceberg tables in Snowflake let you query table data stored in **external cloud storage that you manage** while using Snowflake SQL.

They are useful for **data lakes** and **open table format** architectures.

---

## Key idea
Iceberg tables combine:

- Snowflake query engine
- External cloud storage
- Apache Iceberg table format

Supported storage:
- Amazon S3
- Google Cloud Storage
- Azure Storage

Supported file format:
- **Parquet**

---

## Main Snowflake objects to remember

### External Volume
An **external volume** connects Snowflake to your external cloud storage for Iceberg tables.

Use it to access:
- data files
- metadata files
- manifest files

### Catalog
The **catalog** manages Iceberg metadata and current table state.

Snowflake supports 2 catalog choices:
- **Snowflake as catalog**
- **External catalog**

### Catalog Integration
A **catalog integration** is required when Iceberg metadata is managed outside Snowflake.

Examples:
- AWS Glue
- Iceberg REST catalog
- Snowflake Open Catalog

---

# Two catalog options

## 1. Snowflake as the catalog
Also called **Snowflake-managed Iceberg tables**

### Important points
- Full Snowflake support
- Read and write supported
- Data still stays in external storage
- Snowflake handles maintenance like compaction

### Exam point
This is the **best integrated option** in Snowflake.

---

## 2. External catalog
Also called **externally managed Iceberg tables**

### Important points
- Metadata managed outside Snowflake
- Snowflake uses catalog integration to read metadata
- Limited Snowflake platform support compared to Snowflake-managed tables

### Exam point
Use this when the Iceberg table already exists in another ecosystem.

---

# Storage and billing

## Storage
Iceberg data is stored in **your external cloud storage**, not in Snowflake storage.

## Billing
Snowflake charges for:
- compute
- cloud services
- some refresh / transfer operations

Snowflake does **not** charge for:
- Iceberg storage itself

Your cloud provider charges for:
- external storage
- possible egress costs

### Exam point
Iceberg tables **do not incur Snowflake storage cost**.

---

# Cross-cloud / cross-region
Snowflake supports Iceberg tables across different:
- clouds
- regions

But this can create:
- cloud egress costs
- cross-region transfer costs

---

# Versions supported
Snowflake supports:
- Iceberg v1
- Iceberg v2
- Iceberg v3 (**preview**)

---

# Important advantages
Apache Iceberg provides:
- ACID transactions
- schema evolution
- hidden partitioning
- snapshots

---

# Important limitations for exam

- Only **Parquet** is supported
- No **Fail-safe**
- Some Snowflake features are not supported
- Externally managed Iceberg tables have more limitations than Snowflake-managed ones

---

# Best certification points to memorize

## Very important
- Iceberg tables store data in **external cloud storage**
- Snowflake connects using an **external volume**
- Metadata is managed by either:
  - **Snowflake**
  - **external catalog**
- **Catalog integration** is used with external catalogs
- Supported data file format = **Parquet**
- Snowflake-managed Iceberg tables offer **fuller support**
- Externally managed Iceberg tables offer **less Snowflake functionality**
- Iceberg tables do **not** use Snowflake storage
- No **Fail-safe** for Iceberg tables

---

# Quick comparison

| Point | Snowflake-managed | Externally managed |
|---|---|---|
| Catalog | Snowflake | External catalog |
| Read | Yes | Yes |
| Write | Yes | Yes, with limitations |
| Maintenance | Snowflake handles more | External system handles lifecycle |
| Snowflake feature support | Higher | Lower |

---

# One-line exam summary
Apache Iceberg tables in Snowflake let you query open-format tables stored in your own cloud storage, using an **external volume** for storage access and either **Snowflake** or an **external catalog** for metadata management.

## ====
# CREATE API INTEGRATION — Snowflake Certification Cheat Sheet

## Definition  
`CREATE API INTEGRATION` creates a **secure configuration object** that allows Snowflake to connect to **external HTTPS services**.

👉 Mainly used for:
- External functions  
- Git repository integration  
- Calling cloud API gateways securely  

---

## Core Certification Concept  

API Integration acts as a **security bridge** between Snowflake and external services.

It stores:
- Cloud provider type  
- Authentication configuration  
- Allowed API endpoints  
- Enable / disable status  

---

## Supported API Providers (Exam Focus)

### AWS API Gateway  
- `API_PROVIDER = aws_api_gateway`
- Requires:
  - IAM Role ARN  
- Used for calling AWS-hosted APIs  

### Azure API Management  
- `API_PROVIDER = azure_api_management`
- Requires:
  - Azure Tenant ID  
  - Azure AD Application ID  

### Google Cloud API Gateway  
- `API_PROVIDER = google_api_gateway`
- Requires:
  - Audience claim (JWT authentication)

### Git Repository  
- `API_PROVIDER = git_https_api`
- Used for:
  - Accessing remote Git repositories  
- Authentication methods:
  - Personal access token  
  - OAuth  
  - Snowflake GitHub App  

---

## Key Security Parameters (Very Important for Exam)

### API_ALLOWED_PREFIXES  
Defines **which external URLs Snowflake is allowed to call**.

👉 Works like a **whitelist**  
Best practice → restrict as narrowly as possible.

Example idea:



## ======= 

# Snowflake Connectors — Certification Focus

## Definition
**Snowflake Connectors** are native integrations that allow Snowflake to automatically ingest or synchronize data from third-party systems.

They help:
- access external data without building custom APIs
- automate **initial historical load + incremental refresh**
- keep data **continuously updated in Snowflake**

👉 Certification key idea:  
Connectors simplify **data ingestion and replication from external systems.**

---

## Key characteristics (important for exam)

- Native Snowflake integrations  
- Automatic data refresh based on schedule  
- Support:
  - Initial bulk load
  - Incremental change capture (CDC / replication)
- Reduce need for manual ETL/API development  

---

## Main Snowflake connectors to know

### Google Analytics Connectors

#### Google Analytics Aggregate Data Connector
- Loads **aggregated GA4 metrics**
- Uses GA4 Reporting API  
- Good for dashboards and reporting

#### Google Analytics Raw Data Connector
- Loads **event-level GA4 data**
- More granular analytics and advanced modeling

👉 Exam tip  
Aggregate = summarized data  
Raw = detailed event data  

---

### Google Looker Studio Connector
- Enables **data visualization**
- Connects Snowflake datasets to Looker Studio dashboards  

👉 Certification concept  
Connector used for **BI visualization integration**, not ingestion.

---

### ServiceNow Connector
- Automatically ingests **ServiceNow operational data**
- Supports analytics on ITSM / ticketing data

---

### Database Connectors

#### MySQL Connector
- Loads data from MySQL into Snowflake
- Supports **replication of ongoing changes**

#### PostgreSQL Connector
- Loads and replicates PostgreSQL data into Snowflake

👉 Very important exam idea  
Database connectors support:
- continuous synchronization  
- near real-time analytics on operational databases  

---

### SharePoint Connector
- Ingests:
  - files
  - metadata
  - permissions
- Keeps content updated in Snowflake  
- Can integrate with **Cortex Search for AI analysis**

---

# Certification summary

Snowflake connectors provide:
- automated ingestion from SaaS and databases  
- incremental refresh capabilities  
- simplified integration architecture  
- faster analytics enablement  

Typical use cases:
- marketing analytics (Google Analytics)
- IT operations analytics (ServiceNow)
- database replication (MySQL / PostgreSQL)
- document analytics (SharePoint)
- BI dashboards (Looker Studio)

---

# One-line exam memory
Snowflake connectors are native integrations that automatically ingest and synchronize data from external applications and databases into Snowflake.


## ======
# Using a Git Repository in Snowflake — Certification Summary

## Definition  
Snowflake can integrate with a **remote Git repository**.  
Files from the remote repo are synchronized into a **local Git repository clone inside Snowflake**.

👉 The Snowflake clone is a **full clone**:
- branches  
- tags  
- commits  

---

## Main purpose (exam concept)

Git integration allows Snowflake to:
- use external code stored in Git  
- manage versioned SQL / Python / handler code  
- support modern DevOps workflows  

👉 Certification idea  
Git integration enables **version-controlled code development inside Snowflake.**

---

## What you can do with a Git repository clone

### Common Git-like operations  
- Fetch latest changes from remote repository  
- Browse branches or tags  
- Search files  
- Reference file paths in Snowflake code  

Example usage:
- Stored procedure handler code  
- UDF handler code  
- Task scripts  
- SQL deployment scripts  

---

## Executing code from Git

Snowflake can:
- Execute `.sql` files directly from the repository clone  
- Import code files into procedures or functions  

👉 Certification tip  
Repository files behave similar to **files stored in a stage.**

---

## Writing to the remote repository (important exam detail)

Push operations are supported **only from specific Snowflake features**:

- Workspaces  
- Streamlit apps  
- Snowflake Notebooks  

👉 Normal SQL sessions cannot push changes.

---

## How synchronization works

1. Remote Git repository is integrated with Snowflake  
2. Snowflake creates a **local clone object**
3. Users reference files from the clone  
4. Clone can be refreshed using fetch  

👉 Snowflake acts as **another Git client** separate from local developer machines.

---

## Supported Git platforms (must memorize)

Snowflake supports repositories based on:

- GitHub  
- GitLab  
- Bitbucket  
- Azure DevOps  
- AWS CodeCommit  

👉 Certification memory  
Any Git platform accessible via HTTPS can generally be integrated.

---

## Important commands (possible exam keywords)

- CREATE GIT REPOSITORY  
- ALTER GIT REPOSITORY  
- SHOW GIT REPOSITORIES  
- SHOW GIT BRANCHES  
- SHOW GIT TAGS  

---

## Certification memory sentence  

Git repository integration allows Snowflake to **synchronize version-controlled code from remote Git systems and use it directly inside Snowflake workloads.**

## ======
# Snowflake Column-level Security — COF-C03 Quick Revision

## Definition
Column-level Security protects sensitive data by applying **masking policies** on table or view columns.

Protection is applied **at query runtime**, not by modifying stored data.

Requires **Enterprise Edition or higher**.

---

## Main Features

### Dynamic Data Masking
- Masks plaintext data at query time
- Data remains stored unmasked
- Uses masking policies only (native Snowflake)

### External Tokenization
- Data is tokenized before loading into Snowflake
- Detokenization happens at query runtime
- Requires:
  - masking policy
  - external function
  - third-party tokenization provider

---

## Masking Policy

A masking policy is a **schema-level security object** that controls how column data is shown.

Users may see:
- real value
- masked value
- partially masked value
- tokenized value

Important rule:
- Input and output data types **must match**

---

## Example (Dynamic Masking)

```sql
CREATE MASKING POLICY ssn_mask AS (val STRING) RETURNS STRING ->
CASE
  WHEN CURRENT_ROLE() = 'PAYROLL' THEN val
  ELSE '******'
END;
```
## ==========
# Snowflake Resource Monitors — Certification Summary

## Definition
A **resource monitor** is a Snowflake object used to **monitor and control credit usage for virtual warehouses**.

It helps:
- track warehouse credit consumption
- send notifications at thresholds
- suspend warehouses when limits are reached

---

## Most Important Exam Point
**Resource monitors only work for warehouses.**

They **do not** track:
- serverless features
- AI services

For those, Snowflake recommends using:
- **Budgets**

---

## What Resource Monitors Track
A resource monitor tracks:
- credits consumed by **user-managed virtual warehouses**
- credits consumed by **cloud services supporting those warehouses**

Important:
- it can **monitor** cloud services usage
- it can only **suspend user-managed warehouses**

---

## Main Properties of a Resource Monitor

### 1. Credit Quota
The credit quota is the number of Snowflake credits allocated to the monitor for a time interval.

Example:
- quota = 1000 credits

Snowflake tracks usage against this quota.

---

### 2. Monitor Type
There are two monitor types:

#### Account Monitor
- monitors **all warehouses in the account**
- only **one account monitor** per account

#### Warehouse Monitor
- monitors only **specific assigned warehouses**
- an account can have **multiple warehouse monitors**
- each warehouse can be assigned to **only one warehouse monitor**

If no warehouse or account is assigned, the monitor is dormant.

---

### 3. Schedule
Defines when monitoring starts and when usage resets.

#### Default schedule
- starts immediately
- resets monthly

#### Custom schedule options
Supported frequencies:
- DAILY
- WEEKLY
- MONTHLY
- YEARLY
- NEVER

Important exam point:
- if you set **FREQUENCY**, you must also set **START_TIMESTAMP**
- reset always happens at **12:00 AM UTC**

Example:
- if start date is July 15 and frequency is monthly, usage resets every month on the 15th at 12:00 AM UTC

---

### 4. Actions / Triggers
Actions are executed when usage reaches a threshold percentage of the quota.

Supported actions:

#### NOTIFY
- sends notification only

#### SUSPEND
- sends notification
- suspends assigned warehouses **after current statements finish**

#### SUSPEND_IMMEDIATE
- sends notification
- suspends assigned warehouses immediately
- cancels currently running statements

Important limits:
- 1 SUSPEND action maximum
- 1 SUSPEND_IMMEDIATE action maximum
- up to 5 NOTIFY actions

Important:
- a monitor must have **at least one action**
- otherwise reaching quota does nothing

---

## Assignment Rules

### Account-level assignment
A resource monitor can be assigned to the entire account:
- controls all warehouses in the account

### Warehouse-level assignment
A resource monitor can be assigned to one or more warehouses:
- controls only those assigned warehouses

Important exam point:
- **an account-level monitor does not override warehouse-level monitors**
- if either monitor reaches a suspend threshold, the warehouse is suspended

---

## Suspension and Resumption
When a warehouse is suspended by a resource monitor, it cannot be resumed until one of the following happens:
- a new interval starts
- the quota is increased
- the suspend threshold is increased
- the warehouse is removed from the monitor
- the monitor is dropped

---

## Important Certification Notes

### Resource monitors do NOT control:
- Snowpipe
- automatic clustering
- materialized views
- other serverless compute
- AI services

### Resource monitors are not exact real-time limiters
Snowflake warns that a warehouse may continue consuming some credits after threshold is reached.

Reason:
- suspension can take time
- even SUSPEND_IMMEDIATE is not a perfect hard stop

Best practices:
- use thresholds like **90% instead of 100%**
- assign one warehouse per monitor for tighter control

---

## Notifications
Notifications are disabled by default.

They can be sent:
- by email
- in Snowsight

### For warehouse monitors
Notifications can be sent to:
- account administrators with notifications enabled
- non-admin users in the notification list

### For account monitors
Notifications go to:
- account administrators only

Important exam point:
- **non-admin users cannot receive notifications for account monitors**
- non-admin users can receive email notifications only for **warehouse monitors**

---

## Privileges

### Create resource monitor
By default, only:
- **ACCOUNTADMIN**

### View or modify resource monitors
Privileges that can be granted:
- **MONITOR**
- **MODIFY**

Important:
- only ACCOUNTADMIN can fully manage resource monitors in Snowsight

---

## Main SQL Commands
- `CREATE RESOURCE MONITOR`
- `ALTER RESOURCE MONITOR`
- `SHOW RESOURCE MONITORS`
- `DROP RESOURCE MONITOR`

Related commands:
- `ALTER WAREHOUSE`
- `ALTER ACCOUNT`
- `SHOW WAREHOUSES`

---

## Example 1 — Basic Warehouse Monitor
```sql
CREATE OR REPLACE RESOURCE MONITOR limit1 WITH CREDIT_QUOTA=1000
  TRIGGERS ON 100 PERCENT DO SUSPEND;

ALTER WAREHOUSE wh1 SET RESOURCE_MONITOR = limit1;
```
Key Exam Traps

Trap 1
Resource monitors only monitor warehouses.

Trap 2
An account can have only one account monitor.

Trap 3
A warehouse can belong to only one warehouse monitor.
Trap 4
SUSPEND waits for running statements to finish.

Trap 5
SUSPEND_IMMEDIATE cancels running statements immediately.

Trap 6
Only ACCOUNTADMIN can create a resource monitor by default.

Trap 7
A warehouse monitor can monitor cloud services usage but cannot suspend cloud services themselves.

Trap 8
Serverless features are not controlled by resource monitors.

### ===== 
# Account Usage — Snowflake COF-C03 Quick Revision

## Definition
ACCOUNT_USAGE = schema in the shared `SNOWFLAKE` database that provides:
- object metadata
- historical usage data
- monitoring and governance information

It is mainly used to analyze:
- queries
- warehouse usage
- storage
- logins
- policies
- governance
- security activity

---

## Main Schemas

### ACCOUNT_USAGE
Used for your own Snowflake account.

Contains views for:
- metadata
- usage history
- performance
- cost monitoring
- governance
- security

### READER_ACCOUNT_USAGE
Used for reader accounts created through Secure Data Sharing.

Important:
- only applies if reader accounts exist
- includes an extra column: `READER_ACCOUNT_NAME`

---

## Important Differences vs Information Schema

### 1. Dropped objects
ACCOUNT_USAGE includes dropped objects.
Information Schema does not.

Important exam point:
- ACCOUNT_USAGE may show deleted objects
- many views include a `DELETED` column

### 2. Data latency
ACCOUNT_USAGE is not real time.

Typical latency:
- about 45 minutes to 3 hours
- many views are around 2 hours
- `QUERY_HISTORY` can be as low as 45 minutes

Information Schema:
- no latency

### 3. Retention
ACCOUNT_USAGE keeps historical data longer.

Typical retention:
- 1 year

Information Schema:
- shorter retention
- usually 7 days to 6 months depending on object/view

---

## Certification Memory Rule

Use ACCOUNT_USAGE when you need:
- historical analysis
- longer retention
- dropped object history

Use Information Schema when you need:
- near real-time metadata
- current object state

---

## Very Important Certification Concepts

### ACCOUNT_USAGE is in the SNOWFLAKE database
So queries usually look like:
`SNOWFLAKE.ACCOUNT_USAGE.<view_name>`

### Common examples of important views
You do NOT need to memorize all views.
Know the main families:

- `QUERY_HISTORY`
- `LOGIN_HISTORY`
- `WAREHOUSE_METERING_HISTORY`
- `STORAGE_USAGE`
- `DATABASES`
- `TABLES`
- `VIEWS`
- `USERS`
- `ROLES`
- `RESOURCE_MONITORS`
- `MASKING_POLICIES`
- `POLICY_REFERENCES`
- `COPY_HISTORY`
- `LOAD_HISTORY`

---

## High-value Views for Exam

### QUERY_HISTORY
Used to monitor:
- query executions
- performance
- execution time
- query type

### LOGIN_HISTORY
Used to monitor:
- successful / failed logins
- login activity
- client connection behavior

### WAREHOUSE_METERING_HISTORY
Used to monitor:
- warehouse credit usage over time

### STORAGE_USAGE
Used to monitor:
- storage consumption over time

### DATABASES / TABLES / VIEWS
Used for:
- object metadata

### MASKING_POLICIES / POLICY_REFERENCES
Used for:
- governance and policy tracking

---

## Access Control

By default, access to SNOWFLAKE database schemas is controlled.

Two common ways to allow access:
- grant `IMPORTED PRIVILEGES` on database `SNOWFLAKE`
- grant specific `SNOWFLAKE` database roles

---

## Important Snowflake Database Roles

Snowflake provides database roles for ACCOUNT_USAGE access:

### OBJECT_VIEWER
Can view object metadata

Examples:
- DATABASES
- TABLES
- VIEWS
- FILE_FORMATS
- PIPES

### USAGE_VIEWER
Can view historical usage

Examples:
- WAREHOUSE_METERING_HISTORY
- STORAGE_USAGE
- TASK_HISTORY
- LOAD_HISTORY

### GOVERNANCE_VIEWER
Can view governance-related information

Examples:
- ACCESS_HISTORY
- MASKING_POLICIES
- TAG_REFERENCES
- ROW_ACCESS_POLICIES

### SECURITY_VIEWER
Can view security-related information

Examples:
- LOGIN_HISTORY
- USERS
- ROLES
- NETWORK_POLICIES
- PASSWORD_POLICIES

### READER_USAGE_VIEWER
Used for `READER_ACCOUNT_USAGE`

---

## Important Exam Notes

- ACCOUNT_USAGE views are not real time
- ACCOUNT_USAGE keeps 1 year of history for many historical views
- ACCOUNT_USAGE includes dropped objects
- Information Schema is more immediate but keeps less history
- `QUERY_HISTORY` in ACCOUNT_USAGE has latency, unlike Information Schema equivalents
- Avoid `SELECT *` because Snowflake-specific views may evolve over time

---

## Cost Reconciliation Tip
If comparing ACCOUNT_USAGE with ORGANIZATION_USAGE cost views:
- set session timezone to UTC first

Important command concept:
`ALTER SESSION SET TIMEZONE = UTC;`

---

## Reader Account Notes
`READER_ACCOUNT_USAGE` is:
- a subset of ACCOUNT_USAGE
- for reader accounts only
- empty if no reader accounts exist

Common reader views:
- `LOGIN_HISTORY`
- `QUERY_HISTORY`
- `RESOURCE_MONITORS`
- `STORAGE_USAGE`
- `WAREHOUSE_METERING_HISTORY`

---

## Certification Memory Sentence

ACCOUNT_USAGE = historical monitoring schema in the shared SNOWFLAKE database, with longer retention, dropped object tracking, and some latency.

Information Schema = more current, less history, no dropped objects.

### ==== 

# Dynamic Tables — Certification Summary

## Definition  
A **Dynamic Table** is a table that **automatically refreshes based on a defined query and target freshness (target lag).**  

Certification memory:  
👉 Dynamic Table = **materialized transformation pipeline managed automatically by Snowflake.**

---

## Key Concept (Very Important for Exam)

When creating a dynamic table you define:

- A transformation query  
- A target freshness (target lag)  

Snowflake automatically:

- Tracks dependencies  
- Schedules refresh  
- Updates the table  

👉 No need for manual pipelines or schedulers.

---

## How Dynamic Tables Work

- Snowflake executes the query defined in `CREATE DYNAMIC TABLE`
- Detects changes in base tables  
- Applies **incremental refresh when possible**
- Uses compute resources associated with the dynamic table  

👉 Focus idea  
Dynamic tables behave like **continuously refreshed materialized transformations.**

---

## Target Lag (Important Exam Concept)

Target lag defines **how fresh the data must be.**  

Example:

- Target lag = 5 minutes  
→ dynamic table data will be **at most 5 minutes behind base tables**  

Impacts:

- Refresh frequency  
- Compute cost  
- Data freshness  

Certification tip  
👉 Lower lag = fresher data + higher cost.

---

## Refresh Behavior

- Refresh happens automatically  
- Can also be refreshed manually  
- Refresh mode defined at creation  

Goal:

- Keep table updated within target lag  

---

## Incremental Processing (Performance Key Point)

Dynamic tables:

- Process only **changed data when possible**
- Avoid full recomputation  
- Improve performance and cost efficiency  

Performance depends on:

- Query complexity  
- Data organization  
- Pipeline design  

Best practice:

👉 Break large pipelines into **multiple smaller dynamic tables.**

---

## Immutability Constraints

Used to:

- Keep specific rows unchanged  
- Allow incremental updates on the rest  

Benefit:

- Protect critical historical data  
- Prevent unwanted refresh modifications  

---

## When to Use Dynamic Tables

Use dynamic tables when:

- You want **automatic transformation pipelines**
- You want to materialize query results without orchestration tools  
- You want to chain multiple transformation steps  
- You want freshness control instead of scheduling control  

Do NOT use when:

- You need very precise scheduling logic  
- You want full manual pipeline orchestration  

---

## Typical Use Cases (Exam Focus)

### Slowly Changing Dimensions (SCD)
- Type 1 and Type 2 implementations  
- Use streams + window functions  

### Precomputed joins and aggregations
- Improve analytical query performance  

### Batch → Streaming transition
- Can switch refresh frequency easily using `ALTER DYNAMIC TABLE`

---

## Certification Memory Sentence

Dynamic Table = automatically refreshed transformation table  
→ freshness controlled by **target lag**, not scheduling.


