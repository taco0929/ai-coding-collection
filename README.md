# ai-coding-collection

A personal collection of AI coding conventions, skills, and workflows for use with [Claude Code](https://claude.ai/code) and [oh-my-claudecode (OMC)](https://github.com/Yeachan-Heo/oh-my-claudecode).

## Structure

```
.claude/
  commands/     # Reusable Claude Code slash commands
  doc/          # Coding conventions and style guides
.omc/
  skills/       # Custom OMC workflow skills
```

## Contents

### Coding Conventions (`.claude/doc/`)

Project-agnostic coding style rules to include in Claude Code projects via `@path/to/doc` references in `CLAUDE.md`.

### Commands (`.claude/commands/`)

Reusable slash commands for common development tasks.

### Skills (`.omc/skills/`)

Custom OMC workflow skills. Invoke via keyword triggers in any session running `omc`.

| Skill | Trigger | Description |
|-------|---------|-------------|
| `plan-tdd` | `plan-tdd: <task>` | Requirements interview → testing plan → failing tests → implementation → verify → documentation |

## Usage

Reference docs and skills from any project's `.claude/CLAUDE.md`:

```markdown
@/path/to/ai-coding-collection/.claude/doc/coding_style.md
```

Or copy skills into a project's `.omc/skills/` directory to make them available locally.

## Requirements

- [Claude Code](https://claude.ai/code)
- [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode) for skill support
