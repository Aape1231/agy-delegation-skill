# agy-delegation

Shared rules for delegating implementation work to the Antigravity CLI (`agy`) while staying the
reviewer. Written to be read directly by any agent CLI (Codex, Claude Code, etc.) — plain
Markdown, no tool-specific format.

The content lives in [agy-delegation.md](agy-delegation.md).

## Mounting this into a project

This repo is meant to be pulled into a project as a git submodule, not copy-pasted, so every
project stays on the same version and picks up fixes with a normal submodule update.

```bash
git submodule add <this-repo-url> .agents/agy-delegation
```

That gives the host project `.agents/agy-delegation/agy-delegation.md`.

### Claude Code

Claude Code discovers skills via `.claude/skills/<name>/SKILL.md` — a file with YAML frontmatter
(`name`, `description`) that Claude uses to decide when to auto-trigger the skill. Point it at the
submodule instead of duplicating the content:

`.claude/skills/agy-delegation/SKILL.md`:

```markdown
---
name: agy-delegation
description: How to delegate implementation work to the Antigravity CLI (`agy`) while staying the
  reviewer — dispatch mechanics, model selection, plan-first gating, scope discipline, the
  acceptance standard for delegated changes, and the environment traps that fake a failure. Use
  whenever work is being handed to `agy`, Antigravity, or a local coding agent; whenever someone
  asks to parallelise, delegate, or supervise agent work in this repo; and whenever reviewing a
  change an agent produced.
---

This skill is a thin adapter. The content is version-controlled so that every agent CLI reads the
same copy, and lives at:

**[.agents/agy-delegation/agy-delegation.md](../../../.agents/agy-delegation/agy-delegation.md)**

Read that file now and follow it.
```

### Codex / other CLIs

Point whatever config those tools use at `.agents/agy-delegation/agy-delegation.md` directly.

## Updating

Edit `agy-delegation.md` here, commit, push. In each host project:

```bash
git submodule update --remote .agents/agy-delegation
git add .agents/agy-delegation
git commit -m "chore: bump agy-delegation skill"
```
