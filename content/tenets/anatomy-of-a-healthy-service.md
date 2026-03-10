---
title: "#1 — Service Anatomy"
date: 2026-02-11
description: "Every production service should have these elements. No exceptions. No shortcuts."
tags: ["architecture", "operations", "services"]
draft: false
weight: 1
---
## The Core Belief

A service isn't "done" when the feature works. A service is done when it can be built, deployed, observed, debugged, and handed to another team — all without the original author in the room.

draft: true


## The Big Picture

A perfectly working system has six layers. Each layer depends on the one below it. Skip a layer and the ones above it become unreliable.

```
┌─────────────────────────────────────────────────┐
│            6. Operational Readiness             │  ← Can someone else run this at 3 AM?
├─────────────────────────────────────────────────┤
│            5. Observability                     │  ← Can you see what's happening?
├─────────────────────────────────────────────────┤
│            4. Reliability & Resilience          │  ← What happens when things fail?
├─────────────────────────────────────────────────┤
│            3. Infrastructure & Data             │  ← Where does state live?
├─────────────────────────────────────────────────┤
│            2. Application Core                  │  ← What does the service do?
├─────────────────────────────────────────────────┤
│            1. Source & Delivery                 │  ← How does code become a running service?
└─────────────────────────────────────────────────┘
```

---

## 1. Source & Delivery

> *How does code become a running service?*

The pipeline from commit to production. If any piece is missing, you don't have a service — you have a liability.

### 1.1 Hosted Repository

Your source of truth. Not someone's laptop.

- **Platform**: GitHub, GitLab, Bitbucket — hosted, backed up, access-controlled.
- **Branching strategy**: Documented in the README. Trunk-based development preferred.
- **Branch protection on main**:
  - Required code reviews (minimum 1 approval).
  - All CI checks passing before merge.
  - No force-push.
- **Repository hygiene**:
  - `.gitignore` that actually works — no committed secrets, IDE configs, or build artifacts.
  - **CODEOWNERS** file — every directory has a named owner.
  - Meaningful commit messages (conventional commits or equivalent).
  - PR templates that enforce context: *what*, *why*, *how to test*.

### 1.2 Hosted Build Pipeline (CI)

Every push triggers the pipeline. No manual builds. No "works on my machine."

- **Pipeline stages** (in order):
  1. **Dependency resolution** — pinned versions, lockfiles committed.
  2. **Compilation** — fails fast on syntax errors.
  3. **Unit tests** — fast, isolated, no external dependencies.
  4. **Integration tests** — real databases via testcontainers, not mocks pretending to be databases.
  5. **Static analysis / linting** — code style, complexity, common bugs.
  6. **Security scanning** — dependency vulnerabilities (Dependabot, Snyk, Trivy).
  7. **Code quality gate** — SonarQube or equivalent. New code must pass before merge.
  8. **Artifact generation** — Docker image, JAR, binary. Your deploy unit.
- **Reproducibility**: same commit → same artifact, every time. No build-time side effects.
- **Speed target**: under 10 minutes. Longer than that and you've killed the feedback loop.
- **Build notifications**: failures go to the author and the team channel. No silent red builds.

### 1.3 Hosted Deployment Pipeline (CD)

Shipping code to production should be boring.

- **Environment ladder**: dev → staging → production. Never skip staging.
- **Deployment strategy**: Rolling update, blue-green, or canary. Pick one and document *why*.
  - Rolling: simplest, good for stateless services.
  - Blue-green: instant rollback, good for critical paths.
  - Canary: incremental confidence, good for high-traffic services.
- **Rollback mechanism**: Tested, documented, executable in under 5 minutes.
- **Feature flags**: Decouple deploy from release. Ship dark, enable when ready.
- **Deployment gates**:
  - Smoke tests post-deploy.
  - Automated rollback on health check failure.
  - Approval gates for production (optional, depends on risk tolerance).
- **Audit trail**: Who deployed what, when, and which commit. Non-negotiable.

---

## 2. Application Core

> *What does the service actually do?*

The code itself and the guardrails that keep it honest.

### 2.1 APIs — The Service Boundary

The contract between your service and the outside world.

