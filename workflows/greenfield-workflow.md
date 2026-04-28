# Greenfield App Development Workflow

A structured workflow for building a complete application from scratch using AI agents, skills, and parallel execution — from a rough PRD to a fully tested, documented, and packaged codebase. Uses Azure DevOps for work item tracking and pull request management.

---

## Phase 1: PRD Creation

**Goal**: Turn your rough PRD into a formal, structured Product Requirements Document.

**Skill**: `/write-a-prd`

**Prompt**:
```
improve and format this @PRD.md as a PRD
```

**What happens**:
- The skill interviews you with targeted questions (one at a time, multiple choice):
  - What problem does this solve?
  - Who is the target user?
  - Feature scope — what's in, what's out?
  - Module/component design validation
  - Test coverage expectations
  - Deployment scope
- Rewrites the document into a structured PRD:
  - Problem Statement & Solution
  - User Stories (numbered, persona-tagged)
  - Implementation Decisions (modules, architecture, config schema)
  - Testing Decisions (philosophy + per-module coverage)
  - Out of Scope
  - Appendix (reference material)
- Creates the PRD as an Azure DevOps **Epic** work item (via `mcp_ado_wit_create_work_item`)

---

## Phase 2: Break PRD into Work Items

**Goal**: Convert the PRD into independently-grabbable Azure DevOps User Story work items using vertical slices.

**Skill**: `/prd-to-issues`

**Prompt**:
```
break down the PRD into work items
```

**What happens**:
- Reads the PRD Epic and proposes tracer-bullet vertical slices
- Each slice cuts end-to-end through the stack
- Includes a dependency graph between work items
- Asks for confirmation on granularity, ordering, dependencies
- Creates Azure DevOps User Story work items (via `mcp_ado_wit_create_work_item`) with:
  - Parent link to the PRD Epic (via `mcp_ado_wit_add_child_work_items`)
  - "What to build" section with implementation details
  - Acceptance criteria checklist
  - Dependency links to blockers (via `mcp_ado_wit_work_items_link`)
  - User stories addressed

**Skill**: `/issues-to-tasks`

**Follow-up prompt** to create a tracking file:
```
generate TASKS.md from the PRD work items
```

---

## Phase 3: Sequential Implementation (Foundation)

**Goal**: Build the foundation work items that everything else depends on.

**Prompt**:
```
Now start implementation by looking at PRD.md and selecting a task from TASKS.md taking into account the dependency graph. Leverage /fleet whenever possible to do work in parallel. For each task create a git worktree. After completing the task create a pull request that should be reviewed (check acceptance criteria of the work item) by you before merging it.
```

**What happens for each work item**:

1. **Creates a git worktree** on a feature branch
2. **Launches an implementation agent** (`general-purpose`, background) with a detailed prompt containing:
   - Working directory and existing code context
   - Exact specifications (structs, function signatures, behavior)
   - Test requirements
   - Build/verify steps and commit message template
3. **Verifies** — runs linter + tests in the worktree
4. **Launches a code-review agent** against the diff to check acceptance criteria
5. **Fixes** any review findings (nil checks, validation gaps, etc.)
6. **Pushes and creates a pull request** via `mcp_ado_repo_create_pull_request`, then links the work item to the PR via `mcp_ado_wit_link_work_item_to_pull_request`
7. **Reviews the PR** via `mcp_ado_repo_get_pull_request_changes` and `mcp_ado_repo_vote_pull_request` (approve)
8. **Completes the PR** via `mcp_ado_repo_update_pull_request` (set status to `completed` with squash merge)
9. **Cleans up** — removes worktree, pulls main, updates TASKS.md (❌ → ✅)

Foundation work items are built sequentially since later work items depend on them.

---

## Phase 4: Parallel Feature Implementation

**Goal**: Implement all unblocked work items simultaneously using parallel agents.

**Key insight**: Once the foundation is merged, many work items have no dependencies on each other — only on `main`. Launch them all at once.

**What happens**:

1. **Creates all worktrees at once** (one per unblocked work item)
2. **Launches N implementation agents in parallel** — each works independently in its own worktree
3. **As agents complete**: verifies tests → pushes → creates PR (via `mcp_ado_repo_create_pull_request`) → links work item to PR (via `mcp_ado_wit_link_work_item_to_pull_request`)
4. **Merges sequentially with rebasing** — since parallel branches modify shared files:
   - Merge the least-conflict-prone first (e.g., work items that only add new files)
   - For subsequent PRs that conflict, launch an agent to rebase onto updated main, resolve conflicts (keeping both sides), and run tests
   - Force-push the rebased branch, then complete the PR (via `mcp_ado_repo_update_pull_request`)
5. **Repeats** for any remaining work items that were blocked on this batch

**Merge order strategy**: least conflicts → most conflicts (additive changes first, cross-cutting changes last).

---

## Phase 5: Documentation

**Goal**: Create user-facing and agent-facing documentation.

