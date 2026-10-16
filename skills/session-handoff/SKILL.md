---
name: session-handoff
description: Use when a work session is ending, when the user types /session-handoff, or asks to wrap up, close out, or hand off the session so the next one continues without re-asking. Updates the project's recorded state (handoff file, pending list, decisions log) from evidence gathered in the session, after a read-only verification pass and a single user approval. Captures state; never fixes, deploys, or pushes.
metadata:
  version: "1.0.0"
---

# Session Handoff

## Overview

A close-out protocol for **operational continuity between sessions**. It is not a summary of the conversation: it
updates the project's recorded state so the next session knows what exists, what was done, what was decided,
authorized and forbidden, and what is left, **without asking again**.

Core principle: **evidence before assertion.** Nothing becomes DONE, a fact, or a decision without proof from this
session.

This protocol **captures state; it does not tidy anything.** Closing a session is not the time to fix, clean up,
restart, or deploy.

## Where each kind of information lives (one owner per type)

| Information | Owner | In this protocol |
|---|---|---|
| Real state | the system itself (repo, servers, CI) | read-only checks |
| Stable state: architecture, services, conventions | the project docs (README, `docs/`) | only **point at** what became outdated |
| Pending work | `docs/pending.md` | update |
| Decisions, restrictions, authorizations | `docs/decisions.md` | add or change status, with confirmation |
| What happened this session and what comes next | `HANDOFF.md` (repo root) | **overwrite** (history lives in git) |
| User preferences, lessons for the assistant | the assistant's memory, if the harness has one | only with confirmation |

Use the project's own equivalents if they already exist (an `ADR/` folder, a `TODO.md`, a `CHANGELOG`). Create the
three files above only when the project has nothing comparable, and never invent a fourth context file.

Never copy stable state into the handoff or into memory: point at the file that owns it.

When sources disagree, trust them in this order: **observed system state > changes proven this session > the user's
words > docs > memory > inference.** A disagreement becomes a **CONFLICT** entry in the handoff; never pick a side
silently.

## Steps

### 1. Harvest from the conversation (no commands)

List each item with its evidence (commit, command output, validation, the user's exact words):

- **Real changes**: what changed in the system or repos (BEFORE → AFTER when it helps). Only what was executed and
  confirmed; a plan that was not executed is not a change.
- **Task status**: DONE (with evidence) · PARTIAL · INVESTIGATED · BLOCKED · DEFERRED · NOT STARTED.
  "Discussed" is not DONE.
- **New decisions / restrictions / authorizations** from the user, with the **exact phrase**, date, scope, and
  validity. A suggestion of yours that the user did not confirm is **not a decision**. An authorization with no
  stated validity lasts only this session. Authorization for one action does not cover "similar" actions.
- **New preferences** (how the user likes to work): memory candidates.
- **Pending items**: for each, whether it needs the user, can be done autonomously, depends on something else, or is
  only a future idea.
- **Open hypotheses and investigations**, labeled as such.

### 2. Read-only checks (about a minute)

Run whatever the project documents as its health check, plus a repository status sweep:

```bash
git status -sb && git status --porcelain | wc -l      # uncommitted work? ahead/behind the remote?
git log --oneline -10                                  # commits made this session
# project health check, if one is documented (test suite, lint, `make check`, a status script, ...)
```

Then, for each path, name, or service touched in the session, `grep -rn` the docs and READMEs. Docs that still
describe the old state are **"doc possibly outdated"**. Also compare memory × docs × the decisions log.

Allowed here: read, list, `git status/log/diff`, inspect commands, `grep`, and running the test suite if it has no
side effects. **Forbidden** during close-out: deploy, restart, delete, cleanup, reset, push, touching secrets or
production, even when it looks obvious. If something needs that, it becomes a pending item.

### 3. Show the draft and ask for a single "ok"

Present to the user:

1. the draft `HANDOFF.md` (format below);
2. the **list of proposed writes**, each with its file:
   - `HANDOFF.md` (overwritten) and `docs/pending.md`;
   - new or changed entries in `docs/decisions.md`, one per line, with the source phrase;
   - proposed memories, marked **MEMORY CANDIDATE**;
   - fixes to outdated docs (only the clearly necessary and safe ones; list the rest).

Ask once: "ok to write everything? (or 'ok, except X')". Write nothing before the answer.

### 4. Write and commit locally

After the ok, write the approved files, `git add` only those, and make a **local commit** with the message
`handoff: <date> — <session goal>`. **Do not push** unless the user asks for it, as a separate step. Report the
commit hash and what was left out.

## HANDOFF.md format (max ~40 lines)

```markdown
# Handoff — <project>
**Session:** YYYY-MM-DD · **Goal:** … · **Status:** completed | partial | interrupted
**Verified at close:** <health check result> · repos <up to date | pending: …>

## Done
- DONE <what> — <evidence: commit/validation>

## Not finished
- PARTIAL|BLOCKED|DEFERRED|INVESTIGATED <what> — missing: … — depends on: user | nothing | <task>

## Recorded in docs/decisions.md
- <id> <type> <summary> (<status/validity>)

## Attention
- active risks (broad permissions, open security items, ...)
- CONFLICT: <source A> says X; <source B> shows Y — needs validation

## Next session
1. <only what really needs resuming; nothing invented>

## Not applied
- MEMORY CANDIDATE: …
- Doc possibly outdated: <file:line> — …
```

Empty sections get "—" instead of invented content. Mark confidence when it is not obvious:
`[confirmed YYYY-MM-DD via <source>]`, `[unverified]`, `[hypothesis]`.

## Decisions log format

Each entry has an id, a type, a status, and a source phrase.

- **Types:** DECISION (a choice made), RESTRICTION (what not to do without an explicit request), AUTHORIZATION
  (permission for an action, **only within the written scope**, never generalized).
- **Status:** `in force` · `to confirm` (inferred, or said in a specific context; ask before treating as a rule) ·
  `expired` · `revoked` · `superseded by <id>`.
- **Authorization validity:** default is the session in which it was given. Longer only if the user says so.

## Memory: what goes in and what does not

Goes in (with confirmation): user preferences, lessons that prevent a repeated mistake, stable conventions.
Does not: system state (it lives in the docs), commands that were run, logs, transient results, hypotheses, anything
that goes stale in weeks. When in doubt: **MEMORY CANDIDATE**, do not write.

## Common mistakes

| Mistake | Fix |
|---|---|
| Marking DONE something that was only planned or discussed | DONE requires citable evidence |
| Recording the assistant's suggestion as the user's decision | Only the user's phrase becomes a decision; the rest is a recommendation |
| Authorization without scope or validity, or generalized | Literal scope; default validity = the session |
| Repeating stable state in the handoff | Point at the doc; handoff = delta + next steps |
| "Taking advantage" of the close-out to fix something | It becomes a pending item; close-out is read-only until the ok |
| Resolving a conflict by choosing one version | Record a CONFLICT with both sources |
| Pushing together with the commit | Never; push only on request |
| Oversized handoff | ~40 lines; detail lives in commits and docs |
