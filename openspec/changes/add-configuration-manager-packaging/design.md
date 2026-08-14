## Context

`package_search` 已有项目绑定的 Claude Agent SDK 循环、包仓库只读 MCP 工具、SSE trace、持久对话工作区和通用 clarification broker，但它不在 `skills_service.SUPPORTED_AGENTS` 中，也没有物化 Skills、`Skill` 工具或 `setting_sources=["project"]`。前端同一 Agent 的名称同时存在“检索重构包”和“重构包配置管理员”，且配置管理员明确禁用了非图片附件。

RavenClient 的当前打包器由人先选项目和组件槽位，再由 `packagingService.ts` 机械处理文件。PacketAttr、FileAttr、目标文件名和识别规则均硬编码，并非外置 JSON；它不支持一源多组件、RAR、安全展开、最终验包或自动仓库上传。`Temp/LX10-V1.0.0.3` 中 13 个非元数据文件还包含一源三组件的协议栈包、已打好的 Patch 包、FileAttr 801 的新 Satellite MCP 组件，以及三类当前 Client manifest 未覆盖的硬件资产，因此不能无确认地静默套用旧规则。

普通 clarification 是模型自愿调用且可被用户关闭，无法表达本需求的安全不变量。现有重构包仓库是 `data/raven/uploads` 加 JSON metadata，写入无锁且非原子；Agent 并发发布需要先补齐一致性保证。

## Goals / Non-Goals

**Goals:**

- 保留内部 `package_search` key，将产品身份统一为“配置管理员”，并使其像项目专家/日志分析一样加载内置、Agent 和项目 Skills。
- 允许一次上传多个任意组件文件；打包请求可不预选项目，AI/Skill 先基于文件证据提出项目及组件候选。
- 对每次打包实施无法被模型或用户偏好绕过的服务端 clarification 门禁，确认项目、包版本/类型以及每个上传文件的组件映射或显式排除。
- 以版本化 JSON 规则完成安全、确定性的分类、解压、`si.ini` 生成、TGZ 制包、验包、原子入库和对话下载交付。
- 用 LX10 样例验证多组件归档、直接包含、未知资产、已打包资产和大批量上传。

**Non-Goals:**

- 本次不删除 RavenClient 的打包入口，也不保证 Client 与服务端在所有历史错误行为上字节级一致。
- 不为缺少业务 FileAttr/目标格式的未知硬件资产猜造发布规则；它们会被识别、展示并要求人明确排除或通过后续 JSON 扩展。
- 不把普通包检索改成全局跨项目搜索；无项目启动只对带组件附件的打包请求开放。
- 不在本次改变现有下载授权策略或全面重做所有包管理写 API 的认证模型。

## Decisions

### 1. 保留 key，重命名身份，并允许无仓库项目

内部继续使用 `package_search`，避免迁移历史会话、项目 Agent 关联、metrics 和 API；注册表、提示词、i18n、状态和完成摘要统一显示“配置管理员”/“Configuration Manager”。`requires_repo` 改为 false，因为当前 workspace 本就不 clone，打包只依赖项目目录、Skill 和包仓库。

普通检索的新会话仍要求项目。若请求包含组件文件，则允许没有 `project_repo_id`：先创建 unbound workspace，并把所有启用且支持 `package_search` 的项目作为受限候选目录；确认后再写入权威 `project_repo_id/project_code`。已有 UI 预选只作为高权重证据，仍必须反问确认。

备选方案是始终要求前端选项目；它实现简单，但让人类在 AI 判断前已经作出项目决定，不满足“项目也要由 AI 初判并二次确认”。

### 2. 三层 Skills 加一个只读内置层

`skills_service` 增加 `package_search`。物化顺序为 built-in → Agent → project，同名后者覆盖前者；overview 和 trace 使用同一合并顺序。内置 `full-package-build` 存在源码树中，确保开箱可用；管理员上传同名 Agent Skill或项目 Skill即可覆盖它的工作流和 JSON catalog。Agent 仿照 Project Expert 增加 `Skill`、Skill availability prompt、`setting_sources=["project"]`、`run_start.loaded_skills` 和 `skills_loaded` notice。

