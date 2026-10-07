# Prompt 14e — Skill prompts zone

Source: `bundle/bin/devin` 3000.11.3 via `strings -a` (408525 lines).
Lines 275558..275585 (0-indexed, end-exclusive). Lines: 27. Chars: 3650. Words: 456. Est tokens: ~912 / ~606.
skill_invokestruct / list / search / discover neighbors.
Joined verbatim with newlines.

---

Activate a skill by name (replaces any active skill)Discover skills registered in a project at the given path (scans skill
directories like .devin/skills/ at the project root, does not recurse
into subdirectories)Recursively scan for model-invocable skills under a given path and
filter them by optional keywords
The skill operation to perform. Defaults to "invoke" if omitted.The name of the skill to invoke (required for 'invoke' command).
Use one of the available skills listed in the system prompt.Project path to discover skills in. Scans skill directories (e.g.
.devin/skills/) at the project root; `list` does not recurse into
subdirectories, while `search` does. Relative paths are resolved
against the session's working directory. Provide either `path` or
`source` for list/search, not both.Set to `plugins` to list or search active plugin skills instead of
scanning a project path.List of keywords to search for. Empty or omitted means include all skills.Controls whether all keywords or any keyword need to be present.Input for the legacy multi-command `skill` tool.The name of the skill to invoke.
Use one of the available skills listed in the system prompt. To find a
plugin skill, call `skill_search` with `source: "plugins"`.Project path to discover skills in. Scans skill directories
(e.g. .devin/skills/) at the project root; does not recurse into
subdirectories. Relative paths are resolved against the session's
working directory. Provide either `path` or `source`, not both.Set to `plugins` to list active plugin skills instead of scanning a
project path.
Project path to search skills in. Recursively scans for
model-invocable skills under this path. Relative paths are resolved
against the session's working directory. Provide either `path` or
`source`, not both.Set to `plugins` to search active plugin skills instead of scanning a
project path.Local time out of range for `NaiveDateTime`Activate a skill by name (replaces any active skill). Skills provide context-specific guidance and may grant tool permissions.skill_invokestruct RunHookCommandResponsestruct RunHookCommandRequeststruct SkillInputstruct SkillListInputstruct SkillSearchInputstruct SkillInvokeInputstruct ConvoHistoryCarrierstruct LocalFusionCarrierList skills from exactly one source: provide `path` to scan skill directories at a project root (non-recursive), or `source: "plugins"` to list active plugin skills.skill_listLayoutalignSearch model-invocable skills from exactly one source, filtered by optional keywords: provide `path` to recursively scan a project, or `source: "plugins"` to search active plugin skills. Use plugin search before concluding that a referenced plugin skill does not exist.skill_searcha Display implementation returned an error unexpectedlyErrorMaybeDone polled after value takenInvoke or discover skills. Modes: 'invoke' (default) activates a skill by name; 'list' and 'search' accept exactly one source, either a project `path` or `source: "plugins"` for active plugin skills. 'list' scans only the project root; 'search' recurses and filters by optional keywords. Use plugin search before concluding that a referenced plugin skill does not exist.description() is deprecated; use Display
nonekill
=nulla floatan integera boolean
internal error: entered unreachable codeoverflow deserializing Durationfalse, AgentContextsession_idagent_idagent_depthcustomer_identerprise_idnon_zdr_allowedanthropic_user_profile_idusa_inference_onlyprompt_cache_key_basePerformanceMetricsttft_mstotal_time_msinput_tokensoutput_tokenscache_read_tokenscache_creation_tokenstpot_mstokens_per_seccached_tokens
title
tool_call_idattemptmain_request_idtool_namecommanddecision
