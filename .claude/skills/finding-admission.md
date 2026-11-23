# Formal Finding Admission

This is the one shared admission policy for every reviewer skill in this
folder (`.claude/skills/`). Each `SKILL.md` restates the minimal part of it so
that a single skill stays self-sufficient when installed or handed to a
subagent on its own; this file is the canonical, fuller copy. If you change
the policy, change it here first and then update the
`## Formal Finding Admission` block in each reviewer skill.

The policy is precision-first. Its job is to keep review output small enough to
act on: a reviewer that reports everything it can think of produces a list that
nobody triages carefully, and the one real defect drowns in it.

## Scope

The policy governs delivery review, meaning reviews whose findings can block or
gate a merge. It applies equally to:

- the specialist reviewer skills in this folder;
- any other reviewer you add;
- items raised by hosted AI review bots or other automated reviewers, wherever a
  triage rule in your process says "valid finding".

An explicitly requested exhaustive audit may still optimize for recall. Audit
output does not automatically become a merge-gating finding or a follow-up
issue; whoever owns the PR admits each item separately under this policy.

## Admission tests

A review item is a **formal finding** only when it identifies a concrete risk
in the reviewer's assigned domain and meets at least one of these tests:

1. the change violates an accepted requirement, decision record, contract, or
   explicit invariant;
2. the diff creates a reproducible user, operator, or system failure path;
3. it creates or preserves a security, privacy, authorization, data-loss,
   corruption, or irreversible lifecycle risk;
4. it blocks an already binding release or acceptance gate;
5. the change introduces a regression relative to the base branch;
6. it is the second observed occurrence of the same operational failure;
7. a changed high-risk behavior lacks a load-bearing test needed to distinguish
   correct behavior from a dangerous regression; or
8. an accepted measurable target is missed by reproducible evidence.

## Required fields

Every formal finding states:

- the violated requirement or invariant, or the concrete risk boundary;
- a traced failure sequence or observable consequence;
- evidence, a reproduction, or a bounded falsification step;
- severity: `Must fix before merge` or `Can track after merge`;
- confidence: `Confidence: High`, `Confidence: Medium`, or `Confidence: Low`;
- the smallest proportionate fix or disposition.

Give each finding a stable short title so it can be referenced across review
rounds and merged across reviewers.

## Severity and confidence are separate axes

Severity says what happens if the finding is real. Confidence says how sure the
reviewer is that it is real. Report both on every finding; never let one stand
in for the other.

| Severity | Meaning |
| --- | --- |
| `Must fix before merge` | The risk is serious enough that the PR should not merge at this head. |
| `Can track after merge` | The risk is real and admitted, but its impact does not block this merge. It still needs a closed disposition (see below). |

| Confidence | Typical evidence |
| --- | --- |
| `High` | Reproduced, or traced end to end through code that was actually read. |
| `Medium` | Traced through the code, but not executed; one link in the chain is inferred. |
| `Low` | Plausible and specific, but unconfirmed; depends on behavior the reviewer could not observe. |

`Can track after merge` states severity only. It does not claim that a
follow-up already exists.

## The low-confidence exception class

A low-confidence hypothesis is not a formal finding by default. It remains
reportable as one only when the potential impact is:

- security,
- privacy,
- data loss,
- an irreversible authority mutation (permissions, ownership, deletion,
  publication, anything that cannot be cleanly undone), or
- a concurrent-writer or single-writer-authority violation,

and it includes a concrete falsification step: the command, query, test, or
observation that would confirm or rule it out.

If the relevant safety invariant cannot be established at all, classify the
needed investigation truthfully ("required investigation: we cannot tell whether
X holds") rather than presenting an untraced concern as confirmed.

## What is not admitted

Do not admit as a formal finding:

- pure style, naming, or personal preference;
- speculative future scalability without an accepted target;
- general maintainability taste without a traced risk path;
- additional test ideas after the changed invariant is already load-bearing;
- adjacent cleanup outside the change;
- a merely possible vendor or platform issue without evidence.

A reviewer may still list such items separately as **non-finding
observations**. Observations carry no triage or disposition obligation.

Do not pad the finding list to prove coverage. An empty formal-finding list,
together with named residual verification gaps, is a valid and complete result.

## Dispositions

Every admitted finding ends in exactly one closed disposition, recorded where
your process keeps review triage. The PR body is a good default; with this
template's PR template, use its `## Review` section. The dispositions:

- **Fixed** at a named commit;
- **Tracked** in a named issue, with the failure it predicts;
- **Risk accepted** by someone with the authority to accept it, with a reason;
- **Declined** as not admitted, with a one-line reason naming the failed test.

A security, privacy, data-loss, authorization, authority, or irreversible
lifecycle residual should not be downgraded to a backlog line; give it its own
tracked issue.

Optional, for process and prompt documentation: a `Can track after merge`
finding against agent prompts, runbooks, or reviewer charters can go into one
rolling backlog issue instead of its own issue. Each entry names the failure it
predicts. Promote an entry to its own issue only when that failure is actually
observed, or when the owner asks.

## Hosted and automated reviewers

Hosted review bots cannot be retrained by your repository, and some have an
observable habit of re-raising already-fixed findings or raising progressively
narrower edge hypotheses. Apply the same admission tests to their items. An
item that fails the tests is recorded once as not admitted, with a one-line
reason, and creates no disposition or follow-up obligation. This changes no
gate you have chosen: if a hosted verdict is required, it is still required on
the final head.

## Re-review

A re-review is delta-focused, never a fresh whole-PR discovery pass. The
reviewer verifies its prior findings, the exact delta since the head it last
reviewed, and the surrounding invariants that the delta materially affects,
then gives a result for the current head.

After a reviewer's own first full review of a PR, a new formal finding from
that reviewer is admitted only when:

- the fix or integration delta introduces a regression;
- a correction to that reviewer's own review input still leaves it in
  violation;
- a previously missed critical invariant now has concrete evidence; or
- the new head exposes a security, privacy, or data-loss risk.

Two cases get a full first review instead of a delta review:

- a reviewer newly required at a head it has never reviewed;
- a reviewer whose PR base changed: its earlier review covered a diff that the
  base change replaced, so it reviews the new diff in full, with the base
  change standing as its delta.

Prior findings are classified as `Resolved`, `Still valid`, or
`Needs follow-up`.

**Round cap.** Once two consecutive specialist rounds return no
`Must fix before merge` against repository files, the PR is eligible to ship at
that head. A `Must fix` that is corrected only in non-file surfaces (the PR
body, PR metadata, a review record) does not move the head and does not reset
the count, unless the correction changes an input a reviewer depends on (for
example the declared review tier or routing selection), in which case each
affected reviewer looks again at the same head. The cap is a ceiling, not a
precondition: one clean round already ends review at its head. It bounds how
many heads get cleared, not whether the shipping head is cleared.
