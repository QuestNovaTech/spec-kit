---
description: Generate a custom checklist for the current feature. Supports two modes — requirements-quality (default) and implementation-validation.
scripts:
  sh: scripts/bash/check-prerequisites.sh --json
  ps: scripts/powershell/check-prerequisites.ps1 -Json
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Mode Selection

The checklist command supports two modes. Default is `requirements`.

| Mode | Purpose | When to use |
|------|---------|-------------|
| `requirements` (default) | Validate the quality, clarity, and completeness of requirements | Before or during spec/plan writing |
| `implementation` | Validate that implementation matches the spec and plan | After or during `speckit.implement` |

**Mode detection**:
1. If user explicitly passes `--mode=requirements` or `--mode=implementation`, use that.
2. If no mode flag: infer from context:
   - If `tasks.md` exists and has unchecked items → suggest `implementation`
   - If `spec.md` exists but `plan.md` does not → use `requirements`
   - Otherwise default to `requirements`

## Pre-Execution Checks

**Check for extension hooks (before checklist generation)**:
- Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.before_checklist` key
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

    Wait for the result of the hook command before proceeding to the Execution Steps.
    ```
- If no hooks are registered or `.specify/extensions.yml` does not exist, skip silently

## Execution Steps

1. **Setup**: Run `{SCRIPT}` from repo root and parse JSON for FEATURE_DIR and AVAILABLE_DOCS list.
   - All file paths must be absolute.
   - For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. **IF EXISTS**: Load `/memory/constitution.md` for project principles and governance constraints.

3. **Determine mode** (see Mode Selection above).

4. **Load feature context**: Read from FEATURE_DIR based on mode:

   **For `requirements` mode**:
   - spec.md: Feature requirements and scope
   - plan.md (if exists): Technical details, dependencies

   **For `implementation` mode**:
   - spec.md: Feature requirements (to verify against)
   - plan.md: Technical plan
   - tasks.md: Implementation tasks (to check completion)
   - Source files referenced in tasks.md (spot-check)

5. **Generate checklist** based on selected mode:

---

### Mode: `requirements` — "Unit Tests for Requirements Writing"

**CRITICAL CONCEPT**: Checklists are **UNIT TESTS FOR REQUIREMENTS WRITING** - they validate the quality, clarity, and completeness of requirements in a given domain.

**NOT for verification/testing**:

- ❌ NOT "Verify the button clicks correctly"
- ❌ NOT "Test error handling works"
- ❌ NOT "Confirm the API returns 200"
- ❌ NOT checking if code/implementation matches the spec

**FOR requirements quality validation**:

- ✅ "Are visual hierarchy requirements defined for all card types?" (completeness)
- ✅ "Is 'prominent display' quantified with specific sizing/positioning?" (clarity)
- ✅ "Are hover state requirements consistent across all interactive elements?" (consistency)
- ✅ "Are accessibility requirements defined for keyboard navigation?" (coverage)
- ✅ "Does the spec define what happens when logo image fails to load?" (edge cases)

**Metaphor**: If your spec is code written in English, the checklist is its unit test suite. You're testing whether the requirements are well-written, complete, unambiguous, and ready for implementation - NOT whether the implementation works.

**Generation rules** (same as pre-v1.5.0):

- Create `FEATURE_DIR/checklists/` directory if it doesn't exist
- Generate unique checklist filename based on domain (e.g., `ux.md`, `api.md`, `security.md`)
- If file does NOT exist: Create new file and number items starting from CHK001
- If file exists: Append new items, continuing from last CHK ID
- Never delete or replace existing checklist content

**Category Structure**:
- Requirement Completeness
- Requirement Clarity
- Requirement Consistency
- Acceptance Criteria Quality
- Scenario Coverage
- Edge Case Coverage
- Non-Functional Requirements
- Dependencies & Assumptions
- Ambiguities & Conflicts

**HOW TO WRITE CHECKLIST ITEMS**:

❌ **WRONG** (Testing implementation):
- "Verify landing page displays 3 episode cards"
- "Test hover states work on desktop"
- "Confirm logo click navigates home"

✅ **CORRECT** (Testing requirements quality):
- "Are the exact number and layout of featured episodes specified?" [Completeness]
- "Is 'prominent display' quantified with specific sizing/positioning?" [Clarity]
- "Are hover state requirements consistent across all interactive elements?" [Consistency]

