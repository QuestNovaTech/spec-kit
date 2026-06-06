---
description: Convert existing tasks into actionable issues. Supports multiple issue tracking systems with automatic fallback.
tools: ['github/github-mcp-server/issue_write']
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
  ps: scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Supported Issue Tracking Systems

This command supports multiple backends. Priority order:

1. **GitHub** (via `github/github-mcp-server/issue_write`) — default if available
2. **Linear** — if `linear-mcp-server` tool is available
3. **Jira** — if `jira-mcp-server` tool is available
4. **Fallback** — generate CSV/Markdown for manual import if no MCP tool available

**Backend selection**:
- If user passes `--backend=github|linear|jira`, use that
- Otherwise auto-detect based on available MCP tools
- If no MCP tool available, automatically use fallback mode

## Issue Metadata Mapping

Map task properties to issue fields:

| Task Property | GitHub Issue | Linear Issue | Jira Issue | Fallback CSV |
|---------------|--------------|--------------|------------|--------------|
| Task ID (T###) | Title prefix | Title prefix | Issue key prefix | ID column |
| Description | Body | Description | Description | Description column |
| [P] marker | Label: `parallel` | Label: `parallel` | Label: `parallel` | Parallel column |
| [US#] label | Label: `story-US#` | Label: `story-US#` | Component: `US#` | Story column |
| Priority (from spec) | Label: `priority-P#` | Priority field | Priority field | Priority column |
| File path | Body reference | Description reference | Description reference | File column |

## Pre-Execution Checks

**Check for extension hooks (before tasks-to-issues conversion)**:
- Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.before_taskstoissues` key
- If the YAML cannot be parsed or is invalid, skip hook checking silently and continue normally
- Filter out hooks where `enabled` is explicitly `false`. Treat hooks without an `enabled` field as enabled by default.
- For each remaining hook, do **not** attempt to interpret or evaluate hook `condition` expressions:
  - If the hook has no `condition` field, or it is null/empty, treat the hook as executable
  - If the hook defines a non-empty `condition`, skip the hook and leave condition evaluation to the HookExecutor implementation
- For each executable hook, output the following based on its `optional` flag:
  - **Optional hook** (`optional: true`):
    ```
    ## Extension Hooks

    **Optional Pre-Hook**: {extension}
    Command: `/{command}`
    Description: {description}

    Prompt: {prompt}
    To execute: `/{command}`
    ```
  - **Mandatory hook** (`optional: false`):
    ```
    ## Extension Hooks

    **Automatic Pre-Hook**: {extension}
    Executing: `/{command}`
    EXECUTE_COMMAND: {command}

    Wait for the result of the hook command before proceeding to the Outline.
    ```
- If no hooks are registered or `.specify/extensions.yml` does not exist, skip silently

## Outline

1. Run `{SCRIPT}` from repo root and parse FEATURE_DIR and AVAILABLE_DOCS list. All paths must be absolute. For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. **IF EXISTS**: Load `/memory/constitution.md` for project principles and governance constraints.

3. From the executed script, extract the path to **tasks**.

4. **Determine backend** (see Supported Issue Tracking Systems above).

5. **GitHub mode** (if selected):

   Get the Git remote by running:

   ```bash
   git config --get remote.origin.url
   ```

   > [!CAUTION]
   > ONLY PROCEED TO NEXT STEPS IF THE REMOTE IS A GITHUB URL

   For each task in the list, use the GitHub MCP server to create a new issue in the repository that is representative of the Git remote.

   > [!CAUTION]
   > UNDER NO CIRCUMSTANCES EVER CREATE ISSUES IN REPOSITORIES THAT DO NOT MATCH THE REMOTE URL

6. **Linear mode** (if selected):
   - Use `linear-mcp-server` to create issues
   - Map team/project from configuration or user input
   - Apply labels per metadata mapping table

7. **Jira mode** (if selected):
   - Use `jira-mcp-server` to create issues
   - Map project key from configuration or user input
   - Apply issue type (Task/Story) based on task content

8. **Fallback mode** (if no MCP available):
   - Generate `FEATURE_DIR/issues-export.csv` with columns:
     - `ID`, `Title`, `Description`, `Story`, `Parallel`, `Priority`, `FilePath`, `Status`
   - Also generate `FEATURE_DIR/issues-export.md` as a readable markdown table
   - Instruct user: "No issue tracking MCP tool detected. Generated export files for manual import."

## Error Handling

| Scenario | Action |
|----------|--------|
| MCP tool unavailable | Switch to fallback mode automatically |
| GitHub remote not found | Switch to fallback mode |
| Rate limit hit | Pause, report progress, suggest retry |
| Task parsing fails | Report specific task line, skip and continue |
| Partial success | Report created vs failed counts, list failures |

## Post-Execution Checks

**Check for extension hooks (after tasks-to-issues conversion)**:
Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.after_taskstoissues` key
- If the YAML cannot be parsed or is invalid, skip hook checking silently and continue normally
- Filter out hooks where `enabled` is explicitly `false`. Treat hooks without an `enabled` field as enabled by default.
- For each remaining hook, do **not** attempt to interpret or evaluate hook `condition` expressions:
  - If the hook has no `condition` field, or it is null/empty, treat the hook as executable
  - If the hook defines a non-empty `condition`, skip the hook and leave condition evaluation to the HookExecutor implementation
- For each executable hook, output the following based on its `optional` flag:
  - **Optional hook** (`optional: true`):
    ```
    ## Extension Hooks

    **Optional Hook**: {extension}
    Command: `/{command}`
    Description: {description}

    Prompt: {prompt}
    To execute: `/{command}`
    ```
  - **Mandatory hook** (`optional: false`):
    ```
    ## Extension Hooks

    **Automatic Hook**: {extension}
    Executing: `/{command}`
    EXECUTE_COMMAND: {command}
    ```
- If no hooks are registered or `.specify/extensions.yml` does not exist, skip silently

## Completion Report

Report:
- Backend used (GitHub / Linear / Jira / Fallback)
- Total tasks processed
- Issues created / exported
- Any failures with specific error messages
- Links to created issues (if applicable)
- Path to fallback export files (if applicable)
