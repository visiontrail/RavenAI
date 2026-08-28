## Context

RavenAI resolves a primary and optional backup endpoint before starting a Claude Agent SDK `query()` generator. The selected API key is copied into the SDK subprocess environment and remains fixed for that generator's multi-turn tool loop. The router may abandon an endpoint only before genuine model output; after text, reasoning, or a tool call is committed, replay would duplicate side effects and token spend.

The primary Yinhe/OneAPI endpoint now has 15 independent RavenAI keys. The DeepSeek backup must remain a single-key endpoint. Backend and Celery workers share Redis, while runtime model settings are already persisted in the shared app-data volume and overlaid on `.env` without restart.

## Goals / Non-Goals

**Goals:**

- Store and validate a primary key pool without exposing its values through Admin APIs, logs, traces, or browser state.
- Spread concurrent Agent runs evenly across the pool across all RavenAI processes.
- Recover from a key-local pre-output HTTP 429 by trying one different primary key before the paid backup.
- Preserve single-key deployments and the existing DeepSeek failover semantics.
- Make the configured pool size and live distribution observable and testable.

**Non-Goals:**

- Rotating a key after an Agent run has committed model/tool output.
- Changing OneAPI rate-limit policy or assuming that multiple keys bypass an account-global limit without runtime evidence.
- Creating a DeepSeek key pool, changing provider/model selection, or deploying to production.
- Adding a reverse proxy between the Claude Agent SDK and upstream providers.

## Decisions

### Add a dedicated secret-list setting with legacy fallback

`anthropic_api_keys` is a list of primary secrets in the same runtime-settings overlay as the existing model fields. An Admin save normalizes whitespace, rejects duplicates, enforces a bounded pool size, and treats an empty submission as "keep current". API descriptions expose only `is_set` and `count`. If the list is empty, routing falls back to `anthropic_api_key`, preserving existing `.env` and stored configurations. The backup continues to use only `anthropic_backup_api_key`.

Alternative considered: reinterpret the existing single field as comma-separated text. Rejected because it makes legacy secrets ambiguous, weakens request typing, and encourages accidental logging of the combined string.

### Select one key per SDK run with a shared round-robin cursor

Primary endpoint resolution obtains the next key index from Redis `INCR` modulo pool size. Redis failure degrades to a locked per-process counter, matching the router's existing availability policy. `EndpointChoice` carries the selected index, pool size, and a short SHA-256 identifier; only the opaque identifier may appear in diagnostics.

Alternative considered: rotate on every upstream HTTP request. Rejected because the SDK owns those requests inside a subprocess and a local reverse proxy would introduce a new streaming/authentication boundary. Binding one key per SDK run directly addresses concurrent-run RPM amplification without weakening tool-loop safety.

### Retry one alternate primary key only for a pre-output 429

When `routed_query` receives a classified `rate_limit` error before commit, it inserts one alternate primary-key choice ahead of the backup. The key-local 429 does not immediately count as a provider-wide hard failure. Any other pre-output error follows existing endpoint failover. Once committed, all errors remain terminal and no key retry occurs.

Alternative considered: enumerate and try all pool keys. Rejected because an account-wide rate limit would amplify traffic across every credential. One bounded retry handles a stale/busy key while keeping failure pressure controlled.

### Test and observe without revealing secrets

Admin state and router health include primary `key_count`. The primary connection test can test all newly supplied or saved pool entries concurrently and returns aggregate success/failure counts plus opaque per-key IDs; it never echoes secret strings. Normal route logs record slot, provider, model, key ID, and pool size.

### Validate at the real local runtime boundary

Acceptance uses the configured 15-key pool in local Docker. A burst of concurrent, short Agent requests must succeed through the served UI/API. Backend logs must show multiple opaque key IDs, and the signed-in OneAPI Usage Logs must show calls attributed to multiple `RavenAI-Offical-Multi*` names. The test records the burst size, success count, 429/timeout count, and distinct keys observed.

## Risks / Trade-offs

- [OneAPI may enforce RPM per account/group rather than per key] → Keep 429 recovery bounded and use live OneAPI usage/error evidence; report the upstream limit boundary truthfully if multiple keys do not create independent RPM budgets.
- [A single long Agent run can still exceed one key's RPM after commit] → Preserve safety rather than replay tools; distribute concurrent runs and surface any residual mid-run 429 as an explicit limitation.
- [Redis is unavailable] → Use the existing fail-open local counter; distribution becomes per-process but requests still run.
- [Pool editing can accidentally drop keys] → Blank input keeps the saved pool, duplicates are rejected, and the UI shows saved key count without returning values.
- [429 on one key could trip the provider breaker] → Do not record a provider-wide hard failure when an alternate key retry is available; only exhausted/unrecoverable failures feed existing breaker logic.

## Migration Plan

1. Add the optional field and backward-compatible resolver; existing single-key deployments continue unchanged.
2. Deploy code and configure the 15-key primary pool through the Admin API/UI; keep the DeepSeek backup field untouched.
3. Run connection-pool validation, focused tests, Docker health checks, and the high-RPM acceptance burst.
4. Roll back by clearing the runtime override or reverting the commit; `anthropic_api_key` remains a valid fallback throughout.

## Open Questions

None for implementation. Whether Galaxy OneAPI applies RPM per key or per account remains an external runtime property and is part of acceptance evidence rather than an assumed contract.
