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

## Business Role Matrix

```mermaid
%% Suggested export canvas: 1200x900 or 1600x1200, 4:3
%% Topic: RavenAI business role in the R&D testing workflow
%%{init: {"theme": "base", "flowchart": {"htmlLabels": true, "nodeSpacing": 14, "rankSpacing": 16, "curve": "basis"}, "themeVariables": {"fontFamily": "Arial, Microsoft YaHei", "primaryTextColor": "#0F172A", "lineColor": "#CBD5E1", "clusterBkg": "#FFFFFF", "clusterBorder": "#CBD5E1"}}}%%

flowchart TB

subgraph M["Business Role Matrix"]
direction TB

  subgraph H[" "]
  direction LR
    H1["R&D Asset<br/>Accumulation"]
    H2["Release Package<br/>Delivery"]
    H3["Test Execution<br/>Acceleration"]
    H4["Issue Analysis<br/>Feedback"]
  end

  subgraph R1["Core Scenario"]
  direction LR
    A1["Code submission<br/>PR / Tag / version evolution<br/>Traceable engineering assets"]
    A2["Reconstructed package management<br/>Patch / config / full package<br/>Unified upload, download, statistics"]
    A3["Natural-language device control<br/>Status query / upgrade / configuration<br/>Fewer manual Linux operations"]
    A4["Post-test log submission<br/>Unified staging, management, retrieval<br/>Reusable test records"]
  end

  subgraph R2["Workflow Value"]
  direction LR
    B1["Lower handoff cost<br/>Version changes are traceable<br/>Release notes preserve context"]
    B2["Less version confusion<br/>Testers find target packages quickly<br/>Filter by version, type, and tags"]
    B3["Lower device operation barrier<br/>Testers focus on intent<br/>AI decomposes intent into actions"]
    B4["Shorter debugging loop<br/>Extract key errors from logs<br/>Feed conclusions back to development"]
  end

  subgraph R3["Direct Output"]
  direction LR
    C1[("Commit records<br/>Version history<br/>Release note input")]
    C2[("Package assets<br/>uploads<br/>package-metadata.json<br/>vector-store")]
    C3[("Device status and receipts<br/>Upgrade result<br/>Configuration result<br/>Task record")]
    C4[("Logs and analysis conclusions<br/>Key errors<br/>Root-cause clues<br/>Fix suggestions")]
  end

end

C1 -. "Supports package-change notes" .- A2
C2 -. "Provides versions for testing" .- A3
C3 -. "Produces test logs" .- A4
C4 -. "Feeds issues back" .- A1

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
%%{init: {"theme": "base", "flowchart": {"htmlLabels": true, "nodeSpacing": 14, "rankSpacing": 16, "curve": "basis"}, "themeVariables": {"fontFamily": "Arial, Microsoft YaHei", "primaryTextColor": "#0F172A", "lineColor": "#CBD5E1", "clusterBkg": "#FFFFFF", "clusterBorder": "#CBD5E1"}}}%%

flowchart TB

subgraph M["Agent Capability Matrix"]
direction TB

  subgraph H[" "]
  direction LR
    H1["Release Note<br/>Agent"]
    H2["Package Management<br/>and Retrieval"]
    H3["Device Testing<br/>ChatAgent"]
    H4["Log Analysis<br/>LogAnalysisAgent"]
  end

  subgraph R1["Input Understanding"]
  direction LR
    A1["Reads Git commits<br/>Identifies package-management refactor changes<br/>Extracts version context"]
    A2["Understands package metadata<br/>Version / type / component / tag<br/>Supports natural-language queries"]
    A3["Understands tester goals<br/>Combines device capability prompts<br/>Recognizes query, upgrade, config actions"]
    A4["Understands logs and issue descriptions<br/>Infers stack / OAM / antenna types<br/>Reads metadata.json"]
  end

  subgraph R2["Reasoning and Execution"]
  direction LR
    B1["Generates release notes<br/>Organizes changes from Git history<br/>Captures process context outside code"]
    B2["RAG package search<br/>Embeddings + FAISS + LLM<br/>Similarity search / suggestions / index rebuild"]
    B3["LangGraph ReAct<br/>Plan / Act / Observe<br/>Single-step device_prompt guardrail<br/>Avoids unsafe chained operations"]
    B4["LangGraph ReAct analysis<br/>Plan / search / read / summarize<br/>grep / fs / metadata / archive / search"]
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
    D1[("Package-management release notes<br/>Change summary<br/>Impact scope<br/>Testing focus")]
    D2[("Target packages<br/>Recommended version<br/>Match reason<br/>Downloadable asset")]
    D3[("Device action results<br/>Status query<br/>Upgrade receipt<br/>Configuration result<br/>Execution evidence")]
    D4[("Log analysis report<br/>Key errors<br/>Evidence chain<br/>Root-cause clues<br/>Fix suggestions")]
  end

end

D1 -. "Explains version changes" .- A2
D2 -. "Enters device testing" .- A3
D3 -. "Produces log evidence" .- A4
D4 -. "Feeds fixes back" .- A1

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
