# Spec Kit Role-Based Workflow

This document defines how **Product**, **Design**, **Development**, and **QA** roles interact with the Spec Kit workflow. It establishes clear ownership, handoff criteria, and approval gates for each phase.

---

## Overview

Spec Kit is not just a developer tool — it is a **cross-functional collaboration framework**. Each role contributes at specific phases, owns specific artifacts, and must approve before the next role takes over.

```
Product          Design           Development         QA
   │                │                  │               │
   ▼                ▼                  ▼               ▼
specify ──────► (review) ──────► plan ──────► analyze ──────► tasks
   │                │                  │               │
   │                │                  ▼               │
   │                │            implement ◄────── checklist
   │                │                  │               │
   │                │                  ▼               │
   └──────────── retro ◄─────────────┘               │
                                     │
                                     ▼
                                  commit
```

---

## Role Definitions

### Product (Product Manager / Owner)

**Responsibility**: Define WHAT to build and WHY.

| Phase | Action | Artifact Owned |
|-------|--------|----------------|
| **specify** | Provide feature description, answer clarification questions | `spec.md` |
| **plan** | Review plan for scope alignment | — |
| **implement** | Accept/reject MVP demos | — |
| **retro** | Provide feedback on spec accuracy | `.specify/memory/retro/` |

**Approval Gate**: Product must confirm spec.md captures the intended user value before plan phase begins.

---

### Design (UX/UI Designer)

**Responsibility**: Define HOW users interact with the feature.

| Phase | Action | Artifact Owned |
|-------|--------|----------------|
| **specify** | Review user scenarios for completeness, add UX constraints | `spec.md` (User Scenarios section) |
| **plan** | Review technical approach for UX feasibility | — |
| **checklist** | Generate UX requirements-quality checklist | `checklists/ux.md` |
| **implement** | Validate implemented UI against spec | — |

**Approval Gate**: Design must confirm user scenarios cover all critical flows before tasks are generated.

---

### Development (Engineers / Tech Lead)

**Responsibility**: Define HOW to build it and DO the building.

| Phase | Action | Artifact Owned |
|-------|--------|----------------|
| **plan** | Create technical plan, assess complexity | `plan.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md` |
| **analyze** | Run cross-artifact consistency check | Analysis report (read-only) |
| **tasks** | Generate implementation tasks | `tasks.md` |
| **checklist** | Generate implementation checklist | `checklists/implementation.md` |
| **implement** | Execute tasks, write code | Source code |
| **commit** | Generate commit message | Git commit |

**Approval Gate**: Tech Lead must approve plan.md (including complexity level) before analyze phase.

---

### QA (Quality Assurance / Tester)

**Responsibility**: Verify the feature works as specified.

| Phase | Action | Artifact Owned |
|-------|--------|----------------|
| **specify** | Review acceptance criteria for testability | `spec.md` (Acceptance Scenarios) |
| **checklist** | Generate test coverage checklist | `checklists/test.md` |
| **implement** | Validate against acceptance scenarios | Test results |
| **retro** | Report test coverage gaps | `.specify/memory/retro/` |

**Approval Gate**: QA must confirm acceptance scenarios are testable before implement phase begins.

---

## Artifact Ownership Matrix

| Artifact | Primary Owner | Contributors | Approval Required From |
|----------|--------------|--------------|----------------------|
| `spec.md` | Product | Design, QA | Product |
| `plan.md` | Development (Tech Lead) | Product, Design | Tech Lead |
| `research.md` | Development | — | Tech Lead |
| `data-model.md` | Development | — | Tech Lead |
| `contracts/` | Development | — | Tech Lead |
| `quickstart.md` | Development | QA | Tech Lead |
| `tasks.md` | Development | — | Tech Lead |
| `checklists/*.md` | Context-dependent | All roles | Role-specific |
| Source code | Development | — | Code Reviewer |
| `.specify/memory/retro/*.md` | All roles | — | — |

---

## Phase-by-Phase Role Interaction

### Phase 1: Specify

```
Product ──► provides feature description
    │
    ▼
speckit.specify ──► generates spec.md draft
    │
    ▼
Product ◄── answers clarification questions (max 3)
    │
    ▼
spec.md ──► validated against Spec Quality Checklist
    │
    ▼
Design + QA ──► review for UX/testability gaps
    │
    ▼
[APPROVAL GATE] Product confirms: "This spec captures what we need"
```

**Exit Criteria**:
- [ ] All `[NEEDS CLARIFICATION]` markers resolved
- [ ] Spec Quality Checklist passes
- [ ] Design reviewed user scenarios
- [ ] QA reviewed acceptance criteria

---

### Phase 2: Plan

```
Development (Tech Lead) ──► runs speckit.plan
    │
    ▼
Complexity Assessment ──► determines Full / Standard / Minimal
    │
    ▼
plan.md + optional artifacts ──► generated
    │
    ▼
Product + Design ──► review for scope/UX alignment
    │
    ▼
[APPROVAL GATE] Tech Lead confirms: "This plan is technically feasible"
```

**Exit Criteria**:
- [ ] Complexity level documented
- [ ] Constitution Check passes
- [ ] Product confirms scope alignment
- [ ] Design confirms UX feasibility

---

