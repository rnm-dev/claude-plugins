# Contributing to rnm-plugins

Thanks for adding to the marketplace. Quick rules.

## Workflow

1. Branch off `main`: `git checkout -b your-name/plugin-name`
2. Scaffold a new plugin directory under `plugins/` (see
   [README → Adding a new plugin](./README.md#adding-a-new-plugin)).
3. Build out skills, agents, hooks, or MCP servers.
4. Register the plugin in `.claude-plugin/marketplace.json`.
5. Validate: `claude plugin validate .`
6. Open a PR. Tag a code owner for review.

## Plugin naming

- Use `kebab-case`. Lowercase letters, digits, hyphens. No spaces, no underscores.
- Prefix internal-only plugins with `rnm-` when the name is generic
  (`rnm-deploy`, not `deploy`).
- The plugin `name` in `plugin.json` and in `marketplace.json` MUST match.

## Versioning

- Pick one: pin a `version` and bump it on every release, OR omit `version` and
  let the git SHA do the work.
- Don't set `version` in both `plugin.json` and `marketplace.json` — `plugin.json`
  silently wins and a stale entry will mask your release.

## Reviews

- Every plugin needs at least one approver before merging.
- Plugins that ship hooks or MCP servers need a second pair of eyes (they run
  arbitrary commands on contributors' machines).
- Don't merge anything that fails `claude plugin validate .`.

## Removing a plugin

1. Delete the plugin directory under `plugins/`.
2. Remove the entry from `.claude-plugin/marketplace.json`.
3. Note the removal in the PR description so users know to uninstall.
