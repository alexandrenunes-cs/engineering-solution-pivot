---
name: engineering-solution-pivot
description: Deep-reasoning escalation method, not a default workflow, for engineering problems where the obvious path is exhausted. It is evidence-driven, escapes blocked or thrashing approaches, and challenges designs, constraints, and assumptions — including transforming a constraint rather than working around it. Use when the same causal hypothesis has failed twice; a workaround is growing more complex than the problem it solves; a constraint, dependency, platform limit, or security control appears to make the goal impossible; a request is framed as "do X using Y" and Y may not be required; an architecture, migration, or design decision carries significant security, reliability, performance, or cost trade-offs; or the user asks for alternative approaches, a devil's-advocate or adversarial review of a technical plan, an engineering pivot, or invokes /engineering-solution-pivot. Not for routine tasks with a clear, working path.
---

# Engineering Solution Pivot

A deep-reasoning escalation method for technical problems where the obvious path is exhausted — blocked, failing, or suspiciously complex — and for stress-testing important designs before committing to them.

This is not the default workflow. Normal engineering work uses normal development, debugging, and design practices. Apply this method deliberately, when those practices stop producing progress or when a decision warrants deeper, more skeptical analysis. It trades speed for depth, but not for rigor: it does not brainstorm endlessly, does not propose alternatives without evidence, and does not treat every problem as an architectural crisis.

## Core principles

1. **Optimize the objective, not the first implementation chosen.** A proposed implementation is a hypothesis about how to reach an outcome, not the outcome itself.
2. **Ask both questions.** Not only *"How do we solve this within the current constraints?"* but also *"Can we change the architecture, abstraction, interface, dependency, data flow, workflow, or assumption so that the constraint no longer exists?"* Most wasted engineering effort comes from asking only the first.
3. **Diagnose before pivoting.** Many "impossible" problems are ordinary defects. Changing the architecture to escape a misconfigured secret is not a pivot; it is a misdiagnosis.
4. **Evidence over plausibility.** Every claim that a decision depends on is labeled by how it is known.
5. **Security controls are information, not obstacles.** When a control blocks progress, find the path its designers intended — never disable, weaken, or circumvent it for convenience.
6. **Recommendation is not authorization.** Analysis never implies permission to act.
7. **Stop when the decision is clear.** Exploration is bounded; more options are not better options.

## When to use it — and when not

Use it when there is evidence of one of these conditions:

| Signal | What counts | What does not count |
|---|---|---|
| **Repeated failure** | The same causal hypothesis has failed twice (for example, two variations of the same config change for the same assumed cause) | Transient failures; failures with unrelated causes; a retry that deliberately tests a *new* hypothesis |
| **Workaround growth** | Each fix adds components, special cases, privileges, or manual steps; the workaround is becoming more complex than the original problem | A single necessary adapter with a clear owner |
| **Apparent impossibility** | Documentation shows no path, a dependency lacks a feature, or a platform/security limit blocks the approach | A limit that has been verified as hard *and* whose removal has already been evaluated |
| **Implementation-framed request** | "Do X using Y" where Y may be an assumption rather than a requirement | Y is a verified requirement (contract, policy, compatibility) |
| **High-stakes decision** | Architecture, migration, security boundary, data model, or costly/irreversible change | Local, easily reversed choices |
| **Explicit request** | Alternatives, adversarial review, devil's advocate, pivot | — |

Do **not** use it for straightforward tasks with a working path, or to override a user's explicit and informed choice — in that case, state material risks once, clearly, and proceed as asked.

**Proportionality:** scale depth by *risk × uncertainty*, not by problem size. A low-risk, well-understood issue gets a few sentences. A production, security, or data-integrity decision gets the full method.

## The reasoning model

The model is gated: each stage has an exit condition. Stages are revisited when new evidence contradicts earlier conclusions. Skip stages that do not apply, but do not skip them silently when they do.

### 0. Checkpoint (when triggered by repeated failure)

Stop changing the system. State the attempts made so far and the causal hypothesis they share. Do not attempt a further variation of the same hypothesis without new evidence. Announce this as a review checkpoint, not as a diagnosis ("pausing to review the approach", not "the approach is wrong").

### 1. Frame the objective

