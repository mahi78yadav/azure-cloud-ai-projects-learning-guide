# Module 00 — Azure and Git Fundamentals

## 1. Purpose of This Module

Before implementing Azure Data Factory, Azure Databricks, or Microsoft Foundry applications, we need to understand how cloud resources are organized, how developers maintain code, how access is controlled, and how cloud costs are managed.

These concepts apply to all three projects in this repository.

## 2. Learning Objectives

After completing this module, you should be able to:

- Explain the purpose of Git and GitHub.
- Understand Azure subscriptions, resource groups, resources, and regions.
- Explain authentication, authorization, identities, and role-based access control.
- Understand the relationship between source code, configuration, and deployment.
- Identify common cloud cost and security risks.
- Explain these concepts to a beginner.

---

## 3. Git and GitHub

### 3.1 What is Git?

Git is a version control system. It records changes to files over time and allows developers to review earlier versions, work on branches, and collaborate safely.

Example:

A developer creates a Python application on Monday and improves it on Tuesday. Git can record both versions and show exactly what changed.

### 3.2 What is GitHub?

GitHub is an online platform for storing and collaborating on Git repositories.

It provides:
- Remote repositories.
- Pull requests and code reviews.
- Branch management.
- Issue tracking.
- Documentation through Markdown files.
- Collaboration and project visibility.

Git is the version control tool. GitHub hosts repositories and provides collaboration features.

### 3.3 What is a repository?

A repository is a project location containing files and their version history.

In this learning program, we maintain four repositories:

1. Azure Data Factory implementation.
2. Azure Databricks implementation.
3. Enterprise Retail AI Assistant implementation.
4. A separate learning guide documenting all three projects.

### 3.4 What is a commit?

A commit records a set of changes with a message describing the work.

Example:

`Add Azure and Git fundamentals learning guide`

A useful commit message explains what changed.

### 3.5 What is a branch?

A branch provides an independent line of development.

For example, a developer can create a documentation branch, update a tutorial, and merge it into the main branch after reviewing the changes.

### 3.6 What is Markdown?

Markdown is a lightweight format for writing documentation.

Common syntax:

- `# Heading` — main heading.
- `## Section` — section heading.
- `- Item` — bullet list.
- `` `code` `` — inline code.
- Triple backticks — code block.
- `[Text](URL)` — hyperlink.

GitHub renders Markdown files as readable web pages.

### Practical exercise

Create a Markdown file containing:
- One main heading.
- Three section headings.
- A bullet list.
- One code example.
- One link to an existing project repository.

Commit the file and verify that GitHub renders it correctly.

---

## 4. Microsoft Azure Fundamentals

### 4.1 What is cloud computing?

Cloud computing provides computing resources such as storage, databases, networking, and application platforms through a cloud provider.

Instead of purchasing and maintaining every physical server, an organization can provision resources in Azure and manage them through supported tools.

### 4.2 What is an Azure subscription?

An Azure subscription is a management and billing boundary for Azure resource usage.

It helps organize resource access, quotas, and billing relationships.

Important: a subscription is not the same as a resource group. One subscription can contain multiple resource groups.

### 4.3 What is a resource group?

A resource group is a logical container for related Azure resources.

Example from the AI project:

`rg-ai-retail-dev`

A resource group helps organize resources for a workload or environment and provides a scope for access management.

Resources can sometimes depend on resources located in other resource groups. A resource group does not automatically isolate every network connection or grant access to every resource inside it.

### 4.4 What is an Azure resource?

An Azure resource is an individual service instance managed through Azure.

Examples include:
- Azure Data Factory.
- Azure Storage account.
- Azure SQL Database.
- Azure Databricks workspace.
- Microsoft Foundry resources and projects.
- Azure Functions applications.

Each resource has its own configuration, supported operations, permissions, and possible costs.

### 4.5 What is an Azure region?

A region is a geographic area where Azure provides supported services.

Examples include Central India and South Central India.

Region selection can affect:
- Service availability.
- Model and deployment availability.
- Latency.
- Data residency requirements.
- Pricing.
- Quota availability.

Not every service or AI model is available in every region. Confirm availability before choosing a region.

### 4.6 Resource naming conventions

Use consistent names to make resources easier to identify.

Examples:

- `rg-ai-retail-dev` — resource group.
- `stdatabricksretail2026` — storage account.
- `adf-retail-data-dev-1` — Data Factory.

These names are examples from the learning projects. Naming rules vary by service; some Azure resource names must be globally unique and may allow only specific characters.

---

## 5. Authentication and Authorization

These two concepts are related but different.

### 5.1 Authentication

Authentication answers:

**Who or what are you?**

Examples:
- Signing in to Azure with a user account.
- An application authenticating through Microsoft Entra ID.
- A managed identity representing an Azure-hosted workload.

### 5.2 Authorization

Authorization answers:

**What are you allowed to do?**

An authenticated identity might be allowed to read a storage container but not delete files or modify access permissions.

### 5.3 Microsoft Entra ID

Microsoft Entra ID provides identity and access capabilities used by Azure services and applications.

It supports users, groups, application identities, and managed identities.

### 5.4 Managed identity

A managed identity provides an Azure-managed identity for a supported resource.

For example, an Azure-hosted application can use its managed identity to request access to another Azure service without storing a password or long-lived client secret in its code.

