# Prompt 04 — Sidekick tool description (lead tool schema)

Source: `bundle/bin/devin` 3000.11.3 via `strings -a` (408525 lines).
Lines 279935..279938 (0-indexed, end-exclusive). Lines: 3. Chars: 1479. Words: 229. Est tokens: ~370 / ~305.
Ends before code at 279938 (slash_commands.rs). Correctness-critical non-delegable list inside.
Joined verbatim with newlines. Empty `` = runtime-injected tool names.

---

Hand off work to your persistent Devin sidekick subagent. There is exactly one sidekick for the whole session: its conversation context and runtime persist across handoffs, and it runs on the same machine (shared filesystem and repos). With `block: true` (the default) the call waits and returns the sidekick's report directly, streaming progress. With `block: false` it returns immediately and the report arrives later in a subagent completion notification; when you run out of parallel work, wait for it with `
` (`block: true`) rather than polling in a loop or inventing the report yourself. Calling this again while the sidekick is running injects the new message into the running handoff as an interrupt rather than starting a second sidekick. The sidekick cannot talk to the user and does not own commits, pushes, pull requests or CI; user-facing and authority actions stay with you.
Correctness-critical authoring is NOT delegable through this tool: any query against a shared/production data system, any prompt/rubric/grader/eval-harness text, and any scoring/threshold/sampling/pipeline configuration must be authored by YOU before the handoff (to a file, or verbatim in the brief). A message that asks the sidekick to write, port, adapt, rewrite, or design any of those artifacts is an INVALID brief. Size is irrelevant: a 20-turn rubric or 200-line query is still yours to write. The sidekick executes lead-authored artifacts and reports results; it never composes them.
