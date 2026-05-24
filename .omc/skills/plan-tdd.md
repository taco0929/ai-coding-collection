---
name: plan-tdd
description: Requirements interview → testing plan → failing tests → minimal implementation → verify. For deliberate TDD on an existing Python/FastAPI codebase.
triggers:
  - plan-tdd
  - plan tdd
  - interview then test
  - discuss then implement
---

# plan-tdd

A structured workflow for developers who want to fully understand requirements and design the test surface before writing any code.

## When to use

- Implementing a non-trivial feature or fix where intent and edge cases need to be explicit
- Any change touching the task dispatch pipeline, async callbacks, or DB writes
- When the scope is unclear enough that jumping straight to code risks rework

## Phases

### Phase 1 — Requirements Interview

Conduct a Socratic interview to surface all constraints before touching tests or code.

Ask about:
- **Intent**: What should this do? What problem does it solve?
- **Inputs / outputs**: What data comes in, what must come out?
- **Edge cases**: What are the failure modes? What are the boundary conditions?
- **Constraints**: Performance, backwards compatibility, DB schema limits, existing API contracts?
- **Scope**: What is explicitly out of scope?

Do not proceed to Phase 2 until all ambiguities are resolved or consciously deferred.

Delegate to `analyst` (opus) for complex domain logic or hidden constraint discovery.

Once the interview is complete, write the output to:
```bash
PROJECT=$(git rev-parse --show-toplevel | xargs basename)
BRANCH=$(git branch --show-current)
mkdir -p "/tmp/$PROJECT/$BRANCH"
# write to /tmp/$PROJECT/$BRANCH/plan.md
```

### Phase 2 — Scope Assessment & Testing Plan

First, assess scope size:

- **Small** — single file, single concern: execute Phases 3–5 inline, one verification pass.
- **Medium** — 2–4 files, one system: execute Phases 3–5 inline, one verification pass per file.
- **Large** — 5+ files, multiple systems, or cross-repo changes: **split into stages**. Each stage must be independently testable and verifiable. Spawn one `executor` agent per stage. Brief each agent fully — it has no context from this conversation. If remaining context is substantial after completing a stage, use `/compact` before spawning the next agent.

For large scope, define stage boundaries before writing any tests:

Output for large scope:
```
Stage 1: <what it covers>
Stage 2: <what it covers>
...
```

Confirm the stage breakdown with the user before proceeding. Then execute Phases 3–5 **per stage**, in order. Do not begin Stage N+1 until Stage N is verified.

---

For each stage (or for the whole scope if small), produce an explicit testing plan:

- List every test case by name and expected behaviour
- Identify what must be mocked vs. what must hit real infrastructure
- Note which cases are happy path, which are error path, which are edge cases
- Confirm the plan with the user before proceeding

Output format:
```
Test cases:
- test_<name>: <what it asserts>
- test_<name>_error: <what error condition it covers>
...

Mocked: <list>
Real: <list>
Skipped (out of scope): <list>
```

Write the testing plan to:
```
/tmp/$PROJECT/$BRANCH/test-plan.md
```

Both agents (test-engineer, executor) must read this file at the start of their phase instead of relying on conversation context.

### Phase 3 — Failing Tests (Red)

Delegate to `test-engineer` (sonnet):

> "Write the following tests per the plan below. Run them with `pytest <test_file> -v`. Confirm every test fails for the right reason — not import errors or wrong assertions. Report any test that passes before implementation exists.
> [paste testing plan]"

Do not proceed to Phase 4 until `test-engineer` confirms all tests fail correctly.

### Phase 4 — Minimal Implementation (Green)

Delegate to `executor` (sonnet):

> "Implement only what is needed to make the following failing tests pass. No speculative features. No refactoring beyond what the tests require. Follow `doc/coding_style.md`. Add Sphinx/reStructuredText docstrings to all new and modified public functions, methods, and classes per `doc/docstring_style.md`. Run `pytest <test_file> -v` and confirm all tests pass.
> [paste test file path and testing plan]"

Do not proceed to Phase 5 until `executor` confirms all tests pass.

