# ParadigmSec Product Capabilities and Architecture

## Overview

ParadigmSec is an AI-assisted governance, compliance, and remediation platform for enterprise infrastructure. It is designed to ingest a tenant profile and infrastructure footprint, normalize that information into a structured topology, compare it against a compliance framework, identify security and governance gaps, and generate actionable remediation artifacts.

The system combines:

- a FastAPI backend
- a React frontend
- a LangGraph-based agent workflow
- a Qdrant vector database for control retrieval
- local and cloud LLM routing for analysis and synthesis

This project is best understood as a hybrid AI governance assistant for infrastructure posture review and compliance automation.

---

## Product Vision

The product aims to solve a common enterprise challenge: security and compliance teams often operate with fragmented infrastructure data, inconsistent evidence, and slow manual review cycles. ParadigmSec attempts to automate the path from raw environment information to:

- structured topology understanding
- compliance gap identification
- human-readable assessments
- remediation recommendations
- Terraform-based infrastructure fixes

This transforms the review process from a manual audit into an AI-assisted operational workflow.

---

## Core Product Capabilities

### 1. Multi-environment onboarding

The platform accepts a tenant profile and a list of infrastructure environments for scanning. It is designed to work across:

- AWS
- GCP
- Azure
- on-prem infrastructure via SSH-based discovery

This supports multi-cloud, hybrid, and enterprise-scale onboarding scenarios rather than a single-provider workflow.

### 2. Dynamic provider configuration

The backend exposes provider-specific schema definitions so the frontend can dynamically render forms based on the selected provider. This reduces custom form maintenance and makes it easier to extend provider coverage.

### 3. Sequential audit orchestration

The system processes multiple environments one after another, aggregates the results, and passes the cumulative telemetry into a single AI reasoning flow. This gives the product the concept of a macro enterprise audit rather than a single isolated check.

### 4. Topology extraction and normalization

Raw infrastructure descriptions are converted into a structured model containing:

- security zones
- trust boundaries
- network nodes
- traffic pathways
- exposure assumptions
- segmentation relationships

This normalized view allows the rest of the system to reason consistently over infrastructure data.

### 5. Compliance gap detection

The analyzer compares the normalized topology with relevant compliance controls retrieved from a framework-aligned vector store. This lets the platform:

- find mismatches between architecture and policy
- classify the severity of issues
- identify specific control violations
- produce evidence-backed reasoning

### 6. Remediation synthesis

The final stage of the workflow produces:

- a Markdown report for governance review
- remediation guidance in structured text
- Terraform code for infrastructure-level fixes

This bridges the gap between assessment and execution.

### 7. Local system audit and artifact export

The system also includes a local audit mode that inspects current machine state such as:

- Docker running containers
- active listening ports
- local environment topology

It writes generated outputs to an outputs directory for later review or automation handoff.

---

## Product Components and Architecture

### 1. Frontend layer

The frontend is implemented in React and lives under the frontend directory.

Responsibilities:

- collect tenant metadata
- choose architecture type and compliance target
- select environment providers
- render dynamic credential forms
- trigger the enterprise audit flow
- display results and generated artifacts

The main frontend logic is in:

- frontend/src/App.jsx

This frontend behaves like a guided wizard rather than a full operational dashboard, which is appropriate for a prototype-stage product.

### 2. API layer

The backend is implemented using FastAPI and is defined in:

- app/main.py

This layer exposes routes for:

- provider schema retrieval
- audit initiation
- orchestration of the multi-environment pipeline

It also includes middleware configuration, request validation models, and the primary enterprise audit orchestration logic.

### 3. Ingestion and discovery layer

The ingestion layer is defined in:

- app/core/ingestion.py

Core design principles:

- provider-specific strategy classes
- a central registry for strategy selection
- schema validation for each provider
- normalized telemetry output for the workflow

Supported patterns include:

- AWS strategy
- GCP strategy
- Azure strategy
- on-prem SSH strategy

This design is extensible and keeps provider-specific logic separate from the core audit flow.

### 4. AI and reasoning layer

The AI layer is composed of:

- local LLM access through Ollama
- strategic cloud reasoning through Anthropic-compatible models
- dynamic fallback logic when cloud credentials are missing

The model factory is defined in:

- app/core/llm.py

The idea is to split workloads into:

- tactical tasks handled locally and efficiently
- reasoning-heavy synthesis handled by a stronger cloud model when available

### 5. Graph-based workflow layer

The orchestration engine is implemented with LangGraph in:

- app/agents/graphs/review_graph.py

The workflow state is modeled in:

- app/agents/state.py

The graph progresses through a sequence of nodes:

1. Topology parser
   - transforms raw input into structured topology data

