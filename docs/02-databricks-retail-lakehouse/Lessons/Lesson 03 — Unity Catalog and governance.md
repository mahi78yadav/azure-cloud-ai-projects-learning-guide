# Lesson 03 — Unity Catalog and Data Governance

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Explain the purpose of Unity Catalog.
- Understand the catalog, schema, and table hierarchy.
- Explain the difference between a storage credential and an external location.
- Understand managed and external tables.
- Explain how users, groups, and privileges control access.
- Understand how Unity Catalog governs access to data stored in ADLS Gen2.
- Troubleshoot common permissions and storage access errors.

## 2. Business Scenario

A retail company stores customer, product, order, and store data in an Azure Databricks Lakehouse.

Different users may need different levels of access:

- Data engineers need to create and transform tables.
- Analysts need to query business-ready Gold tables.
- Data administrators need to manage catalogs, schemas, and access permissions.
- Some users must not access sensitive or restricted data.

Without centralized governance, it can be difficult to control who can access data and who can modify it.

Unity Catalog provides a centralized governance framework for supported data assets in Databricks.

It helps organize data objects, control permissions, and manage access to external cloud storage.

## 3. Project Configuration

The project uses the following Unity Catalog objects.

| Component | Name |
|---|---|
| Catalog | `retail` |
| Bronze schema | `bronze` |
| Silver schema | `silver` |
| Gold schema | `gold` |
| Storage credential | `cred_adls_retail` |
| External location | `ext_retail_source` |
| Access Connector | `ac-databricks-retail-dev` |
| Storage account | `stdatabricksretail2026` |

These objects were created for the Retail Lakehouse project.

The storage account is located in the Azure resource group `adf-ecom-dev`, while the Databricks workspace is in `rg-databricks-retail-dev`. This separation is valid when the relevant identities, permissions, and access configuration are correct.

## 4. What Is Unity Catalog?

Unity Catalog is Databricks' centralized data governance solution.

It helps manage access to supported data objects and organize data assets across a Databricks environment.

Its capabilities include:

- Catalog and schema organization.
- Table and view permissions.
- Storage credential management.
- External location governance.
- Centralized privilege management.
- Data discovery and supported lineage capabilities.
- Governance of supported data assets across workspaces.

For this project, Unity Catalog organizes the retail data into Bronze, Silver, and Gold schemas.

## 5. Understand the Unity Catalog Hierarchy

The main hierarchy is:

```text
Metastore
    |
    +-- Catalog: retail
            |
            +-- Schema: bronze
            |       +-- Bronze tables
            |
            +-- Schema: silver
            |       +-- Silver tables
            |
            +-- Schema: gold
                    +-- Gold tables
```

### Metastore

A metastore is the top-level governance container that stores metadata and access-control information for Unity Catalog.

A Databricks workspace must be assigned to a Unity Catalog metastore to use Unity Catalog functionality.

The metastore is an administrative scope, not the same as a catalog or storage container.

### Catalog

A catalog is a top-level logical grouping of data objects.

This project uses:

`retail`

### Schema

A schema groups related tables, views, and other supported objects within a catalog.

This project uses:

- `retail.bronze`
- `retail.silver`
- `retail.gold`

### Table

A table contains structured data.

For example:

`retail.gold.gold_sales`

The fully qualified name follows this format:

`catalog.schema.table`

For the example above:

- Catalog: `retail`
- Schema: `gold`
- Table: `gold_sales`

## 6. Inspect Catalogs and Schemas

You can inspect the Unity Catalog objects using the Catalog Explorer interface or SQL.

### List catalogs

```sql
SHOW CATALOGS;
```

Confirm that `retail` is present.

### List schemas

```sql
SHOW SCHEMAS IN retail;
```

Confirm that the required schemas exist:

- `bronze`
- `silver`
- `gold`

### List Gold tables

```sql
SHOW TABLES IN retail.gold;
```

