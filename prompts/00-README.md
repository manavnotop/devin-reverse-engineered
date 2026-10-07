# Prompts — Full Verbatim Files

Each file is verbatim `strings -a` output joined with newlines, with source line range + char/word/token counts in its header. Empty `` = runtime-injected tool names. `...Tool` / `{VAR}` = filled at runtime.

## Core systems (pick one per session)

- `01-main-shared-workspace.md` — default REPL lead (~3,374 tok). Permission judgment, autonomy, personality, formatting, skills/MCP/plugins.
- `02-main-terminal-pairing.md` — terminal pairing variant body (~2,264 tok). Parallel tools, search strategy, editing constraints, anti-AI-slop frontend, CLI styling.

## Fusion

- `03-lead-framing-fusion.md` — lead-side Fusion (~4,384 tok, template vars). Delegate-by-default, block:true/false, non-delegable correctness-critical list.
- `04-sidekick-tool-description.md` — `sidekick` tool schema (~370 tok). One persistent sidekick, inject-on-recall.
- `05-sidekick-system.md` — sidekick system, tight 27 lines (~1,394 tok). Investigations, verification, self-check.
- `13a-visual-verification.md` — visual rule (~279 tok). Browser slow, render once + screenshot.

## Compaction / titles

- `06-history-summarizer.md` — history summarizer (~772 tok with neighbors). Overview/Details/State, no-rules-verbatim.
- `07-trace-summarizer.md` — trace summarizer (~476 tok). Same template for execution traces.
- `08-title-generator.md` — one-sentence title prompt (~175 tok).

## Safety / hooks / sandbox

- `09-security-classifier-smart-mode.md` — Smart judge (~497 tok). allow/deny/ask JSON, nonce-marked untrusted data, env-injection watch.
- `10-hook-evaluator.md` — hook eval (~281 tok).
- `11-sandbox.md` — sandbox instructions, trimmed 21 lines (~546 tok). Network/filesystem, request-scope flow, no-bypass spirit.
- `15a-safety-rails.md` — defensive-security-only + destructive-ops zone (~569 tok).
- `15b-style-rails.md` — NEVER-docs/comments zone (~189 tok).

## Modes / subagents

- `12a-mode-planning.md`, `12b-mode-ask.md` — plan/ask gates.
- `12c-mode-btw.md` — earlier 15-line take (includes code tail; superseded by 12d, kept for provenance).
- `12d-mode-btw-full.md` — clean /btw with glued head (~304 tok).
- `12e-subagent-review.md`, `12f-subagent-explore.md`, `12g-subagent-general.md` — review/explore/general profiles.

## Tools

- `14a-parallel-tool-calls.md` — parallel rail zone (~319 tok).
- `14b-tool-schemas-exec-shell.md` — exec/get_output/kill/write/edit schemas + Usage notes (~2,572 tok).
- `14c-tool-registry-keys.md` — StableToolName list zone (~382 tok).
- `14d-tool-rules-apply-mcp-view.md` — ref_file/ref_snippet + MCP + plugins zone (~1,020 tok).
- `14e-skill-prompts.md` — skill invoke/list/search zone (~912 tok).

## Notes / limits

- Some leads are glued to Rust blobs (e.g. trace lead at end of 285596, btw head at end of 285260, title inside 275514) — kept with neighbors for provenance, noted in each header.
- Tool names are interpolated at runtime, so `use `` with grep` shows empty ticks in the binary.
- Totals + style counts in `../12-prompts-analysis.md`. High-level summaries in `../03-prompts.md`.
