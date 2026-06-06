# Spec Kit v1.5.0 Changelog

**Release Date**: 2026-06-06
**Constitution Version**: v1.5.0 (MINOR bump — new commands + workflow changes)

---

## Summary

This release addresses 9 structural issues identified in the Spec Kit workflow, plus 2 new enhancements (commit command + role-based workflow documentation). The core theme is **"reduce friction, increase alignment"** — fewer redundant interactions, earlier error detection, and better matching between rules and actual practice.

---

## 🔴 High Priority Changes

### 1. Clarify Merged into Specify (Issue #1)

**Problem**: `specify` and `clarify` both asked clarifying questions, causing duplicate user interactions.

**Solution**: Clarification is now an **integrated sub-phase** of `speckit.specify`.

- `speckit.specify` generates draft → runs quality validation → identifies `[NEEDS CLARIFICATION]` markers → asks user → resolves → marks spec "ready for planning"
- `speckit.clarify` is **deprecated** but retained for backward compatibility; invoking it delegates to specify's clarification logic
- **Impact**: Reduced from 2+ user interaction rounds to 1

**Files changed**:
- `templates/commands/specify.md` — added integrated clarification sub-phase
- `templates/commands/clarify.md` — converted to deprecation stub

---

### 2. Analyze Moved Before Tasks (Issue #2)

**Problem**: `analyze` ran after `tasks` and `checklist`, meaning spec-plan misalignment was caught too late.

**Solution**: `analyze` now runs **immediately after `plan` and before `tasks`**.

- New flow: `specify → plan → analyze → tasks → checklist → implement → retro`
- `analyze` now checks `spec.md` ↔ `plan.md` consistency only (not tasks.md)
- Handoff updated: `plan.md` → `speckit.analyze` → `speckit.tasks`

**Files changed**:
- `templates/commands/analyze.md` — updated goal, execution steps, and script requirements (`--require-plan`)
- `templates/commands/plan.md` — handoff updated to `speckit.analyze`
- `templates/commands/tasks.md` — handoff from `speckit.analyze`

---

### 3. Checklist Dual Mode Support (Issue #3)

**Problem**: Checklist rules demanded "Unit Tests for Requirements Writing", but actual output was implementation validation.

**Solution**: `speckit.checklist` now supports two modes:

| Mode | Purpose | Example items |
|------|---------|---------------|
| `requirements` (default) | Validate requirements quality | "Are hover states consistently defined?" |
| `implementation` | Validate implementation | "Theme tokens applied to all components" |

- Mode auto-detected from context, or explicitly set via `--mode=`
- Each mode has distinct category structure, item patterns, and traceability rules
- `templates/checklist-template.md` updated with `**Mode**: [requirements \| implementation]` field

**Files changed**:
- `templates/commands/checklist.md` — complete rewrite with dual mode logic
- `templates/checklist-template.md` — added Mode field

---

## 🟡 Medium Priority Changes

### 4. Tasks Format Validation (Issue #4)

