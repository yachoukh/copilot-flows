# copilot-flows

Curated agentic workflows and skills for [GitHub Copilot CLI](https://docs.github.com/en/copilot/github-copilot-in-the-cli). They take you from an idea to agent-implemented, reviewed code, with **GitHub** or **Azure DevOps** as the issue tracker.

## Workflows

| Workflow | Use when |
| --- | --- |
| [Greenfield](workflows/greenfield-workflow.md) | New project or feature: idea → `/grill-with-docs` → `/to-spec` → `/to-tickets` → `/implement-spec` → `/retro` |
| [Brownfield](workflows/brownfield-workflow.md) | Existing codebase: the same main flow plus on-ramps for `/triage`, `/diagnosing-bugs`, and `/improve-codebase-architecture` |

## Skills

Skills live in [`.github/skills/`](.github/skills/).

| Skill | Purpose |
| --- | --- |
| `/setup-matt-pocock-skills` | Once per repo: pick the issue tracker (GitHub, Azure DevOps, GitLab, local), triage labels, doc layout |
| `/grill-with-docs` | Interview to shared understanding; maintains `GLOSSARY.md` and ADRs |
| `/grill-me` | Same interview, no files written (non-code or no repo) |
| `/to-spec` | Conversation → spec published to the tracker |
| `/to-tickets` | Spec → tracer-bullet tickets with native blocking links |
| `/implement-spec` | Implement a whole spec with parallel subagents in worktrees on one integration branch |
| `/implement` | Implement a single ticket or small change |
| `/triage` | Move incoming issues through triage labels |
| `/diagnosing-bugs` | Feedback-loop-first debugging |
| `/improve-codebase-architecture` | Survey for deepening opportunities (HTML report) |
| `/retro` | Suggest improvements to the agent environment after a session |

Supporting skills that other skills invoke: `grilling`, `domain-modeling`, `codebase-design`, `tdd`, `code-review`, `pr`, `writing-for-agents`.

### Attribution

All skills come from [mattpocock/skills](https://github.com/mattpocock/skills) by [Matt Pocock](https://github.com/mattpocock) (MIT), imported unchanged at commit [`d81f3a1`](https://github.com/mattpocock/skills/tree/d81f3a183412e71a5b1e84ca21bc1a35eea03a60). The one local change is to `setup-matt-pocock-skills`: it adds an Azure DevOps tracker (`issue-tracker-azure-devops.md`) and MCP fallback sections to the GitHub and Azure DevOps tracker templates.

## Issue trackers: CLI first, MCP fallback

`/setup-matt-pocock-skills` writes `docs/agents/issue-tracker.md` in your repo, and every skill that touches issues reads it.

| Tracker | Primary | Fallback (remote MCP server) |
| --- | --- | --- |
| GitHub | [`gh`](https://cli.github.com/) CLI | `github` → `https://api.githubcopilot.com/mcp/` |
| Azure DevOps | [`az`](https://learn.microsoft.com/cli/azure/) + `azure-devops` extension (`az boards`, `az repos`) | `azure-devops` → `https://mcp.dev.azure.com/{organization}` |

Both servers are configured in [`.mcp.json`](.mcp.json) (Copilot CLI) and [`.vscode/mcp.json`](.vscode/mcp.json) (VS Code). Copy them into your project and:

- Replace `{organization}` with your Azure DevOps organization. The server requires a Microsoft Entra ID–backed org and signs in with OAuth.
- Remove the server you don't need.
- Adjust `X-MCP-Toolsets` (default `wit,repos`) if you want more Azure DevOps tools.

## Prerequisites

- **GitHub Copilot CLI**, installed and authenticated
- **Git**, with worktree support (used by `/implement-spec`)
- For your tracker, either the CLI or the MCP server:
  - GitHub: `gh auth login`
  - Azure DevOps: `az extension add --name azure-devops`, `az login`, `az devops configure --defaults organization=https://dev.azure.com/<org> project=<project>`

## Installation

1. Copy `.github/skills/` into your project (or your user skills directory, e.g. `~/.copilot/skills/`), and copy `.mcp.json` into your project root.
2. In your project, run `/setup-matt-pocock-skills`.
3. Follow a [workflow](#workflows).

## Updating skills from upstream

Re-import from [mattpocock/skills](https://github.com/mattpocock/skills) (skip `agents/openai.yaml`), then re-apply the Azure DevOps and MCP fallback changes in `setup-matt-pocock-skills`, and update the commit SHA above. Review the upstream `CHANGELOG.md` for renamed or removed skills.

## Contributing

Have a workflow pattern that works well with Copilot CLI? Open a PR that adds it to `workflows/`, adds any required skills to `.github/skills/`, and includes the exact prompts used.

## License

MIT