- What outcome must be true at the end? What observable signal proves it (success criteria)?
- Who or what is affected? What outcomes are unacceptable (data loss, downtime, weakened security, broken contracts)?
- Separate the objective from the proposed implementation. If the request is "X using Y", ask whether Y is required, and why.
- For optimization work, identify the metric, the current baseline, and the target. Without a baseline, "faster" or "cheaper" is not a success criterion.

*Exit:* the objective and success criteria can be stated in one or two sentences without naming a specific implementation.

### 2. Establish evidence

Rank evidence by reliability and applicability:

1. Direct observation in the actual environment — tool output, logs, metrics, reproductions.
2. Source code and configuration of the exact versions in use.
3. Official documentation, release notes, and changelogs for those versions.
4. Maintainer statements — issue trackers, advisories, design documents.
5. Community reports — forums, Q&A sites, blog posts. Check the date and version.
6. Model knowledge and inference — lowest tier; always labeled as such.

Label every decision-relevant claim:

- **Verified** — observed directly or confirmed from an authoritative source for this version.
- **Inferred** — follows from verified facts but has not been checked directly.
- **Assumed** — taken as true without evidence; must be validated before the decision depends on it.
- **Unknown** — explicitly not known; state what would resolve it.

If the system or authoritative sources cannot be accessed, say so. Never present inference as verification.

*Exit:* the claims the decision depends on are labeled, and the critical assumptions are identified.

### 3. Diagnose (when something is failing)

- Distinguish the symptom from the cause. Reproduce the failure if it is safe to do so.
- Establish scope and timeline: what changed, where it fails and where it does not, when it started.
- Formulate two to four competing hypotheses, and for each identify the **cheapest discriminating check** — read-only first.
- Decide whether the approach is wrong or whether a correct approach is faulty in its implementation. Only the former justifies a pivot.

Diagnosis is bounded. If the root cause can be obtained cheaply, obtain it before generating alternatives. If it cannot be obtained — opaque vendor behavior, an unreproducible production issue — proceed while recording the uncertainty. For pure design decisions, where nothing is failing, skip this stage.

*Exit:* a leading hypothesis with its supporting evidence and confidence level, or an explicit statement that the cause is unknown and why.

### 4. Classify constraints

For each constraint that shapes the solution, determine its class, whether it has been verified, and who owns it:

| Class | Examples | How to treat it |
|---|---|---|
| Physical or technical limit | Latency floor, API does not expose an operation, protocol limitation | Hard within the current design — a candidate for transformation (stage 5) |
| Security control | Sandbox, permission boundary, network policy, authentication requirement, host security module | Satisfy it. Only its owner can change it, through a proper process. Look for the intended, supported path |
| Regulatory or contractual | Data residency, retention, SLA, license terms | Hard. Owned by legal, compliance, or the customer |
| Organizational standard | Approved stack, deployment process, coding standard | Negotiable with its owner, given evidence |
| Preference | Tool choice, style, familiarity | The user's decision. Surface trade-offs; do not override |
| Artifact of the current design | "The service must call X synchronously" | A prime candidate for transformation |
| Unverified assumption | "The library cannot do this", "the vendor does not support that" | Verify it before relying on it |

Ask of each important constraint: *Is it verified? Who owns it? What would it take to change it? Which assumption, if false, would make the problem disappear?*

*Exit:* the constraints the decision depends on are classified. Negotiable constraints are distinguished from hard ones.

### 5. Transform the constraint, or work around it

This is the defining step. Distinguish two kinds of response:

- **Workaround** — the constraint remains, and machinery is added to cope with it. Costs accumulate: more components, more failure modes, more privileges, more operational burden.
- **Transformation** — the design is changed so that the constraint no longer applies.

Before accepting a workaround, try each lever that is relevant:

| Lever | Example of transformation |
|---|---|
| Interface or contract | Push → pull; synchronous → asynchronous; per-item → batch; expose a narrower, purpose-built API |
| Protocol | Replace a chatty protocol with a streaming or bulk one; change the serialization boundary |
| Abstraction layer | Enforce an invariant in the database instead of application code; use a platform capability instead of custom code; move from code to configuration, or the reverse |
| Architecture | Move computation to the data; split or merge components; add or remove a boundary |
| Dependency | Replace, remove, upgrade, or adopt the officially supported alternative |
| Data flow | Precompute, cache, make operations idempotent, switch to event-driven propagation |
| Deployment or runtime model | Change where or how the workload runs — build service instead of a privileged local build, managed service instead of self-hosted |
| Integration boundary | Integrate at a different point — at the queue instead of the API, at the storage layer instead of the application |
| Ownership boundary | Have the owning team or vendor fix it upstream; request the capability; adopt a fix from a newer release |
| Workflow | Change when or by whom a step is done; remove the step entirely |
| Requirement | Negotiate or relax a requirement with its owner, supported by evidence of cost and risk |

Guidelines:

- **Subtract before adding.** Prefer removing a component, step, requirement, or dependency over introducing an adapter, wrapper, bridge, or proxy. Additive solutions are valid, but they carry ongoing cost and must justify it. Judge an adapter on its merits — correctness, security, maintenance, reversibility — not by labeling it a "hack".
- **Look for the intended path first.** Platforms often ship a supported mechanism for exactly the blocked case — a scoped permission, an official profile, an extension point, a newer API. Check for it before building around the limitation.
- **Research the objective, not only the failure.** Search for "how do others achieve Y when X is unavailable?", not just "how do I make X work?". Issue trackers, changelogs, and alternative implementations often reveal a different route. Apply the evidence ranking (stage 2) before trusting what you find.
- **Absence from documentation is not impossibility, and undocumented is not supported.** Approaches that depend on undocumented behavior are labeled as such, together with their stability and upgrade risk.
- **Pivot question:** *"If the current approach did not exist, how would we reach the objective?"*

### 6. Generate options

Consider every solution class, not a fixed number of options:

1. Fix the root cause within the current approach.
2. Change the implementation while keeping the design.
3. Transform the constraint (stage 5).
4. Adopt a supported platform or upstream capability.
5. Relax a negotiable constraint, with its owner.
6. Accept the limitation, defer the work, or do nothing.

Label each option:

- **Verified** — tested, or documented for this version.
- **Plausible** — consistent with the evidence, but untested.
- **Speculative** — depends on undocumented behavior or on unverified assumptions.

For each option, state the evidence that would disqualify it. Alternatives must be *materially* different — a different mechanism or a different failure mode — and the difference must be explained. Variations of the same idea do not count as separate options.

A single credible option is a valid outcome. So is "no safe option identified". Never pad the list with weak options to appear thorough.

### 7. Adversarial review

Attack the leading option *and* the current approach, including the user's proposal. Actively search for:

- **Correctness:** counterexamples, edge cases, incorrect problem framing.
- **Failure behavior:** partial failure, timeouts and retries, race conditions, ordering and concurrency issues, non-idempotent operations, crash and restart recovery, degraded dependencies, recovery procedures that fail.
- **Data:** integrity, migration and backfill, backward and forward compatibility, rollback of schema or data changes.
- **Security:** trust boundaries, privilege required and privilege granted, secrets handling, data exposure, network exposure, privilege escalation paths, supply chain, abuse cases.
- **Operability:** observability gaps, alerting, on-call burden, configuration drift, upgrade paths, hidden dependencies and coupling.
- **Economics:** infrastructure cost, operational cost, engineering effort, opportunity cost.
- **Complexity:** is there a simpler alternative, or a different abstraction layer?

Discipline:

- Classify each finding as **Confirmed** (evidence shown), **Evidence-backed risk** (a plausible mechanism, with likelihood and impact stated), **Hypothesis** (needs a test), or **Unknown**.
- Do not fabricate problems or inflate severity. A finding is worth stating only if it could change the decision, the design, or the validation plan.
- If the review finds nothing material, say so. A sound plan should not be made to look weak.

### 8. Decide

Compare the options against the objective, using only the criteria that matter for this problem:

- meets the success criteria;
- security; reliability; performance;
- cost — infrastructure, operational, engineering;
- complexity; maintainability;
- compatibility and migration;
- reversibility (a one-way or a two-way door);
- time to value;
- strength of the evidence.

Then choose one of these outcomes:

- **Keep the current approach** — with the specific fix.
- **Investigate further** — state exactly what, and why it could change the decision.
- **Pivot to option X.**
- **Escalate** — a decision is needed from the owner of a constraint or of a risk.
- **No safe option identified** — explain what is missing.

**Decision rights.** When the constraints clearly dictate a technical choice, recommend it firmly and explain why. Preferences, risk acceptance, cost trade-offs, and exceptions to policy belong to the user or to the relevant owner. Inform them; do not decide for them.

### 9. Validate safely

"Small and reversible" does not mean "safe". A reversible action can still cause downtime, data loss, exposure of data, or noisy side effects. Every experiment defines:

- the hypothesis, and the expected result if it is true and if it is false;
- scope and blast radius, preferring local, staging, a copy, a dry run, or read-only operations;
- the baseline and the signal to watch (metrics, logs, test output);
- an abort condition;
- rollback or recovery, and how the rollback itself is verified;
- whether approval is required (see the agent rules below).

Preferred order: **cheap test → evidence → decision**. Avoid: **large change → hope → diagnosis afterwards**.

### 10. Stop rules

Stop exploring and report when any of these holds:

- The evidence already discriminates between the leading options — more information would not change the decision.
- A validated option meets the success criteria.
- The remaining unknowns can only be resolved by the user, by an owner, or by an experiment that requires approval.
- The effort budget is spent. Set a budget up front, proportional to the stakes — hypotheses tested, tool calls, elapsed time. When it is exceeded, report the current state and ask how to proceed.
- **The pivot itself is thrashing.** Switching approaches repeatedly without new evidence is the same fixation as retrying one approach. Return to diagnosis (stage 3).

When stopping, report honestly: what is known, what is not known, the confidence level, and the single most useful next step.

## Operating rules for AI agents

**Evidence and honesty**

- Separate facts from assumptions, and cite the evidence: the command and its output, `file:line`, a documentation page and its version.
- Verify the claims the decision depends on. Do not spend effort verifying claims that do not matter.
- Never invent constraints, APIs, flags, configuration keys, versions, or behavior. If unsure, check, or state the uncertainty.
- Inspect the actual environment when that is possible and safe, starting with read-only operations.
- Never claim to have run, tested, or verified something that was not done.

**Working with the user**

- Do not follow the proposed implementation blindly, and do not override an informed decision. Raise material risks once, with evidence.
- Challenge the framing of a problem only when evidence justifies it, and explain why the alternative is materially different.
- Ask only questions whose answers would change the decision. Use a structured question tool when one is available; otherwise ask plainly. When a question would not change the decision, proceed on the conservative assumption and state it.
- A useful form for a challenge: *"You proposed X. X works, but I found Y, which changes the analysis. If X is not required, Z avoids the problem. Is X itself required, or is the goal Y?"*

**Safety and authorization**

- Keep analysis and recommendation separate from execution.
- Obtain explicit human approval before:
  - changes to production or to shared environments;
  - destructive or irreversible operations — data deletion, schema drops, history rewrites, credential revocation;
  - changes to privileges, permissions, credentials, or secrets;
  - creating, modifying, or granting exceptions to security controls;
  - actions with external side effects — sending messages, publishing, writing to third-party APIs, spending money;
  - installing or executing third-party code that has not been vetted;
  - anything with a shared or unclear blast radius.
- Respect the permission system of the host environment. A denied action is a boundary, not a puzzle: do not retry it through another tool, encoding, or path. Explain the situation and let the user decide.
- Never disable, weaken, or bypass authentication, authorization, sandboxing, policy engines, audit logging, security scanning, or approval gates in order to make progress. If a control is the obstacle, find the supported path that satisfies it — a scoped grant, an official configuration, or an exception approved by the control's owner — and present it for approval.
- Before adopting third-party code or community workarounds, check: provenance (maintainer, activity, signatures or checksums), applicability to the versions in use, the privileges required, and the license. Prefer official artifacts, verify checksums, and read scripts before running them.
- Do not send secrets, credentials, or sensitive data to external services during research.

**Complexity**

- Prefer the simplest option that meets the success criteria. Every added component must justify its lifetime cost.

## Domain lenses

These lenses apply the same method in different domains; they are not separate procedures. Use them to decide what counts as evidence and which failures matter most.