Skill 保持渐进披露：`SKILL.md` 只描述强制流程和专用工具，`references/package-projects.json` 承载 schema version、项目 aliases/命名、PacketAttr、组件 FileAttr、识别证据、提取策略、输出名和版本规则。确定性、安全敏感的文件处理位于服务端工具，不由模型自由编写 shell。

备选方案是首次启动时把 Skill 写入数据卷注册表；这会制造升级、覆盖和只读卷问题，源码内置层更可预测。

### 3. 分类器给建议，服务器拥有最终门禁

上传后先生成 draft plan：每个文件记录稳定 `upload_id`、原名、大小、SHA-256、magic/type、归档成员摘要；分类器按可配置权重评估项目和组件，输出候选、confidence、evidence 和 publishability。一份归档可命中多个组件。已含 `si.ini` 的升级包会识别为 prebuilt，默认不能作为原始组件再次嵌套；没有 FileAttr 的新资产可被识别但不可发布。

服务器随后直接复用 `PermissionBroker`、统一 resolve API 和 `clarification_request/resolved` trace，程序化提出一张多问题卡：目标项目、版本/整包类型，以及 manifest 中每个 `upload_id` 的映射（多选）或“不纳入整包”。该路径无条件启用，不读取 `clarification_enabled`，timeout 固定为 cancel。所有题必答后生成 canonical confirmed plan；plan hash 覆盖 catalog hash、session/user、项目、包元数据、每个文件路径/大小/hash/映射。任何变化使确认失效。

为了保持 trace `seq` 单调，preflight 与后续 Agent `_RunState` 共用同一个 `SeqCounter`。专用 build/publish 工具只接受 workspace 中已确认且 hash 复算一致的 plan。打包模式显式禁用 Bash/Write/Edit/WebFetch 等旁路写工具，包仓库只读 MCP 不变。

备选方案是在 Skill 中写“必须调用 AskUserQuestion”；模型可能跳过，且用户可关普通澄清，不能满足 MUST。

### 4. 二进制附件流式落盘并持久化 manifest

package stream 新增重复 multipart `files` 字段。`UploadFile` 在请求生命周期内以 1 MiB chunk 写入 `<workspace>/package_inputs/`，同时计算 SHA-256；使用清洗后的 collision-safe 存储名，保留原名和序号。实施单文件、总量、文件数、磁盘 reserve 限制，失败清掉本轮部分文件并不启动 job。manifest 通过临时文件 + replace 原子更新；新附件使旧确认失效。

前端使用独立 `selectedPackageFiles`，避免触发日志 metadata 校验或自动路由；picker 支持 multiple 且不设 accept，拖放亦可。乐观用户消息显示全部文件名，发送失败恢复选择。13 个确认问题通过现有 modal 呈现，modal 增加视口内滚动。

备选方案是把文件转 base64 塞进现有 `images` 字段；380 MiB 样例会造成巨大内存放大，且语义错误。

### 5. 复用安全归档后端，构建扁平 TGZ

把日志 workspace 的 ZIP/TAR/7z/RAR 安全能力提取/公开为通用 helper：先验证成员路径、链接、文件数和展开字节，再解压；RAR 沿用 rarfile → unar/lsar → bsdtar fallback。分类阶段优先只列成员，只有 build 阶段才展开，并按 source hash 缓存一次，支持协议栈 ZIP 一次提取生成 CUCP/CUUP/DU。

builder 按 JSON 规则执行 `direct_include`、`copy` 或 `extract_match`，不再采用“找不到名字便取任意同扩展名”。组件按 FileAttr/名称稳定排序，生成与 Client 协议兼容的扁平根目录和 `si.ini`，输出名保留 Client 前缀/版本规则但增加碰撞消解。最终 TGZ 必须重新打开，校验 `si.ini`、FileNum、FileName/FileAttr、非空 payload 和 manifest hash。

### 6. 专用 MCP 是主路径，服务器 fallback 保证交付

`package_builder` in-process MCP 提供只读 plan inspect 和 `build_full_package`。Skill 指示模型在 confirmed plan 存在时调用工具，并在最终答案中说明分类与产物。工具内部复算 confirmation，不允许模型传任意源路径或项目覆盖。