### Query the Gold sales table

```sql
SELECT *
FROM retail.gold.gold_sales
LIMIT 10;
```

These commands are useful for confirming that the catalog hierarchy exists and that the current identity can discover or query the relevant objects.

A permission error may mean that your identity lacks the required privilege; it does not necessarily mean the object is missing.

## 7. What Is a Storage Credential?

A storage credential represents the authentication configuration used by Unity Catalog to access supported external cloud storage.

The project uses:

`cred_adls_retail`

In this architecture, the credential is configured for the identity associated with the Databricks Access Connector.

The credential does not itself define a storage directory. It provides the identity configuration that can be associated with an external location.

### Storage credential responsibilities

- Represent the configured storage identity.
- Support governed access to external storage.
- Allow authorized administrators to manage storage authentication.
- Avoid requiring storage secrets to be embedded directly in notebook code when supported identity-based access is used.

A storage credential is not the same as a catalog, schema, external location, or storage account.

## 8. What Is an External Location?

An external location associates a cloud storage path with a Unity Catalog storage credential.

The project uses:

`ext_retail_source`

Conceptually:

```text
External Location: ext_retail_source
        |
        +-- Cloud storage URL
        |
        +-- Storage Credential: cred_adls_retail
                    |
                    +-- Configured identity
```

The external location provides a governed way to manage access to a specified cloud storage path.

### What should you verify?

1. Open Databricks Catalog Explorer or the relevant Unity Catalog administration view.
2. Locate `ext_retail_source`.
3. Inspect the storage URL.
4. Confirm that it points to the intended storage account and directory.
5. Verify that it uses `cred_adls_retail`.
6. Review the permissions assigned to the external location.

Do not recreate a working external location just to follow a tutorial. Inspect its configuration first.

## 9. Managed Tables and External Tables

Unity Catalog supports different table storage arrangements.

### Managed table

For a managed table, Databricks manages the table's storage lifecycle within the configured managed storage location.

Users generally work with the table through its registered name rather than manually managing its underlying files.

### External table

An external table references data stored at a specified external location. Databricks manages the table metadata, while the underlying data files remain in the external storage location.

Dropping an external table generally removes its table registration without automatically deleting the underlying data files.

The exact behavior depends on the object type and operation. Always verify the storage arrangement before deleting or changing a table.

### Comparison

| Aspect | Managed table | External table |
|---|---|---|
| Table metadata | Managed by Databricks | Managed by Databricks |
| Data location | Managed storage location | Specified external location |
| Data lifecycle | More directly managed by Databricks | Storage lifecycle managed separately |
| Common use | Tables owned by the Lakehouse platform | Data stored at a designated external path |

Do not assume that every table in the `retail` catalog is external simply because the project uses an external location. Inspect the actual table definition and location.

## 10. Understand Users, Groups, and Permissions

Data governance requires more than creating catalogs and schemas. Users must be granted the appropriate privileges.

A **user** represents an individual identity.

A **group** collects users who share access requirements.

A **privilege** grants a specific capability on a supported securable object.

Examples include:

- `USE CATALOG`
- `USE SCHEMA`
- `SELECT`
- `MODIFY`
- `CREATE TABLE`
- `CREATE SCHEMA`

The privileges available and their applicability depend on the object type and Databricks configuration.

### Example governance design

| Role | Example access requirement |
|---|---|
| Data engineer | Read source data and create or modify approved tables |
| Data analyst | Read approved Silver and Gold tables |
| Data administrator | Manage approved governance objects and permissions |
| Reporting consumer | Read approved business-ready datasets |

These are example roles for explaining the design. Do not claim that these groups have already been created in your environment unless you have verified them.

## 11. Understand Privilege Inheritance and Dependencies

A user may need multiple privileges to perform an operation.

For example, querying a table generally requires the appropriate usage privileges on its parent catalog and schema, along with `SELECT` on the table or a suitable inherited grant.

