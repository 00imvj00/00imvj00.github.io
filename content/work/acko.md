---
title: "Acko — Lead, EM, Staff Engineer Experience"
date: 2026-02-25
tags: ["software engineering", "technical leadership", "architecture", "operations"]
draft: false
---

## Working at Acko

**Timeline:** July 2022 – Present

I joined as a Lead Software Engineer in the Embedded team. Over three years, I've grown from leading individual projects → to engineering managing a team → to acting as a Staff Engineer driving technical direction across multiple product lines.

---

### 2022 — Joining & First Deliveries

#### Onboarding 2.0
*Cross-domain customer onboarding platform*
- Built a platform to onboard new customers post-insurance transaction and enable cross-sell couponing
- Time-critical delivery tied to a major sales event — shipped the day before it went live
- Coordinated across mobile apps, auto, and a third-party dynamic link generation service
- Owned system design, development, and cross-team collaboration end-to-end

**What I learned:** Deadline-driven delivery with no margin for error forces you to get the design right before touching code.

---

#### Ninja Task Force
*Production reliability initiative*
- Led a group of 5–6 engineers to eliminate direct production database dependency — **2-week hard deadline**
- Resolved **20+ open data issues** in parallel across the team via daily syncs and targeted ownership
- Designed a GitHub-based query review pipeline as a permanent safe alternative to direct DB access — **eliminated 20+ manual production queries/week**
- Pipeline became the org standard for safe production data changes

**What I learned:** Leading under pressure requires splitting a complex problem into parallel tracks and keeping people moving daily.

---

#### International Travel Insurance — Start
*Greenfield retail insurance product*
- Kicked off Acko's first retail travel insurance product — no existing codebase, no prior patterns
- Integrated with SureOS, an experimental platform being built in parallel — navigated continuous architectural changes on their side
- Introduced **Domain-Driven Design (DDD)** and **Ports & Adapters architecture** to the codebase from day one
- Led integration planning with 5+ platform services: communication, KYC, Finacko, payments, and policy issuance

**What I learned:** Starting from scratch lets you set the right architectural patterns early—DDD and Ports & Adapters enabled long-term flexibility.

---

### 2023 — Shipping Travel, Leading Across Teams

#### Retail Travel Insurance — Full Delivery
*Acko's first retail travel product*
- Delivered the first working version with **4 engineers in 2.5 months** — demoed to senior leadership
- Integrated **10+ services** end-to-end across internal and external platforms
- SureOS integration required ongoing collaboration and feedback loops as their platform evolved mid-build
- The DDD + Ports & Adapters architecture paid off: the system absorbed **2+ years of product changes without structural rework**
- Zero critical production incidents post-launch

**What I learned:** Investing in architecture upfront pays off—our system handled years of change with zero critical incidents.

---

#### Credit-Life — First B2B Life Insurance
*Acko's first embedded B2B life insurance product*
- Co-led engineering delivery for a first-of-its-kind product at Acko
- Owned integrations with two central platforms (MyAccount + CX360), delivering v1 and v2 under a hard deadline
- Proactively expanded scope — added Finacko and communication layer changes that weren't originally assigned
- Delivered on time; product grew to become one of the highest-revenue generators in the Embedded LOB

**What I learned:** Proactive scope expansion and cross-team collaboration drive product success beyond initial requirements.

---

#### Unit Testing Initiative
*Engineering culture*
- Launched a unit testing culture from near-zero across the Embedded team
- Ran community calls, paired with individual engineers, gave live demos on what good unit tests look like
- Engineers started taking ownership of coverage targets — momentum became self-sustaining

**What I learned:** Building a culture takes hands-on advocacy—live demos and pairing drive real change.

---

#### GitHub Copilot — Org-Wide Rollout
*Tooling & productivity*
- Ran the initial POC independently, compiled findings into a report, presented to senior engineering leadership
- Ran a controlled A/B experiment showing **~20% improvement in both delivery speed and code quality**
- Drove adoption across most major teams at Acko — **50 licenses deployed**
- Set up weekly feedback loops; guided teams on using Copilot beyond autocomplete: code review, unit testing, logic generation

