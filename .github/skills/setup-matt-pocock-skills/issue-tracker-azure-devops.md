# Issue tracker: Azure DevOps

Issues and specs for this repo live as Azure DevOps **work items** in Azure Boards. Use the [`az`](https://learn.microsoft.com/cli/azure/) CLI with the `azure-devops` extension for all operations. If the CLI is unavailable, fall back to the `azure-devops` remote MCP server (see [MCP fallback](#mcp-fallback)).

**Organization:** `https://dev.azure.com/<org>` · **Project:** `<project>` · **Work item type for tickets:** `<User Story | Product Backlog Item | Issue | Task>`

## Setup check

Run once per session before the first write:

- `az extension show --name azure-devops` (install with `az extension add --name azure-devops`)
- `az devops configure --defaults organization=https://dev.azure.com/<org> project=<project>`
- `az account show` must succeed (`az login` otherwise)

If any of these fail and can't be fixed, use the MCP fallback.

## Conventions

- **Create a work item**: `az boards work-item create --type "<type>" --title "..." --description "..." [--fields "System.Tags=<tag1>; <tag2>"]`. The description is HTML: convert Markdown to HTML (or wrap it in `<pre>`) before passing it. For long bodies, write the HTML to a temp file and pass `--description "$(cat <file>)"`.
- **Read a work item**: `az boards work-item show --id <id> --expand all -o json`. Comments: `az devops invoke --area wit --resource comments --route-parameters project=<project> workItemId=<id> --api-version 7.1-preview -o json`.
- **List / query work items**: `az boards query --wiql "SELECT [System.Id],[System.Title],[System.State],[System.Tags] FROM WorkItems WHERE [System.TeamProject] = @project AND [System.State] <> 'Closed' AND [System.Tags] CONTAINS '<tag>'" -o json`.
- **Comment on a work item**: `az boards work-item update --id <id> --discussion "..."`.
- **Apply / remove labels**: Azure DevOps uses **tags** as labels. Read the current `System.Tags`, then write the full new list: `az boards work-item update --id <id> --fields "System.Tags=<tag1>; <tag2>"`.
- **Close**: post the explanation with `--discussion`, then `az boards work-item update --id <id> --state "<Closed | Done | Removed>"` (the state name depends on the process template; check with `az boards work-item show`).
- **Parent / child**: `az boards work-item relation add --id <child> --relation-type parent --target-id <parent>`.
- **Blocking**: native **Predecessor/Successor** links. The blocker is the predecessor: `az boards work-item relation add --id <blocked> --relation-type predecessor --target-id <blocker>`. A work item is unblocked when every predecessor is closed.
- **Pull requests**: `az repos pr create --title "..." --description "..." --source-branch <branch> --target-branch main --work-items <id> [<id> ...]`. Use `az repos pr show`, `az repos pr list`, and `az repos pr update` for the rest. Linking with `--work-items` lets a completed PR resolve the work items.

Infer the org and project from `git remote -v` (`https://dev.azure.com/<org>/<project>/_git/<repo>` or `https://<org>.visualstudio.com/<project>/_git/<repo>`).

## Pull requests as a triage surface

**PRs as a request surface: no.** _(Set to `yes` if this repo treats external PRs as feature requests; `/triage` reads this flag.)_

When set to `yes`, PRs run through the same tags and states as work items, using `az repos pr list --status active -o json`, `az repos pr show --id <id>`, and `az repos pr update`. PR labels are managed with `az devops invoke --area git --resource pullRequestLabels`.

Work item IDs and PR IDs are separate number spaces, so always say which one you mean (`AB#123` for a work item, `!45` for a PR).

## When a skill says "publish to the issue tracker"

Create a work item of the configured type.

## When a skill says "fetch the relevant ticket"

Run `az boards work-item show --id <id> --expand all` and read its comments.

## Wayfinding operations

Used by `/wayfinder` (not installed in this repo; available upstream). The **map** is a parent work item with **child** work items as tickets.

- **Map**: a work item tagged `wayfinder:map`, holding the Notes / Decisions-so-far / Fog in its description.
- **Child ticket**: a work item linked to the map with `--relation-type parent`, tagged `wayfinder:<type>`.
- **Blocking**: Predecessor links as above.
- **Frontier query**: WIQL over the map's children, dropping any with an open predecessor or an assignee.
- **Claim**: `az boards work-item update --id <n> --assigned-to "<me>"`.
- **Resolve**: `--discussion "<answer>"`, then close, then append a context pointer to the map's Decisions-so-far.

## MCP fallback

Use this when `az`, the `azure-devops` extension, or `az login` is unavailable and can't be fixed. The `azure-devops` remote MCP server is configured in `.mcp.json` / `.vscode/mcp.json` (`https://mcp.dev.azure.com/<org>`, Microsoft Entra ID sign-in). Pass the project name on each call. Tool names follow the Azure DevOps MCP toolset and may change; if a name below isn't listed, pick the matching tool from the server's tool list.

| Operation | CLI | MCP tool (action) |
| --- | --- | --- |
| Create a work item | `az boards work-item create` | `wit_work_item_write` (`create`) |
| Read a work item | `az boards work-item show` | `wit_work_item` (`get`), comments via `wit_work_item` (`list_comments`) |
| List / query | `az boards query --wiql` | `wit_query` (`wiql`), or `search_workitem` for text search |
| Comment | `update --discussion` | `wit_work_item_comment_write` (`add`) |
| Tags / state | `az boards work-item update --fields` / `--state` | `wit_work_item_write` (`update`) |
| Parent / child | `relation add --relation-type parent` | `wit_work_item_write` (`add_child`) or `wit_work_item_link_write` (`link`, type `parent`) |
| Blocking | `relation add --relation-type predecessor` | `wit_work_item_link_write` (`link`, type `predecessor`) |
| Create a PR | `az repos pr create` | `repo_pull_request_write` (`create`), then `wit_work_item_link_write` (`link_to_pull_request`) |
| Read a PR | `az repos pr show` | `repo_pull_request` (`get`), threads via `repo_pull_request_thread` |

If neither the CLI nor the MCP server is available, stop and tell the user which one to set up. Don't silently switch to a different tracker.
