# Module 03 — Microsoft Foundry Enterprise Retail AI Assistant

## 1. Purpose of This Module

This module explains how to design and build an enterprise-style Generative AI application using Microsoft Azure.

The objective is to understand the purpose of each component, how the components communicate, how responses are grounded in business documents, and how an AI agent can eventually use external tools.

This guide is intended for:
- Azure and data engineering professionals learning AI engineering.
- Developers preparing for Microsoft Azure AI certifications.
- BTech students learning cloud-based AI application development.
- Professionals who want to explain an AI solution in interviews and management discussions.

## 2. Business Scenario

A retail company maintains product catalogues, return policies, warranty conditions, and customer support documentation.

Employees and customers need to ask questions in natural language, such as:

- What is the return policy for laptops?
- How long is the laptop warranty?
- Which laptops are suitable for students under ₹50,000?
- What customer information is required when contacting support?

A traditional application may require users to search multiple documents manually. A Generative AI assistant can retrieve relevant information and present a concise answer with supporting sources.

The application must not invent company policies, product specifications, or prices.

## 3. Target Architecture

```text
Retail Documents
      |
      v
Microsoft Foundry IQ
Knowledge Base
      |
      v
Azure AI Search
Retrieval and Grounding
      |
      v
Microsoft Foundry Model
      |
      v
Foundry AI Agent
      |
      v
Python Chat Application
      |
      v
User
```

Future enhancement:

```text
AI Agent
    |
    v
Product Lookup Tool
    |
    v
Azure Function API
    |
    v
Structured Product Data
```

The knowledge-base flow is the foundation of the solution. The product lookup API is a separate extension that demonstrates tool calling.

## 4. Technical Concepts

### 4.1 Generative AI

Generative AI produces new content, such as text, based on the input and context it receives.

In this project, the model interprets the user's question, uses retrieved retail information, and generates a readable answer.

**Why it is used:** It allows users to ask questions naturally instead of learning a fixed set of search commands.

### 4.2 Large Language Model (LLM)

An LLM is a model trained to understand and generate language.

The project uses the deployed `gpt-4.1-mini` model.

The model generates answers but should not be treated as the authoritative source for company-specific policies or product information.

**Why it is used:** It converts a user's question and retrieved context into a useful response.

### 4.3 Microsoft Foundry

Microsoft Foundry is the Azure platform used to build and manage AI applications and agents.

The project uses Foundry to configure the model, create the knowledge base connection, define agent instructions, and test responses.

**Why it is used:** It provides a managed environment for developing AI applications without building every AI infrastructure component from scratch.

### 4.4 Azure AI Search and Foundry IQ

Azure AI Search provides search capabilities for finding relevant information in indexed content. Foundry IQ provides knowledge capabilities that agents can use to retrieve information from connected knowledge sources.

In this project, four retail documents are used:

- `Customer_Support_Policy.docx`
- `Retail_Product_Catalog.docx`
- `Laptop_Warranty_Policy.docx`
- `Laptop_Return_Policy.docx`

The configured knowledge source is `ks-file-339`, and the knowledge base is `retail-knowledge-base`.

**Why they are used:** The assistant needs access to company-approved information rather than relying only on the model's general training.

### 4.5 Retrieval-Augmented Generation (RAG)

RAG combines information retrieval with language generation.

The general process is:

1. Receive the user's question.
2. Retrieve relevant information from the knowledge source.
3. Provide the retrieved information to the model.
4. Generate an answer grounded in that information.
5. Present source references when available.

**Why it is used:** Company policies change over time. RAG lets the application retrieve current approved documents without retraining the language model every time a document changes.

### 4.6 Embeddings and Vector Search

An embedding represents text as a numerical vector that captures aspects of its meaning.

The project uses the `text-embedding-3-small` embedding model.

Vector search can find semantically related content even when the user's wording differs from the wording in the document.

For example, a question about returning a defective laptop may match a policy section discussing manufacturing defects.

**Why it is used:** Semantic retrieval can find relevant information beyond exact keyword matches.

### 4.7 AI Agent

An AI agent combines a model with instructions and connected capabilities to work toward a user's request.

The configured agent is `retail-ai-assistant-mahi2026`.

