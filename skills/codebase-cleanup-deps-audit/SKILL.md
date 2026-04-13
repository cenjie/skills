---
name: codebase-cleanup-deps-audit
description: Review project dependencies for security, licensing, maintenance, and supply-chain risk. Use this whenever the user asks for a dependency audit, `npm audit` help, package vulnerability review, SBOM or license compliance checks, stale package cleanup, upgrade planning, Dependabot/Renovate triage, or a production-readiness scan that should include third-party dependencies, even if they do not explicitly ask for a "dependency audit."
---

# Dependency Audit and Cleanup

Use this skill to turn a vague "check our dependencies" request into a concrete audit with prioritized remediation. The goal is not to dump scanner output. The goal is to tell the user what matters, what can wait, and what to change safely.

## When To Use

Use this skill when the user asks to:

- audit dependencies, packages, libraries, modules, or third-party software
- check for vulnerabilities, CVEs, advisories, `npm audit` issues, or supply-chain risk
- review license compliance, copyleft exposure, or acceptable-use policy conflicts
- find stale, unused, duplicated, or outdated dependencies
- plan dependency upgrades, cleanup work, or remediation PRs
- assess Dependabot or Renovate noise and decide what should actually be merged
- include dependency health in a broader security, compliance, or production-readiness review

Do not use this skill for:

- application code review that is not meaningfully about third-party dependencies
- infrastructure or OS package audits unless the user explicitly includes them in scope
- one-off bugfixes unrelated to package management

## Inputs To Gather

Start by identifying scope before recommending fixes:

- ecosystem(s): Node, Python, Rust, Go, Java, Ruby, PHP, .NET, mixed monorepo
- manifest and lock files present
- whether the user wants read-only analysis, patch suggestions, or actual edits
- production vs development dependency sensitivity
- org constraints such as blocked licenses, upgrade freeze windows, or required tooling

If the repo has no dependency manifests or lockfiles, say that clearly and stop the audit rather than pretending to have findings.

## Audit Workflow

Follow this sequence. Skip steps that are impossible in the current environment, but say what was skipped.

### 1. Inventory The Dependency Surface

- Find manifest and lock files first.
- Separate direct dependencies from transitive ones when the ecosystem exposes that distinction.
- Note package managers and workspace structure.
- Call out missing lockfiles because that is itself a supply-chain risk.

### 2. Run The Best Available Local Checks

Prefer project-native tools when available. Examples:

- Node: `npm audit`, `pnpm audit`, `yarn audit`, `npm outdated`, lockfile inspection
- Python: `pip-audit`, `safety`, `poetry show --outdated`, `pip list --outdated`
- Rust: `cargo audit`, `cargo outdated`
- Go: `govulncheck`, `go list -m -u all`
- Java: dependency-check, Maven/Gradle dependency reports
- Ruby: `bundle audit`, `bundle outdated`

If tool execution is unavailable, infer what you can from manifests, lockfiles, and version ranges, and label those findings as lower-confidence.

### 3. Evaluate Risk, Not Just Severity

Prioritize issues using context, not scanner output alone:

- production runtime dependency beats dev-only tooling
- reachable or internet-facing code paths beat dormant packages
- known fix available beats "no patched version yet"
- abandoned or rarely maintained packages increase operational risk
- broad transitive exposure or duplicate vulnerable trees increase urgency

### 4. Check License And Policy Exposure

- identify strong copyleft, unknown, custom, or missing licenses
- compare findings against the repo or company license when known
- flag items that need legal review instead of making legal claims you cannot support

### 5. Look For Cleanup Opportunities

- packages that are outdated but low-risk
- unused or duplicate libraries serving the same purpose
- broad version ranges that undermine reproducibility
- packages that should move from runtime to dev-only scope
- obvious candidates for replacement with built-in platform features or a better-maintained alternative

### 6. Recommend The Safest Remediation Path

Favor the smallest safe change that materially reduces risk:

- patch/minor upgrades before major rewrites when they solve the issue
- targeted overrides/resolutions when full upgrades are too risky
- staged rollout for majors or framework-level dependencies
- temporary mitigations when no fix exists

Explain the tradeoff for each recommendation so the user can decide quickly.

## Output Requirements

Keep the report concise, ranked, and actionable. Do not paste raw audit JSON unless the user asks for it.

Use this structure:

## Dependency Audit Summary
- Scope:
- Ecosystems and manifests found:
- Audit confidence:
- Highest-priority action:

## Critical Findings
- Package / ecosystem:
- Risk:
- Why it matters:
- Recommended action:
- Confidence:

Repeat only for findings that truly matter.

## License And Compliance Notes
- blocked or risky licenses
- unknown or missing license metadata
- items requiring legal review

## Maintenance And Cleanup Opportunities
- outdated but non-urgent packages
- unused or duplicative packages
- lockfile or versioning hygiene issues

## Remediation Plan
- Now: immediate fixes worth doing first
- Next: medium-risk upgrades or cleanup
- Later: larger migrations or monitoring improvements

## Assumptions And Gaps
- tools you could not run
- ecosystems or folders not inspected
- anything that would change the recommendation

## Working Style

- Be explicit about whether a finding is direct, transitive, dev-only, or production-facing.
- Distinguish observed evidence from inference.
- Prefer concrete upgrade targets when they are visible locally.
- Do not recommend `--force` style upgrades casually; mention regression risk when using them.
- If the user asks for code changes, pair each dependency update with the validation you expect to run after it.
- If the audit turns up nothing important, say so plainly and summarize residual risk instead of manufacturing issues.

## Resource

Open [resources/implementation-playbook.md](C:/Users/cenji/.agents/skills/codebase-cleanup-deps-audit/resources/implementation-playbook.md) when you need deeper examples for multi-ecosystem discovery, license analysis, supply-chain review, remediation scripting, or CI monitoring patterns.
