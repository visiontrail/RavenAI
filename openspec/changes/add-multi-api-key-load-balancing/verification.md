# Local Docker Acceptance Evidence

Date: 2026-08-28 (Asia/Singapore)

Scope: local Docker at `http://127.0.0.1:8085` only. No bastion or production
host was accessed. API key values were entered through Chrome and were never
printed, logged, added to this change, or committed.

## Persisted configuration and served runtime

- Chrome Admin saved the complete primary pool and reloaded it as `15` keys.
- A secret-safe in-container inspection reported:
  - primary provider `yinhe`, key count `15`;
  - backup enabled, provider `deepseek`, key shape `single`.
- Backend, worker, bugfix worker, beat, and frontend containers were healthy.
- Backend/worker/beat image: `sha256:fde381f27eb322225f7817598c02c7b77ee0771d950e5322763ee80f9d9a1400`.
- Frontend image: `sha256:bc885c7b9fc00370d37576febe9f301c6acbf383224950d7c31fe5a6b975e35f`.
- `/health` returned `status=healthy`, `environment=development` after the
  Browser acceptance run.

## High-RPM and Browser acceptance

The acceptance separated request rate from unlimited simultaneous model
concurrency so the test measured the original RPM concern instead of merely
exhausting upstream connection slots.

1. The unsaved 15-key pool was first probed simultaneously. Four calls returned
   `200` and eleven hit the 30-second probe timeout. Backend logs showed only
   four upstream responses, identifying an upstream concurrent-connection
   ceiling rather than invalid credentials.
2. Chrome then probed the same pool in five batches of three. Every batch
   returned `3/3`, proving all `15/15` keys authenticate. The five batches
   completed in roughly one minute, already exceeding the previously observed
   eight-RPM ceiling.
3. A real Browser Agent smoke request completed through GeneralAgent and logged
   `slot=primary`, `pool_size=15`, and an opaque `key_id`.
4. A deliberately aggressive 15-session Browser burst completed all `15/15`
   visible conversations. It also demonstrated that unrestricted multi-tab
   concurrency can exceed the primary model's 20-second first-token deadline
   and exercise the existing DeepSeek fallback. This is a concurrency/TTFT
   boundary, not a 429/RPM failure.
5. A paced real-Agent Browser run sent eight independent conversations at about
   one every ten seconds. Every conversation reached two messages, exposed the
   final Markdown/PDF controls, and showed no visible rate-limit, timeout, or
   terminal error. Backend routing recorded primary pool size `15`; slow primary
   calls were recovered by the configured single-key DeepSeek fallback.

No `HTTP 429` or `rate_limit` marker appeared in the high-RPM Agent windows.
The intentionally aggressive local multi-tab burst did expose SQLite lock noise
and primary TTFT failover, so it is not used as evidence that the application
supports unlimited concurrency.

## OneAPI external evidence

Chrome loaded the signed-in OneAPI common usage log after the run:

- `88` visible successful calls used `RavenAI-Offical-Multi*` keys;
- all `15` distinct multi-key names appeared;
- the maximum rolling 60-second window contained `23` successful calls
  (`2026-08-28 20:37:24` through `20:38:08`), well above eight RPM;
- no key values were read back or recorded in this report.

## Final local state

Test-created circuit-breaker samples and probe locks were removed by exact
Redis key name after evidence capture. The API-key cursor and persistent model
configuration were retained. The final health snapshot reported:

- serving slot `primary`;
- primary breaker closed;
- primary `yinhe`, `15` keys, zero retained bad samples;
- backup `deepseek`, `1` key, zero retained bad samples;
- local `/health` healthy.

Relevant commits before this evidence record:

- RavenAIService: `6b915f73e392fdd03a1ec3937aff3b8702bfba0d`
- umbrella repository: `231025b1bd03200d5830d098e6fe2d927dd2fe8b`
