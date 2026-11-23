---
name: backend-api-reviewer
description: Review pull requests or local changes for backend/API correctness only, especially authentication, authorization, ownership checks, data-access policies such as row-level security, resource lifecycle transitions, upload/download URL flows, storage adapter boundaries, public token access, event recording, and API error semantics. Use when the user asks for a read-only backend/API review, wants findings ordered by severity, or asks not to modify files or post PR comments yet.
---

# Backend API Reviewer

## Overview

Use this skill to perform a read-only code review focused only on backend and API correctness. Prioritize concrete bugs, security or privacy regressions, authorization gaps, invalid state transitions, broken API contracts, storage misuse, and missing tests for risky edge cases.

Do not modify files or stage commits in the reviewer role; requested implementation
belongs to whoever is authorized to implement. A direct, user-facing review may
post GitHub comments only when the user explicitly authorized that publication,
in the initial request or in standing instructions for the session; no second
request after the review is needed. A reviewer running as a delegated subagent
stays read-only and returns proposed comment text to whoever publishes.

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

Domain application: authorization, data-access policy, ownership, and
single-writer risks sit in the exception class (they are authority, data-loss,
or concurrent-writer impacts) — report them at any confidence with a concrete
falsification step. An irreversible lifecycle risk meets admission test 3 on
its own; a reversible lifecycle hypothesis at low confidence is a non-finding
observation, as is an API-shape preference without a traced failure.

**Counterexample check.** Where a pattern looks suspicious, actively attempt to
construct a concrete counterexample before dismissing it. Your systematic
checklist review stays primary; this is a check on dismissals, not a shift to
attack-first review.

## Workflow

1. Resolve the review target:
   - If the user gives a PR number, branch, commit, or URL, inspect that exact target.
   - If a GitHub connector or CLI is available, prefer it for PR metadata and patches.
   - If local checkout state may be stale, verify the latest PR head before reviewing.

2. Read required context:
   - Read repository agent instructions first when present, such as `AGENTS.md`, `CLAUDE.md`, or `.github/copilot-instructions.md`.
   - Read product, API, architecture, security, or decision-record docs named by the user.
   - If the user names project-specific files, read those files and the closest tests.
   - If the user does not name files, discover relevant backend/API files with `rg --files` and inspect routes/controllers, auth helpers, repositories/services, storage adapters, database migrations/policies, public endpoints, and tests.
   - Do not recursively read historical archives or generated folders unless the repo instructions say to.

3. Keep scope strict:
   - Review backend/API correctness only.
   - Ignore frontend styling, copy, product polish, dependency hygiene, and broad architecture preferences unless they directly affect API correctness, security, privacy, or data integrity.
   - Do not propose broad rewrites when a targeted guard, transaction, validation, or test would fix the issue.

4. Check backend/API correctness:
   - Authentication: bearer/session token parsing, verification with the trusted auth provider, expired or invalid token handling, and 401/403 mapping.
   - Authorization and ownership: every read/write must prove the caller owns or can access the target resource, especially when service-role or admin clients bypass database access policies (for example, row-level security).
   - Resource lifecycle: create/draft/publish/revoke/archive/delete or equivalent transitions must be explicit, idempotent where needed, and reject invalid transitions.
   - Upload/download flows: signed URL creation must be limited to authorized users and valid resource states; attachment/finalization must validate path/key ownership, object/part type, ordering, size/type limits, and metadata integrity.
   - Storage boundaries: API code should use storage adapters instead of concrete provider calls when the project has an adapter; paths should be provider-neutral and validated before read, write, sign, or delete.
   - Public token access: public links or invite tokens must be unguessable, hashed or otherwise protected at rest when appropriate, revocable, expiry-aware, and scoped to the intended resource.
   - Event recording: public or client-supplied events must be sanitized, allowlisted, bounded, and prevented from carrying sensitive raw identifiers or user content.
   - Side-effect ordering: durable state changes, storage cleanup, invite revocation, and analytics logging must not leave exposed, deleted, or half-published resources when one step fails.
   - Error semantics: return appropriate status classes for auth, authorization, validation, missing resources, and server failures without leaking sensitive details.
   - Database enforcement: migrations, constraints, indexes, access policies, uniqueness, and foreign keys should backstop application-layer checks.
   - Tests: verify the risky paths above have focused unit, integration, or access-policy tests, including failure and race/idempotency cases.