**What I learned:** Data-driven experiments and structured feedback loops accelerate org-wide adoption of new tools.

---

#### Hoppscotch — Postman Replacement
*Cost reduction*
- Identified open-source Hoppscotch as a Postman replacement; ran POC with DevOps and Travel teams
- Presented findings across all LOBs; drove org-wide adoption
- Eliminated **~$6,000/month** in tooling costs

**What I learned:** Open-source solutions can deliver massive cost savings if you drive adoption across teams.

---

### Jan 2024 – Dec 2024 — Engineering Manager

*I stepped into the EM role for the Embedded team while continuing to contribute technically.*

#### Ackcelerator v1
*B2B partner onboarding automation*
- Led development and delivery of Ackcelerator, automating end-to-end partner and product onboarding
- Reduced average onboarding turnaround from **~1 week → under 1 hour**
- Managed requirements gathering, system design reviews, code quality, and unit test standards across the team
- Mentored the intern on the project; oversaw their ramp-up and contribution

**What I learned:** Automation and mentoring can transform delivery speed and team growth.

---

#### Credit Combi
*Life + General Insurance combined product*
- Pulled into the project mid-cycle; took full ownership of delivery
- Added complexity: issuing policies across both Life Insurance (LI) and General Insurance (GI)
- Led cross-team execution; delivered on schedule

**What I learned:** Taking ownership mid-cycle and leading cross-team execution ensures delivery despite complexity.

---

#### Ledger
*Technical mentorship*
- Not a primary delivery role — guided a junior engineer (Tushar) through the full project lifecycle
- Mentored through the core research phase and technical complexity
- Drove adoption of Domain-Driven Design philosophy on the project
- Actively reviewed code and PRs; provided structured, actionable feedback throughout

**What I learned:** Mentorship and structured feedback accelerate junior engineers' growth and project success.

---

#### Travel Enhancements
*Ongoing product depth*
- Shipped couponing & rewards for travel policies
- Built real-time claims status view for customers
- Led TPA migration from existing provider to Europe Assistance
- Added pre-existing disease data collection to improve underwriting accuracy and reduce claim disputes

**What I learned:** Continuous enhancements and data-driven improvements deepen product value and reliability.

---

#### Visa Product
*New product line*
- Reduced escalations by owning customer communication flows end-to-end
- Built app-based application tracking for visa customers
- Developed a payment link tool for the CX team
- Built a Visa + Insurance bundled SKU, increasing cross-sell conversions

**What I learned:** Owning communication flows and building bundled SKUs drive customer satisfaction and conversions.

---

#### Software Task Estimation Framework
*Engineering productivity*
- Built an estimation system based on task complexity, dependency count, and system familiarity
- Collaborated with EMs across Embedded to calibrate base weights
- Adopted for sprint planning; improved accuracy of timeline forecasts

**What I learned:** Collaborative estimation frameworks improve sprint planning and delivery accuracy.

---

#### Product Metrics Monitoring & Anomaly Detection
*Observability*
- Implemented Amplitude-based alerting across key product and business metrics
- Shifted the team from reactive debugging to proactive issue detection
- Ran full POC with the Travel product team; managed data integration and configuration

**What I learned:** Proactive monitoring and alerting shift teams from reactive to preventive engineering.

---

#### Org-Wide Infrastructure Upgrades
*Reliability & compliance*
- Owned three critical, time-sensitive upgrades outside formal scope:
  - **SureOS stability upgrade** — zero downtime across all products
  - **Central Cookie Policy compliance** — met regulatory deadline with zero CX impact
  - **Finacko payment failure resolution** — identified root causes, deployed fixes during off-peak hours, eliminated revenue leakage
- Designed rollback strategies for each; produced post-upgrade documentation to prevent recurrence