- **Contract-first design**:
  - OpenAPI/Swagger spec for REST, protobuf for gRPC — committed to the repo.
  - Generated *before* code, not reverse-engineered after.
  - Breaking changes go through a deprecation cycle, never a surprise.
- **Versioning**: URL path (`/v1/`), header, or content negotiation. Pick one, be consistent.
- **Input validation at the boundary**: Never trust the caller. Validate types, ranges, required fields, and reject early.
- **Consistent error responses**:
  ```json
  {
    "error": "VALIDATION_ERROR",
    "message": "email is required",
    "requestId": "abc-123"
  }
  ```
  Same shape, every endpoint, every error.
- **Rate limiting & throttling**: Even internal services need this. Your upstream neighbor's retry storm shouldn't take you down.
- **API documentation**: Auto-generated from the contract. Always current. If a developer has to ask Slack how an endpoint works, the docs have failed.
- **Pagination**: Every list endpoint. No exceptions. Unbounded queries are ticking time bombs.
- **Idempotency keys**: For any mutating operation that could be retried (payments, order creation, etc.).

### 2.2 Business Logic — The Domain Layer

The reason the service exists.

- **Separation of concerns**:
  - Transport layer (HTTP handlers / gRPC servers) — serialization, auth, routing.
  - Domain layer — business rules, validations, state transitions.
  - Persistence layer (repositories) — data access, query construction.
  - **Business logic lives in the domain layer.** Not in controllers. Not in SQL queries. Not in queue consumers.
- **Testability**: Domain logic must be testable without a database, a queue, or an HTTP server. If you need Docker to run a unit test, it's not a unit test.
- **Clarity test**: If you can't explain where the business logic lives in 30 seconds, refactor until you can.
- **Error handling philosophy**:
  - Domain errors (validation failures, business rule violations) are **expected** — model them as types, not exceptions.
  - Infrastructure errors (DB timeout, network failure) are **unexpected** — let them bubble up for retry/circuit-breaking.
  - Never swallow errors silently. Log, metric, or propagate — pick at least one.

### 2.3 Business Logic Tests — Proving Correctness

Tests aren't a chore. They're the specification your code must satisfy.

- **Test pyramid** (bottom to top):
  1. **Unit tests** — fast, isolated, no I/O. Cover edge cases, boundary conditions, and failure paths — not just happy paths.
  2. **Integration tests** — validate real interactions with databases, queues, caches. Use testcontainers or embedded instances, not mocks pretending to be infrastructure.
  3. **Contract tests** — if you consume or provide APIs, verify the contract hasn't silently broken (Pact, Spring Cloud Contract).
  4. **End-to-end tests** — sparingly. Cover critical user journeys only. Flaky E2E tests erode trust faster than no tests.
- **Code coverage**: A signal, not a target. 80% coverage with thoughtless assertions is worse than 50% with meaningful ones.
- **Test hygiene**:
  - Tests run in CI. Tests that only pass on someone's machine don't exist.
  - Each test is independent — no shared mutable state, no ordering dependencies.
  - Test names describe behavior: `should_reject_order_when_inventory_is_zero`, not `test1`.
  - Flaky tests get fixed or deleted within a sprint. Never ignored.

### 2.4 Code Quality Measurement — Keeping the Bar High

Static analysis catches what code review misses.

- **SonarQube** (or equivalent) integrated into CI.
- **Quality gate enforced**: New code must pass before merge. No overrides.
- **What to track and trend**:
  | Metric | Why it matters |
  |--------|---------------|
  | Code smells | Accumulated friction that slows future work |
  | Cognitive complexity | Functions too complex for a human to reason about |
  | Duplication | Copy-paste debt compounding silently |
  | Security hotspots | Potential vulnerabilities flagged early |
  | Test coverage delta | Are new changes tested? |
- **The rule**: Don't just install it — **act on it**. A dashboard full of ignored warnings is decoration, not engineering.

---

## 3. Infrastructure & Data

> *Where does state live?*

The stateful and asynchronous backbone. This layer is where most production incidents originate.

### 3.1 Queues & Workflows — Async Processing

Not everything needs a synchronous response.

- **When to use async**:
  - Work that takes > 500ms.
  - Work that can be retried independently.
  - Fan-out to multiple downstream consumers.
  - Work whose failure shouldn't block the caller.
