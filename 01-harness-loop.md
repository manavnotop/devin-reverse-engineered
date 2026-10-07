# 01 — Harness Loop

Source: `chisel-commands/src/repl`, `chisel-agent/src/acp_server/agent_impl.rs:await_resume_turn_complete`, `chisel-core/src/{translator,profiles,turn_stats,flags,model_provider}.rs`, `affogato/src/cog.rs`, `agent-ext/src/{hooks,skills,rules,compactor,smart_permission,lints}`, `toolbox/src/composite.rs`.

## 1. Pipeline

```
SessionStart
  → UserPromptSubmit
  → [ pre_inference → inference (stream) → PreToolUse → permission_check
      → exec → PostToolUse → post_inference ]*
  → Stop (Complete|Cancelled|Interrupted|Error)
  → PostCompaction / SessionEnd → PostAgentIteration (summarizer)
```

- Hook stages verbatim from strings: `core/pre_tool core/post_tool core/pre_stop core/session_start core/permission_request core/pre_inference core/post_inference core/post_agent_iteration core/compaction`.
- Every hook stdin: `session_id` (stable) + `prompt_id` (rotated per user prompt); plus `turnClientMessageId/turnRequestId`, `chain_head`, `message_nodes(session_id,compaction_epoch,seq)`.
- REPL spawns `devin acp` child (inherits `DEVIN_MODEL`/`DEVIN_REFUSAL_FALLBACK`) over ACP JSON-RPC stdio; `--cloud` relays to `chisel_acp_relay`/WebSocket.
- Starting strings: `Starting repl iteration setup`, `Creating new session.../Resumed session`, `Print mode runs a single turn non-interactively`, `No turns completed yet`, `Another turn already running`.

## 2. Turn vs step vs session

| Unit | Definition | Evidence |
|---|---|---|
| Session | SQLite row, `session_id`, survives restarts, `compaction_epoch` increments on compact | `session_db.rs`, `cognition.ai/session\|chain\|compaction\|agent_stopped\|turn_stats` |
| Turn | One user prompt until `Stop` | `await_resume_turn_complete`, `/steps`, `TurnStatsPayload` |
| Step | Single tool call within turn | `/steps List conversation steps (for /fork,/revert)`, `revert/engine.rs steps.rs`, `/fork [step] /revert <step>` rewinds files+conversation |

- `/loop <prompt>` requires clean git, auto-review diff loop. `/btw <prompt>` forks sidechain, boxed, not added to history.
- `/clear(/new) /continue /fork [step] /revert <step> /resume /ls /title /rename-session /rm-session /export /exit`.
- Profiles: `normal`=all tools; `plan`=read-only `grep,glob,read,todo,ask_user_question,exit_plan_mode`; `ask`=same minus `exit_plan_mode`.

## 3. Termination

`StopReason::{Complete Cancelled Interrupted Error}` + `Cancelled/Interrupted/quota_exhausted/tool_rejected/auth_required/max_turn_requests/output_truncated/content_filter/shutdown`.

- Limits: `max_turn_requests / windsurf_max_turn_requests (0=unlimited) / max_tokens`, `agent.compaction_threshold_tokens` + `MODEL_ALIAS_*` cascades, `ContextTooLong/PayloadTooLarge`.
- Stats per turn: `TurnStatsPayload{toolCalls,filesChanged,commandsRun,inputTokens,outputTokens,ttftMs,...,creditCost,acuCost}`.
- Refusal: `--refusal-fallback` (`DEVIN_REFUSAL_FALLBACK` csv, each tried ≤once per session for Anthropic policy/cyber, OpenAI cyber). See `09-why-never-says-no.md`.
- Retry: `cognition.ai/retryable+retryAfterSeconds`, `reqwest::retry`, `Transient team settings fetch failure (attempt N), retrying`, `connection_retry`.

## 4. Parallel tool execution

- Prompt-enforced, no barrier primitive besides subagents: `Call tools in parallel when independent; keep dependent, destructive, or approval-gated commands sequential.` + `core/parallel-tool-calls## Parallel tool calls` + `Batch independent tool calls in parallel; keep dependent or destructive commands sequential.`
- `Don't launch multiple foreground subagents in parallel—use background then foreground`.
- Skill orchestrator (`health-check`→`/research-changes`+`/validate-tests`) is one-level deep; subagent skills run inline when nested.
- Shell: `Set timeout=0 to run long-running commands in background... monitor using get_output... kill with kill_shell.` `a still-running command is backgrounded and shell_id returned`. One-shot (no `shell_id`) shares no state; persistent `shell_session` per `shell_id` preserves cwd.

## 5. Interruption / cancellation

- `global.cancel (Ctrl+C) cannot be unbound`, `(esc again/twice to interrupt)`, `Canceled due to user interrupt`, `Skipped due to interrupt before tool call began`, `User confirmed/edited tool call`, `Timeout interrupted...shell`, `Interrupted. Running in background shell`.
- Subagents **park, not kill**: `Interrupting agent does not kill subagents. Running subagents park intact, resume next message` (`subagents.mdx:167-170`). `Ctrl+B` backgrounds fg subagent, panel `f` foreground/`x` cancel, `Ctrl+C/Esc` cancels fg.
- Sidekick brief injected as interrupt; new user message steers active task unless clearly cancel (`If user sends new message while working, treat as steering...`).
- Compaction: `Compaction cancelled by revert / user / interrupted`, `ForceCompact received while busy→reject`.

## 6. REPL slash surface

Modes: `/mode [normal|accept-edits|smart|plan|bypass]`, `/normal /accept-edits /smart /plan /ask <q> /bypass(/yolo,/dangerous) /autonomous`, `/model /fast(SWE-1.6 Fast) /theme`, `/fusion`, `/compact`, `/context /usage /session-stats(/stats)`, `/workspace /add-dir /undo-add-dir /remove-dir`, `/hooks /mcp`, `/cloud /handoff /repo /platform /open[web|desktop] /ssh /pickup /rename /archive`, `/help /shortcuts /config /bug /update /upgrade /login /logout /login-status /org /copy /feedback /mouse`.
ACP extra: `/ask /plan /compact /fast /loop /btw /session-stats /workspace /mcp /bug` categorized Account/Session/System.

## 7. Constants / tradeoffs

- `max_turn_requests` (0=unlimited) — runaway guard vs long tasks.
- `compaction_thresholds <spawn>,<apply>,<hard>` e.g. `183500,183500,262144` — background summarize vs block inference.
- `timeout=0` background vs foreground — TTFT vs orphan shells.
- Park-not-kill subagents — resume cost vs lost work.