### Phase 5 — Verify

Delegate to `verifier` (sonnet):

> "Verify the following implementation against the original plan. Run `pytest` (full suite), run `ruff check`, confirm no regressions. Then review the Phase 1 requirements and confirm each is addressed with no unplanned behaviour introduced. Finally, perform a production-grade backend code review against the criteria below.
> [paste Phase 1 interview output and implementation summary]"

**Code review criteria for verifier:**

1. **Correctness** — implementation matches plan; no logical inconsistencies or missing edge-case handling
2. **Code quality** — readability, maintainability, clear abstractions, appropriate typing and structure
3. **Performance** — flag N+1 queries, unnecessary DB round trips, inefficient iteration/memory usage, query scalability issues
4. **Database discipline** — flag DB session acquisition inside utility/helper functions, missing schema qualification, unsafe bulk insert/update behaviour, transaction boundary issues
5. **Architecture** — prefer explicit typed structures (Pydantic models, enums) where they improve clarity; avoid unnecessary abstraction
6. **Security / reliability** — flag input validation gaps, unsafe query construction, improper error handling, resource lifecycle issues

Output format:
- `PASS`
- `ISSUES FOUND` — tagged `[BLOCKING]`, `[SHOULD FIX]`, or `[NIT]`

`verifier` reports findings back to the orchestrator. Do not loop directly with `executor`.

#### If verifier reports issues — Fix Loop

The orchestrator triages the report:

| Issue type | Action |
|------------|--------|
| Failing tests / regressions | Re-brief `executor` with specific failures; re-run verifier |
| Logic drift from Phase 1 plan | Re-brief `executor` with the exact requirement mismatch; re-run verifier |
| Ambiguity requiring design decision | Re-engage `analyst` or re-interview; do not guess |
| Repeated failures (2+ cycles) | Escalate to `architect` (opus) for root cause before retrying |

Loop: orchestrator → executor → verifier → orchestrator, until all success criteria are met.

Success criteria (reported by `verifier`):
- All new tests pass
- No existing tests broken
- `ruff check` clean
- Implementation logic aligns to the plan agreed in Phase 1 — each requirement addressed, no unplanned behaviour introduced

### Phase 6 — Feature Documentation

Delegate to `writer` (haiku):

> "Write a feature documentation page for the following implementation. Use the Phase 1 interview output and implementation summary as source material. Cover:
> 1. **Overview** — what the feature does and why it exists
> 2. **Business logic** — key rules, constraints, and decision points
> 3. **Workflow diagram** — Mermaid flowchart or sequence diagram showing the data/control flow
> 4. **Related modules** — which files are involved and their roles
> 5. **Edge cases and failure modes** — as identified during Phase 1
>
> Write to `doc/<feature-name>.md`. Link it from any existing doc index if relevant.
> [paste Phase 1 interview output and implementation summary]"

## Project-specific constraints

- **No HTTP 500 assertions** in API tests — all errors must be handled explicitly and return 4xx or appropriate codes; a 500 in a test means missing error handling
- **AsyncTask fields** (`token_usage`, `json_result`) must be written in callbacks, not assumed to be set elsewhere
- **Celery task routing**: all business logic (ML classify, parser selection) lives in `doc-ai` scheduler (`cron/jobs.py`), not in `celery_cloud`

## Pitfalls

- Skipping Phase 2 and writing tests ad hoc leads to gaps in edge case coverage
- Writing implementation before confirming tests fail for the right reason masks test bugs
- Over-implementing during Green phase — stop when tests pass, refactor separately

## Delegation

| Phase | Agent | Model |
|-------|-------|-------|
| 1 — Requirements interview | `analyst` (complex domain) or orchestrator | opus / inline |
| 2 — Scope & testing plan | orchestrator (confirm with user) | inline |
| 3 — Failing tests | `test-engineer` | sonnet |
| 4 — Implementation | `executor` | sonnet |
| 5 — Verification | `verifier` | sonnet |

Each agent receives the output of the previous phase as explicit context in its prompt. Phases 3–5 repeat per stage for large-scope tasks.
