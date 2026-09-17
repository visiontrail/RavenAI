# Agent tool protocol and multi-step execution

Implementation: `RavenClient/third_party/ChatermForRaven`.
Local implementation commit: `afba6d5`; RavenClient integration commit: `2930844e`.

## Incident and cause

The September 17, 2026 log-collection request stopped before executing its first
command. The persisted assistant response contained a complete DSML
`execute_command` invocation, but the parser only understood Chaterm XML. The UI
therefore displayed the invocation as prose. `handleTextBlock` then entered
`completion_result`, so the task waited for another user message without running
the requested operation.

Two related execution-boundary defects were also identified: end-of-stream
finalization promoted unfinished tool blocks to executable blocks, and native
bridge calls opened XML tags before all their arguments arrived, allowing
interleaved calls to become nested XML.

## Execution contract

- `parseAssistantMessageWithProtocol` accepts existing XML and complete DSML
  envelopes for tools/parameters in the shared registry. It preserves literal
  DSML parameter values, rejects unknown/duplicate parameters, and keeps quoted
  DSML examples and XML parameter content as data.
- All accepted calls use the existing dispatcher, workspace restrictions,
  command security checks, user approvals and tool-result history. The format
  adapter does not grant execution permission.
- Native Raven bridge calls are emitted atomically after their arguments are
  received. Invalid JSON and unfinished native calls fail explicitly.
- Incomplete text tool calls never become executable when streaming ends. The
  next model turn receives a format-repair instruction. Repeated protocol errors
  and prose-only execution turns use the existing three-mistake intervention
  limit. Successfully executed earlier calls are not automatically replayed.
- Tasks with a host or database context continue after prose-only responses and
  require `attempt_completion` or `ask_followup_question` to finish or request
  input. Chat mode and hostless explanations retain conversational behavior.
- Multi-step commands remain sequential: each result is stored and returned to
  the model before it selects the next operation. A delayed partial preview must
  not advance past a completed call that arrived while the preview was pending.

## Validation (2026-09-17)

Automated validation covers fragmented envelopes, opaque shell/XML/JSON values,
invalid/duplicate/unknown fields, native-call interleaving, truncated streams,
bounded repair, chat-mode behavior, workspace policies and delayed presentation.
The integration cases run the real Raven bridge and Task loop with a controlled
model transport. XML, DSML and native events each drive real local shell commands
to discover a log, create an archive and verify its contents. Transport and UI/DB
boundaries are fixtures; these tests do not claim a real model or SSH connection.

Live RavenClient acceptance used the configured model and an existing device
connection. Task `42be811e-47fe-4afc-a240-43ed0fcf89fd` performed three separate
read-only commands: service state/PID lookup, process lookup using the returned
PID, and the latest five service log records. The first response had a mismatched
XML closing tag; the agent corrected it without executing the incomplete call.
Later turns switched to DSML, including a todo update and command in one
envelope. Persisted tool results show all three commands executed once, and the
UI displayed 3/3 complete with an explicit completion response.

On the final rebuilt runtime, task `17da5287-936a-4c8f-9313-d24af4b028a8`
repeated the original log-collection operation using the configured model. Five
command results cover discovery (including recovery from garbled terminal
output), staging, archiving and verification. The resulting device file is
`/tmp/frr-logs-20260917-164443.tar.gz`, 6,643 bytes. The successful `tar -tzvf`
result lists service journal output (26,550 bytes), daemon journal matches
(33,650 bytes), syslog matches (31,523 bytes), and the existing reload log
(4,059 bytes). The journal export covers retained logs; this does not establish
availability of historical records outside the device's retention window.
Malformed completion frames were corrected automatically before the UI showed
the archive path and completion. No service restart or configuration command
appears in this task's command history. Temporary staging files remain on the
device alongside the archive.

Final scoped result: **19 test files, 233 tests passed, none skipped**. Node and
renderer type checks, changed-file lint/format checks, and the embedded build
passed. The local RavenClient was restarted with the rebuilt embedded bundle;
no RavenAIService deployment was required.

Commands used for the scoped checks:

```sh
# From RavenClient/third_party/ChatermForRaven:
npm run typecheck
node_modules/.bin/vitest run --project=main-process --coverage.enabled=false \
  src/main/agent/core/task/__tests__ src/main/agent/core/assistant-message \
  src/main/agent/api/__tests__
# From RavenClient:
node scripts/build-chaterm.js --skip-install
```

ESLint and Prettier checks are scoped to the changed files. The full repository
test suite is outside this validation scope.