Its instructions tell it to:
- Use the retail knowledge base as the primary source.
- Avoid inventing product and policy details.
- State when the requested information is unavailable.
- Explain product recommendations using retrieved information.
- Identify source documents where possible.

**Why it is used:** The agent provides a reusable, configured interface for answering retail questions.

### 4.8 Tool Calling and Function Calling

Tool calling allows an AI application to request that an external function perform a defined task.

For example, a product lookup tool could accept a maximum price and intended purpose, then return matching products.

The tool performs the lookup; the model explains the returned results to the user.

**Why it is used:** A model should not be expected to calculate live inventory or retrieve authoritative structured records from memory.

**Implementation status:** The knowledge-base agent is implemented. The separate Azure Function product lookup integration is a planned enhancement and must not be described as completed until it is connected and tested.

### 4.9 Python and Streamlit

Python provides the application logic and integrates with the Azure SDKs.

Streamlit provides a simple browser-based chat interface with conversation history, user input, and displayed responses.

The Python client is documented under the repository's `app/` folder.

**Why they are used:** Python connects the application to Foundry, while Streamlit provides an accessible user interface for demonstrations.

### 4.10 Microsoft Entra ID and Managed Identity

Microsoft Entra ID provides identity and authentication services. Managed identities allow supported Azure resources to authenticate without embedding passwords or API keys in source code.

The local Python application uses `DefaultAzureCredential`, which can use a supported developer sign-in such as Azure CLI authentication.

The Azure AI Search service was granted the **Search Index Data Reader** role to the Foundry project's managed identity to resolve a knowledge-base access error.

**Why they are used:** Identity-based access reduces the need to store long-lived secrets in application code.

### 4.11 Evaluation and Responsible AI

AI responses must be evaluated for correctness and reliability.

Important checks include:
- Groundedness: Is the answer supported by retrieved information?
- Relevance: Does the answer address the question?
- Retrieval quality: Were the correct document passages found?
- Unsupported claims: Did the model invent facts?
- Security: Could untrusted document content manipulate the model?
- Access control: Can the application access only authorized resources?

**Why they are used:** A successful response in a playground is not sufficient evidence that an AI application is production-ready.

## 5. Azure Resources and Configuration

| Component | Configured value | Purpose |
|---|---|---|
| Resource group | `rg-ai-retail-dev` | Groups project resources |
| Foundry project | `gmahi-78-3687` | Manages the AI development project |
| Chat model | `gpt-4.1-mini` | Generates answers |
| Embedding model | `text-embedding-3-small` | Supports semantic retrieval |
| Search resource | `gmahi-78-3687-srch-1c32` | Supports knowledge retrieval |
| Knowledge source | `ks-file-339` | Connects the four uploaded documents |
| Knowledge base | `retail-knowledge-base` | Exposes retail knowledge to the agent |
| AI agent | `retail-ai-assistant-mahi2026` | Applies instructions and answers questions |

These names document the current learning environment. They are not intended to be hard-coded as universal values for other deployments.

## 6. Implementation Stages

### Stage 1 — GitHub Repository and Documentation

**Purpose:** Establish version control and a reproducible learning guide.

**Technical aspects:** Git, GitHub, Markdown, repository structure, commits, and documentation.

**Validation:** The project guide is committed and can be read from GitHub.

### Stage 2 — Azure Foundation

**Purpose:** Establish the Azure boundary for the project.

**Technical aspects:** Subscription, resource group, region, resource naming, access control, and cost awareness.

**Validation:** The project resource group exists and the selected resources are associated with it.

### Stage 3 — Foundry Project and Model

**Purpose:** Configure the model used to generate answers.

**Technical aspects:** Foundry project, model deployment, model endpoint, deployment availability, and quota.

**Validation:** The selected model deployment succeeds and can be used by the project.

### Stage 4 — Knowledge Source and RAG

**Purpose:** Make company documents searchable and available to the AI assistant.

**Technical aspects:** Document ingestion, indexing, embeddings, semantic retrieval, permissions, and grounded generation.

**Validation:** The knowledge source is active, and the agent can retrieve information from the documents.

### Stage 5 — AI Agent

**Purpose:** Define how the assistant uses the model and its knowledge.

