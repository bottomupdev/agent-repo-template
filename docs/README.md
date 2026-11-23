<!-- Summary: Guide to the docs. Which doc to read for which question, which docs have authority, which doc to update for each change, and writing rules. -->

# docs/

Where <project name> records its current state, decisions, architecture, and review rules. When code changes, these
docs change **in the same PR**. Agent working rules are in [AGENTS.md](../AGENTS.md) at the repository root.

<!-- Template: italic text is guidance; "Example" blocks describe a fictional to-do app. Delete both once filled in. -->

## Start from your question

Do not read every doc from the top. Open only the doc that answers the question in front of you.

| What you want to know | Doc to read |
|---|---|
| What we are building and for whom; what works now and what does not | [00-current-state.md](00-current-state.md) |
| What is already decided; what not to do | [01-decision-log.md](01-decision-log.md) |
| What is not decided yet | [02-open-questions.md](02-open-questions.md) |
| What a term means; what a number refers to | [03-glossary.md](03-glossary.md) |
| Components and boundaries, data flow, source locations for the reuse check | [04-architecture.md](04-architecture.md) |
| Which reviewers to run on this PR; how to handle findings | [05-review.md](05-review.md) |
| Context and alternatives for a durable decision | [adr/](adr/README.md) |
| What to add as the project grows, and when | [growing.md](growing.md) |

## Authority and evidence

- **Docs with authority**: `00-current-state`, `01-decision-log`, Accepted ADRs, `04-architecture`.
  Requirements and boundaries come from these.
- **Docs that are evidence only**: `research/` (if any), docs marked as a proposal or history, and handoff notes.
  You can read them, but do not implement from them alone. They gain authority only once promoted into the decision
  log or an ADR.
- **A folder does not grant authority.** One folder can hold both accepted docs and proposals. Check a doc's status
  and source before relying on its claims.
- **When docs and code disagree, the code is right.** The doc is wrong, so fix it in the PR that found it. If the code
  violates a decision, do not implement; ask the owner.

## Which doc to update when

| If you make this change | Update |
|---|---|
| A feature is added, changed, or removed; a known gap opens or closes | 00-current-state |
| A decision is made or changed (after owner approval) | 01-decision-log — add it under a new number; mark a changed decision `Superseded` instead of deleting it. Add an ADR too if the decision is durable |
| Something cannot be decided yet; a question is settled by a decision | 02-open-questions — add a new question; leave a settled question as a tombstone pointing to the decision number |
| You start using a new term; you find the same thing called by different names | 03-glossary |
| Components, boundaries, or data flow change; the source location of a high-risk primitive moves | 04-architecture |
| A path that matches a review routing condition is added or changed; a reviewer is added or removed | 05-review (+ the reviewer list in the PR template) |
| You add doc structure (journal, archive, and so on) | Move the entry out of growing.md and update both tables in this doc |

## Writing rules

- Write short, plain sentences.
- Start every doc with a one-line `<!-- Summary: one-line summary -->`. Agents can tell what a doc holds without opening it.
- Do not put a line with a date or commit at the top of a doc. Git history knows when it changed. Such a line has to be
  fixed separately after merge, so it goes stale quickly.
- Write anything not verified as "needs verification", not "done".
- Write dates as `YYYY-MM-DD`.
- Write PRs and issues as links that include the repository name, such as
  `[<owner>/<repo>#N](https://github.com/<owner>/<repo>/pull/N)`. They keep their meaning when the repository moves or
  is quoted elsewhere.
- Do not put `#` on decision, question, or list numbers (`D12`, `Question 7`). See [03-glossary.md](03-glossary.md).
- **Do not write pre-launch private identifiers.** Docs are also covered by the banned-term check (`scripts/scrub.mjs`).
  Do not write local paths or paths to personal notes either.
- Write each fact in one place only. Link to it from other docs.

## PR descriptions

PR descriptions follow [.github/pull_request_template.md](../.github/pull_request_template.md).

- **Docs**: "docs: updated docs" (for example `docs: 00-current-state, 04-architecture`) or "docs: none (reason)".
- **Reuse check** and **Architecture admission**: per sections 3 and 4 of [AGENTS.md](../AGENTS.md). If they do not
  apply, give the reason.
- **Review**: the tier, the reviewers you ran (with a reason for each one you did not run), and `Must fix`
  dispositions. The standard is [05-review.md](05-review.md).
- **Verified / Not verified**: commands, screens, links. Anything not verified comes with a reason. If nothing, "none".
- A PR description describes the change only. Instructions for agents belong in issues or comments.
