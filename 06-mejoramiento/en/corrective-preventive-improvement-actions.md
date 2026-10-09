# Corrective, Preventive and Improvement Actions — Guardian Escolar

---

## Table of Contents

- [1. Introduction and Objective](#1-introduction-and-objective)
  - [1.1 Purpose](#11-purpose)
  - [1.2 Objectives](#12-objectives)
  - [1.3 Scope](#13-scope)
  - [1.4 Audience and Usage](#14-audience-and-usage)
  - [1.5 Terms and Acronyms](#15-terms-and-acronyms)
- [2. Conceptual Framework for ACPM Actions](#2-conceptual-framework-for-acpm-actions)
  - [2.1 Corrective Action](#21-corrective-action)
  - [2.2 Preventive Action](#22-preventive-action)
  - [2.3 Improvement Action](#23-improvement-action)
  - [2.4 Distinction Between Action Types](#24-distinction-between-action-types)
  - [2.5 Detection Sources and Application Scope](#25-detection-sources-and-application-scope)
- [3. Classification and Criteria for Severity and Priority](#3-classification-and-criteria-for-severity-and-priority)
  - [3.1 Classification Criterion by Type](#31-classification-criterion-by-type)
  - [3.2 Severity Scale](#32-severity-scale)
  - [3.3 Priority Scale and Response Timelines](#33-priority-scale-and-response-timelines)
  - [3.4 Correspondence with Incident Classification](#34-correspondence-with-incident-classification)
  - [3.5 Admissible Evidence by Action Type](#35-admissible-evidence-by-action-type)
  - [3.6 Escalation and Block Rules](#36-escalation-and-block-rules)
- [4. Documented ACPM Management Procedure](#4-documented-acpm-management-procedure)
  - [4.1 General Procedure Flow](#41-general-procedure-flow)
  - [4.2 Stages, Responsibilities and Timelines](#42-stages-responsibilities-and-timelines)
  - [4.3 Stage 2: Finding Registration](#43-stage-2-finding-registration)
  - [4.4 Stage 3: Root Cause Analysis](#44-stage-3-root-cause-analysis)
    - [4.4.1 Five Whys Technique](#441-five-whys-technique)
    - [4.4.2 Ishikawa Diagram (Six Categories)](#442-ishikawa-diagram-six-categories)
    - [4.4.3 Root Cause Acceptance Criterion](#443-root-cause-acceptance-criterion)
  - [4.5 Stage 4: Action Definition and Approval](#45-stage-4-action-definition-and-approval)
  - [4.6 Stage 5: Implementation](#46-stage-5-implementation)
  - [4.7 Stage 6: Efficacy Verification](#47-stage-6-efficacy-verification)
  - [4.8 Stage 7: Closure and Lessons Learned](#48-stage-7-closure-and-lessons-learned)
  - [4.9 Roles and Responsibilities](#49-roles-and-responsibilities)
  - [4.10 Traceability and Communication of ACPMs](#410-traceability-and-communication-of-acpms)
- [5. Master ACPM Register](#5-master-acpm-register)
  - [5.1 Register Reading Criteria](#51-register-reading-criteria)
  - [5.2 Corrective Actions](#52-corrective-actions)
  - [5.3 Preventive Actions](#53-preventive-actions)
  - [5.4 Improvement Actions](#54-improvement-actions)
  - [5.5 Register Synthesis](#55-register-synthesis)

---

## 1. Introduction and Objective

### 1.1 Purpose

This document establishes the **ACPM management procedure** (Corrective, Preventive, and Improvement Actions) for the Guardian Escolar project, along with the **master register** containing them, the **metrics** measuring them, and the **dashboard indicators** submitted to leadership review.

Guardian Escolar is a web and mobile platform for school transportation monitoring and safety: real-time GPS tracking, QR code-based boarding control, notifications and alerts. The nature of the problem it solves — traceability of every trip and timely communication with guardians — demands that any deviation detected between specification, implementation, and system operation be addressed in a **traceable, root-cause-analyzed, efficacy-verified, and evidence-closed** manner.

### 1.2 Objectives

| # | Objective | Measurable Outcome |
| --- | --- | --- |
| OB-01 | Define a standardized process for identifying, analyzing, and closing ACPMs | Single documented procedure adopted by all team members |
| OB-02 | Establish severity and priority scales ensuring consistent classification | All findings classified using agreed criteria within 24 hours of detection |
| OB-03 | Maintain a master register providing complete audit trail | Every ACPM tracked from discovery through closure with dates, owners, and results |
| OB-04 | Provide metrics enabling trend analysis and early warning detection | Monthly report with count by type, status distribution, average resolution time |
| OB-05 | Ensure lessons learned are captured and applied to prevent recurrence | Post-closure summary stored; similar findings reduced by ≥ 30% quarter-over-quarter |

### 1.3 Scope

This procedure applies to all phases of the Guardian Escolar lifecycle: design, implementation, testing, deployment, and production operations. It covers:

- **Defects** found during development testing (unit, integration, end-to-end).
- **Incidents** reported during production operations (service outages, data integrity issues, security vulnerabilities).
- **Nonconformities** identified during code reviews or architecture audits.
- **Customer/user complaints** received via support channels.
- **Audit findings** from internal or external compliance reviews.
- **Preventive opportunities** identified through trend analysis or risk assessment.

Not covered: hardware failures on GPS devices aboard buses (managed under separate field operations procedure), third-party provider outages (managed under vendor relationship management).

### 1.4 Audience and Usage

| Audience | How They Use This Document |
| --- | --- |
| Development Team | Follow procedure when logging defects or implementing fixes |
| QA Engineer | Classify test failures; track regression recurrence |
| DevOps Engineer | Manage production incident response; post-mortem documentation |
| Project Manager | Review dashboard metrics; prioritize actions per severity |
| Institution Representative | Receive summary reports of production-impacting incidents |
| Instructor/Mentor | Validate procedure adherence during milestone reviews |

### 1.5 Terms and Acronyms

| Term | Definition |
| --- | --- |
| ACPM | Corrective, Preventive, and Improvement Actions |
| CA | Corrective Action: eliminates root cause of an existing nonconformity |
| PA | Preventive Action: eliminates cause of a potential nonconformity before occurrence |
| IA | Improvement Action: enhances performance of a process or product without fixing a defect |
| RCA | Root Cause Analysis |
| SLO | Service Level Objective |
| MTTR | Mean Time To Resolve |
| MTTD | Mean Time To Detect |
| SLA | Service Level Agreement |
| KPI | Key Performance Indicator |
| JIRA/GitHub Issues | Tracking tools for logging ACPMs |
| Post-Mortem | Structured analysis of a production incident after resolution |

---

## 2. Conceptual Framework for ACPM Actions

### 2.1 Corrective Action

A corrective action addresses a **nonconformity that has already occurred**. It follows a systematic process: detect → document → analyze root cause → define solution → implement → verify efficacy → close.

Examples in Guardian Escolar context:
- A API endpoint returning 500 errors → investigate server logs → fix bug in query → add test case preventing regression.
- GPS signal drops repeatedly on specific route → analyze device firmware → update device configuration → monitor for improvement.
- QR scan fails intermittently → discover race condition in scanner logic → refactor scanning flow → load test under concurrent usage.

### 2.2 Preventive Action

A preventive action addresses a **potential nonconformity not yet realized**. It uses historical data, trend analysis, or risk assessment to identify vulnerabilities before they materialize into defects.

Examples in Guardian Escolar context:
- Monitored services show memory leak pattern over 30 days → schedule memory profiling investigation → apply fix before crash occurs in production.
- Kafka lag increasing gradually → proactively scale broker cluster → avoid event processing delays before users notice.
- Dependency audit reveals vulnerable library version → upgrade dependency in next sprint → prevent security breach from exploitation.

### 2.3 Improvement Action

An improvement action enhances **process efficiency, product usability, or system reliability** beyond current baselines. It does not fix a defect but elevates standards.

Examples in Guardian Escolar context:
- Average API latency p95 at 180ms → target ≤ 150ms through query optimization → deploy optimized version → measure improvement.
- Test coverage at 72% branch → set target ≥ 80% → assign coverage debt stories to backlog → measure quarterly.
- Build pipeline takes 25 minutes → optimize Docker layer caching → reduce to ≤ 15 minutes → measure throughput gain.

### 2.4 Distinction Between Action Types

| Dimension | Corrective | Preventive | Improvement |
| --- | --- | --- | --- |
| Trigger | Nonconformity occurred | Potential issue identified | Current state acceptable but improvable |
| Urgency | Must address | Schedule based on risk | Schedule based on effort/value ratio |
| Success Metric | Defect no longer recurs | Issue never materializes | Measurable performance uplift |
| Risk of Inaction | System degrades or breaks | Issue will eventually occur | Competitive/quality gap widens |
| Investment Level | Often reactive (can be urgent) | Proactive investment pays dividends | Incremental investment compounds value |

Decision tree for classification:

```
Has something gone wrong?
├── YES → Was it a one-time anomaly or recurring?
│         ├── One-time → Log as finding; decide if CA needed
│         └── Recurring → CA required
└── NO → Is there evidence of a potential future issue?
          ├── YES → PA required based on risk level
          └── NO → Consider IA for enhancement opportunity
```

### 2.5 Detection Sources and Application Scope

| Source | Type Detected | Typical Frequency | Example |
| --- | --- | --- | --- |
| Automated tests (CI) | Defects (CA) | Every commit | Unit test failure blocks merge |
| Manual QA testing | Defects (CA) | Per sprint | Regression discovered during UAT |
| Production monitoring | Incidents (CA) | As they occur | Alert fires: service unhealthy |
| User feedback/support tickets | Defects & UX issues (CA/IA) | Weekly | Parent reports notification delay |
| Code review comments | Nonconformities (CA) | Every PR | Style violation, missing validation |
| Architecture audit | Design debt (PA/IA) | Per milestone | Service coupling too tight |
| Performance benchmarks | Degradation trends (PA) | Monthly | Latency increasing over 3 sprints |
| Dependency scans | Vulnerabilities (PA) | Weekly | CVE detected in npm package |
| Sprint retrospectives | Process improvements (IA) | Every sprint | "We need better staging environment" |
| Leadership review meetings | Strategic initiatives (IA) | Quarterly | "Let us add multi-language support" |

---

## 3. Classification and Criteria for Severity and Priority

### 3.1 Classification Criterion by Type

Actions are classified first by type (CA, PA, IA), then scored on two independent dimensions: severity and priority.

- **Severity**: Impact magnitude if the issue remains unaddressed.
- **Priority**: Order in which the action should be executed considering urgency, effort, and resource availability.

### 3.2 Severity Scale

| Level | Description | Examples |
| --- | --- | --- |
| Critical | System unusable; data loss or security breach imminent; entire class of users affected | Auth service down; PII data exposed; all routes failing |
| High | Major functionality impaired; workaround exists but painful; significant user impact | Route assignment broken; QR scanning fails for 50%+ of attempts |
| Medium | Functionality degraded; partial feature not working; limited user impact | Notification delivery delayed > 30 seconds; map marker flickers |
| Low | Minor issue; cosmetic; negligible operational impact | Typo in error message; button alignment slightly off |

### 3.3 Priority Scale and Response Timelines

| Priority | Definition | Max Response Time | Max Resolution Time | Target Sprint |
| --- | --- | --- | --- | --- |
| P1 (Immediate) | Critical severity; production impacted | 30 minutes acknowledgment | 4 hours resolution | Current sprint / hotfix |
| P2 (Urgent) | High severity; major functionality blocked | 2 hours acknowledgment | 24 hours resolution | Current sprint |
| P3 (Important) | Medium severity; workaround available | 24 hours acknowledgment | Next sprint | Next sprint |
| P4 (Normal) | Low severity; minor issue | 48 hours acknowledgment | Within quarter | Backlog prioritized |

### 3.4 Correspondence with Incident Classification

| Incident Type | Default Severity | Default Priority | Escalation Rule |
| --- | --- | --- | --- |
| Service outage (all users) | Critical | P1 | Auto-page on-call engineer |
| Service degradation (partial) | High | P2 | Alert channel; triage in standup |
| Data inconsistency | High | P2 | Investigate within 4 hours |
| Security vulnerability | Critical-High (CVSS ≥ 7) | P1-P2 | Immediate assessment by security lead |
| Performance regression (> 20% latency increase) | Medium-High | P2-P3 | Profile and fix in next sprint |
| UX complaint (single user) | Low | P4 | Backlog item |
| Feature request | N/A (IA) | P3-P4 | Product owner prioritizes |

### 3.5 Admissible Evidence by Action Type

| Evidence Type | Accepted For | Collection Method |
| --- | --- | --- |
| Error logs | CA | Application logs, monitoring dashboards |
| Stack traces | CA | Crash reports, exception handling frameworks |
| Screenshots/screen recordings | CA, UX IA | User-submitted, QA team capture |
| Reproduction steps | CA | Standardized template with environment details |
| Metrics/trend charts | PA, IA | Prometheus/Grafana dashboards exported as PNG |
| User feedback transcripts | CA, IA | Support ticket text, interview notes |
| Code snippets | CA | Linked pull requests showing defect location |
| Audit trail entries | CA | System-generated immutable log records |
| Risk assessment matrices | PA | Scoring worksheet completed by tech lead |

### 3.6 Escalation and Block Rules

**Escalation triggers:**
- P1 incident unresolved after 2 hours → escalate to project manager + instructor
- P2 incident unresolved after 24 hours → escalate to project manager
- Same defect reappears after marked-closed → reopen with higher severity (+1 level)
- Severity disagreement between QA and dev → resolved by tech lead decision

**Work blocks:**
- Open P1/P2 CA blocks all non-essential commits to affected service
- Open security CA blocks deployments until fixed
- Coverage below threshold (< 70%) blocks merges regardless of other gates
- Unresolved critical bugs block release sign-off

---

## 4. Documented ACPM Management Procedure

### 4.1 General Procedure Flow

The ACPM procedure consists of seven stages flowing sequentially:

```
Stage 1: DETECTION                          Identified by automated systems, manual testing, monitoring, or user reports
       │
       ▼
Stage 2: REGISTRATION                      ACPM logged in tracker with classification, severity, priority, evidence attached
       │
       ▼
Stage 3: ROOT CAUSE ANALYSIS               Five Whys + Ishikawa to identify fundamental cause(s)
       │
       ▼
Stage 4: ACTION DEFINITION                 Solution designed, approved by responsible parties, acceptance criteria defined
       │
       ▼
Stage 5: IMPLEMENTATION                    Fix developed, tested, reviewed, merged
       │
       ▼
Stage 6: EFFICACY VERIFICATION             Monitor for ≥ 7 days post-deployment; confirm original symptom absent
       │
       ▼
Stage 7: CLOSURE                           Mark closed; document lessons learned; update knowledge base
```

### 4.2 Stages, Responsibilities and Timelines

| Stage | Primary Owner | Contributors | Duration Limit | Exit Criteria |
| --- | --- | --- | --- | --- |
| 1. Detection | Any team member / automated system | None | Continuous | Finding observed and documented |
| 2. Registration | Finder assigns to PM/QA | None | ≤ 24 hours from detection | ACPM created with complete metadata |
| 3. RCA | Tech Lead + Assignee | QA (if defect), DevOps (if infra) | ≤ 48 hours for P1/P2; ≤ 5 days for P3/P4 | Root cause documented with supporting evidence |
| 4. Definition | Tech Lead | Dev proposing solution, QA defining verification | ≤ 24 hours | Solution approach approved; ACs written |
| 5. Implementation | Developer(s) | Code reviewer, QA tester | Per priority timeline (see 3.3) | Fix merged; tests pass; verified against ACs |
| 6. Verification | QA + Original reporter | None | ≥ 7 days observation window | Symptom confirmed absent; no regression introduced |
| 7. Closure | PM | Tech Lead validates | ≤ 2 days post-verification | ACPM marked closed; lessons documented |

### 4.3 Stage 2: Finding Registration

Every finding must be registered in the tracking tool (GitHub Issues / JIRA) using the standardized template:

```markdown
### ACPM Template

**ID:** ACPM-{YYYY}-{NNN} (e.g., ACPM-2026-042)
**Type:** CA / PA / IA
**Title:** [One-line summary]
**Severity:** Critical / High / Medium / Low
**Priority:** P1 / P2 / P3 / P4
**Detected By:** [Name/Role/System]
**Detection Date:** YYYY-MM-DD
**Affected Component(s):** [Service name, module, screen]
**Environment:** dev / test / production
**Description:** [Detailed description including what was expected vs. what actually happened]
**Reproduction Steps:** [If applicable, numbered steps to reproduce]
**Expected Behavior:** [What should have happened]
**Actual Behavior:** [What actually happened]
**Evidence:** [Links to logs, screenshots, metrics, stack traces]
**Related ACPMs:** [Link to any previously related findings]
**Assigned To:** [Initial assignee name]
**Due Date:** [Based on priority timeline]
```

PM assigns priority and sets due date within 24 hours of registration.

### 4.4 Stage 3: Root Cause Analysis

#### 4.4.1 Five Whys Technique

Apply iterative "why?" questioning until reaching the fundamental cause (typically 3–5 iterations):

```
Problem: QR scanning occasionally fails to record boarding events.

Why #1: Because the database insert sometimes times out.
Why #2: Because the connection pool is exhausted during peak boarding periods.
Why #3: Because max pool size is hardcoded to 20, insufficient for 50 concurrent buses scanning simultaneously.
Why #4: Because pool sizing was set for development load testing, not production capacity.
Root Cause: Connection pool configuration not calibrated for production concurrency requirements.

→ Solution: Make pool size configurable via environment variable tied to fleet size metric.
```

Record the five whys chain directly in the ACPM comment thread. Attach a screenshot or export if done interactively.

#### 4.4.2 Ishikawa Diagram (Six Categories)

For complex systemic problems, use the fishbone diagram organized around six categories:

```
                    PEOPLE                PROCESS              TECHNOLOGY
                   /    |    \            /    |    \           /    |    \
                  ?      ?     ?         ?      ?     ?        ?      ?     ?
─────────────────────────────────────────────────────────────────────────────────
POLICY          REGULATIONS             ENVIRONMENT            MEASUREMENT
   |    |    \      |    |    \          |    |    \           |    |    \
   ?      ?     ?      ?      ?     ?    ?      ?     ?        ?      ?     ?
```

Each category explored for contributing factors. Most common root causes typically cluster under Technology and Process.

Categories explained:
- **People**: Human errors, skill gaps, training deficiencies.
- **Process**: Workflow flaws, missing procedures, unclear responsibilities.
- **Technology**: Software bugs, hardware failures, configuration errors.
- **Policy**: Organizational rules conflicting with operational needs.
- **Regulations**: External compliance requirements constraining options.
- **Environment**: Physical conditions, network quality, infrastructure limitations.
- **Measurement**: Inadequate monitoring, flawed metrics, blind spots in observability.

#### 4.4.3 Root Cause Acceptance Criterion

RCA is considered complete when:

1. At least one contributor from each relevant Ishikawa category has been evaluated.
2. The Five Whys chain reaches a conclusion verifiable with objective evidence (not speculation).
3. The proposed root cause explains ALL observed symptoms (if multiple anomalies exist, single-cause hypothesis rejected).
4. A concrete, actionable solution directly addresses the identified root cause.

### 4.5 Stage 4: Action Definition and Approval

Solution proposal includes:

| Element | Description |
| --- | --- |
| Proposed Fix | Technical description of change needed |
| Files Affected | List of files/modules requiring modification |
| Test Changes | New or modified test cases to prevent regression |
| Risk Assessment | Potential side effects of implementing this fix |
| Effort Estimate | Story points or hours; complexity rating (S/M/L/XL) |
| Acceptance Criteria | Specific conditions confirming the fix works |

Approval workflow:
- **P1/P2**: Tech Lead + QA sign-off required before implementation starts.
- **P3/P4**: Tech Lead sign-off sufficient.
- **Production-only fixes**: Additional approval from DevOps lead for rollback plan confirmation.

### 4.6 Stage 5: Implementation

Standard development workflow applied:

```bash
▶ Create feature branch: git checkout -b fix/acpm-2026-042-main-issue
▶ Implement changes following TDD methodology (red-green-refactor)
▶ Write unit + integration tests covering new behavior
▶ Submit PR with reference to ACPM ID in title and description
▶ Pass CI pipeline (lint → build → test → scan)
▶ Code review by peer developer + tech lead approval
▶ Merge to main branch
▶ Deploy according to priority timeline (Section 3.3)
```

Rollback plan always defined for production deployments:

```yaml
rollback_plan:
  trigger_conditions:
    - Error rate exceeds 5% for 5 consecutive minutes
    - Health check failures on > 33% of instances
  procedure:
    - Roll back container image to previous version
    - Run Liquibase rollback if schema changed
    - Re-run smoke tests
    - Verify service restoration
  estimated_rollback_time_minutes: 10
```

### 4.7 Stage 6: Efficacy Verification

After deployment, monitor for minimum 7 days:

| Check | Method | Success Criteria |
| --- | --- | --- |
| Original symptom absence | Review logs/metrics for same error pattern | Zero occurrences of original error |
| No regression introduced | Run full regression suite | All tests green |
| User-reported confirmation | Survey affected users or check support tickets | No repeat reports for ≥ 7 days |
| Performance stability | Compare pre/post metrics | Within ± 10% of baseline |
| Monitoring alert silence | Verify no new alerts triggered by same condition | Zero matching alerts in 7-day window |

If efficacy check fails → reopen ACPM at same or higher severity; escalate to P1 immediately.

### 4.8 Stage 7: Closure and Lessons Learned

Closure requires three items documented in the ACPM record:

1. **Closure Summary**: What was the actual root cause? What fix was deployed? When did it go live?
2. **Lessons Learned**: One paragraph capturing transferable insight for future work.
3. **Knowledge Base Update**: If lesson applies broadly, create/update wiki page or runbook entry.

Example closure entry:

```json
{
  "closure_summary": "Root cause was hardcoded connection pool size (20) insufficient for production burst traffic. Fixed by making pool size configurable: MIN_POOL=20, MAX_POOL=fleet_size × 2. Deployed to prod on 2026-04-15.",
  "lessons_learned": "Connection pool parameters must be derived from load test results, not default values. Add pool-sizing story to every service scaffolding checklist.",
  "knowledge_base_update": "Added 'Infrastructure Configuration Best Practices' wiki page section on capacity planning."
}
```

### 4.9 Roles and Responsibilities

| Role | Responsibility |
| --- | --- |
| Discovery Reporter | Log ACPM with complete evidence; follow reproduction steps |
| QA Engineer | Classify severity; execute verification tests in Stage 6 |
| Developer | Perform RCA (Stage 3); implement fix (Stage 5) |
| Tech Lead | Approve RCA outcome (Stage 3); approve solution design (Stage 4); resolve disputes |
| DevOps Engineer | Coordinate deployment; define rollback plan; monitor post-deployment |
| Project Manager | Set priority and due date (Stage 2); track metrics; escalate per Section 3.6 |
| Product Owner | Prioritize IA backlog; approve scope-affecting improvements |
| Institution Rep | Receive summary reports of production-impacting CAs |

### 4.10 Traceability and Communication of ACPMs

All ACPMs maintain bidirectional links:
- Each ACPM linked to its source artifact (Jira issue, GitHub issue, support ticket).
- Each linked to commits implementing the fix (`Fixes ACPM-2026-042` in commit message).
- Each linked to related ACPMs (duplicates, related root causes).
- Status visible on team dashboard updated automatically from tracker tool.

Communication cadence:
| Audience | Frequency | Content | Format |
| --- | --- | --- | --- |
| Development Team | Daily standup | Open P1/P2 items; blockers | Verbal update |
| QA Team | Weekly sync | Verification status; regression risks | Slack channel + shared doc |
| Project Manager | Weekly report | Count by severity; aging open items; MTTR trend | Email report + spreadsheet |
| Institution Rep | Monthly summary | Production incidents affecting service; resolution status | Executive brief (1 page) |
| Entire Organization | Quarterly retrospective | Top 3 lessons learned; process improvements adopted | Meeting + wiki page |

---

## 5. Master ACPM Register

### 5.1 Register Reading Criteria

The register is organized chronologically with filtering options:

| Filter | Purpose |
| --- | --- |
| Year | Group by calendar year (ACPM-{YEAR}-*) |
| Type | Show only CAs, PAs, or IAs |
| Severity | Focus on Critical/High items |
| Priority | View P1/P2 urgent items |
| Status | OPEN / IN_PROGRESS / VERIFICATION / CLOSED / REOPENED |
| Assigned To | Personal work queue |
| Module/Component | Service-specific view |
| Date Range | Custom period filter |

### 5.2 Corrective Actions (Sample Entries)

| ID | Title | Severity | Priority | Status | Closed Date | Resolution |
| --- | --- | --- | --- | --- | --- | --- |
| ACPM-2026-001 | JWT token refresh fails for expired sessions | High | P2 | ✅ Closed | 2026-02-10 | Rotated token store; added cleanup cron |
| ACPM-2026-007 | QR scanner double-scans on rapid successive taps | Medium | P3 | ✅ Closed | 2026-03-01 | Added debounce timeout (2-second cooldown) |
| ACPM-2026-015 | Map marker jitters due to GPS noise | Low | P4 | ✅ Closed | 2026-03-15 | Applied Kalman filter smoothing to positions |
| ACPM-2026-023 | Authentication service timeout during login spike | Critical | P1 | ✅ Closed | 2026-04-02 | Increased connection pool; added circuit breaker |
| ACPM-2026-031 | Bus route assignment UI shows stale data | Medium | P3 | 🔍 Verification | — | Cache invalidation added; monitoring setup |

### 5.3 Preventive Actions (Sample Entries)

| ID | Title | Severity | Priority | Status | Target Date | Action Planned |
| --- | --- | --- | --- | --- | --- | --- |
| ACPM-2026-010 | Memory usage trending upward in notification service | Medium | P2 | ⏳ Scheduled | 2026-04-20 | Profile heap dump; identify leak source |
| ACPM-2026-018 | Kafka consumer lag increasing across topics | High | P2 | 🔍 Verification | 2026-03-28 | Added partition rebalancing config |
| ACPM-2026-027 | npm packages with known CVEs in web admin | Critical | P1 | ✅ Closed | 2026-04-05 | Upgraded all packages to patched versions |
| ACPM-2026-035 | Database connection pool nearing max capacity | High | P2 | ⏳ Scheduled | 2026-05-01 | Right-size pool based on load test projections |

### 5.4 Improvement Actions (Sample Entries)

| ID | Title | Priority | Status | Baseline | Target | Improved |
| --- | --- | --- | --- | --- | --- | --- |
| ACPM-2026-005 | Reduce CI pipeline duration | P3 | ✅ Closed | 25 min | ≤ 15 min | 14.2 min |
| ACPM-2026-012 | Increase test branch coverage | P3 | ✅ Closed | 68% | ≥ 80% | 82.3% |
| ACPM-2026-020 | Add dark mode theme to admin UI | P4 | ✅ Closed | Light only | Light + Dark | Both themes functional |
| ACPM-2026-029 | Document all API endpoints in Swagger | P3 | ✅ Closed | 15% documented | 100% | All endpoints documented |
| ACPM-2026-038 | Implement Grafana on-call alert routing | P3 | ⏳ Scheduled | Pager duty email | PagerDuty integration | Setup in progress |

### 5.5 Register Synthesis

Monthly synthesis report format:

| Metric | January | February | March | April | Trend |
| --- | --- | --- | --- | --- | --- |
| Total ACPMs opened | 8 | 12 | 15 | 10 | ↘ decreasing |
| CA / PA / IA split | 6 / 1 / 1 | 8 / 2 / 2 | 10 / 3 / 2 | 5 / 3 / 2 | CA ↓, PA ↑ |
| Average age (days) | 4.2 | 5.1 | 6.3 | 3.8 | ↘ improving |
| P1/P2 count | 2 | 3 | 4 | 1 | ↘ improving |
| MTTR (days) | 3.5 | 4.8 | 5.2 | 2.9 | ↘ improving |
| Reopened rate | 12% | 17% | 20% | 8% | ↘ improving |
| Top CA category | Database | Network | Auth | Infrastructure | Varies |

Leadership dashboard highlights:

- **Overall health**: Good. Decreasing trend in total ACPMs and MTTR indicates maturing processes.
- **Concern area**: P3 volume still high. Consider reducing medium-priority backlog by dedicating sprint capacity.
- **Improvement focus**: Shift from reactive CA to proactive PA and IA investment. Target 30% PA+IA ratio by Q3.

---

*Document end — Corrective, Preventive and Improvement Actions Management for Guardian Escolar.*
