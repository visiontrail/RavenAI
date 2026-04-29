# RavenAI

English | [中文](README.zh.md)

RavenAI is an AI-powered satellite baseband payload development and testing platform. It connects the workflow from engineering commits, release package management, intelligent package retrieval, natural-language device operation, test log submission, and agentic log analysis.

This repository is the top-level workspace for two major subprojects:

- `RavenClient`: an Electron-based desktop client built on Cherry Studio, customized for satellite payload testing, MCP device operations, package creation, and tester-facing AI workflows.
- `RavenAIService`: backend services for log staging, device-link orchestration, AI chat, asynchronous log processing, package management, and intelligent package search.

## What RavenAI Solves

RavenAI is designed around the real R&D testing loop:

1. Developers submit code and evolve versions through Git.
2. A lightweight Release Note Agent reads Git history and automatically drafts release notes for package-management refactors.
3. Reconstructed package management supports package creation, upload, metadata extraction, search, and distribution.
4. Testers use natural language to operate test devices through RavenClient, ChatAgent, Device Link, MCP Server, OAM tools, and Python automation.
5. After testing, logs are submitted to RavenAIService and analyzed by LogAnalysisAgent to produce evidence, issue clues, and feedback for development.

## R&D Testing Flow Comparison

Without Agent participation, the R&D testing flow relies on manual handoffs, manual package lookup, manual device operations, and manual log inspection:

```mermaid
%%{init: {"theme": "base", "flowchart": {"htmlLabels": true, "nodeSpacing": 24, "rankSpacing": 24, "curve": "basis"}, "themeVariables": {"fontFamily": "Arial, Microsoft YaHei", "primaryTextColor": "#0F172A", "lineColor": "#64748B"}}}%%

flowchart LR

A["Code Commit"] --> B["Manual Notes"]
B --> C["Package Handoff"]
C --> D["Package Search"]
D --> E["Device Ops"]
E --> F["Scattered Logs"]
F --> G["Manual Debug"]
G --> H["Fix"]

P1["Handoff Loops"] -.-> B
P2["Version Drift"] -.-> C
P3["High Barrier"] -.-> E
P4["Slow Debug"] -.-> G

classDef normal fill:#F8FAFC,stroke:#64748B,stroke-width:1.3px,color:#0F172A;
classDef manual fill:#FEF2F2,stroke:#DC2626,stroke-width:1.5px,color:#7F1D1D;
classDef pain fill:#FFF7ED,stroke:#EA580C,stroke-width:1.3px,color:#7C2D12;

class A,H normal;
class B,C,D,E,F,G manual;
class P1,P2,P3,P4 pain;
```

With RavenAI, Agents sit in the key friction points: release notes, package management, package retrieval, device control, and log analysis:

```mermaid
%%{init: {"theme": "base", "flowchart": {"htmlLabels": true, "nodeSpacing": 24, "rankSpacing": 24, "curve": "basis"}, "themeVariables": {"fontFamily": "Arial, Microsoft YaHei", "primaryTextColor": "#0F172A", "lineColor": "#475569"}}}%%

flowchart LR

A["Code Commit"] --> B["Agent Notes"]
B --> C["Agent Packages"]
C --> D["Agent Search"]
D --> E["Agent Device"]
E --> F["Log Staging"]
F --> G["Agent Analysis"]
G --> H["Fix"]

V1["Less Handoff"] -.-> B
V2["Lower Delivery"] -.-> C
V3["Lower Barrier"] -.-> E
V4["Faster Diagnosis"] -.-> G

classDef normal fill:#F8FAFC,stroke:#64748B,stroke-width:1.3px,color:#0F172A;
classDef agent fill:#F5F3FF,stroke:#7C3AED,stroke-width:1.7px,color:#1E1B4B;
classDef value fill:#ECFDF5,stroke:#059669,stroke-width:1.3px,color:#064E3B;
classDef repair fill:#FFF7ED,stroke:#EA580C,stroke-width:1.5px,color:#7C2D12;

class A,F normal;
class B,C,D,E,G agent;
class V1,V2,V3,V4 value;
class H repair;
```

