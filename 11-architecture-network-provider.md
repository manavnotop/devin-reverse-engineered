# 11 — Architecture, Network, Provider, Persistence

## 1. Rust crates (from strings paths)

`toolbox/src/...` (tools), `chisel-agent/src/...` (`acp_server/agent_impl`, `local_fusion/*`, `browser_preview/*`, `browser-preview/src/*`), `chisel-server/src/...`, `chisel-commands/src/repl`, `chisel-core/src/{translator,profiles,turn_stats,flags,model_provider}.rs`, `chisel-api/src/...`, `agent-ext/src/{hooks,skills,rules,compactor,smart_permission,lints}`, `affogato/src/cog.rs`, `sandbox-runtime/src/manager.rs+macos.rs+socks_proxy.rs+http_proxy.rs`, `local-agent/src/sandbox_mode.rs+agent_setup.rs:424`, `user-config/src/trusted_workspaces.rs`, `chisel-otel/src/runtime.rs|http.rs|provider.rs|extension.rs|code_activity.rs`, `chisel/smart_permission`.

## 2. Client/server split

- Local: REPL → `devin acp` stdio (ACP JSON-RPC). Tools execute locally. Session SQLite, sandbox seatbelt/bwrap, OTEL.
- Cloud: `devin acp --cloud` relays to cloud ACP WebSocket (`chisel_acp_relay`). `cloud drs ...`, `ssh`, `forward`, `/handoff`, `/pickup`.
- Server: `AssignModel` (fusion/sidekick routing), `GetImageCaption/GetChatMessage`, `SeatManagementService/GetUserStatus`, team settings, usage/billing. CLI passes `model_uid`, `prompt_cache_key_base/footer`, `agent_context`.

## 3. Endpoints & config

```
https://static.devin.ai/cli/current/manifest.json
https://static.devinenterprise.com/cli/current/manifest-enterprise.json
https://static.windsurf.com/cli/current/manifest-windsurfcom.json
https://static.devin.ai/devin-rs/remote (STATIC_BASE_URL)
api_server_url, devin_webapp_host, devin_api_url, API_URL override
DRS_API_KEY/ORG_ID/DEVIN_API_KEY/DEVIN_API_URL
/exa.api_server_pb.ApiServerService/AssignModel|GetImageCaption|GetChatMessage
/exa.seat_management_pb.SeatManagementService/GetUserStatus
cognition.ai/session|chain|compaction|agent_stopped|turn_stats|retryable|sidekick|subagent_context|isFusion
```

## 4. Provider layer

- `model_provider.rs`, `MODEL_ALIAS_*`, `ModelFamilyMetadataValue{compaction_thresholds,aliases,is_model_router}`, `model_uid{label,description,max_context_tokens,max_output_tokens,cost_tier,cost_summary,family_uid,slug,aliases,variants}`.
- `completion_config{temperature,top_p,top_k,max_tokens}`, streaming SSE, `reqwest::retry`, `retryAfterSeconds`, `ContextTooLong/PayloadTooLarge`, `content_filter`, `Refusal` → `--refusal-fallback`.
- No local tokenizer; token/cost accounting from server (`TurnStatsPayload`, `CognitionUsageUpdateMeta`).

## 5. Session store

- SQLite `session_db.rs`: `message_nodes(session_id,compaction_epoch,seq)`, `chain_head`, `turnClientMessageId/turnRequestId`.
- Files: `~/.local/share/devin/summaries/<session>.md`, `~/.config/devin/{config,mcp_config}.json`, `.devin/{config,config.local,mcp_config,mcp_config.local,hooks.v1.json}`, `~/.local/share/devin/cli/_versions/`.
- `/fork /revert /steps /resume /export`, `revert/engine.rs steps.rs`.
- Migration: `cognition/ chisel/ → devin/` + compat symlink.

## 6. Observability

- OTEL (`docs/reference/otel.mdx`): meter `ai.devin.local`, logs `ai.devin.local.events`, `service.name=devin-local`, `opentelemetry_sdk-0.31.0 + otlp-0.31.0 (HTTP protobuf only)`, `OTEL_EXPORTER_OTLP_*, OTEL_LOG_USER_PROMPTS, OTEL_LOG_TOOL_DETAILS, otel{endpoint,logs_endpoint,metrics_endpoint,headers,timeout_ms=10000,log_export_interval_ms=5000,metric_export_interval_ms=60000}`.
- Metrics: `devin.token.usage{model,type}`, `devin.session.count`, `devin.code_edit_tool.decision`, `devin.lines_of_code.count`, `devin.commit.count`, `devin.pull_request.count`, `devin.active_time.total`; events `api_request, user_prompt{prompt_length}, assistant_response{response_length}, tool_result{tool_input_size_bytes}, compaction{prompt.id,summary_length}, AGENT_TOOL_CALLED, SMART_DECISION, PERMISSION_PROMPT_RESOLVED`.
- Sentry: `https://codeium-i5.sentry.io`, `https://a3373c...@o4507463137361920.ingest.us.sentry.io/4510507667685376`, `sentry.rust/0.47.0`.
- `doctor [--json]`: checks custom subagent profiles, otel exporter; exit non-zero on fail.

## 7. Extensibility (pointers)

- Skills: frontmatter `name/description/argument-hint/model/subagent:bool/agent:profile/allowed-tools/permissions{allow,deny,ask}/triggers[user,model]`; `SKILL.md`, `agents/<name>.md`.
- Rules: `AGENTS.md/AGENTS.local.md/AGENT.md/.windsurfrules/CLAUDE.md` + `.devin/rules/*.md`, triggers `always_on/manual/model_decision/agent/glob`.
- Hooks: 8 events `PreToolUse PostToolUse PermissionRequest UserPromptSubmit Stop PostCompaction SessionStart SessionEnd`, matcher regex on `tool_name`, `type:command|prompt`, stdout `{decision:approve|block,reason,hookSpecificOutput:{...}}`, exit `0 ok/2 block`.
- MCP: `mcpServers{command/args/env|url/transport:http|sse/headers/oauthClientId/Secret/Resource/disabled}`, `mcp add/list/get/remove/login/logout/enable/disable`, `prompts→/mcp__srv__prompt`.
- Plugins: `my-plugin/{.devin-plugin/plugin.json(name*),AGENTS.md,rules/,agents/,hooks.json,.mcp.json,skills/*/SKILL.md}`, authority `Enterprise>Org>Repo>User`, namespace `/<plugin>:<skill>`.
- Config: `agent{model,show_history_on_continue,preferred_family_models,model_mixture,compaction_threshold_tokens,codex_tools}`, `permissions{allow,deny,ask}`, `sandbox{allowed_domains,denied_domains,network_mode,excluded}`, `read_config_from{agents_standard,cursor,windsurf,claude,copilot,opencode,zed}`.