Conceptually, querying a Gold table requires:

```text
USE CATALOG on retail
        |
        v
USE SCHEMA on retail.gold
        |
        v
SELECT on retail.gold.gold_sales
```

A grant on the table alone may not be sufficient if the identity cannot use the parent catalog or schema.

Similarly, a user may be able to query an existing table without having permission to create new tables.

**Important:** The precise set of required privileges depends on the operation, object type, and existing grants. Inspect the effective permissions instead of applying broad permissions to solve every error.

## 12. Inspect Existing Grants

Before modifying permissions, inspect the current configuration.

### Inspect catalog grants

```sql
SHOW GRANTS ON CATALOG retail;
```

### Inspect schema grants

```sql
SHOW GRANTS ON SCHEMA retail.gold;
```

### Inspect table grants

```sql
SHOW GRANTS ON TABLE retail.gold.gold_sales;
```

The identity running these commands must have the required permission to inspect the relevant grants.

Use the results to understand which principals have access and which privileges have been granted.

## 13. Grant Read Access to an Analyst Group

The following is an example of a least-privilege pattern for a group named `retail_analysts`.

**Run this only if the group exists, you are authorized to grant privileges, and the intended access has been approved.**

```sql
GRANT USE CATALOG
ON CATALOG retail
TO `retail_analysts`;

GRANT USE SCHEMA
ON SCHEMA retail.gold
TO `retail_analysts`;

GRANT SELECT
ON TABLE retail.gold.gold_sales
TO `retail_analysts`;
```

This example provides usage privileges on the catalog and Gold schema, plus read access to the specified table.

It does not automatically grant access to every table in the catalog.

If analysts need to query additional Gold tables, grant the necessary privileges according to the approved access design.

### Validate the access

After the grants are applied, test with an identity that belongs to the intended group.

For example:

```sql
SELECT *
FROM retail.gold.gold_sales
LIMIT 10;
```

The test should succeed if the identity has all required privileges and the table is accessible.

Testing with an administrator account is not sufficient to prove that an analyst group has the intended access.

## 14. Grant Table Creation Access to a Data Engineering Group

A data engineering group may need to create tables in a designated schema.

The following is an example for a group named `retail_data_engineers`.

**Do not execute this example without verifying the group, approval, and intended scope.**

```sql
GRANT USE CATALOG
ON CATALOG retail
TO `retail_data_engineers`;

GRANT USE SCHEMA
ON SCHEMA retail.silver
TO `retail_data_engineers`;

GRANT CREATE TABLE
ON SCHEMA retail.silver
TO `retail_data_engineers`;
```

These grants illustrate how to provide the privileges needed to use a catalog, use a schema, and create tables in the Silver schema.

Creating or modifying data may require additional privileges depending on the operation and target object. Do not grant broad administrative privileges unless they are actually required.

## 15. Understand Azure Storage Permissions Versus Unity Catalog Permissions

Two different authorization layers are involved in this architecture.

### Azure storage authorization

Azure permissions govern whether the configured storage identity can perform operations on the ADLS Gen2 path.

Examples include the relevant Azure RBAC roles and, where applicable, directory or file ACLs.

### Unity Catalog authorization

Unity Catalog permissions govern which users and groups can use catalogs, schemas, tables, storage credentials, and external locations.

### Why both matter

A user may have permission to query a registered table but may not be allowed to administer the external location.

Likewise, a storage identity may have permission to read the underlying files, but a Databricks user may lack the Unity Catalog privileges needed to query a registered table.

Successful access requires the appropriate configuration across the relevant layers.

## 16. Troubleshooting Governance Problems

