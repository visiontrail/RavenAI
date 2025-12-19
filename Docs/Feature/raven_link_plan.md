# RavenClient ↔ RavenAIService Long-Link Feature Plan

Goal: enable RavenClient and RavenAIService to maintain a long-lived WebSocket link so that ChatAgent can push prompts to a chosen client device, get the client-side LLM reply, and keep visibility in a new “设备列表” view.

## Protocol (plain text JSON over WebSocket)
- Endpoint: `ws://<raven_ai_service_host>:8085/ws/device-link`.
- Messages are JSON strings with `type` plus payload:
  - `register` (client → server): `{type, device_id, device_name, client_version, host, models: [name], capabilities?: {...}}`
  - `register_ack` (server → client): `{type:"register_ack", device_id, heartbeat_interval, server_time}`
  - `ping` / `pong`: `{type:"ping"|"pong"}`
  - `prompt` (server → client): `{type:"prompt", request_id, session_id, prompt, system_prompt?, target_device_id, metadata?}`; `session_id` comes from AIChat.
  - `prompt_ack` (client → server, optional): `{type:"prompt_ack", request_id, session_id, topic_id?}`
  - `prompt_result` (client → server): `{type:"prompt_result", request_id, session_id, topic_id, answer, raw_messages?: [...] }`
  - `error`: `{type:"error", request_id?, message}`
- Server keeps `device_id -> connection` with `status`, `last_seen`, `metadata`, and a pending map `request_id -> future`.
- Client keeps `session_id -> topic_id` mapping so all prompts from one AIChat session land in one Topic.

## Tasks & Ready-to-run Prompts

### 1) Shared contracts & config surface
Prompt:
```
Document the WebSocket contract in code comments and add config knobs:
- RavenAIService: expose env var (e.g., DEVICE_LINK_HEARTBEAT_SEC, DEVICE_LINK_TIMEOUT_SEC) in app/config.py.
- RavenClient: add config (main) for RavenAIService base URL/port and device identity (use hostname if unset).
No behavior change yet; only constants/docs to anchor later steps.
```

### 2) RavenAIService backend: device link manager, WebSocket, device list API
Prompt:
```
In RavenAIService, add a device link manager and WebSocket endpoint.
- New models in app/models/device_link.py: DeviceInfo (id,name,host,models,capabilities,last_seen,status), PromptEnvelope (request_id, session_id, prompt, system_prompt?, target_device_id, metadata?).
- New service app/services/device_link_service.py:
  - DeviceLinkManager storing active websockets, metadata, heartbeats, and pending futures keyed by request_id.
  - send_prompt(device_id, payload) -> await prompt_result with timeout; raise clear errors when offline/timeout.
  - update heartbeat and status on connect/close/ping; store last_seen.
- New router app/api/device_link.py:
  - WebSocket route /ws/device-link to accept/parse register/ping/pong/prompt_result, call manager; ensure plain JSON text framing.
  - REST GET /api/v1/device-links returning device list/status, and optional GET /api/v1/device-links/<id>/ping to force status check (optional).
- Wire router in app/main.py; add logger hooks; avoid breaking existing middleware exclusions (RequestLoggingMiddleware).
```

### 3) RavenAIService backend: Chat API + tool hook
Prompt:
```
Allow AIChat to target a device and expose a LangChain tool.
- Update app/models/chat.py ChatRequest with optional target_device_id (string) and target_device_name (string).
- In app/services/ai_chat_service.py, carry target info into agent invocation (state/system_prompt context).
- Implement a LangChain tool in app/agents/tools/device_prompt_tool.py that wraps device_link_service.send_prompt(device_id, session_id, prompt, system_prompt?). Return both answer text and topic_id.
- Refactor ChatAgent (app/agents/chat_agent.py):
  - Build llm = _make_llm().bind_tools([device_prompt_tool]) so model can call the tool.
  - Expand LangGraph to handle tool calls: node call_llm -> tool_router -> call_tools (using ToolNode) -> call_llm until no tool calls; keep streaming compatibility or document if stream is limited to non-tool path.
  - Preserve existing default system prompt; include device context (target_device_id/name) in system message to guide the model.
- Ensure chat/stream endpoints still return session_id/model/messages; include tool result in history so follow-up turns have the client LLM answer.
```

