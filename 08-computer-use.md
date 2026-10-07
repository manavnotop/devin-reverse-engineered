# 08 — Computer Use

## 1. What exists in binary

- `computer use`: 0 natural-language hits. Only structured enum `computer_use_action` co-occurring with `browser_action` in `ToolUpdateEvent` variants (20002,20540,24009,26722,26774,34227).
- Structs verbatim:
```
struct CognitionComputerUseAction with 9 elements:
 action_type,coordinate,duration,key,region,scroll_amount,scroll_direction,start_coordinate,text
struct CognitionComputerUseMeta with 5 elements
CognitionComputerUseAction|coordinate|duration|key|region|scroll_amount|scroll_direction|start_coordinate|text
```
- Tool union includes `computer_use_action + screenshot_keys` alongside `browser_action`.
- Permission union in same area: `sk::core/bypass, core/plan_mask, core/autonomous, core/accept_edits, core/read_only_kind, core/smart_permission` — computer actions go through same `allow|deny|ask` gate.

## 2. Interpretation

- Schema is OS-action grounding: `action_type` (click/press/scroll/move), `coordinate`/`start_coordinate`/`region`, `duration`, `key`/`text`, `scroll_amount`/`scroll_direction`.
- No VNC/RDP strings; grounding is via browser-preview screenshots + coordinates (`screenshot_key`, `ImageData{width,height,base64_data}`).
- No embedded computer-use system prompt — behavior is model-driven (frontier vision) + harness-enforced via permissions/sandbox, not via separate computer-use agent instructions.
- Likely surfaces via Desktop/cloud (`devin desktop`, `devin ssh/forward`) rather than terminal REPL — REPL strings expose `browser_preview` tool but no `computer_use` tool description.

## 3. Safety

- Same as shell: `smart` judge would `ask` on destructive/irreversible coordinate actions; `deny` always wins; sandbox seatbelt/bwrap bounds filesystem, not screen.
- No evidence of OS-level input sandboxing in CLI; enterprise `sandbox.excluded` only covers `Exec(...)`.

## 4. Limits

- Cannot conclude full computer-use capability from CLI alone — cloud Desktop agent likely holds the driver. CLI binary only carries types/telemetry, not a computer-use loop.
