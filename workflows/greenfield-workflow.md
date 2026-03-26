# Greenfield App Development Workflow

A structured workflow for building a complete application from scratch using AI agents, skills, and parallel execution — from a rough PRD to a fully tested, documented, and packaged codebase.

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
- Optionally submits as a GitHub issue

---

## Phase 2: Break PRD into Issues

**Goal**: Convert the PRD into independently-grabbable GitHub issues using vertical slices.

**Skill**: `/prd-to-issues`

**Prompt**:
```
break down the PRD into issues
```

**What happens**:
- Reads the PRD and proposes tracer-bullet vertical slices
- Each slice cuts end-to-end through the stack
- Includes a dependency graph between issues
- Asks for confirmation on granularity, ordering, dependencies
- Creates GitHub issues with:
  - Parent PRD reference
  - "What to build" section with implementation details
  - Acceptance criteria checklist
  - "Blocked by" dependencies
  - User stories addressed

**Follow-up prompt** to create a tracking file:
```
Add this table and dependency graph to a file called TASKS.md — the issue number column should have a URL to the GitHub issue, also add a column indicating whether implemented or not
```

---

## Phase 3: Sequential Implementation (Foundation)

**Goal**: Build the foundation issues that everything else depends on.

**Prompt**:
```
Now start implementation by looking at PRD.md and selecting a task from TASKS.md taking into account the dependency graph. Leverage /fleet whenever possible to do work in parallel. For each task create a git worktree. After completing the task create a pull request that should be reviewed (check acceptance criteria of issue) by you before merging it.
```

**What happens for each issue**:

1. **Creates a git worktree** on a feature branch
2. **Launches an implementation agent** (`general-purpose`, background) with a detailed prompt containing:
   - Working directory and existing code context
   - Exact specifications (structs, function signatures, behavior)
   - Test requirements
   - Build/verify steps and commit message template
3. **Verifies** — runs linter + tests in the worktree
4. **Launches a code-review agent** against the diff to check acceptance criteria
5. **Fixes** any review findings (nil checks, validation gaps, etc.)
6. **Pushes, creates PR, merges** (squash + delete branch)
7. **Cleans up** — removes worktree, pulls main, updates TASKS.md (❌ → ✅)

Foundation issues are built sequentially since later issues depend on them.

---

## Phase 4: Parallel Feature Implementation

**Goal**: Implement all unblocked issues simultaneously using parallel agents.

**Key insight**: Once the foundation is merged, many issues have no dependencies on each other — only on `main`. Launch them all at once.

**What happens**:

1. **Creates all worktrees at once** (one per unblocked issue)
2. **Launches N implementation agents in parallel** — each works independently in its own worktree
3. **As agents complete**: verifies tests → pushes → creates PR
4. **Merges sequentially with rebasing** — since parallel branches modify shared files:
   - Merge the least-conflict-prone first (e.g., issues that only add new files)
   - For subsequent PRs that conflict, launch an agent to rebase onto updated main, resolve conflicts (keeping both sides), and run tests
   - Force-push the rebased branch, then merge
5. **Repeats** for any remaining issues that were blocked on this batch

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
┌─────────────────────────────────────────────┐
│  Phase 1: PRD (/write-a-prd)                │
│  Raw notes → Structured PRD + GitHub issue   │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│  Phase 2: Issues (/prd-to-issues)           │
│  PRD → vertical slices + dependency graph    │
│  + TASKS.md tracking file                    │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│  Phase 3: Sequential Foundation              │
│  Critical-path issues built one at a time    │
│  (worktree → agent → review → merge)         │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│  Phase 4: Parallel Feature Implementation    │
│  N agents in parallel worktrees              │
│  → verify → PR → sequential merge+rebase     │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│  Phase 5: Documentation                      │
│  explore agent → README.md + CLAUDE.md       │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│  Phase 6: Run & Debug                        │
│  Build → run → observe → investigate → plan  │
└─────────────────────────────────────────────┘
```

## Key Patterns

| Pattern | Why |
|---------|-----|
| **Vertical slices, not horizontal layers** | Each issue delivers end-to-end value; enables parallel work |
| **Git worktrees for parallel branches** | Multiple agents work on the same repo simultaneously without conflicts |
| **Background agents for implementation** | Frees you to monitor, review, and manage while work happens |
| **Code review agents before merge** | Catches nil checks, validation gaps, missing error handling automatically |
| **Sequential merge with rebase** | Parallel branches touch shared files; merge one at a time and rebase the rest |
| **Explore agents for research** | Gather API details, library docs, or codebase context before acting |
| **TASKS.md as progress tracker** | Single source of truth for what's done, with linked GitHub issues |
| **CLAUDE.md for agent context** | Gives future agents the conventions, structure, and recipes they need |
| **Detailed agent prompts** | Include existing code context, exact function signatures, and test specs |
| **Verify before merge** | Always run linter + tests on the worktree before creating PR |

## Skills Used

| Skill | Phase | Purpose |
|-------|-------|---------|
| `/write-a-prd` | 1 | Interview → structured PRD with user stories, modules, config schema |
| `/prd-to-issues` | 2 | PRD → vertical-slice GitHub issues with dependency graph |

## Agent Types Used

| Agent | When | Purpose |
|-------|------|---------|
| `explore` | Before implementation, before docs | Research APIs, gather codebase details |
| `general-purpose` | Implementation | Build features in worktrees with detailed specs |
| `general-purpose` | Conflict resolution | Rebase branches, resolve merge conflicts, run tests |
| `code-review` | Before merge | Review diffs against acceptance criteria, find bugs |