A managed identity does not automatically have permission to access every resource. Appropriate roles and access policies must still be configured.

### 5.5 Role-based access control (RBAC)

RBAC assigns permissions through roles at defined scopes.

Common scopes include:
- Management group.
- Subscription.
- Resource group.
- Individual resource.

Example:

A Foundry project identity needs permission to retrieve data from an Azure AI Search knowledge base. Assigning an appropriate Search data-plane role at the search service scope can grant the required access.

Always select the narrowest scope and role that satisfy the requirement.

### 5.6 Control plane and data plane

The control plane manages the Azure resource itself.

Examples:
- Creating a storage account.
- Configuring resource settings.
- Assigning resource permissions.

The data plane accesses or operates on the contents of a resource.

Examples:
- Reading files from storage.
- Querying a search index.
- Reading or writing database records.

Permission to manage a resource does not necessarily imply permission to access its data, and the reverse is also true.

---

## 6. Configuration and Secrets

### 6.1 Source code

Source code contains the application's logic.

Examples:
- Python functions.
- Data transformation logic.
- API request handling.
- Agent instructions stored in application configuration.

### 6.2 Configuration

Configuration contains values that may vary between environments.

Examples:
- Resource endpoints.
- Agent names.
- Deployment names.
- Environment identifiers.

Use environment variables or supported configuration mechanisms rather than scattering environment-specific values throughout the code.

### 6.3 Secrets

Secrets include passwords, API keys, access tokens, and other sensitive credentials.

Never commit secrets to GitHub, even in a private repository.

For local development, use an appropriate local configuration mechanism and exclude secret files from Git. For deployed workloads, use supported identity-based authentication or an appropriate secret-management service.

If a secret is accidentally committed, removing it from the latest version alone does not remove it from the repository's history. Revoke or rotate the exposed secret.

---

## 7. Cloud Cost Management

Azure resources may incur charges even when they are not actively being used.

Before creating a resource, identify:
- The service pricing model.
- The selected pricing tier.
- Compute and storage charges.
- Model token or request charges.
- Minimum capacity or reservation requirements.
- Whether the resource can be stopped or deleted when not needed.

Important examples:
- A serverless database may still have storage charges.
- AI model usage may incur charges based on the model and deployment type.
- A search service or compute resource may have ongoing capacity charges.
- Deleting a resource may not delete every related resource or stored dataset.

Before starting each lab, check current Azure pricing and the available subscription quota. Set a budget or cost alert where supported, while remembering that an alert is not necessarily a hard spending limit.

### Lab cleanup checklist

- Identify all resources created for the lab.
- Confirm whether data must be retained.
- Stop resources when supported and appropriate.
- Delete resources that are no longer required.
- Check for related resources that remain.
- Review actual costs after the lab.

---

## 8. How These Concepts Apply to Our Three Projects

| Concept | Data Factory | Databricks | AI Assistant |
|---|---|---|---|
| Resource groups | Organize ingestion resources | Organize workspace resources | Organize Foundry and related resources |
| Storage | Raw and processed files | Bronze, Silver and Gold data | Retail knowledge documents |
| Identity and permissions | Access storage and SQL | Access storage and catalog objects | Access knowledge bases and models |
| Version control | Pipeline documentation and changes | Notebooks and project documentation | Python application and agent integration |
| Cost management | Pipeline and SQL usage | Compute and storage usage | Model, search, and application usage |

The specific permissions and resource configurations differ by service. Do not assume one role or identity configuration works for all three projects.

---

## 9. Knowledge Check

Answer these questions without looking at the definitions:

1. What is the difference between Git and GitHub?
2. What is the difference between an Azure subscription and a resource group?
3. What is the difference between authentication and authorization?
4. Why does a managed identity still need permissions?
5. What is the difference between control-plane and data-plane access?
6. Why should API keys never be committed to GitHub?
7. Why might an Azure resource continue to cost money when the application is not being used?
8. Why is a successful deployment not sufficient proof that the application works?

## 10. Completion Criteria

Mark this module **Verified** when you can:

- Explain all the concepts in your own words.
- Navigate your GitHub repositories and review commit history.
- Identify the resource group and region used by each existing project.
- Explain how the relevant identities access Azure resources.
- Locate the pricing and cleanup guidance for each lab.

Until these checks are completed, mark the module **Documented**.

## Next Module

Proceed to **Module 01 — Azure Data Factory Retail Data Integration**, starting with the business problem, source-to-target architecture, and the purpose of each Azure resource before examining pipeline implementation.

---

## Architecture Thinking and Communication

These resources build the mindset and communication skills required for an AI-Enabled Data Architect.

### 1. Enterprise Architect Thinking Framework

**File:** `enterprise-architect-thinking-framework.md`

Learn to approach architecture in this sequence:

Business understanding → Business data → Architecture → Governance → Resilience → Modernization → AI adoption.

### 2. AI-Enabled Data Architect Vocabulary

**File:** `architect-vocabulary-and-communication.md`

Learn important business, data, architecture, governance, resilience, cloud, and AI terminology, with practical examples and professional meeting sentences.

### Recommended Learning Method

1. Understand the concept in simple language.
2. Connect it to one of the three Azure projects.
3. Explain the architectural significance in your own words.
4. Practise presenting the concept to a manager or a BTech fresher.

**Guiding principle:** Explain the business purpose first, the technical decision second, and the expected value and trade-offs third.
