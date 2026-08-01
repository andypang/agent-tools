# Coordination Protocol for Agent-Driven Projects

## Purpose

Coordinate multiple limited-context agent sessions building one system, optimizing functionality/quality/tokens. The mechanism: one small authoritative doc of decisions, requirements, and contracts; disposable per-PR specs derived from it; budgets owned by the human; an adversarial cost agent to resist growth.

## Artifacts

### 1. Contracts Doc (authoritative, persistent)

**Charter (first line of the doc):** This doc contains only statements that, if two independent implementers disagreed on them, would produce broken integration or a failed requirement. Everything else is decided at PR time.

Three entry types, each with an ID (D#, R#, C#) and a mechanical inclusion test:

| Type | Definition | Inclusion test |
|---|---|---|
| **Decision** | A choice between viable alternatives that is expensive to reverse | Must name the rejected alternative and why. No named alternative → not a decision → cut |
| **Requirement** | Externally observable behavior or constraint | Must be falsifiable by an acceptance test written without knowledge of internals. References internals → leaked design → cut |
| **Contract** | A boundary two or more PRs touch: interface, schema, state ownership, invariant, error semantics | Must name ≥2 consumers. One consumer → internal detail → cut |

**Budgets** (human-owned; initial values deliberately low):
- Decisions: ≤ 10
- Requirements: ≤ 20
- Contracts: ≤ 8 named boundaries (a boundary may hold multiple entries; the cap is on boundaries, i.e. architecture fan-out)
- At budget, adding requires cutting or merging. Raising a budget requires human approval with a one-line case.

**Versioning:** The doc carries a version number, bumped on every merged diff. PR specs pin the version they derived from. A change to any entry flags in-flight PRs citing that entry's ID for re-derivation.

### 2. PR Specs (derived, disposable)

- Generated per PR from the Contracts Doc + local context.
- Must cite the IDs (D/R/C) they implement and pin the Contracts Doc version they derived from.
- Never reconciled back. If implementation reveals a contract must change, that is a Contracts Doc change request (see protocol below); the PR spec itself is discarded after merge.

## Roles

| Role | Owns |
|---|---|
| **Human** | Budgets and budget changes; exception adjudication; disputes between author and cost agent; final approval of Contracts Doc diffs |
| **Author agent** | Drafting entries; applying inclusion tests; proposing diffs with justification |
| **Cost agent** | Contesting every proposed addition, arguing from the same inclusion tests; proposing cuts and merges; running garbage collection |
| **PR agents** | Deriving PR specs; implementing; filing contract change requests when integration reality disagrees with the doc |

Author and cost agents do not negotiate to consensus. Unresolved disputes escalate to the human as: proposed entry, author's case, cost agent's case, each ≤ 2 sentences.

## Protocols

### Change protocol (Contracts Doc)
1. Every edit is a diff with a one-line justification tied to a discovered integration failure, requirement failure, or failed inclusion test.
2. Merging a diff bumps the doc version and flags in-flight PRs citing any changed ID for re-derivation (author agent performs both as part of the merge).
3. "This would be clearer if…" is not a valid justification. Clarity work belongs in PR specs.
4. Cost agent reviews every diff before it reaches the human.

### Review rules
Reviewers (agent or human) may raise only three objection types:
1. An entry fails its inclusion test.
2. Two entries conflict.
3. A missing entry will cause integration failure between named pieces X and Y.

Elaboration, interior design, and style suggestions are out of scope; route to the relevant PR spec. Review terminates when no objections of these three types remain.

**Review budget:** ≤ 3 rounds. A doc not converged in 3 rounds is over-scoped; the remedy is cutting entries, not more review.

### Garbage collection
- Scheduled, not opportunistic: cost agent runs GC every N merged PRs (start N = 5).
- An entry is GC-eligible only once a PR has merged in its area (otherwise unimplemented-but-valid entries get gutted early).
- Any eligible entry cited by zero PR specs is flagged for deletion; author agent may defend it (once, ≤ 2 sentences); human adjudicates ties.

## Invariants of the process itself

1. Exactly one authoritative doc. Requirements are a section of it, never a peer document.
2. Authority flows one way: Contracts Doc → PR specs. Nothing flows back except explicit contract change requests.
3. All budgets are human-set. Agents enforce, apply, and escalate; they never adjust.
4. Growth requires passing an inclusion test *and* surviving the cost agent. Deletion requires only a failed test or zero citations.
5. When in doubt, exclude. An error of omission surfaces as a cheap contract change request; an error of inclusion compounds silently as churn.
