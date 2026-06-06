---
description: Conduct a post-implementation retrospective to collect feedback on spec accuracy, plan quality, and process improvements. Writes findings to `.specify/memory/retro/` for future reference.
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
  ps: scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Purpose

The retrospective captures learnings from the completed feature to continuously improve the Spec Kit workflow. It answers:

- Was the spec accurate? Did it miss anything critical?
- Was the plan realistic? Were estimates close?
- Were tasks complete? Anything missing or unnecessary?
- How can the process itself be improved?

## When to Run

- **Automatically suggested** by `speckit.implement` after all tasks are marked complete
- **Manually invoked** by the user after feature completion or deployment
- **Optional** — user can skip with `--skip-retro` flag

## Pre-Execution Checks

**Check for extension hooks (before retro)**:
- Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.before_retro` key
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

2. **Gather quantitative metrics** from feature artifacts:

   | Metric | Source | How to calculate |
   |--------|--------|------------------|
   | Spec version count | spec.md revision history | Count `[NEEDS CLARIFICATION]` resolutions + manual edits |
   | Plan complexity level | plan.md | Read Complexity Assessment section |
   | Total tasks | tasks.md | Count `- [ ]` / `- [x]` lines |
   | Tasks added mid-flight | tasks.md vs original | Compare initial task count to final |
   | Tasks removed | tasks.md history | Count deleted tasks |
   | Checklists passed | checklists/ | Count files with 0 incomplete items |
   | Checklists failed | checklists/ | Count files with >0 incomplete items |

3. **Prompt user for qualitative feedback** (interactive):

   Present each question with suggested answer options. User can select an option or provide free-form text.

   **Q1: Spec Accuracy**
   > How accurate was the initial spec?
   | Option | Description |
   |--------|-------------|
   | A | Highly accurate — no significant gaps |
   | B | Mostly accurate — minor gaps found during implementation |
   | C | Significant gaps — required spec updates mid-implementation |
   | D | Poor — spec was misleading or wrong about key aspects |

   **Q2: Plan Realism**
   > How realistic was the implementation plan?
   | Option | Description |
   |--------|-------------|
   | A | Very realistic — estimates were close |
   | B | Slightly optimistic — took ~20% longer than expected |
   | C | Overly optimistic — took >50% longer than expected |
   | D | Unrealistic — plan missed major technical hurdles |

   **Q3: Task Completeness**
   > Were the tasks complete and well-scoped?
   | Option | Description |
   |--------|-------------|
   | A | Perfect — all necessary tasks included, no extras |
   | B | Good — minor tasks missing or a few unnecessary ones |
   | C | Incomplete — significant tasks missing, had to add mid-flight |
   | D | Poor — many missing tasks or mostly irrelevant tasks |

   **Q4: Process Improvement**
   > What would improve the Spec Kit workflow for future features?
   (Free-form text, suggest categories: specify, plan, tasks, checklist, implement, analyze)

   **Q5: Key Learning**
   > What is the single most important lesson from this feature?
   (Free-form text)

4. **Generate retro report**:

   Create `.specify/memory/retro/` directory if it doesn't exist.

   Generate filename: `YYYY-MM-DD-[feature-short-name].md`

   Report structure:

   ```markdown
   # Retrospective: [Feature Name]

   **Date**: [YYYY-MM-DD]
   **Feature Directory**: [path]
   **Spec Version**: [version from spec.md]

   ## Quantitative Metrics

   | Metric | Value |
   |--------|-------|
   | Spec version count | [N] |
   | Plan complexity level | [Full/Standard/Minimal] |
   | Total tasks | [N] |
   | Tasks added mid-flight | [N] |
   | Tasks removed | [N] |
   | Checklists passed | [N] |
   | Checklists failed | [N] |

   ## Qualitative Feedback

   ### Spec Accuracy: [A/B/C/D]
   [User's answer or free-form comment]

   ### Plan Realism: [A/B/C/D]
   [User's answer or free-form comment]

   ### Task Completeness: [A/B/C/D]
   [User's answer or free-form comment]

   ### Process Improvement Suggestions
   [User's free-form text]

   ### Key Learning
   [User's free-form text]

   ## Action Items

   - [ ] [Specific, actionable improvement for future features]
   - [ ] [Another action item]

   ## Patterns for Future Reference

   > If this feature revealed reusable patterns, document them here for future specs.

   [Any reusable insights, e.g., "UI-only changes should use Minimal plan level",
   "Authentication features always need security checklist", etc.]
   ```

5. **Write retro report** to `.specify/memory/retro/YYYY-MM-DD-[feature-short-name].md`.

6. **Update feature memory index**:

   If `.specify/memory/retro/index.md` exists, append a link to the new retro.
   If not, create it:

   ```markdown
   # Retrospective Index

   | Date | Feature | Spec Accuracy | Plan Realism | Task Completeness |
   |------|---------|---------------|--------------|-------------------|
   | YYYY-MM-DD | [feature] | [A/B/C/D] | [A/B/C/D] | [A/B/C/D] |
   ```

## Completion Report

Report to user:
- Retro report path
- Summary of quantitative metrics
- Key themes from qualitative feedback
- Action items for future features
- Suggested: "These insights will be referenced in future `speckit.specify` runs."

## Done When

- [ ] Retro report written to `.specify/memory/retro/`
- [ ] Index updated with new entry
- [ ] User presented with summary and action items
- [ ] Extension hooks dispatched or skipped
