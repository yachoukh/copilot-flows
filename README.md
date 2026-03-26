# copilot-flows

Curated agentic workflows and skills for [GitHub Copilot CLI](https://docs.github.com/en/copilot/github-copilot-in-the-cli). Structured patterns for building apps end-to-end with AI agents, parallel execution, and reusable skills.

## Workflows

| Workflow | Description |
|----------|-------------|
| [Greenfield App Development](workflows/greenfield-workflow.md) | Build a complete application from a rough PRD — structured into 6 phases: PRD creation → issue breakdown → sequential foundation → parallel implementation → documentation → run & debug |

## Skills

This repo includes skills in [`.github/skills/`](.github/skills/) that are used by the workflows. Place them in your skills directory to use them with GitHub Copilot CLI.

| Skill | Description |
|-------|-------------|
| `/write-a-prd` | Interview → structured PRD with user stories, modules, config schema |
| `/prd-to-issues` | PRD → vertical-slice GitHub issues with dependency graph |
| `/issues-to-tasks` | GitHub issues → TASKS.md tracking file with status table and dependency graph |
| `/prd-to-plan` | PRD → multi-phase implementation plan saved as local Markdown |
| `/grill-me` | Stress-test a plan or design through relentless questioning |

### Skill Attribution

The `/write-a-prd`, `/prd-to-issues`, `/prd-to-plan`, and `/grill-me` skills are from [mattpocock/skills](https://github.com/mattpocock/skills) by [Matt Pocock](https://github.com/mattpocock). See that repo for the latest versions and additional skills. The `/issues-to-tasks` skill is original to this repo.

## Prerequisites

- **GitHub Copilot CLI** — installed and authenticated
- **GitHub CLI (`gh`)** — required for `/prd-to-issues`, `/issues-to-tasks`, and any workflow step that creates GitHub issues or pull requests. [Install guide](https://cli.github.com/)
- **Git** — with worktree support (for parallel implementation workflows)

## Installation

### 1. Install skills

Copy the skills from `.github/skills/` into your local skills directory:

```bash
# Find your skills directory (check your Copilot CLI config)
# Default: ~/.agents/skills/

cp -r .github/skills/* ~/.agents/skills/
```

Or point your Copilot CLI config to this repo's skills directory:

```json
{
  "skill_directories": [
    "/path/to/copilot-flows/.github/skills"
  ]
}
```

### 2. Follow a workflow

Open a terminal in your project directory and follow the prompts in any workflow file. Each phase includes the exact prompts to use.

## How It Works

The workflows combine three types of automation:

1. **Skills** (`/write-a-prd`, `/prd-to-issues`) — domain-specific instructions that guide the agent through structured processes like interviewing you for requirements or breaking down work into issues

2. **Agent types** — specialized agents launched for different tasks:
   - `explore` — research APIs, gather codebase context
   - `general-purpose` — implement features, resolve conflicts
   - `code-review` — review diffs, catch bugs before merge

3. **Parallel execution** — git worktrees let multiple agents work on the same repo simultaneously, with sequential merge and rebase to integrate

## Contributing

Have a workflow pattern that works well with Copilot CLI? Open a PR:

1. Add your workflow to `workflows/`
2. Add any required skills to `.github/skills/`
3. Include the prompts used (copyable) and describe what happens at each phase

## License

MIT
