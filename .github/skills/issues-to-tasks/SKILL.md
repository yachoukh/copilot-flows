---
name: issues-to-tasks
description: Generate a TASKS.md tracking file from Azure DevOps work items linked to a PRD Epic. Creates a table with work item URLs, status, and a dependency graph. Use when user wants to create a task tracker, generate TASKS.md, or track work item progress.
---

# Work Items to TASKS.md

Generate a `TASKS.md` tracking file from Azure DevOps work items created for a PRD Epic. Produces a status table with linked work item URLs and a visual dependency graph.

## Process

### 1. Locate the PRD and its child work items

Ask the user for the PRD Epic work item ID.

Fetch all child User Story work items linked to the Epic using the `mcp_ado_wit_query_by_wiql` tool with a WIQL query:

```
SELECT [System.Id], [System.Title], [System.State]
FROM WorkItemLinks
WHERE ([Source].[System.Id] = <epic-id>)
  AND ([System.Links.LinkType] = 'System.LinkTypes.Hierarchy-Forward')
MODE (MustContain)
```

Then fetch each child work item's details (including relations) using `mcp_ado_wit_get_work_item` to get the full description and link data.

> **Fallback** (if MCP tools are unavailable):
> ```bash
> az boards query --wiql "SELECT [System.Id], [System.Title], [System.State] FROM WorkItems WHERE [System.Parent] = <epic-id>" --output json
> az boards work-item show --id <id> --output json
> ```

If the query returns no results, ask the user to confirm the correct project and Epic work item ID, or ask them to provide the work item IDs directly.

### 2. Extract dependency information

For each work item, extract from its **relations** (not the description body):

- **Blocked by**: predecessor/dependency links to other work items
- **User stories addressed**: from the "User stories addressed" section in the work item description
- **Status**: map Azure DevOps states to icons:
  - `Closed`, `Resolved`, `Done` → ✅
  - All other states (`New`, `Active`, etc.) → ❌

### 3. Build the tracking table

Create a Markdown table with these columns:

| Column | Description |
|--------|-------------|
| Work Item | Work item ID as a clickable link to the Azure DevOps URL (`https://dev.azure.com/{org}/{project}/_workitems/edit/{id}`) |
| Title | Work item title |
| Status | ✅ (Closed/Resolved/Done) or ❌ (all other states) |
| Blocked by | List of blocking work item IDs (as links) |
| User stories | User story numbers from the PRD |

Sort the table in dependency order — work items with no blockers first, then work items that depend on them, and so on.

### 4. Build the dependency graph

Create an ASCII dependency graph showing the relationships between work items. Use a format like:

```
#1001 (Project setup)
├── #1002 (Core module)
│   ├── #1004 (Feature A)
│   └── #1005 (Feature B)
└── #1003 (Config system)
    └── #1006 (Feature C)
```

Root nodes are work items with no blockers. Child nodes are work items blocked by the parent.

### 5. Write TASKS.md

Write the file to `./TASKS.md` in the repository root using the template below.

<tasks-template>
# Tasks

> Source PRD: [Epic #<epic-id>](https://dev.azure.com/{org}/{project}/_workitems/edit/<epic-id>)

## Progress

| Work Item | Title | Status | Blocked by | User stories |
|-----------|-------|--------|------------|--------------|
| [#1001](https://dev.azure.com/{org}/{project}/_workitems/edit/1001) | Project setup | ❌ | None | 1, 2 |
| [#1002](https://dev.azure.com/{org}/{project}/_workitems/edit/1002) | Core module | ❌ | [#1001](https://dev.azure.com/{org}/{project}/_workitems/edit/1001) | 3 |
| ... | ... | ... | ... | ... |

## Dependency Graph

```
#1001 (Project setup)
├── #1002 (Core module)
│   └── #1004 (Feature A)
└── #1003 (Config system)
```
</tasks-template>

### 6. Confirm with user

Show the user a summary of what was written:

- Total number of work items tracked
- How many are ✅ vs ❌
- The dependency graph

Ask if any adjustments are needed before finalizing.
