---
description: Manage work items in RNM's Plane workspace at plane.rnm.dev. Use when the user wants to list, search, create, update, comment on, or close Plane issues/tasks, or browse Plane projects, cycles, modules, states, or labels.
---

# Plane task management

Interact with RNM's self-hosted Plane workspace via its REST API. All requests
go through `curl` against `https://plane.rnm.dev/api/v1`.

## Configuration

| Setting | Value |
| :--- | :--- |
| Base URL | `https://plane.rnm.dev/api/v1` |
| Workspace slug | `rnm` |
| Auth header | `X-API-Key: $PLANE_API_KEY` |
| Rate limit | 60 requests/minute |
| Pagination | Cursor-based via `cursor` query param |

**Before any request**, verify the API key is loaded:

```bash
test -n "$PLANE_API_KEY" || echo "PLANE_API_KEY is not set — see the plane plugin README"
```

If it's missing, stop and ask the user to set it. Do not prompt them for the
key inline — keys go in shell profile env vars, not the chat.

## Universal request pattern

Always use this curl shape so headers and base URL stay consistent:

```bash
curl -sS \
  -H "X-API-Key: $PLANE_API_KEY" \
  -H "Content-Type: application/json" \
  "https://plane.rnm.dev/api/v1/workspaces/rnm/<path>"
```

Pipe through `jq` for readable output and to extract IDs.

## Common workflows

### 1. Find a project ID

Most endpoints need a `project_id`. Resolve it from the project name first:

```bash
curl -sS -H "X-API-Key: $PLANE_API_KEY" \
  "https://plane.rnm.dev/api/v1/workspaces/rnm/projects/" \
  | jq '.results[] | {id, name, identifier}'
```

`identifier` is the short prefix shown in work-item IDs (e.g. `WEB` in `WEB-123`).

### 2. List work items in a project

```bash
curl -sS -H "X-API-Key: $PLANE_API_KEY" \
  "https://plane.rnm.dev/api/v1/workspaces/rnm/projects/$PROJECT_ID/work-items/" \
  | jq '.results[] | {id, sequence_id, name, state, priority, assignees}'
```

Useful query params: `?state=<state_id>`, `?priority=high`, `?assignees=<user_id>`,
`?fields=id,name,state`, `?expand=state,assignees,labels`.

### 3. Get a single work item

By UUID:

```bash
curl -sS -H "X-API-Key: $PLANE_API_KEY" \
  "https://plane.rnm.dev/api/v1/workspaces/rnm/projects/$PROJECT_ID/work-items/$ISSUE_ID/"
```

By short identifier (e.g. `WEB-123`):

```bash
curl -sS -H "X-API-Key: $PLANE_API_KEY" \
  "https://plane.rnm.dev/api/v1/workspaces/rnm/projects/$PROJECT_ID/work-items/identifier/WEB-123/"
```

### 4. Create a work item

Minimum body: `name`. Most useful extras: `description_html`, `priority`, `state`,
`assignees`, `labels`.

```bash
curl -sS -X POST \
  -H "X-API-Key: $PLANE_API_KEY" \
  -H "Content-Type: application/json" \
  "https://plane.rnm.dev/api/v1/workspaces/rnm/projects/$PROJECT_ID/work-items/" \
  -d '{
    "name": "Fix login redirect bug",
    "description_html": "<p>Users land on /home instead of the intended URL.</p>",
    "priority": "high"
  }'
```

`priority` values: `urgent`, `high`, `medium`, `low`, `none`.