**Technical aspects:** Agent instructions, model selection, knowledge connections, responses, and source references.

**Validation:** Questions about returns, warranties, customer support, and the product catalogue receive relevant answers from the knowledge base.

### Stage 6 — Python Chat Application

**Purpose:** Provide a repeatable application interface outside the Foundry playground.

**Technical aspects:** Python, Streamlit, Azure SDKs, authentication, dependency management, and conversation history.

**Validation:** The Python application runs and receives a response from the configured Foundry agent.

### Stage 7 — Product Lookup Tool

**Purpose:** Demonstrate an agent using a separate structured-data lookup capability.

**Technical aspects:** Python functions, HTTP APIs, Azure Functions, OpenAPI schemas, input validation, tool invocation, and returned results.

**Validation:** The tool returns correct product data for a known request, and the agent uses the result in its answer.

**Status:** Planned enhancement. Do not mark this stage complete until the external tool is connected and tested.

### Stage 8 — Evaluation, Security, and Monitoring

**Purpose:** Assess reliability and prepare for a more production-oriented implementation.

**Technical aspects:** Groundedness, relevance, error handling, identity permissions, secrets management, logging, monitoring, and cost control.

**Validation:** Record test cases and results, verify access permissions, and document known limitations.

### Stage 9 — Final Documentation and Interview Preparation

**Purpose:** Make the project reproducible and explainable.

**Technical aspects:** Architecture documentation, setup instructions, environment configuration, test evidence, troubleshooting, and design trade-offs.

**Validation:** Another learner can follow the guide, reproduce the supported stages, and explain why each component is used.

## 7. RAG Test Scenarios Already Executed

The following scenarios were successfully tested against the configured Foundry knowledge base:

| Scenario | What it validates |
|---|---|
| Laptop return policy | Policy retrieval and grounded summarization |
| Laptop warranty period | Retrieval of warranty terms |
| Customer support contact options | Retrieval of support procedures |
| Available laptops in the catalogue | Retrieval of product information |

The knowledge-base access issue was resolved by assigning the Foundry project's managed identity the Search Index Data Reader role on the Azure AI Search resource.

These scenarios are completed baseline tests. They should not be repeated unless a configuration change causes a regression.

## 8. Cost and Security Considerations

- Model inference, embedding generation, search resources, and application hosting may incur charges.
- Check pricing and quota before deploying additional Azure resources.
- Stop or remove resources that are no longer required for the learning exercise.
- Do not commit API keys, passwords, access tokens, or `.env` files to GitHub.
- Use Microsoft Entra ID and managed identities where supported.
- Give identities only the permissions required for their tasks.
- Treat retrieved documents as data, not as instructions that can override system or agent instructions.
- Do not claim production readiness solely because the demonstration works.

## 9. Expected Learning Outcomes

After completing this module, a learner should be able to:

1. Explain the difference between a language model, RAG application, and AI agent.
2. Describe how embeddings and search help retrieve relevant enterprise information.
3. Explain how a Foundry knowledge base grounds an answer in company documents.
4. Build and run a Python client for a configured Foundry agent.
5. Describe how a custom tool can connect an agent to structured product data.
6. Explain identity-based access, evaluation, monitoring, and cost considerations.
7. Reproduce the project by following the documented implementation stages.

## 10. Microsoft Certification Alignment

This project provides practical learning relevant to Azure AI application and agent development, including Generative AI, retrieval, agent configuration, tool integration, security, and evaluation.

Use the current official exam guide to confirm the latest certification scope:

- [Microsoft AI-103 exam page](https://learn.microsoft.com/en-us/credentials/certifications/exams/ai-103/)
- [Microsoft AI-103 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/AI-103)

A portfolio project supports preparation but does not, by itself, establish certification or complete coverage of every exam objective.

## 11. Project Status

**Completed:** Foundry project setup, model deployment, knowledge source, knowledge base, agent configuration, identity permission correction, four baseline RAG scenarios, and a basic Python client that successfully invokes the configured agent.

**In progress or requiring separate validation:** Product lookup tool integration, comprehensive evaluation, production security review, monitoring, and final reproducibility testing.

The status of each component should be updated only after its implementation and validation evidence are available.
