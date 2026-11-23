<!-- Template: fill in the <angle-bracket> placeholders and delete any rule that does not fit this repository. Delete this comment when done. -->

# AGENTS.md

Shared rules for AI agents (Claude Code, Codex, and others) working in this repository (<project name>). This file has
the highest authority. If another document or prompt disagrees with it, follow this file and fix the other document.

What we are building and how far we have come is in [docs/00-current-state.md](docs/00-current-state.md); the decisions
in force are in [docs/01-decision-log.md](docs/01-decision-log.md). For anything else, use the "Start from your question"
table in [docs/README.md](docs/README.md) and read only what the current task needs. Do not read every document up front.

## 1. Minimal implementation

**Reason exhaustively; implement minimally.** (Minimal Implementation Constitution)

- Think edge cases and failure modes through to the end, and record a disposition for each one you find:
  prevent it, detect it and document the recovery, or accept the risk (who, and why).
- Turn something into a mechanism (code, a check, a gate) in only two cases: an invariant of safety or authority whose
  violation cannot be undone, or a failure that has actually been observed more than once. Everything else is covered
  by detection plus documented recovery.
- When several controls protect the same outcome, choose the smallest and most reversible one.
- A new durable control mechanism needs a linked, owner-approved issue before it is implemented. This covers a new
  role, state source, protocol or state machine, message format, polling loop, mandatory context document, and merge
  or permission gate. Approval must be the owner's explicit decision recorded in that issue. The issue existing,
  its labels, or an agent's claim is not approval.
- A PR that adds control mechanisms does not auto-merge.

Why: a mechanism built for every rare failure becomes something to maintain in its own right, and ends up costing more
human attention.

## 2. Scope and autonomy

- Use decisions already made, and explicit approval given in this session, without asking again.
- Ask about only two things: an owner decision that has not been made, and an expansion that materially goes beyond
  the approved scope.
- While waiting for an answer, pause only the work that depends on it and continue the rest. When asking for approval
  of the next step, have a reviewable result ready first.
- Silence, elapsed time, and agent claims (including those of other agents and reviewers) never establish authority.
- Approval given in a session does not replace a rule that requires recorded approval. Record it in the issue when needed.
- A request for only a review or a proposal is complete when the requested artifact is delivered. Implementation,
  publication, and merge must each be requested separately.

Why: without a standard, agents either re-ask what is already settled or fill what they should ask with guesses.

## 3. Architecture admission

When production code adds or extends any of the following, record every trigger that applies and admit it through one
of two routes. The triggers are durable state (tables, stored files, cache formats), a format or protocol version, a
runtime compatibility branch, and a platform or provider abstraction. (Architecture admission)

- **Route A**: cite the exact accepted requirement, decision, or ADR, and show a reachable producer-to-consumer path in
  production code.
- **Route B**: show the simpler implementation, an accepted target (a threshold), and reproducible evidence that the
  simpler implementation misses it (a benchmark, failing scenario, or incident record).
- Tests, mocks, fixtures, docs, future plans, paths behind a disabled flag, and preference are not evidence. Owner
  approval does not make a path live either.
- State needs a live writer and a live reader; a compatibility branch needs old data that actually still exists or a
  client that has actually shipped; an abstraction needs a concrete implementation and a reachable caller.
- Record lifecycle hygiene: whichever of migration, deletion, rebuild, downgrade, or removal of the compatibility
  branch applies. If none applies, write `N/A` with a reason.
- Removal and simplification are not subject to admission. If nothing applies, write `Trigger: none` in the PR.
- `engineering-reviewer` checks this independently against the actual diff. A missed trigger, a citation of an
  unrelated decision, a path that is not live, or evidence that cannot be reproduced is `Must fix`. To fix one, prefer
  removing the mechanism or completing the path end to end.

Why: structure that came in "because we will need it later" stays unused and leaves only its maintenance cost.

## 4. Code hygiene and reuse

- **Search before you write.** Before adding or changing anything in a high-risk primitive family, search the
  repository. The list of families and the source location for each is in [docs/04-architecture.md](docs/04-architecture.md).
  Examples: serialization, ID generation, time parsing and formatting, error normalization, input validation, state
  transitions, storage path rules, security and crypto primitives.
