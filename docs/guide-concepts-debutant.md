# Technical Reference Guide — Microsoft Fabric Concepts

This guide provides a **technical mapping** of Microsoft Fabric concepts for engineers migrating from Cloudera (Hadoop/Spark on-prem). It assumes familiarity with distributed systems, Spark, SQL engines, and orchestration tools.

---

## Table of Contents

1. [Azure & Cloud Fundamentals](#1-azure--cloud-fundamentals)
2. [Microsoft Fabric Overview](#2-microsoft-fabric-overview)
3. [Tenant & Entra ID (Identity Layer)](#3-tenant--entra-id-identity-layer)
4. [Workspaces](#4-workspaces)
5. [Fabric Capacity (Compute)](#5-fabric-capacity-compute)
6. [Lakehouse](#6-lakehouse)
7. [OneLake](#7-onelake)
8. [Delta Tables](#8-delta-tables)
9. [SQL Analytics Endpoint](#9-sql-analytics-endpoint)
10. [Notebooks](#10-notebooks)
11. [Spark Runtime](#11-spark-runtime)
12. [Data Pipelines](#12-data-pipelines)
13. [REST APIs — Step-by-Step Guide](#13-rest-apis--step-by-step-guide)
14. [Semantic Model](#14-semantic-model)
15. [Semantic Link](#15-semantic-link)
16. [Security (RLS)](#16-security-rls)
17. [Shortcuts](#17-shortcuts)
18. [On-Premises Data Gateway](#18-on-premises-data-gateway)
19. [Glossary](#19-glossary)

---

## 1. Azure & Cloud Fundamentals

### Azure
**Azure** is Microsoft's hyperscale cloud platform. Fabric resources (capacities) are provisioned as Azure resources within a subscription and resource group, just like any other Azure service (Storage Accounts, AKS clusters, etc.).

### Azure Resources
A **resource** is any provisioned entity in Azure — a Fabric capacity, a Key Vault, a storage account. Each resource has:
- A unique **resource ID** (`/subscriptions/{sub}/resourceGroups/{rg}/providers/...`)
- A **region** (e.g., `westeurope`)
- **RBAC** (Role-Based Access Control) managed via Azure IAM

---

## 2. Microsoft Fabric Overview

Microsoft Fabric is a **unified SaaS analytics platform** that consolidates:
- **Data Lake storage** (OneLake, ADLS Gen2 backend)
- **Spark compute** (managed Spark pools, Livy-compatible)
- **SQL compute** (T-SQL endpoint, serverless)
- **Orchestration** (Data Pipelines, ADF-compatible)
- **BI** (Power BI, integrated semantic models)

Under a single **security boundary**, **billing model** (CU-based), and **governance layer**.

### Cloudera → Fabric Architecture Mapping

| Aspect | Cloudera (current) | Fabric (target) |
|--------|----------|--------|
| Storage | HDFS (self-managed) | OneLake (ADLS Gen2, managed) |
| Compute (batch) | Spark on YARN | Managed Spark pools |
| SQL / Query | Hive & Impala (ODBC) | SQL analytics endpoint (serverless) |
| Orchestration | Oozie / Airflow | Data Pipelines (ADF-based) |
| Catalog | Hive Metastore | Lakehouse catalog (Unity-like) |
| BI | External tools | Power BI (native) |
| On-prem connectivity | Direct access | On-premises Data Gateway (ODBC to Hive & Impala) |
| Admin | Ambari / Cloudera Manager | Fabric Admin Portal + Azure Portal |
| Infra | Self-managed cluster nodes | Fully managed (serverless) |

---

## 3. Tenant & Entra ID (Identity Layer)

### Tenant
A **tenant** is your organization's identity boundary in Microsoft cloud. It maps to an **Entra ID directory** (formerly Azure Active Directory). All Fabric workspaces exist within a tenant.

### Entra ID
**Microsoft Entra ID** manages:
- **User identities** (UPN-based: `user@org.onmicrosoft.com`)
- **Groups** (security groups for RBAC)
- **App Registrations / Service Principals** (for programmatic access — CI/CD, external orchestrators)
- **Licenses** (Fabric, Power BI Pro/Premium Per User)

### Key Points for Automation
- **Service Principals** (App Registrations) are the recommended auth method for non-interactive workloads (scripts, pipelines, CI/CD)
- **Managed Identities** are not yet supported for Fabric APIs
- **Guest accounts** (`#EXT#`) have limited capabilities in Fabric — prefer native tenant accounts

---

## 4. Workspaces

A **workspace** is the top-level container in Fabric. All items (Lakehouses, Notebooks, Pipelines, Reports) live inside a workspace.

### Access Roles

| Role | Read | Write | Share | Admin |
|------|------|-------|-------|-------|
| **Viewer** | Yes | No | No | No |
| **Contributor** | Yes | Yes | No | No |
| **Member** | Yes | Yes | Yes | No |
| **Admin** | Yes | Yes | Yes | Yes |

- Roles are assigned to **users, groups, or service principals**
- Workspace roles are **additive** (a Member has all Contributor permissions plus sharing)
- Equivalent to Ranger policies scoping access at the namespace level

---

## 5. Fabric Capacity (Compute)

A **capacity** is the compute engine (CU = Capacity Units) attached to one or more workspaces. It's provisioned as an Azure resource.

| SKU | CUs | Typical Use |
|-----|-----|-------------|
| F2 | 2 | Dev / PoC |
| F8 | 8 | Small team |
| F64 | 64 | Production |
| F128+ | 128+ | Enterprise |

- All users in a workspace share the capacity
- Capacity can be **paused** (cost stops, workspaces become read-only)
- Capacity can be **scaled** up/down without downtime
- Spark, SQL, and Pipeline workloads all consume CUs from the same pool (with burst)

---

## 6. Lakehouse

The **Lakehouse** merges the data lake (raw files) and the data warehouse (structured tables) into a single artifact.

### Structure
```
demolakehouse/
├── Tables/          ← Delta tables (managed, queryable via Spark & SQL)
│   ├── ventes_demo
│   └── produits_demo
└── Files/           ← Raw files (CSV, JSON, Parquet, images...)
    ├── raw/
    ├── staging/
    └── exports/
```

| Zone | Purpose | Cloudera Equivalent |
|------|---------|---------------------|
| `Tables/` | Managed Delta tables, schema-enforced, SQL-queryable | Hive / Impala managed tables |
| `Files/` | Unstructured/raw file storage | HDFS directories |

- `Tables/` are automatically registered in the Lakehouse catalog and exposed via the SQL analytics endpoint
- `Files/` are accessible via `abfss://` paths or Spark's `Files/` relative path

---

## 7. OneLake

**OneLake** is the unified storage layer for the entire Fabric tenant. It's backed by **ADLS Gen2** and uses the `abfss://` protocol.

| HDFS / Hive / Impala (Cloudera) | OneLake (Fabric) |
|-----------------|------------------|
| One HDFS per cluster | One OneLake per tenant |
| `hdfs://namenode/path` | `abfss://workspace@onelake.dfs.fabric.microsoft.com/lakehouse.Lakehouse/...` |
| Hive/Impala tables via ODBC | Delta tables via SQL endpoint or Spark |
| Manual replication (factor 3) | Managed replication (GRS/ZRS) |
| No format enforcement | Delta Lake by default for tables |

### Key Features
- **Cross-workspace access** via shortcuts (no data copying)
- **Compatible** with any ADLS Gen2 client (Azure Storage Explorer, `azcopy`, Spark `abfss://`)
- **OneLake File Explorer** (Windows app) for local file system integration
- **On-premises Data Gateway** support for hybrid connectivity to Hive & Impala via ODBC

---

## 8. Delta Tables

All Lakehouse tables use the **Delta Lake** format (open-source, originally from Databricks).

### Why Delta over raw Parquet/CSV

| Feature | CSV / Parquet | Delta Lake |
|---------|---------------|------------|
| ACID Transactions | No | Yes |
| Schema enforcement | No (Parquet partial) | Yes |
| Time travel / versioning | No | Yes (`VERSION AS OF`) |
| UPDATE / DELETE / MERGE | No | Yes |
| Predicate pushdown | Parquet only | Yes (Z-ordering, data skipping) |
| Concurrent writes | Unsafe | Safe (optimistic concurrency) |

In practice, `df.write.format("delta").saveAsTable("my_table")` is the standard write pattern. The Lakehouse catalog handles registration automatically.

---

## 9. SQL Analytics Endpoint

Each Lakehouse auto-generates a **SQL analytics endpoint** — a serverless T-SQL interface for **read-only queries** over Delta tables.

| Spark (Notebook) | SQL Analytics Endpoint |
|-------------------|------------------------|
| Read + Write | Read-only |
| Spark startup ~30–60s | Instant |
| PySpark / SparkSQL | T-SQL |
| CU: Spark pool | CU: SQL pool |

- Equivalent to **Impala** (ODBC) or **Hive LLAP** in Cloudera — Quadient currently uses both via ODBC through the Data Gateway
- Supports standard T-SQL syntax: `SELECT`, `JOIN`, `GROUP BY`, window functions
- This is where **RLS** (Row-Level Security) can be configured
- Power BI reports typically connect to the SQL endpoint or the semantic model

---

## 10. Notebooks

Fabric Notebooks are **Jupyter-compatible** notebooks with:
- **Languages**: PySpark (default), SparkSQL (`%%sql` magic), Scala, R
- **Built-in Spark session**: `spark` object is pre-configured — no cluster provisioning needed
- **`mssparkutils`**: Fabric-specific utility library (credentials, file system, notebook orchestration)
- **Lakehouse attachment**: Notebooks mount a Lakehouse, making its tables available as `spark.sql("SELECT * FROM my_table")`

### Key Difference from Cloudera
- No `spark-submit`, no YARN queue config, no resource negotiation
- Just open the notebook and run — Spark pool spins up automatically
- Session startup: ~30–60s (first cell), then instant

---

## 11. Spark Runtime

Fabric uses **managed Apache Spark pools** (Spark 3.4+ as of 2025).

- **Same Spark API** as Cloudera — `DataFrame`, `SparkSQL`, `spark.read`, `spark.write` all work identically
- **No cluster management**: no YARN, no node sizing, no driver/executor config (auto-tuned)
- **Libraries**: pre-installed runtime includes pandas, numpy, scikit-learn, semantic-link, plotly, etc. Custom libraries can be added per workspace or per notebook session
- **Livy compatible**: Spark jobs can be submitted via Livy REST endpoint

---

## 12. Data Pipelines

**Data Pipelines** are Fabric's orchestration engine, based on Azure Data Factory (ADF).

### Available Activities

| Activity | Purpose | Cloudera Equivalent |
|----------|---------|---------------------|
| **Notebook** | Run a Spark notebook | Spark Action (Oozie) |
| **Copy Data** | Source → Destination bulk copy | DistCp / Sqoop |
| **Dataflow Gen2** | Visual ETL (Power Query) | N/A |
| **Script** | Run T-SQL | Hive Action |
| **Web** | HTTP call (invoke external API) | Shell Action (curl) |
| **ForEach** | Loop over array | Fork/Join |
| **If Condition** | Conditional branching | Decision node |
| **Set Variable** | Store runtime values | Oozie parameters |
| **Fail** | Force pipeline failure | Kill node |

### Triggers

| Trigger Type | Description | Cloudera Equivalent |
|-------------|-------------|---------------------|
| Manual | On-demand (UI click or API call) | `oozie job -run` |
| Scheduled | Cron-like (hourly, daily, etc.) | Coordinator / cron |
| Tumbling Window | Fixed-size time windows | N/A |
| Event-based | File arrival in OneLake | N/A (required custom dev) |

### How to Create a Pipeline (Click-by-Click)
1. In your workspace → **+ New** → **Data Pipeline**
2. Name it (e.g., `pipeline-daily-etl`)
3. Drag activities from the toolbar onto the canvas
4. Connect them with **success** (green), **failure** (red), or **completion** (blue) arrows
5. Configure each activity (select notebook, set parameters, etc.)
6. Click **Schedule** to set a trigger, or **Run** for manual execution

---

## 13. REST APIs — Step-by-Step Guide

This section provides a **complete, click-by-click walkthrough** of the Fabric REST APIs — how to authenticate, what endpoints to call, and how to integrate with external systems.

### 13.1 What are Fabric REST APIs?

Fabric exposes a full REST API at `https://api.fabric.microsoft.com` that lets you **programmatically manage** workspaces, items, and job execution. Use cases:
- Trigger notebook/pipeline runs from **Jenkins**, **GitHub Actions**, or **Airflow**
- Enumerate workspace items for **inventory/governance scripts**
- Automate Lakehouse creation, item provisioning, or capacity management

### 13.2 Authentication — Two Methods

| Method | When to Use | How |
|--------|-------------|-----|
| **User Identity** (interactive) | Testing from a Fabric notebook | `mssparkutils.credentials.getToken("https://api.fabric.microsoft.com")` |
| **Service Principal** (non-interactive) | Production scripts, CI/CD, external orchestrators | App Registration in Entra ID + client secret/certificate |

#### Option A: User Identity (for testing in Fabric Notebooks)

This is the simplest method. Inside a Fabric notebook:
```python
token = mssparkutils.credentials.getToken("https://api.fabric.microsoft.com")
headers = {"Authorization": f"Bearer {token}"}
```
No additional setup required — uses the identity of the logged-in user.

#### Option B: Service Principal (for production)

**Step-by-step setup in Azure Portal:**

1. **Create an App Registration**
   - Go to [Azure Portal](https://portal.azure.com) → **Microsoft Entra ID** → **App registrations** → **+ New registration**
   - Name: e.g., `fabric-api-quadient`
   - Supported account types: "Accounts in this organizational directory only"
   - Click **Register**

2. **Create a Client Secret**
   - In the App Registration → **Certificates & secrets** → **+ New client secret**
   - Description: e.g., `fabric-api-key`
   - Expiration: choose appropriately (6 months, 12 months, 24 months)
   - **Copy the secret value** immediately (it won't be shown again)

3. **Note the App Details**
   - **Application (client) ID**: found on the app's Overview page
   - **Directory (tenant) ID**: found on the app's Overview page
   - **Client Secret**: the value you copied in step 2

4. **Enable Service Principal Access in Fabric**
   - Go to [Fabric Admin Portal](https://app.fabric.microsoft.com/admin-portal) → **Tenant settings**
   - Search for: **"Service principals can use Fabric APIs"**
   - Enable it → Apply to **the entire organization** or a specific **security group**
   - Also enable: **"Service principals can access read-only admin APIs"** (if needed)

5. **Grant Workspace Access**
   - Go to your workspace in Fabric → **Manage access** (top right)
   - **+ Add people or groups** → Paste the App Registration name → Assign **Contributor** or **Member** role

6. **Acquire a Token Programmatically**
   ```python
   import requests

   tenant_id = "<your-tenant-id>"
   client_id = "<your-client-id>"
   client_secret = "<your-client-secret>"

   token_url = f"https://login.microsoftonline.com/{tenant_id}/oauth2/v2.0/token"
   token_payload = {
       "grant_type": "client_credentials",
       "client_id": client_id,
       "client_secret": client_secret,
       "scope": "https://api.fabric.microsoft.com/.default"
   }

   response = requests.post(token_url, data=token_payload)
   token = response.json()["access_token"]
   headers = {"Authorization": f"Bearer {token}"}
   ```

### 13.3 API Walkthrough — Common Operations

All endpoints use base URL: `https://api.fabric.microsoft.com`

#### List Workspaces
```
GET /v1/workspaces
Authorization: Bearer {token}
```
Returns an array of workspaces the authenticated identity has access to.

#### List Items in a Workspace
```
GET /v1/workspaces/{workspaceId}/items
Authorization: Bearer {token}
```
Returns all items (Lakehouses, Notebooks, Pipelines, Semantic Models, etc.) with their `id`, `displayName`, and `type`.

#### Trigger a Notebook Run
```
POST /v1/workspaces/{workspaceId}/items/{notebookId}/jobs/instances?jobType=RunNotebook
Authorization: Bearer {token}
```
- Returns `202 Accepted` with a `Location` header containing the job monitoring URL
- The notebook runs asynchronously

#### Trigger a Pipeline Run
```
POST /v1/workspaces/{workspaceId}/items/{pipelineId}/jobs/instances?jobType=Pipeline
Authorization: Bearer {token}
```
- Same pattern as notebook: `202 Accepted` + `Location` header

#### Monitor Job Status
```
GET /v1/workspaces/{workspaceId}/items/{itemId}/jobs/instances/{jobInstanceId}
Authorization: Bearer {token}
```
Returns job status: `NotStarted`, `InProgress`, `Completed`, `Failed`, `Cancelled`.

#### Create an Item
```
POST /v1/workspaces/{workspaceId}/items
Authorization: Bearer {token}
Content-Type: application/json

{
  "displayName": "my-new-lakehouse",
  "type": "Lakehouse"
}
```

### 13.4 Quick Reference Table

| Action | Method | Endpoint | Cloudera Equivalent |
|--------|--------|----------|---------------------|
| List workspaces | `GET` | `/v1/workspaces` | N/A |
| List items | `GET` | `/v1/workspaces/{id}/items` | `hdfs dfs -ls` |
| Run Notebook | `POST` | `.../items/{id}/jobs/instances?jobType=RunNotebook` | `spark-submit` |
| Run Pipeline | `POST` | `.../items/{id}/jobs/instances?jobType=Pipeline` | `oozie job -run` |
| Monitor Job | `GET` | `.../items/{id}/jobs/instances/{runId}` | `oozie job -info` |
| Upload file | `PUT` | OneLake API (ADLS Gen2 compatible) | `hdfs dfs -put` |
| Create item | `POST` | `/v1/workspaces/{id}/items` | N/A |
| Refresh semantic model | `POST` | `.../items/{id}/jobs/instances?jobType=RefreshDataset` | N/A |

**Full API documentation**: https://learn.microsoft.com/en-us/rest/api/fabric/core/

### 13.5 Practical Integration Scenarios

#### Scenario 1: External Airflow triggers Fabric
```python
# In your Airflow DAG (PythonOperator)
def trigger_fabric_notebook(**context):
    token = get_fabric_token()  # Service Principal auth
    url = f"https://api.fabric.microsoft.com/v1/workspaces/{WS_ID}/items/{NB_ID}/jobs/instances?jobType=RunNotebook"
    resp = requests.post(url, headers={"Authorization": f"Bearer {token}"})
    job_url = resp.headers["Location"]
    # Poll job_url until status == "Completed"
```

#### Scenario 2: GitHub Actions CI/CD
```yaml
# .github/workflows/run-notebook.yml
- name: Trigger Fabric Notebook
  run: |
    TOKEN=$(curl -s -X POST "https://login.microsoftonline.com/$TENANT_ID/oauth2/v2.0/token" \
      -d "grant_type=client_credentials&client_id=$CLIENT_ID&client_secret=$CLIENT_SECRET&scope=https://api.fabric.microsoft.com/.default" \
      | jq -r '.access_token')
    curl -X POST "https://api.fabric.microsoft.com/v1/workspaces/$WS_ID/items/$NB_ID/jobs/instances?jobType=RunNotebook" \
      -H "Authorization: Bearer $TOKEN"
```

#### Scenario 3: Progressive Cloudera Migration
```
Phase 1 (Now):     Existing Airflow/Oozie → calls Fabric REST APIs
Phase 2 (Migrate): Recreate workflows as native Fabric Data Pipelines
Phase 3 (Target):  Full Fabric: Data Pipelines + Notebooks, no external orchestrator
```

---

## 14. Semantic Model

A **semantic model** (formerly "dataset" in Power BI) is a business logic layer on top of your Lakehouse data. It's the interface between raw Delta tables and consumers (Power BI reports, Semantic Link in notebooks, Excel pivot tables).

### What It Contains

| Component | Description | Example |
|-----------|-------------|---------|
| **Tables** | Mapped from Lakehouse Delta tables | `ventes_demo`, `produits_demo` |
| **Relationships** | Foreign key definitions between tables | `ventes_demo[ProduitId]` → `produits_demo[Id]` |
| **Measures** | DAX expressions — reusable business logic | `Total Revenue := SUM(ventes_demo[CA])` |
| **Calculated columns** | Row-level DAX computations | `Margin := [CA] - [Cost]` |
| **Hierarchies** | Drill-down paths for reports | Year → Quarter → Month |
| **RLS roles** | Row-level security rules | "France" role filters `Filiale = "France"` |

### Default vs Custom Semantic Model

| | Default (auto-generated) | Custom |
|--|--------------------------|--------|
| **Created when** | Lakehouse is created | You click "New semantic model" |
| **Name** | Same as Lakehouse (`demolakehouse`) | Any name you choose |
| **Tables** | All Lakehouse tables auto-included | You select which tables to include |
| **Measures** | None (just raw tables) | You define DAX measures |
| **Relationships** | Auto-detected (best effort) | You define explicitly |
| **When to use** | Quick exploration, Semantic Link tests | Production reports, governed KPIs |

### How to Find the Default Semantic Model (Click-by-Click)

1. Go to your workspace `demo-quadient` in Fabric
2. Look for an item named **`demolakehouse`** with a **bar chart icon** (type: Semantic model)
3. If it's not visible:
   - Open the Lakehouse `demolakehouse`
   - Click the dropdown in the top-right → switch to **SQL analytics endpoint**
   - Go back to the workspace view — the default model should now appear
4. If still missing: Lakehouse → **New semantic model** → select tables → Save

### How to Add Measures and Relationships (Click-by-Click)

**Adding a Measure:**
1. Open the semantic model `demolakehouse` in your workspace
2. Switch to the **Model view** (bottom-left icon, or top tab)
3. Select a table (e.g., `ventes_demo`)
4. In the formula bar at the top, click **New measure**
5. Type the DAX expression:
   ```dax
   Total CA := SUM(ventes_demo[CA])
   ```
6. Press **Enter** — the measure appears under the table
7. Repeat for other KPIs:
   ```dax
   Avg CA := AVERAGE(ventes_demo[CA])
   CA France := CALCULATE(SUM(ventes_demo[CA]), ventes_demo[Filiale] = "France")
   ```

**Adding a Relationship:**
1. In **Model view**, you see all tables as boxes
2. Drag a column from one table to a matching column in another (e.g., `ventes_demo[ProduitId]` → `produits_demo[Id]`)
3. The relationship dialog appears — confirm cardinality (`Many-to-One`) and cross-filter direction
4. Click **OK**

### How to Refresh the Semantic Model

The default semantic model auto-syncs with Lakehouse tables (schema changes propagate). For **data refresh**:

| Method | How |
|--------|-----|
| **Manual** | Open model → **Refresh now** button (top bar) |
| **Scheduled** | Model settings → **Scheduled refresh** → set frequency |
| **Via API** | `POST .../items/{modelId}/jobs/instances?jobType=RefreshDataset` |
| **Via Pipeline** | Add a **"Refresh semantic model"** activity in a Data Pipeline |

### Connection to Power BI Reports

```
Lakehouse (Delta tables)
    ↓ auto-sync
Semantic Model (measures, relationships, RLS)
    ↓ live connection
Power BI Report (visuals, dashboards)
```

- Reports connect to the **semantic model**, not directly to the Lakehouse
- Any measure defined in the model is available in Power BI visuals
- RLS defined in the model is enforced for all report consumers
- **One model can serve multiple reports**

> **Important**: For Notebook 04, the semantic model name is **`demolakehouse`** (matching the Lakehouse name).

---

## 15. Semantic Link

**Semantic Link** (`sempy`) is a Python library (pre-installed in Fabric) that bridges **PySpark notebooks** and **Power BI semantic models**.

### Core Functions

```python
import sempy.fabric as fabric

# List all semantic models in the current workspace
fabric.list_datasets()

# List tables in a specific model
fabric.list_tables("demolakehouse")

# Load a table into a Pandas DataFrame
df = fabric.read_table("demolakehouse", "ventes_demo")

# Execute a DAX query
result = fabric.evaluate_dax("demolakehouse", "EVALUATE SUMMARIZE(...)")

# List measures defined in the model
fabric.list_measures("demolakehouse")
```

### Use Cases
- Data scientists reuse BI-curated data without duplicating ETL logic
- DAX measures are evaluated server-side (respects RLS)
- Feed ML pipelines with business-ready data

---

## 16. Security (RLS)

**Row-Level Security** restricts data visibility at the row level based on the user's identity.

### Current Fabric Support

| Access Method | RLS Enforced? |
|--------------|---------------|
| SQL Analytics Endpoint | Yes |
| Semantic Model (Power BI) | Yes |
| Spark / Notebooks | **No** |
| OneLake Shortcuts (Spark) | **No** (returns 403) |

RLS is configured via T-SQL on the SQL analytics endpoint:
```sql
CREATE FUNCTION dbo.fn_rls_filter(@region NVARCHAR(50))
RETURNS TABLE
WITH SCHEMABINDING
AS
    RETURN SELECT 1 AS result WHERE @region = USER_NAME();

CREATE SECURITY POLICY rls_policy
ADD FILTER PREDICATE dbo.fn_rls_filter(Region) ON dbo.ventes_demo;
```

> **Known limitation**: RLS does not propagate through Spark/OneLake shortcuts. This is a recognized Fabric product gap.

---

## 17. Shortcuts

A **shortcut** is a pointer to data in another Lakehouse or external storage — no data copy, no duplication.

### Supported Sources
- **OneLake** (another Lakehouse in the same or different workspace)
- **ADLS Gen2** (external Azure storage)
- **AWS S3** (cross-cloud)
- **Google Cloud Storage** (cross-cloud)

### How to Create a Shortcut (Click-by-Click)
1. Open the target Lakehouse in Fabric
2. Right-click on **Tables** (or **Files**) → **New shortcut**
3. Select the source type (e.g., **Microsoft OneLake**)
4. Browse to the source Lakehouse → select the table or folder
5. Confirm → the shortcut appears instantly

Analogous to `ln -s` (symlink) in Linux — but over distributed cloud storage.

---

## 18. On-Premises Data Gateway

The **on-premises data gateway** is a bridge between Microsoft Fabric (cloud) and data sources behind a corporate VPN/firewall (on-prem).

### Current Status at Quadient

| Component | Status |
|-----------|--------|
| Gateway installation | Operational |
| ODBC connection to **Hive** | Working |
| ODBC connection to **Impala** | Working |
| Data refresh in Power BI / Fabric | Verified |
| Network (VPN/firewall) | On-prem data securely accessible |

### How It Works

```
On-Prem (behind VPN)              Cloud (Fabric)
┌──────────────────┐              ┌──────────────────┐
│  Cloudera Cluster │              │  Fabric Workspace │
│  ├── Hive (ODBC) │◄────────────►│  ├── Lakehouse    │
│  └── Impala(ODBC)│  Data Gateway│  ├── Pipeline     │
└──────────────────┘              │  └── Power BI     │
                                  └──────────────────┘
```

1. The **gateway agent** runs on a Windows server inside the corporate network
2. It establishes an **outbound HTTPS** connection to Azure Service Bus (no inbound firewall rule needed)
3. Fabric sends queries through this tunnel → gateway executes them against Hive/Impala via ODBC → results flow back
4. Supports **scheduled refresh** (Power BI datasets, Dataflow Gen2) and **live connections**

### Key Considerations

| Topic | Detail |
|-------|--------|
| **Authentication** | Gateway uses the configured ODBC DSN credentials (Kerberos or username/password) |
| **Performance** | Data travels through the gateway — for large volumes, consider migrating data to OneLake instead |
| **High availability** | Multiple gateway instances can form a cluster for failover |
| **Monitoring** | Gateway logs available in Windows Event Viewer + Fabric Admin Portal |
| **Hybrid pattern** | Use the gateway for incremental/live access during migration; target full OneLake for steady-state |

### Step-by-Step: Configure a Hive/Impala Connection in Fabric

1. **Install the gateway** on a Windows server with network access to Cloudera
2. **Install ODBC drivers** (Cloudera Hive ODBC / Impala ODBC) on the gateway machine
3. **Create a system DSN** (ODBC Data Source Administrator → System DSN → Add → configure host, port, auth)
4. **Register the gateway** in Fabric: Settings → Manage connections and gateways → + New → On-premises
5. **Create a connection**: Select gateway cluster → ODBC data source type → enter DSN name + credentials
6. **Use in Pipeline or Dataflow**: In a Copy Data activity or Dataflow Gen2, select the gateway connection as source
7. **Test refresh**: Trigger a manual refresh to validate end-to-end connectivity

### Next Steps
- Review workspace usage, potentially via a new clean Fabric workspace
- Explore triggering Fabric workflows via REST APIs or file-based events (see [Section 13](#13-rest-apis--step-by-step-guide))
- Plan progressive migration: gateway (hybrid) → full OneLake (target state)

---

## 19. Glossary

| Term | Definition |
|------|-----------|
| **Azure** | Microsoft's cloud platform |
| **Tenant** | Organization's identity boundary in Microsoft cloud |
| **Entra ID** | Identity & access management (formerly Azure AD) |
| **Fabric** | Unified SaaS analytics platform |
| **Workspace** | Top-level container for all Fabric items |
| **Capacity** | Compute engine (CU-based), attached to workspaces |
| **Lakehouse** | Unified storage: Delta tables + raw files |
| **OneLake** | Tenant-wide data lake (ADLS Gen2 backend) |
| **Delta Lake** | Open-source table format (ACID, versioning, schema enforcement) |
| **SQL Endpoint** | Serverless T-SQL read-only layer over Lakehouse tables |
| **Notebook** | Interactive Jupyter-compatible code + markdown document |
| **Spark** | Distributed compute engine (managed pools in Fabric) |
| **Pipeline** | Visual orchestration workflow (ADF-based) |
| **REST API** | Programmatic interface to manage Fabric resources |
| **Semantic Model** | Business logic layer (tables + measures + relationships) |
| **Semantic Link** | Python library (`sempy`) bridging notebooks ↔ Power BI |
| **RLS** | Row-Level Security (row-level data filtering by identity) |
| **Shortcut** | Pointer to remote data without copying |
| **DAX** | Data Analysis Expressions (Power BI query language) |
| **PySpark** | Python API for Apache Spark |
| **Token** | OAuth2 bearer token for API authentication |
| **Service Principal** | Non-interactive identity (App Registration) for automation |
| **mssparkutils** | Fabric-specific utility library in notebooks |
| **CU** | Capacity Unit (compute billing unit in Fabric) |
| **Data Gateway** | On-premises bridge for accessing data behind VPN/firewall from Fabric |
| **ODBC** | Open Database Connectivity — standard driver interface for Hive/Impala access |

---

*This guide is part of the [fabric-demo-quadient](../README.md) project.*
