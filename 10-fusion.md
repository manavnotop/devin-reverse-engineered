# 10 — Fusion: Complete Analysis

> `Fusion pairs a frontier lead model with a cost-efficient sidekick, delivering frontier intelligence at a lower cost.` — `fusion.mdx:5`
> `For most users, we recommend Fusion — it delivers frontier intelligence for cheaper.` — `models.mdx:29`

## 1. Docs (verbatim, bundle is stub)

`bundle/share/devin/docs/fusion.mdx` (27 lines full):
```mdx
title: "Fusion in Devin CLI"
description: "Fusion pairs a frontier lead model with a cost-efficient sidekick, delivering frontier intelligence at a lower cost in Devin CLI."
## Selecting Fusion
Run `/fusion` during a session to open the Fusion model picker and choose your lead, effort, and sidekick.
You can switch away from Fusion to a specific model at any time with `/model`.
<Fusion /> // import from "/snippets/fusion.mdx" — NOT shipped
```
`adaptive.mdx`: `Adaptive is Cognition's intelligent model router that automatically selects the best AI model for each task.` Select via `/model adaptive`, `--model adaptive`, `"agent":{"model":"adaptive"}`. `<Adaptive />` snippet also not shipped.
`models.mdx`: short names `opus, sonnet, swe, codex, gemini` always resolve to latest; `trying swe, gpt, opus covers vast majority`; `swe (fast) for straightforward edits`; `opus|gpt for multi-file refactors`; default `"agent":{"model":"swe-1-6-fast"}`; `Alt+T` thinking level.

## 2. Binary proof it is local, not just routing

```
chisel-agent/src/local_fusion/sidekick_tool.rs
chisel-agent/src/local_fusion/sidekick_inheritable.rs
chisel-agent/src/local_fusion/mod.rs
chisel-agent/src/local_fusion/guidance.rs
local_fusion/lead_framing
local_fusion/handoff_history + LocalFusionCarrier + devin-rs/convo_history
chisel/fusion_lead_model_uid
fusion lead router '' assigned sidekick '' /exa.api_server_pb.ApiServerService/AssignModel
Resolving model router '' via AssignModel RPC
no model assignment service to assign a sidekick for / ' resolved to no sidekick model
Do not use subagents other than `sidekick` unless user asks
Persistent subagent. Keeps its conversation context across handoffs
Sidekick handoff started / Handed off to sidekick / Handing off to sidekick
Local Fusion: injected mid-handoff brief into running sidekick
Local Fusion: dropped blocking wait could not restore sidekick to background
DEVIN_LOCAL_FUSION_BROAD_EXPLORATION, _NO_FIRST_MESSAGE_REMINDER, _FORCE_FIRST_MESSAGE_REMINDER, _NO_FIRST_EDIT_REMINDER, _FORCE_FIRST_EDIT_REMINDER
SIDEKICK_PREFER_EXEC_TOOL / --sidekick-prefer-exec-tool / DEVIN_SIDEKICK_PREFER_EXEC_TOOL
cli-local-fusion-killswitch (remote-config killswitch)
--harness-lead-only / --harness-sidekick-only / DEVIN_HARNESS_LEAD_ONLY / DEVIN_HARNESS_SIDEKICK_ONLY / HARNESS_LEAD_ONLY
cognition.ai/sidekick, cognition.ai/subagent_context, SidekickMeta, SubagentStarted/Completed
model_mixture, preferred_family_models, show_history_on_continue
cognition.ai/isFusion, isFusionCompatible, isFastSpeedVariantOf, isBaseSpeedVariantOf, pickerLabel, section/group/badge/tooltip
Estimated Fusion savings: / Fusion Savings / Estimated by sidekick tokens at lead prices
--fusion-lead: #4eb6f7; --fusion-sidekick: #90a9bf; --fusion-highlight: #cfefff;
*Fusion / Fusion Fast / Choose a different Fusion pairing: /fusion
/fusion == /model fusion
```

`Adaptive`: 1 hit (dictionary word), no router code — routing is server `AssignModel`.

## 3. Roles

- **Lead (frontier, expensive)**: owns outcome, user-facing, authority actions (commits/pushes/PRs/CI), designs/reviews, never mentions sidekick (`describe all work as own`). Stays in main context.
- **Sidekick (cheap, persistent)**: exactly one per session, shared filesystem/repos, separate shell sessions, own chain head. Does hands-on exploration/implementation/verification. `block:true` waits/streams report; `block:false` returns immediately + completion notification; calling again injects as interrupt (`fold mid-task lead_update in, don't restart`).
- **Not delegable** (sidekick tool desc): `queries against shared/prod data, prompt/rubric/grader/eval-harness, scoring/threshold/sampling/pipeline must be authored by YOU` — correctness-critical authoring stays on lead.
- Lead framing variables: `{SIDEKICK_TOOL} {HANDS_ON_EXPLORATION} {BROWSER_IMPLEMENTATION} {PARALLEL_WORK} {TASK_MANAGEMENT} {VISUAL_VERIFICATION}` — roles templated per task.
- `Do not use subagents other than sidekick unless user asks` — Fusion replaces general subagents.