| Problem | What to investigate |
|---|---|
| Catalog is not visible | Check workspace metastore assignment and catalog privileges |
| Schema cannot be accessed | Check catalog and schema usage privileges |
| Table query returns permission denied | Check table privileges and parent object usage privileges |
| Storage credential error | Inspect the credential configuration and identity |
| External location access fails | Verify the URL, storage credential, Azure permissions, and Unity Catalog grants |
| Storage path is inaccessible | Check the path, identity, Azure RBAC, and applicable ACLs |
| User can query but cannot create a table | Check `CREATE TABLE` and required parent privileges |
| Administrator can query but analyst cannot | Test with the analyst identity and inspect its effective grants |
| External table points to unexpected data | Inspect the registered table location and external location configuration |

When troubleshooting, identify which authorization layer is producing the error before changing permissions.

## 17. Security Best Practices

- Use groups to manage access for users with similar responsibilities.
- Follow the principle of least privilege.
- Grant read access only to approved datasets.
- Restrict permission to create or modify governance objects.
- Limit who can manage storage credentials and external locations.
- Review effective grants periodically.
- Avoid embedding storage keys or secrets in notebooks.
- Test permissions using the intended user or group.
- Inspect the actual table location before deleting or changing an object.

These are recommended practices. Document individual controls as implemented only after verifying the current environment.

## 18. Practical Validation Checklist

Before marking this lesson complete, verify the following:

- [ ] The `retail` catalog exists.
- [ ] The `bronze`, `silver`, and `gold` schemas exist.
- [ ] The Gold sales table can be queried by an authorized identity.
- [ ] The storage credential `cred_adls_retail` exists.
- [ ] The external location `ext_retail_source` exists.
- [ ] The external location references the intended storage path.
- [ ] The relevant identity has appropriate Azure storage permissions.
- [ ] Existing catalog, schema, and table grants have been inspected.
- [ ] Any new grants are limited to the approved scope.
- [ ] Permissions have been tested using the intended identity.
- [ ] No secrets have been placed in notebooks or source control.

## 19. Interview Questions and Answers

### Q1. What is Unity Catalog?

Unity Catalog is Databricks' centralized governance solution for supported data assets, providing organization, access control, and related governance capabilities.

### Q2. What is the difference between a catalog and a schema?

A catalog groups schemas, while a schema groups tables and other supported data objects within a catalog.

### Q3. What is a storage credential?

A storage credential represents the authentication configuration used to access supported external cloud storage.

### Q4. What is an external location?

An external location associates a cloud storage path with a storage credential and provides a governed access point for that path.

### Q5. What is the difference between a managed table and an external table?

Databricks manages the storage lifecycle of managed tables within their configured managed storage. External tables reference data stored at an external location, with the data files managed separately from the table registration.

### Q6. Why might a user need both `USE CATALOG` and `USE SCHEMA`?

These privileges allow the user to use the parent catalog and schema. Access to a table generally also requires the relevant table privilege, such as `SELECT`.

### Q7. What is the difference between Azure RBAC and Unity Catalog permissions?

Azure RBAC governs access to Azure resources. Unity Catalog governs access to supported Databricks data objects and storage-related securable objects. Both layers may be relevant to a storage operation.

### Q8. Why should groups be used for access management?

Groups simplify administration by allowing privileges to be granted to a role or team instead of managing each user's grants separately.

### Q9. Does creating an external location grant every user access to the data?

No. Access depends on the applicable Unity Catalog privileges, the configured storage identity, Azure storage permissions, and the operation being performed.

### Q10. How would you troubleshoot a table permission error?

Check the table privilege, parent catalog and schema privileges, the user's group membership, and whether the table is accessible under the intended identity.

## 20. Lesson Summary

In this lesson, you learned how Unity Catalog organizes the retail Lakehouse and governs access to data.

You covered catalogs, schemas, tables, storage credentials, external locations, managed and external tables, and permissions for users and groups.

You also learned why both Unity Catalog permissions and Azure storage authorization matter.

The key principle is:

**Grant the minimum required access at the correct governance layer, and verify permissions using the identity that needs the access.**
