# 04 — Compaction & Context Management

Source: `agent-ext/src/compactor/{async_file_compactor,file_compactor,summarize,todo_list,handoff_history}.rs`, `file_compactor.rs:504`, strings `compaction`, `COMPACTION_THRESHOLDS`, `ModelFamilyMetadataValue{compaction_thresholds}`, docs `/compact /usage /context`.

## 1. Trigger model: spawn / apply / hard

- Config: `--compaction-thresholds` / `DEVIN_COMPACTION_THRESHOLDS` / `SIDEKICK_COMPACTION_THRESHOLDS` (hidden from `--help`):
  `Override thresholds with absolute token counts: <spawn>,<apply>,<hard> (start a summary, apply it when ready, block inference), e.g. 183500,183500,262144. <spawn>,<hard> applies only at hard limit.`
- Validation: `expected <spawn>,<hard> or <spawn>,<apply>,<hard> positive counts with spawn <= apply <= hard, got `
- Server defaults per model family (`ModelFamilyMetadataValue{...,compaction_thresholds,aliases}`); user override wins; sidekick has separate thresholds (fallback when unset).
- Signals: `System prefix exceeds compaction threshold`, `Compacting conversation history with messages and tokens`, `AsyncFileCompactor: spawned background compaction at node / evicting pending summary / split point no longer in chain`, `ForceCompact received while busy→reject`.
- Manual: `/compact` == `Force conversation compaction`; `/usage` == `Show context window usage`; `/context` shows sections.

## 2. How summarization works

- Background by default: `AsyncFileCompactor: spawned background compaction...falling back to sync`; `AsyncFileCompactor: override model failed, retrying with fallback`; `Compaction with override model failed, falling back to default model`.
- History summarizer prompt (full in `03-prompts.md`): `<summary> ## Overview ## Key Details & Breadcrumbs ## Current State </summary>`, `Do NOT reproduce rules/instructions verbatim (<rules>/<rule>)`, `full conversation saved to history file ()`.
- Trace summarizer: same template for execution trace, `You have no tools. Output saved automatically... Output full summary every time do not output diffs/partials`.
- Title generator: `max 80 chars, ONLY title text`.
- Resume: `You are continuing work from previous thread. Below is summary...` + `<conversation_history>` with omitted prefix noted.
- Errors: `No nodes to summarize / too few messages to summarize`, `Compaction stream creation failed (attempt ), retrying...`, `Conversation to summarize: / #Full conversation history saved at / Summary:`, `Fatal compaction error: / FatalCompaction(CompactionFailed`, `Compaction cancelled by revert / user / interrupted`, `Cannot compact: no active model / Nothing to compact / Context compacted / Force compaction failed`.
- Post-compaction hook (`core/compaction`, `PostCompaction(summary)`) can re-inject context; OTEL `compaction{prompt.id,summary_length}`.
- Flags: `cli-agents-md-lazy-walk`, `enable-devin-cli-cloud-compaction-schedule`, `enable-devin-cli-swe2-self-compaction`.

## 3. What is preserved vs dropped

- Preserved: summary (overview/details/state), todo list (`todo_list` compactor), handoff history (`handoff_history`), file `~/.local/share/devin/summaries/<session>.md` (summarizer agent-type), SQLite `compaction_epoch` increment.
- Dropped: raw messages before split point, rules verbatim (re-injected from config), thinking blocks per `preserved_thinking`/`system_prefix_len`.
- `show_history_on_continue`, `preferred_family_models`, `model_mixture` affect what re-loads.

## 4. Storage

- SQLite: `compaction_epoch INTEGER DEFAULT 0, UNIQUE(session_id,compaction_epoch,sequence_number)`, `session_db.rs`, `message_nodes`, `chain_head`, `-- Chain heads for persistent subagents (e.g. Local Fusion sidekick)`.
- `/steps`, `/fork [step]`, `/revert <step>` operate on pre-compaction nodes (`revert/engine.rs steps.rs`, `cognition.ai/revert/{listSteps,execute,forkFromStep,preview}`).
- No client-side truncation beyond `output_trunc, content redacted or truncated before upload, Truncatable scrollback`.

## 5. Compaction vs caching conflict

- Caching is prefix-stability dependent (server-side). Compaction rewrites history → invalidates cache. Hence `spawn` (start summary early) vs `apply` (swap when ready) vs `hard` (block inference) — background summary hides latency and preserves one last cached prefix.
- `Switching to one-shot response mode. Tools attached only to preserve the prompt cache; do NOT call tools...` — tools kept attached solely to avoid prefix shift.
