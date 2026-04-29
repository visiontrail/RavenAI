# RavenAI

[English](README.md) | 中文

RavenAI 是面向卫星基带载荷研发测试场景的 AI 平台。它把研发代码提交、升级包管理、智能检索、自然语言设备控制、测试日志提交，以及 Agentic 日志分析串联成一个闭环。

本仓库是顶层工作区，包含两个主要子项目：

- `RavenClient`：基于 Electron 与 Cherry Studio 定制的桌面客户端，面向卫星载荷测试、MCP 设备操作、升级包制作和测试人员 AI 工作流。
- `RavenAIService`：后端服务集合，提供日志暂存、设备联动、AI 对话、异步日志处理、包管理和智能包检索等能力。

## 项目解决的问题

RavenAI 围绕真实研发测试流程设计：

1. 研发人员通过 Git 提交代码并推进版本演进。
2. 简单的 Release Note Agent 读取 Git 提交历史，自动编写包管理重构相关 Release Note。
3. 重构后的包管理能力支持升级包制作、上传、元数据提取、检索和分发。
4. 测试人员通过 RavenClient、ChatAgent、Device Link、MCP Server、OAM 工具和 Python 自动化脚本，用自然语言管理测试设备。
5. 测试完成后，日志提交至 RavenAIService，由 LogAnalysisAgent 分析并产出证据链、问题线索和研发回流建议。

## 业务作用矩阵

```mermaid
%% 建议导出画布：1200x900 或 1600x1200，4:3
%% 主题：RavenAI 在研发测试流程中的业务作用
%%{init: {"theme": "base", "flowchart": {"htmlLabels": true, "nodeSpacing": 14, "rankSpacing": 16, "curve": "basis"}, "themeVariables": {"fontFamily": "Arial, Microsoft YaHei", "primaryTextColor": "#0F172A", "lineColor": "#CBD5E1", "clusterBkg": "#FFFFFF", "clusterBorder": "#CBD5E1"}}}%%

flowchart TB

subgraph M["业务作用矩阵"]
direction TB

  subgraph H[" "]
  direction LR
    H1["研发资产沉淀"]
    H2["升级包交付"]
    H3["测试执行提效"]
    H4["问题分析回流"]
  end

  subgraph R1["核心场景"]
  direction LR
    A1["研发代码提交<br/>PR / Tag / 版本演进<br/>形成可追踪研发资产"]
    A2["重构升级包管理<br/>补丁包 / 配置包 / 完整包<br/>统一上传、下载、统计"]
    A3["自然语言管理测试设备<br/>查询状态 / 启动升级 / 下发配置<br/>减少手工 Linux 操作"]
    A4["测试完成后提交日志<br/>统一暂存、管理、检索<br/>沉淀可复用测试记录"]
  end

  subgraph R2["流程价值"]
  direction LR
    B1["降低研发交付沟通成本<br/>版本变化有据可查<br/>Release Note 自动补齐背景"]
    B2["减少包分发与版本混乱<br/>测试人员始终可找到目标包<br/>支持按版本、类型、标签筛选"]
    B3["降低设备操作门槛<br/>测试人员专注测试目标<br/>AI 将意图拆成可执行动作"]
    B4["缩短问题定位链路<br/>从日志中提取关键错误<br/>将结论回流给研发修复"]
  end

  subgraph R3["直接产物"]
  direction LR
    C1[("提交记录<br/>版本历史<br/>包管理重构说明")]
    C2[("升级包资产<br/>uploads<br/>package-metadata.json<br/>vector-store")]
    C3[("设备状态与执行回执<br/>升级结果<br/>配置结果<br/>任务记录")]
    C4[("日志与分析结论<br/>关键错误<br/>根因线索<br/>修复建议")]
  end

end

C1 -. "支撑包管理说明" .- A2
C2 -. "供测试选择版本" .- A3
C3 -. "产生测试日志" .- A4
C4 -. "问题回流研发" .- A1

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
%%{init: {"theme": "base", "flowchart": {"htmlLabels": true, "nodeSpacing": 14, "rankSpacing": 16, "curve": "basis"}, "themeVariables": {"fontFamily": "Arial, Microsoft YaHei", "primaryTextColor": "#0F172A", "lineColor": "#CBD5E1", "clusterBkg": "#FFFFFF", "clusterBorder": "#CBD5E1"}}}%%

flowchart TB

subgraph M["Agent 能力矩阵"]
direction TB

  subgraph H[" "]
  direction LR
    H1["Release Note Agent"]
    H2["包管理与检索 Agent 能力"]
    H3["设备测试 ChatAgent"]
    H4["日志分析 LogAnalysisAgent"]
  end

  subgraph R1["输入理解"]
  direction LR
    A1["读取 Git 提交记录<br/>识别包管理重构相关变更<br/>提炼版本演进背景"]
    A2["理解升级包元数据<br/>版本 / 类型 / 组件 / 标签<br/>支持自然语言查询意图"]
    A3["理解测试人员目标<br/>结合设备能力提示<br/>识别查询、升级、配置等动作"]
    A4["理解日志包与问题描述<br/>识别 stack / OAM / antenna 类型<br/>读取 metadata.json"]
  end

  subgraph R2["推理与执行"]
  direction LR
    B1["自动生成 Release Note<br/>从提交历史组织变更说明<br/>补齐代码库外研发流程信息"]
    B2["RAG 智能检索<br/>Embeddings + FAISS + LLM<br/>相似度搜索 / 搜索建议 / 索引重建"]
    B3["LangGraph ReAct<br/>Plan → Act → Observe<br/>device_prompt 单步护栏<br/>避免一次执行复杂串联操作"]
    B4["LangGraph ReAct 分析<br/>计划 / 检索 / 读取 / 总结<br/>grep / fs / metadata / archive / search"]
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
    D1[("包管理重构 Release Note<br/>变更摘要<br/>影响范围<br/>测试关注点")]
    D2[("目标升级包<br/>推荐版本<br/>匹配原因<br/>可下载资产")]
    D3[("设备动作结果<br/>状态查询<br/>升级回执<br/>配置结果<br/>执行证据")]
    D4[("日志分析报告<br/>关键错误<br/>证据链<br/>根因线索<br/>修复建议")]
  end

end

D1 -. "说明版本变化" .- A2
D2 -. "进入设备测试" .- A3
D3 -. "产生日志证据" .- A4
D4 -. "反馈研发修复" .- A1

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

## 技术架构概览

- **客户端层**：`RavenClient`、Electron、React、MCP Client、包管理工具、DeviceLinkClient。
- **AI 编排层**：基于 LangGraph/LangChain 的 ChatAgent 与 LogAnalysisAgent，接入 OpenAI-compatible LLM。
- **服务层**：`RavenAIService` 中的 FastAPI 服务、Express `package-server`，以及 Electron `update-server`。
- **通信与任务层**：REST API、WebSocket Device Link、Celery、Redis。
- **数据与知识层**：升级包、包元数据、日志包、数据库记录和 FAISS 向量索引。
- **设备工具层**：MCP Server、OAM 接口封装和面向卫星载荷测试的 Python 自动化脚本。

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
