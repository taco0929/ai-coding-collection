Decompose the task into implementation subtasks and execute them.

## Task sizing

Before starting, assess total scope:

- **Small** — single file, single concern: execute inline, one review pass.
- **Medium** — 2–4 files, one system: execute inline, one review pass per file.
- **Large** — 5+ files, multiple systems, or cross-repo changes: **spawn one Agent per logical subtask** using the Agent tool (`subagent_type="general-purpose"`). Brief each agent fully — it has no context from this conversation.

For large tasks, if remaining context is substantial after completing a subtask, use `/compact` before spawning the next agent to avoid context pressure affecting later work.

## Execution

After each implementation pass, perform a production-grade backend code review.

Review criteria:

1. Correctness

* Verify implementation matches the plan and requirements
* Identify logical inconsistencies or missing edge-case handling

2. Code quality

* Readability
* Maintainability
* Clear abstractions
* Appropriate typing and structure

3. Performance
   Pay special attention to:

* N+1 queries
* Unnecessary DB round trips
* Inefficient iteration / memory usage
* Query scalability

4. Database discipline
   Flag:

* DB session acquisition inside utility/helper functions
* Missing explicit schema qualification where required
* Unsafe bulk insert/update behavior
* Transaction boundary issues

5. Architecture
   Prefer explicit typed structures when they improve clarity and validation (e.g. Pydantic models, enums), but avoid unnecessary abstraction.

6. Security / reliability
   Check for:

* Input validation gaps
* Unsafe query construction
* Improper error handling
* Resource lifecycle issues

Output format:

* PASS
* ISSUES FOUND

  * [BLOCKING]
  * [SHOULD FIX]
  * [NIT]



If issues are found, perform targeted revision passes and re-review until resolved.

Do not invent missing requirements.
If clarification is needed, stop and ask.
