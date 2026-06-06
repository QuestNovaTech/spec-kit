---
description: Generate a conventional commit message based on the current feature's spec, plan, and completed tasks. Ensures commit messages are consistent, traceable, and follow the project's commit conventions.
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

`speckit.commit` generates a well-structured commit message by analyzing the feature's artifacts (spec.md, plan.md, tasks.md). It ensures:

- Commit messages follow **Conventional Commits** specification
- Messages reference the feature's user stories and requirements
- Scope and type are inferred from the feature domain
- Breaking changes are flagged explicitly

## When to Run

- **Before committing** a feature's implementation
- **After `speckit.implement`** completes all tasks
- **Manually** when the user wants a standardized commit message

## Commit Message Format

Follows [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <short description>

<body>

<footer>
```

### Type Inference

| Feature Domain | Type | Example |
|----------------|------|---------|
| New feature / user story | `feat` | `feat(auth): add OAuth2 login` |
| Bug fix | `fix` | `fix(payment): resolve timeout on checkout` |
| Refactor | `refactor` | `refactor(api): consolidate error handling` |
| Performance | `perf` | `perf(search): index optimization` |
| Documentation | `docs` | `docs(readme): update setup instructions` |
| Tests | `test` | `test(auth): add integration tests` |
| UI-only change | `style` | `style(button): update hover states` |

### Scope Inference

Derive scope from:
1. Feature short name (e.g., `user-auth`, `payment-flow`)
2. Primary module/component affected
3. User story label if applicable (e.g., `us1`, `us2`)

## Execution Steps

1. Run `{SCRIPT}` from repo root and parse FEATURE_DIR and FEATURE_SPEC.

2. **Load feature context**:
   - Read `spec.md`: extract feature name, user stories, functional requirements
   - Read `plan.md`: extract tech stack, complexity level
   - Read `tasks.md`: extract completed tasks (marked `[x]`)

3. **Determine commit type**:
   - If feature is new (has user stories, P1 priority): `feat`
   - If feature is bug fix (description contains "fix", "bug", "resolve"): `fix`
   - If feature is refactor (no new behavior, restructuring): `refactor`
   - If feature is performance (description contains "optimize", "speed", "cache"): `perf`
   - If user explicitly provides type via `--type=`: use that

4. **Determine scope**:
   - Use feature short name (from spec directory name, e.g., `003-user-auth` → `user-auth`)
   - Fallback: primary component from tasks.md file paths

5. **Generate short description** (max 50 chars):
   - Summarize the primary user story or main change
   - Use imperative mood: "add" not "added", "fix" not "fixed"
   - Examples:
     - "add user authentication with OAuth2"
     - "fix payment timeout on checkout"
     - "refactor API error handling"

6. **Generate body**:
   - List completed user stories
   - Reference key functional requirements implemented
   - Mention any breaking changes
   - Reference related issues (if available)

7. **Generate footer**:
   - If breaking change: `BREAKING CHANGE: <description>`
   - If closes issues: `Closes #123, #124`
   - If references spec: `Spec: specs/003-user-auth/spec.md`

8. **Present to user**:

   ```markdown
   ## Proposed Commit Message

   ```
   <type>(<scope>): <short description>

   <body>

   <footer>
   ```

   **Type**: [feat/fix/refactor/perf/docs/test/style]
   **Scope**: [scope]
   **Breaking**: [yes/no]

   You can:
   - Say "yes" or "commit" to accept
   - Say "edit" to modify (provide new message)
   - Say "skip" to cancel
   ```

9. **If user accepts**: Output the final commit message for copy-paste or direct execution.

## Examples

### Feature Commit

```
feat(user-auth): add OAuth2 authentication flow

Implement user authentication via OAuth2 providers (Google, GitHub).

- US1: Users can sign in with OAuth2
- US2: Users can link multiple providers
- FR-001: OAuth2 login endpoint
- FR-002: Provider linking

Spec: specs/003-user-auth/spec.md
```

### Bug Fix Commit

```
fix(payment): resolve checkout timeout for large carts

Fix timeout issue when processing carts with >50 items.

- Root cause: synchronous inventory check blocking payment
- Fix: async inventory validation with 5s timeout

Closes #456
```

### Refactor Commit

```
refactor(api): consolidate error handling middleware

Replace per-route error handling with centralized middleware.

- No functional changes
- All existing tests pass
- Improves maintainability

Spec: specs/007-error-handling/spec.md
```

## Completion Report

Report to user:
- Generated commit message
- Type and scope used
- Suggestion: "Copy the message above or run `git commit -m '...'`"

## Done When

- [ ] Commit message generated from feature artifacts
- [ ] Type and scope inferred or user-specified
- [ ] Message presented to user for approval
- [ ] User accepted, edited, or skipped
