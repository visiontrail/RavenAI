## ADDED Requirements

### Requirement: Configure a secret primary API-key pool
The system SHALL accept a bounded list of unique primary Anthropic-compatible API keys, persist it in runtime model settings, and keep the existing single primary API key as a fallback when no pool is configured. The backup endpoint MUST continue to accept exactly one API key.

#### Scenario: Save multiple primary keys
- **WHEN** an administrator saves two or more valid, unique primary API keys
- **THEN** the runtime configuration reports the saved pool count and new Agent runs can use the pool without a service restart

#### Scenario: Preserve existing pool on blank input
- **WHEN** an administrator saves the model form without entering replacement primary keys
- **THEN** the previously configured key pool remains unchanged

#### Scenario: Reject duplicate primary keys
- **WHEN** an administrator submits duplicate values in the primary key pool
- **THEN** the API rejects the update and persists none of the submitted pool changes

#### Scenario: Fall back to the legacy key
- **WHEN** no primary key pool is configured but `anthropic_api_key` is set
- **THEN** Agent routing continues with that single key exactly as before

### Requirement: Distribute new Agent runs across primary keys
The system SHALL select primary keys with a round-robin cursor shared through Redis and SHALL degrade to a process-local cursor when Redis is unavailable.

#### Scenario: Shared round-robin distribution
- **WHEN** consecutive Agent runs resolve the same primary endpoint with a three-key pool
- **THEN** the selected key identifiers cycle across all three keys before repeating

#### Scenario: Redis outage fallback
- **WHEN** Redis is unavailable while a multi-key pool is configured
- **THEN** Agent runs remain usable and each process rotates through its local key pool without exposing secret values

### Requirement: Recover from a key-local pre-output rate limit
The system SHALL retry at most one different primary key when a selected primary key returns HTTP 429 before any genuine model output, and SHALL place that retry before the single-key backup endpoint.

#### Scenario: Pre-output 429 rotates the primary key
- **WHEN** the selected primary key returns a classified HTTP 429 before text, reasoning, or tool output is committed
- **THEN** the run retries with a different primary key and does not immediately mark the whole primary provider unhealthy

#### Scenario: Alternate key also fails
- **WHEN** the bounded alternate primary-key attempt fails before output
- **THEN** the existing backup/final-error routing semantics apply without trying every configured key

#### Scenario: Post-commit 429 is not replayed
- **WHEN** a run has already committed genuine model or tool output and then encounters HTTP 429
- **THEN** the error is surfaced without retrying another key or replaying the run

### Requirement: Keep key-pool observability secret-safe
The system MUST NOT return or log primary key values and SHALL expose only pool counts and opaque non-reversible key identifiers for routing and test evidence.

#### Scenario: Read Admin model settings
- **WHEN** an authenticated administrator reads model settings after saving a key pool
- **THEN** the response contains `is_set` and `count` metadata but no API key values

#### Scenario: Inspect route diagnostics
- **WHEN** multiple Agent runs are served by different primary keys
- **THEN** router diagnostics distinguish the opaque key identifiers without containing any configured secret substring

### Requirement: Validate the configured pool end to end
The system SHALL support a primary pool connection test and SHALL be accepted only after a local-Docker high-RPM burst succeeds through the served RavenAI runtime with Browser or Computer evidence.

#### Scenario: Test all configured primary keys
- **WHEN** an administrator tests a configured primary key pool
- **THEN** the result reports total, healthy, and failed key counts plus secret-safe per-key outcomes

#### Scenario: High-RPM local acceptance
- **WHEN** concurrent short Agent requests exceed the previous single-key RPM envelope in local Docker
- **THEN** the requests complete without RPM-induced timeout, multiple configured key identifiers are observed, and OneAPI Usage Logs attribute calls to multiple RavenAI multi-key names
