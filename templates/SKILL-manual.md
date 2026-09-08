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

# Delegating to the Antigravity CLI

You stay the reviewer. `agy` writes the code. A *different* `agy` session verifies it.
You decide what gets committed and you never accept a claim you have not seen evidence for.

This split exists because an agent's exploration does not enter your context — you read only the
final diff and the evidence. That is where the savings come from. The risk it creates is that you
lose sight of *how* a result was produced, so the acceptance standard below is what keeps the
arrangement honest.

Works the same whether you are Claude Code or Codex: everything here is shell plus `agy`.

## Dispatch

Write the task to a file, then pass it in. Long prompts inline get mangled by quoting.

```bash
agy --print "$(cat task.txt)" \
    --model <id> [--effort high] \
    --mode plan|accept-edits \
    --print-timeout 40m \
    --dangerously-skip-permissions \
    > out.txt 2>&1
```

Run it inside a **tracked background command** of your harness, not `nohup ... &`. With `nohup`
you lose the exit code, and a timeout, a bad flag and a real failure all collapse into the same
symptom: an empty file. You need to tell them apart.

| Model id | `--effort`? | Use for |
|---|---|---|
| `gemini-3.8-flash-high` | yes | default; fast and reliable for scoped work |
| `gemini-3.7-flash-medium`, `gemini-3.6-flash-medium`, `gemini-3.1-pro-low` | yes | fallbacks |
| `claude-sonnet-4-6` | **no** | mechanical execution of an approved plan |
| `claude-opus-4-6-thinking` | **no** | design decisions with several interacting constraints |
| `gpt-oss-120b-medium` | no | last resort |

Passing `--effort` to a Claude model fails with *"--effort is not supported"*. Effort is already
baked into those ids.

## Plan first when the design is not settled

`--mode plan` produces a plan and writes nothing. Read it, decide, then dispatch execution with
`--mode accept-edits` restating what you accepted and what you changed.

This is worth the extra round trip whenever a wrong design would be expensive to unwind: schema
changes, anything touching money or auth, anything where two fixes have to agree with each other.
It has already caught a plan that would have fixed a vulnerability while silently deleting the
coverage proving the feature still worked.

Give the plan a way to disagree with you. Ask "am I wrong to pair these?" or "verify this claim and
correct me". Briefs written from memory contain errors, and an agent that has been invited to push
back will say so instead of designing on top of a false premise.

## Scope

Every task carries an explicit file list. Anything outside it requires coming back to you first —
even when the agent is right, because a change you did not scope cannot be reviewed against
anything.

State the prohibitions plainly in the task: no `git push`, no `git commit`, no `git add`, none of
`checkout`/`stash`/`restore`/`reset`/`clean`, no edits to already-pushed migrations, nothing that
reaches a remote. The worktree is usually shared with work that is not committed yet.

## Acceptance

This is the part that matters. A delegated change is not accepted because the tests pass.

**Every new assertion must fail against the committed code and pass against the change.** Prove it
by swapping the file back with `git show HEAD:<path>` and rerunning. An assertion green in both
states protects nothing while reading as protection — the most expensive kind of defect, because it
retires the question.

**Refuse assertions on source text.** Checking that a literal string appears in an implementation
breaks on honest refactors and stays silent on real regressions. If a property can only be
expressed that way, it is usually not the property you want.

**Refuse test-only hooks in production code.** A caller-controlled delay or bypass added so a test
can observe a race is a backdoor in the shipping path. When a race genuinely cannot be observed
from outside, take the documented gap instead: keep the guard, write a comment next to it saying it
has no end-to-end coverage and why. An honest gap is worth more than a test that manufactures its
own green.

**Check the failure reason, not just the failure.** An assertion can fail against old code for a
reason unrelated to the defect — a missing credential, an unrelated error. It still discriminates,
but it proves less than its name suggests, and that belongs in the commit message.

**Run the suite in more than one order.** Fixtures that fight a database trigger produce a suite
that passes or fails according to what ran before it. A single green run cannot detect that, since
the bug is precisely that a green run exists.

**Have a different session verify.** The author's own run is evidence, not proof. Ask the verifier
specifically: is any new assertion green against both versions, and does each failure fail for the
right reason.

## Traps that fake a failure

Before concluding an agent broke something, rule these out. Each one has produced a convincing
false failure.

- **A run "succeeded" with a clean worktree and a tiny output file.** Three different causes: the
  CLI timed out, a flag was rejected, or the model stopped mid-investigation. Read the output;
  changing models for a flag error wastes a model that was fine.
- **`zsh` does not word-split unquoted variables.** `CMD="python -m pytest"; $CMD file` looks for a
  binary with spaces in its name. Use a shell function.
- **BSD `xargs` has no `-a`.** Redirect instead.
- **A readiness probe that accepts 503.** Kong answers 503 while the edge worker boots. Poll until
  the status is not 000, 502, 503 or 504 — the CI workflow already does this; match it.
- **A virtualenv missing `pyvenv.cfg`.** `site-packages` is intact but the interpreter falls back to
  the system one, and every import fails as though nothing were installed. Check `sys.prefix`.
- **Tests run without the fixture step the gates apply.** A reset database is not a seeded one.
- **Two agents on one local stack.** A `db reset` under another agent's run destroys its evidence.
  Serialise anything that touches the database; run read-only analysis in parallel instead.

Tell the agent not to trust your description of the environment either — including the sentence
saying the stack is up. Have it check, and report the discrepancy if it finds one.

## Committing

Gate the commit on the whitespace check actually passing:

```bash
if git diff --check; then git add <paths> && git commit -F -; else echo "blocked"; fi
```

Writing the check and the commit as separate statements lets a failing check print a warning while
the commit proceeds anyway. Stage explicit paths, never `-A`, since another agent may have work in
flight in the same tree.

Say in the message what the change does not cover. The gaps are the part a future reader cannot
reconstruct.
