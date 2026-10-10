# Lesson 5 — Configure Connections and Authentication

## Overview

Azure Data Factory needs secure connections to the services involved in the data integration workflow.

In this project, Data Factory connects to:

1. Azure Data Lake Storage Gen2 (ADLS Gen2), which stores source and processed files.
2. Azure SQL Database, which stores watermark and error-log information.

Data Factory uses **linked services** to define these connections. Authentication and permissions determine whether the configured identity can perform the required operations.

In this lesson, we will configure or inspect the linked services and understand how to validate connectivity.

**Important:** The existing project already contains the linked services `LS_ADLS_Retail` and `LS_AzureSQL_Retail`. Inspect and document their actual settings before making changes.

## Learning Objectives

By the end of this lesson, you will be able to:

- Explain linked services and their role in Data Factory.
- Configure an ADLS Gen2 linked service.
- Configure an Azure SQL Database linked service.
- Understand managed identity and other supported authentication methods.
- Distinguish authentication from authorization.
- Validate connections and troubleshoot common errors.
- Apply least-privilege security principles.

## 1. Understand Linked Services

A linked service defines the connection information that Data Factory uses to access an external service.

Think of a linked service as a connection configuration rather than a data location.

For example:

| Component | Responsibility |
|---|---|
| Linked service | Defines the connection and authentication configuration |
| Dataset | Identifies the file, folder, table, or other data location |
| Pipeline | Coordinates activities that use the connection and dataset |
| Activity | Performs an operation such as copying data or reading metadata |

The project uses these linked services:

| Linked service | Target |
|---|---|
| `LS_ADLS_Retail` | Azure Data Lake Storage Gen2 |
| `LS_AzureSQL_Retail` | Azure SQL Database |

## 2. Understand Authentication and Authorization

These concepts are related but different.

### Authentication

Authentication answers:

**Who or what is connecting?**

Examples include:

- A managed identity.
- A Microsoft Entra identity.
- A service principal.
- A supported SQL authentication method.

### Authorization

Authorization answers:

**What is the authenticated identity allowed to do?**

Examples include:

- Read files from a storage container.
- Write files to a destination container.
- Read a SQL watermark table.
- Insert records into a SQL error-log table.

A successful authentication does not guarantee that the identity has permission to perform the requested operation.

## 3. Prepare the ADLS Gen2 Connection

The existing storage account is:

`stretaildata202610`

The ADLS Gen2 linked service is:

`LS_ADLS_Retail`

### Step 1 — Open Data Factory Studio

1. Open the Azure portal.
2. Open `adf-retail-data-dev-1`.
3. Select **Launch Studio**.
4. Open the **Manage** area.
5. Select **Linked services**.
6. Locate `LS_ADLS_Retail`.

### Step 2 — Inspect the configuration

Open the linked service and record:

- Connection type.
- Storage account endpoint.
- Authentication method.
- Selected identity or credential.
- Integration runtime.
- Connection-test result, if available.

For the existing project, the intended storage endpoint is associated with `stretaildata202610`.

Do not replace a working authentication method merely to match an example in this lesson.

## 4. Configure an ADLS Gen2 Linked Service

Perform these steps only when creating a new linked service or reproducing the project in a separate environment.

### Step 1 — Create the linked service

1. Open **Manage → Linked services**.
2. Select **New**.
3. Search for **Azure Data Lake Storage Gen2**.
4. Select the connector.
5. Select **Continue**.

### Step 2 — Configure the connection

Enter the required settings:

| Setting | Value or guidance |
|---|---|
| Name | `LS_ADLS_Retail` |
| Account selection | Select or enter the target storage account |
| Storage account | `stretaildata202610` for the existing project |
| Authentication | Use the approved method configured for your environment |
| Integration runtime | Select a compatible runtime |
| Description | Optional description of the connection |

The exact authentication options depend on the connector and environment.

### Step 3 — Select authentication

For a cloud-based Data Factory solution, a managed identity is often preferable because it avoids embedding storage credentials in pipeline definitions.

If using the Data Factory managed identity:

1. Select the supported managed-identity authentication option.
2. Confirm the correct Data Factory identity is selected or used.
3. Save the linked service configuration.
4. Grant the identity the required storage permissions.

