---
description: Distill a completed coding session into reusable knowledge — file change heatmap + extracted patterns. Writes findings to `.specify/memory/distill/` for future sessions.
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --paths-only
  ps: scripts/powershell/check-prerequisites.ps1 -Json -PathsOnly
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Purpose

`speckit.distill` captures the **tangible output** of a coding session so future sessions can build on it. Unlike `speckit.retro` (which asks "how did the process feel?"), `distill` asks:

- What files changed and how intensely? (heatmap)
- What reusable patterns emerged? (pattern extraction)
- What pitfalls should be avoided next time? (anti-patterns)
- What context should be loaded automatically for related features? (memory seeds)

## When to Run

- **After a long coding session** that produced meaningful changes
- **After `speckit.implement`** completes a feature
- **Before switching contexts** (so the next session can pick up distilled knowledge)
- **Manually** when the user says "沉淀一下本次会话"

## Relationship to Other Commands

| Command | Focus | Output |
|---------|-------|--------|
| `speckit.retro` | Process feedback (spec accuracy, plan realism) | `.specify/memory/retro/*.md` |
| `speckit.distill` | Session artifacts + reusable knowledge | `.specify/memory/distill/*.md` |
| `speckit.heatmap` | Sub-component of distill — file change visualization | Inline table in distill report |

**Recommended flow**: `implement → retro → distill`
- `retro` collects qualitative team feedback
- `distill` extracts objective patterns and heatmap data

## Pre-Execution Checks

