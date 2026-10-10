# Lesson 02 — Databricks Workspace and Storage Setup

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Explain the role of an Azure Databricks workspace.
- Understand how ADLS Gen2 supports a Lakehouse architecture.
- Identify the Access Connector used by the project.
- Explain managed identity-based access to cloud storage.
- Understand the relationship between a storage credential and an external location.
- Verify the project's storage configuration.
- Troubleshoot common storage access problems.

## 2. Architecture Overview

The project uses Azure Databricks to process retail data stored in Azure Data Lake Storage Gen2.

The high-level relationship is:

```text
Azure Databricks Workspace
          |
          | Uses governed storage access
          v
Unity Catalog Storage Credential
          |
          v
External Location
          |
          v
ADLS Gen2 Storage Account
          |
          +-- Source retail files
          +-- Lakehouse data
```

The Access Connector provides an Azure-managed identity that can be assigned permissions to access the storage account.

Unity Catalog uses the configured storage credential and external location to govern access to the external storage path.

These components serve different purposes:

| Component | Responsibility |
|---|---|
| Databricks workspace | Provides the environment for notebooks, compute, jobs, and data engineering |
| ADLS Gen2 | Stores source files and lake data |
| Access Connector | Provides an Azure resource identity for supported Databricks storage access |
| Managed identity | Authenticates the supported storage access request |
| Storage credential | Represents the identity configuration used by Unity Catalog |
| External location | Connects a governed storage path to a storage credential |

## 3. Project Resource Inventory

The following resources were created for this project.

| Resource | Name |
|---|---|
| Resource group | `rg-databricks-retail-dev` |
| Databricks workspace | `dbw-retail-lakehouse-dev` |
| ADLS Gen2 storage account | `stdatabricksretail2026` |
| Storage resource group | `adf-ecom-dev` |
| Access Connector | `ac-databricks-retail-dev` |
| Storage credential | `cred_adls_retail` |
| External location | `ext_retail_source` |

**Important:** The storage account is in a separate resource group from the Databricks workspace. This is a valid Azure configuration. The resources can work together when the correct identity, permissions, networking, and storage paths are configured.

The storage account name and resource names above are specific to this project. When reproducing the project in another subscription, use the names of the resources you create.

## 4. Understand the Databricks Workspace

An Azure Databricks workspace is the main environment where data engineers develop, execute, and manage data processing workloads.

It provides access to capabilities such as:

- Notebooks.
- Compute resources.
- Jobs and workflows.
- SQL queries and warehouses.
- Data exploration.
- Unity Catalog integration.
- Monitoring and execution history.

The project workspace is:

`dbw-retail-lakehouse-dev`

### Inspect the workspace

1. Sign in to the Azure portal.
2. Search for **Azure Databricks**.
3. Open the workspace `dbw-retail-lakehouse-dev`.
4. Confirm that its provisioning state is successful.
5. Select **Launch Workspace**.
6. Verify that you can access the Databricks environment.
7. Locate the notebooks and job associated with the retail Lakehouse project.

### What to understand

The workspace is not the same as the data lake.

The workspace provides the development and execution environment, while ADLS Gen2 stores the files and data.

A workspace may use multiple storage locations, and a storage account can be shared by multiple services when access is configured correctly.

## 5. Understand ADLS Gen2

Azure Data Lake Storage Gen2 is Azure storage with data lake capabilities, including hierarchical namespace support.

It is used to store data for the Lakehouse.

The project's storage account is:

`stdatabricksretail2026`

The storage account is in the resource group:

`adf-ecom-dev`

### Why use ADLS Gen2?

ADLS Gen2 supports:

- Storage of files in common data formats.
- Hierarchical directory structures.
- Large-scale analytical workloads.
- Integration with Azure data engineering services.
- Identity-based access control.
- Storage of Delta Lake data and transaction logs.

Databricks can read and write supported data formats in ADLS Gen2, provided that the appropriate access and configuration are in place.

### Inspect the storage account

1. Open the Azure portal.
2. Search for the storage account `stdatabricksretail2026`.
3. Open the resource.
4. Verify that the storage account exists and is available.
5. Inspect the **Configuration** section.
6. Confirm that hierarchical namespace is enabled if the account is intended to operate as ADLS Gen2.
7. Open **Containers**.
8. Inspect the containers and folders used by the project.

Do not change storage configuration settings simply to match an example. Some changes may affect existing workloads or require migration.

## 6. Understand the Source Data

The Lakehouse project uses four retail datasets:

- Customers.
- Products.
- Orders.
- Stores.

These files provide the input for Bronze ingestion and downstream transformations.