- **Dead-letter queues (DLQ)**: Configured for *every* queue. Failed messages go somewhere visible, not into the void.
- **Idempotent consumers**: Messages *will* be delivered more than once. Design for it.
  - Use a deduplication key (message ID, idempotency key).
  - Make processing naturally idempotent (upserts over inserts).
- **Retry policies**:
  - Exponential backoff with jitter. Don't hammer a failing downstream.
  - Max retry limit before sending to DLQ.
  - Configurable per queue, not hardcoded.
- **Workflow orchestration** (for multi-step processes):
  - State machine or saga pattern for long-running business processes.
  - Compensation logic for rollback when a step fails midway.
  - Visibility into workflow state: what step is it on, when did it last progress, is it stuck?
- **Monitoring**: Queue depth, consumer lag, DLQ size, processing latency — all dashboarded and alerted.

### 3.2 Cache — Speed Where It Matters

A cache is an optimization, not an architecture.

- **Strategy** (pick one, document it):
  - **Cache-aside**: Application reads cache first, falls back to source, populates on miss.
  - **Write-through**: Writes go to cache and source simultaneously.
  - **Write-behind**: Writes go to cache immediately, flushed to source asynchronously.
- **TTL**: Defined for *every* cached entry. "Cache forever" means "stale forever."
- **Invalidation**: 
  - Event-driven invalidation preferred over time-based expiry for critical data.
  - Document the invalidation strategy. It is one of the two hard problems, after all.
- **Monitoring**:
  - Hit/miss ratio. A cache with a 20% hit rate is just a slow database with extra steps.
  - Eviction rate. If you're evicting constantly, the cache is undersized.
  - Latency. If cache reads are slow, you've added latency, not removed it.
- **Graceful degradation**: The service must function (possibly slower) when the cache is down. If cache failure = service failure, you've built a single point of failure, not a cache.
- **Serialization**: Use a format that supports schema evolution (Protobuf, Avro) for complex cached objects. Java serialization in Redis is a classic trap.

### 3.3 RDBMS / Data Store — The Source of Truth

The data outlives the code. Treat it accordingly.

- **Schema management**:
  - Migrations via Flyway, Liquibase, Alembic — never manual DDL in production.
  - Forward-only, backward-compatible migrations. No breaking changes without a multi-phase rollout.
  - Migration scripts committed to the repo and run as part of CD.
- **Connection management**:
  - Connection pooling configured and tuned (HikariCP, PgBouncer) — not left at defaults.
  - Connection limits aligned with instance capacity. One service shouldn't exhaust the pool.
  - Connection leak detection enabled.
- **Read/write separation**:
  - Read replicas for read-heavy workloads. Don't punish your write path.
  - Understand replication lag and design reads accordingly.
- **Index hygiene**:
  - Indexes reviewed quarterly.
  - Missing indexes kill you slowly (full table scans during peak traffic).
  - Unused indexes kill your writes (every insert/update maintains them for nothing).
  - Composite index column order matters — verify with `EXPLAIN ANALYZE`.
- **Query discipline**:
  - No `SELECT *` in application code.
  - Slow query logging enabled. Alert on queries exceeding threshold.
  - N+1 queries detected and eliminated.
- **Backup & recovery**:
  - Automated backups with defined RPO (Recovery Point Objective).
  - **Restore tested regularly** — not just configured. If you haven't restored from a backup, you don't have backups.
  - Point-in-time recovery capability for critical data stores.

### 3.4 File Storage — Binary & Unstructured Data

Files don't belong in databases or on local disk.

- **Object storage**: S3, GCS, Azure Blob — managed, durable, scalable.
- **Organization**: Consistent key/path naming conventions. Include tenant, date, or content type in the path.
- **Lifecycle policies**: Automatic transitions (hot → warm → cold → delete) based on age and access patterns.
- **Access control**:
  - Principle of least privilege. Buckets are private by default.
  - Pre-signed URLs for temporary access to specific objects.
  - No public buckets unless explicitly justified and reviewed.
- **Upload safety**:
  - Content-type validation at upload time.
  - File size limits enforced.
  - Virus/malware scanning for user-uploaded content.
- **CDN integration**: For frequently accessed public assets. Don't serve static files from your application servers.

### 3.5 Data Retention Policies — Nothing Lives Forever