### Phase 3: Analyze

```
Development ──► runs speckit.analyze
    │
    ▼
Spec-Plan Consistency Report ──► generated (read-only)
    │
    ▼
If CRITICAL issues:
    ├──► Fix spec.md or plan.md
    └──► Re-run analyze
    │
    ▼
[APPROVAL GATE] Zero CRITICAL issues, HIGH issues acknowledged
```

**Exit Criteria**:
- [ ] No CRITICAL issues
- [ ] All HIGH issues have remediation plan or accepted risk
- [ ] Coverage % ≥ 80% (requirements with plan coverage)

---

### Phase 4: Tasks

```
Development ──► runs speckit.tasks
    │
    ▼
tasks.md ──► generated with format validation
    │
    ▼
QA ──► review tasks for test coverage
    │
    ▼
[APPROVAL GATE] Tech Lead confirms: "Tasks are complete and executable"
```

**Exit Criteria**:
- [ ] Format validation passes (all ERRORs resolved)
- [ ] Every user story has implementation tasks
- [ ] QA confirms test tasks included (if TDD requested)

---

### Phase 5: Checklist

```
Context-dependent role ──► runs speckit.checklist
    │
    ▼
Mode selection:
    ├── requirements mode ──► Design / Product owns
    └── implementation mode ──► Development / QA owns
    │
    ▼
checklists/*.md ──► generated
    │
    ▼
[APPROVAL GATE] All checklists pass (0 incomplete items)
```

**Exit Criteria**:
- [ ] Requirements checklists pass (if generated)
- [ ] Implementation checklists pass (if generated)
- [ ] All roles signed off on their domain checklists

---

### Phase 6: Implement

```
Development ──► runs speckit.implement
    │
    ▼
Phase-by-phase execution:
    ├── Setup
    ├── Foundational
    ├── User Story 1 (MVP)
    ├── User Story 2+
    └── Polish
    │
    ▼
QA ──► validates each user story independently
    │
    ▼
[APPROVAL GATE] All tasks marked [x], all checklists pass
```

**Exit Criteria**:
- [ ] All tasks completed and marked `[x]`
- [ ] Implementation matches spec and plan
- [ ] All checklists pass
- [ ] QA acceptance scenarios pass

---

### Phase 7: Retro (Optional)

```
All roles ──► contribute to speckit.retro
    │
    ▼
Quantitative metrics ──► auto-collected
Qualitative feedback ──► interactive questions
    │
    ▼
retro report ──► written to `.specify/memory/retro/`
    │
    ▼
Action items ──► documented for future features
```

**Exit Criteria**:
- [ ] Retro report generated
- [ ] Action items documented
- [ ] Patterns for future reference captured

---

### Phase 8: Commit

```
Development ──► runs speckit.commit
    │
    ▼
Conventional commit message ──► generated from artifacts
    │
    ▼
Developer ──► approves, edits, or skips
    │
    ▼
git commit ──► executed
```

---

## Handoff Criteria Summary

| From | To | Criteria | Approver |
|------|-----|----------|----------|
| Product | Development | Spec approved, clarifications resolved | Product |
| Development | Development | Plan approved, complexity assessed | Tech Lead |
| Development | Development | Analyze passes, no CRITICAL issues | Tech Lead |
| Development | Development | Tasks validated, format checks pass | Tech Lead |
| Development | QA | Implementation complete, checklists pass | Tech Lead |
| QA | Product | Acceptance scenarios pass | QA |
| All | All | Retro completed (optional) | — |

---

## Communication Patterns

### Async Handoffs

Most Spec Kit phases are **async-friendly**:
- Product writes feature description, comes back to answer clarifications
- Design reviews spec.md on their own schedule
- QA reviews acceptance criteria before implementation

### Sync Points

These require real-time or near-real-time collaboration:
- **Clarification Q&A** (specify phase): Product must respond to questions
- **Plan review** (plan phase): Product + Design + Tech Lead alignment meeting
- **MVP validation** (implement phase): Demo User Story 1 before proceeding

### Escalation Path

| Conflict | Escalate To | Resolution |
|----------|-------------|------------|
| Spec scope too large | Product + Tech Lead | Split into multiple features |
| Plan violates constitution | Tech Lead + Architecture | Constitution amendment or plan adjustment |
| Tasks incomplete | Tech Lead + QA | Regenerate tasks with QA input |
| Implementation blocks on design | Tech Lead + Design | Design spike or temporary solution |

---

## Checklist for New Team Members

### Product
- [ ] Understand how to write a feature description for `speckit.specify`
- [ ] Know how to answer clarification questions
- [ ] Can review `spec.md` for completeness
- [ ] Understands approval gate responsibilities

### Design
- [ ] Knows when to review user scenarios (specify phase)
- [ ] Can generate UX requirements checklists
- [ ] Understands how to validate implemented UI

### Development
- [ ] Can run all speckit commands
- [ ] Understands complexity assessment
- [ ] Knows how to generate and validate tasks
- [ ] Can interpret analyze reports

### QA
- [ ] Knows when to review acceptance criteria
- [ ] Can generate test coverage checklists
- [ ] Understands how to validate user stories independently

---

*This document should be reviewed and updated quarterly or after any significant Spec Kit version change.*
