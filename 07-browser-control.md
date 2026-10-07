# 07 — Browser Control

## 1. TL;DR: no Playwright/Puppeteer

- 0 hits for `playwright`, `puppeteer`, `selenium`, `chromedriver`.
- Custom stack: `browser-preview/*` proxy + `exa.browser_preview_pb` protobuf service.
- Sources in strings: `chisel-agent/src/browser_preview/tool.rs:185,160 + sink.rs`, `browser-preview/src/proxy.rs:210,217,224,330,346 + server.rs:73,121 + service.rs:97 + csp.rs + inject.rs + manager.rs`, `const SERVICE_PATH = "/exa.browser_preview_pb.BrowserPreviewService"` (`browser_preview.proto / browser_context.proto`).

## 2. How it works

- `browser_preview{url}`: `Spin up a browser preview for running web server`. `close_browser_preview{preview_id}`.
- Proxy runs on loopback, serves hand-written `cascade-browser-integration.js as-is (no bundler, Connect unary over binary protobuf, x-codeium-csrf-token, MAX_Z_INDEX 2147483647, capped console-error buffer)` at `/cascade-browser-integration.js` on proxy origin.
- Only local dev servers: `browser previews only support local dev servers, but target host is... / unsupported target URL scheme / invalid target URL`, `failed to build preview http client / failed to bind/read preview port`, `Blocked request with disallowed Host`, `Failed to start/forward capture / Closed/Closing preview`, `Could not open browser preview in a browser / preview already running`, `Browser preview running with ID: / did not inject into non-UTF8/>bytes HTML / proxy request failed / upstream upgrade failed`.
- `browser_enabled` flag; captures tagged `\[Browser capture #(\d+): [^\]]*\]`.

## 3. Action model

Single-string union (from `ToolUpdateEvent` variants 20002,20540,24009,26722,26774,34227):
`view|navigate|click|press_key|scroll_up|scroll_down|java_script|console|move_mouse|restart|select_file|select_option|set_mobile|set_desktop` + `screenshot_key,url_before_actions,url,browser_action,action_type,screenshot_paths,screenshot_base64`.

- `cognition.ai/actions,eventType,screenshotKey,inputKey,outputKey`; `QuoteScreenshot{url,filename}`; `ImageData{width,height,base64_data}`.
- ACP caps: `browser_preview, browser_preview_open, clipboard_write`.
- System guidance: `fail silently when done blind... judge rendered pixels yourself with browser tools`, `Delegate at most simple open-page-and-screenshot check, never build or rendered-state judgment`.
- Sidekick visual rule: `browser tools are slow use when lead asks or absolutely necessary; never exploratory clicking/e2e... render changed surface once, exercise only covered interactions, save screenshots as evidence, stop... A check that looks same if broken (build, DOM/log, merely loads) does not answer visual ask... If cannot render, say explicitly; if user-facing and not asked, state visually unverified... passing check = one line + screenshot paths...`

## 4. Cost/latency posture

- Prompts repeatedly call browser `slow` — use once, screenshot as evidence, stop. No auto-crawl, no multi-page agent.
- No background browser fleet; one preview per ID, user opens via `Could not open browser preview in a browser` fallback.

## 5. vs computer use

- Browser preview = DOM/proxy + screenshots for web dev verification. Computer use = OS-level coordinate actions (see `08-computer-use.md`). They share screenshot plumbing but are separate tool unions.
