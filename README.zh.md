# RavenAI

[English](README.md) | 中文

RavenAI 是一个面向研发与测试工作流的 AI 平台。它把研发代码提交、升级包管理、智能检索、自然语言设备控制、测试日志提交，以及 Agentic 日志分析串联成一个闭环。

RavenAI 不绑定特定业务领域。它最初为卫星通信领域设计，现已演进为一个通用平台，目标是承载**任意项目**的研发与测试工作流；并且通过定制开发，还可以承载非研发角色的 Agent。

本仓库是顶层工作区，包含两个主要子项目：

- `RavenClient`：基于 Electron 与 Cherry Studio 定制的桌面客户端，面向研发与测试人员，提供 MCP 设备操作、升级包制作和 AI 工作流。
- `RavenAIService`：后端服务集合，提供日志暂存、设备联动、AI 对话、异步日志处理、包管理和智能包检索等能力。

## 项目解决的问题

RavenAI 围绕真实研发测试流程设计：

1. 研发人员通过 Git 提交代码并推进版本演进。
2. 简单的 Release Note Agent 读取 Git 提交历史，自动编写包管理重构相关 Release Note。
3. 重构后的包管理能力支持升级包制作、上传、元数据提取、检索和分发。
4. 测试人员通过 RavenClient、ChatAgent、Device Link、MCP Server、设备工具和 Python 自动化脚本，用自然语言管理测试设备。
5. 测试完成后，日志提交至 RavenAIService，由 LogAnalysisAgent 分析并产出证据链、问题线索和研发回流建议。

## 研发测试流程对比

没有 Agent 参与时，研发测试流程主要依赖人工交接、人工找包、手动操作设备和人工查日志：

```mermaid
%%{init: {"theme": "base", "flowchart": {"htmlLabels": true, "nodeSpacing": 24, "rankSpacing": 24, "curve": "basis"}, "themeVariables": {"fontFamily": "Arial, Microsoft YaHei", "primaryTextColor": "#0F172A", "lineColor": "#64748B"}}}%%

flowchart LR

A["研发提交"] --> B["写说明"]
B --> C["传包"]
C --> D["找包"]
D --> E["控设备"]
E --> F["日志散落"]
F --> G["查日志"]
G --> H["研发修复"]
H --> A

P1["沟通反复"] -.-> B
P2["版本混乱"] -.-> C
P3["门槛高"] -.-> E
P4["定位慢"] -.-> G

classDef normal fill:#F8FAFC,stroke:#64748B,stroke-width:1.3px,color:#0F172A;
classDef manual fill:#FEF2F2,stroke:#DC2626,stroke-width:1.5px,color:#7F1D1D;
classDef pain fill:#FFF7ED,stroke:#EA580C,stroke-width:1.3px,color:#7C2D12;

class A,H normal;
class B,C,D,E,F,G manual;
class P1,P2,P3,P4 pain;
```

在 RavenAI 中，Agent 进入说明生成、包管理、查包、设备控制和日志分析这些关键摩擦点：

```mermaid
%%{init: {"theme": "base", "flowchart": {"htmlLabels": true, "nodeSpacing": 24, "rankSpacing": 24, "curve": "basis"}, "themeVariables": {"fontFamily": "Arial, Microsoft YaHei", "primaryTextColor": "#0F172A", "lineColor": "#475569"}}}%%

flowchart LR

A["研发提交"] --> B["Agent说明"]
B --> C["Agent管包"]
C --> D["Agent查包"]
D --> E["Agent控设备"]
E --> F["日志入库"]
F --> G["Agent析日志"]
G --> H["研发修复"]
H --> I["AI Coding Agent"]
I --> A

V1["降沟通"] -.-> B
V2["降交付"] -.-> C
V3["降门槛"] -.-> E
V4["快定位"] -.-> G

classDef normal fill:#F8FAFC,stroke:#64748B,stroke-width:1.3px,color:#0F172A;
classDef agent fill:#F5F3FF,stroke:#7C3AED,stroke-width:1.7px,color:#1E1B4B;
classDef value fill:#ECFDF5,stroke:#059669,stroke-width:1.3px,color:#064E3B;
classDef repair fill:#FFF7ED,stroke:#EA580C,stroke-width:1.5px,color:#7C2D12;

class A,F normal;
class B,C,D,E,G,I agent;
class V1,V2,V3,V4,V5 value;
class H repair;
```

## 业务作用矩阵

