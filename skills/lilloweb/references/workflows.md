# Page workflows

The JSON below describes **MCP tool name and argument pairs**, not shell commands or a batch API. Call each tool separately using the host's actual namespace, inspect its result, and continue only when appropriate. Replace `TAB_ID` and example locators with values from the current page. The example assumes the user requested the described action; labels are illustrative, not discovered targets.

## Fill and verify a form

For an inspected form with an email field and native country select:

```json
[
  {"tool":"page_inspect","arguments":{"tab_id":"TAB_ID"}},
  {"tool":"page_type","arguments":{"tab_id":"TAB_ID","locator":"label=Email","text":"reader@example.com","clear":true}},
  {"tool":"page_select","arguments":{"tab_id":"TAB_ID","locator":"label=Country","value":"PH"}},
  {"tool":"page_expect","arguments":{"tab_id":"TAB_ID","checks":[{"value":{"locator":"label=Email","equals":"reader@example.com"}},{"value":{"locator":"label=Country","equals":"PH"}}]}},
  {"tool":"page_click","arguments":{"tab_id":"TAB_ID","locator":"role=button[name=\"Save\"]"}},
  {"tool":"page_expect","arguments":{"tab_id":"TAB_ID","checks":[{"text":"Changes saved"}],"timeout_ms":10000}}
]
```

Use `page_fill_form` to reduce round trips when every field is known: `fields` is a list of `{ "locator": "…", "value": "…" }`. Checkbox/radio values are the strings `"true"` and `"false"`. Keep `submit` false or omitted while preparing a form that should not yet be submitted. Reinspect after a partial failure before continuing from the unresolved field.

A custom country dropdown needs `page_click` on its observed trigger, a fresh inspection, and `page_click` on the observed option. Do not assume `page_select` applies just because a control looks like a select.

## Navigation and dialogs

Use `tab_navigate` with the explicit `tab_id` and absolute URL. After navigation, or `tab_history` with `action: "back"`, `"forward"`, or `"reload"`, wait for the expected state. `load: true` waits for loading; `text`, `locator`, and `url_includes` express task-specific readiness. Multiple supplied conditions must all hold. `page_wait_for` defaults to 15 seconds and has a 60-second maximum; prefer a shorter meaningful bound and inspect on timeout.

```json
[
  {"tool":"page_wait_for","arguments":{"tab_id":"TAB_ID","load":true,"url_includes":"/settings","timeout_ms":10000}},
  {"tool":"page_inspect","arguments":{"tab_id":"TAB_ID"}},
  {"tool":"page_dialog","arguments":{"tab_id":"TAB_ID","accept":false}}
]
```

The last call is an example of declining an **observed JavaScript dialog** when that matches the user's intent. Do not run it merely because a wait failed. For a prompt, pass `text` with the intended response and an explicit `accept`. Do not click behind a blocking dialog or blindly accept a before-unload question.

## Separate sessions

For a signed-out or second-user test, call `context_open` with a descriptive `name` and `isolated: true`. Record the returned context ID, then pass it as `context_id` to `tab_open` with the target URL. Record that tab's returned ID and keep passing it to page tools. Use `contexts` when context state needs reconciliation.

At the end, `context_close` with the exact context ID closes every tab in that context. Close only a context you created for this task; never choose an existing context just because its name looks temporary. Similarly, `tab_close` is for your own temporary tabs, unless the user explicitly requested closing another tab. Preserve requested deliverables or a tab the user wants left open.

## Files and evidence

- For a file input, use `page_upload` with the inspected input `locator` and authorized absolute file `paths` on the Lilloweb machine. An empty `paths` list clears it. An OS picker opened by clicking cannot be controlled through page tools; do not use that as the upload path.
- After an export action, use `downloads` with a bounded `wait_ms` and a small `limit`. Inspect the returned status and path before claiming the file exists or is complete. File paths belong to the machine running Lilloweb.
- Use `page_pdf` when the requested artifact is print output. `filename` is a file name, not an arbitrary destination path; output goes to the download folder. Use `page_screenshot` for visual evidence (`full_page: true` when appropriate), and actually inspect the image before making appearance claims.
- Avoid reading cookies or session storage merely to inspect a page. Session restoration or storage changes should be scoped to the requested test; retain secrets locally and do not include them in reports.
