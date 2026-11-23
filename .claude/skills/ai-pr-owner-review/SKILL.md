---
name: ai-pr-owner-review
description: Explain AI-authored pull requests to a product owner or technical lead and separate merge blockers from follow-up work. Use when the owner requests PR explanation or interactive teach-back, or an explicit existing owner-review gate applies. A routine merge request alone does not require interactive teach-back.
---

# AI PR Owner Review

## Purpose

Use this skill to help a product owner, founder, tech lead, or maintainer understand an AI-authored PR deeply enough to supervise it. Explain the implementation and product/architecture tradeoffs. Run an interactive teach-back only when the owner requests it or an explicit existing gate requires it; AI authorship, non-triviality, or a routine merge request alone creates no such gate.

Preserve explicit owner-decision and risk-acceptance gates, reusing decisions
already recorded for the same action and scope. If the repository keeps a
per-PR delivery or decision record, record teach-back questions with model
answers there when an owner decision, an operational procedure, or a material
risk acceptance must be retained; routine delivery can mark that field `N/A`
with a short reason. Writing that record does not itself require waiting for
the owner to answer the questions.

## Core Workflow

1. Establish the review target.
   - Identify the PR, branch, commit range, or local diff.
   - Check repository instructions such as `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, architecture docs, decision records, product specs, issue descriptions, and PR templates.
   - If GitHub tools are available and the user names a PR, inspect PR metadata, changed files, review comments, and linked issues.

2. Map the implementation.
   - Summarize the PR by layers: product flow, UI/client, backend/API, data model, storage, auth/access control, background jobs/native code, tests, docs, and ops.
   - Explain each layer in plain language for a developer who is new to the stack.
   - Name the important files and the role they play.
   - Describe the main runtime path from user action to persisted state and back to user-visible output.

3. Compare against source-of-truth decisions.
   - Identify which product promises, architecture boundaries, privacy/security constraints, and non-goals the PR is supposed to preserve.
   - Call out where the implementation enforces those constraints in code, tests, database policies, or docs.
   - Distinguish intentional MVP limitations from accidental gaps.

4. Identify owner-critical understanding points.
   - List the concepts the owner must understand to direct future work.
   - Highlight irreversible or expensive-to-change decisions.
   - Highlight places where a future agent could accidentally violate the product direction.
   - Separate merge blockers, post-merge follow-ups, and open product decisions.

5. When interactive teach-back is requested or explicitly required, ask teach-back questions.
   - Ask concise questions that force the owner to explain the design in their own words.
   - Cover product promise, data flow, trust boundaries, lifecycle/state machine, failure handling, security/privacy, testing gaps, and future extensibility.
   - Avoid trivia. Prefer questions that reveal whether the owner can supervise future changes.

6. If the owner supplies answers, evaluate them.
   - Confirm correct understanding directly.
   - Correct misunderstandings with specific, concrete explanations.
   - Add missing nuance only where it affects product direction, merge safety, or future agent work.
   - If the answers reveal a documentation gap, recommend the exact doc, decision record, issue, or PR template update.

7. End with an action-oriented merge readiness summary.
   - State the remaining owner-supervision requirements, if any; do not infer understanding or approval from absent answers.
   - List remaining blockers, if any.
   - List follow-up issues or checks that can happen after merge.
   - Call out unverified checks explicitly (with a PR template, compare them against its `## Not verified` section).

## Output Shape

When first explaining a PR, use this structure unless the user asks for something else:

- Overall purpose of the PR.
- Architecture and file map.
- Runtime flow.
- Important product/privacy/security boundaries.
- Testing and verification coverage.
- MVP limitations and follow-ups.
- Teach-back questions for the owner when interactive teach-back is requested or explicitly required.

When evaluating the owner's answers, use this structure:

- What they understood correctly.
- Corrections or missing nuance.
- Owner-critical takeaways.
- Recommended docs/issues/template updates.

## Review Heuristics

Treat these as especially important in AI-authored PRs:

- Does the code match the product promise, or did it silently change the product?
- Are trust boundaries explicit and tested?
- Are lifecycle states clear, and are invalid transitions blocked?
- Are privacy-sensitive values kept out of logs, analytics, URLs, storage, and client-side code where appropriate?
- Does the implementation rely on a concrete provider where an interface was promised?
- Are failure paths and cleanup paths designed, not just happy paths?
- Are tests protecting behavior and edge cases, or mostly asserting mocks?
- Are manual verification gaps named in a way that can become issues?
- Are docs and decision records updated when durable decisions changed?

## Question Bank

Adapt these questions to the PR:

- What product promise does this PR protect or advance?
- What data enters the system, where is it transformed, and where is it persisted?
- Which parts of the implementation are provider-specific, and which are abstracted?
- What can a normal authenticated user do directly, and what requires a server-side privileged path?
- What sensitive data must never be stored, logged, uploaded, or exposed?
- What lifecycle states exist, and which operations are allowed in each state?
- What happens if the operation fails halfway through?
- Which tests prove the high-risk boundaries, and which checks are still manual?
- Which limitations are acceptable for MVP, and which must be fixed before beta or merge?
- What future AI-agent task would be risky without owner review?

## Ground Rules

- Do not assume the PR is correct because tests pass.
- Do not overwhelm the owner with every file; prioritize mental models and risk-bearing details.
- Do not hide uncertainty. Mark inferences and unverified claims.
- Do not create or modify code unless the user explicitly asks for implementation work.
- Prefer concrete file references and state transitions over abstract reassurance.
