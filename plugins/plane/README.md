# plane

Lets Claude Code manage tasks in RNM's self-hosted [Plane](https://plane.so)
workspace at [plane.rnm.dev](https://plane.rnm.dev).

Ships a single skill, `plane:tasks`, that teaches Claude how to hit the Plane
REST API directly with `curl`. No MCP server, no external runtime — just the
`curl` binary you already have and one env var.

## Setup

1. **Generate a Plane API key:**
   - Open https://plane.rnm.dev
   - Click your avatar → **Settings** → **API Tokens**
   - Click **Add API token**, name it `claude-code`, copy the token
2. **Export it** in your shell profile (`~/.zshrc` or `~/.bashrc`):
   ```bash
   export PLANE_API_KEY="plane_api_..."
   ```
   Reload your shell or `source ~/.zshrc`.

The workspace slug (`rnm`) and base URL (`https://plane.rnm.dev`) are baked
into the skill — only `PLANE_API_KEY` is per-user.

## Install

```
/plugin install plane@rnm-plugins
```

## Usage

Just ask in plain English. Claude will pick up the skill automatically:

> "List my open work items in the WEB project"
>
> "Create a Plane task: fix the login redirect bug, high priority"
>
> "Mark WEB-123 as done and leave a comment that it shipped"

Or invoke the skill explicitly:

```
/plane:tasks list everything assigned to me in the Mobile project
```

## What's inside

```
plane/
├── .claude-plugin/plugin.json
├── README.md
└── skills/tasks/SKILL.md   # API reference + curl recipes
```

The `SKILL.md` covers projects, work items, comments, states, labels,
cycles, modules, and workspace members. For anything more exotic, Claude
falls back to the [official API docs](https://developers.plane.so/api-reference/introduction).

## Rotating the API key

Generate a new token in Plane, update `PLANE_API_KEY` in your shell profile,
restart Claude Code. Revoke the old token from the Plane API Tokens page.
