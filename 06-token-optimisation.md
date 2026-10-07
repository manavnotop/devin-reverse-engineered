# 06 — Token Optimisation

## 1. Prompt caching (server-side, CLI preserves prefix)

- No `cache_control`/`ephemeral` strings (48 cache hits are only `cacheRead/cacheWrite/cache_creation`). No TTL strings. Caching is implemented server-side (Cognition API / Anthropic/OpenAI compatible).
- CLI role: keep prefix stable so cache hits.
  - `InferenceRequest{model,messages,tools,completion_config{temperature,top_p,top_k,max_tokens},..., agent_context{disable_prompt_cache_writes,prefix_mismatch_behavior,preserved_thinking,system_prefix_len}, prompt_cache_key_base, prompt_cache_footer (barista/prompt_cache_footer)}`
  - `ContextSection{kind,tokens} / ContextUsage{modelId,totalTokens,maxTokens,sections}`
  - `Switching to one-shot response mode. Tools attached only to preserve the prompt cache; do NOT call tools...` — tools kept attached even when unneeded to avoid prefix shift.
  - Dynamic content (timestamps, session IDs) goes to footer; stable system/tools/history stay at head.
  - `Adaptive`: 1 hit (dictionary word), no client router — routing via `AssignModel` RPC keeps model-specific prefixes server-side.

## 2. Tool-result shaping

- `Truncatable scrollback`, `output_trunc`, `Some content was redacted or truncated before upload`.
- `exec`: one command per call, no `echo "===="` chaining, `--body-file` for large bodies — keeps results parseable and short.
- `apply_patch` FREEFORM (no JSON wrapper), `+` prefix even for new files — avoids escaping overhead.
- `read` requires offset/limit via `readfile{file,start_line,end_line}` in `restricted_exec`; `grep`/`glob`/`tree{levels}` bound result size.
- `view_image is only for image files`; `ImageData{width,height,base64_data}`, `QuoteScreenshot{url,filename}` — images passed as metadata + base64, not re-rendered.
- `Fast Context`/`restricted_exec`: `Must use SINGLE restricted_exec call at most %d commands each {type:rg|readfile|tree}` — one round-trip instead of N.

## 3. Lazy / dynamic context

- `cli-agents-md-lazy-walk` — walk `AGENTS.md` hierarchy lazily instead of loading all.
- `tool_search` — load tool schemas on demand (reduces tool-set size pressure).
- `skill_search/discover`, `mcp_list_tools` before `mcp_call_tool` — fetch only needed schemas.
- Rules triggers: `always_on/manual/model_decision/agent/glob` — `model_decision` loads only when model asks.
- `instant_context` vs `fast_context` — single-shot vs multi-turn subagent retrieval.

## 4. Compaction interaction

- See `04-compaction-context-management.md`. Summarizer drops raw messages but preserves `Overview/Details/State` + todos + handoffs. `Do NOT reproduce rules verbatim` — rules re-injected from config (saves summary tokens).
- `spawn/apply/hard` (e.g. `183500,183500,262144`) lets background summary complete before swapping, preserving one last cache hit.

## 5. What is absent

- No client token counter, no `tiktoken`, no per-message `tokens` accounting client-side (only `ContextSection{kind,tokens}` reported by server).
- No embeddings/RAG/index — search is `rg`/glob + subagent synthesis, not vector retrieval.
- No structured-output compression beyond `provide_structured_output` latest-wins.