## Business Role Matrix

```mermaid
%% Suggested export canvas: 1200x900 or 1600x1200, 4:3
%% Topic: RavenAI business role in the R&D testing workflow
%%{init: {"theme": "base", "flowchart": {"htmlLabels": true, "nodeSpacing": 24, "rankSpacing": 30, "curve": "basis"}, "themeVariables": {"fontFamily": "Arial, Microsoft YaHei", "primaryTextColor": "#0F172A", "lineColor": "#CBD5E1", "clusterBkg": "#FFFFFF", "clusterBorder": "#CBD5E1"}}}%%

flowchart TB

subgraph M["Business Role Matrix"]
direction TB

  subgraph H[" "]
  direction LR
    H1["R&D Assets"]
    H2["Package<br/>Delivery"]
    H3["Faster<br/>Testing"]
    H4["Analysis<br/>Feedback"]
  end

  subgraph R1["Core Scenario"]
  direction LR
    A1["Code commits<br/>PR / Tag<br/>Version trace"]
    A2["Package management<br/>Patch / config / full<br/>Upload / download"]
    A3["Device control<br/>Status / upgrade / config<br/>Fewer commands"]
    A4["Log submission<br/>Stage / search<br/>Test records"]
  end

  subgraph R2["Workflow Value"]
  direction LR
    B1["Less handoff<br/>Trace changes<br/>Auto notes"]
    B2["Less version drift<br/>Fast package find<br/>Tag filters"]
    B3["Lower barrier<br/>Focus on tests<br/>Intent to action"]
    B4["Faster debug<br/>Extract errors<br/>Feedback loop"]
  end

  subgraph R3["Direct Output"]
  direction LR
    C1[("Commit records<br/>Version history<br/>Release notes")]
    C2[("Packages<br/>metadata<br/>vector-store")]
    C3[("Device status<br/>Receipts<br/>Task records")]
    C4[("Log findings<br/>Root-cause clues<br/>Fix suggestions")]
  end

end

C1 -. "Supports notes" .- A2
C2 -. "Select version + Test version" .- A3
C3 -. "Creates logs" .- A4
C4 -. "Feeds back" .- A1

classDef head fill:#0F172A,stroke:#0F172A,stroke-width:1.2px,color:#FFFFFF;
classDef scene fill:#FFF7ED,stroke:#EA580C,stroke-width:1.2px,color:#7C2D12;
classDef value fill:#EFF6FF,stroke:#2563EB,stroke-width:1.2px,color:#0F172A;
classDef data fill:#ECFDF5,stroke:#059669,stroke-width:1.2px,color:#064E3B;

class H1,H2,H3,H4 head;
class A1,A2,A3,A4 scene;
class B1,B2,B3,B4 value;
class C1,C2,C3,C4 data;
```

## Agent Capability Matrix

