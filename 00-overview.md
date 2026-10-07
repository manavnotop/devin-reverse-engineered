# 00 — Overview: What Devin CLI Actually Is

## 1. Identity

- Binary: `bundle/bin/devin` — `Mach-O 64-bit executable arm64`, 163 MB (`171,232,496 B`), version `devin 3000.11.3 (9c803229faa4)`.
- Language: Rust (`Cargo [[bin]] target name devin`). Evidence strings: `toolbox/src/tools/*.rs`, `chisel-agent/src/...`, `chisel-server/src/...`, `chisel-commands/src/...`, `chisel-api/src/...`, `agent-ext/src/...`, `affogato/src/cog.rs`, build path `/Volumes/.../cognition/devin-webapp/apps/chisel/target/...`.
- Stripped: `nm` shows only `U _CF* _CCRandom*` imports. `otool -L`: AppKit, CoreVideo, CoreData, CoreImage, CoreGraphics, CloudKit, QuartzCore, Foundation, CoreText, IOKit, CoreFoundation, libobjc, CoreServices, CFNetwork, Security, SystemConfiguration, libiconv — i.e. system frameworks + Rust static link, no bundled Chromium/Electron.
- Tagline from `--help`: `A fast and minimal agent that lives both in your terminal and in the cloud.`

## 2. Install flow (`https://cli.devin.ai/install.sh`)

1. `detect_target()`: `aarch64-apple-darwin`, `x86_64-apple-darwin`, `x86_64-unknown-linux`, `aarch64-unknown-linux`, else `windows` → PowerShell installer.
2. `MANIFEST_URL=https://static.devin.ai/cli/current/manifest.json` (or `manifest-enterprise.json`, `manifest-windsurfcom.json`, `manifest-federal.json`). `PINNED_VERSION` supported for RCs.
3. Parse `version` + per-target `{url, sha256}` via grep/sed, verify with `sha256sum`/`shasum -a 256`, `tar xzf` to `$XDG_DATA_HOME/devin/cli/_versions/<version>/`, symlink `_versions/current → <version>`, symlink `~/.local/bin/devin → .../current/bin/devin`.
4. Migrates legacy namespaces `~/.local/share/cognition/` and `chisel/` → `devin/` + compat symlink.
5. Writes `distribution` marker per install method, links man pages, then runs `devin setup` wizard.
6. Current manifest (2026-10-07): `version 3000.11.3`, 10 platform entries including `-musl` and `windows-gnu/gnullvm` aliases.

## 3. Client vs cloud — the key architectural fact

- **Local agent by default.** REPL (`chisel-commands/src/repl`) spawns `devin acp` child speaking ACP JSON-RPC over stdio. Inference goes to Cognition API (`AssignModel`, `GetChatMessage`, etc.) but tools execute locally.
- **`--cloud` mode**: REPL talks to `devin acp --cloud`, which relays ACP to Devin cloud ACP WebSocket. Requires `auth login`. Then `ssh`/`forward` attach to cloud box, `/handoff`, `/pickup`, `/open` move between local/cloud.
- **`acp` server mode**: `devin acp [--agent-type summarizer|review] [--model] [--refusal-fallback] [--cloud]` — used by Devin Desktop, Zed/JetBrains/Xcode ACP clients, and headless. `summarizer: no tools, PostAgentIteration cog persists to ~/.local/share/devin/summaries/<session>.md`.
- Old names survive: `cognition/`, `chisel/`, `exa.*_pb` (browser preview, api server), `codeium` Sentry DSN — Cognition = company, Chisel = CLI codename, Exa/Codeium = legacy infra.

## 4. Entry points

```
devin [OPTIONS] [PATH]... [-- <PROMPT>...] [COMMAND]
devin --model opus -- refactor this module
devin -p "fix bug"                  # print: single turn non-interactive
devin -c / -r <session>             # continue / resume
devin --permission-mode auto|accept-edits|smart|dangerous
devin --sandbox                     # forces autonomous mode
devin --cloud                       # cloud relay
```

Subcommands: `auth mcp models doctor rules skills plugins cloud desktop list rm ssh forward update version migrate sandbox setup uninstall acp help`.
Quirk: `[PATH]...` swallows bare `devin mcp list` as prompt; use `devin <sub> <subsub> --help`.

## 5. Defaults that matter

- Model default: `swe-1-6-fast` (`config-file.mdx`, `agent.model`). Recommended: Fusion.
- Permission default: `auto` (read-only auto, rest prompt).
- Config: `~/.config/devin/config.json` > `.devin/config.json` > `.devin/config.local.json` > session grants > org/team (highest).
- Sessions: SQLite (`session_db.rs`, `message_nodes(session_id,compaction_epoch,seq)`, `chain_head`, `turnClientMessageId/turnRequestId`).
- Telemetry: OTEL `ai.devin.local` + Sentry (`codeium-i5.sentry.io`, `o4507463137361920.ingest.us.sentry.io/4510507667685376`, `sentry.rust/0.47.0`).

## 6. What it is NOT

- Not a fully offline agent — models + AssignModel router + team settings are server-side.
- Not Playwright/Puppeteer browser — custom `browser-preview` proxy (see `07-browser-control.md`).
- Not a framework — a harness: drives a model in a loop with tools. No embeddings/RAG/planner in binary.
