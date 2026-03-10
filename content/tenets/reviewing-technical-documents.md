---
title: "#3 — The Art of Technical Document Review"
date: 2026-02-11
description: "A reviewer's guide to technical documents — because approving without questioning is just co-signing someone else's assumptions."
tags: ["technical-design", "documentation", "code-review"]
draft: true
weight: 3
---
## The Core Belief

Reviewing a technical document is not reading it. Reading is passive. Reviewing is adversarial collaboration — you are trying to break the design on paper so it doesn't break in production.

draft: true


---

## The Reviewer's Mindset

Before you open the document, calibrate your thinking:

| Mindset | What it means | What it sounds like |
|---------|---------------|---------------------|
| **The Skeptic** | Assume every claim is unproven until justified | "What evidence supports this?" |
| **The Operator** | Imagine running this at 3 AM during an incident | "How do I debug this when it fails?" |
| **The Adversary** | Think like the traffic, the attacker, the edge case | "What happens when this gets 100x the expected load?" |
| **The Historian** | Remember what went wrong last time | "We tried this in 2024. What's different now?" |
| **The New Hire** | Read as if you have zero context about this system | "Would I understand this on my first week?" |

Switch between these lenses as you move through the document. No single perspective catches everything.

---

## The Three Passes

Don't review a tech doc in one pass. You'll miss the forest or the trees. Do three.

### Pass 1: The Altitude Check (5 minutes)

Read the title, description, goals, non-goals, and conclusion. Don't read the middle yet.

**Ask yourself:**
- Do I understand what problem this solves?
- Do the goals make sense given the business context?
- Are the non-goals explicitly stated? (If not, the scope is unbounded.)
- Does the conclusion follow logically from the stated goals?
- Is the scope right-sized? (Too ambitious = won't ship. Too narrow = won't matter.)

**Red flags at this stage:**
- No non-goals section → the author hasn't thought about scope.
- Goals are vague ("improve performance") → unmeasurable = unverifiable.
- The "why" is missing → solution looking for a problem.

If the altitude check fails, stop here. Comment on the framing before investing time in the details.

### Pass 2: The Layer Walk (20–30 minutes)

Now read the full design, structured around the [six layers of a healthy service](/tenets/anatomy-of-a-healthy-service/). For each layer, apply the relevant questions below.

### Pass 3: The Stress Test (10 minutes)

After understanding the design, try to break it. See "The Stress Test Protocol" section below.

---

## Pass 2: Layer-by-Layer Review Guide

### Layer 1 — Source & Delivery

**What to look for:**
- Is the repository structure defined or does it assume "we'll figure it out"?
- Is the CI pipeline described? What gates exist before merge?
- What's the deployment strategy? Is rollback addressed?
- Are feature flags mentioned for decoupling deploy from release?

**Questions to ask:**
- "How long will the build take? What's the feedback loop?"
- "What's the rollback plan if the first deploy fails? How long does rollback take?"
- "Is the deployment order documented if multiple services change?"

**Instant no-go:**
- No rollback plan → the author is assuming nothing goes wrong.
- Manual deployment steps → it will break when the person who knows the steps is on vacation.

---

### Layer 2 — Application Core

#### APIs

**What to look for:**
- Are API contracts specified (request/response shapes, status codes)?
- Is versioning addressed?
- Is error response format consistent with existing services?
- Are idempotency requirements identified for mutating endpoints?

**Questions to ask:**
- "What does the caller see when this fails? Show me the error response."
- "Is this endpoint idempotent? What happens if the client retries?"
- "What's the pagination strategy for list endpoints?"
- "Have you validated this contract with the consuming team?"

**Instant no-go:**
- No error response examples → the author hasn't thought about failure from the caller's perspective.
- Unbounded list endpoints → production time bomb.

#### Business Logic

**What to look for:**
- Is the domain model clear? Can you draw the entity relationships from the doc?
- Is business logic separated from transport and persistence?
- Are state transitions documented (e.g., order lifecycle, claim states)?

**Questions to ask:**
- "Where does this business rule live in the code? Controller? Service? Repository?"
- "What's the state machine for [entity]? Are invalid transitions prevented?"
- "Can I test this logic without spinning up a database?"

**Instant no-go:**
- Business logic embedded in SQL queries or HTTP handlers → untestable, unmaintainable.

#### Testing Strategy

**What to look for:**
- Are test types specified (unit, integration, contract, E2E)?
- Is the testing approach proportional to the risk?
- Are edge cases and failure paths called out as test scenarios?

**Questions to ask:**
- "What's the highest-risk code path? How is it tested?"
- "Are you testing the failure modes or just the happy path?"
- "How are integration tests run? Testcontainers? Embedded DB? Manual?"

**Instant no-go:**
- "We'll add tests later" → no, you won't.
- Only happy-path test scenarios listed → the interesting bugs live in the edge cases.

---

### Layer 3 — Infrastructure & Data

#### Data Store

**What to look for:**
- Schema design with column types, constraints, and indexes.
- Migration strategy (Flyway/Liquibase or raw DDL?).
- Query patterns — are the indexes aligned with how data is actually read?
- Backup and recovery mentioned.