To set `state`, `assignees`, or `labels`, you need their UUIDs — fetch them
first from the endpoints in the [reference](#endpoint-reference) below.

### 5. Update a work item

PATCH only the fields that change:

```bash
curl -sS -X PATCH \
  -H "X-API-Key: $PLANE_API_KEY" \
  -H "Content-Type: application/json" \
  "https://plane.rnm.dev/api/v1/workspaces/rnm/projects/$PROJECT_ID/work-items/$ISSUE_ID/" \
  -d '{"priority": "urgent"}'
```

To close an item, PATCH `state` to the UUID of a "Completed" or "Cancelled"
state (fetch via `GET /states/`).

### 6. Comment on a work item

```bash
curl -sS -X POST \
  -H "X-API-Key: $PLANE_API_KEY" \
  -H "Content-Type: application/json" \
  "https://plane.rnm.dev/api/v1/workspaces/rnm/projects/$PROJECT_ID/work-items/$ISSUE_ID/issue-comments/" \
  -d '{"comment_html": "<p>Picked this up, ETA tomorrow EOD.</p>"}'
```

## Endpoint reference

Replace `{ws}` with `rnm` and `{pid}` with the project UUID.

### Projects
- `GET /workspaces/{ws}/projects/` — list
- `GET /workspaces/{ws}/projects/{pid}/` — retrieve
- `POST /workspaces/{ws}/projects/` — create
- `PATCH /workspaces/{ws}/projects/{pid}/` — update

### Work items
- `GET /workspaces/{ws}/projects/{pid}/work-items/` — list
- `GET /workspaces/{ws}/projects/{pid}/work-items/{id}/` — retrieve
- `GET /workspaces/{ws}/projects/{pid}/work-items/identifier/{short_id}/` — retrieve by `PROJ-123` form
- `POST /workspaces/{ws}/projects/{pid}/work-items/` — create
- `PATCH /workspaces/{ws}/projects/{pid}/work-items/{id}/` — update
- `DELETE /workspaces/{ws}/projects/{pid}/work-items/{id}/` — delete
- `GET /workspaces/{ws}/projects/{pid}/work-items/search/?search=<query>` — search

### Comments
- `GET /workspaces/{ws}/projects/{pid}/work-items/{id}/issue-comments/` — list
- `POST /workspaces/{ws}/projects/{pid}/work-items/{id}/issue-comments/` — add
- `PATCH /workspaces/{ws}/projects/{pid}/work-items/{id}/issue-comments/{cid}/` — edit
- `DELETE /workspaces/{ws}/projects/{pid}/work-items/{id}/issue-comments/{cid}/` — delete

### Lookup tables (needed for filters & creation)
- `GET /workspaces/{ws}/projects/{pid}/states/` — workflow states
- `GET /workspaces/{ws}/projects/{pid}/labels/` — labels
- `GET /workspaces/{ws}/projects/{pid}/cycles/` — cycles (sprints)
- `GET /workspaces/{ws}/projects/{pid}/modules/` — modules (epics)
- `GET /workspaces/{ws}/members/` — workspace members (for assignee UUIDs)

### Cycles & modules — adding items
- `POST /workspaces/{ws}/projects/{pid}/cycles/{cycle_id}/cycle-issues/` — add issues to cycle, body: `{"issues": ["<issue_id>", ...]}`
- `POST /workspaces/{ws}/projects/{pid}/modules/{module_id}/module-issues/` — add issues to module

## Output formatting

When reporting results to the user:

- Use short identifiers (`WEB-123`) over UUIDs whenever they're available
- For lists of >5 items, render as a markdown table with the columns the user
  asked for (default: `id | name | state | priority | assignee`)
- For single items, include a clickable URL:
  `https://plane.rnm.dev/{workspace_slug}/projects/{project_id}/work-items/{issue_id}/`

## Error handling

- `401 Unauthorized` → API key is missing or revoked. Stop and tell the user.
- `403 Forbidden` → token doesn't have access to that workspace/project.
- `404 Not Found` → resource UUID is wrong, or the user used a project
  identifier where a UUID is required.
- `429 Too Many Requests` → wait 60 seconds (rate limit is per minute).

Never invent UUIDs. If you don't have one, fetch it first via the relevant
list endpoint.

## Full API docs

For anything not covered here: https://developers.plane.so/api-reference/introduction
