# session-handoff

A Claude Code skill that closes a work session by updating the project's recorded state, so the next session
continues where this one stopped **without re-asking what was already decided, authorized, or forbidden**.

It is not a conversation summary. It is a small, evidence-based protocol with one rule at its core: nothing is
recorded as done, decided, or true without proof from the session.

## What it does

1. **Harvests** from the conversation: real changes, task status (`DONE` only with evidence), new decisions,
   restrictions and authorizations (with the user's exact phrase, scope and validity), pending items, open
   hypotheses.
2. **Verifies read-only**: repo status, recent commits, the project's own health check if there is one, and a
   `grep` for docs that still describe the state you just changed.
3. **Shows a draft** of `HANDOFF.md` plus the exact list of files it wants to write, and asks for **one** "ok".
4. **Writes and commits locally** (`handoff: <date> — <goal>`). It never pushes on its own.

## Design choices

- **One owner per kind of information.** Stable facts stay in the docs; the handoff only holds the delta and the
  next steps (about 40 lines, overwritten each time, with history in git).
- **Capture, don't fix.** Close-out is read-only: no deploy, restart, cleanup, or push. Anything that needs one of
  those becomes a pending item.
- **Decisions come from the user, not the assistant.** A suggestion the user never confirmed is a recommendation.
  Authorizations are scoped literally and last only the session unless the user says otherwise.
- **Conflicts are surfaced, not resolved.** Observed system state beats session changes, which beat the user's words,
  docs, memory, and inference; a disagreement is written down as `CONFLICT`.

## Files the skill maintains

| File | Role | Update mode |
|---|---|---|
| `HANDOFF.md` | what happened, what is next, risks, conflicts | overwritten each session |
| `docs/pending.md` | open work with its dependency (user / autonomous / other task) | edited |
| `docs/decisions.md` | decisions, restrictions, authorizations with source phrase and status | appended, with confirmation |

If your project already has equivalents (ADRs, `TODO.md`), the skill uses them instead of creating new files.

## Install

```bash
git clone https://github.com/alexandrenunes-cs/engineering-solution-pivot.git
mkdir -p ~/.claude/skills
cp -r engineering-solution-pivot/skills/session-handoff ~/.claude/skills/
```

For a single project, copy it into `.claude/skills/` instead. Start a new Claude Code session and run:

```
/session-handoff
```

or just say "wrap up the session" / "hand off".

## Adapting it to your project

Edit step 2 of `SKILL.md` to list your real health checks (test command, status script, deploy dry-run). The more
precise that list, the more the handoff can claim as verified. If you want a project-specific close-out (for
example for infrastructure), copy the skill, rename it, and tighten the "forbidden during close-out" list to your
environment.