**Questions to ask:**
- "What's the expected data volume in 6 months? 12 months?"
- "Show me the heaviest query. Is there an index for it?"
- "Are these migrations backward-compatible? Can the old code run against the new schema during rolling deployment?"
- "What's the read/write ratio? Do we need read replicas?"
- "When was the last time a backup restore was tested?"

**Instant no-go:**
- No schema definition → design isn't concrete enough to review.
- Schema migration breaks old code → zero-downtime deployment is impossible.

#### Cache

**What to look for:**
- What's being cached and why?
- TTL values with justification.
- Invalidation strategy.
- Behavior when cache is unavailable.

**Questions to ask:**
- "What happens when the cache is cold? First deploy, cache restart, eviction storm?"
- "How stale can this data be before it causes a user-visible problem?"
- "What's the invalidation trigger? Time-based? Event-based? Both?"
- "If Redis goes down, does the service degrade gracefully or fall over?"

**Instant no-go:**
- Cache with no TTL → data will be stale and nobody will know.
- No degradation plan → cache is a single point of failure, not an optimization.

#### Queues & Async Processing

**What to look for:**
- Message schema/format.
- Consumer idempotency approach.
- DLQ configuration and alerting.
- Retry policy with backoff.
- Ordering guarantees (or explicit acknowledgment that ordering doesn't matter).

**Questions to ask:**
- "What happens if this message is processed twice?"
- "What happens if messages arrive out of order?"
- "What's in the DLQ alert? Who investigates it? What's the SOP?"
- "What's the retry budget? After how many retries do you give up?"

**Instant no-go:**
- No DLQ → failed messages disappear silently.
- No idempotency strategy → duplicate processing will cause data corruption.

#### Data Retention

**Questions to ask:**
- "How long is this data kept? What's the legal/compliance basis?"
- "Is there PII in this data? How is right-to-erasure handled?"
- "What's the cleanup mechanism? Automated job or manual?"

---

### Layer 4 — Reliability & Resilience

**What to look for:**
- Dependency map: what does this service call? What calls it?
- Classification of dependencies (hard vs. soft).
- Timeout values for every outbound call.
- Circuit breaker configuration.
- Rate limiting strategy (inbound and outbound).

**Questions to ask:**
- "What happens when [dependency X] is down for 30 minutes?"
- "What are the timeout values? How were they chosen? Are they tested?"
- "What does the circuit breaker look like in the open state? What do users see?"
- "If this endpoint gets 10x normal traffic, what breaks first?"
- "Are retries safe here? What's the amplification factor?"

**The Retry Amplification Test:**
If Service A retries 3 times, and downstream Service B retries 3 times on its own dependency, a single failure generates $3 \times 3 = 9$ calls. Add another layer and it's 27. Ask: "What's the total retry amplification across the call chain?"

**Instant no-go:**
- No timeouts specified → a slow dependency will hang your entire service.
- No dependency classification → the author doesn't know what breaks when something is unavailable.

---

### Layer 5 — Observability

**What to look for:**
- Key metrics to instrument (RED, USE, business).
- Logging strategy: structured? Correlation IDs?
- Dashboards: new or additions to existing?
- Alert definitions with severity.

**Questions to ask:**
- "If this fails silently, how long until someone notices? What metric catches it?"
- "Can I trace a single user request across all the services involved?"
- "What alert fires when [most likely failure mode] happens? What's the SOP?"
- "Are business metrics instrumented? How does the product team know this feature is working?"
- "What does the dashboard look like? Can you sketch the panels?"

**Instant no-go:**
- No observability section → the author plans to debug by reading logs on individual containers.
- Alerts without SOPs → someone will get paged and have no idea what to do.

---

### Layer 6 — Operational Readiness

**What to look for:**
- Health check design (liveness / readiness).
- Configuration & secrets management.
- SOPs for anticipated failure modes.
- Security considerations.
- Runbook or on-call documentation.

**Questions to ask:**
- "If I'm on call and this alerts, what do I do? Where's the runbook?"
- "Where do secrets live? Vault? Environment variables? (Please don't say committed to the repo.)"
- "What's the configuration change process? Is it audited?"
- "Has the security review happened? Any new attack surfaces?"

**Instant no-go:**
- No health checks → the load balancer can't distinguish a healthy instance from a dead one.
- Secrets in config files → security incident waiting to happen.

---

## Pass 3: The Stress Test Protocol

After understanding the design, run these scenarios mentally:

### The Failure Cascade
Pick the most critical dependency. Imagine it goes down for 15 minutes during peak traffic.
- What happens to in-flight requests?
- Does the service degrade or crash?
- What do users see?
- How does the on-call know?
- How long does recovery take after the dependency is back?

### The Traffic Spike
Imagine 10x normal traffic for 5 minutes.
- What breaks first? (Connection pool? Database? Rate limiter?)
- Is there backpressure or does the service accept everything and die?
- Does the queue depth grow unboundedly?
- Do retries amplify the problem?

### The Data Corruption Scenario
Imagine a bug causes incorrect data to be written for 2 hours before detection.
- How is the bad data identified?
- Can it be corrected? Backfill? Manual fix?
- Is there an audit trail to determine the blast radius?
- What do downstream consumers of this data experience?

### The Midnight Deploy
Imagine the first deployment happens at 11 PM on a Friday (because it will, eventually).
- Can this be deployed without the author present?
- Is the rollback plan executable by someone unfamiliar with the service?
- Are the health checks sufficient to auto-detect a bad deploy?
- What's the blast radius if it goes wrong?

### The New Hire Test
Hand the document to someone who joined last month.
- Can they understand the system from this doc alone?
- Can they set up a local development environment?
- Can they find the right dashboard during an incident?

If any scenario reveals a gap, that's a review comment.

---

## How to Write a Good Review Comment

Bad review comments are vague, unhelpful, or just assertions of opinion. Good ones are specific, questioning, and actionable.

### The Comment Formula

```
[OBSERVATION] → [QUESTION or CONCERN] → [SUGGESTION (optional)]
```

**Bad comments:**
- "This won't scale." (Assertion without evidence.)
- "I don't like this approach." (Opinion without alternative.)
- "Looks good." (Not a review.)
- "+1" (Definitely not a review.)

**Good comments:**

> "The design uses a single database for both read and write traffic (observation). At the projected 5000 QPS read volume in 6 months, this could saturate the connection pool (concern). Have you considered a read replica or caching layer for the high-frequency read paths? (suggestion)"

> "I don't see a retry budget or circuit breaker for the payment gateway call (observation). If the gateway is slow for 5 minutes, the thread pool will be exhausted (concern). What's the timeout, and should we add a circuit breaker here? (question)"

> "The migration adds a NOT NULL column without a default (observation). During rolling deployment, old instances will fail on INSERT because they don't know about the new column (concern). Can this be split into: 1) add column as nullable, 2) backfill, 3) add NOT NULL constraint? (suggestion)"

### Comment Severity Tags

Prefix your comments for clarity:

| Tag | Meaning | Author must... |
|-----|---------|----------------|
| `[blocking]` | This must be resolved before approval | Address it |
| `[major]` | Significant concern that needs discussion | Respond with reasoning |
| `[minor]` | Suggestion for improvement, not critical | Consider it |
| `[question]` | Genuine clarification request | Answer it |
| `[nit]` | Cosmetic / style preference | Ignore if they want |

---

## The Reviewer's Checklist

Before you click "Approve," verify:

### Framing
- [ ] The problem is clearly stated and justified
- [ ] Goals are specific and measurable
- [ ] Non-goals are explicitly listed
- [ ] Constraints and assumptions are named

### Design Completeness (per [Anatomy of a Healthy Service](/tenets/anatomy-of-a-healthy-service/))
- [ ] Source & Delivery: CI/CD, rollback, feature flags
- [ ] Application Core: APIs, domain logic, tests, quality gates
- [ ] Infrastructure & Data: Storage, cache, queues, retention
- [ ] Reliability: Dependencies classified, timeouts, circuit breakers
- [ ] Observability: Metrics, logs, traces, dashboards, alerts
- [ ] Operational Readiness: Health checks, SOPs, config, security

### Critical Sections
- [ ] Alternatives section has at least 2 alternatives with genuine reasoning
- [ ] Failure modes are analyzed, not just happy paths
- [ ] Migration/rollback plan exists (brownfield)
- [ ] Data schema and migration are backward-compatible
- [ ] Open questions have owners and deadlines

### Stress Test
- [ ] Dependency failure scenario addressed
- [ ] Traffic spike scenario addressed
- [ ] Data corruption scenario addressed
- [ ] The new hire can understand this document

---

## When to Block vs. When to Let Go

Not every concern is a blocker. Calibrate:

**Block when:**
- There's no rollback plan.
- A failure mode is unhandled and could cause data loss.
- Security is not addressed (auth, secrets, injection).
- The alternatives section is empty or trivially dismissed.
- The migration can't be done without downtime and that hasn't been acknowledged.

**Comment but don't block when:**
- You'd make a different architectural choice but theirs is defensible.
- You want more detail in the observability section but the basics are covered.
- The naming convention is different from what you'd pick.
- The doc could benefit from one more diagram.

**Let it go when:**
- It's a stylistic preference with no impact on correctness.
- The author has considered your concern and made a conscious trade-off.
- You're pattern-matching to a past project that isn't actually analogous.

The best reviewers know when their concern is a genuine risk and when it's a preference wearing the mask of a principle.

---

## The Meta-Review: Reviewing Yourself

After submitting your review, ask:

1. **Did I ask questions or just make statements?** Questions create dialogue. Statements create defensiveness.
2. **Did I focus on the design or the designer?** "This approach has a gap" vs. "You missed this" — word choice matters.
3. **Did I suggest alternatives where I raised concerns?** Pointing out problems is easy. Proposing solutions is useful.
4. **Did I acknowledge what's good?** If the design is strong, say so. Reviewers who only find problems eventually get tuned out.
5. **Would I want to receive this review?** The golden rule of code review applies to doc review too.

---

*This tenet evolves. Last refined: February 2026.*
