# Azure Data Platform Architecture

> **Document Version:** 1.0
> **Last Updated:** YYYY-MM-DD
> **Author(s):** [Name(s)]
> **Reviewers:** [Name(s)]
> **Status:** Draft | In Review | Approved

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Scope and Objectives](#2-scope-and-objectives)
3. [Architecture Overview](#3-architecture-overview)
4. [Environment Strategy](#4-environment-strategy)
5. [Azure Services Inventory](#5-azure-services-inventory)
6. [Azure Data Lake Storage Gen 2 (ADLS Gen2)](#6-azure-data-lake-storage-gen-2-adls-gen2)
7. [Azure Databricks](#7-azure-databricks)
8. [Azure Data Factory (ADF)](#8-azure-data-factory-adf)
9. [Azure Key Vault](#9-azure-key-vault)
10. [Networking and Connectivity](#10-networking-and-connectivity)
11. [Identity and Access Management (IAM)](#11-identity-and-access-management-iam)
12. [Data Governance and Cataloging](#12-data-governance-and-cataloging)
13. [Monitoring, Logging, and Alerting](#13-monitoring-logging-and-alerting)
14. [Security Architecture](#14-security-architecture)
15. [Disaster Recovery and Business Continuity](#15-disaster-recovery-and-business-continuity)
16. [CI/CD and Infrastructure as Code](#16-cicd-and-infrastructure-as-code)
17. [Cost Management](#17-cost-management)
18. [Operational Runbook](#18-operational-runbook)
19. [Naming Conventions](#19-naming-conventions)
20. [Tagging Strategy](#20-tagging-strategy)
21. [Decision Log](#21-decision-log)
22. [Glossary](#22-glossary)
23. [Appendices](#23-appendices)

---

## 1. Executive Summary

Provide a high-level summary of the data platform, its purpose, and the business value it delivers.

> **Example:**
> This document describes the architecture of [Organization Name]'s Azure-based data platform. The platform ingests, transforms, stores, and serves data from [number] source systems to support analytics, reporting, machine learning, and operational decision-making. The platform is deployed across four environments (Development, Test, Acceptance, Production) and follows a Medallion Architecture (Bronze → Silver → Gold) within Azure Data Lake Storage Gen 2.

### 1.1 Key Business Drivers

| # | Business Driver | Description |
|---|-----------------|-------------|
| 1 | [e.g., Centralized Analytics] | [Consolidate data from disparate sources into a single platform] |
| 2 | [e.g., Self-Service BI] | [Enable business users to build reports without IT dependency] |
| 3 | [e.g., Regulatory Compliance] | [Meet GDPR / SOX / HIPAA data handling requirements] |
| 4 | [e.g., Advanced Analytics / ML] | [Enable data science teams to build and deploy models] |

### 1.2 Key Stakeholders

| Role | Name | Responsibility |
|------|------|----------------|
| Platform Owner | [Name] | Strategic direction, budget approval |
| Lead Architect | [Name] | Architecture decisions, technical oversight |
| Data Engineering Lead | [Name] | Pipeline development, data modeling |
| DevOps / Platform Engineer | [Name] | Infrastructure, CI/CD, monitoring |
| Security Officer | [Name] | Security policies, access reviews |
| Data Steward | [Name] | Data quality, governance, catalog |

---

## 2. Scope and Objectives

### 2.1 In Scope

- Data ingestion from [list source systems, e.g., SAP, Salesforce, on-premises SQL Server, REST APIs, file drops]
- Data transformation and enrichment (ETL/ELT)
- Data storage in a lakehouse architecture
- Data serving for BI, reporting, and advanced analytics
- Orchestration and scheduling of data pipelines
- Secrets management and security controls
- Monitoring, alerting, and operational support
- Infrastructure provisioning across four environments

### 2.2 Out of Scope

- [e.g., Front-end application development]
- [e.g., On-premises infrastructure management]
- [e.g., Third-party SaaS integrations not listed in source systems]

### 2.3 Assumptions

| # | Assumption |
|---|------------|
| 1 | All environments share the same Azure tenant: `[tenant-id]` |
| 2 | Each environment has its own Azure subscription (or resource group strategy) |
| 3 | Azure AD is the identity provider for all authentication |
| 4 | Network connectivity to on-premises is established via [ExpressRoute / VPN Gateway] |

### 2.4 Constraints

| # | Constraint |
|---|------------|
| 1 | [e.g., Data must not leave the EU region (West Europe / North Europe)] |
| 2 | [e.g., Maximum monthly platform budget is €X] |
| 3 | [e.g., All infrastructure must be provisioned via IaC — no portal changes in Production] |

---

## 3. Architecture Overview

### 3.1 High-Level Architecture Diagram

> Insert or reference a high-level architecture diagram here.
> Recommended tool: [draw.io / Visio / Lucidchart / Azure Architecture Diagramming tool]
>
> **Diagram location:** `docs/diagrams/high-level-architecture.png`

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           DATA SOURCES                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │   SAP    │  │Salesforce│  │ SQL DBs  │  │REST APIs │  │ Files/   │     │
│  │          │  │          │  │          │  │          │  │ SFTP     │     │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘     │
└───────┼──────────────┼──────────────┼──────────────┼──────────────┼──────────┘
        │              │              │              │              │
        ▼              ▼              ▼              ▼              ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      INGESTION LAYER                                        │
│                   Azure Data Factory                                        │
│          (Pipelines, Linked Services, Triggers)                             │
└──────────────────────────────┬──────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      STORAGE LAYER                                          │
│                 Azure Data Lake Storage Gen 2                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                      │
│  │    BRONZE     │  │    SILVER     │  │     GOLD     │                      │
│  │  (Raw Data)   │  │ (Cleansed)   │  │ (Curated)    │                      │
│  └──────────────┘  └──────────────┘  └──────────────┘                      │
└──────────────────────────────┬──────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                   PROCESSING / TRANSFORMATION LAYER                         │
│                        Azure Databricks                                     │
│      (Notebooks, Jobs, Delta Lake, Unity Catalog, MLflow)                   │
└──────────────────────────────┬──────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      SERVING / CONSUMPTION LAYER                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │  Power BI    │  │Azure Synapse │  │   APIs /     │  │  ML Model    │   │
│  │              │  │  Serverless  │  │  Apps        │  │  Serving     │   │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    CROSS-CUTTING CONCERNS                                    │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────────────┐   │
│  │ Key Vault  │  │ Monitor /  │  │ Azure AD / │  │ Microsoft Purview  │   │
│  │            │  │ Log Analyt.│  │  RBAC      │  │  (Governance)      │   │
│  └────────────┘  └────────────┘  └────────────┘  └────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Data Flow Summary

| Step | Source | Process | Destination | Frequency |
|------|--------|---------|-------------|-----------|
| 1 | [Source System] | ADF Copy Activity | ADLS Bronze | [e.g., Daily 02:00 UTC] |
| 2 | ADLS Bronze | Databricks Notebook (cleansing) | ADLS Silver | [e.g., Daily 03:00 UTC] |
| 3 | ADLS Silver | Databricks Notebook (aggregation) | ADLS Gold | [e.g., Daily 04:00 UTC] |
| 4 | ADLS Gold | Power BI DirectQuery / Import | Power BI Datasets | [e.g., Daily 06:00 UTC] |

### 3.3 Medallion Architecture

| Layer | Purpose | Data Format | Retention | Example Contents |
|-------|---------|-------------|-----------|------------------|
| **Bronze** (Raw) | Exact copy of source data, immutable | Parquet / JSON / CSV (as-is from source) | [e.g., 7 years] | Raw SAP extracts, API responses, file drops |
| **Silver** (Cleansed) | Deduplicated, typed, validated, conformed | Delta Lake | [e.g., 5 years] | Cleansed customer records, validated transactions |
| **Gold** (Curated) | Business-level aggregations, KPI-ready | Delta Lake | [e.g., 3 years] | Revenue by region, customer 360 views, ML feature tables |

---

## 4. Environment Strategy

### 4.1 Environment Overview

The platform is deployed across four isolated environments. Each environment is provisioned with its own set of Azure resources to ensure separation of concerns, security, and stability.

| Environment | Abbreviation | Purpose | Azure Subscription / Resource Group | Region |
|-------------|-------------|---------|-------------------------------------|--------|
| **Development** | DEV | Feature development, experimentation, unit testing | `sub-data-dev` / `rg-data-dev` | [e.g., West Europe] |
| **Test** | TST | Integration testing, regression testing, QA validation | `sub-data-tst` / `rg-data-tst` | [e.g., West Europe] |
| **Acceptance (UAT)** | ACC | User acceptance testing, pre-production validation, performance testing | `sub-data-acc` / `rg-data-acc` | [e.g., West Europe] |
| **Production** | PRD | Live workloads, production data processing | `sub-data-prd` / `rg-data-prd` | [e.g., West Europe] |

### 4.2 Environment Topology Diagram

> Insert a diagram showing the four environments side by side, each containing the same set of services.
>
> **Diagram location:** `docs/diagrams/environment-topology.png`

### 4.3 Environment Isolation Strategy

| Concern | Strategy |
|---------|----------|
| **Subscription Isolation** | [Each environment in a separate subscription / All in one subscription with resource groups] |
| **Network Isolation** | [Separate VNets per environment / VNet peering only from ACC↔PRD for migration testing] |
| **Data Isolation** | [No production data in non-production environments / Anonymized subsets in TST/ACC] |
| **Access Isolation** | [Separate Azure AD groups per environment / Different RBAC role assignments] |
| **Secrets Isolation** | [Separate Key Vault per environment / No cross-environment secret access] |

### 4.4 Environment Sizing

| Resource | DEV | TST | ACC | PRD |
|----------|-----|-----|-----|-----|
| ADLS Gen2 Storage | [e.g., LRS, Hot] | [e.g., LRS, Hot] | [e.g., ZRS, Hot] | [e.g., GRS, Hot+Cool tiering] |
| Databricks SKU | [Premium] | [Premium] | [Premium] | [Premium] |
| Databricks Cluster (default) | [e.g., 2-4 workers, Standard_DS3_v2] | [e.g., 2-8 workers, Standard_DS3_v2] | [e.g., 2-8 workers, Standard_DS4_v2] | [e.g., 4-16 workers, Standard_DS4_v2] |
| ADF Integration Runtime | [Auto-resolve] | [Auto-resolve] | [Self-hosted + Auto-resolve] | [Self-hosted + Auto-resolve] |
| Key Vault SKU | [Standard] | [Standard] | [Standard] | [Premium (HSM-backed)] |

### 4.5 Promotion Path

```
DEV  ──►  TST  ──►  ACC  ──►  PRD
 │          │         │         │
 └──────────┴─────────┴─────────┘
        CI/CD Pipeline (Azure DevOps / GitHub Actions)
        Infrastructure: Terraform / Bicep
        Application: ADF ARM templates, Databricks bundles / repos
```

| Stage | Gate | Approver |
|-------|------|----------|
| DEV → TST | Automated tests pass, code review approved | Dev Lead |
| TST → ACC | Integration tests pass, QA sign-off | QA Lead |
| ACC → PRD | UAT sign-off, change advisory board approval | Platform Owner + CAB |

---

## 5. Azure Services Inventory

### 5.1 Resource Inventory per Environment

| Azure Service | Resource Name (PRD example) | Resource Type | SKU / Tier | Purpose |
|---------------|----------------------------|---------------|------------|---------|
| ADLS Gen2 | `stadlsprd[suffix]` | Storage Account | Standard LRS/GRS, HNS enabled | Data Lake storage |
| Azure Databricks | `dbw-data-prd` | Databricks Workspace | Premium | Data processing, analytics, ML |
| Azure Data Factory | `adf-data-prd` | Data Factory v2 | — | Data ingestion, orchestration |
| Azure Key Vault | `kv-data-prd` | Key Vault | Standard / Premium | Secrets, keys, certificates |
| Azure Monitor | `log-data-prd` | Log Analytics Workspace | Per-GB | Centralized logging |
| Application Insights | `appi-data-prd` | Application Insights | — | ADF / pipeline telemetry |
| Virtual Network | `vnet-data-prd` | Virtual Network | — | Network isolation |
| Network Security Group | `nsg-data-prd` | NSG | — | Network traffic control |
| Private Endpoints | `pe-adls-prd`, `pe-kv-prd` | Private Endpoint | — | Private connectivity to PaaS |
| Azure DNS Private Zone | `privatelink.dfs.core.windows.net` | DNS Zone | — | DNS resolution for private endpoints |
| Microsoft Purview | `pview-data-prd` | Purview Account | — | Data governance, catalog, lineage |
| Azure SQL Database | `sql-meta-prd` | SQL Database | [Tier] | Metadata store (if applicable) |
| Azure DevOps / GitHub | — | — | — | CI/CD, source control |

### 5.2 Resource Group Structure

```
Subscription: sub-data-prd
├── rg-data-prd-core          (ADLS, ADF, Key Vault, Databricks)
├── rg-data-prd-network       (VNet, NSGs, Private Endpoints, DNS Zones)
├── rg-data-prd-monitor       (Log Analytics, Application Insights, Alerts)
├── rg-data-prd-governance    (Purview, metadata stores)
└── rg-data-prd-managed       (Databricks managed resource group — auto-created)
```

> Adjust the resource group strategy to match your organization's conventions.

---

## 6. Azure Data Lake Storage Gen 2 (ADLS Gen2)

### 6.1 Overview

| Property | Value |
|----------|-------|
| **Service** | Azure Data Lake Storage Gen 2 (Storage Account with HNS enabled) |
| **Purpose** | Centralized data lake for all raw, cleansed, and curated data |
| **Account Kind** | StorageV2 |
| **Hierarchical Namespace** | Enabled |
| **Replication (PRD)** | [GRS / ZRS / RA-GRS] |
| **Replication (Non-PRD)** | [LRS] |
| **Access Tier (default)** | Hot |
| **Minimum TLS Version** | 1.2 |
| **Public Network Access** | Disabled (Private Endpoint only) |

### 6.2 Container (Filesystem) Layout

| Container | Layer | Description | Access Pattern |
|-----------|-------|-------------|----------------|
| `bronze` | Raw | Exact copies of source data, partitioned by source and ingestion date | Write: ADF; Read: Databricks |
| `silver` | Cleansed | Deduplicated, schema-enforced, conformed data (Delta format) | Write: Databricks; Read: Databricks |
| `gold` | Curated | Business-ready aggregations, KPIs, feature stores (Delta format) | Write: Databricks; Read: Power BI, Synapse, APIs |
| `sandbox` | Exploration | Ad-hoc analysis, data science experimentation | Write/Read: Data Scientists (controlled access) |
| `config` | Configuration | Schema definitions, mapping tables, reference data | Write: CI/CD; Read: ADF, Databricks |
| `archive` | Archive | Historical snapshots, regulatory retention | Write: Lifecycle policy; Read: On-demand |

### 6.3 Folder Structure and Partitioning

```
bronze/
├── {source_system}/
│   ├── {entity}/
│   │   ├── year=YYYY/
│   │   │   ├── month=MM/
│   │   │   │   ├── day=DD/
│   │   │   │   │   ├── {source}_{entity}_{timestamp}.parquet
│   │   │   │   │   └── _metadata/
│   │   │   │   │       └── _ingestion_log.json

silver/
├── {domain}/
│   ├── {entity}/
│   │   ├── _delta_log/
│   │   ├── part-00000-*.parquet
│   │   └── ...

gold/
├── {domain}/
│   ├── {dataset_or_aggregate}/
│   │   ├── _delta_log/
│   │   ├── part-00000-*.parquet
│   │   └── ...
```

### 6.4 Lifecycle Management Policies

| Rule Name | Scope | Condition | Action |
|-----------|-------|-----------|--------|
| `archive-bronze-after-90d` | `bronze/` | Last modified > 90 days | Move to Cool tier |
| `archive-bronze-after-365d` | `bronze/` | Last modified > 365 days | Move to Archive tier |
| `delete-sandbox-after-30d` | `sandbox/` | Last modified > 30 days | Delete blob |
| `delete-soft-deleted-after-7d` | All containers | Soft-deleted > 7 days | Permanently delete |

### 6.5 Security Configuration

| Setting | Value |
|---------|-------|
| **Authentication** | Azure AD (OAuth 2.0); no shared key access in PRD |
| **Authorization** | Azure RBAC + ACLs at folder level |
| **Encryption at Rest** | Microsoft-managed keys (or CMK via Key Vault) |
| **Encryption in Transit** | TLS 1.2 enforced |
| **Soft Delete** | Enabled (7-day retention) |
| **Versioning** | [Enabled / Disabled — state your choice and rationale] |
| **Firewall** | Deny all public access; allow via Private Endpoint + trusted Azure services |
| **Diagnostic Logging** | Enabled → Log Analytics Workspace |

### 6.6 ACL and RBAC Matrix

| Principal | Container | RBAC Role / ACL | Justification |
|-----------|-----------|-----------------|---------------|
| `sp-adf-prd` (ADF MSI) | `bronze` | Storage Blob Data Contributor | ADF writes ingested data |
| `sp-dbw-prd` (Databricks MSI / SPN) | `bronze`, `silver`, `gold` | Storage Blob Data Contributor | Databricks reads/writes across layers |
| `sg-data-engineers` (AD Group) | `silver`, `gold` | Storage Blob Data Reader | Engineers can browse/debug data |
| `sg-data-scientists` (AD Group) | `gold`, `sandbox` | Storage Blob Data Contributor (sandbox), Reader (gold) | Exploration in sandbox, read curated data |
| `sp-powerbi-prd` (Power BI SPN) | `gold` | Storage Blob Data Reader | Power BI reads curated data |

---

## 7. Azure Databricks

### 7.1 Overview

| Property | Value |
|----------|-------|
| **Service** | Azure Databricks |
| **SKU** | Premium (required for Unity Catalog, RBAC, Private Link) |
| **Purpose** | Data transformation (ETL/ELT), analytics, ML model training, Delta Lake management |
| **Deployment** | VNet-injected into dedicated subnets |
| **Unity Catalog** | [Enabled / Planned — describe status] |
| **Runtime Version** | [e.g., DBR 14.x LTS or later] |

### 7.2 Workspace Configuration

| Setting | DEV | TST | ACC | PRD |
|---------|-----|-----|-----|-----|
| Workspace URL | `adb-xxxx.azuredatabricks.net` | `adb-yyyy.azuredatabricks.net` | `adb-zzzz.azuredatabricks.net` | `adb-wwww.azuredatabricks.net` |
| VNet Injection | Yes | Yes | Yes | Yes |
| Public Subnet | `snet-dbw-public-{env}` | — | — | — |
| Private Subnet | `snet-dbw-private-{env}` | — | — | — |
| No Public IP (Secure Cluster Connectivity) | [Yes/No] | — | — | [Yes] |
| Unity Catalog Metastore | [Shared / Per-env] | — | — | — |

### 7.3 Cluster Policies

| Policy Name | Scope | Configuration | Rationale |
|-------------|-------|---------------|-----------|
| `default-job-cluster` | Job clusters | Min workers: 1, Max workers: [N], Node type: [type], Auto-terminate: 20 min, Spot instances: [Yes/No] | Cost-optimized for batch jobs |
| `interactive-dev` | DEV only | Max workers: 4, Auto-terminate: 60 min, Single user mode | Development and debugging |
| `high-concurrency-shared` | All envs | Shared mode, Table ACLs, Max workers: [N] | Multi-user interactive analytics |
| `ml-gpu-cluster` | DEV, PRD | GPU-enabled nodes, ML runtime, Max workers: [N] | Model training |

### 7.4 Cluster Configurations

| Cluster Name | Type | Node Type | Workers | Auto-terminate | Spark Config |
|-------------|------|-----------|---------|----------------|--------------|
| `cluster-etl-{env}` | Job | `Standard_DS3_v2` | 2–8 (autoscale) | 20 min | `spark.sql.shuffle.partitions=200` |
| `cluster-interactive-{env}` | All-purpose | `Standard_DS3_v2` | 1–4 (autoscale) | 120 min | — |
| `cluster-ml-{env}` | All-purpose | `Standard_NC6s_v3` | 1–2 | 60 min | ML Runtime |

### 7.5 Unity Catalog Structure

```
Metastore: metastore-{region}
├── Catalog: catalog_{env}          (one catalog per environment)
│   ├── Schema: bronze
│   │   ├── Table: {source}_{entity}
│   │   └── ...
│   ├── Schema: silver
│   │   ├── Table: {domain}_{entity}
│   │   └── ...
│   ├── Schema: gold
│   │   ├── Table: {domain}_{aggregate}
│   │   └── ...
│   └── Schema: sandbox
│       └── ...
```

### 7.6 Databricks Repos / Asset Bundles

| Repository | Branch Strategy | Deployment |
|------------|----------------|------------|
| `databricks-notebooks` | `main` → PRD, `develop` → DEV, feature branches | Databricks Asset Bundles / Repos sync |
| `databricks-jobs` | Same as above | Job definitions in YAML, deployed via CI/CD |
| `databricks-libraries` | Tagged releases | Wheels/JARs uploaded to DBFS or Unity Catalog Volumes |

### 7.7 Secret Scopes

| Scope Name | Backend | Contents |
|------------|---------|----------|
| `kv-{env}` | Azure Key Vault-backed | All secrets from the environment's Key Vault |
| `internal` | Databricks-managed | Databricks-specific tokens (if any) |

### 7.8 Libraries and Dependencies

| Library | Version | Scope | Installation Method |
|---------|---------|-------|---------------------|
| [e.g., `great-expectations`] | [version] | Data quality validation | Cluster init script / `%pip install` |
| [e.g., `delta-spark`] | Bundled with DBR | Delta Lake operations | Pre-installed |
| [e.g., custom wheel] | [version] | Internal utilities | Unity Catalog Volume / DBFS |

---

## 8. Azure Data Factory (ADF)

### 8.1 Overview

| Property | Value |
|----------|-------|
| **Service** | Azure Data Factory v2 |
| **Purpose** | Data ingestion (source → Bronze), pipeline orchestration, triggering Databricks jobs |
| **Managed VNet** | [Enabled / Disabled] |
| **Git Integration** | [Azure DevOps Repos / GitHub — branch: `main` for PRD, `develop` for DEV] |
| **Global Parameters** | `environment`, `keyVaultUrl`, `storageAccountName`, `databricksWorkspaceUrl` |

### 8.2 Integration Runtimes

| Runtime Name | Type | Purpose | Environment(s) |
|-------------|------|---------|-----------------|
| `AutoResolveIntegrationRuntime` | Azure (auto-resolve) | Cloud-to-cloud data movement | All |
| `ir-selfhosted-{env}` | Self-hosted | On-premises connectivity (e.g., SQL Server, file shares) | ACC, PRD |
| `ir-managed-vnet-{env}` | Azure (managed VNet) | Secure cloud data movement via private endpoints | PRD |

### 8.3 Linked Services

| Linked Service Name | Target Service | Authentication | Secret Storage |
|--------------------|----------------|----------------|----------------|
| `ls_adls_{env}` | ADLS Gen2 | Managed Identity | — |
| `ls_keyvault_{env}` | Azure Key Vault | Managed Identity | — |
| `ls_databricks_{env}` | Azure Databricks | Managed Identity / Access Token (from KV) | Key Vault |
| `ls_sqlserver_onprem` | On-premises SQL Server | SQL Auth (password from KV) | Key Vault |
| `ls_rest_api_{name}` | External REST API | API Key / OAuth (from KV) | Key Vault |
| `ls_sftp_{name}` | SFTP Server | SSH Key / Password (from KV) | Key Vault |

### 8.4 Pipeline Architecture

```
Master Pipeline (pl_master_{domain})
├── Validation Activity        (check source availability)
├── For Each (source entities)
│   ├── Copy Activity           (source → Bronze)
│   │   ├── Source Dataset      (parameterized)
│   │   └── Sink Dataset        (ADLS Bronze, parameterized path)
│   └── Stored Procedure        (log ingestion metadata)
├── Databricks Notebook Activity (Bronze → Silver transformation)
├── Databricks Notebook Activity (Silver → Gold transformation)
├── Notification Activity       (success/failure email or Teams webhook)
└── Error Handling
    ├── Web Activity             (send alert to monitoring)
    └── Set Variable             (error details for downstream)
```

### 8.5 Pipeline Inventory

| Pipeline Name | Domain | Source(s) | Schedule | SLA |
|---------------|--------|-----------|----------|-----|
| `pl_ingest_{source}_{entity}` | [Domain] | [Source System] | [Cron expression / Trigger type] | [e.g., Complete by 06:00 UTC] |
| `pl_transform_bronze_silver_{domain}` | [Domain] | ADLS Bronze | After ingestion | [e.g., 2 hours after ingestion] |
| `pl_transform_silver_gold_{domain}` | [Domain] | ADLS Silver | After Silver | [e.g., 4 hours after ingestion] |
| `pl_master_{domain}` | [Domain] | End-to-end | [Daily / Hourly] | [SLA] |

### 8.6 Triggers

| Trigger Name | Type | Schedule / Event | Pipelines |
|-------------|------|------------------|-----------|
| `tr_daily_0200utc` | Schedule | Daily at 02:00 UTC | `pl_master_sales`, `pl_master_finance` |
| `tr_hourly` | Schedule | Every hour | `pl_ingest_api_events` |
| `tr_blob_arrived_{source}` | Storage Event | Blob created in `bronze/{source}/landing/` | `pl_ingest_{source}_filewatch` |
| `tr_tumbling_window_daily` | Tumbling Window | Daily, 24h window | `pl_ingest_historical_backfill` |

### 8.7 Parameterization Strategy

| Parameter | Scope | Example Values | Purpose |
|-----------|-------|----------------|---------|
| `p_source_system` | Pipeline | `sap`, `salesforce`, `sqlserver` | Dynamic source selection |
| `p_entity_name` | Pipeline | `customers`, `orders`, `invoices` | Dynamic entity selection |
| `p_load_date` | Pipeline | `2024-01-15` | Partition and filter source data |
| `p_environment` | Global | `dev`, `tst`, `acc`, `prd` | Environment-aware resource names |
| `p_full_or_incremental` | Pipeline | `full`, `incremental` | Load type |

### 8.8 Error Handling and Retry Strategy

| Aspect | Configuration |
|--------|---------------|
| **Activity Retry** | 3 retries, 30-second interval (exponential backoff) |
| **Pipeline Failure** | Failure path → Log error to metadata DB → Send alert (email/Teams) |
| **Timeout** | Default: 12 hours; Copy activities: 4 hours |
| **Concurrency** | Max concurrent pipeline runs: [N] |
| **Dead Letter** | Failed records written to `bronze/{source}/{entity}/_errors/` |

---

## 9. Azure Key Vault

### 9.1 Overview

| Property | Value |
|----------|-------|
| **Service** | Azure Key Vault |
| **Purpose** | Centralized secrets, keys, and certificate management for all data platform services |
| **SKU (PRD)** | [Premium (HSM-backed) / Standard] |
| **SKU (Non-PRD)** | Standard |
| **Soft Delete** | Enabled (90-day retention) |
| **Purge Protection** | Enabled (PRD); [Enabled/Disabled] (Non-PRD) |
| **Public Network Access** | Disabled (Private Endpoint only) |

### 9.2 Secrets Inventory

| Secret Name Pattern | Description | Consumers | Rotation Policy |
|--------------------|-------------|-----------|-----------------|
| `secret-sqlserver-{name}-password` | On-prem SQL Server credentials | ADF | [e.g., 90 days] |
| `secret-sftp-{name}-password` | SFTP server credentials | ADF | [e.g., 180 days] |
| `secret-api-{name}-key` | External API keys | ADF, Databricks | [Per API provider policy] |
| `secret-spn-{name}-client-secret` | Service principal client secrets | Multiple | [e.g., 365 days, automated rotation] |
| `secret-databricks-pat` | Databricks personal access token (if not using MSI) | ADF | [e.g., 90 days] |
| `secret-storage-account-key` | Storage account key (if needed) | Legacy integrations | [e.g., 90 days] |

### 9.3 Access Policies / RBAC

| Principal | Role / Policy | Permissions |
|-----------|---------------|-------------|
| `sp-adf-{env}` (ADF Managed Identity) | Key Vault Secrets User | GET secrets |
| `sp-dbw-{env}` (Databricks SPN) | Key Vault Secrets User | GET secrets |
| `sg-data-platform-admins` | Key Vault Administrator | Full management |
| `sg-data-engineers` | Key Vault Secrets Officer | GET, LIST, SET secrets (non-PRD only) |
| CI/CD Service Principal | Key Vault Secrets Officer | SET secrets during deployment |

### 9.4 Key Vault Networking

| Setting | Value |
|---------|-------|
| **Default Action** | Deny |
| **Private Endpoint** | `pe-kv-{env}` in `snet-privatelink-{env}` |
| **Trusted Services** | Allow trusted Microsoft services |
| **IP Rules** | [None / specific IPs for emergency access] |

### 9.5 Secrets Rotation Procedure

1. New secret value generated (manually or via automation)
2. New version created in Key Vault (old version remains accessible until expiry)
3. Dependent services pick up new version automatically (ADF, Databricks secret scopes)
4. Validation in DEV → TST → ACC → PRD
5. Old version disabled after confirmation
6. Audit log reviewed

---

## 10. Networking and Connectivity

### 10.1 Network Topology

| Component | CIDR / Range | Environment | Purpose |
|-----------|-------------|-------------|---------|
| `vnet-data-{env}` | `10.{x}.0.0/16` | Per env | Main VNet |
| `snet-databricks-public-{env}` | `10.{x}.1.0/24` | Per env | Databricks public subnet (NAT) |
| `snet-databricks-private-{env}` | `10.{x}.2.0/24` | Per env | Databricks private subnet (workers) |
| `snet-privatelink-{env}` | `10.{x}.3.0/24` | Per env | Private endpoints for PaaS services |
| `snet-integration-{env}` | `10.{x}.4.0/24` | Per env | Self-hosted IR VMs, jump boxes |

### 10.2 Private Endpoints

| Resource | Private Endpoint Name | Sub-resource | Private DNS Zone |
|----------|-----------------------|-------------|------------------|
| ADLS Gen2 | `pe-adls-dfs-{env}` | `dfs` | `privatelink.dfs.core.windows.net` |
| ADLS Gen2 | `pe-adls-blob-{env}` | `blob` | `privatelink.blob.core.windows.net` |
| Key Vault | `pe-kv-{env}` | `vault` | `privatelink.vaultcore.azure.net` |
| Databricks | `pe-dbw-{env}` | `databricks_ui_api` | `privatelink.azuredatabricks.net` |
| SQL Database | `pe-sql-{env}` | `sqlServer` | `privatelink.database.windows.net` |
| ADF | `pe-adf-{env}` | `dataFactory` | `privatelink.datafactory.azure.net` |

### 10.3 Network Security Groups (NSGs)

| NSG Name | Associated Subnet | Key Rules |
|----------|-------------------|-----------|
| `nsg-dbw-public-{env}` | Databricks public | [Databricks-required rules — see Microsoft docs] |
| `nsg-dbw-private-{env}` | Databricks private | [Databricks-required rules] |
| `nsg-privatelink-{env}` | Private endpoints | Deny all inbound except VNet; allow 443 from known subnets |
| `nsg-integration-{env}` | Integration / IR | Allow 443 outbound; allow RDP/SSH from bastion only |

### 10.4 On-Premises Connectivity

| Property | Value |
|----------|-------|
| **Connection Type** | [ExpressRoute / Site-to-Site VPN / Point-to-Site VPN] |
| **Gateway** | `vgw-data-{env}` |
| **On-Prem Address Space** | `192.168.x.0/16` (example) |
| **Bandwidth** | [e.g., 1 Gbps ExpressRoute] |
| **Failover** | [e.g., VPN backup for ExpressRoute] |

### 10.5 DNS Resolution

| DNS Zone | Purpose | Linked VNets |
|----------|---------|-------------|
| `privatelink.dfs.core.windows.net` | ADLS private endpoint resolution | All env VNets |
| `privatelink.vaultcore.azure.net` | Key Vault private endpoint resolution | All env VNets |
| `privatelink.azuredatabricks.net` | Databricks private endpoint resolution | All env VNets |
| Custom DNS Forwarder | On-prem DNS resolution | Hub VNet (if hub-spoke) |

---

## 11. Identity and Access Management (IAM)

### 11.1 Authentication Strategy

| Service | Authentication Method |
|---------|----------------------|
| ADLS Gen2 | Azure AD (Managed Identity, Service Principals) |
| Databricks | Azure AD SSO (SCIM provisioning for users/groups) |
| ADF | Managed Identity |
| Key Vault | Azure AD (Managed Identity, Service Principals) |
| Power BI | Azure AD (Service Principal for automated refresh) |

### 11.2 Service Principals and Managed Identities

| Identity | Type | Purpose | Key Vault Access | ADLS Access | Databricks Access |
|----------|------|---------|-------------------|-------------|-------------------|
| `sp-adf-{env}` | System-assigned MI | ADF pipeline execution | Secrets User | Blob Data Contributor (Bronze) | Notebook execution |
| `sp-dbw-{env}` | User-assigned MI / SPN | Databricks → ADLS, KV | Secrets User | Blob Data Contributor (all layers) | — |
| `sp-cicd-{env}` | SPN | CI/CD deployments | Secrets Officer | — | Workspace admin (deploy only) |
| `sp-powerbi-{env}` | SPN | Power BI data refresh | — | Blob Data Reader (Gold) | SQL Warehouse access |

### 11.3 Azure AD Groups

| Group Name | Members | Purpose |
|------------|---------|---------|
| `sg-data-platform-admins` | Platform team | Full admin access to all resources |
| `sg-data-engineers-{env}` | Data engineers | Contributor on ADF, Databricks; Reader on ADLS |
| `sg-data-scientists-{env}` | Data scientists | Databricks user, ADLS sandbox contributor |
| `sg-data-analysts-{env}` | Business analysts | Databricks SQL user, Gold reader |
| `sg-data-viewers-{env}` | Stakeholders | Read-only access to Gold / Power BI |

### 11.4 RBAC Role Assignments Summary

| Scope | Principal | Role | Environment |
|-------|-----------|------|-------------|
| Resource Group | `sg-data-platform-admins` | Contributor | All |
| Storage Account | `sp-adf-{env}` | Storage Blob Data Contributor | Per env |
| Storage Account | `sp-dbw-{env}` | Storage Blob Data Contributor | Per env |
| Key Vault | `sp-adf-{env}` | Key Vault Secrets User | Per env |
| Databricks Workspace | `sg-data-engineers-{env}` | Contributor | Per env |

### 11.5 Privileged Access Management

| Control | Implementation |
|---------|----------------|
| Just-in-Time Access | [Azure AD PIM for elevated roles] |
| Break-glass Account | [Dedicated emergency account with MFA, monitored] |
| Access Reviews | [Quarterly review of all role assignments] |
| Conditional Access | [MFA required, compliant device, approved location] |

---

## 12. Data Governance and Cataloging

### 12.1 Microsoft Purview (or alternative)

| Property | Value |
|----------|-------|
| **Service** | [Microsoft Purview / Azure Purview / Unity Catalog / Other] |
| **Purpose** | Data cataloging, lineage tracking, classification, access governance |
| **Scanned Sources** | ADLS Gen2 (all layers), Azure SQL, Databricks (via Unity Catalog) |
| **Scan Frequency** | [Weekly / Daily / Event-driven] |

### 12.2 Data Classification

| Classification | Description | Examples | Handling |
|---------------|-------------|----------|----------|
| Public | Non-sensitive, open data | Product catalogs, public reference data | No restrictions |
| Internal | Business-internal, not regulated | Sales aggregates, operational metrics | Access-controlled |
| Confidential | PII, financial, regulated | Customer names, email, SSN, revenue | Encrypted, masked, audited |
| Restricted | Highly sensitive | Passwords, keys, health records | Encrypted, minimal access, logged |

### 12.3 Data Quality Framework

| Aspect | Tool / Approach | Description |
|--------|----------------|-------------|
| Schema Validation | [Great Expectations / Databricks expectations / custom] | Validate schema on ingestion |
| Data Quality Rules | [dbt tests / Great Expectations / custom Spark checks] | Null checks, range checks, referential integrity |
| Data Quality Dashboard | [Power BI / Databricks dashboard] | Visibility into quality metrics |
| Alerting | [Azure Monitor / Teams / Email] | Alert on quality threshold breaches |

### 12.4 Data Lineage

| Source | Tool | Granularity |
|--------|------|-------------|
| ADF Pipelines | Purview / ADF built-in | Pipeline → Dataset level |
| Databricks | Unity Catalog lineage / Purview | Column-level lineage (Delta tables) |
| Power BI | Purview integration | Dataset → Report level |

---

## 13. Monitoring, Logging, and Alerting

### 13.1 Monitoring Architecture

| Component | Monitoring Tool | Data Destination |
|-----------|----------------|-----------------|
| ADF Pipeline Runs | ADF Monitor + Diagnostic Settings | Log Analytics Workspace |
| Databricks Clusters/Jobs | Databricks built-in + Diagnostic Logs | Log Analytics / Event Hub |
| ADLS Gen2 | Diagnostic Settings (read/write/delete logs) | Log Analytics Workspace |
| Key Vault | Diagnostic Settings (audit events) | Log Analytics Workspace |
| Network | NSG Flow Logs | Log Analytics / Storage Account |
| Infrastructure | Azure Monitor Metrics | Log Analytics Workspace |

### 13.2 Log Analytics Workspace

| Property | Value |
|----------|-------|
| **Workspace Name** | `log-data-{env}` |
| **Retention** | [30 / 90 / 180 / 365 days] |
| **Daily Cap** | [e.g., 5 GB / day in non-PRD, uncapped in PRD] |

### 13.3 Key Metrics and KPIs

| Metric | Source | Threshold | Alert Severity |
|--------|--------|-----------|----------------|
| ADF Pipeline failure rate | ADF Diagnostic Logs | > 0 failures on critical pipelines | Sev 1 (PRD), Sev 3 (Non-PRD) |
| ADF Pipeline duration | ADF Monitor | > 2x normal duration | Sev 2 |
| Databricks job failure | Databricks Jobs API / Logs | Any failure on scheduled jobs | Sev 1 (PRD) |
| ADLS storage capacity | Azure Monitor Metrics | > 80% of quota | Sev 3 |
| Key Vault throttling | Key Vault Diagnostic Logs | HTTP 429 responses | Sev 2 |
| Cluster auto-scale events | Databricks Logs | Sustained max workers > 1 hour | Sev 3 (informational) |

### 13.4 Alert Configuration

| Alert Name | Condition | Action Group | Notification |
|-----------|-----------|-------------|-------------|
| `alert-adf-pipeline-failure-{env}` | ADF pipeline failed | `ag-data-platform-{env}` | Email + Teams channel |
| `alert-databricks-job-failure-{env}` | Job run status = FAILED | `ag-data-platform-{env}` | Email + Teams + PagerDuty (PRD) |
| `alert-kv-unauthorized-access-{env}` | KV 403 responses | `ag-security-{env}` | Email + Security team |
| `alert-adls-high-egress-{env}` | Egress > [threshold] GB/day | `ag-data-platform-{env}` | Email |

### 13.5 Dashboards

| Dashboard | Tool | Purpose | Audience |
|-----------|------|---------|----------|
| Platform Health | Azure Monitor Workbook | Overall service health, pipeline status | Platform team |
| Data Pipeline Status | Power BI / ADF Monitor | Pipeline run history, SLA tracking | Data engineers, managers |
| Cost Dashboard | Azure Cost Management | Spend by service, environment, team | Platform owner, finance |
| Data Quality | Power BI / Databricks | Quality scores by domain and entity | Data stewards |

---

## 14. Security Architecture

### 14.1 Security Principles

| Principle | Implementation |
|-----------|----------------|
| **Zero Trust** | All services accessed via Private Endpoints; no public access in PRD |
| **Least Privilege** | RBAC with minimal permissions; JIT access for elevated operations |
| **Defense in Depth** | NSGs + Private Endpoints + Firewall + Encryption + Audit Logging |
| **Encryption Everywhere** | TLS 1.2 in transit; AES-256 at rest (Microsoft-managed or CMK) |
| **Separation of Duties** | Distinct roles for admin, engineer, analyst; CI/CD for PRD changes |

### 14.2 Encryption

| Scope | Method | Key Management |
|-------|--------|---------------|
| Data at rest (ADLS) | AES-256 | Microsoft-managed keys / CMK (Key Vault) |
| Data at rest (Databricks) | AES-256 | Databricks-managed / CMK |
| Data in transit | TLS 1.2 | Azure-managed certificates |
| Secrets | Key Vault encryption | HSM-backed (Premium) / Software (Standard) |

### 14.3 Threat Model Summary

| Threat | Mitigation |
|--------|-----------|
| Unauthorized data access | Private Endpoints, RBAC, ACLs, Azure AD Conditional Access |
| Data exfiltration | NSG rules, no public endpoints, managed VNet for ADF, DLP policies |
| Credential leakage | Key Vault for all secrets, Managed Identities where possible, no hard-coded credentials |
| Insider threat | Audit logging, access reviews, PIM, separation of duties |
| Data corruption | Soft delete, Delta Lake time travel, backup policies |

### 14.4 Compliance Requirements

| Regulation | Requirement | Implementation |
|-----------|------------|----------------|
| [GDPR] | Data minimization, right to deletion | PII tagging in Purview, deletion pipelines |
| [SOX] | Audit trail, access controls | Key Vault audit logs, RBAC, change management |
| [HIPAA] | Encryption, access logging | CMK encryption, diagnostic logging |
| [Internal Policy] | Data residency | All resources in [region], geo-replication within allowed regions |

---

## 15. Disaster Recovery and Business Continuity

### 15.1 RPO and RTO Targets

| Component | RPO (Recovery Point Objective) | RTO (Recovery Time Objective) |
|-----------|-------------------------------|-------------------------------|
| ADLS Gen2 (PRD) | [e.g., 1 hour — GRS replication lag] | [e.g., 4 hours] |
| Databricks Workspace | [N/A — stateless compute; config in IaC] | [e.g., 2 hours — redeploy from IaC] |
| ADF Pipelines | [N/A — config in Git/ARM] | [e.g., 1 hour — redeploy from Git] |
| Key Vault | [Near-zero — built-in replication] | [e.g., < 1 hour] |
| Delta Lake Tables | [Time travel retention: 30 days] | [e.g., minutes — RESTORE command] |

### 15.2 Backup Strategy

| Component | Backup Method | Frequency | Retention |
|-----------|-------------|-----------|-----------|
| ADLS Gen2 | GRS replication + soft delete + versioning | Continuous | [Policy-defined] |
| ADF | Git integration (full pipeline definitions in repo) | Every commit | Git history |
| Databricks Notebooks | Git integration (Repos / Asset Bundles) | Every commit | Git history |
| Key Vault | Built-in soft delete + purge protection | Continuous | 90 days |
| Metadata DB | Azure SQL automated backups | Continuous | [7–35 days] |

### 15.3 DR Procedure

1. **Detection:** Azure Monitor alert triggers on service unavailability
2. **Assessment:** Platform team assesses scope and impact
3. **Failover Decision:** If RPO/RTO thresholds are breached, initiate failover
4. **Recovery Steps:**
   - ADLS: Initiate GRS failover (if regional outage) or restore from soft-deleted/versioned data
   - ADF: Redeploy from Git/ARM templates to secondary region
   - Databricks: Deploy new workspace from IaC in secondary region
   - Key Vault: Access replicated vault in paired region
5. **Validation:** Run smoke tests on recovered environment
6. **Communication:** Notify stakeholders of status and ETA

---

## 16. CI/CD and Infrastructure as Code

### 16.1 Source Control

| Repository | Platform | Purpose |
|------------|----------|---------|
| `infra-data-platform` | [Azure DevOps / GitHub] | Terraform / Bicep IaC for all Azure resources |
| `adf-data-platform` | [Azure DevOps / GitHub] | ADF pipeline definitions (ARM export or adf_publish) |
| `databricks-data-platform` | [Azure DevOps / GitHub] | Databricks notebooks, job definitions, libraries |
| `docs-data-platform` | [Azure DevOps / GitHub] | Architecture documentation (this document) |

### 16.2 Infrastructure as Code

| Aspect | Detail |
|--------|--------|
| **Tool** | [Terraform / Bicep / Pulumi / ARM Templates] |
| **State Backend** | [Azure Storage Account with state locking] |
| **Module Structure** | Modules per service (e.g., `modules/adls`, `modules/databricks`, `modules/adf`) |
| **Environment Parameterization** | [tfvars per environment / Bicep parameter files] |

### 16.3 CI/CD Pipelines

| Pipeline | Trigger | Stages | Tool |
|----------|---------|--------|------|
| **Infra Deployment** | PR merge to `main` | Plan → Approve → Apply (DEV → TST → ACC → PRD) | [Azure DevOps / GitHub Actions] |
| **ADF Deployment** | Publish from `adf_publish` branch or ARM export | Validate → Deploy to DEV → TST → ACC → PRD | [Azure DevOps / GitHub Actions] |
| **Databricks Deployment** | PR merge to `main` | Lint → Test → Deploy jobs/notebooks (DEV → TST → ACC → PRD) | [Databricks Asset Bundles / Azure DevOps] |
| **Data Quality Tests** | Post-deployment or scheduled | Run Great Expectations / dbt tests | [Azure DevOps / Databricks Workflows] |

### 16.4 Branch Strategy

```
main (protected)          ← Production-ready code
├── release/x.y.z         ← Release candidate (optional)
├── develop                ← Integration branch
│   ├── feature/JIRA-123   ← Feature branches
│   ├── bugfix/JIRA-456    ← Bug fix branches
│   └── hotfix/JIRA-789    ← Hotfixes (may branch from main)
```

### 16.5 Deployment Artifact Versioning

| Artifact | Versioning Strategy |
|----------|---------------------|
| Infrastructure (Terraform) | Terraform plan output, state versioning |
| ADF Pipelines | ARM template version in Git commit SHA |
| Databricks Notebooks | Git commit SHA, Databricks Asset Bundle version |
| Libraries / Wheels | Semantic versioning (`1.2.3`) |

---

## 17. Cost Management

### 17.1 Cost Breakdown by Service (Monthly Estimate)

| Service | DEV | TST | ACC | PRD | Total |
|---------|-----|-----|-----|-----|-------|
| ADLS Gen2 | €[X] | €[X] | €[X] | €[X] | €[X] |
| Databricks (DBU) | €[X] | €[X] | €[X] | €[X] | €[X] |
| ADF (pipeline runs) | €[X] | €[X] | €[X] | €[X] | €[X] |
| Key Vault | €[X] | €[X] | €[X] | €[X] | €[X] |
| Networking (PE, VNet) | €[X] | €[X] | €[X] | €[X] | €[X] |
| Monitoring | €[X] | €[X] | €[X] | €[X] | €[X] |
| **Total** | **€[X]** | **€[X]** | **€[X]** | **€[X]** | **€[X]** |

### 17.2 Cost Optimization Measures

| Measure | Description | Estimated Savings |
|---------|-------------|-------------------|
| Spot instances (Databricks) | Use spot VMs for non-critical/retryable workloads | [e.g., 60-80% on compute] |
| Auto-terminate clusters | All clusters auto-terminate after idle period | [Variable] |
| Reserved capacity | Azure Reserved Instances for persistent compute | [e.g., 30-50%] |
| Lifecycle policies (ADLS) | Move cold data to Cool/Archive tiers | [e.g., 50-70% on storage] |
| Shut down non-PRD | Schedule non-PRD environments off outside business hours | [e.g., 60% on DEV/TST compute] |
| Photon acceleration | Use Photon runtime for eligible workloads | [e.g., 2x performance → fewer DBUs] |

### 17.3 Cost Alerts and Budgets

| Budget Name | Scope | Monthly Budget | Alert Thresholds |
|------------|-------|----------------|-----------------|
| `budget-data-platform-prd` | PRD subscription/RG | €[X] | 50%, 80%, 100%, 120% |
| `budget-data-platform-nonprod` | DEV+TST+ACC | €[X] | 50%, 80%, 100% |

---

## 18. Operational Runbook

### 18.1 Daily Operations Checklist

- [ ] Review ADF pipeline runs in Monitor (check for failures)
- [ ] Review Databricks job runs (check for failures, long-running jobs)
- [ ] Check Azure Monitor alerts for any triggered alerts
- [ ] Verify data freshness in Gold layer (spot-check key datasets)

### 18.2 Common Operational Procedures

#### 18.2.1 Re-run a Failed ADF Pipeline

1. Open ADF Studio → Monitor → Pipeline Runs
2. Identify the failed run, click to view error details
3. Determine root cause (source unavailable, timeout, data issue)
4. Fix the root cause
5. Click "Rerun" on the failed pipeline (or trigger a new run)
6. Monitor until completion

#### 18.2.2 Restart a Failed Databricks Job

1. Open Databricks Workspace → Workflows → Job Runs
2. Identify the failed run, click to view error/stack trace
3. Check cluster logs for underlying issues (OOM, driver failure, data issues)
4. Fix root cause (resize cluster, fix notebook code, fix data)
5. Click "Repair Run" (resumes from failed task) or "Run Now" (full re-run)

#### 18.2.3 Rotate a Secret in Key Vault

1. Generate the new credential in the source system
2. Go to Key Vault → Secrets → Select the secret
3. Click "New Version" and paste the new value
4. Test in DEV by triggering a dependent pipeline
5. Promote through TST → ACC → PRD
6. Disable the old secret version after validation
7. Document in change log

#### 18.2.4 Scale a Databricks Cluster

1. Navigate to Databricks Workspace → Compute → Select cluster
2. Edit cluster configuration
3. Adjust min/max workers, node type
4. Confirm the change (cluster will restart if running)
5. For job clusters: update the job definition or cluster policy

### 18.3 Incident Response

| Severity | Description | Response Time | Notification |
|----------|-------------|---------------|-------------|
| Sev 1 (Critical) | PRD pipeline failure blocking business-critical reports | < 30 minutes | PagerDuty + Phone + Email |
| Sev 2 (High) | PRD pipeline degraded performance or partial failure | < 2 hours | Email + Teams |
| Sev 3 (Medium) | Non-PRD failure or non-critical PRD issue | < 8 hours (business hours) | Teams |
| Sev 4 (Low) | Informational alert, optimization opportunity | Next business day | Email |

---

## 19. Naming Conventions

### 19.1 General Pattern

```
{resource-type}-{workload}-{environment}[-{region}][-{instance}]
```

### 19.2 Resource Naming Table

| Resource Type | Abbreviation | Example (PRD) | Example (DEV) |
|--------------|-------------|---------------|---------------|
| Resource Group | `rg` | `rg-data-prd` | `rg-data-dev` |
| Storage Account (ADLS) | `st` / `stadls` | `stadlsprd001` | `stadlsdev001` |
| Data Factory | `adf` | `adf-data-prd` | `adf-data-dev` |
| Databricks Workspace | `dbw` | `dbw-data-prd` | `dbw-data-dev` |
| Key Vault | `kv` | `kv-data-prd` | `kv-data-dev` |
| Virtual Network | `vnet` | `vnet-data-prd` | `vnet-data-dev` |
| Subnet | `snet` | `snet-dbw-private-prd` | `snet-dbw-private-dev` |
| NSG | `nsg` | `nsg-dbw-prd` | `nsg-dbw-dev` |
| Private Endpoint | `pe` | `pe-adls-dfs-prd` | `pe-adls-dfs-dev` |
| Log Analytics | `log` | `log-data-prd` | `log-data-dev` |
| Service Principal | `sp` | `sp-adf-prd` | `sp-adf-dev` |
| Azure AD Group | `sg` | `sg-data-engineers-prd` | `sg-data-engineers-dev` |

### 19.3 ADF Naming Conventions

| Object Type | Pattern | Example |
|-------------|---------|---------|
| Pipeline | `pl_{action}_{source}_{entity}` | `pl_ingest_sap_customers` |
| Dataset | `ds_{format}_{system}_{entity}` | `ds_parquet_adls_customers` |
| Linked Service | `ls_{service}_{env}` | `ls_adls_prd` |
| Trigger | `tr_{type}_{schedule}` | `tr_daily_0200utc` |
| Integration Runtime | `ir_{type}_{env}` | `ir_selfhosted_prd` |

### 19.4 Databricks Naming Conventions

| Object Type | Pattern | Example |
|-------------|---------|---------|
| Notebook | `{layer}_{domain}_{entity}` | `silver_sales_customers` |
| Job | `job_{layer}_{domain}_{schedule}` | `job_silver_sales_daily` |
| Cluster | `cluster-{purpose}-{env}` | `cluster-etl-prd` |
| Cluster Policy | `policy-{purpose}` | `policy-job-default` |
| Secret Scope | `kv-{env}` | `kv-prd` |
| Catalog (Unity) | `catalog_{env}` | `catalog_prd` |
| Schema (Unity) | `{layer}` | `bronze`, `silver`, `gold` |
| Table (Unity) | `{source_or_domain}_{entity}` | `sap_customers`, `dim_customer` |

### 19.5 ADLS Path Naming Conventions

| Layer | Pattern | Example |
|-------|---------|---------|
| Bronze | `bronze/{source_system}/{entity}/year=YYYY/month=MM/day=DD/` | `bronze/sap/customers/year=2024/month=01/day=15/` |
| Silver | `silver/{domain}/{entity}/` | `silver/sales/customers/` |
| Gold | `gold/{domain}/{dataset}/` | `gold/sales/revenue_by_region/` |

---

## 20. Tagging Strategy

### 20.1 Required Tags

| Tag Name | Description | Example Values |
|----------|-------------|----------------|
| `Environment` | Deployment environment | `dev`, `tst`, `acc`, `prd` |
| `Project` | Project or workload name | `data-platform` |
| `CostCenter` | Financial cost center | `CC-12345` |
| `Owner` | Responsible team or person | `data-engineering-team` |
| `ManagedBy` | How the resource is managed | `terraform`, `bicep`, `manual` |
| `Confidentiality` | Data classification | `public`, `internal`, `confidential`, `restricted` |

### 20.2 Optional Tags

| Tag Name | Description | Example Values |
|----------|-------------|----------------|
| `CreatedDate` | Resource creation date | `2024-01-15` |
| `ExpiryDate` | Expected decommission date (for temp resources) | `2024-06-30` |
| `SLA` | Service level agreement tier | `gold`, `silver`, `bronze` |
| `DR` | Disaster recovery tier | `tier1`, `tier2`, `none` |

### 20.3 Tag Enforcement

| Method | Scope | Action |
|--------|-------|--------|
| Azure Policy | Subscription | Require `Environment`, `Project`, `CostCenter`, `Owner` tags on all resources |
| Azure Policy | Subscription | Inherit tags from resource group if not specified |
| CI/CD Validation | Pipeline | Fail deployment if required tags are missing |

---

## 21. Decision Log

Record significant architectural decisions using the ADR (Architecture Decision Record) format.

### ADR-001: Use Medallion Architecture (Bronze/Silver/Gold)

| Field | Value |
|-------|-------|
| **Date** | YYYY-MM-DD |
| **Status** | Accepted |
| **Context** | Need a structured approach to organize data at different stages of processing |
| **Decision** | Adopt the Medallion (multi-hop) architecture with Bronze (raw), Silver (cleansed), Gold (curated) layers |
| **Rationale** | Industry-standard pattern; clear separation of concerns; enables incremental reprocessing; aligns with Delta Lake best practices |
| **Consequences** | Three distinct zones in ADLS; clear data contracts between layers; potential data duplication across layers |

### ADR-002: [Title]

| Field | Value |
|-------|-------|
| **Date** | YYYY-MM-DD |
| **Status** | [Proposed / Accepted / Deprecated / Superseded] |
| **Context** | [What is the issue or context?] |
| **Decision** | [What was decided?] |
| **Rationale** | [Why was this decision made?] |
| **Consequences** | [What are the implications?] |

> Add additional ADRs as architectural decisions are made. Keep this log up to date.

---

## 22. Glossary

| Term | Definition |
|------|-----------|
| **ADLS Gen2** | Azure Data Lake Storage Gen 2 — scalable storage with hierarchical namespace |
| **ADF** | Azure Data Factory — cloud ETL/ELT and orchestration service |
| **Bronze Layer** | Raw data zone containing exact copies of source data |
| **CMK** | Customer-Managed Key — encryption keys managed by the customer in Key Vault |
| **DAG** | Directed Acyclic Graph — represents pipeline dependencies |
| **DBR** | Databricks Runtime — the runtime version used on Databricks clusters |
| **DBU** | Databricks Unit — unit of processing capability for billing |
| **Delta Lake** | Open-source storage layer providing ACID transactions on data lakes |
| **Gold Layer** | Curated data zone with business-ready aggregations and KPIs |
| **HNS** | Hierarchical Namespace — ADLS feature enabling file system semantics |
| **IaC** | Infrastructure as Code — managing infrastructure through code/templates |
| **MSI** | Managed Service Identity — Azure-managed identity for service authentication |
| **PIM** | Privileged Identity Management — Azure AD feature for just-in-time access |
| **RBAC** | Role-Based Access Control — Azure authorization model |
| **Silver Layer** | Cleansed and conformed data zone |
| **SPN** | Service Principal Name — Azure AD application identity |
| **Unity Catalog** | Databricks governance layer for data, analytics, and AI assets |
| **VNet Injection** | Deploying Databricks clusters into a customer-managed virtual network |

---

## 23. Appendices

### Appendix A: Reference Architecture Diagrams

> Place detailed diagrams here or link to external diagram files.

| Diagram | Location | Description |
|---------|----------|-------------|
| High-Level Architecture | `docs/diagrams/high-level-architecture.png` | Overall platform architecture |
| Environment Topology | `docs/diagrams/environment-topology.png` | Four environments side by side |
| Network Topology | `docs/diagrams/network-topology.png` | VNets, subnets, peering, private endpoints |
| Data Flow | `docs/diagrams/data-flow.png` | Source → Bronze → Silver → Gold → Consumers |
| CI/CD Pipeline | `docs/diagrams/cicd-pipeline.png` | Deployment flow across environments |
| Security Architecture | `docs/diagrams/security-architecture.png` | Identity, network, encryption layers |

### Appendix B: Source System Inventory

| # | Source System | Type | Owner | Data Volume | Frequency | Protocol | Environment |
|---|-------------|------|-------|-------------|-----------|----------|-------------|
| 1 | [SAP ECC] | ERP | [Team] | [e.g., 5 GB/day] | Daily | [JDBC / RFC / OData] | On-prem |
| 2 | [Salesforce] | CRM | [Team] | [e.g., 500 MB/day] | Hourly | REST API | Cloud |
| 3 | [SQL Server] | Database | [Team] | [e.g., 2 GB/day] | Daily | JDBC (Self-hosted IR) | On-prem |
| 4 | [File Drop] | Files | [Team] | [e.g., 100 MB/day] | Event-driven | SFTP / Blob trigger | Cloud |

### Appendix C: Data Entity Catalog

| Domain | Entity | Source | Bronze Table | Silver Table | Gold Table | Refresh |
|--------|--------|--------|-------------|-------------|-----------|---------|
| Sales | Customers | SAP | `bronze.sap_customers` | `silver.sales_customers` | `gold.dim_customer` | Daily |
| Sales | Orders | SAP | `bronze.sap_orders` | `silver.sales_orders` | `gold.fact_order` | Daily |
| Marketing | Leads | Salesforce | `bronze.sfdc_leads` | `silver.marketing_leads` | `gold.dim_lead` | Hourly |

### Appendix D: Contact and Escalation Matrix

| Level | Contact | Method | Response Time |
|-------|---------|--------|---------------|
| L1 (First response) | [Data Engineering on-call] | Teams / PagerDuty | < 30 min (Sev 1) |
| L2 (Escalation) | [Platform Engineering Lead] | Phone / Teams | < 1 hour (Sev 1) |
| L3 (Vendor / Microsoft) | [Microsoft Support / TAM] | Azure Support Ticket | Per support plan SLA |
| Management | [Platform Owner / CTO] | Email / Phone | As needed |

### Appendix E: Change Log

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | YYYY-MM-DD | [Name] | Initial document creation |
| 1.1 | YYYY-MM-DD | [Name] | [Description of changes] |

---

> **Document maintained by:** [Team Name]
> **Review cadence:** [Quarterly / After major architecture changes]
> **Next scheduled review:** YYYY-MM-DD
