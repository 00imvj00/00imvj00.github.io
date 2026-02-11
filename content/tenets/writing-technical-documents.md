---
title: "#2 — Technical Design Documents Done Right"
date: 2026-02-11
description: "How to write technical documents that survive contact with reality — from empty canvas to legacy minefield."
tags: ["technical-design", "documentation", "architecture"]
draft: false
weight: 2
---

## The Core Belief

A technical document is not a ritual — it's a thinking tool. If it doesn't force clarity, challenge assumptions, and surface risks before a single line of code is written, it's just bureaucracy with a Google Doc link.

The best tech doc isn't the longest one. It's the one where every section made someone uncomfortable enough to ask a better question.

Greenfield and brownfield projects demand fundamentally different documents. This protocol covers both — structured around the six layers of a [healthy service](/tenets/anatomy-of-a-healthy-service/).

---

## Why Most Tech Docs Fail

Before the templates, understand why documents die:

| Failure Mode | Symptom | Root Cause |
|-------------|---------|------------|
| "The Novel" | 40-page doc nobody reads | Author confused thoroughness with usefulness |
| "The Napkin" | Two paragraphs and a diagram | Author confused confidence with clarity |
| "The Postmortem Disguised as a Design" | Written after the code | Document is justification, not design |
| "The Copy-Paste" | Same template for every project | No distinction between greenfield and brownfield |
| "The Ghost" | Approved, never updated, silently wrong | No ownership after approval |

This protocol exists to kill all five.

---

## Part I: Greenfield — Designing in the Void

When there's no existing system, the document carries the full weight of alignment. You're not just describing *what* you'll build — you're justifying *why this shape* over every other.

### The Four Questions Every Greenfield Doc Must Answer

1. **Why does this system need to exist?** — Business context. If you can't explain it to a PM in two sentences, you don't understand it yet.
2. **What are the non-negotiable constraints?** — Latency budgets, compliance, team skill set, deployment targets. These aren't afterthoughts; they *are* the design.
3. **What did you reject and why?** — The alternatives section is the most important part. It proves you explored the space, not just picked the first thing that compiled in your head.
4. **Where will this hurt in 18 months?** — Every design has a shelf life. Name the assumptions that will break first.

### Greenfield Template — Layer by Layer

Structured around the [Anatomy of a Healthy Service](/tenets/anatomy-of-a-healthy-service/) so nothing falls through the cracks.

```
1. CONTEXT & MOTIVATION
   - Business problem (2–3 sentences, not a novel)
   - Who are the users / callers?
   - What happens if we don't build this?

2. GOALS & NON-GOALS
   - Explicit goals (measurable where possible)
   - Explicit non-goals (equally important — what are we NOT solving?)

3. CONSTRAINTS & ASSUMPTIONS
   - Latency / throughput requirements
   - Compliance / regulatory constraints
   - Team capabilities & timeline
   - Assumptions that, if wrong, invalidate the design

4. PROPOSED DESIGN

   4a. Source & Delivery
       - Repository structure
       - CI/CD pipeline design
       - Deployment strategy (rolling / blue-green / canary)
       - Feature flag strategy

   4b. Application Core
       - API contracts (endpoints, request/response shapes)
       - Domain model & key entities
       - Business logic boundaries (what goes where)
       - Error handling philosophy

   4c. Infrastructure & Data
       - Data store choice & schema design
       - Cache strategy (what, where, TTL, invalidation)
       - Queue/workflow design (async flows, DLQs, retries)
       - File storage needs
       - Data retention & PII handling

   4d. Reliability & Resilience
       - Dependency map (hard vs. soft dependencies)
       - Timeout & circuit breaker strategy
       - Rate limiting & backpressure approach
       - Idempotency requirements

   4e. Observability
       - Key metrics to instrument (RED, USE, business)
       - Logging strategy (what to log, correlation IDs)
       - Tracing: critical paths to instrument
       - Initial dashboard design
       - Alerts: what conditions, what severity

   4f. Operational Readiness
       - Health check design (liveness / readiness / deep)
       - Configuration & secrets management
       - SOPs: what failure scenarios to document upfront
       - Security considerations (auth, scanning, network)

5. KEY FLOWS
   - Happy path (sequence diagram)
   - Failure modes (what breaks, what happens)
   - Edge cases worth calling out

6. ALTERNATIVES CONSIDERED
   - For each alternative:
     - What was it?
     - Why was it rejected?
     - Under what conditions would we revisit?

7. TRADE-OFFS & KNOWN LIMITATIONS
   - What are we consciously accepting?
   - What's the shelf life of this design?

8. ROLLOUT PLAN
   - Phased rollout milestones
   - Definition of "done" for each phase
   - Rollback trigger criteria

9. OPEN QUESTIONS
   - Unresolved decisions (with owners and deadlines)
   - Dependencies on other teams
```

### Greenfield Anti-Patterns