"Keep everything forever" is not a policy — it's a compliance incident waiting to happen.

- **Retention period**: Defined for every data store, every table, every bucket. Written down, not assumed.
- **Automated cleanup**: Scheduled jobs that purge expired data. Manual cleanup doesn't scale and doesn't happen.
- **PII handling**:
  - Inventory: What PII is collected, where it's stored, who can access it.
  - Minimization: Don't collect what you don't need.
  - Purge capability: Can you delete a specific user's data on request? (Right to be forgotten.)
  - Anonymization: For analytics data that needs to outlive the retention window.
- **Regulatory alignment**: GDPR, DPDP, SOC2, PCI-DSS — whatever applies to your domain.
- **Archival strategy**: Data that's no longer hot but can't be deleted moves to cold storage with a defined access pattern and eventual deletion date.
- **Audit trail**: Who accessed or deleted what data, when, and why. Retention of the audit trail itself is defined separately.

---

## 4. Reliability & Resilience

> *What happens when things fail?*

Everything fails. The question is whether your service fails *gracefully* or *catastrophically*.

### 4.1 Dependency Management

Your service is only as reliable as its weakest dependency.

- **Dependency inventory**: Document every downstream service, database, cache, queue, and external API your service depends on.
- **Classify each dependency**:
  - **Hard dependency**: Service cannot function without it (e.g., primary database).
  - **Soft dependency**: Service degrades but continues (e.g., recommendation engine, analytics).
- **Design for failure of each**:
  - Hard dependencies: Fast failure detection + retries + circuit breakers.
  - Soft dependencies: Graceful degradation with fallback responses.

### 4.2 Circuit Breakers & Timeouts

- **Timeouts on every outbound call**. No infinite waits. Ever.
  - Connect timeout: how long to wait for a connection (short — 1-3s).
  - Read timeout: how long to wait for a response (context-dependent, but always defined).
- **Circuit breaker pattern**: Stop calling a failing dependency before it drags you down.
  - Closed → Open (after N failures in a window).
  - Open → Half-open (after a cooldown period, try one request).
  - Half-open → Closed (if the probe succeeds).
- **Bulkheads**: Isolate thread pools or connection pools per dependency. One slow downstream shouldn't exhaust resources for all callers.
- **Retry discipline**:
  - Retry only on transient failures (5xx, timeouts), never on 4xx.
  - Exponential backoff with jitter.
  - Max retry count. Infinite retries = infinite amplification.

### 4.3 Load Management

- **Rate limiting**: Protect your service from callers that don't know when to stop.
- **Backpressure**: When overwhelmed, push back (HTTP 429, queue consumer pausing) rather than accepting work you can't process.
- **Graceful shedding**: When at capacity, shed low-priority traffic first. Not all requests are equal.
- **Auto-scaling rules**: If applicable, define scale-up triggers (CPU, queue depth, request latency) and scale-down behaviors.

### 4.4 Data Integrity

- **Transactions**: Use database transactions for operations that must be atomic. Understand isolation levels.
- **Eventual consistency**: When using async patterns, design consumers to handle out-of-order and duplicate messages.
- **Idempotency**: Every write operation that can be retried must produce the same result when called multiple times.
- **Validation at every boundary**: Don't assume upstream sent clean data. Validate at API entry, queue consumption, and event handling.

---

## 5. Observability

> *Can you see what's happening?*

If you can't see it, you can't fix it. If you can't fix it fast, you shouldn't have shipped it.

### 5.1 Telemetry — The Three Pillars

All three. Not just the one that's easiest to set up.

#### 5.1.1 Logs

- **Structured logging** (JSON). No `System.out.println`. No `print("here")`.
- **Correlation IDs / trace IDs**: Propagated across every service boundary. A single user request should be traceable end-to-end.
- **Log levels used correctly**:
  | Level | Use for | Example |
  |-------|---------|---------|
  | ERROR | Things that wake people up | Payment processing failed, DB connection lost |
  | WARN | Things that need attention soon | Retry succeeded after 2 attempts, cache miss rate high |
  | INFO | Business events | Order created, user onboarded, deployment started |
  | DEBUG | Development context | Method entry/exit, variable state (never in production) |