### 4) RavenAIService frontend: device list tab
Prompt:
```
Add a “设备列表” page to monitor connections.
- Router: add /devices route; update AppNavbar.vue to include the new nav item with active state.
- API wrapper: frontend/src/api/deviceLink.ts calling /api/v1/device-links.
- New view frontend/src/views/DeviceList.vue showing name/host/status/last_seen/models, auto-refresh (poll or interval).
- Keep styling aligned with existing AppNavbar/LogList patterns.
```

### 5) RavenAIService frontend: AIChat device mention (@)
Prompt:
```
Enhance AIChat.vue so users can choose a device via '@'.
- Load device list on mount (reuse deviceLink API); keep in local state.
- Add mention UI: when typing '@', show dropdown of devices; selected device sets targetDeviceId/Name shown near the input.
- Include target_device_id/target_device_name in the payload sent to /api/v1/ai-chat/chat/stream.
- Display the chosen device tag in the chat header or near the textarea so it’s visible during the session.
```

### 6) RavenClient main process: WebSocket client & IPC bridge
Prompt:
```
Implement a long-lived WebSocket client in the Electron main process.
- New service e.g., src/main/services/DeviceLinkClient.ts:
  - On app ready, connect to ws://<config-host>:8085/ws/device-link; send register with device id/name/host/models.
  - Heartbeat/auto-reconnect/backoff; log state via loggerService.
  - Handle incoming 'prompt' messages: forward to renderer via IPC (new channel e.g., device-link:prompt). Include request_id/session_id/prompt/system_prompt/target_device_id.
  - Accept results from renderer via IPC (device-link:prompt-result) and send prompt_result back.
- Update packages/shared/IpcChannel.ts, src/main/ipc.ts, src/preload/index.ts to expose the new channels.
  - Device identity: default to hostname + app version + a persisted short random code (generated once and stored in config) to avoid collisions; allow override via configManager (device id/name editable in settings).
```

### 7) RavenClient renderer: process prompts, create topic, call LLM, return result
Prompt:
```
In renderer, add a handler that listens to the IPC prompt event and routes it through existing chat pipelines.
- New service (e.g., src/renderer/src/services/DeviceLinkHandler.ts):
  - For every incoming prompt, create a fresh topic (use getDefaultAssistant/getDefaultTopic); no session/topic reuse across prompts.
  - Create a user Message representing the incoming prompt (role 'user', assistantId=default, topicId from the newly created topic).
  - Dispatch existing sendMessage thunk or ApiService.fetchChatCompletion to generate assistant reply using configured model; ensure blocks/messages persisted so UI refresh shows the history.
  - Once answer is finalized, send IPC reply with {request_id, session_id, topic_id, answer, raw_messages?}.
  - Handle errors by replying with error payload.
- Wire the IPC listener (likely in src/renderer/src/init.ts or a provider) and ensure cleanup on unmount.
```

### 8) Validation & logging
Prompt:
```
Add logging and basic checks:
- Server: log device connect/disconnect, prompt request/response timing, timeouts.
- Client: log reconnect attempts and per-request timings.
- Provide a manual test checklist in docs (README snippet and Chinese): start RavenAIService, start RavenClient, see device online in /devices, send a prompt via ChatAgent tool, see RavenClient topic created and answer returned to AIChat.
```

#### Manual test checklist (README-ready / 手动验证清单)
- English:
  1. Start RavenAIService and confirm it is listening on port 8085.
  2. Start RavenClient; wait for the device link to register.
  3. Open `/devices` in the RavenAIService UI and verify the client shows as online with recent heartbeat.
  4. In AIChat, use the ChatAgent device prompt tool to send a prompt to the target device.
  5. Confirm RavenClient creates a new topic for the prompt and the answer is returned to AIChat.
- 中文：
  1. 启动 RavenAIService（8085 端口），确保服务运行正常。
  2. 启动 RavenClient，等待设备长链接注册完成。
  3. 打开 RavenAIService 前端的 `/devices` 页面，确认客户端在线且心跳时间更新。
  4. 在 AIChat 中通过 ChatAgent 设备工具发送一条 Prompt 到目标设备。
  5. 确认 RavenClient 创建了新的会话 Topic，并将回复回传到 AIChat。

## Notes / Open Questions
- Default model on RavenClient: use existing default assistant/model; keep configurable later.
- Persistence: mapping session_id -> topic_id can live in topic metadata or a small Dexie table; ensure replays stay in one topic during app lifetime.
- Streaming: if tool-enabled streaming complicates ChatAgent.astream, document behavior (e.g., stream only when no tool call).