2. Security analyzer
   - retrieves relevant compliance controls from Qdrant
   - evaluates the topology against those controls
   - produces a list of compliance gaps

3. Remediation compiler
   - synthesizes GRC report output
   - generates Terraform remediation code

This creates a pipeline where data is progressively transformed from raw infrastructure input into actionable security outputs.

### 6. Data and memory layer

The project uses Qdrant for vector-based storage and retrieval of compliance controls.

Relevant files:

- docker-compose.yml
- ingest_compliance.py

This provides:

- semantic retrieval of framework control descriptions
- framework-aware memory for policy evaluation
- separation between prompts and knowledge base content

The system also uses embedding models from Ollama, which supports context-grounded search and similarity matching.

### 7. Domain model layer

The infrastructure domain model is defined in:

- app/models/network.py

It includes:

- traffic direction types
- trust level definitions
- network nodes
- traffic pathways
- security zones
- complete topology schema

This is crucial because the product must turn messy infrastructure details into a consistent structure that LLMs and validators can interpret reliably.

---

## Workflow Execution Model

The system works in a sequence like this:

1. User provides organization profile and architecture classification.
2. The frontend requests provider schemas.
3. User provides credentials or configuration for one or more providers.
4. The backend validates the provider payload against a Pydantic model.
5. The selected strategy executes provider-specific discovery logic.
6. Telemetry is aggregated into a unified environment summary.
7. The LangGraph workflow receives the summarized infrastructure context.
8. The topology parser extracts structured topology details.
9. The security analyzer retrieves relevant compliance controls from Qdrant.
10. The analyzer finds security gaps and risk conditions.
11. The remediation compiler generates final outputs.
12. The result is returned to the client as Markdown and Terraform artifacts.

This is the product’s central operational loop.

---

## Key Files and Responsibilities

### app/main.py
Main FastAPI application, including:

- API initialization
- CORS setup
- provider schema endpoints
- enterprise audit orchestration
- router registration

### app/core/ingestion.py
Defines ingestion schemas and provider strategy classes.

### app/core/llm.py
Defines model selection and routing between local and cloud AI services.

### app/core/orchestrator.py
Contains a reusable enterprise audit orchestration flow for sequential multi-environment workflows.

### app/agents/state.py
Defines the graph state schema.

### app/agents/graphs/review_graph.py
Defines the LangGraph workflow and node behaviors.

### app/models/network.py
Defines the shared infrastructure topology schema.

### ingest_compliance.py
Seeds the framework-specific knowledge base in Qdrant.

### test_chassis.py
Provides an operational entry point for running a live system audit and exporting generated artifacts.

### frontend/src/App.jsx
Provides the onboarding and result workflow for the UI.

---

## Current Strengths

### 1. Strong architecture for AI-assisted governance
The product has a clear layered design: frontend, API, ingestion, graph workflow, memory layer, and output generation.

### 2. Multi-cloud and hybrid intent
The design is not limited to a single cloud provider or a single infrastructure model.

### 3. Practical remediation output
The platform goes beyond findings by producing Terraform and report artifacts, which makes it useful to engineering teams.

### 4. Hybrid model routing
The local/cloud model split is a good design choice for cost and performance optimization.

### 5. Extensible provider architecture
The strategy pattern supports expansion without disrupting the rest of the system.

---

## Current Limitations

### 1. Prototype-level discovery abstraction
The ingestion layer currently acts more like a strategic simulation framework than a full production cloud discovery engine.

### 2. Need for stronger evidence validation
The platform should validate that findings are grounded in actual evidence before presenting them as final compliance conclusions.

### 3. Product maturity still early
The project is highly useful as a proof of concept but still requires hardening to become a production-ready SaaS or enterprise platform.

### 4. Operational readiness is still developing
Functions such as audit history, job persistence, metrics, observability, role-based access, and secure secret management are still areas for growth.

---

## Recommended Product Direction

To evolve from a prototype into a mature product, the team should prioritize:

1. real cloud discovery integrations
2. persistent audit job records
3. stronger evidence-based validation for findings
4. formalized output quality checks
5. dashboard and audit history features
6. secure configuration and secret management
7. deployment and monitoring infrastructure

At that point, the system can become a real enterprise governance and posture-management platform.

---

## Conclusion

ParadigmSec is a compelling AI-first governance and compliance product concept. It already integrates the right ingredients for an enterprise-grade compliance engine:

- structured infrastructure modeling
- multi-provider intake
- vector-backed control retrieval
- a LangGraph reasoning workflow
- generated remediation artifacts

The architecture is coherent, modular, and extensible. The main work remaining is moving from conceptual automation to production-grade evidence, operations, and platform reliability.

This makes the project an excellent prototype and a strong foundation for a future compliance automation platform.
