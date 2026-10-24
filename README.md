# Lilloweb skill

A portable [Agent Skill](https://agentskills.io/specification) for using the Lilloweb browser through its existing MCP tools. It teaches an agent to pick the right tab, inspect fresh page state, act through live locators, and verify the result. Focused references cover forms, isolated sessions, debugging, responsive checks, and safe workspace mocking.

The package is Markdown instructions. It does not install Lilloweb, replace its MCP server, or automatically modify agent configuration or permissions.

## Install in your project

Run from the project where you want the skill available, and select your agent interactively:

```sh
npx skills add gnobob/lilloweb-skill --skill lilloweb
```

Or select one explicitly, for example Codex:

```sh
npx skills add gnobob/lilloweb-skill --skill lilloweb --agent codex
```

Replace `codex` with the appropriate installer ID below. Choose multiple IDs only if you want the skill installed for each of those agents. Preview repository discovery without installing:

```sh
npx skills add gnobob/lilloweb-skill --list
```

These use the [Vercel skills installer](https://github.com/vercel-labs/skills). Project installation avoids differences between global skill directories. The installer can create links from an agent's directory to the shared project skill; check its summary and the destination `SKILL.md`, then reload skills or start a new agent session as your client requires.

### Manual installation

Copy the **entire** [skills/lilloweb](skills/lilloweb) directory into your agent's project skill directory from the table below. For example, the resulting file for Codex is `.agents/skills/lilloweb/SKILL.md`, with `references/` beside it. Copying only `SKILL.md` loses the supporting workflows. Inspect an existing `lilloweb` skill before replacing it.

For a user-wide manual install, use the user directory documented by your agent. Current Codex documentation names `~/.agents/skills/`; the skills installer's Codex global target has differed (`$CODEX_HOME/skills`, default `~/.codex/skills`). Do not assume global and project installation are equivalent. [Codex skill documentation](https://learn.chatgpt.com/docs/build-skills), [installer agent definitions](https://github.com/vercel-labs/skills/blob/main/src/agents.ts).

## Connect Lilloweb separately

1. Run Lilloweb on the same computer as the agent's MCP connector.
2. Open Lilloweb's **Connect an agent** dialog and use its runtime URL and token-file information in your agent's MCP setup. Lilloweb uses authenticated Streamable HTTP on loopback. Keep the bearer token private.
3. Confirm the host exposes Lilloweb tools, then ask it to use Lilloweb for a concrete page task.

The skill can be installed wherever the format is supported, but a cloud agent cannot reach your local Lilloweb endpoint without a separate supported local connector. This repository does not provide one or recommend exposing localhost publicly. See [connection guidance](skills/lilloweb/references/connection.md) for narrow troubleshooting steps.

Example requests:

- “Use Lilloweb to inspect my open localhost tab and explain why the save button fails.”
- “Use Lilloweb to check this page at 390 × 844 in dark mode and show the evidence.”
- “Use Lilloweb to verify the signed-out flow in a temporary isolated context.”

## Agent compatibility

The table records **documented skill discovery and installer IDs**, as checked by the upstream dive-skill on 2026-09-14 and not re-checked for Lilloweb since. It does not claim an end-to-end Lilloweb MCP run in every listed application. Each agent still needs its own working local MCP connection and applicable tool permissions.

| Agent | Installer `--agent` ID | Documented project location for the `lilloweb` folder | Primary documentation |
| --- | --- | --- | --- |
| Claude Code | `claude-code` | `.claude/skills/lilloweb/` | [Skills](https://code.claude.com/docs/en/skills) |
| Codex | `codex` | `.agents/skills/lilloweb/` | [Build skills](https://learn.chatgpt.com/docs/build-skills) |
| Cursor | `cursor` | `.agents/skills/lilloweb/` or `.cursor/skills/lilloweb/` | [Skills](https://cursor.com/docs/skills) |
| OpenCode | `opencode` | `.agents/skills/lilloweb/` or `.opencode/skills/lilloweb/` | [Agent skills](https://opencode.ai/docs/skills/) |
| Zed Agent | `zed` | `.agents/skills/lilloweb/` | [Skills](https://zed.dev/docs/ai/skills) |
| Gemini CLI | `gemini-cli` | `.agents/skills/lilloweb/` or `.gemini/skills/lilloweb/` | [Use skills](https://geminicli.com/docs/cli/using-agent-skills/) |
| GitHub Copilot | `github-copilot` | `.agents/skills/lilloweb/` or `.github/skills/lilloweb/` | [Agent skills](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills) |
| Windsurf / Cascade | `windsurf` | `.windsurf/skills/lilloweb/` | [Cascade skills](https://docs.windsurf.com/windsurf/cascade/skills) |

Zed Agent documents native `SKILL.md` support. If you use an external ACP agent or terminal agent inside Zed, install/configure the skill and MCP for that actual agent. The Windsurf documentation currently redirects to Devin Desktop's Cascade documentation; it retains the `.windsurf/skills/` path and documents `@lilloweb` invocation. No minimum released client versions are claimed here.

Other agents may consume the same standard folder. Check their current native skill discovery and MCP support before claiming compatibility; an installer ID alone does not establish a working browser integration.

## What is in the package

```text
skills/lilloweb/
  SKILL.md                  Essential tab, locator, action, and verification guidance
  references/
    connection.md           Local MCP connection and troubleshooting
    workflows.md            Forms, dialogs, isolated sessions, and files
    debugging.md            Diagnostics, snapshots, emulation, and mock rules
```

### Validation boundaries

Frontmatter, relative references, and example arguments can be checked independently of a browser. Installer discovery/project placement and a real browser read/action/result flow are separate checks. Compatibility documentation is not a substitute for those runtime checks, and page emulation is not physical-device or native-window verification.

## Provenance and license

The whole package is available under [MIT](LICENSE). The upstream research considered the compact MCP workflow in [ChromeDevTools' skill](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/skills/chrome-devtools/SKILL.md), progressive instruction loading in [Vercel agent-browser](https://github.com/vercel-labs/agent-browser/tree/main/skills/agent-browser), and observable browser verification in [djmorgan26/chrome-skill](https://github.com/djmorgan26/chrome-skill). No source text or implementation from those skills is bundled here. Their names, products, and upstream licenses remain their own; these links do not imply affiliation.