For example, the identity may require a suitable storage data role to read and write files. If directory-level ACLs are used, those ACLs must also permit the required operations.

A management-plane role alone does not necessarily grant permission to read or write storage data.

### Step 4 — Test the connection

1. Select **Test connection**, if available.
2. Review the result.
3. If the test fails, inspect the endpoint, authentication, permissions, network configuration, and integration runtime.
4. Save the linked service when its configuration is correct.

### Expected result

The linked service should connect to the intended storage account using the configured identity and connectivity model.

## 5. Grant Storage Permissions

When using a managed identity, the identity must have permission to perform the required data operations.

### Step 1 — Identify the identity

In the Azure portal:

1. Open the Data Factory resource.
2. Open **Identity**.
3. Inspect the system-assigned managed identity status and principal information, if that identity is being used.
4. Confirm that the linked service uses the intended identity.

Do not assume that every linked service uses the Data Factory system-assigned identity. Verify the actual configuration.

### Step 2 — Assign the required role

1. Open the storage account `stretaildata202610`.
2. Open **Access control (IAM)**.
3. Select **Add → Add role assignment**.
4. Select an appropriate storage data role.
5. Assign it to the verified Data Factory identity.
6. Choose the narrowest practical scope.

For a copy operation, the source identity needs read access and the destination identity needs write access. If the same identity performs both operations, it needs both sets of permissions.

Where directory-level ACLs are used, verify the required directory traversal and file permissions as well.

### Step 3 — Validate access

Confirm that the linked service can connect and that an actual read or write operation succeeds.

A connection test alone may not prove that every required container and path is accessible.

## 6. Prepare the Azure SQL Connection

The existing SQL resources are:

| Property | Value |
|---|---|
| SQL server | `sql-retail-data-dev-2026` |
| Server endpoint | `sql-retail-data-dev-2026.database.windows.net` |
| Database | `sqldb-retail-dev` |
| Linked service | `LS_AzureSQL_Retail` |

The SQL linked service is used by activities that read or update the watermark and log errors.

## 7. Configure an Azure SQL Database Linked Service

### Step 1 — Open Linked Services

1. Open Data Factory Studio.
2. Navigate to **Manage → Linked services**.
3. Select **New**.
4. Search for **Azure SQL Database**.
5. Select the connector.
6. Select **Continue**.

### Step 2 — Enter the connection details

| Setting | Value |
|---|---|
| Name | `LS_AzureSQL_Retail` |
| Server | `sql-retail-data-dev-2026.database.windows.net` |
| Database | `sqldb-retail-dev` |
| Authentication | The supported authentication method configured for your environment |
| Integration runtime | A compatible runtime |
| Description | Optional |

### Step 3 — Choose authentication

Use an authentication method supported by the connector and permitted by your organization's security requirements.

Where supported and appropriately configured, Microsoft Entra authentication with a managed identity can reduce reliance on stored passwords.

If using a managed identity:

1. Select the appropriate managed-identity authentication option.
2. Confirm which identity the connector uses.
3. Ensure that identity has a database user and the required SQL permissions.
4. Ensure the network path permits connectivity to Azure SQL.

If the existing linked service uses another authentication method, document it accurately and avoid changing a working configuration without a reason.

### Step 4 — Test the connection

1. Select **Test connection**, if available.
2. Review the result.
3. Resolve authentication, database, network, or permission errors.
4. Save the configuration when correct.

## 8. Grant SQL Database Permissions

A managed identity must be recognized in the database and granted the required permissions.

The exact SQL setup depends on the authentication method and existing database configuration.

For a managed-identity setup, an authorized database administrator typically needs to:

1. Configure the appropriate Microsoft Entra administrator or supported database identity setup.
2. Create or verify a database user for the Data Factory identity.
3. Grant only the permissions required by the pipeline.

For example, the pipeline may need permission to:

- Read `dbo.ETL_Watermark`.
- Update the watermark value.
- Insert records into `dbo.ETL_ErrorLog`.

A database administrator can grant narrow permissions on the specific objects rather than granting broad database-owner privileges.

Illustrative SQL permissions, after the identity has been created as a database user, are:

```sql
GRANT SELECT, UPDATE
ON OBJECT::dbo.ETL_Watermark
TO [<data-factory-identity>];

GRANT INSERT
ON OBJECT::dbo.ETL_ErrorLog
TO [<data-factory-identity>];
```

