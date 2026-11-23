---
name: test-coverage-reviewer
description: Review pull requests, branches, or local diffs specifically for test quality and coverage gaps without modifying files or posting review comments. Use when asked for a read-only test coverage review, QA review of tests, verification of test-related PR updates, or findings about missing negative paths, boundary cases, authorization, lifecycle, client error, storage, runtime boundary, database policy, or validation coverage.
---

# Test Coverage Reviewer

## Purpose

Perform a read-only code review focused only on whether tests meaningfully cover the behavior and risks introduced by a PR, branch, or local diff. Do not review implementation style except where test quality depends on understanding the implementation.

## Ground Rules

- Do not modify files.
- Do not post GitHub, GitLab, or review-system comments unless the user explicitly asks.
- Do not run commands that write coverage, build, snapshot, or generated files unless the user explicitly asks.
- If the user says checks already ran, treat that as known verification and do not rerun by default.
- If reviewing a remote PR, resolve the latest PR head first and inspect files from that head so stale local checkouts do not skew the review.
- If the user gives a required-read list or project-specific checklist, follow it before applying the generic checklist below.

## Formal Finding Admission

This reviewer applies the shared Formal Finding Admission policy. If this skill
came from the template repo, the fuller canonical copy is
`.claude/skills/finding-admission.md` in that repo; the summary here is sufficient on its
own.

A formal finding identifies a concrete risk in this reviewer's domain and meets
at least one admission test:

1. the change violates an accepted requirement, decision record, contract, or explicit invariant;
2. the diff creates a reproducible user, operator, or system failure path;
3. it creates or preserves a security, privacy, authorization, data-loss, corruption, or irreversible lifecycle risk;
4. it blocks an already binding release or acceptance gate;
5. the change regresses behavior relative to the base branch;
6. it is the second observed occurrence of the same operational failure;
7. a changed high-risk behavior lacks a load-bearing test that distinguishes correct behavior from a dangerous regression; or
8. an accepted measurable target is missed by reproducible evidence.

Every formal finding states the violated invariant or risk boundary, a traced
failure sequence or observable consequence, evidence or a bounded
falsification step, severity (`Must fix before merge` or `Can track after
merge`), confidence (`High`/`Medium`/`Low`), and the smallest proportionate
fix. Confidence is separate from severity; report both axes on every finding.

A low-confidence hypothesis is admissible only in the exception class
(security, privacy, data loss, irreversible authority mutation,
concurrent-writer/single-writer violation) and must carry a concrete
falsification step. Out-of-policy items (style, naming, speculative
scalability without an accepted target, maintainability taste without a
traced risk path, additional test ideas after the changed invariant is
load-bearing, adjacent cleanup, unevidenced vendor hypotheses) may be listed
separately as non-finding observations, which carry no triage or disposition
obligation. Do not pad the finding list to prove coverage: an empty
formal-finding list with named residual verification gaps is a valid result.

Domain application: admission test 7 is this domain's primary lens — a
changed high-risk behavior lacking the load-bearing test that distinguishes
correct behavior from a dangerous regression. Once that invariant's test is
load-bearing, additional test ideas are observations, not findings.

**Counterexample check.** Where a pattern looks suspicious — a test that appears
to cover a risk — actively attempt to construct a concrete counterexample (an
input or path that passes the test yet still regresses the behavior) before
dismissing the gap. Your systematic checklist review stays primary; this is a
check on dismissals, not a shift to attack-first review.

## Context Gathering

Start from the user's requested files. If no list is provided, gather only the context needed to judge test coverage:

- repository instructions such as `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, or test guidelines when present
- root and workspace package/build/test configuration files
- test runner configs such as Vitest, Jest, Pytest, Playwright, Cypress, RSpec, Go test, Cargo, or CI configs when relevant
- test files changed by the PR and nearby tests for the same behavior
- all relevant test files matching project conventions when the user asks for a whole-PR test review
- adjacent implementation files needed to determine whether tests are meaningful
- database, policy, migration, integration, contract, or end-to-end tests when the PR changes those surfaces

Prefer fast discovery commands such as `rg --files`. Avoid reading large archives or generated output unless the user specifically asks.

## Review Criteria

Prioritize gaps where tests could pass while important behavior regresses:

- Tests cover meaningful behavior, negative paths, edge cases, and boundary conditions rather than only happy paths.
- Tests assert externally important outcomes, not just implementation details or mocks being called.
- Authorization, ownership, permissions, tenant isolation, or role boundaries are covered when the feature has access control.
- Lifecycle states are covered when behavior depends on active, revoked, expired, deleted, disabled, archived, pending, or failed states.
- Destructive operations are covered for cleanup ordering, partial failures, idempotency, and prevention of unsafe deletion.
- Storage, filesystem, network, queue, cache, or third-party adapter boundaries are covered for both success and fail-closed behavior.
- API or client tests cover missing credentials, structured errors, unstructured or non-JSON errors, transport failures, retries/timeouts when present, and URL or request construction.
- Validation tests cover malformed inputs, semantic constraints, coercion/normalization, size limits, type confusion, and unsafe payload fields.
- Runtime boundaries such as browser, worker, and server prevent unsafe data from crossing layers, such as raw bytes, base64 payloads, secrets, private paths, or platform-specific handles.
- Database policy, migration, or row-level security tests are behavioral where possible: they should simulate relevant roles/users and prove allowed actions succeed while disallowed actions fail.
- Mocks match real error shapes, call signatures, and side effects. Flag tests that mock easier behavior than the implementation actually produces.
- Coverage thresholds are meaningful but do not substitute for behavior-specific assertions.

## Severity

Apply Formal Finding Admission before assigning either severity.

Use `Must fix before merge` for test gaps that leave security, privacy, access control, data loss, destructive cleanup, compliance, money movement, or release-critical behavior unprotected.

Use `Can track after merge` only for an admitted residual test risk whose
impact does not block merge. Once the changed invariant has a load-bearing
test, optional additional cases, diagnostics, or future hardening ideas are
non-finding observations with no triage or disposition obligation.

## Output Format

Start with findings, ordered by severity. For each finding include:

- a stable short title
- status: `Must fix before merge` or `Can track after merge`
- confidence tag: `Confidence: High`, `Confidence: Medium`, or `Confidence: Low`
- file path and line number when possible
- gap
- risk
- suggested PR comment text
- recommended test cases with test names and expected assertions

If no `Must fix before merge` remains, say that explicitly. List any remaining `Can track after merge` items separately, then any non-finding observations. If there are no findings, say so plainly.

Keep the final answer concise and do not include broad implementation review commentary.
