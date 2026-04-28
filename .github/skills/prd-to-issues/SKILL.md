---
name: prd-to-issues
description: Break a PRD into independently-grabbable Azure DevOps User Story work items using tracer-bullet vertical slices. Use when user wants to convert a PRD to work items, create implementation tickets, or break down a PRD into work items.
---

# PRD to Work Items

Break a PRD into independently-grabbable Azure DevOps User Story work items using vertical slices (tracer bullets).

## Process

### 1. Locate the PRD

Ask the user for the PRD Epic work item ID.

If the PRD is not already in your context window, fetch it using the `mcp_ado_wit_get_work_item` tool with the work item ID.

> **Fallback**: `az boards work-item show --id <id>`

### 2. Explore the codebase (optional)

If you have not already explored the codebase, do so to understand the current state of the code.

### 3. Draft vertical slices

Break the PRD into **tracer bullet** work items. Each work item is a thin vertical slice that cuts through ALL integration layers end-to-end, NOT a horizontal slice of one layer.

Slices may be 'HITL' or 'AFK'. HITL slices require human interaction, such as an architectural decision or a design review. AFK slices can be implemented and merged without human interaction. Prefer AFK over HITL where possible.

<vertical-slice-rules>
- Each slice delivers a narrow but COMPLETE path through every layer (schema, API, UI, tests)
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
</vertical-slice-rules>

### 4. Quiz the user

Present the proposed breakdown as a numbered list. For each slice, show:

- **Title**: short descriptive name
- **Type**: HITL / AFK
- **Blocked by**: which other slices (if any) must complete first
- **User stories covered**: which user stories from the PRD this addresses

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the dependency relationships correct?
- Should any slices be merged or split further?
- Are the correct slices marked as HITL and AFK?

Iterate until the user approves the breakdown.

### 5. Create the Azure DevOps work items

For each approved slice, create an Azure DevOps **User Story** work item. Create work items in dependency order (blockers first) so you can reference real work item IDs in dependency links.

For each work item:

1. **Create the User Story** using the `mcp_ado_wit_create_work_item` tool:
   - **type**: `User Story`
   - **title**: The slice title
   - **description**: The work item body (use the template below)

2. **Link to the parent Epic** using the `mcp_ado_wit_add_child_work_items` tool:
   - **parentId**: The PRD Epic work item ID
   - **childIds**: The newly created User Story ID

3. **Add dependency links** (if blocked by other slices) using the `mcp_ado_wit_work_items_link` tool:
   - Link the current work item to each blocker using a predecessor/successor relationship

> **Fallback** (if MCP tools are unavailable):
> ```bash
> az boards work-item create --type "User Story" --title "<title>" --description "<description>"
> az boards work-item relation add --id <child-id> --relation-type "System.LinkTypes.Hierarchy-Reverse" --target-id <epic-id>
> az boards work-item relation add --id <id> --relation-type "System.LinkTypes.Dependency-Reverse" --target-id <blocker-id>
> ```

<work-item-template>
## What to build

A concise description of this vertical slice. Describe the end-to-end behavior, not layer-by-layer implementation. Reference specific sections of the parent PRD Epic rather than duplicating content.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## User stories addressed

Reference by number from the parent PRD:

- User story 3
- User story 7

</work-item-template>

> **Note**: Parent Epic and dependency ("blocked by") relationships are captured as formal Azure DevOps work item links rather than text in the description body. This enables Azure DevOps to display the dependency graph natively in Boards.

Do NOT close or modify the parent PRD Epic.
