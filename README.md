# Engineering Solution Pivot

**A deep-reasoning escalation skill for Claude Code — for when the obvious engineering path is exhausted.**

Engineering Solution Pivot is **not** a default workflow. For normal engineering work, keep using normal development, debugging, architecture, testing, and documentation practices.

Reach for this skill deliberately, when those practices stop producing progress. Typical signs: you are stuck, workarounds keep piling up, a constraint looks impossible, or a decision is important enough to deserve deeper and more skeptical analysis than a normal workflow provides.

```
NORMAL ENGINEERING WORK
  → normal development, debugging, architecture, testing, and documentation workflows

STUCK / HIGH-COMPLEXITY / NON-OBVIOUS PROBLEM
  → invoke /engineering-solution-pivot
```

---

## When to reach for it

Use it when:

- you have tried the obvious implementation paths and are still blocked;
- a technical problem seems to have no straightforward solution;
- each workaround is more complex than the one before;
- the team is stuck in a local optimization and cannot see an alternative architecture;
- an apparently hard technical constraint may be framed incorrectly;
- the problem might belong at a different abstraction layer;
- the current architecture itself may be the source of the limitation;
- you want an independent, skeptical technical analysis of a plan;
- a significant architectural, security, reliability, performance, or AI/ML engineering decision needs deeper analysis;
- you need to think outside the box on purpose, instead of iterating on the same approach again;
- the problem calls for unconventional, but technically defensible, solution paths.

Do **not** use it for routine tasks that already have a clear, working path. Not every problem is an architectural crisis.

## The core idea

A normal workflow asks: *"How do I make the current approach work?"*

Engineering Solution Pivot asks deeper questions:

- Are we solving the right problem?
- Is the stated constraint actually real?
- Is the constraint fundamental, or just an artifact of the current architecture?
- Are we solving the problem at the wrong abstraction layer?
- Can the interface, dependency, architecture, workflow, protocol, data flow, or boundary change?
- Are we accumulating workarounds instead of fixing the root cause?
- Is there a completely different technical path?
- **What would make the current limitation disappear, instead of just working around it?**

The goal is not creativity for its own sake. The goal is **evidence-driven engineering reasoning** that finds a solution path the original framing of the problem hid.

## Depth over speed

The skill intentionally trades speed for depth. It pushes toward:

- deeper investigation and root-cause analysis;
- independent reasoning, including challenging the problem framing and the assumptions behind it;
- alternative architectures and **constraint transformation**, meaning a design change that removes a constraint instead of working around it;
- adversarial validation, with explicit trade-off, security, and reliability analysis;
- verification before conclusions, and conclusions before changes.

It is equally explicit about what to avoid:

- endless brainstorming or random alternatives;
- speculative solutions presented without evidence;
- criticism for its own sake, or invented problems;
- replacing a simple solution with an elaborate one;
- treating every problem as an architectural crisis.

**In short:** use it when the obvious path is exhausted or insufficient, and go deeper than a normal engineering workflow would.

## What the skill does

The skill works through a gated reasoning model. Stages that don't apply are skipped, and earlier stages are revisited when new evidence contradicts them:

| # | Stage | Purpose |
|---|---|---|
| 0 | Checkpoint | Stop repeated attempts that share one failing hypothesis |
| 1 | Frame the objective | Separate the desired outcome from the proposed implementation |
| 2 | Establish evidence | Label each claim *Verified*, *Inferred*, *Assumed*, or *Unknown*, and rank the sources |
| 3 | Diagnose | Separate symptom from cause, and find the cheapest check that tells hypotheses apart |
| 4 | Classify constraints | Technical limit, security control, regulatory or contractual, organizational, preference, design artifact, or unverified assumption |
| 5 | Transform or work around | Apply 11 design levers so the constraint no longer applies |
| 6 | Generate options | Materially different options, each labeled *Verified*, *Plausible*, or *Speculative* |
| 7 | Adversarial review | Failure modes, races, partial failures, security, operability — without fabricated problems |
| 8 | Decide | Keep, investigate, pivot, escalate, or "no safe option identified" |
| 9 | Validate safely | Scope, blast radius, signal, abort condition, rollback |
| 10 | Stop | Stop when more information would not change the decision, including when the pivoting itself has become thrashing |

**The 11 transformation levers:** interface or contract, protocol, abstraction layer, architecture, dependency, data flow, deployment or runtime model, integration boundary, ownership boundary, workflow, and requirement.

**Safety is built in:**
- security controls are treated as information, not as obstacles;
- the skill never bypasses authentication, sandboxing, or approval gates;
- analysis is kept separate from execution, and high-impact actions require explicit human approval.

The method is domain-agnostic. Short *domain lenses* adapt it to software engineering, architecture and distributed systems, infrastructure, DevOps and SRE, security, data engineering, performance, AI/ML and LLM agent systems, automation, and developer tooling.

## Example

> **Engineer:** Our deploy has been failing since the credentials rotation. We've retried twice. Let's migrate to a different deploy tool.

A normal workflow might start the migration. The skill first checks whether a pivot is actually justified:

```
Engineering Solution Pivot
Objective:        Deploys succeed again with the rotated credentials
Evidence:         [Verified] deploy step returns "invalid key format" since the rotation
Diagnosis:        tool failure (unlikely) vs. malformed secret (likely) → read the raw error
Recommendation:   Keep the current tool — fix the secret, add a format check to the pipeline.
                  Confidence: high. No migration warranted.
Validation:       re-run the deploy on staging; rollback = previous secret version
```

In cases where a pivot *is* justified — an inbound-webhook requirement blocked by network policy, for example — the skill looks for a way to change the design so the constraint no longer applies, such as switching from push to pull, before it accepts a workaround like a firewall exception.

## Installation

This is a skill for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

**For your user** (available in all projects):

```bash
git clone https://github.com/alexandrenunes-cs/engineering-solution-pivot.git
mkdir -p ~/.claude/skills
cp -r engineering-solution-pivot/skills/engineering-solution-pivot ~/.claude/skills/
```

**For a single project** (shared with your team through the repository):

```bash
mkdir -p .claude/skills
cp -r engineering-solution-pivot/skills/engineering-solution-pivot .claude/skills/
```

Start a new Claude Code session. Then invoke the skill explicitly:

```
/engineering-solution-pivot
```

You can also ask for it in plain language, for example *"we're stuck, give me a pivot analysis"*, *"challenge this design adversarially"*, or *"is this constraint actually real?"*. Claude may also load the skill on its own when it detects strong signals, such as the same hypothesis failing repeatedly.

## Repository structure

```
skills/
  engineering-solution-pivot/
    SKILL.md      # triggers, reasoning model, agent rules, output format, examples
LICENSE
README.md
```

## Contributing

Issues and pull requests are welcome, especially:

- real cases where the skill activated when it should not have, or failed to activate when it should have;
- better generic examples of constraint transformation;
- clearer wording that makes the method shorter without losing capability.

## License

[MIT](LICENSE)
