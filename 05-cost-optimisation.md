# 05 — Cost Optimisation

`--help` has no cost flag. Optimisation = Fusion (lead/sidekick split) + prompt cache + fast-model judge + local subagents.

## 1. Fusion pricing (the main lever)

Structs verbatim from strings:
```
struct FusionPricing / struct FusionUsage / struct TokenPrices{input,output,cacheRead,cacheWrite}
FusionPricing{leadModelUid,sidekickModelUid,leadPricesUsdPerMillion,sidekickPricesUsdPerMillion}
TokenUsage{inputTokens,cacheReadTokens,cacheWriteTokens,outputTokens}
RequestUsage{requestId,createdAt,responseDimensions,tokenCostUsd,fusionUsage}
```

- UX: `Estimated Fusion savings: `, `Fusion Savings`, `Estimated by sidekick tokens at lead prices` — i.e. counterfactual: what sidekick tokens *would* have cost at lead prices.
- Docs: `fusion.mdx`: `Fusion pairs a frontier lead model with a cost-efficient sidekick, delivering frontier intelligence at a lower cost.` `models.mdx`: `For most users, we recommend Fusion — it delivers frontier intelligence for cheaper.`
- Mechanism: lead (frontier, expensive, e.g. `opus`) plans/reviews; sidekick (cheap, e.g. `swe-1-6-fast`, `decart-swe-1.7`, `kimi-k2p6`, `glm-5-3-low`) explores/implements/verifies. Full detail in `10-fusion.md`.
- Models enumerated: `swe-1-6-fast (default), swe-1-6, swe-2, swe-1.7, decart-swe-1.7, compute-swe-1.7`, `claude-opus-4-7-medium, claude-sonnet-4, claude-haiku-4-5`, `gpt-5-2025-08-07, gpt-5-nano, gpt-5.2/5.3-codex/5.4/5.4-mini/nano/5.5/5.6-sol/terra/luna, gpt-6-astra, gpt-daybreak-*`, `gemini-2.5-flash/lite, gemini-3-flash-preview(+minimal/low/medium/high), gemini-3.1-flash-lite-preview`, `kimi-k2p6/k2p7-code/k3-rl, glm-5-3-low/high/max, routers/swe-1p6-cognition|devin, pigeon-v4/v6-devin`, `accounts/fireworks/models/llama-v3p3-70b-instruct`, `model_uid{label,description,max_context_tokens,max_output_tokens,cost_tier,cost_summary,family_uid,slug,aliases,variants}`.

## 2. Billing

- `CognitionUsageUpdateMeta{cognition.ai/inputTokens,outputTokens,cachedReadTokens,cachedWriteTokens,totalCreditCost,totalAcuCost}`
- `TurnStatsPayload{...,creditCost,acuCost}`, `ACUs / ACUs consumed / ACU limit / No credits or ACUs consumed yet / /session-stats / https://app.devin.ai/plans`
- `PerformanceMetrics{ttft_ms,total_time_ms,input_tokens,output_tokens,cache_read_tokens,cache_creation_tokens,tpot_ms,tokens_per_sec}`
- OTEL: `devin.token.usage{model,type=input|output|cacheRead|cacheCreation}`, `devin.session.count`, `devin.code_edit_tool.decision`, `devin.lines_of_code.count`, `devin.commit.count`, `devin.pull_request.count`, `devin.active_time.total`, events `api_request{input_tokens,output_tokens,cache_read_tokens,cache_creation_tokens}`, `user_prompt{prompt_length}`, `assistant_response{response_length}`, `tool_result{tool_input_size_bytes}`.

## 3. Smart mode as cost saver

- `smart` uses a fast model to judge safety: `a fast model judges whether the action is safe... If clearly safe, runs without prompt; else prompt.` Saves *human* time (no prompt round-trip) and avoids expensive lead re-reasoning after interrupt. Never-auto list (installs, mutating git, `rm/sudo`, `kubectl delete`, cloud destructive, dotenv/keys) bounds risk.
- Fail-open to prompt: `Smart classifier timed out/interrupted; falling back to prompt`. Telemetry: `SMART_DECISION/guardcache/classifier/stage/abstain`, `AGENT_TOOL_CALLED`, `PERMISSION_PROMPT_RESOLVED`.

## 4. Cache economics

- Server-side prompt cache; CLI tracks `cacheRead/cacheWrite/cache_creation`. Cache reads are ~10x cheaper than inputs (standard Anthropic/OpenAI pricing reflected in `TokenPrices`).
- Prefix stability is a cost feature: `prompt_cache_key_base`, `prompt_cache_footer`, `agent_context{disable_prompt_cache_writes,prefix_mismatch_behavior,...}`. See `06-token-optimisation.md`.
- `disable_prompt_cache_writes` exists for one-shots where caching would waste write cost.

## 5. What is NOT cost-optimised client-side

- No local tokenizer/`tiktoken` (0 hits) — no client token counting to skip requests.
- No request batching/dedup; no speculative decoding; no local distillation.
- Subagents cost *more* (independent context+inference, parent-model multiplier for `general`), used for isolation not savings — except `sidekick` (cheap model) and `Fast Context`/`restricted_exec` (single cheap call).
