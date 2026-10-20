# Debugging and visual checks

Use only the sections relevant to the requested investigation. These examples describe individual MCP calls, not a batch API. Replace `TAB_ID`, request IDs, and example targets with observed values.

## Find the app and the failure

Call `dev_servers` to discover listening local apps and their URLs instead of guessing a development port. Select the server that matches the project and then open or navigate the intended tab. If none is present, starting a dev server is a separate project action using the available terminal tooling and the project's instructions.

For a reported failure, `page_report` provides a quick Markdown summary of console problems and failed requests. Combine it with `page_inspect` to connect diagnostics to current user-visible state. Narrow deeper investigation to the relevant entries:

```json
[
  {"tool":"page_report","arguments":{"tab_id":"TAB_ID"}},
  {"tool":"console_tail","arguments":{"tab_id":"TAB_ID","limit":20}},
  {"tool":"network_list","arguments":{"tab_id":"TAB_ID","limit":20}},
  {"tool":"network_body","arguments":{"tab_id":"TAB_ID","request_id":"REQUEST_ID_FROM_NETWORK_LIST"}}
]
```

Fetch a response body only when a particular observed request is relevant. Captured bodies are limited and may contain personal data or tokens; avoid indiscriminate collection and redact sensitive material from deliverables. `page_component` with an observed element locator can point to a React component/source file in a development build. Missing or minified production metadata is a limit, not evidence that no component exists.

## Compare before and after

Call `page_snapshot` on the chosen tab before a relevant change. Make the change, wait for its expected result, then call `page_diff` on the same tab. It compares text, structure, errors, and requests with the prior snapshot. This is structural/diagnostic evidence, not a pixel diff. Pair before/after screenshots with inspection when verifying appearance; use `page_expect` for the behavior the user cares about.

## Responsive and media behavior

Use a task-owned tab when possible so existing user emulation preferences are preserved. Record any known original overrides before changing an existing tab; if their original values cannot be established, use a new tab instead of assuming a reset restores them.

Call `page_devices` to discover valid preset IDs. Choose either a returned preset or exact `width`/`height`. Presets can change user agent, touch, pixel ratio, and safe-area behavior; a user-agent change can reload the tab. Wait and reinspect after resizing before evaluating layout.

```json
[
  {"tool":"page_resize","arguments":{"tab_id":"TAB_ID","width":390,"height":844}},
  {"tool":"page_appearance","arguments":{"tab_id":"TAB_ID","color_scheme":"dark","reduced_motion":"reduce"}},
  {"tool":"page_inspect","arguments":{"tab_id":"TAB_ID"}},
  {"tool":"page_screenshot","arguments":{"tab_id":"TAB_ID","full_page":true}}
]
```

Inspect screenshot and page behavior at the requested sizes/preferences. For a preset, `ui` can be `"browser"`, `"standalone"`, or `"none"`; `orientation` accepts `"portrait"` or `"landscape"`. These preset options do not belong to an exact-size request. Emulation verifies the rendered page under those settings; it is not physical-device, operating-system, or native window verification.

Restore the previous values of overrides you changed. On a task-owned tab that originally had none, `page_resize` with `reset: true` clears viewport emulation; `page_appearance` with `"system"` for each changed field clears those media overrides. Do not clear fields the task did not change. Closing your temporary tab also avoids leaving its overrides behind.

## Slow networks and API failures

For a tab-scoped loading test, `page_throttle` accepts `profile: "offline"`, `"slow-3g"`, `"fast-3g"`, or `"none"`. Apply it on a task-owned tab, reload with `tab_history`, and assert the intended loading/error behavior with a bounded wait. Clear your throttle with `"none"` afterward or close the temporary tab.

Mock/rewrite rules have a different scope: **the active workspace**. `rules_set` replaces the entire list, including rules unrelated to the tab under test. An isolated cookie context does not make rules tab-local.

When workspace mocking is within the requested task:

1. Read `rules_list` and preserve the full original list. Check the workspace scope and use a narrowly targeted URL pattern for your rule.
2. Add the task rule to that preserved list, giving it a distinct ID, then pass the combined list as `rules` to `rules_set`. Do not send just the new rule or an empty list when existing rules should survive.
3. Reload the test tab, wait for the relevant result, and verify the error/recovery state. Do not reload unrelated tabs.
4. Read the latest rules before cleanup. Remove only your rule while preserving concurrent unrelated changes; if an existing rule was intentionally changed, reconcile it against the saved original. Restore the saved list directly only when nothing else changed. If ownership is ambiguous, report the conflict instead of overwriting someone else's work.

One rule's source-defined shape is:

```json
{"id":"lilloweb-test-status-503","pattern":"https://example.com/api/status","enabled":true,"action":{"kind":"mock","status":503,"content_type":"application/json","body":"{\"error\":\"temporary failure\"}"}}
```

This illustrative rule is not the complete `rules_set` argument and should not be applied to a real service without an applicable test request. Other supported actions are `{"kind":"block"}` and `{"kind":"header","name":"…","value":"…"}`. Treat stored headers as potentially sensitive when saving or reporting rule state.
