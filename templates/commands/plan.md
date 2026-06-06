---
description: Execute the implementation planning workflow using the plan template to generate design artifacts. Phase 0/1 are now optional based on complexity assessment.
handoffs:
  - label: Analyze Spec-Plan Consistency
    agent: speckit.analyze
    prompt: Run cross-artifact consistency analysis between spec and plan
    send: true
  - label: Create Tasks
    agent: speckit.tasks
    prompt: Break the plan into tasks
    send: true
scripts:
  sh: scripts/bash/setup-plan.sh --json
  ps: scripts/powershell/setup-plan.ps1 -Json
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Pre-Execution Checks

**Check for extension hooks (before planning)**:
- Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.before_plan` key
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

1. **Setup**: Run `{SCRIPT}` from repo root and parse JSON for FEATURE_SPEC, IMPL_PLAN, SPECS_DIR, BRANCH. For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. **Load context**: Read FEATURE_SPEC and `/memory/constitution.md`. Load IMPL_PLAN template (already copied).

3. **Complexity Assessment** (NEW in v1.5.0):

   Before generating artifacts, assess the feature complexity to determine which planning phases are needed:

   | Indicator | Weight | Threshold |
   |-----------|--------|-----------|
   | New data model / entities | +2 | |
   | External API / service integration | +2 | |
   | New interface contracts (API, CLI, UI) | +2 | |
   | Cross-cutting architectural change | +2 | |
   | Security or compliance implications | +2 | |
   | Performance or scalability requirements | +1 | |
   | UI-only change (no new data/contract) | -1 | |
   | Bug fix / refactor (no new behavior) | -1 | |
   | Well-understood domain (CRUD, standard patterns) | -1 | |

   **Scoring**:
   - **Score >= 4**: Full plan — Phase 0 (Research) + Phase 1 (Design & Contracts) + Phase 2 (Task-Ready Plan)
   - **Score 1-3**: Standard plan — Phase 1 (Design & Contracts, lightweight) + Phase 2
   - **Score <= 0**: Minimal plan — Skip Phase 0 and Phase 1 artifacts; generate only `plan.md` with Technical Context + Constitution Check + Project Structure

   **User override**: If user explicitly requests `--full-plan` or `--minimal-plan`, respect the override.

4. **Execute plan workflow** based on complexity level:

   - **Fill Technical Context** (mark unknowns as "NEEDS CLARIFICATION")
   - **Fill Constitution Check section** from constitution
   - **Evaluate gates** (ERROR if violations unjustified)

   **If Full or Standard plan**:
   - Phase 0: Generate `research.md` (resolve all NEEDS CLARIFICATION)
   - Phase 1: Generate `data-model.md`, `contracts/`, `quickstart.md`
   - Phase 1: Update agent context by running the agent script
   - Re-evaluate Constitution Check post-design

   **If Minimal plan**:
   - Skip Phase 0 and Phase 1 artifact generation
   - Document assumptions directly in plan.md Technical Context
   - Proceed directly to Phase 2 structure

## Mandatory Post-Execution Hooks

**You MUST complete this section before reporting completion to the user.**

Check if `.specify/extensions.yml` exists in the project root.
- If it does not exist, or no hooks are registered under `hooks.after_plan`, skip to the Completion Report.
- If it exists, read it and look for entries under the `hooks.after_plan` key.
- If the YAML cannot be parsed or is invalid, skip hook checking silently and continue to the Completion Report.
- Filter out hooks where `enabled` is explicitly `false`. Treat hooks without an `enabled` field as enabled by default.
- For each remaining hook, do **not** attempt to interpret or evaluate hook `condition` expressions:
  - If the hook has no `condition` field, or it is null/empty, treat the hook as executable
  - If the hook defines a non-empty `condition`, skip the hook and leave condition evaluation to the HookExecutor implementation
- For each executable hook, output the following based on its `optional` flag:
  - **Mandatory hook** (`optional: false`) — **You MUST emit `EXECUTE_COMMAND:` for each mandatory hook**:
    ```
    ## Extension Hooks

    **Automatic Hook**: {extension}
    Executing: `/{command}`
    EXECUTE_COMMAND: {command}
    ```
  - **Optional hook** (`optional: true`):
    ```
    ## Extension Hooks

    **Optional Hook**: {extension}
    Command: `/{command}`
    Description: {description}

    Prompt: {prompt}
    To execute: `/{command}`
    ```

## Completion Report

Command ends after Phase 2 planning. Report:
- Complexity score and plan level (Full / Standard / Minimal)
- Branch, IMPL_PLAN path, and generated artifacts
- If Minimal plan: explicitly note which artifacts were skipped and why

## Phases

### Phase 0: Outline & Research (Optional — Full plan only)

1. **Extract unknowns from Technical Context** above:
   - For each NEEDS CLARIFICATION → research task
   - For each dependency → best practices task
   - For each integration → patterns task

2. **Generate and dispatch research agents**:

   ```text
   For each unknown in Technical Context:
     Task: "Research {unknown} for {feature context}"
   For each technology choice:
     Task: "Find best practices for {tech} in {domain}"
   ```

3. **Consolidate findings** in `research.md` using format:
   - Decision: [what was chosen]
   - Rationale: [why chosen]
   - Alternatives considered: [what else evaluated]

**Output**: research.md with all NEEDS CLARIFICATION resolved

### Phase 1: Design & Contracts (Optional — Full/Standard plan)

**Prerequisites:** `research.md` complete (if Phase 0 ran)

1. **Extract entities from feature spec** → `data-model.md`:
   - Entity name, fields, relationships
   - Validation rules from requirements
   - State transitions if applicable

2. **Define interface contracts** (if project has external interfaces) → `/contracts/`:
   - Identify what interfaces the project exposes to users or other systems
   - Document the contract format appropriate for the project type
   - Examples: public APIs for libraries, command schemas for CLI tools, endpoints for web services, grammars for parsers, UI contracts for applications
   - Skip if project is purely internal (build scripts, one-off tools, etc.)

3. **Create quickstart validation guide** → `quickstart.md`:
   - Document runnable validation scenarios that prove the feature works end-to-end
   - Include prerequisites, setup commands, test/run commands, and expected outcomes
   - Use links or references to contracts and data model details instead of duplicating them
   - Do not include full implementation code, model/service/controller bodies, migrations, or complete test suites
   - Keep this artifact as a validation/run guide; implementation details belong in `tasks.md` and the implementation phase

4. **Agent context update**:
   - Update the plan reference between the `<!-- SPECKIT START -->` and `<!-- SPECKIT END -->` markers in `__CONTEXT_FILE__` to point to the plan file created in step 1 (the IMPL_PLAN path)

**Output**: data-model.md, /contracts/*, quickstart.md, updated agent context file

## Key rules

- Use absolute paths for filesystem operations; use project-relative paths for references in documentation and agent context files
- ERROR on gate failures or unresolved clarifications
- Minimal plan is the default for UI-only changes, bug fixes, and well-understood patterns

## Done When

- [ ] Plan workflow executed and design artifacts generated (according to complexity level)
- [ ] Complexity assessment documented in plan.md
- [ ] Extension hooks dispatched or skipped according to the rules in Mandatory Post-Execution Hooks above
- [ ] Completion reported to user with branch, plan path, complexity level, and generated artifacts
