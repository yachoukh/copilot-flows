# copilot-flows

Curated agentic workflows and skills for [GitHub Copilot](https://docs.github.com/en/copilot) with **Azure DevOps** integration. Structured patterns for building apps end-to-end with AI agents, parallel execution, and reusable skills — using Azure DevOps for work item tracking, pull requests, and boards.

## Workflows

| Workflow | Description |
|----------|-------------|
| [Greenfield App Development](workflows/greenfield-workflow.md) | Build a complete application from a rough PRD — structured into 6 phases: PRD creation → work item breakdown → sequential foundation → parallel implementation → documentation → run & debug |

## Skills

This repo includes skills in [`.github/skills/`](.github/skills/) that are used by the workflows. Place them in your skills directory to use them with GitHub Copilot.

| Skill | Description |
|-------|-------------|
| `/write-a-prd` | Interview → structured PRD created as an Azure DevOps Epic |
| `/prd-to-issues` | PRD → vertical-slice Azure DevOps User Stories with dependency graph |
| `/issues-to-tasks` | Azure DevOps work items → TASKS.md tracking file with status table and dependency graph |
| `/prd-to-plan` | PRD → multi-phase implementation plan saved as local Markdown |
| `/grill-me` | Stress-test a plan or design through relentless questioning |

### Skill Attribution

The `/write-a-prd`, `/prd-to-issues`, `/prd-to-plan`, and `/grill-me` skills are adapted from [mattpocock/skills](https://github.com/mattpocock/skills) by [Matt Pocock](https://github.com/mattpocock) (originally GitHub-oriented). See that repo for the original versions and additional skills. The `/issues-to-tasks` skill is original to this repo. All skills in this branch have been rewritten to use Azure DevOps.

## Prerequisites

- **GitHub Copilot** — installed and authenticated
- **Azure DevOps MCP Server** — the [Azure DevOps MCP](https://github.com/microsoft/azure-devops-mcp) server provides the primary integration for creating work items, managing pull requests, and querying boards. Enable it in your VS Code MCP settings.
- **Azure DevOps organization and project** — you need an Azure DevOps organization and project where work items and repos will be managed
- **Azure CLI (`az`)** with the devops extension (fallback) — required if the MCP server is unavailable. [Install Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli), then add the extension:
  ```bash
  az extension add --name azure-devops
  az devops configure --defaults organization=https://dev.azure.com/{your-org} project={your-project}
  ```
- **Git** — with worktree support (for parallel implementation workflows)

## Installation

### 1. Install skills

Copy the skills from `.github/skills/` into your local skills directory:

```bash
# Find your skills directory (check your Copilot config)
# Default: ~/.agents/skills/

cp -r .github/skills/* ~/.agents/skills/
```

Or point your Copilot config to this repo's skills directory:

```json
{
  "skill_directories": [
    "/path/to/copilot-flows/.github/skills"
  ]
}
```

### 2. Configure Azure DevOps MCP

Ensure the Azure DevOps MCP server is enabled and authenticated. The skills use MCP tools like `mcp_ado_wit_create_work_item`, `mcp_ado_repo_create_pull_request`, etc. to interact with Azure DevOps.

### 3. Follow a workflow

Open a terminal in your project directory and follow the prompts in any workflow file. Each phase includes the exact prompts to use.

## How It Works

The workflows combine three types of automation:

1. **Skills** (`/write-a-prd`, `/prd-to-issues`) — domain-specific instructions that guide the agent through structured processes like interviewing you for requirements or breaking down work into Azure DevOps work items

2. **Agent types** — specialized agents launched for different tasks:
   - `explore` — research APIs, gather codebase context
   - `general-purpose` — implement features, resolve conflicts
   - `code-review` — review diffs, catch bugs before merge

3. **Parallel execution** — git worktrees let multiple agents work on the same repo simultaneously, with sequential merge and rebase to integrate

4. **Azure DevOps MCP** — the skills use Azure DevOps MCP tools to create Epics, User Stories, pull requests, and dependency links directly in Azure DevOps Boards and Repos

## Contributing

Have a workflow pattern that works well? Open a pull request:

1. Add your workflow to `workflows/`
2. Add any required skills to `.github/skills/`
3. Include the prompts used (copyable) and describe what happens at each phase

## License

MIT
