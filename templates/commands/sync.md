---
description: Auto-detect constitution changes and validate template alignment. Generate a Sync Impact Report listing templates that need updates.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Purpose

`speckit.sync` automates the constitution-to-template consistency check that was previously a manual step in `speckit.constitution`. It:

1. Reads the current constitution
2. Scans all templates and command files for constitution references
3. Detects misalignment (outdated version references, missing principles, stale rules)
4. Generates a Sync Impact Report with specific fix instructions

## When to Run

- **After `speckit.constitution`** — recommended in constitution command completion
- **Before starting a new feature** — ensure templates reflect latest governance
- **Periodically** — as part of project maintenance
- **CI/CD integration** — can be run in automated checks

## Execution Steps

1. **Load constitution**:
   - Read `.specify/memory/constitution.md`
   - Extract: version, principles (names + key MUST/SHOULD statements), governance rules

2. **Scan templates for constitution alignment**:

   Check each file in `.specify/templates/`:

   | File | What to check |
   |------|---------------|
   | `spec-template.md` | Mandatory sections match constitution requirements |
   | `plan-template.md` | Constitution Check section aligns with principles |
   | `tasks-template.md` | Task categories reflect principle-driven types |
   | `checklist-template.md` | Checklist categories align with quality gates |
   | `constitution-template.md` | Version placeholder format matches current |

3. **Scan command files for constitution alignment**:

   Check each file in `.specify/templates/commands/*.md`:

   | Check | Description |
   |-------|-------------|
   | Version reference | Does the file reference the correct constitution version? |
   | Principle names | Are principle names current (not renamed/removed)? |
   | Gate rules | Do Constitution Check gates match current MUST statements? |
   | Handoff prompts | Do agent handoffs reference correct downstream commands? |

4. **Detect misalignment patterns**:

   | Pattern | Example | Severity |
   |---------|---------|----------|
   | Version mismatch | Template says "v1.3.0" but constitution is "v1.4.0" | WARNING |
   | Missing principle | Constitution added "Observability" but plan-template gates don't include it | ERROR |
   | Stale principle name | Template references "Testing-First" but constitution renamed to "Test-First" | ERROR |
   | Orphaned reference | Template references a principle that was removed | WARNING |
   | Gate mismatch | Constitution says "MUST use TDD" but plan allows skipping tests | ERROR |

5. **Generate Sync Impact Report**:

   Output a Markdown report (no file writes unless `--apply` flag):

   ```markdown
   # Constitution Sync Impact Report

   **Constitution Version**: [current version]
   **Scan Date**: [YYYY-MM-DD]
   **Templates Scanned**: [N]
   **Commands Scanned**: [N]

   ## Alignment Status

   | Template/Command | Status | Issues |
   |------------------|--------|--------|
   | spec-template.md | ✅ Aligned | — |
   | plan-template.md | ⚠ Warning | Version mismatch (v1.3.0 → v1.4.0) |
   | tasks-template.md | ❌ Error | Missing "Observability" principle in task categories |
   | ... | ... | ... |

   ## Detailed Findings

   ### plan-template.md
   - **Issue**: Constitution version reference outdated
   - **Location**: Line 42
   - **Current**: `Constitution v1.3.0`
   - **Expected**: `Constitution v1.4.0`
   - **Fix**: Replace version string

   ### tasks-template.md
   - **Issue**: Missing principle-driven task category
   - **Current categories**: Setup, Foundational, User Stories, Polish
   - **Missing**: "Observability" tasks (constitution principle IV)
   - **Fix**: Add observability task examples to Phase N (Polish)

   ## Recommended Actions

   1. [Priority] Fix ERROR-level issues before next feature
   2. [Recommended] Fix WARNING-level issues during next maintenance window
   3. [Optional] Run `speckit.sync --apply` to auto-fix safe changes (version strings only)

   ## Auto-Fixable Issues

   The following can be safely auto-fixed with `--apply`:
   - [ ] Version string updates
   - [ ] Date updates
   ```

6. **Optional auto-fix** (`--apply` flag):

   If user passes `--apply`, automatically fix safe issues:
   - Version string updates (constitution version references)
   - Date updates (last amended dates in templates)
   - **Never auto-fix**: Principle name changes, gate rule changes, structural changes

## Completion Report

Report to user:
- Constitution version scanned
- Total files checked
- Error count, Warning count, Aligned count
- Path to full report (if written to disk with `--save`)
- Recommendation: fix ERRORs before next feature

## Done When

- [ ] All templates and commands scanned
- [ ] Misalignment detected and categorized
- [ ] Sync Impact Report generated
- [ ] Auto-fix applied (if `--apply` flag)
- [ ] User informed of remaining manual fixes
