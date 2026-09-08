---
name: agy-delegation
description: How to delegate implementation work to the Antigravity CLI (`agy`) while staying the
  reviewer — dispatch mechanics, model selection, plan-first gating, scope discipline, the
  acceptance standard for delegated changes, and the environment traps that fake a failure. Use
  this skill whenever work is being handed to `agy`, Antigravity, or a local coding agent;
  whenever someone asks to parallelise, delegate, or supervise agent work in this repo; and
  whenever you are reviewing a change an agent produced and need to decide whether to accept it.
  Also use it when an agent run "finishes" with a clean worktree and almost no output, because
  that has several different causes and they are not distinguishable without this skill.
---

This skill is a thin adapter. The content lives in the `agy-delegation` submodule (its own repo,
so every project that pulls it in reads the same version-controlled copy, and any agent CLI —
Codex, Claude Code, etc. — can read it directly, not just this one):

**[.agents/agy-delegation/agy-delegation.md](../../../.agents/agy-delegation/agy-delegation.md)**

Read that file now and follow it.