**What I learned:** Owning upgrades and designing rollback strategies ensure reliability and compliance under pressure.

---

#### Engineering Management
*Jan 2024 – Dec 2024*
- Led a team of **6–7 engineers** — delivery planning, 1:1s, performance feedback, upskilling
- Conducted **20+ technical interviews** for SDE-3 and LSE roles
- Mentored engineers at all levels across Embedded and Travel pods
- Pioneered tech adoption — team was first to experiment with AI-based claims processing and microservices migration

**What I learned:** Leadership is about planning, feedback, and pioneering new technology for team growth.

---

### Jan 2025 – Present — Acting Staff Engineer

*Transitioned from EM back to a technical leadership track, driving architecture and systems across product lines.*

#### Travel-Pass — Platform Architecture
*High-visibility bundled product*
- Designed end-to-end architecture for Travel-Pass: Insurance + Visa + Claims under one journey
- Built for horizontal scalability — new categories can be onboarded with **minimal engineering lift**
- Navigated continuous scope changes from evolving product vision while maintaining technical integrity
- Ran AI/OCR POC for real-time claims processing — validated feasibility for sub-minute settlement decisions
- Maintained launch-readiness throughout despite shifting timelines

**What I learned:** Building scalable platforms and validating AI solutions enable rapid expansion and launch-readiness.

---

#### AI Gmail Integration — Travel Pass
*The most technically complex system I've built at Acko*
- Built an AI layer that connects to users' Gmail inboxes in realtime
- Classifies incoming emails as travel-related or not using an LLM-based classification layer
- Extracts structured flight information from raw email content using multiple **prompt engineering techniques** — handles varied formats, forwarded itineraries, and airline-specific layouts reliably
- Currently processing **~2,000 inboxes in realtime** with **sub-1-second per-email latency**
- Entire workflow orchestrated with **Temporal** — durable execution, full observability, fault-tolerant at scale
- This project pushed my depth in **LLM systems engineering**: prompt design, output schema validation, latency optimisation, failure handling, and scaling inference in production

**What I learned:** Building production LLM systems requires deep prompt engineering, schema validation, and scalable orchestration.

---

#### Mobile Backend Services
*New ownership — ongoing*
- Recently took on leadership of mobile backend services at Acko
- Expanding technical ownership beyond Travel and Embedded into platform-level mobile infrastructure

**What I learned:** Expanding technical ownership broadens impact and drives platform-level improvements.

---

#### SureClaims Platform — Embedded Representative
*Cross-functional technical leadership*
- Represented the Embedded LOB in design discussions for the org-wide SureClaims platform
- Provided architectural context on how Embedded's claims platform (Jarvis) operates
- Ensured Embedded-specific requirements were reflected in the platform design from the start

**What I learned:** Cross-functional leadership ensures platform designs meet all business unit requirements.

---

### Skills I've Built Depth In

**System Architecture**
Domain-Driven Design · Ports & Adapters · Microservices · Event-driven systems · Temporal workflows

**LLM & AI Systems**
Prompt engineering · Output validation · LLM-based classification · OCR/document extraction · Scaling inference in production

**Engineering Leadership**
Technical mentorship · Performance management · Hiring · Cross-functional delivery · Stakeholder communication

**Observability & Reliability**
Anomaly detection · Zero-downtime deployments · Rollback strategy design · Production incident resolution

**Languages & Tools**
Java · Spring ecosystem · Temporal · AWS tooling & services · New Relic & OpenTelemetry (logs, metrics, traces)

---

### How I Think About Engineering

- **Boundaries before implementation** — the design decisions made before writing a line of code determine system quality for years
- **LLM systems need engineering rigour** — latency, validation, failure modes, and prompt reliability are distributed systems problems
- **Culture is a system too** — testing habits, review quality, and ownership mindset compound over time just like technical debt does
- **Scope is a starting point** — the most impactful work is usually the work nobody formally assigned