## 4. Prompts (full in `03-prompts.md`)

- Lead framing (24074-24140): delegate by default, `ONLY implement yourself if {BROWSER_IMPLEMENTATION}`, delegate env setup/repair, blocking dispatch default, `block:false` only with parallel work.
- Sidekick system (offset `61e4f0e`): investigations (`code that can produce symptom != did... verify reachable... Rank verified-reachable above plausible`), verification (`report evidence... don't verify between small edits, verify once after last edit... narrowest command...`), visual (`browser tools are slow... render once... save screenshots...`).
- First-message/edit reminders: `local_fusion/first_message_reminder/first_edit_reminder/<system_note>` (24323) + `DEVIN_LOCAL_FUSION_*_NO/FORCE_*` envs.

## 5. Routing & models

- `Resolving model router '' via AssignModel RPC` — server `AssignModel` maps `fusion` → `(leadUid, sidekickUid)` given `/fusion` picker (lead, effort, sidekick) + `isFusionCompatible`, `isFastSpeedVariantOf/isBaseSpeedVariantOf`.
- `model_mixture, preferred_family_models` — lead can prefer families; `show_history_on_continue` controls reload.
- Enumerated UIDs: `swe-1-6-fast swe-1-6 swe-2 swe-1.7 decart-swe-1.7 compute-swe-1.7`, `claude-opus-4-7-medium claude-sonnet-4 claude-haiku-4-5`, `gpt-5* gpt-6-astra gpt-daybreak-*`, `gemini-2.5-flash/lite gemini-3-flash-preview gemini-3.1-flash-lite-preview`, `kimi-k2p6/k2p7-code/k3-rl glm-5-3-low/high/max routers/swe-1p6-cognition|devin pigeon-v4/v6-devin`.
- Killswitch: `cli-local-fusion-killswitch` (remote config) + `--harness-lead-only/--harness-sidekick-only` debug flags to run one side alone.

## 6. Pricing & savings

```
FusionPricing{leadModelUid,sidekickModelUid,leadPricesUsdPerMillion,sidekickPricesUsdPerMillion}
TokenPrices{input,output,cacheRead,cacheWrite} / TokenUsage{inputTokens,cacheReadTokens,cacheWriteTokens,outputTokens}
RequestUsage{requestId,createdAt,responseDimensions,tokenCostUsd,fusionUsage}
fusionUsageTokenPrices leadModelUid/sidekickModelUid
Estimated Fusion savings: {counterfactual: sidekick tokens at lead prices − actual}
```
- TUI colors: `--fusion-lead: #4eb6f7; --fusion-sidekick: #90a9bf; --fusion-highlight: #cfefff`.
- `/fusion` picker labels: `*Fusion / Fusion Fast / Choose a different Fusion pairing`.

## 7. Compaction & context isolation

- Separate thresholds: `SIDEKICK_COMPACTION_THRESHOLDS` (fallback when unset), `-- Chain heads for persistent subagents (e.g. Local Fusion sidekick)`, SQLite per-`compaction_epoch` chains.
- `handoff_history`, `LocalFusionCarrier`, `devin-rs/convo_history` — handoffs summarized, not full-copied.
- `SIDEKICK_PREFER_EXEC_TOOL` — sidekick prefers `exec` over edit tools (fewer round-trips, cheaper).

## 8. Tradeoffs / failure modes

- Lead must not mention sidekick — attribution confusion vs UX simplicity.
- `Local Fusion: dropped blocking wait could not restore sidekick to background` — blocking dispatch can strand sidekick.
- `no model assignment service... resolved to no sidekick model` — Fusion degrades to lead-only when router unavailable.
- Correctness-critical code stays on lead — sidekick is for hands, not judgment.
- One sidekick only — no fan-out; parallelism via `block:false` + lead doing parallel work.

## 9. Fusion vs Adaptive vs single model

|  | Fusion | Adaptive | Single (`--model opus`) |
|---|---|---|---|
| Selection | `/fusion` picker (lead+effort+sidekick) | `/model adaptive` router per task | explicit UID |
| Execution | lead + persistent sidekick, shared FS | server picks model per request | one model, subagents inherit |
| Cost | sidekick tokens cheap, savings shown | router optimizes per task | pay frontier for all tokens |
| Code | `local_fusion/*` + `sidekick_tool` | `AssignModel` RPC only | `DEVIN_MODEL` passthrough |
