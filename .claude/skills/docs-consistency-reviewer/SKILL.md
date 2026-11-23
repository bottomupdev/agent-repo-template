---
name: docs-consistency-reviewer
description: Review a pull request for documentation, ADR/decision-record, scope, product, and architecture consistency. Use when asked for a read-only engineering review focused on whether docs match implemented behavior, decisions are recorded durably, open questions remain unresolved, project/issue tracking docs are accurate, and historical/reference docs are clearly non-canonical. Do not use for broad code correctness, security, CI, or implementation review unless the user explicitly scopes the task to docs/product/architecture consistency.
---

# Docs Consistency Reviewer

Use this skill to perform a read-only consistency review of a PR's documentation and related implementation evidence.

## Hard Rules

- Do not modify files.
- A direct, user-facing review may post PR, issue, or review comments only when the user explicitly authorized that publication, in the initial request or in standing instructions for the session; no second request after the review is needed. A reviewer running as a delegated subagent stays read-only and returns proposed comment text to whoever publishes.
- Keep the review scoped to documentation, decision records, product/scope claims, architecture claims, and project-tracking consistency.
- Do not expand into general bug review, security review, CI debugging, formatting, or implementation cleanup unless it directly proves or disproves a docs consistency issue.
- Treat the user's review scope as authoritative. If they provide project-specific invariants, verify those invariants without baking them into the skill.

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

Domain application: a doc/code contradiction that would misdirect an
executing agent or misstate an invariant meets admission test 1; wording
taste without a traced misdirection path is an observation.

**Counterexample check.** Where a pattern looks suspicious, actively attempt to
construct a concrete counterexample before dismissing it. Your systematic
checklist review stays primary; this is a check on dismissals, not a shift to
attack-first review.

## Context Gathering

Read the project's agent instructions and source-of-truth docs before judging consistency. Common examples include:

- agent instruction files such as `AGENTS.md`, `CLAUDE.md`, `.cursor/rules`, or repo-specific equivalents
- current-state or overview docs
- decision logs, ADRs, RFCs, architecture decision records, or design docs
- product specs, MVP/scope docs, roadmap docs, scorecards, tracking docs, and open-question docs
- relevant docs under `docs/`, `architecture/`, `product/`, or equivalent repo folders

Also inspect PR metadata, changed filenames, and enough changed implementation files to verify whether the docs match code. Use GitHub tools when available. Use local `git`, `gh`, or repository files only for gaps such as branch state, project-board/issue metadata, or file-level diff details.

Avoid recursively reading archived or historical directories by default. If historical evidence is needed, start from the repo's source index or README-style routing docs and open only the specific files required.

## Review Checklist

Check whether:

- Current docs agree with the implemented behavior changed or relied on by the PR.
- Durable decisions are reflected in the project's decision log, ADRs, RFCs, or equivalent records.
- Open questions are not accidentally presented elsewhere as settled decisions.
- Settled implementation choices are not still listed as unresolved questions.
- Product scope docs match the repo's current scope boundaries and any explicit user-provided invariants.
- Architecture docs accurately describe current boundaries, ownership, interfaces, and future modularity.
- Backend, storage, infrastructure, or provider choices are described with the correct current-vs-future status.
- Project management docs match actual PR, issue, milestone, label, and project-board metadata when they make concrete claims.
- Archive, research, migration, and session-summary docs are clearly marked as historical or non-canonical when they preserve superseded ideas.
- When the repo requires a docs update for a kind of code change (for example a "which doc to update when" table), the PR made that update. With a PR template, its `## Docs` section names the updated docs, or the reason none applied, consistently with the actual diff.

## Severity Guidance

Apply Formal Finding Admission before assigning either severity; the examples
below do not admit otherwise excluded observations.

Mark a finding `Must fix before merge` when it could mislead future contributors or agents about current scope, privacy/safety invariants, accepted architecture, durable decisions, project execution state, or implementation boundaries.

Mark an admitted finding `Can track after merge` when its concrete residual docs-consistency risk does not block merge. Optional cleanup or clearer wording without a traced misdirection path is a non-finding observation.

Do not report historical/reference wording as a blocker if the file or containing folder clearly states that canonical docs and accepted decisions override it.

## Practical Workflow

1. Resolve the PR number, repository, branch, and latest head SHA.
2. Read the required docs named by the user and the repo's agent instructions.
3. Inspect the PR changed filenames and relevant diff/file contents.
4. Sample implementation files that define the product or architecture boundaries discussed by the docs.
5. Verify issue/project tracking against actual metadata when docs make concrete tracking claims.
6. Compare implementation, canonical docs, decision records, open questions, tracking docs, and historical/reference docs.
7. Produce a concise review with findings first.

## Output Format

Return findings first, ordered by severity.

For each finding include:

- A stable short title.
- Status: `Must fix before merge` or `Can track after merge`.
- Confidence: `Confidence: High`, `Confidence: Medium`, or `Confidence: Low`.
- File path and line number when possible.
- The inconsistency.
- Why it matters for product/architecture consistency.
- Suggested PR comment text.

After findings, include:

- `Docs to update before merge` with exact files, or `None`.
- `Verification notes` summarizing what was inspected and any limits, such as unavailable project-board fields, connector limitations, or local checkout mismatch.

If no blockers remain, say that clearly and list only residual can-track items that are still actionable.

## Prompt Improvements For Users

When the user provides a review prompt, preserve their project-specific checklist but encourage these additions when useful:

- Name the repository and PR number or URL.
- State whether to inspect actual GitHub issue/project metadata.
- List canonical docs to read first, plus any historical/archive routing rules.
- Provide explicit current scope invariants for the project.
- Say whether existing PR comments should be considered already addressed.
- Say whether the output should include suggested comment text only or also a final verdict.