Replace the placeholder with the actual database principal name. Confirm whether the pipeline requires additional permissions before applying these grants.

Do not execute these statements blindly against a working environment. First inspect the existing database users, permissions, and linked-service authentication method.

## 9. Understand Integration Runtime Selection

An integration runtime provides the infrastructure for Data Factory activities to connect to and move data.

Common types include:

- Azure integration runtime.
- Self-hosted integration runtime.
- Azure-SSIS integration runtime.

For this cloud-based project, use the integration runtime compatible with the source, destination, and network configuration.

A self-hosted runtime may be needed when a resource is accessible only through a private network or local infrastructure.

Check the existing linked services before changing runtime settings.

## 10. Validate Both Connections

Use this checklist to confirm readiness.

### ADLS Gen2

- [ ] `LS_ADLS_Retail` exists.
- [ ] It points to the correct storage account.
- [ ] Its authentication method is understood.
- [ ] The selected identity has the required storage permissions.
- [ ] The integration runtime is appropriate.
- [ ] The connection test succeeds, if supported.
- [ ] Required source and destination paths are accessible.

### Azure SQL Database

- [ ] `LS_AzureSQL_Retail` exists.
- [ ] The server endpoint is correct.
- [ ] The database name is correct.
- [ ] Its authentication method is understood.
- [ ] The selected identity has the required database permissions.
- [ ] The network path permits the connection.
- [ ] The connection test succeeds, if supported.
- [ ] Required watermark and error-log operations are authorized.

## 11. Troubleshooting

| Problem | Possible cause | What to check |
|---|---|---|
| ADLS connection fails | Incorrect endpoint or authentication configuration | Review the storage account and linked service |
| Access denied to a file | Missing storage data permissions or ACLs | Check RBAC, ACLs, and the identity used |
| SQL connection fails | Incorrect server, database, authentication, or network settings | Inspect the linked service and SQL connectivity |
| SQL login succeeds but query fails | Missing object-level permissions | Check the database user and grants |
| Test succeeds but copy fails | The activity accesses a different path or requires additional permissions | Inspect activity details and source/destination settings |
| Changes do not affect execution | Changes were not published or the wrong authoring context is active | Verify the published configuration |
| Private endpoint connectivity fails | Incorrect DNS, network, or runtime configuration | Review the approved network design |

## 12. Security Best Practices

- Prefer managed identity when supported and appropriate.
- Avoid hardcoded passwords and storage keys.
- Keep storage containers private.
- Grant only the permissions required by each operation.
- Use the narrowest practical resource scope.
- Review access when identities or responsibilities change.
- Avoid broad SQL permissions for pipeline identities.
- Do not include secrets in GitHub documentation.
- Keep development and production credentials separate.
- Review connection and access failures through approved monitoring processes.

## 13. Knowledge Check

### Question 1

What is a linked service?

A. A SQL table that stores source data.

B. A connection configuration for an external service.

C. A schedule trigger.

### Question 2

What is the difference between authentication and authorization?

A. Authentication identifies the connecting identity; authorization determines its permitted actions.

B. Authentication schedules the pipeline; authorization copies files.

C. They are identical.

### Question 3

Does a successful ADLS connection test guarantee that every file path is accessible?

A. Yes, every path is automatically authorized.

B. No, path-level permissions and activity configuration must also be verified.

C. Yes, provided the storage account exists.

### Question 4

Why should a pipeline identity not receive excessive SQL permissions?

A. To enforce least privilege and reduce the impact of unauthorized operations.

B. To make the pipeline run every minute.

C. To avoid creating datasets.

### Answers

1. B
2. A
3. B
4. A

## Summary

In this lesson, we examined how Azure Data Factory connects to ADLS Gen2 and Azure SQL Database.

You learned how to:

- Configure and inspect linked services.
- Understand authentication and authorization.
- Grant appropriate storage and database permissions.
- Select an appropriate integration runtime.
- Test connections and troubleshoot failures.
- Apply least-privilege security principles.

The next lesson is **Lesson 6 — Create and Configure Datasets**, where we will define the source files, destination paths, SQL control data, and reusable dataset parameters.

**Key principle:** A connection is ready only when the correct identity can authenticate, has the required permissions, and can reach the intended resource.