**Traceability Requirements**:
- MINIMUM: ≥80% of items MUST include at least one traceability reference
- Each item should reference: spec section `[Spec §X.Y]`, or use markers: `[Gap]`, `[Ambiguity]`, `[Conflict]`, `[Assumption]`

**🚫 ABSOLUTELY PROHIBITED**:
- Any item starting with "Verify", "Test", "Confirm", "Check" + implementation behavior
- References to code execution, user actions, system behavior
- "Displays correctly", "works properly", "functions as expected"
- "Click", "navigate", "render", "load", "execute"
- Test cases, test plans, QA procedures
- Implementation details (frameworks, APIs, algorithms)

---

### Mode: `implementation` — "Validation of Implementation Against Requirements"

**CRITICAL CONCEPT**: Checklists verify that the **actual implementation** satisfies the requirements defined in the spec and plan.

**NOT for requirements quality**:

- ❌ NOT "Are error handling requirements defined?"
- ❌ NOT "Is 'fast' quantified with specific metrics?"

**FOR implementation validation**:

- ✅ "All functional requirements (FR-001 through FR-005) have corresponding implementation"
- ✅ "User Story 1 acceptance scenarios pass manual validation"
- ✅ "Theme tokens from plan.md are applied to all specified components"
- ✅ "Error handling matches spec edge cases (§Edge Cases)"
- ✅ "Build compiles without errors: `./gradlew assembleDebug`"

**Generation rules**:

- Create `FEATURE_DIR/checklists/` directory if it doesn't exist
- Generate filename: `implementation.md` (or user-specified name)
- If file exists: Append new items, continuing from last CHK ID

**Category Structure**:
- Functional Requirements Coverage (map each FR to implementation)
- User Story Validation (verify each story's acceptance scenarios)
- Plan Compliance (verify tech stack, architecture decisions implemented)
- Edge Case Handling (verify spec edge cases are handled in code)
- Build & Integration (compile, test, integration checks)
- Cross-Cutting Concerns (logging, error handling, performance)

**HOW TO WRITE IMPLEMENTATION CHECKLIST ITEMS**:

❌ **WRONG** (Testing requirements quality):
- "Are error handling requirements defined for all API failure modes?"
- "Is 'fast loading' quantified with specific timing thresholds?"

✅ **CORRECT** (Testing implementation):
- "FR-001: User registration endpoint returns 201 with valid input"
- "FR-002: Email validation rejects malformed addresses (test cases: @missing, double@@, no-tld)"
- "Edge Case: System handles empty search results gracefully (no crash, shows empty state UI)"
- "Build: `./gradlew assembleDebug` completes with zero errors"
- "US1: User can complete checkout end-to-end in under 3 minutes"

**Traceability Requirements**:
- Every item MUST reference the spec requirement or plan element it validates
- Format: `[FR-###]`, `[US#]`, `[Plan §X.Y]`, or `[Spec §X.Y]`

---

6. **Structure Reference**: Generate the checklist following the canonical template in `templates/checklist-template.md` for title, meta section, category headings, and ID formatting. If template is unavailable, use: H1 title, purpose/created meta lines, `##` category sections containing `- [ ] CHK### <item>` lines with globally incrementing IDs starting at CHK001.

7. **Report**: Output full path to checklist file, item count, and summarize whether the run created a new file or appended to an existing one. Summarize:
   - Mode used (`requirements` or `implementation`)
   - Focus areas selected
   - Any explicit user-specified must-have items incorporated

## Example Checklist Types by Mode

### Requirements Mode Examples

**UX Requirements Quality:** `ux.md`
- "Are visual hierarchy requirements defined with measurable criteria? [Clarity, Spec §FR-1]"
- "Is the number and positioning of UI elements explicitly specified? [Completeness, Spec §FR-1]"

**API Requirements Quality:** `api.md`
- "Are error response formats specified for all failure scenarios? [Completeness]"
- "Are rate limiting requirements quantified with specific thresholds? [Clarity]"

### Implementation Mode Examples

**Implementation Validation:** `implementation.md`
- "[FR-001] User registration form submits to `/api/register` and returns 201"
- "[US1] Checkout flow completes end-to-end without errors"
- "[Plan §3.2] Theme tokens applied to all card components in `src/components/cards/`"
- "[Build] `./gradlew assembleDebug` completes successfully"
- "[Edge Case] Empty state displayed when no episodes available"

## Post-Execution Checks

**Check for extension hooks (after checklist generation)**:
Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.after_checklist` key
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