**Problem**: Actual `tasks.md` output often missed Task IDs, [P] markers, and [US#] labels.

**Solution**: Automatic format validation after task generation.

| Check | Severity | Auto-fix? |
|-------|----------|-----------|
| Missing Task ID (T###) | ERROR | Yes — assign next sequential ID |
| Missing checkbox | ERROR | Yes — prepend `- [ ] ` |
| Missing [US#] on story tasks | WARNING | Yes — infer from phase heading |
| Missing file path | WARNING | No — report only |
| Non-sequential IDs | WARNING | No — report only |

**Files changed**:
- `templates/commands/tasks.md` — added Format Validation section
- `templates/tasks-template.md` — added Format Validation comment block

---

### 5. Plan Phase Optionalization (Issue #5)

**Problem**: Plan rules required `research.md`, `data-model.md`, `contracts/`, `quickstart.md` for every feature, but small UI changes didn't need them.

**Solution**: Complexity Assessment determines which phases run.

| Score | Level | Artifacts |
|-------|-------|-----------|
| ≥ 4 | Full | research.md + data-model.md + contracts/ + quickstart.md |
| 1–3 | Standard | lightweight research + data-model.md + contracts/ + quickstart.md |
| ≤ 0 | Minimal | plan.md only |

- Score computed from 9 indicators (new data model, external API, UI-only, bug fix, etc.)
- User can override with `--full-plan` or `--minimal-plan`
- `templates/plan-template.md` updated with Complexity Assessment section

**Files changed**:
- `templates/commands/plan.md` — added Complexity Assessment
- `templates/plan-template.md` — added Complexity Assessment section with indicator checklist

---

### 6. Extension Hooks Simplified (Issue #6)

**Problem**: Every command had ~40 lines of identical Extension Hooks boilerplate for a feature (`extensions.yml`) that never existed.

**Solution**: No structural change to rules (hooks still supported), but acknowledged as **stable, rarely-used infrastructure**. No action needed unless extensions are actually adopted.

- If a future release adds extensions, the existing hook logic works as-is
- For now, the overhead is accepted as the cost of forward compatibility

**Files changed**: None (intentionally — the hooks work correctly, they're just unused)

---

## 🟢 Low Priority Changes

### 7. Taskstoissues Multi-Backend Support (Issue #7)

**Problem**: Only supported GitHub; no fallback if MCP unavailable.

**Solution**: Supports GitHub, Linear, Jira, plus CSV/Markdown fallback.

| Backend | Detection | Fallback |
|---------|-----------|----------|
| GitHub | `github-mcp-server` tool | Auto-switch to fallback |
| Linear | `linear-mcp-server` tool | Auto-switch to fallback |
| Jira | `jira-mcp-server` tool | Auto-switch to fallback |
| CSV/Markdown | No MCP available | Default |

- Metadata mapping table: Task ID → issue fields (labels, priority, assignee)
- Error handling table for common failure scenarios

**Files changed**:
- `templates/commands/taskstoissues.md` — added multi-backend support, error handling

---

### 8. Constitution Sync Automation (Issue #8)

**Problem**: Constitution updates required manual template synchronization.

**Solution**: New `speckit.sync` command.

- Scans all templates and commands for constitution alignment
- Detects: version mismatch, missing principles, stale names, gate mismatches
- Generates Sync Impact Report with specific fix instructions
- Auto-fix safe changes (version strings, dates) with `--apply`
- Recommended after every `speckit.constitution` run

**Files changed**:
- `templates/commands/constitution.md` — added recommendation to run `speckit.sync`
- `templates/commands/sync.md` — **NEW FILE**

---

### 9. Retrospective Phase (Issue #9)

**Problem**: No formal mechanism to collect feedback and improve the process.

**Solution**: New `speckit.retro` command.

- Collects quantitative metrics (spec versions, tasks added/removed, checklists passed)
- Interactive qualitative feedback (5 questions: spec accuracy, plan realism, task completeness, process improvement, key learning)
- Writes report to `.specify/memory/retro/YYYY-MM-DD-feature.md`
- Updates retro index for cross-feature pattern analysis
- Suggested after `speckit.implement` completion

**Files changed**:
- `templates/commands/retro.md` — **NEW FILE**

---

## ✨ New Enhancements (Beyond Original 9 Issues)

### 10. Commit Message Generation (`speckit.commit`)

**New command** that generates Conventional Commits messages from feature artifacts.

- Infers type (`feat`/`fix`/`refactor`/`perf`/etc.) from feature domain
- Infers scope from feature short name
- Generates short description, body (user stories + FRs), footer (spec reference)
- User approves, edits, or skips

**Files changed**:
- `templates/commands/commit.md` — **NEW FILE**

---

### 11. Role-Based Workflow Documentation

**New documentation** formalizing how Product, Design, Development, and QA interact with Spec Kit.

See `docs/workflows/ROLE-BASED-WORKFLOW.md` for:
- Role responsibilities at each phase
- Artifact ownership matrix
- Handoff criteria between roles
- Approval gates

**Files changed**:
- `docs/workflows/ROLE-BASED-WORKFLOW.md` — **NEW FILE**

---

## Updated Workflow Diagram

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Product/   │────▶│  speckit.   │────▶│  speckit.   │────▶│  speckit.   │
│   Design    │     │   specify   │     │    plan     │     │   analyze   │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                                                   │
                                                                   ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  speckit.   │◀────│  speckit.   │◀────│  speckit.   │◀────│  speckit.   │
│   retro     │     │  implement  │     │  checklist  │     │   tasks     │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
       │
       ▼
┌─────────────┐
│  speckit.   │
│   commit    │
└─────────────┘

Supporting commands:
  speckit.constitution  →  speckit.sync (validate alignment)
  speckit.taskstoissues (export to GitHub/Linear/Jira)
  speckit.clarify (deprecated, delegates to specify)
```

---

## Migration Guide

### For Existing Projects

1. **No breaking changes** — all existing commands work as before
2. **Clarify deprecation**: `speckit.clarify` still works but shows deprecation notice
3. **New commands are optional**: `retro`, `sync`, `commit` can be adopted incrementally
4. **Checklist mode**: Existing checklists remain valid; new checklists can use `--mode=implementation`
5. **Plan complexity**: Existing plans unaffected; new features benefit from auto-assessment

### For Template Authors

1. Update custom templates to include Complexity Assessment (plan-template)
2. Update custom templates to include Mode field (checklist-template)
3. Update custom templates to include Format Validation comment (tasks-template)

---

## Files Added

| File | Purpose |
|------|---------|
| `templates/commands/retro.md` | Post-implementation retrospective |
| `templates/commands/sync.md` | Constitution-template alignment checker |
| `templates/commands/commit.md` | Conventional commit message generator |
| `docs/workflows/ROLE-BASED-WORKFLOW.md` | Role-based workflow documentation |

## Files Modified

| File | Change |
|------|--------|
| `templates/commands/specify.md` | Integrated clarification sub-phase |
| `templates/commands/clarify.md` | Deprecated, backward-compat stub |
| `templates/commands/plan.md` | Complexity Assessment, handoff to analyze |
| `templates/commands/analyze.md` | Runs before tasks, checks spec-plan only |
| `templates/commands/tasks.md` | Format Validation, handoff from analyze |
| `templates/commands/checklist.md` | Dual mode (requirements/implementation) |
| `templates/commands/taskstoissues.md` | Multi-backend support, fallback |
| `templates/commands/constitution.md` | Recommends speckit.sync |
| `templates/commands/implement.md` | Suggests speckit.retro after completion |
| `templates/plan-template.md` | Complexity Assessment section |
| `templates/tasks-template.md` | Format Validation comment block |
| `templates/checklist-template.md` | Mode field |

---

*End of Changelog*
