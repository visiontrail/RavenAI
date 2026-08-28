## Why

RavenAI currently binds every primary-provider Agent run to one API key, so concurrent runs can exhaust that key's RPM budget and turn otherwise healthy OneAPI traffic into repeated reconnects and timeouts. Fifteen independent RavenAI OneAPI keys are now available and need to be consumed as one managed pool while the DeepSeek backup remains a single-key failover endpoint.

## What Changes

- Add a secret primary API-key pool to runtime model settings while preserving the existing single-key setting as a backward-compatible fallback.
- Distribute new primary Agent runs across the configured keys with a shared round-robin cursor so backend and Celery processes make one coordinated choice.
- If a primary key returns HTTP 429 before any genuine model output, retry once with another primary key before considering the single-key DeepSeek backup.
- Keep the existing safety boundary: never replay a run after text, reasoning, or a tool call has been committed.
- Expose only key counts and opaque key identifiers in Admin/API/log observability; never return or log secret values.
- Extend the Admin model-settings UI and connection test to accept multiple primary keys while keeping the backup API-key field single-value.
- Add regression, Docker, high-RPM, and Browser/Computer acceptance coverage.

## Capabilities

### New Capabilities

- `multi-api-key-routing`: Configure, validate, observe, and dynamically distribute primary model traffic across multiple API keys with bounded pre-output 429 recovery.

### Modified Capabilities

None.

## Impact

- Backend configuration, runtime settings schema, Admin model-settings API, model router, routed SDK query wrapper, and connection testing.
- Admin Vue model-settings types, endpoint form, translations, and component tests.
- Existing primary/backup routing tests plus new multi-key and 429-rotation regression tests.
- Local Docker runtime configuration and OneAPI usage; no production deployment or DeepSeek key-pool change.
