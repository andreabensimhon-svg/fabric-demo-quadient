# Fabric Demo — Cloudera to Microsoft Fabric Migration

## Context

This repository contains a **hands-on technical demo** of Microsoft Fabric for **Quadient**, as part of a migration from **Cloudera** (Hadoop/Spark on-prem) to Microsoft Fabric (unified cloud data platform).

## Demo Objectives

1. **Fabric Workspace** — Understand the workspace abstraction and its components (items, roles, capacity)
2. **Lakehouse & OneLake** — Unified storage layer replacing Hive + Impala on-prem
3. **Data Pipelines & REST APIs** — Orchestration and programmatic automation (replacing Oozie/Airflow), with **step-by-step API walkthrough**
4. **Semantic Link** — Query Power BI semantic models from a PySpark notebook

## Repository Structure

```
fabric-demo-quadient/
├── README.md                          ← This file
├── docs/
│   └── guide-concepts-debutant.md     ← Technical reference guide: Fabric concepts
└── notebooks/
    ├── 01-decouverte-workspace.ipynb   ← Notebook 1: Workspace discovery & Spark environment
    ├── 02-lakehouse-onelake.ipynb      ← Notebook 2: Lakehouse, Delta tables, OneLake
    ├── 03-pipelines-apis.ipynb         ← Notebook 3: Data Pipelines & Fabric REST APIs
    └── 04-semantic-link.ipynb          ← Notebook 4: Semantic Link (read semantic models from Python)
```

## Cloudera → Fabric Mapping

| Cloudera Component | Fabric Equivalent | Notebook |
|--------------------|-------------------|----------|
| Hadoop Cluster | Fabric Workspace + Capacity | 01 |
| Hive / Impala (on-prem, ODBC) | Lakehouse + SQL analytics endpoint | 02 |
| Hive Metastore | Lakehouse catalog (Delta tables, Unity-like) | 02 |
| Spark Submit / Zeppelin | Fabric Notebook (PySpark, managed Spark pools) | 01, 02 |
| Oozie / Airflow | Data Pipeline (visual orchestrator) | 03 |
| Oozie REST API | Fabric REST API (`api.fabric.microsoft.com`) | 03 |
| Ranger / Sentry | Workspace Roles + RLS (SQL endpoint) | 02 |
| On-prem data (VPN/firewall) | On-premises Data Gateway (ODBC) | — |
| N/A | Semantic Link (Python ↔ Power BI bridge) | 04 |

## Prerequisites

- A **Microsoft Fabric tenant** with an active capacity (F2 minimum for testing)
- A user account with **Member** or **Admin** role on a workspace
- A web browser (Edge or Chrome)
- For Notebook 04: the `semantic-link` library (pre-installed in Fabric runtime)

## Environment Details

| Parameter | Value |
|-----------|-------|
| Workspace name | `demo-quadient` |
| Lakehouse name | `demolakehouse` |
| Default semantic model | `demolakehouse` (auto-created with Lakehouse) |
| On-premises Data Gateway | Installed & operational |
| ODBC connections | Hive & Impala (working, data refresh verified) |
| On-prem network | Accessible behind VPN/firewall |

## How to Use This Demo

1. **Read the technical reference**: start with [docs/guide-concepts-debutant.md](docs/guide-concepts-debutant.md) for a mapping of Fabric concepts
2. **Import notebooks into Fabric**: In your workspace → `+ New` → `Import notebook` → select the `.ipynb` files
3. **Attach the Lakehouse**: In each notebook, attach `demolakehouse` from the left panel
4. **Run notebooks in order**: 01 → 02 → 03 → 04
5. **Each notebook** contains markdown explanations + executable code cells

## Recommended Demo Flow (30 min)

| Time | Topic | Material |
|------|-------|----------|
| 0–2 min | Introduction, migration context | Oral |
| 2–10 min | Fabric Workspace (Cloudera parallel) | Notebook 01 + live walkthrough |
| 10–17 min | Lakehouse & OneLake deep-dive | Notebook 02 + live walkthrough |
| 17–25 min | Data Pipelines & REST APIs (step-by-step) | Notebook 03 + live walkthrough |
| 25–28 min | Semantic Link | Notebook 04 |
| 28–30 min | Q&A | — |

## Key Technical Details for the API Section

The API walkthrough in Notebook 03 covers:
- **Token acquisition** using `mssparkutils.credentials.getToken()` (user identity flow)
- **Workspace enumeration** via `GET /v1/workspaces`
- **Item listing** via `GET /v1/workspaces/{id}/items`
- **Notebook execution** via `POST .../jobs/instances?jobType=RunNotebook`
- **Pipeline execution** via `POST .../jobs/instances?jobType=Pipeline`
- **Job monitoring** via `GET .../jobs/instances/{runId}`

For production use, see the guide on setting up a **Service Principal** (App Registration in Entra ID) for automated, non-interactive authentication.

Full API reference: https://learn.microsoft.com/en-us/rest/api/fabric/core/

## Author

Demo prepared by the Microsoft Solution Engineering team.