```mermaid
%% Suggested export canvas: 1200x900 or 1600x1200, 4:3
%% Topic: RavenAI agent capabilities
%%{init: {"theme": "base", "flowchart": {"htmlLabels": true, "nodeSpacing": 24, "rankSpacing": 30, "curve": "basis"}, "themeVariables": {"fontFamily": "Arial, Microsoft YaHei", "primaryTextColor": "#0F172A", "lineColor": "#CBD5E1", "clusterBkg": "#FFFFFF", "clusterBorder": "#CBD5E1"}}}%%

flowchart TB

subgraph M["Agent Capability Matrix"]
direction TB

  subgraph R1["Input Understanding"]
  direction LR
    A1["Git commits<br/>Change detection<br/>Version context"]
    A2["Package metadata<br/>Version / type / tag<br/>Natural queries"]
    A3["Tester goals<br/>Device capability<br/>Action detection"]
    A4["Logs and issues<br/>Type detection<br/>metadata.json"]
  end

  subgraph R2["Reasoning and Execution"]
  direction LR
    B1["Generate notes<br/>Organize commits<br/>Add context"]
    B2["RAG search<br/>FAISS + LLM<br/>Suggest / rebuild"]
    B3["LangGraph ReAct<br/>Plan / Act / Observe<br/>Single-step guardrail"]
    B4["ReAct analysis<br/>Search / read / summarize<br/>Tool calls"]
  end

  subgraph R3["Technical Anchor"]
  direction LR
    C1["Git log<br/>Commit history<br/>Version tags"]
    C2["package-server<br/>RAGService<br/>PackageService<br/>vector-store"]
    C3["ChatAgent<br/>Device Link WebSocket<br/>MCP Server<br/>OAM tools / Python scripts"]
    C4["RavenAIService<br/>Celery + Redis<br/>LogAnalysisAgent<br/>Log toolset"]
  end

  subgraph R4["Output"]
  direction LR
    D1[("Release notes<br/>Change summary<br/>Test focus")]
    D2[("Target package<br/>Recommended version<br/>Match reason")]
    D3[("Device result<br/>Status / upgrade / config<br/>Evidence")]
    D4[("Analysis report<br/>Key errors<br/>Fix suggestions")]
  end

end

D1 -. "Explains version" .- A2
D2 -. "Starts testing" .- A3
D3 -. "Produces logs" .- A4
D4 -. "Feeds fixes" .- A1

classDef head fill:#312E81,stroke:#312E81,stroke-width:1.2px,color:#FFFFFF;
classDef input fill:#FFF7ED,stroke:#EA580C,stroke-width:1.2px,color:#7C2D12;
classDef agent fill:#F5F3FF,stroke:#7C3AED,stroke-width:1.3px,color:#1E1B4B;
classDef tech fill:#EFF6FF,stroke:#2563EB,stroke-width:1.2px,color:#0F172A;
classDef output fill:#ECFDF5,stroke:#059669,stroke-width:1.2px,color:#064E3B;

class H1,H2,H3,H4 head;
class A1,A2,A3,A4 input;
class B1,B2,B3,B4 agent;
class C1,C2,C3,C4 tech;
class D1,D2,D3,D4 output;
```

## Architecture Overview

- **Client layer**: `RavenClient`, Electron, React, MCP Client, package tools, device-link client.
- **AI orchestration layer**: ChatAgent and LogAnalysisAgent built with LangGraph/LangChain and OpenAI-compatible LLM access.
- **Service layer**: FastAPI services in `RavenAIService`, Express-based `package-server`, and the Electron `update-server`.
- **Communication and tasks**: REST APIs, WebSocket Device Link, Celery, and Redis.
- **Data and knowledge layer**: uploaded packages, package metadata, log archives, database records, and FAISS vector indexes.
- **Device tool layer**: MCP Server, OAM interface wrappers, and Python automation scripts for satellite payload testing.

## Repository Layout

```text
RavenAI/
├── RavenClient/                 # Desktop client, package tooling, MCP and device workflows
├── RavenAIService/              # Backend services, log staging, AI analysis, package server
│   ├── app/                     # FastAPI application, agents, services, tasks
│   ├── frontend/                # Web frontend for service-side UI
│   └── package-server/          # Express package management and RAG search service
├── Docs/                        # Project documentation and feature notes
├── .gitmodules                  # Submodule definitions
└── README.md                    # This document
```

## Quick Start

Clone the repository with submodules:

```bash
git clone --recurse-submodules <repo-url>
cd RavenAI
```

If the repository was cloned without submodules:

```bash
git submodule update --init --recursive
```

Start the backend log staging service:

```bash
cd RavenAIService
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --host 0.0.0.0 --port 8085 --reload
```

Start the package server:

```bash
cd RavenAIService/package-server
npm install
npm run dev
```

Start the desktop client:

```bash
cd RavenClient
yarn install
yarn dev
```

## Key Services

- RavenAIService API: `http://localhost:8085`
- Package server: `http://localhost:8083/raven`
- Update server: `RavenClient/update-server`
- Desktop client: launched through Electron during `yarn dev`

## Notes

The Release Note Agent mentioned in the workflow is part of the project process: it reads Git commit history and drafts release notes for the package-management refactor. It is included in the architecture description because it completes the R&D testing workflow, even though it is not represented as a full standalone module in this codebase.
