---
name: code-audit
description: Comprehensive code audit for bugs, security, performance, architecture, and production readiness. Use this whenever the user asks for a code audit, security audit, architecture audit, production-readiness review, compliance-oriented scan, or a broad health check of a repository, service, feature area, or codebase. Prefer this skill over PR review skills when the task is repository-wide or multi-dimensional rather than a review of a specific diff.
---

# Code Audit

Run a broad, risk-first audit of the target codebase. The goal is to find the issues that matter most, explain why they matter, and give the user a practical path to fix them.

This skill is for repository-scale or feature-scale audits. If the user wants feedback on a specific PR, commit range, or current git diff, prefer the `code-review` skill instead.

## What Good Looks Like

A strong audit:
- Finds real defects, not generic advice
- Prioritizes user and business risk over style preferences
- Maps findings to concrete files, flows, and boundaries
- Explains impact and remediation clearly
- Scales depth to the user's request instead of exhaustively reading everything

Use this priority order throughout the audit:

```text
User safety > Correctness > Maintainability > Performance > Elegance
```

## When To Use This Skill

Use this skill when the user asks for any of the following:
- A full codebase audit or health check
- A security review across a repo, service, or feature area
- A production-readiness or launch-readiness assessment
- An architecture, reliability, or maintainability audit
- A compliance-oriented review where code, data handling, and operational controls all matter
- A pre-migration, pre-acquisition, or pre-refactor technical assessment

Do not use this skill for:
- Reviewing only the current diff or pull request
- Pure style cleanups
- Small bug fixes with no request for an audit

## Scope Calibration

Match the depth of the audit to the request.

| Request shape | Default approach | Deliverable depth |
| --- | --- | --- |
| Single file or function | Stay local; inspect only adjacent dependencies as needed | Findings + short summary |
| Feature or module | Map inputs, outputs, auth, data flow, and integration points | Core deliverables |
| Full repository | Run full discovery, then audit hotspots first and widen only where justified | Core + extended deliverables |
| Targeted audit, such as security or performance | Go deep on the requested dimension and check nearby failure modes | Core deliverables scoped to target |

## Workflow

### 1. Discovery

Build a short mental model before listing findings.

Inspect enough of the repo to answer:
- What languages, frameworks, runtimes, and package managers are in use?
- What are the main entry points and request or execution flows?
- Where do auth, data access, external calls, and background jobs live?
- What environments exist: browser, server, edge, mobile, workers, CLI, CI?
- Where are config, secrets, env vars, deployment files, and pipeline definitions?
- What documentation exists, and where does it disagree with the code?

Produce a brief `Repo Mental Map` before the main findings.

### 2. Risk-First Pass

Do not audit every file with equal weight. Start with the highest-risk surfaces:
- Auth, session, and authorization logic
- Payments, money movement, quotas, or billing
- PII, secrets, uploads, and logging
- Public endpoints, webhooks, admin paths, background jobs, and external integrations
- Shared libraries that many packages depend on

Use lightweight search to map blast radius before going deep.

### 3. Deep Inspection Areas

Inspect only the areas that are relevant to the stack and request.

#### Security and Safety
- Input validation, output encoding, injection risks, XSS, SSRF, path traversal
- Authentication, authorization, tenancy checks, and session handling
- Secrets exposure, unsafe public env vars, token storage, and credential handling
- PII leakage in logs, analytics, caches, or client storage
- CORS, CSRF, headers, rate limiting, abuse resistance, and auditability
- Dependency and supply-chain risks when evidence is available

#### Correctness and Reliability
- Broken invariants, missing edge-case handling, race conditions, partial failure paths
- Inconsistent error handling, swallowed exceptions, retry storms, and non-idempotent flows
- Unsafe assumptions across async boundaries, queues, webhooks, or cron jobs
- State synchronization issues between UI, API, cache, and database

#### Data Layer
- N+1 patterns, unbounded queries, missing indexes, and over-fetching
- Migration safety, backward compatibility, and rollback risk
- Connection management for the runtime model: long-lived server, serverless, edge, mobile sync
- Validation split across client, service, and database layers

#### API and Service Boundaries
- Contract validation, error shape consistency, and breaking-change risk
- Middleware ordering and missing enforcement layers
- Caching, retries, timeouts, circuit breaking, and observability around external dependencies
- Separation between transport concerns and business logic