- **Sensitive data redacted**: No passwords, tokens, PII, or credit card numbers in logs. Use masking patterns.
- **Log aggregation**: Centralized (ELK, Loki, CloudWatch Logs). Logs on individual containers are useless at scale.
- **Retention**: Define how long logs are kept. 30 days hot, 90 days cold is a reasonable default.

#### 5.1.2 Metrics

- **RED metrics** for every API endpoint:
  - **R**ate — requests per second.
  - **E**rrors — error count and error rate (percentage).
  - **D**uration — latency distribution (P50, P95, P99). Because averages lie.
- **USE metrics** for infrastructure:
  - **U**tilization — CPU, memory, disk, connection pool usage.
  - **S**aturation — queue depth, thread pool saturation, pending requests.
  - **E**rrors — hardware errors, OOM kills, connection failures.
- **Business metrics**: The numbers the business cares about.
  - Orders processed, payments completed, users onboarded.
  - Conversion rates, funnel drop-offs, feature adoption.
  - These matter more than CPU utilization and most teams forget to instrument them.
- **Custom application metrics**:
  - Cache hit/miss ratio.
  - Queue consumer lag.
  - Circuit breaker state changes.
  - Feature flag evaluation counts.
- **Histograms over averages**: Always. An average latency of 200ms hides the P99 at 5 seconds.

#### 5.1.3 Traces

- **Distributed tracing** across service boundaries (OpenTelemetry preferred).
- **Trace context propagated** through:
  - HTTP headers (W3C Trace Context standard).
  - Queue message attributes.
  - Async job metadata.
  - Scheduled task context.
- **Meaningful spans**:
  - Instrument the critical path with spans that tell a story.
  - Include relevant attributes (user ID, order ID, operation type).
  - Not just "entered method X" — capture *what happened* and *how long it took*.
- **Sampling strategy**: 100% tracing at low traffic, head-based or tail-based sampling at scale. Always capture 100% of error traces.

### 5.2 Dashboards — Making Telemetry Actionable

Data without visualization is noise.

- **Service health dashboard** (the first thing you open during an incident):
  - RED metrics for all endpoints.
  - Error rate trends (is it getting worse?).
  - Infrastructure health (CPU, memory, connections, disk).
  - Dependency status (up/down, latency to each downstream).
- **Business dashboard**:
  - The numbers the product team and leadership track.
  - Updated in real-time, not daily batch reports.
- **Infrastructure dashboard**:
  - Database: query latency, connection pool usage, replication lag.
  - Cache: hit rate, eviction rate, memory usage.
  - Queues: depth, consumer lag, DLQ size.
- **Dashboards as code**: Grafana JSON, Terraform definitions — committed to the repo. Hand-crafted dashboards in a UI are one accidental click from gone.

### 5.3 Alerts — Turning Visibility Into Action

An alert is a question: "Does a human need to do something right now?"

