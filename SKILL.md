---
name: visual-companion
description: Use when a discussion involves visual questions — mockups, layout or style comparisons, diagrams, visual hierarchy — that are better shown in a live browser than described in text.
disable-model-invocation: true
---

# Visual Companion

Run one browser companion across a discussion. The invocation is consent: start the session directly, then choose the browser only when seeing the question is better than reading it.

Treat this skill directory as read-only runtime. Write every session artifact under `<project>/.mockup/<session-id>/`.

## 1. Resolve the project root

Use the current Git repository root when one exists. Otherwise use the current working directory. If the selected directory is not writable, ask the user for a location.

**Complete when:** `PROJECT_ROOT` is an absolute, writable directory.

## 2. Start the session

Run `<skill-dir>/scripts/start-server.sh --project-dir "$PROJECT_ROOT" --open` in an execution session that can remain alive across turns. Save the returned `session_dir`, `screen_dir`, `state_dir`, and complete keyed `url`.

The browser opens when the first screen is ready. Share the complete URL, including `?key=...`, as a fallback.

`.mockup/` is generated local state and embeds the session key. In a Git repository, check immediately whether `.mockup/` is ignored; if it appears in version-control status, tell the user and offer to add a local ignore rule that follows the project's policy.

**Complete when:** `state_dir/server-info` identifies a live process, the keyed URL responds, and `.mockup/` is confirmed outside version control (or the user has decided otherwise).

## 3. Route each question

Use the browser for mockups, diagrams, layout or style comparisons, visual hierarchy, and spatial relationships. Use the conversation for requirements, scope, conceptual choices, trade-offs, and technical decisions.

When moving from a visual question to a text question, push a fresh waiting screen so the browser reflects the current phase.

**Complete when:** the current question has exactly one active surface.

## 4. Show a visual question

Before the first visual screen, read [visual-guide.md](visual-guide.md) completely. It is the single source of truth for content, interaction, assets, and recovery.

Confirm the server is alive. Write a new HTML file with a fresh semantic filename in `screen_dir`; keep the question, fidelity, and interaction focused on the decision at hand. Tell the user what is displayed and provide the session URL.

**Complete when:** the new file is the newest screen, the keyed URL responds, and the user knows what to inspect.

## 5. Listen and validate

On the next turn, read `state_dir/events` when present and merge it with the user's conversational feedback. Treat conversational feedback as primary and browser events as structured evidence.

Revise the current screen with a fresh filename when requested. Advance only after the current visual question is confirmed or the next revision is explicit.

**Complete when:** the current visual question is validated or has a concrete revision target.

## 6. Keep the browser current

Push a new visual screen for each visual question and a fresh waiting screen for each return to text. Reuse the same server for the full discussion. If it stopped, restart with the same `PROJECT_ROOT` and replace the saved session paths with the new values.

**Complete when:** the browser content matches the current discussion phase.

## 7. Close the session

When the user ends the companion session, run `<skill-dir>/scripts/stop-server.sh "$SESSION_DIR"`. Preserve the generated content under `.mockup/`.

**Complete when:** the server process has exited and `state_dir/server-stopped` records the shutdown.