5. Treat findings as bugs, not preferences:
   - Describe a concrete failure scenario for each finding. A plausible but unconfirmed scenario is admissible at `Confidence: Low`/`Medium` only under the Formal Finding Admission exception class, with a concrete falsification step; otherwise list it as a non-finding observation.
   - Prefer "must fix" only when the issue can cause unauthorized access, privacy exposure, data loss, broken public links, invalid irreversible state, incorrect uploads/deletes, or materially wrong API behavior.
   - Mark only admitted residual backend/API risks that do not block merge as "can track after merge". Optional hardening, extra coverage, or future concurrency polish without an admitted risk is a non-finding observation.

## Output Format

Start with findings, ordered by severity.

For each finding include:

- A stable short title.
- Severity label: `Must fix before merge` or `Can track after merge`.
- Confidence: `Confidence: High`, `Confidence: Medium`, or `Confidence: Low`.
- File path and line number when possible.
- Concrete failure scenario.
- Why it matters for backend/API correctness.
- Suggested PR comment text that can be pasted later.

If no merge-blocking issues remain, say explicitly:

```text
No Must Fix remains.
```

Then list residual `Can track after merge` items separately. If there are none, say:

```text
Can Track: None.
```

Keep summaries short. Do not hide findings below a long overview.

## Reusable Prompt Template

Use this structure when the user wants a backend/API review prompt:

```text
Review the given PR or change set.

Focus only on backend/API correctness. Do not modify files. Do not post PR comments yet.

Read first:
- Repository agent instructions, if present.
- Product/API/security/architecture docs named in the task.
- Backend/API route handlers, auth helpers, service/repository layers, storage adapters, database migrations/policies, public endpoint handlers, and relevant tests.

Check:
- Bearer/session token validation and auth error mapping.
- Authorization, ownership, tenant, or scope checks, including privileged clients that bypass database access policies.
- Resource lifecycle transitions and invalid-transition handling.
- Upload/download URL creation, attachment/finalization ordering, and metadata validation.
- Publish/revoke/delete/archive behavior and side-effect ordering.
- Storage adapter usage and path/key validation.
- Public token/link verification, revocation, expiry, and scoping.
- Public/client event recording boundaries and payload sanitization.
- Database constraints, access policies, uniqueness, and tests backing up application checks.

Output:
- Findings first, ordered by severity.
- Include file path, line number if possible, concrete failure scenario, and suggested PR comment text.
- Mark each finding as "Must fix before merge" or "Can track after merge".
- Tag each finding with a confidence level ("Confidence: High/Medium/Low"); admit findings under the Formal Finding Admission policy, and list out-of-policy items separately as non-finding observations.
- Include missing tests or edge cases.
- If no Must Fix remains, say that explicitly and list residual Can Track items separately.
```

## Prompt Improvements To Suggest

When adapting a project-specific prompt, suggest these improvements when useful:

- Replace product-specific nouns with generic resource names, or define a project-specific resource map at the top.
- Include the exact PR number, branch, or commit and ask the reviewer to verify the latest head.
- Name the relevant backend entry points and tests, but allow the reviewer to discover analogous files in other stacks.
- Distinguish merge blockers from follow-up hardening.
- Ask for concrete failure scenarios and suggested PR comment text.
- Add explicit side-effect ordering checks for state changes, storage deletion, token creation/revocation, and analytics.
- Add database-enforcement checks, not just application-code checks.
- State whether comments should be posted or only drafted.
