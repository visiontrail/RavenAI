## 1. Configuration Manager and Skills foundation

- [x] 1.1 Register `package_search` as a Skills host, add built-in Skill precedence, and cover materialization/override behavior with tests.
- [x] 1.2 Extend `PackageSearchAgent` with Skill discovery, prompt layering, setting sources, loaded-Skills trace metadata, and packaging-mode side-effect restrictions.
- [x] 1.3 Rename all user-facing package-search identity/status/i18n text to Configuration Manager and allow repository-less projects.

## 2. Full-package Skill and classification

- [x] 2.1 Scaffold and validate the built-in `full-package-build` Skill with concise workflow instructions and a versioned JSON project/component catalog.
- [x] 2.2 Implement catalog validation, archive-member inspection, evidence scoring, one-to-many component classification, version inference, and canonical plan hashing.
- [x] 2.3 Add LX10 fixture rules including Satellite MCP 801, protocol-stack expansion, prebuilt-package detection, ambiguous BPO evidence, and recognition-only unknown assets.

## 3. Upload and mandatory human confirmation

- [x] 3.1 Add repeated multipart component files to the package stream and safely stage them into a hash-bound persistent session manifest with size/disk/rollback limits.
- [x] 3.2 Support unbound packaging workspaces and project inference when no project is preselected, while retaining project-required behavior for pure search.
- [x] 3.3 Implement the programmatic clarification preflight using the existing broker/trace/resolve protocol, requiring project and every file mapping even when ordinary clarification is disabled.
- [x] 3.4 Persist confirmed plans bound to user/session/catalog/input hashes and reject incomplete, timed-out, cancelled, changed, or bypassed confirmations.

## 4. Deterministic build and repository delivery

- [x] 4.1 Implement confirmed-plan-only component materialization using safe ZIP/TAR/7z/RAR workspace backends, direct include, extract-match, one-source-many-components, and cleanup.
- [x] 4.2 Generate and reopen-validate a collision-safe flat TGZ, `si.ini`, component/input manifest, versions, attributes, and hashes.
- [x] 4.3 Add package-builder MCP tools plus an idempotent server fallback, and expose the tool workflow through the built-in Skill.
- [x] 4.4 Make repository metadata publication locked and atomic with rollback, then surface structured artifacts and a deterministic conversation download link.

## 5. Conversation UI

- [x] 5.1 Add an independent multi-file Configuration Manager attachment picker/drop flow, FormData/store propagation, optimistic filename display, and send failure recovery.
- [x] 5.2 Make large multi-question clarification cards scrollable, expose loaded Skills from trace metadata, and clarify that packaging confirmation overrides the ordinary clarification preference.
- [x] 5.3 Extend completion-summary/download rendering tests and make project selection optional only for attachment-bearing packaging turns.

## 6. Verification and delivery

- [x] 6.1 Add backend unit/integration coverage for Skills, catalog/classification, staging, hard-gate bypass prevention, build validation, atomic publication, artifacts, and download.
- [x] 6.2 Add frontend type/unit coverage for names, repeated files, clarification completeness, loaded Skills, and download links.
- [x] 6.3 Build a complete LX10 artifact from `Temp/LX10-V1.0.0.3`, verify its contents/hash/repository record, and run Browser/Computer end-to-end upload-confirm-download validation.
- [x] 6.4 Run focused and regression test suites, update affected agent documentation, review the final diff, and commit only this change.
