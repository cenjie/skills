---
name: code-review-expert
description: Expert review of current code changes, pull requests, staged edits, or commit ranges. Use this whenever the user asks for a code review, PR review, diff review, merge-readiness check, or feedback on specific changes rather than a repository-wide audit. Focus on correctness, security, maintainability, regressions, and review findings that should block or shape the change before merge.
---

# Code Review Expert

Review the current change set like a senior engineer doing merge-gating review. The goal is to identify the issues that matter before the code lands: correctness bugs, regressions, security problems, weak design choices, and missing tests.

This skill is for reviews of a diff, PR, staged changes, or a specific commit range. If the user wants a broad repository health check, production-readiness assessment, or multi-dimensional audit of an entire codebase, prefer the `code-audit` skill instead.

## What Good Looks Like

A strong review:
- Focuses on the changed code and the contracts it can break
- Prioritizes findings over summaries
- Explains why a problem matters in production terms
- Distinguishes blocking issues from optional suggestions
- Avoids speculative architecture advice that is not grounded in the diff

Use this priority order throughout the review:

```text
Correctness > Security > Reliability > Maintainability > Performance > Style
```

## When To Use This Skill

Use this skill when the user asks for:
- A review of the current git diff, staged changes, or a pull request
- A merge-readiness check on a feature branch
- Feedback on a specific commit range or set of modified files
- A senior-engineer pass on implementation quality
- Review comments, blocking findings, or regression detection before shipping

Do not use this skill for:
- Repository-wide audits or health checks
- Production-readiness or compliance reviews across an entire system
- Requests that are primarily asking you to implement code instead of review it

## Severity Levels

Use these levels consistently.

| Level | Use when |
| --- | --- |
| P0 | Security vulnerability, data loss, auth bypass, or a correctness bug that must block merge |
| P1 | Significant logic error, regression risk, broken contract, or serious maintainability problem |
| P2 | Local code smell, missing guardrail, weak abstraction, or moderate test gap |
| P3 | Minor clarity, naming, or cleanup suggestion |

Bias toward P0 or P1 when user impact is immediate or the blast radius is large.

## Workflow

### 1. Scope The Review

Start by establishing exactly what changed.

Use the smallest set of commands that gives you context:
- `git status -sb`
- `git diff --stat`
- `git diff`
- `git diff --cached` when staged changes matter
- Additional targeted search with `rg` for callers, contracts, and related tests

Before writing findings, answer these questions:
- What files changed?
- What behavior changed?
- What interfaces, invariants, or downstream consumers could break?
- Which paths are high-risk: auth, payments, persistence, concurrency, migrations, network boundaries?

If the diff is large, group it by feature or subsystem instead of reviewing in file order.

### 2. Inspect For High-Value Risks

Focus review effort on issues the author would want caught before merge.

#### Correctness and Regressions
- Broken logic, edge cases, missing null or error handling, off-by-one mistakes
- Contract drift between caller and callee, API and UI, schema and code
- Incomplete updates where one code path changed but siblings did not
- Accidental behavior changes hidden inside refactors

#### Security and Safety
- Missing auth or authorization checks
- Injection, XSS, SSRF, path traversal, unsafe deserialization, secret exposure
- Tenant isolation gaps, unsafe logging, or trust-boundary violations

#### Reliability and Operations
- Race conditions, retry storms, unbounded work, background job hazards
- Partial failure handling, idempotency gaps, timeout issues, missing cleanup
- Monitoring or alerting blind spots created by the change

#### Maintainability and Design
- Mixed responsibilities, weak abstractions, duplicated logic, confusing ownership
- SOLID violations that materially raise future change cost
- New complexity that is not justified by the problem

#### Performance
- N+1 queries, extra renders, expensive loops, unnecessary allocations, cache misses
- Bundle or runtime regressions that are plausible from the changed code

#### Tests
- Missing tests for the critical path of the change
- Assertions that do not actually protect the behavior being modified
- Flaky or over-mocked tests that hide regressions

### 3. Ground Every Finding

Every finding should be tied to evidence in the diff or directly affected code.

For each finding, include:
- File and line reference
- Severity: P0, P1, P2, or P3
- Why it matters
- The smallest credible fix or follow-up

Avoid padding the review with generic best practices. If something is only a question, label it as a question rather than a defect.

### 4. Handle Review Edge Cases

- If there is no diff, say so plainly and ask whether to review staged changes, a commit range, or specific files.
- If generated files changed, focus on the source change unless the generated output itself is suspicious.
- If the change mixes unrelated concerns, review by concern and call out the coupling as a finding if it raises risk.
- If you cannot verify an area, say so in a short `Not Verified` note.

## Output Format

Lead with findings. Keep the summary brief.

Use this structure:

```markdown
## Findings

### P0
1. [file:line] Short title
- Why it matters
- Suggested fix

### P1
...

### P2
...

### P3
...

## Open Questions

## Review Summary
- Files reviewed: X
- Overall assessment: APPROVE, COMMENT, or REQUEST_CHANGES
- Not verified: ...
```

Rules:
- Number findings continuously across severity sections
- Put the most important finding first, regardless of file order
- If there are no findings, state that explicitly and list residual risks or testing gaps

## Inline Review Comments

When emitting inline findings, use the review comment directive with tight line ranges and concise explanations.

## Closing Step

After the review, offer next-step choices only if findings exist or the user is deciding what to do next.

Use this format:

```markdown
## Next Steps

I found X issues (P0: _, P1: _, P2: _, P3: _).

1. Fix all findings
2. Fix only P0 and P1 findings
3. Fix specific findings you choose
4. Stop at the review report
```

Do not implement fixes unless the user explicitly asks for them.

## Metadata

Version: 1.1.0
Status: Active
Last Updated: 2026-03-12
