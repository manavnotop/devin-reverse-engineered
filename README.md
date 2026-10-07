# Devin CLI Reverse Engineering — Index

Version: `3000.11.3 (9c803229faa4)` — Mach-O arm64 163 MB Rust binary
Bundle: `https://static.devin.ai/cli/3000.11.3/devin-3000.11.3-aarch64-apple-darwin.tar.gz`
Date: 2026-10-07 | Method: `strings -a` (338k lines), `--help` per subcommand, bundled `*.mdx` docs, `otool -L`

## Files

- `00-overview.md` — binary identity, install flow, client vs cloud, versioning
- `01-harness-loop.md` — REPL → ACP → cog loop, turn vs step, termination, interrupts, parallel exec
- `02-tools.md` — full tool inventory with descriptions, schemas, families
- `03-prompts.md` — ALL extracted prompts verbatim (system, sidekick, compaction, classifier, subagents)
- `04-compaction-context-management.md` — compaction thresholds, summarizers, SQLite chain, history
- `05-cost-optimisation.md` — Fusion pricing, ACU/credits, smart-mode, cache economics
- `06-token-optimisation.md` — prompt caching, prefix stability, truncation, lazy context
- `07-browser-control.md` — browser-preview proxy, tools, actions, no Playwright
- `08-computer-use.md` — CognitionComputerUse structs, grounding, permissions
- `09-why-never-says-no.md` — permission model, trust, sandbox, refusal → fallback
- `10-fusion.md` — Fusion deep dive (lead/sidekick, routing, pricing, killswitches, UX)
- `11-architecture-network-provider.md` — Rust crates, ACP relay, API endpoints, OTEL/Sentry, session store

## How to reproduce

```bash
curl -sSf https://static.devin.ai/cli/current/manifest.json | python3 -m json.tool
curl -fL "https://static.devin.ai/cli/3000.11.3/devin-3000.11.3-aarch64-apple-darwin.tar.gz" -o devin.tar.gz
tar xzf devin.tar.gz -C bundle
file bundle/bin/devin
./bundle/bin/devin --help
strings -a -n 8 bundle/bin/devin > /tmp/devin.strings.txt
wc -l /tmp/devin.strings.txt  # ~45010 with -n 8, 338002 default
grep -a -o "toolbox/src/tools/[a-z_/]*\.rs" /tmp/devin.strings | sort -u
grep -a -o "chisel-agent/src/[a-z_/]*\.rs" /tmp/devin.strings | sort -u
```

All claims below carry evidence: `bundle/...` doc path, `--help` output, or `strings` sample + source path.
