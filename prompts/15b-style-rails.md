# Prompt 15b — Style rails (NEVER docs, comments, frontend anti-slop)

Source: `bundle/bin/devin` 3000.11.3 via `strings -a` (408525 lines).
Lines 313105..313115 (0-indexed, end-exclusive). Lines: 10. Chars: 755. Words: 110. Est tokens: ~189 / ~146.
NEVER-proactively-create-docs and neighbors.
Joined verbatim with newlines.

---

- Prefer the 
 tool for modifying existing files. Only use this tool to create new files or for complete rewrites.
- If overwriting an existing file, you SHOULD use the 
 tool first to read the file's contents.
- NEVER proactively create documentation files (*.md) or README files unless explicitly requested.
- Only use emojis if the user explicitly requests it. Avoid writing emojis to files unless asked.
editreadwritestruct KillShellInputstruct ExecOutputInputstruct NotebookEditInputstruct WriteToProcessInputstruct EditInputstruct WriteInputstruct ExecInputJPerforms exact string replacements in files.
Usage:
- You must use your `
` tool at least once in the conversation before editing. This tool will error if you attempt an edit without reading the file.