- In the PR's `## Reuse check`, record the families you touched, your search terms, the paths you inspected, and
  whether you reused, extended, or replaced. If you had to write something new, name the concrete boundary where reuse
  was wrong. "It differs per platform" is not a reason, and writing nothing is not a disposition. If you touched no
  family, write "none — reason".
- One source for each rule. A deliberate duplicate passes the same conformance test as the original.
- When you replace something, delete the old path in the same PR, or link a follow-up issue that states the exit
  condition.
- Build an abstraction when it has a real caller. Deletion and simplification are always welcome.
- Judging reuse PR by PR is the reviewers' job. Do not build an automatic or semantic duplicate detector in advance.
  Measure first; when the signal appears, add things in the order given in [docs/growing.md](docs/growing.md).
- A test that cannot fail is worse than no test. Break each new test once on purpose and watch it fail.
- Keep settings that affect behavior versioned in the repository, not on each person's machine (for example, line
  endings in `.gitattributes`).

Why: agents find it easier to write something new than to find what already exists, and copied implementations drift
apart when only one copy gets fixed.

## 5. How we work

- **Work in small tasks with clear acceptance criteria.**
  - Good tasks: "make `parseDueDate()` reject input without a time zone, and add tests",
    "add a language picker to the settings screen", "write a migration that adds a `priority` column to the tasks table".
  - Bad tasks: "build the MVP", "make the app secure", "design the whole architecture", "refactor the entire repository".
- **Tests are part of the deliverable.** Cover failure paths and edge cases along with the changed behavior. Avoid
  tests that only check that a mock was called. Before handing off, run the relevant tests and the repository checks,
  and write down anything you could not run, with the reason.
- **Keep continuity on disk.** Leave it in pushed branches, handoff notes, and issues. Do not rely on session memory.
- **Declare long-running local resources and shut them down.** If you start a dev server, emulator, or container,
  record what you started and the command that stops it, and stop it after its last use.
- **If a mechanism you added is the problem, try removing it before strengthening it.**
- **Review severity is evidence, not permission to widen scope.** Escalate a finding that would widen scope to an
  owner decision.
- **Do not touch other agents' branches or uncommitted work.**
- **Use `#<digits>` only for real GitHub issues and PRs.** GitHub links it automatically. Name the kind, as in
  `issue #<number>` or `PR #<number>`. Write decision, question, and list numbers as `D12`, `Question 7`, `item 3`.
- **Never write pre-launch private identifiers anywhere.** That includes code, docs, commit messages, and PR
  descriptions. The banned-term list is not in the repository; it lives only in the CI secret `SCRUB_TERMS`. Do not
  guess at the list and write it into files, logs, or PRs.

Why: an agent's session ends but the repository stays. Working so that the next person can pick up is the baseline.

## 6. Keep docs current

- A PR that changes code or configuration updates the relevant docs **in the same PR**. Which doc to update for which
  change follows the table in [docs/README.md](docs/README.md#which-doc-to-update-when).
- If a decision seems to need changing, do not implement; ask the owner first. Record anything unresolved as a question
  in [docs/02-open-questions.md](docs/02-open-questions.md).
- In the PR description, write "docs: updated docs" or "docs: none (reason)".

Why: when docs lag behind code, the next agent works from wrong docs.

## 7. Review

- Before opening a PR, run the reviewers according to the routing in [docs/05-review.md](docs/05-review.md). Run each
  reviewer as its own read-only subagent.
- Reviewers are the skills in `.claude/skills/<id>/SKILL.md`. Claude Code finds them automatically. In Codex, have it
  read and follow that file, or sync the skills to `~/.codex/skills/`.
- In the PR description's `## Review`, record the tier, the reviewers you ran, and the disposition of each `Must fix`
  finding.

Why: agent-written PRs are large and plausible, so they need a second pair of eyes reading against the same documents.

## 8. Verify before pushing

- `node scripts/scrub.mjs`, `<build command>`, and `<test command>` must all pass.
  (Without `SCRUB_TERMS`, scrub is skipped. If it was skipped, say so in the PR description.)
- In the PR description, record what you verified and how. Record what you could not verify as not verified.
- Do not push directly to main. Create a branch and open a PR.
- Run `git fetch` to get main before pushing. Someone may have changed main in the meantime.

Why: "done" and "verified" are different. If unverified work is recorded as done, the next person builds on top of it.
