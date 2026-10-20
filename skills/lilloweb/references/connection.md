# Connect the skill to Lilloweb

Installing the skill and connecting MCP are separate steps. A supported local agent needs a running Lilloweb browser and a configured MCP connection. Lilloweb serves **Streamable HTTP on loopback**, with an `Authorization: Bearer …` header. The connector must run on the same computer as Lilloweb; a cloud-hosted agent cannot reach this computer's localhost just because it can load the skill. Do not publish or tunnel the endpoint to make a cloud integration work.

## Use the running browser's connection details

Open Lilloweb's **Connect an agent** dialog, or use already-provided connection information. It supplies the current endpoint and token-file location for that running instance. Follow the host agent's current MCP configuration flow using those runtime values. Do not guess a port, OS-specific profile path, or configuration format shared across all agents.

The conceptual settings are:

```text
Transport: Streamable HTTP
URL: <MCP_URL_FROM_LILLOWEB_CONNECT_DIALOG>
Header: Authorization: Bearer <LILLOWEB_TOKEN>
Token source, when supported: <TOKEN_FILE_PATH_FROM_LILLOWEB_CONNECT_DIALOG>
```

These are placeholders, not a ready-to-run configuration. Resolve the token locally using the client's supported secret mechanism; do not print it to logs, commit it, include it in screenshots/reports, or upload it to an external service. If the client requires a literal header, keep its configuration private. The skill contains no installer that changes MCP configuration automatically.

Once connected, discover the tools and call `lilloweb_capabilities`, then `tabs_list` and `page_inspect` for an explicitly selected tab. A raw MCP client must perform protocol initialization/session handling through a compatible MCP SDK; this is not a stateless JSON endpoint to wrap with guessed `curl` calls.

## Diagnose the narrow failure

| Symptom | Next useful check |
| --- | --- |
| Skill appears, but no Lilloweb tools | Confirm MCP is configured and enabled in the actual host agent. Installing Markdown does not register a server. External agents launched inside an editor use their own agent configuration. |
| Connection refused | Confirm Lilloweb is running and compare the endpoint against the current Connect dialog. A remote shell/container's localhost may be a different machine. |
| Unauthorized | Check the configured bearer header against the runtime token source without exposing the token. Do not disable authentication. |
| Session/protocol error | Reconnect through the host's MCP client so it initializes a fresh session. Then rediscover capabilities and tab IDs. |
| Expected tab missing | Remember `tabs_list` is scoped to the active workspace. Confirm workspace context using an available authorized surface; do not pretend the page MCP switches native workspaces. |
| Tool or argument rejected | Read the live schema; use its exact argument names. `tab_id` is not `id`. Refresh discovery after a browser upgrade or reconnect. |
| Evaluation refused | Respect the capability flag. Use the dedicated page tool or describe the missing capability. |

Configuration authority: [Lilloweb MCP transport](https://github.com/YOUR-GITHUB-NAME/YOUR-BROWSER-REPO/blob/main/crates/dive-mcp/src/http.rs), [Connect dialog](https://github.com/YOUR-GITHUB-NAME/YOUR-BROWSER-REPO/blob/main/apps/desktop/src/components/McpDialog.tsx).