若模型失败、provider 不支持 MCP、或输出契约错误但 confirmed plan 已存在，chat service 在终态前调用同一 deterministic builder 作为一次性 fallback；result 文件和 confirmation id 提供幂等保护，避免重复发布。这样 AI/Skill 仍负责标准主流程，而交付不会因模型漏调工具而丢失。

### 7. 原子入库与模型无关的下载链接

`RavenPackageService` 的 metadata 读改写加进程锁，保存采用同目录临时文件、flush/fsync、`os.replace`。专用 publish 先把验证产物复制到唯一临时名，计算 hash，再原子 rename；metadata 保存失败则删除新文件。记录复用现有 package schema，并在 `metadata.customFields.fullPackageBuild` 保存 manifest/catalog/confirmation 摘要。

build result 原子写入 workspace。Agent result、terminal SSE `done.artifacts` 和持久化 assistant answer 都从这个权威文件补充包名、大小、SHA-256、package id 与 `[下载整包](/raven/api/download/{id})`，不依赖模型记得输出链接。现有 markdown renderer 直接提供可点击下载。

### 8. 样例的默认规则边界

默认 Lingxi-10 catalog 迁移 Client 已知的 301/302/303/307/308/313/315/401/403/404/405/406，并根据样例证据增加 `satellite_mcp_server/801`。协议栈 ZIP 映射三组件；OAM、SCT master、银河核心网、鹏城三包和 Satellite MCP 使用明确规则。现成 Satellite Patch、SCT/BPO SF2 工程和 SCT M3 固件会被识别为 prebuilt/unconfigured，必须逐项确认排除，直到管理员提供真实 FileAttr 与目标格式。BPO master 的 313/315 歧义必须显示证据并由人决定。

## Risks / Trade-offs

- [380 MiB multipart 和归档展开占用磁盘/时间] → 分块复制、总量/展开限制、direct include 不解压、同源只展开一次，并在失败/会话清理时删除临时目录。
- [一个 clarification 卡含十几个问题会过长] → 使用可滚动 modal、紧凑 evidence 描述和逐题必答校验；一次卡完成可避免多轮上限。
- [项目级 catalog 覆盖发生在项目确认之后] → 项目初判只用 built-in/Agent catalog；项目确认后加载项目 Skill，并将其 catalog hash 纳入最终 plan，若规则改变则重新确认。
- [单进程锁不保护多容器共享卷] → 本次实现至少保证当前单进程部署；未来多 worker/shared volume 需文件锁或数据库事务。原子 replace 可防止 JSON 半写。
- [MCP provider 能力不一致] → deterministic server fallback 使用同一 confirmed plan；不得回退到任意 shell 脚本。
- [未知样例资产不能组成业务定义上的“全组件齐套”] → 明确标为不可发布并要求人排除，不伪造 FileAttr；产物“完整”指确认计划完整且结构/manifest 一致，而不是猜齐未知组件。
- [现有公开写 API 仍可由外部客户端调用] → 配置管理员打包 run 禁用旁路写工具并只经专用 publisher；全面 API 鉴权另立兼容性变更。

## Migration Plan

1. 部署 Skills host、内置 Skill、JSON catalog 和只读分类能力，不改变旧 Client。
2. 部署 package stream 多文件与 mandatory preflight；旧的纯检索请求和历史 session 保持兼容。
3. 部署 deterministic builder、原子 repository publisher、structured artifacts 和下载链接。
4. 发布前运行后端/前端回归、LX10 service integration 和 Browser E2E；确认包仓库新增记录与下载 hash。
5. 回滚时可禁用前端附件入口和内置 Skill；旧 `package_search` key、检索 MCP、既有包记录与下载 API 均不变。

## Open Questions

- `sct_sf2`、`bpo_sf2`、`sct_m3` 的正式 FileAttr、目标文件名和升级器格式仍需产品配置后才能发布；默认规则只识别并要求排除。
- 多实例共享存储若成为正式部署方式，应把 JSON metadata 迁移到数据库或增加跨进程文件锁。
