# agy-delegation

Shared rules for delegating implementation work to the Antigravity CLI (`agy`) while staying the
reviewer. Written to be read directly by any agent CLI (Codex, Claude Code, etc.) — plain
Markdown, no tool-specific format.

The content lives in [agy-delegation.md](agy-delegation.md).

## Init: adapter or manual

**If you are an agent (Claude Code, Codex, etc.) setting this skill up in a project for the first
time, stop and ask the user which mode they want before creating any file.** Don't default to one
silently — they have different tradeoffs and the wrong one is annoying to unwind later.

Ask something like: *"Set up `agy-delegation` as a submodule adapter (stays in sync with this repo,
needs `git submodule update` to pick up changes) or copy the full rules in directly (self-contained,
but you own future updates by hand)?"*

| | **Adapter** (submodule) | **Manual** (inline copy) |
|---|---|---|
| Setup | `git submodule add` + a 6-line pointer `SKILL.md` | Copy [templates/SKILL-manual.md](templates/SKILL-manual.md) as-is |
| Stays current | Yes, via `git submodule update --remote` | No — edits here don't propagate; someone has to notice and re-copy |
| Works offline / no extra clone | No — needs the submodule fetched | Yes — one self-contained file |
| Right for | Multiple projects sharing the same rules, or you expect to tweak `agy-delegation.md` centrally | A single project, a fork you intend to diverge, or an environment where submodules are awkward (CI containers, one-off clones) |

### Adapter mode

```bash
git submodule add <this-repo-url> .agents/agy-delegation
mkdir -p .claude/skills/agy-delegation
cp .agents/agy-delegation/templates/SKILL-adapter.md .claude/skills/agy-delegation/SKILL.md
```

That gives the host project `.agents/agy-delegation/agy-delegation.md` plus a thin pointer skill.
For non-Claude-Code CLIs (Codex, etc.), just point whatever config those tools use straight at
`.agents/agy-delegation/agy-delegation.md`.

### Manual mode

```bash
mkdir -p .claude/skills/agy-delegation
curl -sL https://raw.githubusercontent.com/Aape1231/agy-delegation-skill/master/templates/SKILL-manual.md \
  -o .claude/skills/agy-delegation/SKILL.md
```

No submodule, no external dependency — the full rules live in that one file. Rerun the `curl` by
hand whenever you want to pick up upstream changes.

## Updating

Edit `agy-delegation.md` here, commit, push. In each host project:

```bash
git submodule update --remote .agents/agy-delegation
git add .agents/agy-delegation
git commit -m "chore: bump agy-delegation skill"
```
