# azure_eventhub_etl

An ETL pipeline project that ingests data from Azure Event Hub and processes it in Databricks.

## 📋 Prerequisites

- [UV](https://docs.astral.sh/uv/getting-started/installation/) installed
- [Databricks CLI](https://docs.databricks.com/dev-tools/cli/databricks-cli.html) installed
- Azure Event Hub and Storage Account configured
- Access to Databricks Workspace

---

## ⚙️ Configuration Guide

### 1. `databricks.yml` Configuration

Update the following items according to your environment:

| Location | Parameter | Description | Example |
|----------|-----------|-------------|---------|
| `targets.dev.workspace.host` | `<workspace_url>` | Dev environment Databricks Workspace URL | `https://adb-1234567890.12.azuredatabricks.net` |
| `targets.prod.workspace.host` | `<workspace_url>` | Prod environment Databricks Workspace URL | `https://adb-0987654321.12.azuredatabricks.net` |
| `targets.prod.workspace.root_path` | `<your_email>` | Your email address | `john.doe@company.com` |
| `targets.prod.permissions.user_name` | `<your_email>` | Your email address | `john.doe@company.com` |

**Example:**
```yaml
targets:
  dev:
    mode: development
    default: true
    workspace:
      host: https://adb-1234567890.12.azuredatabricks.net  # ✅ Replace with actual URL

  prod:
    mode: production
    workspace:
      host: https://adb-0987654321.12.azuredatabricks.net  # ✅ Replace with actual URL
      root_path: /Workspace/Users/john.doe@company.com.bundle/${bundle.name}/${bundle.target}  # ✅ Replace with your email
    permissions:
      - user_name: john.doe@company.com  # ✅ Replace with your email
        level: CAN_MANAGE
```

---

### 2. `resources/azure_eventhub_etl_pipeline.pipeline.yml` Configuration

#### Azure Event Hub Settings

| Parameter | Description | Example |
|-----------|-------------|---------|
| `<your-scope>` | Databricks Secrets Scope name | `eventhub-secrets` |
| `<your-eventhub-access-key-name>` | Event Hub Access Key name | `RootManageSharedAccessKey` |
| `<your-eventhub-name>` | Event Hub name | `my-eventhub` |
| `<your-eventhub-namespace>` | Event Hub Namespace | `my-eventhub-namespace` |

#### Azure Storage Settings

| Parameter | Description | Example |
|-----------|-------------|---------|
| `<your-azure-container>` | Azure Storage Container name | `iot-data` |
| `<your-azure-storage-account>` | Azure Storage Account name | `mystorageaccount` |

#### Databricks Path Settings

| Parameter | Description | Example |
|-----------|-------------|---------|
| `<your_email>` (libraries.glob.include) | Your email address | `john.doe@company.com` |
| `<your_email>` (root_path) | Your email address | `john.doe@company.com` |
| `catalog` | Unity Catalog name (change if needed) | `bu1dev` |
| `schema` | Schema name (change if needed) | `bronze` |

**Example:**
```yaml
resources:
  pipelines:
    azure_eventhub_etl_pipeline:
      name: azure_eventhub_etl_pipeline
      configuration:
        "io.ingestion.eh.secretsScopeName": "eventhub-secrets"  # ✅ Secrets Scope name
        "iot.ingestion.eh.accessKeyName": "RootManageSharedAccessKey"  # ✅ Access Key name
        "iot.ingestion.eh.name": "my-eventhub"  # ✅ Event Hub name
        "iot.ingestion.eh.namespace": "my-eventhub-namespace"  # ✅ Namespace
        "iot.ingestion.kafka.requestTimeout": "60000"
        "iot.ingestion.kafka.sessionTimeout": "30000"
        "iot.ingestion.spark.failOnDataLoss": "false"
        "iot.ingestion.spark.maxOffsetsPerTrigger": "50000"
        "iot.ingestion.spark.startingOffsets": "latest"
        "storage": "abfss://iot-data@mystorageaccount.dfs.core.windows.net/iot/"  # ✅ Storage path
      libraries:
        - glob:
            include: /Users/john.doe@company.com/azure_eventhub_etl_pipeline/transformations/**  # ✅ Replace with your email
      catalog: bu1dev  # ✅ Change catalog if needed
      channel: CURRENT
      photon: true
      root_path: /Users/john.doe@company.com/azure_eventhub_etl_pipeline  # ✅ Replace with your email
      schema: bronze  # ✅ Change schema if needed
      serverless: true
```

---

## 🔐 Databricks Secrets Setup

You need to configure Databricks Secrets for Event Hub connection:

```bash
# Create Secrets Scope (if not exists)
databricks secrets create-scope <your-scope>

# Store Event Hub Access Key
databricks secrets put-secret <your-scope> <your-eventhub-access-key-name> --string-value "<your-access-key>"
```

---

## 🚀 Getting Started

### 1. Authenticate to Databricks
```bash
databricks configure
```

### 2. Deploy to Development Environment
```bash
databricks bundle deploy --target dev
```
> `dev` is the default target, so the `--target` parameter is optional.

### 3. Deploy to Production Environment
```bash
databricks bundle deploy --target prod
```

### 4. Run the Pipeline
```bash
databricks bundle run
```

---

## 📚 Additional Resources

- [Databricks Asset Bundles Documentation](https://docs.databricks.com/dev-tools/bundles/index.html)
- [Databricks VS Code Extension](https://docs.databricks.com/dev-tools/vscode-ext.html)
- [Azure Event Hub Documentation](https://docs.microsoft.com/azure/event-hubs/)
- [Databricks Deployment Modes](https://docs.databricks.com/dev-tools/bundles/deployment-modes.html)