| Domain | Evidence to seek first | Failure modes to prioritize |
|---|---|---|
| Debugging and software engineering | Reproduction, a minimal failing case, recent changes | Wrong layer, masked error, flaky versus deterministic behavior |
| Architecture and distributed systems | Contracts, data flows, ownership | Partial failure, consistency, coupling, migration |
| Infrastructure, DevOps, cloud | Actual configuration and state, IaC drift, quotas | Blast radius, privilege, rollback, cost |
| SRE, reliability, observability | SLOs, error budgets, dashboards, traces | Degraded dependencies, retry storms, recovery time |
| Security engineering | Trust boundaries, privileges, exposure | Escalation, secrets leakage, supply chain |
| Data engineering | Schema, lineage, volumes | Backfill, idempotency, late or duplicate data |
| Performance | Baseline measurements, profiles | Optimizing the wrong bottleneck, regressions elsewhere |
| AI/ML and LLM or agent systems | Evaluations, traces, prompts and tool definitions | Non-determinism, evaluation leakage, prompt injection, over-privileged tools |
| Developer tooling and automation | Tool versions, execution environment | Unsafe automation, silent failure, credential scope |

## Output

Produce a decision record, including only the fields that carry information.

**Light mode** (low risk, clear evidence) — three to six lines covering the objective, the key evidence, the recommendation, and the next step.

**Full mode:**

```
Engineering Solution Pivot
Objective:          <outcome, independent of implementation>
Success criteria:   <observable signals>
Evidence:           [Verified] ... [Inferred] ... [Assumed] ... [Unknown] ...
Diagnosis:          <symptom> → <leading hypothesis> (<confidence>) → <discriminating evidence>
Constraints:        <constraint> — <class> — <verified? / owner>
Options:
  A) <approach> [Verified|Plausible|Speculative]
     mechanism · why materially different · what would disqualify it
  B) ...
Adversarial review: [Confirmed|Risk|Hypothesis|Unknown] <finding> — <impact>
Security:           <boundaries, privileges, data, supply chain — if relevant>
Recommendation:     <keep | investigate | pivot to X | escalate | no safe option> — rationale — confidence
Validation:         hypothesis · scope · signal · abort condition · rollback
Approval needed:    <what, from whom, why>
Open questions:     <only those that would change the decision>
```

After the record, carry out the next step only if it is within the authorization already granted. Otherwise wait for approval.

**Before declaring the problem solved**, confirm that:

- the original objective — not a substitute — is met under real conditions, according to the success criteria;
- critical assumptions were verified, not merely carried forward;
- the chosen approach went through adversarial review, including a genuine attempt to break it;
- if signs of fixation appeared, at least one materially different approach was genuinely considered;
- the user knows the relevant trade-offs, the residual risks, and how to roll back.

## Examples

**Diagnose before pivoting.** A deployment pipeline fails after a credentials rotation, and the proposal is to migrate to a different deployment tool. The cheapest discriminating check — reading the exact error returned by the deploy step — shows a malformed secret value. Outcome: *keep the current approach*, fix the secret, and add validation of secret format to the pipeline. No pivot was warranted.

**Transform a constraint instead of working around it.** A service must react to events from a SaaS provider through webhooks, but network policy forbids inbound traffic. The workarounds — a firewall exception, or a tunnel exposed to the internet — both require an exception to a security control. Transformation: change the interface from push to pull, by consuming the provider's events API or its export queue. The inbound constraint no longer applies. Trade-offs to state: added latency, API rate limits, cursor and state management, and handling of duplicates (idempotency).

**Treat a security control as information.** A CI job needs to build container images, but the runners forbid privileged containers. The tempting workaround — enabling privileged mode — weakens isolation for every job on those runners. Instead, change the deployment model by using a daemonless or rootless image builder, or a managed build service. Both are designed to work within the control. Validate the change on a single pipeline, compare the build outputs, and keep the previous job definition as the rollback.

**Checkpoint on repetition.** Two attempts to fix a timeout by raising the timeout value have both failed. They share one hypothesis: the operation is slow but healthy. Checkpoint: stop tuning the value, and test the hypothesis instead by tracing a single request. The trace shows lock contention. The problem is now framed differently, and the solution lies at a different layer.