When inspecting the storage account, identify the actual source folder or container referenced by the notebooks.

Verify:

- The source files exist.
- File names match the notebook configuration.
- File formats match the configured readers.
- The Databricks identity has permission to read the files.
- The source path is consistent with the notebook code.

**Important:** Do not assume that a container or directory is used just because it exists. The source path configured in the notebook is the authoritative reference.

## 7. Understand the Access Connector

The project uses this Access Connector:

`ac-databricks-retail-dev`

An Azure Databricks Access Connector is an Azure resource that can provide a managed identity for supported Databricks access to Azure resources.

A managed identity allows an Azure service to authenticate without requiring a storage account key or password to be embedded in notebook code.

### Why use an Access Connector?

It supports identity-based access and helps avoid embedding long-lived storage secrets in application code.

The managed identity still needs appropriate authorization. Creating an Access Connector alone does not automatically grant permission to read or write storage data.

### Inspect the Access Connector

1. Open the Azure portal.
2. Search for `ac-databricks-retail-dev`.
3. Open the resource.
4. Inspect its identity settings.
5. Confirm which managed identity is enabled and used by the storage configuration.
6. Record the identity type and principal details for administrative troubleshooting, where permitted.

Do not copy sensitive credentials or tokens into the learning guide.

## 8. Understand Authentication and Authorization

Authentication and authorization are different concepts.

**Authentication** answers:

> Which identity is making the request?

**Authorization** answers:

> Is that identity allowed to perform this operation?

For this project:

1. The configured identity is used for supported storage access.
2. Azure validates the identity.
3. Azure authorization controls determine whether the identity can access the requested resource.
4. Unity Catalog permissions also govern access to catalog-managed data objects and external locations.

Access can fail even when authentication succeeds if the identity does not have the required permissions.

Similarly, a user may be able to query a Unity Catalog table but not have permission to administer the underlying storage credential.

## 9. Understand Azure Storage Permissions

The managed identity must have appropriate storage permissions.

Depending on the access design, this may involve Azure role-based access control (Azure RBAC) and, where applicable, hierarchical namespace access control lists (ACLs).

### Inspect role assignments

1. Open the storage account `stdatabricksretail2026`.
2. Open **Access control (IAM)**.
3. Select **Role assignments**.
4. Locate the managed identity associated with `ac-databricks-retail-dev`.
5. Inspect the assigned role and scope.
6. Confirm that the permissions are appropriate for the intended workload.

A role assignment may be scoped to the storage account, a container, or another supported scope. Check the actual assignment rather than assuming that access is granted at the account level.

### Least-privilege principle

Grant only the permissions required for the intended operations.

For example, a read-only workload should not automatically receive broad write permissions.

Do not assign additional roles merely to make an error disappear without understanding the underlying access requirement.

## 10. Understand Unity Catalog Storage Credentials

The project uses the storage credential:

`cred_adls_retail`

A Unity Catalog storage credential represents the authentication configuration used to access supported cloud storage.

In this architecture, it is associated with the identity configured for storage access.

A storage credential is not the same as a storage account key, a catalog, or an external location.

### Inspect the storage credential

1. Open the Databricks workspace.
2. Open the **Catalog** interface or the relevant Unity Catalog administration view.
3. Locate the storage credential `cred_adls_retail`.
4. Inspect its configuration and associated identity details.
5. Verify that the credential is available to the authorized administrators.
6. Do not recreate the credential if the existing configuration is working.

The exact interface may differ depending on the Databricks release and the permissions assigned to your account.

## 11. Understand External Locations

The project uses the external location:

`ext_retail_source`

An external location associates a cloud storage path with a Unity Catalog storage credential.

Conceptually:

```text
External Location
        |
        +-- Cloud storage URL
        |
        +-- Storage credential
                    |
                    +-- Configured identity
```

The external location provides a governed entry point for supported operations on the specified storage path.

### Inspect the external location

1. Open the Unity Catalog interface.
2. Locate `ext_retail_source`.
3. Inspect its URL.
4. Confirm that the URL points to the intended storage account and container or directory.
5. Confirm that it uses `cred_adls_retail`.
6. Review the permissions granted on the external location.
7. Verify that the configuration matches the notebook paths.

An external location does not automatically grant every user access to the underlying data. The relevant identity and Unity Catalog permissions must be configured correctly.

## 12. Verify Storage Access from Databricks

After verifying the Azure resources and Unity Catalog configuration, validate that the project can access the intended data.

Use the existing project notebooks or an approved test notebook.

### Validation steps

