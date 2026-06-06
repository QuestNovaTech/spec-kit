---
description: >
  **DEPRECATED — functionality merged into `speckit.specify`**. 
  This command is retained for backward compatibility. Invoking it will delegate
to `speckit.specify`'s integrated clarification sub-phase.
handoffs:
  - label: Build Technical Plan
    agent: speckit.plan
    prompt: Create a plan for the spec. I am building with...
scripts:
   sh: scripts/bash/check-prerequisites.sh --json --paths-only
   ps: scripts/powershell/check-prerequisites.ps1 -Json -PathsOnly
---

## User Input

```text
$ARGUMENTS
```

## Deprecation Notice

The `speckit.clarify` command has been **deprecated** as of Spec Kit v1.5.0.
Clarification is now an **integrated sub-phase** of `speckit.specify`.

**Why**: The previous separate `specify` → `clarify` flow caused redundant user
interactions. Both commands asked clarifying questions, leading to:
- Duplicate questioning
- Confusion about which command "owns" clarification
- Extra round-trips before planning could begin

**What changed**:
- `speckit.specify` now generates the spec draft, runs quality validation,
  identifies `[NEEDS CLARIFICATION]` markers, and asks the user for answers
  **all in one invocation**.
- The spec is only marked "ready for planning" once all clarifications are
  resolved and validation passes.

## Backward-Compatibility Behavior

If this command is invoked directly:

1. Run `{SCRIPT}` from repo root and parse `FEATURE_DIR` and `FEATURE_SPEC`.
2. Load the current spec file.
3. **If the spec has unresolved `[NEEDS CLARIFICATION]` markers**:
   - Present them to the user as questions (same format as the integrated
     specify clarification sub-phase).
   - Update the spec with answers.
   - Re-run the Spec Quality Checklist validation.
4. **If no unresolved markers exist**:
   - Report: "No unresolved clarifications. Spec is ready for planning."
5. Suggest next command: `__SPECKIT_COMMAND_PLAN__`

## Done When

- [ ] User informed of deprecation and directed to `speckit.specify` for future use
- [ ] Any remaining `[NEEDS CLARIFICATION]` markers resolved (if present)
- [ ] Completion reported with path to updated spec
