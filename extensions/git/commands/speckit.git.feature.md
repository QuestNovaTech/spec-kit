---
description: "Generate a feature identifier and spec directory without creating a git branch"
---

# Generate Feature Identifier

Generate a feature identifier (directory name prefix) for the given specification. This command **does not create branches** — it only generates a feature number/name and creates the spec directory structure. All work happens directly on the current branch (e.g., `main`).

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Environment Variable Override

If the user explicitly provided `GIT_BRANCH_NAME` (e.g., via environment variable, argument, or in their request), use it as the feature identifier. When `GIT_BRANCH_NAME` is set:
- The script uses the exact value as the feature identifier
- `--short-name`, `--number`, and `--timestamp` flags are ignored
- `FEATURE_NUM` is extracted from the name if it starts with a numeric prefix, otherwise set to the full identifier

## Prerequisites

- Verify Git is available by running `git rev-parse --is-inside-work-tree 2>/dev/null`
- If Git is not available, warn the user and skip

## Numbering Mode

Determine the numbering strategy by checking configuration in this order:

1. Check `.specify/extensions/git/git-config.yml` for `branch_numbering` value
2. Check `.specify/init-options.json` for `branch_numbering` value (backward compatibility)
3. Default to `sequential` if neither exists

## Execution

Generate a concise short name (2-4 words) for the feature:
- Analyze the feature description and extract the most meaningful keywords
- Use action-noun format when possible (e.g., "add-user-auth", "fix-payment-bug")
- Preserve technical terms and acronyms (OAuth2, API, JWT, etc.)

Run the appropriate script based on your platform:

- **Bash**: `.specify/extensions/git/scripts/bash/create-new-feature.sh --json --short-name "<short-name>" "<feature description>"`
- **Bash (timestamp)**: `.specify/extensions/git/scripts/bash/create-new-feature.sh --json --timestamp --short-name "<short-name>" "<feature description>"`
- **PowerShell**: `.specify/extensions/git/scripts/powershell/create-new-feature.ps1 -Json -ShortName "<short-name>" "<feature description>"`
- **PowerShell (timestamp)**: `.specify/extensions/git/scripts/powershell/create-new-feature.ps1 -Json -Timestamp -ShortName "<short-name>" "<feature description>"`

**IMPORTANT**:
- Do NOT pass `--number` — the script determines the correct next number automatically
- Always include the JSON flag (`--json` for Bash, `-Json` for PowerShell) so the output can be parsed reliably
- You must only ever run this script once per feature
- The JSON output will contain `FEATURE_NUM` and `BRANCH_NAME` (which is now just a feature identifier, not a git branch)

## Graceful Degradation

If Git is not installed or the current directory is not a Git repository:
- Feature identifier generation continues with a warning: `[specify] Warning: Git repository not detected; skipped git checks`
- The script still outputs `BRANCH_NAME` and `FEATURE_NUM` so the caller can reference them

## Output

The script outputs JSON with:
- `BRANCH_NAME`: The feature identifier (e.g., `003-user-auth` or `20260319-143022-user-auth`) — used as directory name, not a branch
- `FEATURE_NUM`: The numeric or timestamp prefix used
- `SPEC_FILE`: Path to the spec file that was created
- `HAS_GIT`: Whether git was detected