1. Open the Databricks workspace.
2. Open the source-ingestion notebook.
3. Identify the configured source path.
4. Confirm that the path corresponds to the intended storage location.
5. Execute the existing read operation against the source data.
6. Confirm that the expected records are returned.
7. Review errors if the operation fails.

For example, if the source is configured as a CSV path, the notebook might contain a read operation similar to:

```python
df = (
    spark.read
    .option("header", "true")
    .option("inferSchema", "true")
    .csv(source_path)
)

display(df)
```

This is a generic illustration. Use the actual variable and path configuration from the project's notebook rather than copying this example blindly.

For production pipelines, explicitly defined schemas are generally preferable to relying exclusively on schema inference.

### Expected result

The notebook should read the expected source data without storage access errors.

If the operation fails, investigate the storage path, identity, permissions, file format, and notebook configuration before changing the architecture.

## 13. Common Storage Access Problems

| Symptom | What to investigate |
|---|---|
| Storage path not found | Verify the account, container, directory, and file name |
| Permission denied | Check the managed identity, Azure RBAC, ACLs, and Unity Catalog grants |
| Storage credential error | Inspect the credential configuration and identity |
| External location error | Verify the storage URL, credential, and permissions |
| Files exist but cannot be read | Confirm the path and the permissions required for the operation |
| Notebook works for one user but not another | Inspect user permissions and the execution identity |
| Authentication succeeds but writes fail | Verify write permissions and the destination configuration |
| Catalog object cannot be accessed | Check catalog, schema, table, and external-location privileges |
| Unexpected schema | Inspect source headers, delimiters, types, and reader options |

## 14. Security Best Practices

Follow these practices when managing storage access:

- Prefer supported managed identity-based access over embedded account keys.
- Apply least-privilege permissions.
- Restrict who can create or modify storage credentials.
- Restrict who can create or modify external locations.
- Review Unity Catalog grants regularly.
- Avoid storing secrets in notebooks or GitHub.
- Validate source and target paths before running write operations.
- Keep development and production resources appropriately separated.

These are recommended practices. Document each security control as implemented only after verifying the actual environment.

## 15. Practical Validation Checklist

Before proceeding to the Unity Catalog and governance lesson, confirm the following:

- [ ] The Databricks workspace `dbw-retail-lakehouse-dev` is available.
- [ ] The ADLS Gen2 account `stdatabricksretail2026` exists.
- [ ] The intended source files are present.
- [ ] The Access Connector `ac-databricks-retail-dev` exists.
- [ ] The relevant managed identity is configured.
- [ ] The storage identity has the required permissions.
- [ ] The storage credential `cred_adls_retail` exists.
- [ ] The external location `ext_retail_source` exists.
- [ ] The external location references the intended storage path.
- [ ] The source-ingestion notebook can access the expected data.
- [ ] No secrets have been added to notebooks or source control.

## 16. Interview Questions and Answers

### Q1. What is an Azure Databricks workspace?

It is the environment used to develop, execute, and manage Databricks notebooks, compute, jobs, and supported data engineering workloads.

### Q2. What is ADLS Gen2?

Azure Data Lake Storage Gen2 is Azure storage with hierarchical namespace support and capabilities suited to analytical data workloads.

### Q3. Why use an Access Connector?

It provides an Azure resource identity for supported Databricks access to other Azure resources, reducing the need to embed long-lived credentials in code.

### Q4. What is the difference between authentication and authorization?

Authentication establishes the identity making a request. Authorization determines whether that identity has permission to perform the requested operation.

### Q5. What is a Unity Catalog storage credential?

It represents the authentication configuration used to access supported external storage.

### Q6. What is an external location?

It associates a cloud storage path with a Unity Catalog storage credential and provides a governed way to manage access to that location.

### Q7. Does creating an Access Connector automatically grant storage access?

No. The identity must be configured and authorized appropriately, and the relevant Unity Catalog settings and permissions must also be correct.

### Q8. Can the storage account be in a different resource group from the Databricks workspace?

Yes. Azure resources can communicate across resource groups when their configuration, permissions, networking, and access requirements are satisfied.

### Q9. Why should storage account keys not be embedded in notebooks?

Embedding keys increases the risk of exposing long-lived credentials through notebook output, source control, or shared code. Supported identity-based access is generally preferable.

## 17. Lesson Summary

In this lesson, you learned how the Databricks workspace, ADLS Gen2, Access Connector, storage credential, and external location work together.

The Databricks workspace provides the development and processing environment. ADLS Gen2 stores the data. The configured identity authenticates supported storage requests, while Azure permissions and Unity Catalog govern access.

The key principle is:

**Secure data access requires both a valid identity and the correct permissions on the intended storage path and data objects.**
