## 1. Runtime Configuration

- [x] 1.1 Add the secret-list primary key setting with normalization, duplicate/size validation, legacy single-key fallback, and secret-safe Admin metadata
- [x] 1.2 Extend Admin request/response typing, connectivity testing, and environment documentation for the primary key pool while leaving backup single-key

## 2. Routing Behavior

- [x] 2.1 Add Redis-backed round-robin key selection with a process-local fallback and opaque key identifiers
- [x] 2.2 Add one bounded pre-output 429 alternate-key retry before backup while preserving post-commit no-replay semantics
- [x] 2.3 Expose secret-safe key-pool count and route diagnostics

## 3. Admin User Interface

- [x] 3.1 Add a primary multi-key textarea/count state and keep the backup single-key input
- [x] 3.2 Update bilingual guidance and component/API types for pool testing and save semantics

## 4. Verification

- [x] 4.1 Add backend and frontend regression coverage for pool persistence, secrecy, round-robin selection, Redis fallback, and 429 rotation
- [x] 4.2 Run focused and full relevant test/build checks, review the scoped diff, and commit the RavenAIService plus root OpenSpec/gitlink changes
- [x] 4.3 Configure all 15 RavenAI multi keys in local Docker and verify health, persisted runtime state, DeepSeek single-key preservation, and served artifacts
- [x] 4.4 Execute a concurrent high-RPM Agent burst and complete Browser/Computer acceptance with backend and OneAPI usage evidence