```mermaid
%% 建议导出画布：1200x900 或 1600x1200，4:3
%% 主题：RavenAI 在研发测试流程中的业务作用
%%{init: {"theme": "base", "flowchart": {"htmlLabels": true, "nodeSpacing": 24, "rankSpacing": 30, "curve": "basis"}, "themeVariables": {"fontFamily": "Arial, Microsoft YaHei", "primaryTextColor": "#0F172A", "lineColor": "#CBD5E1", "clusterBkg": "#FFFFFF", "clusterBorder": "#CBD5E1"}}}%%

flowchart TB

subgraph M["业务作用矩阵"]
direction TB

  subgraph H[" "]
  direction LR
    H1["研发资产"]
    H2["包交付"]
    H3["测试提效"]
    H4["分析回流"]
  end

  subgraph R1["核心场景"]
  direction LR
    A1["代码提交<br/>PR / Tag<br/>版本追踪"]
    A2["包管理<br/>补丁 / 配置 / 完整包<br/>上传下载"]
    A3["设备控制<br/>查状态 / 升级 / 配置<br/>少敲命令"]
    A4["日志提交<br/>暂存 / 检索<br/>测试留痕"]
  end

  subgraph R2["流程价值"]
  direction LR
    B1["少沟通<br/>变更可查<br/>自动说明"]
    B2["少乱版<br/>快速找包<br/>标签筛选"]
    B3["低门槛<br/>聚焦测试<br/>意图转动作"]
    B4["快定位<br/>提取错误<br/>结论回流"]
  end

  subgraph R3["直接产物"]
  direction LR
    C1[("提交记录<br/>版本历史<br/>Release Note")]
    C2[("升级包<br/>metadata<br/>vector-store")]
    C3[("设备状态<br/>执行回执<br/>任务记录")]
    C4[("日志结论<br/>根因线索<br/>修复建议")]
  end

end

C1 -. "支撑说明" .- A2
C2 -. "选择版本+测试版本" .- A3
C3 -. "生成日志" .- A4
C4 -. "回流研发" .- A1

classDef head fill:#0F172A,stroke:#0F172A,stroke-width:1.2px,color:#FFFFFF;
classDef scene fill:#FFF7ED,stroke:#EA580C,stroke-width:1.2px,color:#7C2D12;
classDef value fill:#EFF6FF,stroke:#2563EB,stroke-width:1.2px,color:#0F172A;
classDef data fill:#ECFDF5,stroke:#059669,stroke-width:1.2px,color:#064E3B;

class H1,H2,H3,H4 head;
class A1,A2,A3,A4 scene;
class B1,B2,B3,B4 value;
class C1,C2,C3,C4 data;
```

## Agent 能力矩阵

```mermaid
%% 建议导出画布：1200x900 或 1600x1200，4:3
%% 主题：RavenAI 的 Agent 能力架构
%%{init: {"theme": "base", "flowchart": {"htmlLabels": true, "nodeSpacing": 24, "rankSpacing": 30, "curve": "basis"}, "themeVariables": {"fontFamily": "Arial, Microsoft YaHei", "primaryTextColor": "#0F172A", "lineColor": "#CBD5E1", "clusterBkg": "#FFFFFF", "clusterBorder": "#CBD5E1"}}}%%

flowchart TB

subgraph M["Agent 能力矩阵"]
direction TB

  subgraph R1["输入理解"]
  direction LR
    A1["Git 提交<br/>变更代码<br/>版本背景"]
    A2["包元数据<br/>版本 / 类型 / 标签"]
    A3["测试目标<br/>设备能力<br/>领域知识"]
    A4["业务代码<br/>类型识别<br/>领域知识"]
  end

  subgraph R2["推理与执行"]
  direction LR
    B1["生成 Release Note<br/>整理提交<br/>补齐背景"]
    B2["RAG 检索<br/>FAISS + LLM<br/>建议 / 重建"]
    B3["LangGraph ReAct<br/>Plan / Act / Observe<br/>单步护栏"]
    B4["ReAct 分析<br/>检索 / 读取 / 总结<br/>工具调用"]
  end

  subgraph R3["技术落点"]
  direction LR
    C1["Git log<br/>提交历史<br/>版本标签"]
    C2["package-server<br/>RAGService<br/>PackageService<br/>vector-store"]
    C3["ChatAgent<br/>Device Link WebSocket<br/>MCP Server<br/>OAM 工具 / Python 脚本"]
    C4["RavenAIService<br/>Celery + Redis<br/>LogAnalysisAgent<br/>日志工具集"]
  end

  subgraph R4["输出结果"]
  direction LR
    D1[("Release Note<br/>变更摘要<br/>测试关注")]
    D2[("目标包<br/>推荐版本<br/>匹配原因")]
    D3[("测试结果<br/>状态 / 升级 / 配置<br/>执行证据")]
    D4[("问题分析报告<br/>关键错误<br/>修复建议")]
  end

end

D1 -. "说明版本" .- A1
D2 -. "进入测试" .- A2
D3 -. "产生日志" .- A3
D4 -. "反馈修复" .- A4

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

style A1 font-size:34px
style A2 font-size:34px
style A3 font-size:34px
style A4 font-size:34px
style D1 font-size:24px
style D2 font-size:24px
style D3 font-size:24px
style D4 font-size:24px
```

