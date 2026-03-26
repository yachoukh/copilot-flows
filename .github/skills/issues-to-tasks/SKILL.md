---
name: issues-to-tasks
description: Generate a TASKS.md tracking file from GitHub issues linked to a PRD. Creates a table with issue URLs, status, and a dependency graph. Use when user wants to create a task tracker, generate TASKS.md, or track issue progress.
---

# Issues to TASKS.md

Generate a `TASKS.md` tracking file from GitHub issues created for a PRD. Produces a status table with linked issue URLs and a visual dependency graph.

## Process

### 1. Locate the PRD and its issues

Ask the user for the PRD GitHub issue number (or URL).

Fetch the PRD issue with `gh issue view <number>`. Then list all issues that reference the PRD as their parent:

```bash
gh issue list --search "\"Parent PRD\" #<prd-number>" --json number,title,state,url --limit 100
```

If the search returns no results, ask the user to confirm the correct repository and PRD issue number, or ask them to provide the issue numbers directly.

### 2. Extract dependency information

For each issue, read the issue body to extract:

- **Blocked by**: which issues block this one (from the "Blocked by" section)
- **User stories addressed**: which user stories from the PRD this covers
- **Status**: open or closed (maps to ❌ or ✅)

### 3. Build the tracking table

Create a Markdown table with these columns:

| Column | Description |
|--------|-------------|
| Issue | Issue number as a clickable link to the GitHub issue URL |
| Title | Issue title |
| Status | ✅ (closed/merged) or ❌ (open/in progress) |
| Blocked by | List of blocking issue numbers (as links) |
| User stories | User story numbers from the PRD |

Sort the table in dependency order — issues with no blockers first, then issues that depend on them, and so on.

### 4. Build the dependency graph

Create an ASCII dependency graph showing the relationships between issues. Use a format like:

```
#1 (Project setup)
├── #2 (Core module)
│   ├── #4 (Feature A)
│   └── #5 (Feature B)
└── #3 (Config system)
    └── #6 (Feature C)
```

Root nodes are issues with no blockers. Child nodes are issues blocked by the parent.

### 5. Write TASKS.md

Write the file to `./TASKS.md` in the repository root using the template below.

<tasks-template>
# Tasks

> Source PRD: #<prd-issue-number>

## Progress

| Issue | Title | Status | Blocked by | User stories |
|-------|-------|--------|------------|--------------|
| [#1](<url>) | Project setup | ❌ | None | 1, 2 |
| [#2](<url>) | Core module | ❌ | [#1](<url>) | 3 |
| ... | ... | ... | ... | ... |

## Dependency Graph

```
#1 (Project setup)
├── #2 (Core module)
│   └── #4 (Feature A)
└── #3 (Config system)
```
</tasks-template>

### 6. Confirm with user

Show the user a summary of what was written:

- Total number of issues tracked
- How many are ✅ vs ❌
- The dependency graph

Ask if any adjustments are needed before finalizing.