**Prompt**:
```
Create README.md to explain and create a CLAUDE.md for agentic development where we explain the stack, project structure, and key details which are needed to make agents work in the best way with this repo
```

**What happens**:
1. Launches an `explore` agent to exhaustively document every public API, config field, test count, etc.
2. Writes **README.md** (user-facing): quick start, config reference, features, development commands
3. Writes **CLAUDE.md** (agent-facing):
   - Project overview
   - Stack and dependencies
   - Project structure (tree with every file described)
   - Module dependency graph
   - Key conventions (config, error handling, concurrency, protocols)
   - Testing patterns and coverage
   - Build and run commands
   - Step-by-step recipes for common tasks (adding features, extending modules)

---

## Phase 6: Run and Debug

**Goal**: Build, run against a real target, and debug issues.

**Prompt** (example):
```
build and run using cli with this ip and port <host>, <port> use <N> clients
```

**What happens**:
1. Builds the binary
2. Creates a run config targeting the real server
3. Runs and observes logs
4. On errors: investigates with `explore` agents, writes debug scripts, analyzes output
5. Creates a plan (`plan.md`) with prioritized fixes

---

## Workflow Diagram

```
┌─────────────────────────────────────────────────┐
│  Phase 1: PRD (/write-a-prd)                    │
│  Raw notes → Structured PRD + Azure DevOps Epic  │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│  Phase 2: Work Items (/prd-to-issues)           │
│  PRD → vertical slices + dependency graph        │
│  + TASKS.md tracking file                        │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│  Phase 3: Sequential Foundation                  │
│  Critical-path work items built one at a time    │
│  (worktree → agent → review → merge)             │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│  Phase 4: Parallel Feature Implementation        │
│  N agents in parallel worktrees                  │
│  → verify → PR → sequential merge+rebase         │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│  Phase 5: Documentation                          │
│  explore agent → README.md + CLAUDE.md           │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│  Phase 6: Run & Debug                            │
│  Build → run → observe → investigate → plan      │
└─────────────────────────────────────────────────┘
```

## Key Patterns

| Pattern | Why |
|---------|-----|
| **Vertical slices, not horizontal layers** | Each work item delivers end-to-end value; enables parallel work |
| **Git worktrees for parallel branches** | Multiple agents work on the same repo simultaneously without conflicts |
| **Background agents for implementation** | Frees you to monitor, review, and manage while work happens |
| **Code review agents before merge** | Catches nil checks, validation gaps, missing error handling automatically |
| **Sequential merge with rebase** | Parallel branches touch shared files; merge one at a time and rebase the rest |
| **Explore agents for research** | Gather API details, library docs, or codebase context before acting |
| **TASKS.md as progress tracker** | Single source of truth for what's done, with linked Azure DevOps work items |
| **CLAUDE.md for agent context** | Gives future agents the conventions, structure, and recipes they need |
| **Detailed agent prompts** | Include existing code context, exact function signatures, and test specs |
| **Verify before merge** | Always run linter + tests on the worktree before creating PR |

## Skills Used

| Skill | Phase | Purpose |
|-------|-------|---------|
| `/write-a-prd` | 1 | Interview → structured PRD as Azure DevOps Epic with user stories, modules, config schema |
| `/prd-to-issues` | 2 | PRD → vertical-slice Azure DevOps User Stories with dependency graph |
| `/issues-to-tasks` | 2 | Azure DevOps work items → TASKS.md tracking file with status and dependency graph |

## Agent Types Used

| Agent | When | Purpose |
|-------|------|---------|
| `explore` | Before implementation, before docs | Research APIs, gather codebase details |
| `general-purpose` | Implementation | Build features in worktrees with detailed specs |
| `general-purpose` | Conflict resolution | Rebase branches, resolve merge conflicts, run tests |
| `code-review` | Before merge | Review diffs against acceptance criteria, find bugs |

## Azure DevOps MCP Tools Used

| Tool | When | Purpose |
|------|------|---------|
| `mcp_ado_wit_create_work_item` | Phase 1, 2 | Create Epic (PRD) and User Story (slice) work items |
| `mcp_ado_wit_get_work_item` | Phase 2, 3 | Fetch work item details and acceptance criteria |
| `mcp_ado_wit_add_child_work_items` | Phase 2 | Link User Stories as children of the PRD Epic |
| `mcp_ado_wit_work_items_link` | Phase 2 | Create dependency links between work items |
| `mcp_ado_wit_query_by_wiql` | Phase 2 | Query child work items of the Epic for TASKS.md |
| `mcp_ado_repo_create_pull_request` | Phase 3, 4 | Create pull requests from feature branches |
| `mcp_ado_repo_get_pull_request_changes` | Phase 3, 4 | Get PR diffs for code review |
| `mcp_ado_repo_vote_pull_request` | Phase 3, 4 | Approve PRs after review |
| `mcp_ado_repo_update_pull_request` | Phase 3, 4 | Complete (merge) PRs |
| `mcp_ado_wit_link_work_item_to_pull_request` | Phase 3, 4 | Link work items to their PRs for traceability |
