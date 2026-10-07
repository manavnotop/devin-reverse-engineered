# Prompt 10 — Hook evaluator

Source: `bundle/bin/devin` 3000.11.3 via `strings -a` (408525 lines).
Start line: 275682 (0-indexed in /tmp devin-strings.txt). Lines: 15. Chars: 1123. Words: 131. Est tokens: ~281 (chars/4) / ~174 (words*1.33).

Rust splits paragraphs into separate string entries; joined here with newlines verbatim. Empty `` = runtime-injected tool names. `{VAR}` = template vars filled at runtime.

---

agent-ext/hooksHook command returned no outputFailed to acquire LLM semaphore: Failed to acquire LLM semaphoreYou are evaluating a hook condition for an AI coding agent.
Respond with valid JSON only. Do not include any explanation outside the JSON.
For filter evaluators, respond with one of:
- `true` or `false` (boolean)
- `{"pass": true/false}`
- `{"pass": true/false, "match": {...}}` with additional match criteria
For effects evaluators, respond with:
```json
  "decision": "approve" | "block" | "ask",
  "reason": "optional explanation",
  "add_context": "optional context to add"
agent-ext.hooks_evaluatorNo inference backend available for prompt evaluatortool_nametool_kindtool_use_id hooks for pre_tooltool_result
post_tool hooks for post_tooluser_prompt hooks for user_prompt hooks for pre_stoppost_compaction hooks for post_compactionsession_start hooks for session_startsession_end hooks for session_endpermission_request hooks for permission_requestmid > lencalled `Result::unwrap()` on an `Err` valueinternal error: entered unreachable code: invalid Once state$INPUT$ARGUMENTS
<ACCEPT><REJECT>
completedrejected or blocked