## 技术架构概览

### 系统技术与能力结构

![RavenAI 架构图](Docs/raven-architecture.svg)

- **客户端层**：`RavenClient`、Electron、React、MCP Client、包管理工具、DeviceLinkClient。
- **AI 编排层**：基于 Claude Agent SDK 的 ChatAgent 与 LogAnalysisAgent，接入 OpenAI-compatible LLM。
- **服务层**：`RavenAIService` 中的 FastAPI 服务、Express `package-server`，以及 Electron `update-server`。
- **通信与任务层**：REST API、WebSocket Device Link、Celery、Redis。
- **数据与知识层**：升级包、包元数据、日志包、数据库记录和 FAISS 向量索引。
- **设备工具层**：MCP Server、设备接口封装和面向设备操作与测试的 Python 自动化脚本。

关键架构说明：
- **提示词分层**：Agent 级基础层（`prompts_config.yaml`）与 Project 级追加层（`data/project_prompts/<project_code>/system_prompt.md`）在运行前合并，编辑即时生效。
- **Skill 分层（Claude Agent SDK）**：Agent 级（`data/agent_skills/<agent>/store`）与 Project 级（`data/project_skills/<project_code>/store`），运行前物化到 `.claude/skills/`。
- **Service ⇄ Client 反向穿透**：客户端主动外连 WS 到 `Service:8085/ws/device-link`；心跳 ping/pong + 指数退避重连；设备侧无需公网入站端口。
- **代码库安全接入**：LogAnalysis / BugFix / ProjectExpert Agent 经 SSH Key / Token 认证访问代码库，克隆为只读副本。

### Agent 上下文分层与装配

![RavenAI Agent 上下文](Docs/raven-agent-context.svg)

每次 Agent 运行前，从两个层次组装上下文：

- **Agent 级（基础层）**：按 agent 名称选取，跨所有项目共享。提示词来自 `prompts_config.yaml`（`claude_agent_<name>`，按 zh/en locale 选择），Skill 来自 `data/agent_skills/<agent>/store`（由 `_registry.json` 控制启用）。
- **Project 级（追加层）**：按 `project_code` 隔离，用户在前端选定项目后生效。提示词来自 `project_prompts/<code>/system_prompt.md`（编辑即时生效），Skill 来自 `project_skills/<code>/store`（与 Agent Skill 平行；同名则覆盖 Agent 级）。
- **运行前组装**：最终系统提示词 = base + 项目追加段 + 回复语言指令；Skill 物化到 `.claude/skills/`（Agent Skill 先放入，项目级同名覆盖）；代码库 `git clone` 到 `<workspace>/repo/`（已有则复用 `.git`）。
- **工作区 `<workspace>/`**：包含合成后的 `system_prompt`、`.claude/skills/<name>/`、`repo/`（只读：Read/Grep/Glob/Bash/git）和 `task.json + logs/`。

## 仓库结构

```text
RavenAI/
├── RavenClient/                 # 桌面客户端、包工具、MCP 与设备联动流程
├── RavenAIService/              # 后端服务、日志暂存、AI 分析、包管理服务
│   ├── app/                     # FastAPI 应用、agents、services、tasks
│   ├── frontend/                # 服务侧 Web 前端
│   └── package-server/          # Express 包管理与 RAG 检索服务
├── Docs/                        # 项目文档与功能说明
├── .gitmodules                  # 子模块定义
└── README.md                    # 默认英文说明文档
```

## 快速开始

克隆仓库并拉取子模块：

```bash
git clone --recurse-submodules <repo-url>
cd RavenAI
```

如果克隆时没有包含子模块：

```bash
git submodule update --init --recursive
```

启动日志暂存后端服务：

```bash
cd RavenAIService
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --host 0.0.0.0 --port 8085 --reload
```

启动包管理服务：

```bash
cd RavenAIService/package-server
npm install
npm run dev
```

启动桌面客户端：

```bash
cd RavenClient
yarn install
yarn dev
```

## 关键服务

- RavenAIService API：`http://localhost:8085`
- 包管理服务：`http://localhost:8083/raven`
- 更新服务：`RavenClient/update-server`
- 桌面客户端：通过 `yarn dev` 启动 Electron

## 说明

流程中提到的 Release Note Agent 是项目研发流程的一部分：它通过读取 Git 提交历史，自动编写包管理重构相关 Release Note。虽然它不是当前代码库中的完整独立模块，但它补齐了从研发提交到测试交付之间的重要环节，因此在架构说明中保留。
