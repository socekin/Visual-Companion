# Visual Companion Reference

This file defines the browser content and interaction contract. The operational loop lives only in `SKILL.md`.

## Media routing edge cases

Apply the routing rule from `SKILL.md` based on the answer format:

- "What should this wizard accomplish?" belongs in the conversation; "Which wizard layout communicates progress better?" belongs in the browser.
- An architecture choice described through trade-offs belongs in the conversation; its component relationships or data flow belong in the browser.
- A text-heavy list or table belongs in the conversation even when the surrounding topic is UI.
- A visual comparison belongs in the browser even when each option also needs a short written rationale.

## Session contract

`start-server.sh` returns JSON with these fields:

```json
{
  "type": "server-started",
  "port": 52341,
  "url": "http://localhost:52341/?key=...",
  "session_dir": "<project>/.mockup/<session-id>",
  "screen_dir": "<project>/.mockup/<session-id>/content",
  "state_dir": "<project>/.mockup/<session-id>/state"
}
```

Keep the complete keyed URL private to the session. The key authorizes HTTP and WebSocket access. The browser stores it in a session cookie after the first load.

The server writes the same startup JSON to `state_dir/server-info`. A stopped server removes that file and writes `state_dir/server-stopped`.

The version-control check for `.mockup/` happens at session start; see `SKILL.md` step 2.

## Screen documents

Write a content fragment by default. The server wraps fragments in `frame-template.html` and injects `helper.js`.

```html
<h2>Which layout communicates priority better?</h2>
<p class="subtitle">Compare scan order and visual hierarchy.</p>
```

Use a full document beginning with `<!DOCTYPE` or `<html` when the screen needs complete layout, styling, or script control. The server still injects the interaction helper.

Use a fresh semantic filename for every screen:

- `dashboard-layout.html`
- `dashboard-layout-v2.html`
- `navigation-model.html`
- `waiting-for-terminal.html`

The newest HTML file by modification time is the active screen. New screens clear the previous `state/events` file.

## Fidelity and options

Match fidelity to the question:

- Use wireframes for structure and layout.
- Use polished treatments for spacing, hierarchy, color, or visual tone.
- Use real content when content affects the decision.
- Keep each screen focused on one question.
- Present two to four options when comparison is useful.

## Choice interactions

Add `data-choice` to clickable options and call `toggleSelect(this)` for the built-in selected state.

```html
<div class="options">
  <div class="option" data-choice="single-column" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>Single column</h3>
      <p>Focused reading with a simple scan path.</p>
    </div>
  </div>
  <div class="option" data-choice="split-view" onclick="toggleSelect(this)">
    <div class="letter">B</div>
    <div class="content">
      <h3>Split view</h3>
      <p>Persistent context beside the primary workspace.</p>
    </div>
  </div>
</div>
```

Add `data-multiselect` to `.options` or `.cards` when more than one choice may remain selected.

For custom interactions, use the injected API:

```javascript
window.visualCompanion.send({ type: 'preview', choice: 'compact' });
window.visualCompanion.choice('compact', { source: 'toolbar' });
```

## Available frame classes

- `.options`, `.option`, `.letter`, `.content` for text choices
- `.cards`, `.card`, `.card-image`, `.card-body` for visual directions
- `.mockup`, `.mockup-header`, `.mockup-body` for framed previews
- `.split` for side-by-side comparison
- `.pros-cons`, `.pros`, `.cons` for supporting trade-offs
- `.mock-nav`, `.mock-sidebar`, `.mock-content`, `.mock-button`, `.mock-input` for wireframes
- `.placeholder`, `.subtitle`, `.section`, `.label` for supporting structure

## Session assets

Place screen-specific HTML, CSS, JavaScript, images, and JSON in `screen_dir`. Reference peer files through `/files/<filename>`:

```html
<link rel="stylesheet" href="/files/dashboard.css">
<script src="/files/dashboard.js"></script>
<img src="/files/dashboard.png" alt="Dashboard direction">
```

The shipped runtime stays in the skill's read-only `scripts/` directory. Place every generated or task-specific asset in the session `content/` directory.

## Browser events

Choice clicks are appended to `state_dir/events` as JSON Lines:

```jsonl
{"type":"click","choice":"single-column","text":"A Single column Focused reading","timestamp":1706000101}
{"type":"click","choice":"split-view","text":"B Split view Persistent context","timestamp":1706000115}
```

The event stream shows exploration order. Treat the last selection as evidence; confirmation comes from the merged conversational response.

## Waiting screen

Push a fresh waiting screen when the discussion returns to text:

```html
<div style="display:flex;align-items:center;justify-content:center;min-height:60vh">
  <p class="subtitle">Continuing in the conversation...</p>
</div>
```

## Platform and recovery

The default command supports macOS, Linux, Windows Git Bash, Codex, Claude Code, and Copilot CLI. Codex and environments that reap detached processes use foreground mode inside a persistent execution session.

For a remote or containerized environment, bind a reachable interface while keeping the displayed URL appropriate for the user's browser:

```bash
scripts/start-server.sh \
  --project-dir /path/to/project \
  --host 0.0.0.0 \
  --url-host localhost \
  --open
```

The transport is plain HTTP and WebSocket. On a non-loopback bind the session key travels unencrypted, so use a non-loopback bind only on a trusted network or through an encrypted tunnel (SSH port forwarding or equivalent), and prefer the tunnel with the default loopback bind when available.

Confirm liveness before every visual push: `state_dir/server-info` names a live server. A `state_dir/server-stopped` marker triggers a restart. A restart with the same project root reuses the saved port and session key when available, allowing an open tab to reconnect.

The client retries with exponential backoff and shows a paused overlay after a prolonged disconnect. The server exits after four idle hours by default; `--idle-timeout-minutes <n>` changes that limit.

## Troubleshooting

- **No browser opened:** share the complete keyed URL from `server-info`.
- **Forbidden page:** reopen the complete URL including `?key=...`.
- **Stale screen:** confirm the new HTML file is the newest file in `screen_dir`.
- **No click events:** confirm the element has `data-choice` and the browser is connected.
- **Server stopped:** restart with the same project root and use the new `screen_dir` and `state_dir`.
- **Port already in use:** the server selects a fallback port and reports the resulting URL.