- Jumping straight into the solution without establishing constraints.
- Only sunny-day sequence diagrams — no failure mode analysis.
- Writing the doc *after* the code. At that point it's documentation, not design.
- Choosing infrastructure before understanding the domain. Pick the data model before the database.
- Skipping the "Operational Readiness" sections because "we'll add monitoring later." (You won't.)

---

## Part II: Brownfield — Designing with Gravity

Existing systems have inertia. The document's job shifts from *inventing* to *negotiating* — with legacy schemas, existing traffic patterns, team knowledge, and the deployment pipeline you actually have.

### The Four Questions Every Brownfield Doc Must Answer

1. **What exists today?** — A brutally honest picture of the current state. Include the warts. If you're embarrassed by it, good — that's why you're changing it.
2. **What's the smallest change that delivers the outcome?** — Resist the rewrite urge. The best brownfield designs are surgical.
3. **What's the migration path?** — The design *is* the migration. A beautiful target state with a hand-wavy "we'll migrate later" is not a design, it's a wish.
4. **What breaks during the transition?** — Dual-write gotchas, backward compatibility, feature flags, data backfills. This section separates senior engineers from everyone else.

### Brownfield Template — Layer by Layer

```
1. CONTEXT & MOTIVATION
   - Business problem driving the change
   - Why now? What's the cost of not changing?

2. CURRENT STATE (the honest mirror)

   2a. Architecture Today
       - System diagram (as it actually is, not as it was designed)
       - Key components and their responsibilities
       - Known tech debt & pain points

   2b. Data Landscape
       - Current schema / data model
       - Data volumes, growth rate
       - Existing retention policies (or lack thereof)

   2c. Observability Baseline
       - What monitoring exists today?
       - Current alert coverage and gaps
       - Known blind spots

   2d. Operational Reality
       - Deployment frequency and confidence level
       - Recent incidents related to this area
       - Current on-call burden

3. GOALS & NON-GOALS

4. PROPOSED CHANGES — Layer by Layer

   4a. Source & Delivery Changes
       - Pipeline modifications
       - New deployment requirements
       - Feature flag plan for the transition

   4b. Application Core Changes
       - API changes (new endpoints, deprecated endpoints, contract changes)
       - Business logic modifications
       - What stays the same (equally important as what changes)

   4c. Infrastructure & Data Changes
       - Schema migrations (exact DDL, phased approach)
       - Cache changes (new keys, invalidation updates)
       - Queue modifications (new consumers, message format changes)
       - Data backfill requirements

   4d. Reliability Impact
       - How does this change the failure domain?
       - New dependencies introduced
       - Timeouts / circuit breakers to add or adjust

   4e. Observability Changes
       - New metrics / dashboards needed
       - Alert modifications
       - New SOPs for new failure modes

5. MIGRATION STRATEGY
   - Phase 1: ... (scope, duration, success criteria)
   - Phase 2: ... (scope, duration, success criteria)
   - Phase N: ...
   - Dual-write / dual-read windows
   - Data consistency verification approach
   - How to handle in-flight requests during switchover

6. BACKWARD COMPATIBILITY & ROLLBACK
   - What remains backward compatible?
   - Rollback plan for each phase
   - Point of no return (if any) — and what to do if you need to go back after it

7. RISK MATRIX
   | Risk | Likelihood | Impact | Mitigation |
   |------|-----------|--------|------------|
   | ...  | ...       | ...    | ...        |

8. ALTERNATIVES CONSIDERED

9. OPEN QUESTIONS
```

### Brownfield Anti-Patterns

- Treating it like a greenfield doc — ignoring the existing system entirely.
- No rollback plan. If you can't undo it, you can't ship it safely.
- Underestimating data migration complexity by 10x (everyone does, every time).
- "Big bang" migration instead of phased approach. The blast radius of a single cutover is always larger than you think.
- Not involving the on-call team in the review. They're the ones who'll deal with the consequences at 3 AM.

---

## Part III: Universal Laws

Regardless of greenfield or brownfield, these apply:

### On Writing

- **Write for the skeptic, not the supporter.** Your best reviewer is the person who thinks this is a bad idea.
- **Diagrams > paragraphs.** A sequence diagram of the critical path is worth a thousand words of prose.
- **Time-box the doc, not the thinking.** Spend 80% thinking, 20% writing. If the doc took longer to write than to think through, the ratio is wrong.
- **One doc, one decision.** Don't bundle three unrelated changes into one document. Each decision deserves its own space.

### On Assumptions

- **Name them explicitly.** "We assume < 1000 QPS" is infinitely more useful than silence.
- **Assign a shelf life.** "This assumption is valid until we onboard Tenant X in Q3."
- **Revisit post-launch.** Which assumptions held? Which broke? Document the delta — it's how you calibrate future estimates.

### On Reviews

- **Three types of reviewers you need:**
  1. The domain expert — "Is this the right thing?"
  2. The systems thinker — "Will this work at scale?"
  3. The on-call engineer — "Can I debug this at 3 AM?"
- **Review the alternatives section hardest.** If it's thin, the thinking was thin.
- **Mandate questions, not just approvals.** A doc approved with zero comments probably wasn't read.

### On Lifecycle

- **A tech doc that's never updated after approval is a lie on a wiki.**
- Revisit post-launch and annotate:
  - What you got right.
  - What you got wrong and why.
  - What you'd do differently.
- This turns design docs into institutional memory, not just approval artifacts.

---

## The Cheat Sheet

| | Greenfield | Brownfield |
|---|-----------|------------|
| **Primary challenge** | Infinite design space | Constrained by reality |
| **Biggest risk** | Building the wrong thing | Breaking the existing thing |
| **Most important section** | Alternatives Considered | Migration Strategy |
| **Hardest section to write** | Failure Modes | Backward Compatibility |
| **Most commonly skipped** | Operational Readiness | Current State (honest version) |
| **Key reviewer** | Systems thinker | On-call engineer |
| **Success metric** | Design survives first contact with production | Migration completes without rollback |

---

*This tenet evolves. Last refined: February 2026.*
