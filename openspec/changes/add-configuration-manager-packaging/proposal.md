## Why

RavenAIService 当前的“检索重构包”对话 Agent 只能完成检索，无法像项目专家、日志分析一样通过后台 Skills 扩展能力，也无法替代 RavenClient 中依赖人工选择的整包打包流程。需要将其升级为“配置管理员”，在保留人类最终决策权的前提下，由 AI 完成组件识别、整包构建、仓库入库和对话内交付。

## What Changes

- 将前端及服务端的“检索重构包”Agent 重命名为“配置管理员”，并接入与其他专业 Agent 一致的后台 Skills 加载机制。
- 新增可扩展的软件升级整包打包 Skill，参考 RavenClient 的项目/组件 JSON 配置模型，并允许后续通过配置增加项目、组件、识别规则和打包规则。
- 支持用户在对话中上传一个或多个组件文件或归档文件；后台安全解压并依据文件名、扩展名和归档内容给出项目与组件的初步识别结果。
- **BREAKING**：整包打包在任何项目、任何组件场景下都必须进入现有反问机制；用户明确确认目标项目以及每个上传文件与组件的映射前，禁止执行打包或上传。
- 确认后构建完整升级包，校验产物并上传到现有重构包仓库。
- 在配置管理员对话中展示构建结果、仓库记录以及可点击的整包下载链接。
- 使用 `Temp/LX10-V1.0.0.3` 样例完成可重复的集成与端到端验证。

## Capabilities

### New Capabilities

- `configuration-manager-agent`: 配置管理员的身份、Skills 动态加载、附件感知与对话路由能力。
- `package-component-classification`: 基于上传文件及归档内容的项目/组件初判，以及覆盖全部项目和组件的强制人类二次确认工作流。
- `full-package-build`: JSON 驱动、可扩展且支持常见压缩格式的软件升级整包构建与产物校验能力。
- `package-repository-delivery`: 整包上传重构包仓库、持久化元数据，并在对话中提供安全下载链接的能力。

### Modified Capabilities

<!-- No existing root specifications are present. -->

## Impact

- RavenAIService：Agent 注册与提示词、Skills 加载器、聊天附件与反问状态、打包工具、重构包仓库服务/API、下载接口及前端对话 UI。
- RavenClient：仅作为现有项目/组件配置和打包行为的参考；本次不移除客户端打包功能。
- 运行环境：压缩/解压工具、打包工作区、上传大小与路径安全策略、仓库存储配置。
- 测试：新增组件识别、确认门禁、打包、仓库交付和浏览器端到端覆盖。
