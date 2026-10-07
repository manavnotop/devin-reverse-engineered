# 02 — Tools: Full Inventory

Evidence: `grep -a -o toolbox/src/tools/...` (32 hits), `docs/extensibility/hooks/lifecycle-hooks.mdx:349-367` matcher table, `StableToolName`, `harnessToolKind: delete/move/search/execute/think/fetch/other`, strings tool descriptions.

## 1. Source files

```
toolbox/src/tools/read.rs write.rs edit.rs apply_patch.rs grep.rs glob_tool.rs
  exec_tool.rs exec/{raw_exec,pty,shell_session_manager,session_manager,shell_processor,login_shell_env,native_terminal_host,background_emitter}.rs
  exec_output.rs(get_output) kill_shell.rs write_to_process.rs
  webfetch.rs web_search.rs view_image.rs notebook_read.rs notebook_edit.rs
  todo.rs exit_plan_mode.rs write_plan.rs request_scope.rs tool_search.rs
  fast_context.rs instant_context.rs readonly_path_extractor.rs powershell_parser.rs
  subagent.rs persistent_subagent.rs user_question.rs
  mcp/{mod,service,registry,config,oauth,sse_transport,list_servers,list_tools,call_tool,read_resource,prompts,stderr_log}.rs
```

## 2. Canonical names

- File: `read write edit apply_patch notebook_read notebook_edit`
- Search: `grep glob` (+ `code_search`/`Fast Context` subagent wrapper)
- Shell: `exec get_output write_to_process kill_shell`
- Web: `webfetch` (+ `web_search`, `browser_preview`/`close_browser_preview`, `view_image`)
- Plan/tasks: `todo_write write_plan exit_plan_mode`
- Skills: `skill` (`skill_invoke`, `skill_search/discover`, `list/search/invoke`)
- Subagents: `run_subagent read_subagent` (+ `sidekick` persistent)
- Perms: `request_scope`
- MCP: `mcp_list_servers mcp_list_tools mcp_call_tool mcp_read_resource` (+ namespaced `mcp__<server>__<tool>`)
- Misc: `ask_user_question tool_search instant_context fast_context`
- ACP caps: `browser_preview, browser_preview_open, clipboard_write`
- ACP `restricted_exec`: `Execute restricted commands (rg,readfile,tree,ls,glob) in parallel.` with `command1/2/3 oneOf rg{pattern,path,include,exclude} readfile{file,start_line,end_line} tree{path,levels} ls{path,long_format,all} glob{pattern,path,type_filter}` (line 30113, len 13724).

## 3. Descriptions (verbatim)

- Parallel: `Use only the available tools and their supplied schemas. Call tools in parallel when independent; keep dependent, destructive, or approval-gated commands sequential.`
- `exec`: `IMPORTANT: This tool is for terminal operations like git,npm,docker, etc. DO NOT use for file operations - use specialized tools... Prefer one command per call...`
- `get_output/kill`: `IMPORTANT: This command will NOT work if no command running.`
- `apply_patch`: `Use apply_patch to edit files. FREEFORM emit patch directly, do not wrap JSON. Must include Add/Update/Delete header. Prefix new lines + even when creating. Delete via *** Delete File: <path>. Move via *** Move to: <new path> after *** Update File... MUST always absolute path...` + `ALWAYS prefer editing existing. NEVER write new files unless required.`
- `find_file_by_name`: `Fast file pattern matching... brace expansion **/*.{ts,tsx}... NOT for contents...`
- `code_search/Fast Context`: `A search subagent user calls Fast Context ideal for exploring... runs parallel grep/readfile over turns... Fill details smart model can infer... Always use to start search... may make mistakes... IMPORTANT: YOU CANNOT CALL THIS TOOL IN PARALLEL.`
- `skill_invoke/list/search`: `Activate skill by name (replaces active). May grant perms... List from exactly one source: path (non-recursive) or source:plugins... Search recurses/filter keywords...`
- `mcp`: `IMPORTANT: You MUST call mcp_list_tools before mcp_call_tool...Never guess...`
- `read`: `If existing file, MUST use read first...`
- `provide_structured_output`: `You MUST call before finishing; latest wins...`
- `sidekick`: `Hand off work to your persistent Devin sidekick... exactly one sidekick... block:true waits/returns report streaming; block:false returns immediately... Calling again injects into running handoff... sidekick cannot talk to user, own commits/pushes/PRs/CI... Correctness-critical authoring NOT delegable: queries against shared/prod data, prompt/rubric/grader/eval-harness, scoring/threshold/sampling/pipeline must be authored by YOU...`
- `todo_write`: `for multi-step work, use todo_write to plan/track`
- `webfetch`: `Fetches web page as readable text`; `view_image is only for image files`
- `exit_plan_mode`: `Do not begin implementing until user approves plan via exit_plan_mode`
- Expert wrapper (38422-38465): `You are expert software engineer, responsible for providing context to another engineer... provide file paths + line ranges with ALL relevant info... IMPORTANT: relevant != only to-modify, include definitions... include ENTIRE semantic blocks (functions/classes)... - Tool access: use restricted_exec ONLY - Must use SINGLE restricted_exec call at most %d commands each {type:rg|readfile|tree}...`

## 4. Families & permission mapping

| Family | Tools | Normal | Smart | Bypass |
|---|---|---|---|---|
| Read/search | read grep glob notebook_read | Auto | Auto | Auto |
| Web fetch | webfetch web_search browser_preview | Prompt | Judge | Auto |
| Shell | exec get_output write_to_process kill_shell | Prompt | Judge | Auto |
| Edit | edit write apply_patch notebook_edit | Prompt (Auto in workspace for Accept-Edits+) | Auto workspace | Auto |

Tool-based perms: `read edit grep glob exec` + `mcp__*`. Scope-based: `Read(glob) Write(glob) Exec(prefix word-boundary) Fetch(URLPattern/domain:)`.

## 5. Design notes

- `apply_patch` FREEFORM (not JSON) — saves tokens, avoids escaping; `Add/Update/Delete` headers, `*** Move to:` for renames.
- `exec` is terminal-only by prompt; file ops redirected to specialized tools to avoid `cat/sed/awk` token spill.
- `get_output`/`kill_shell`/`write_to_process` = background/interactive shell trio; `timeout=0` backgrounds.
- `tool_search` + `instant_context`/`fast_context` = dynamic/lazy loading to keep tool-set small (selection pressure).
- `request_scope` lets sandbox sessions request `Write()` expansion mid-run.
- `ask_user_question` auto-decided in background/bypass; subagents inherit pre-approved perms.
- Review judge: `Any above issues MUST ALWAYS result in rejection... You MUST end with exactly one verdict: - Write <ACCEPT> if correct/ready - Write <REJECT> if issues...`
