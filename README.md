# Azure Cloud and AI Projects — Complete Learning Guide

A step-by-step, hands-on learning guide covering three enterprise-style projects using Microsoft Azure. This repository explains not only how to implement each project, but also why each technology is used and how the components work together.

## Learning Objectives

By completing these tutorials, learners will be able to:

- Explain the architecture and purpose of each major component.
- Build Azure data engineering and AI solutions step by step.
- Understand technical concepts before implementing them.
- Validate results and troubleshoot common errors.
- Reproduce the projects from documented instructions.
- Explain the solutions in technical interviews and teach the fundamentals to BTech students.

## Projects Covered

### Project 1 — Azure Data Factory Retail Data Integration

**Technologies:** Azure Data Factory, ADLS Gen2, Azure SQL Database, incremental loading, watermark management, error logging, triggers and monitoring.

**Business objective:** Build a repeatable data ingestion pipeline that processes files, tracks incremental changes, and records pipeline failures.

Existing implementation: [Azure ADF Retail Data Integration](https://github.com/mahi78yadav/azure-adf-retail-data-integration)

### Project 2 — Azure Databricks Retail Lakehouse

**Technologies:** Azure Databricks, PySpark, Delta Lake, Unity Catalog, ADLS Gen2, Auto Loader, data quality and scheduled jobs.

**Business objective:** Transform retail source files through Bronze, Silver and Gold layers to produce trusted business data for analytics.

Existing implementation: [Azure Databricks Retail Lakehouse](https://github.com/mahi78yadav/azure-databricks-retail-lakehouse)

### Project 3 — Enterprise Retail AI Assistant

**Technologies:** Microsoft Foundry, language models, embeddings, Azure AI Search, Retrieval-Augmented Generation (RAG), Python, AI agents and tool integration.

**Business objective:** Answer retail questions using company documents and, as the project evolves, retrieve structured product information through connected tools.

Existing implementation: [Enterprise Retail AI Assistant](https://github.com/mahi78yadav/azure-enterprise-retail-ai-assistant)

## How Each Tutorial Is Structured

Every stage will cover:

1. Objective and business purpose
2. Technical concepts and terminology
3. Architecture and component interactions
4. Prerequisites
5. Detailed implementation instructions
6. Expected results and screenshots
7. Validation and testing
8. Troubleshooting and fixes
9. Security, permissions and cost considerations
10. Knowledge checks and teaching exercises

## Learning Roadmap

| Module | Subject |
|---|---|
| 00 | Azure, Git and project foundations |
| 01 | Azure Data Factory data integration |
| 02 | Azure Databricks lakehouse |
| 03 | Microsoft Foundry and the Retail AI Assistant |
| 04 | Troubleshooting and lessons learned |
| 05 | Teaching material, exercises and interview preparation |

## Implementation Status

Each stage will be marked:

- **Verified:** Implemented and tested.
- **Documented:** Instructions are written; independent reproduction still needs validation.
- **Pending:** Implementation or validation is incomplete.

A completed existing project will not automatically be treated as a fully reproducible tutorial. Documentation and clean-rebuild validation will be tracked separately.

## Prerequisites

- Basic SQL and Python knowledge
- A GitHub account
- Access to an Azure subscription and the required services
- Familiarity with command-line tools
- Awareness of cloud resource costs and security

Some Azure AI models and services require an eligible subscription, available quota, and billable usage. Check current availability and pricing before deployment.

## Security Guidelines

- Never commit API keys, passwords, tokens or confidential company data.
- Prefer identity-based authentication and least-privilege permissions.
- Use safe sample datasets.
- Follow resource cleanup instructions after each lab.

## Intended Audience

Azure data engineers, aspiring AI engineers, cloud developers, interview candidates and BTech students learning enterprise application development.

## Guiding Principle

**Understand the purpose first. Implement second. Validate third. Teach it clearly.**