**Check for extension hooks (before distill)**:
- Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.before_distill` key
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

2. **Determine session boundary**:
   - If user passes `--since=<git-ref>`: use that as start point
   - If user passes `--session-start=<timestamp>`: use that
   - Otherwise: use the most recent feature branch creation or last commit on current branch

3. **Generate Heatmap** (sub-module):

   a. Collect file change data:
      ```bash
      git diff --stat <session-start-ref>..HEAD
      ```
   b. Cross-reference with `tasks.md` file path references
   c. Compute heat score per file:
      - `+lines + -lines` from git diff
      - `+references in tasks.md` × weight
      - `+checklist items mentioning file` × weight
   d. Categorize heat levels:

      | Heat Score | Level | Icon |
      |------------|-------|------|
      | ≥ 200 | Very Hot | 🔥🔥🔥🔥🔥 |
      | 100–199 | Hot | 🔥🔥🔥🔥 |
      | 50–99 | Warm | 🔥🔥🔥 |
      | 10–49 | Mild | 🔥🔥 |
      | 1–9 | Cool | 🔥 |
      | 0 | Untouched | ❄️ |

   e. Output heatmap table:
      ```markdown
      ## File Change Heatmap

      | Rank | File | Insertions | Deletions | Net Change | Task References | Heat |
      |------|------|------------|-----------|------------|-----------------|------|
      | 1 | `src/core/auth.ts` | +240 | -80 | 320 | 5 | 🔥🔥🔥🔥🔥 |
      | 2 | `src/components/LoginForm.tsx` | +120 | -30 | 150 | 3 | 🔥🔥🔥🔥 |
      ```

4. **Extract Patterns** (sub-module):

   Read from the session's artifacts:
   - `spec.md`: domain concepts, user stories, constraints
   - `plan.md`: tech choices, architecture decisions, complexity level
   - `tasks.md`: implementation patterns, file organization
   - `checklists/*.md`: quality gates that passed/failed
   - `retro/*.md` (if exists): lessons learned

   Extract these pattern categories:

   | Category | What to capture | Example |
   |----------|---------------|---------|
   | **Architecture Pattern** | Reusable structural decisions | "UI-only features use Minimal plan level" |
   | **Implementation Pattern** | Code organization conventions | "Auth features always add middleware before routes" |
   | **Domain Pattern** | Business logic insights | "Albums are never nested — flat hierarchy enforced" |
   | **Tooling Pattern** | Dev workflow conventions | "Use `uv` for Python, `npm` for Node" |
   | **Anti-Pattern** | Things to avoid | "Don't put business logic in React components" |
   | **Decision Rationale** | Why a choice was made | "Chose SQLite over Postgres because offline-first requirement" |

   Pattern extraction rules:
   - Only extract patterns that are **generalizable** beyond this specific feature
   - Include the **context** where the pattern applies
   - Include a **confidence** level (High/Medium/Low) based on how many times it appeared
   - Link back to source artifacts `[Source: plan.md §3.2]`

5. **Generate Memory Seeds**:

   Create short "memory seeds" — one-sentence context snippets that future `speckit.specify` runs can reference:

   ```markdown
   - This project uses a flat entity hierarchy for albums. [Source: specs/001-photo-app/spec.md]
   - Authentication middleware must be implemented before route handlers. [Source: specs/003-user-auth/tasks.md]
   - UI-only changes default to Minimal plan complexity. [Source: retro/2026-06-06-photo-app.md]
   ```

6. **Write distill report**:

   Create `.specify/memory/distill/` directory if it doesn't exist.

   Generate filename: `YYYY-MM-DD-[feature-short-name].md`

   Report structure:

   ```markdown
   # Session Distillation: [Feature Name]

   **Date**: [YYYY-MM-DD]
   **Feature Directory**: [path]
   **Session Range**: [git-ref-start]..[git-ref-end]
   **Total Files Changed**: [N]
   **Total Lines Changed**: +[N] / -[N]

   ## File Change Heatmap

   | Rank | File | Insertions | Deletions | Net Change | Task References | Heat |
   |------|------|------------|-----------|------------|-----------------|------|
   | ... | ... | ... | ... | ... | ... | ... |

   ## Hotspots Summary

   🔥 **Top 3 hottest files**:
   1. `src/core/auth.ts` (320 lines changed, 5 task references)
   2. `src/components/LoginForm.tsx` (150 lines changed, 3 task references)
   3. `src/utils/api.ts` (90 lines changed, 4 task references)

   **Observation**: [One-line insight, e.g., "Authentication touched 40% of the codebase"]

   ## Reusable Patterns

   ### Architecture Patterns
   - **[High]** UI-only features should use Minimal plan level. [Source: plan.md §Complexity Assessment]
   - **[Medium]** New data models require data-model.md + contracts/. [Source: plan.md §Phase 1]

   ### Implementation Patterns
   - **[High]** Always implement middleware before route handlers in auth features. [Source: tasks.md §Phase 2]
   - **[Medium]`

   ### Domain Patterns
   - **[High]`

   ### Tooling Patterns
   - ...

   ### Anti-Patterns
   - **[Medium]** Avoid putting business logic in React components; keep it in services/hooks. [Source: retro/...]

   ## Memory Seeds for Future Sessions

   - "This project uses a flat entity hierarchy for albums." [Source: specs/001-photo-app/spec.md]
   - "Authentication middleware must precede route handlers." [Source: specs/003-user-auth/tasks.md]

   ## Suggested Next Actions

   - [ ] Review hottest files for refactoring opportunities
   - [ ] Apply extracted patterns to the next related feature
   - [ ] Update project constitution if any pattern should become governance
   ```

7. **Update distill index**:

   If `.specify/memory/distill/index.md` exists, append a link to the new distill report.
   If not, create it:

   ```markdown
   # Distillation Index

   | Date | Feature | Files Changed | Hotspots | Patterns Extracted |
   |------|---------|---------------|----------|-------------------|
   | YYYY-MM-DD | [feature] | [N] | [N] | [N] |
   ```

8. **Optional: feed memory seeds to constitution**:

   If any extracted pattern has **High confidence** and should become project-wide governance, suggest:
   > "Pattern '[pattern]' appears with high confidence. Consider adding it to `.specify/memory/constitution.md` via `speckit.constitution`."

## Command Options

| Flag | Description |
|------|-------------|
| `--since=<ref>` | Start of session (git ref or timestamp) |
| `--feature=<dir>` | Explicitly specify feature directory |
| `--no-heatmap` | Skip heatmap generation, patterns only |
| `--no-patterns` | Skip pattern extraction, heatmap only |
| `--output=<path>` | Custom output path for report |

## Completion Report

Report to user:
- Distill report path
- Total files changed
- Top 3 hotspots
- Number of patterns extracted by category
- Memory seeds generated
- Suggested: "Future `speckit.specify` runs can reference these patterns."

## Done When

- [ ] Heatmap generated from git diff + tasks.md
- [ ] Patterns extracted from session artifacts
- [ ] Memory seeds documented
- [ ] Distill report written to `.specify/memory/distill/`
- [ ] Index updated with new entry
- [ ] User presented with summary and suggested next actions
- [ ] Extension hooks dispatched or skipped