- **Alert on symptoms, not causes**:
  - Good: "Error rate on `/checkout` exceeded 5% for 5 minutes."
  - Bad: "CPU > 80%." (So what? Maybe it's handling traffic fine.)
- **Every alert must have**:
  - A clear, human-readable title.
  - Severity level with defined SLAs:
    | Severity | Response time | Example |
    |----------|---------------|---------|
    | P1 — Critical | 15 minutes | Service down, data loss, payment failures |
    | P2 — High | 1 hour | Significant degradation, elevated errors |
    | P3 — Medium | Next business day | Non-critical feature broken, slow degradation |
    | P4 — Low | Next sprint | Cosmetic issues, minor inefficiencies |
  - Link to the corresponding SOP/runbook (mandatory, see §6.1).
  - Correct routing: PagerDuty for P1/P2, Slack channel for P3, ticket for P4.
- **The golden rules**:
  - **No alert without an action.** If the response is "ignore it," delete the alert.
  - **No action without an alert.** If the team keeps manually checking something, it should be automated.
  - **Alert fatigue is a service health problem.** Review and prune alerts monthly.
  - **Alert on rates and trends, not single data points.** One error is noise. 50 errors in a minute is a signal.

---

## 6. Operational Readiness

> *Can someone else run this at 3 AM?*

The difference between "it works" and "it works when the on-call has never seen this service before."

### 6.1 SOPs for Alerts (Standard Operating Procedures)

**Every alert maps to a runbook. No exceptions.**

The runbook must be usable by someone encountering the service for the first time during an incident.

**SOP template:**
```
1. ALERT NAME & SEVERITY
   What triggered and at what level.

2. WHAT THIS MEANS (plain English)
   No jargon. Explain like the reader has zero context.

3. IMMEDIATE IMPACT
   What's broken for users right now?
   What's the blast radius?

4. DIAGNOSIS STEPS
   a. Dashboards to open (with links)
   b. Log queries to run (with exact queries)
   c. Commands to execute (with exact commands)
   d. Common root causes ranked by likelihood

5. REMEDIATION STEPS
   For each common root cause:
   a. Step-by-step fix
   b. Expected outcome after fix
   c. How to verify the fix worked
   d. Rollback instructions if the fix makes things worse

6. ESCALATION PATH
   a. When to escalate (time-based and severity-based triggers)
   b. Who to contact (name, role, phone, Slack handle)
   c. What to include in the escalation message

7. POST-INCIDENT
   a. What to document in the incident timeline
   b. Follow-up ticket template
   c. Blameless post-mortem trigger criteria
```

- **SOPs live in the repo** — next to the code, versioned, code-reviewed. Not in a wiki nobody can find under pressure.
- **Updated after every incident.** If the SOP didn't help, it's broken — fix it as part of the post-mortem.
- **Dry-run quarterly.** Muscle memory matters at 3 AM. Run game days that simulate the alert firing.

### 6.2 Health Checks & Readiness Probes

The first line of automated defense.

- **Liveness probe** — "Is the process alive?" If not, restart it.
  - Simple: responds 200 on `/health/live`.
  - Should not check dependencies — this is about the process, not the world.
- **Readiness probe** — "Can it serve traffic?" If not, remove from the load balancer.
  - Checks connectivity to critical dependencies (database, cache).
  - Returns degraded status indicating which dependency is unhealthy.
- **Deep health check** — "What's the full picture?" For diagnostics, not load balancers.
  - Checks all dependencies with latency measurements.
  - Reports version, uptime, last deployment time.
  - Protected endpoint (not publicly accessible).

### 6.3 Configuration Management

Code is versioned. Configuration should be too.

- **Externalized configuration**: Nothing hardcoded. No `.properties` files with production credentials committed to the repo.
- **Layered config**:
  - Defaults in code (sensible baseline).
  - Environment-specific overrides via environment variables or config service.
  - Runtime-toggleable via feature flags for operational switches.
- **Secrets management**:
  - Secrets in a vault (AWS Secrets Manager, HashiCorp Vault, GCP Secret Manager).
  - Never in the repo. Never in environment variables on a developer's machine.
  - Rotated automatically on a schedule. Manual rotation doesn't happen consistently.
- **Change auditing**: Who changed what configuration, when, and why. This is critical for incident diagnosis ("what changed?").

### 6.4 Security Baseline

Security isn't a feature. It's a property of every layer.

- **Authentication & authorization**: On every endpoint. Internal doesn't mean unprotected.
  - Enforce at the API gateway *and* at the service level (defense in depth).
  - Principle of least privilege for service-to-service communication.
- **Dependency scanning**: Automated in CI (Dependabot, Snyk, Trivy). Build fails on critical/high vulnerabilities.
- **Container image scanning**: Before deployment. No deploying images with known CVEs.
- **Network policies**: Services should only talk to what they need. Default-deny, explicit-allow.
- **Credential hygiene**:
  - API keys, tokens, and certificates rotated automatically.
  - Short-lived tokens preferred over long-lived secrets.
  - Service accounts with minimal permissions.
- **OWASP Top 10**: The team should be able to point to where each risk is mitigated in the codebase.

### 6.5 Service Documentation — The Owner's Manual

If the README can't onboard a new hire, the service is bus-factor 1.

- **README** (non-negotiable):
  - What does this service do? (2–3 sentences, not a novel.)
  - How do I run it locally? (Copy-paste commands that work.)
  - How do I deploy it? (Link to pipeline, not "ask the team lead.")
  - Who owns it? (Team name, Slack channel, on-call rotation.)
- **Architecture diagram**:
  - C4 model or equivalent — at least Context and Container levels.
  - Kept current — updated when dependencies change, not a relic from the initial design doc.
  - Committed to the repo as code (Mermaid, PlantUML, Structurizr).
- **Dependency map**: What this service calls, what calls this service. Inbound and outbound.
- **Decision log (ADRs)**: Why was this built this way? Architecture Decision Records that capture the *why* behind non-obvious choices.
- **On-call guide**: Escalation contacts, common issues, and links to SOPs and dashboards.

### 6.6 Incident Management & Post-Mortems

Incidents are inevitable. Learning from them is not.

- **Incident response process**: Defined, documented, rehearsed.
  - Roles: Incident Commander, Communications Lead, Technical Lead.
  - Communication channels: Dedicated Slack channel per incident, status page updates.
  - Severity-based response times aligned with alert severity (see §5.3).
- **Blameless post-mortems**: Conducted for every P1 and P2 incident within 72 hours.
  - Timeline: What happened, minute by minute.
  - Root cause analysis: 5 Whys or Fishbone — go deep, not shallow.
  - Action items: Assigned, tracked, with due dates. Not "we'll do better next time."
  - Shared publicly within the engineering org. Incidents are learning opportunities, not secrets.
- **Incident metrics tracked over time**:
  - MTTD (Mean Time to Detect) — how fast did we know?
  - MTTR (Mean Time to Recover) — how fast did we fix it?
  - Incident recurrence — are the same things breaking repeatedly?

---

## The Hierarchy at a Glance

```
A Healthy Service
│
├── 1. SOURCE & DELIVERY
│   ├── 1.1 Hosted Repository
│   │   ├── Branch protection & code review
│   │   ├── CODEOWNERS & PR templates
│   │   └── Commit hygiene & .gitignore
│   ├── 1.2 Build Pipeline (CI)
│   │   ├── Compile → Test → Analyze → Scan → Package
│   │   ├── Reproducible builds < 10 min
│   │   └── Quality gates (SonarQube)
│   └── 1.3 Deployment Pipeline (CD)
│       ├── Environment ladder (dev → staging → prod)
│       ├── Deployment strategy & rollback
│       ├── Feature flags
│       └── Audit trail
│
├── 2. APPLICATION CORE
│   ├── 2.1 APIs
│   │   ├── Contract-first & versioned
│   │   ├── Input validation & error format
│   │   ├── Rate limiting & pagination
│   │   └── Idempotency keys
│   ├── 2.2 Business Logic
│   │   ├── Domain layer separation
│   │   ├── Testable without infrastructure
│   │   └── Error handling philosophy
│   ├── 2.3 Tests
│   │   ├── Unit → Integration → Contract → E2E
│   │   ├── Meaningful coverage
│   │   └── No flaky tests tolerated
│   └── 2.4 Code Quality
│       ├── Static analysis in CI
│       └── Quality gate enforced
│
├── 3. INFRASTRUCTURE & DATA
│   ├── 3.1 Queues & Workflows
│   │   ├── DLQs & idempotent consumers
│   │   ├── Retry policies & backoff
│   │   └── Saga/state machine for multi-step
│   ├── 3.2 Cache
│   │   ├── Strategy & TTLs
│   │   ├── Invalidation documented
│   │   └── Graceful degradation
│   ├── 3.3 RDBMS / Data Store
│   │   ├── Managed migrations
│   │   ├── Connection pooling & index hygiene
│   │   ├── Read/write separation
│   │   └── Backup & restore tested
│   ├── 3.4 File Storage
│   │   ├── Object storage with lifecycle
│   │   ├── Access control & upload safety
│   │   └── CDN for public assets
│   └── 3.5 Data Retention
│       ├── Per-store retention policies
│       ├── Automated cleanup & PII handling
│       └── Regulatory compliance
│
├── 4. RELIABILITY & RESILIENCE
│   ├── 4.1 Dependency Management
│   │   ├── Hard vs. soft dependency classification
│   │   └── Failure design per dependency
│   ├── 4.2 Circuit Breakers & Timeouts
│   │   ├── Timeouts on every outbound call
│   │   ├── Circuit breaker states
│   │   ├── Bulkheads & retry discipline
│   │   └── Backoff with jitter
│   ├── 4.3 Load Management
│   │   ├── Rate limiting & backpressure
│   │   ├── Graceful shedding
│   │   └── Auto-scaling rules
│   └── 4.4 Data Integrity
│       ├── Transactions & isolation levels
│       ├── Eventual consistency patterns
│       └── Idempotency everywhere
│
├── 5. OBSERVABILITY
│   ├── 5.1 Telemetry
│   │   ├── Logs: structured, correlated, redacted
│   │   ├── Metrics: RED + USE + Business + histograms
│   │   └── Traces: distributed, sampled, meaningful
│   ├── 5.2 Dashboards
│   │   ├── Service health (first stop in incidents)
│   │   ├── Business metrics
│   │   ├── Infrastructure details
│   │   └── Dashboards as code
│   └── 5.3 Alerts
│       ├── Symptom-based, severity-leveled
│       ├── SOP link mandatory
│       ├── Routed by severity
│       └── Monthly pruning
│
└── 6. OPERATIONAL READINESS
    ├── 6.1 SOPs for every alert
    │   ├── Diagnosis → Remediation → Escalation
    │   ├── Lives in the repo
    │   └── Dry-run quarterly
    ├── 6.2 Health Checks
    │   ├── Liveness / Readiness / Deep
    │   └── Automated load balancer integration
    ├── 6.3 Configuration Management
    │   ├── Externalized & layered
    │   ├── Secrets in vault
    │   └── Change auditing
    ├── 6.4 Security Baseline
    │   ├── AuthN/AuthZ on every endpoint
    │   ├── Dependency & container scanning
    │   ├── Network policies
    │   └── Credential rotation
    ├── 6.5 Service Documentation
    │   ├── README, architecture diagram, ADRs
    │   ├── Dependency map
    │   └── On-call guide
    └── 6.6 Incident Management
        ├── Response process & roles
        ├── Blameless post-mortems
        └── MTTD / MTTR tracking
```

---

## The Scorecard

Rate your service honestly. Score each element: ✅ Present | ⚠️ Partial | ❌ Missing

| # | Layer | Element | Score |
|---|-------|---------|-------|
| 1.1 | Source & Delivery | Hosted repo with protection & CODEOWNERS | |
| 1.2 | | CI pipeline: test → analyze → scan → package | |
| 1.3 | | CD pipeline with rollback < 5 min | |
| 2.1 | Application | API contracts versioned & documented | |
| 2.2 | | Business logic isolated in domain layer | |
| 2.3 | | Test pyramid running in CI, no flaky tests | |
| 2.4 | | SonarQube quality gate enforced | |
| 3.1 | Infrastructure | Queues with DLQs, idempotent consumers | |
| 3.2 | | Cache with TTLs, invalidation, degradation plan | |
| 3.3 | | DB: migrations, pooling, indexes, backups tested | |
| 3.4 | | File storage with lifecycle & access control | |
| 3.5 | | Data retention policies defined & automated | |
| 4.1 | Reliability | Dependencies classified (hard/soft) | |
| 4.2 | | Circuit breakers & timeouts on all outbound calls | |
| 4.3 | | Rate limiting & backpressure configured | |
| 4.4 | | Idempotency & data integrity patterns | |
| 5.1 | Observability | Logs + Metrics + Traces: all three pillars | |
| 5.2 | | Dashboards as code (health, business, infra) | |
| 5.3 | | Alerts: symptom-based, severity-leveled, SOP-linked | |
| 6.1 | Operations | SOPs for every alert, dry-run quarterly | |
| 6.2 | | Health checks: liveness + readiness + deep | |
| 6.3 | | Config externalized, secrets in vault | |
| 6.4 | | Security: scanning, network policies, rotation | |
| 6.5 | | README, architecture diagram, ADRs, on-call guide | |
| 6.6 | | Incident process & blameless post-mortems | |

**Scoring guide:**
- **20+ ✅**: Production-*ready*. Ship with confidence.
- **15–19 ✅**: Production-*capable*. Address the gaps before the next incident teaches you the hard way.
- **10–14 ✅**: Production-*tolerated*. You're running on luck and institutional knowledge.
- **< 10 ✅**: Production-*reckless*. One senior engineer leaving will expose everything.

---

*This tenet evolves. Last refined: February 2026.*
