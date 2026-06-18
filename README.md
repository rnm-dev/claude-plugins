# rnm-plugins

RNM's internal marketplace of [Claude Code](https://code.claude.com) plugins.
Skills, agents, hooks, and MCP servers we use across the team — packaged so
anyone with Claude Code can install them in one command.

## Install the marketplace

```bash
/plugin marketplace add rnm-dev/claude-plugins
```

Then browse and install plugins:

```bash
/plugin
```

Or install a specific plugin directly:

```bash
/plugin install plane@rnm-plugins
/plugin install heroboard@rnm-plugins
```

## Auto-install for the whole team

Drop this in your repo's `.claude/settings.json` so anyone who trusts the
project folder gets prompted to install:

```json
{
  "extraKnownMarketplaces": {
    "rnm-plugins": {
      "source": {
        "source": "github",
        "repo": "rnm-dev/claude-plugins"
      }
    }
  },
  "enabledPlugins": {
    "plane@rnm-plugins": true,
    "heroboard@rnm-plugins": true
  }
}
```

## Available plugins

| Plugin | Description |
| :--- | :--- |
| [`plane`](./plugins/plane) | Manage tasks in RNM's Plane workspace via the Plane REST API |
| [`heroboard`](./plugins/heroboard) | Track effort on Heroboard — MCP task tools, Monkey/Agent effort heartbeats, and `/heroboard` commands |

## Adding a new plugin

1. Scaffold a directory under `plugins/`:
   ```bash
   mkdir -p plugins/your-plugin-name/.claude-plugin
   ```
2. Create `plugins/your-plugin-name/.claude-plugin/plugin.json` with `name`,
   `description`, `version`, and an `author`. See
   [`plugins/plane/.claude-plugin/plugin.json`](./plugins/plane/.claude-plugin/plugin.json)
   for an example.
3. Build your plugin. A plugin can include any combination of these directories
   at the plugin root (NOT inside `.claude-plugin/`):

   | Directory / File | Purpose |
   | :--- | :--- |
   | `skills/<name>/SKILL.md` | Model-invokable skills (and slash commands) |
   | `agents/<name>.md` | Custom subagents |
   | `hooks/hooks.json` | Event hooks (PreToolUse, PostToolUse, etc.) |
   | `.mcp.json` | MCP servers |
   | `.lsp.json` | LSP servers |
   | `monitors/monitors.json` | Background monitors |
   | `bin/` | Executables added to `PATH` while the plugin is enabled |
   | `settings.json` | Default settings applied when the plugin is enabled |
4. Register the plugin in the root `.claude-plugin/marketplace.json` `plugins`
   array.
5. Validate before opening the PR:
   ```bash
   claude plugin validate .
   ```
6. Open a PR. Once merged, anyone running `/plugin marketplace update` picks
   up the new plugin.

## Local development

Test a plugin without going through the marketplace:

```bash
claude --plugin-dir ./plugins/your-plugin-name
```

Or test the marketplace end-to-end:

```bash
/plugin marketplace add ./
/plugin install your-plugin-name@rnm-plugins
```

After making changes while Claude Code is running:

```
/reload-plugins
```

## Versioning

Plugins resolve their version from the first of these that's set:

1. `version` in the plugin's `plugin.json`
2. `version` in the marketplace entry
3. The git commit SHA

If you set `version`, **bump it on every release** or users won't see updates.
If you omit it, every commit counts as a new version — easier for actively-
developed internal plugins.

## Reference

- [Claude Code plugins docs](https://code.claude.com/docs/en/plugins)
- [Plugin marketplaces docs](https://code.claude.com/docs/en/plugin-marketplaces)
- [Plugins reference](https://code.claude.com/docs/en/plugins-reference)