#### Frontend or Client Surfaces
- Secret leakage into bundles or client state
- Loading, error, empty, and optimistic UI correctness
- Accessibility, form safety, navigation integrity, and unsafe rendering
- Bundle size, render churn, and expensive client-side work on hot screens

#### Architecture and Maintainability
- Boundary leaks, circular dependencies, and mixed responsibilities
- Oversized modules, duplicated logic, and unstable abstractions
- Platform leakage across shared packages
- Weak test seams, poor naming, and high-change hotspots that signal design problems

#### Tooling and Operations
- Type safety posture, lint enforcement, and test coverage quality
- CI guarantees, reproducible builds, lockfile hygiene, and release safety
- Logging, tracing, alerts, health checks, and incident-readiness gaps
- Compliance-relevant controls when the user asks for them or the domain makes them material

### 4. Evidence Standard

Every finding should be grounded in repo evidence.

For each finding, include:
- Exact location: file path and symbol, route, job, or flow when possible
- Severity: Critical, High, Medium, or Low
- Confidence: High, Medium, or Low
- Category: Security, Correctness, Performance, Architecture, Testing, Operations, or Compliance
- Why it matters: user impact, exploitability, reliability risk, or cost
- Fix: the smallest credible remediation or next change set
- Tests: what should be added or updated to keep the issue from returning

Avoid generic recommendations with no evidence.

### 5. Assumptions and Limits

If something important is unclear:
- Make the narrowest reasonable assumption
- State the assumption explicitly
- Keep auditing instead of stopping

If you could not verify an area, say so plainly in a `Not Verified` section.

## Severity Guidance

Use severity consistently.

| Severity | Use when |
| --- | --- |
| Critical | Account takeover, fund loss, data loss, major auth bypass, exploitable secret exposure, severe PII leak |
| High | Likely production incident, privilege gap, unsafe data handling, major correctness bug, or easy-to-hit performance collapse |
| Medium | Material maintainability or reliability issue, partial protection gap, moderate perf waste, weak test coverage on critical paths |
| Low | Minor robustness, clarity, or local design issue with limited blast radius |

Bias upward when exploitability and impact are both high.

## Deliverables

### Core Deliverables

Always provide these sections in this order:

```markdown
## Repo Mental Map

## Executive Summary

## Top Risks

## Findings
```

Rules:
- `Executive Summary`: 5-10 bullets maximum
- `Top Risks`: the most important issues first, ordered by real-world impact rather than file order
- `Findings`: grouped by subsystem, package, or risk area, not by the order you discovered them

### Finding Template

Use this structure for each finding:

```markdown
### [Severity] Short Title
- Location: path/to/file.ts:123 (`functionName`)
- Category: Security
- Confidence: High
- Issue: What is wrong
- Why it matters: User or business impact
- Fix: Minimal remediation or next change
- Tests: Specific test coverage to add or update
```

### Extended Deliverables

Include these when the user asks for a deep audit or when the repository scope warrants them:
- `Security Checklist`
- `Performance Hotspots`
- `Refactor Proposals`
- `Operational Guardrails`
- `Sequenced Action Plan`

Keep these concise and tied to evidence already surfaced.

## Audit Heuristics

Use these heuristics to stay sharp:
- Follow trust boundaries before reading utility code
- Prefer one verified severe issue over ten speculative low-value notes
- Flag repeated patterns once, then note blast radius
- Recommend small, orderable fixes instead of rewrite fantasies
- Separate confirmed findings from suspicions
- Distinguish missing evidence from evidence of absence

## Response Style

Write like a senior engineer reporting to an owner who needs to act.

- Be direct and specific
- Default to findings first, summary second if the user asked for a review
- Do not pad the report with generic best practices
- Call out what you did not verify
- Do not implement fixes unless the user explicitly asks for them after the audit

## Closing Step

End with a clear next-step choice only when the audit is complete.

Use this format:

```markdown
## Next Steps

I found X issues (Critical: _, High: _, Medium: _, Low: _).

1. Fix all issues
2. Fix only Critical and High issues
3. Fix specific findings you choose
4. Stop at the audit report
```

Wait for the user's direction before changing code.

## Metadata

Version: 1.1.0
Status: Active
Last Updated: 2026-03-12
