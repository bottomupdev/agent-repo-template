<!-- Summary: Docs and mechanisms to add as the project grows, the signal that justifies each, and its smallest form. -->

# What to add as you grow

This template starts small on purpose. Below are things that become useful as a project grows.

**Rule: add something only when its signal appears.** "We might need it someday" is not a signal. This is the minimal
implementation principle in [AGENTS.md](../AGENTS.md) applied to the docs and tools themselves.

- When you add something, record it in the decision log and update both tables in [docs/README.md](README.md). Adding a
  mandatory context document or a blocking check (a gate) adds a control mechanism, so it needs an owner-approved issue.
- If something you added produces no useful result for months, look at removing it before trying to fix it up.
- The usual order is **record → measure → block**. Do not start with a blocking check for something you have not measured.

---

## Conflicts and reconciliation record

- **Signal**: more than once, an old document (a past discussion, research, a superseded decision) disagreed with a
  current decision, and fixing the old document would have erased why things changed.
- **Smallest form**: a single `docs/conflicts.md`. Each entry sits under `C<n>: title` with three lines: "conflicting
  input", "current reconciliation", and "governing decision". Do not edit the old documents; this doc reconciles them.
  One rule at the top: the current decision log and accepted ADRs take precedence over older records.

## Project journal (+ fragment collection)

- **Signal**: "why did the direction change?" is hard to answer from the decision log alone. You keep telling the story
  from scratch to each newcomer.
- **Smallest form**: `docs/journal.md`. Update it only where a theme ends, such as a change of direction, a milestone,
  or a failure that changed how you work. Not on every PR. It is not an authoritative doc, and agents do not read it by
  default. When consolidating, start from decision log changes, merged PRs, and closed issues since the last
  consolidation PR. No PR is ever blocked on the journal being current.
- **Fragment collection**: collect journal-worthy context from conversations (options considered and dropped, reasons
  for a change of direction) as comments on a single issue. When consolidating the journal, read those comments, fold
  them in, and mark with a comment how far you got.

## archive/ and a source index

- **Signal**: documents no longer in use clutter search results and agent context.
- **Smallest form**: move them to `archive/` at the repository root, and record in `archive/source-index.md` what is
  where and why it was moved. Agents do not read `archive/` wholesale; when needed, they start from the index. Moved
  documents have no authority.

## research/ folder

- **Signal**: you need to keep research, experiment, or comparison results, but mixing them into authoritative docs
  risks having them read as decisions.
- **Smallest form**: `docs/research/YYYY-MM-DD-<topic>.md`, with the date in the name. Evidence only, no authority; it
  becomes grounds for implementation only once promoted into the decision log or an ADR. This is already stated under
  "Authority and evidence" in [docs/README.md](README.md).

## Context routing

- **Signal**: the docs have grown too large for an agent to read at once. The "Start from your question" table is not
  enough to choose what to read, or you keep hand-picking the same set of docs for each kind of task.
- **Smallest form**: first, add a "task type → sections to read (`doc#section`)" table to `docs/README.md`. Only if that
  is still not enough, add a small script that takes a task type and outputs the sections to read. For an unknown task
  type, it stops instead of making up a list.

## Advisory quality baseline

- **Signal**: you answer "is the code getting worse?" by gut feeling. The same kind of structural problem keeps coming up
  in review.
- **Smallest form**: one doc recording a measurement of a single commit with tools you already have (duplicate detection,
  dead-code detection, complexity), with the numbers and a disposition for each kind of finding. It blocks nothing.
  Record the measured commit as a main commit that survives squash merges, and do not rewrite old measurements.

## Monthly duplicate and dead-code check (advisory)

- **Signal**: what PR-level reuse checks and reviews cannot see, meaning duplicates and dead code already merged,
  starts to show.
- **Smallest form**: once a month, when the owner asks, measure main with the same tools as the baseline and compare it
  with the previous month. Report the results as a proposal issue; the owner picks what to fix. It is not a check step
  and blocks nothing. Do not attach a scheduler or automatic runs at the start.
- Do not put a model-based semantic duplicate check on every PR up front. Large feature PRs tend to exceed what it can
  see at once and produce nothing, and small PRs have few duplicates to find.

## Ratchet gate

- **Signal**: measurements show copies of a particular primitive family (one that is dangerous to diverge, such as
  security and crypto, wire formats, or ID generation) growing again and again, and review keeps failing to catch them.
- **Smallest form**: cover that one family only. Choose the source module, and check in a list of the existing sites
  outside it, each with its reason. The check counts per file so that counts **can go down but never up**. It counts
  without judging, and runs with no dependencies, network, or model calls. It is a blocking check, so an owner-approved
  issue comes first. Extend it to another family only when the same signal appears there.

## Language edition structure sync check

- **Signal**: you keep docs in two languages, and sections actually keep getting added to or dropped from only one of them.
- **Smallest form**: at first, one line in the PR template: "Did you update the other language edition?" If drift still
  recurs, a small script that compares only the heading structure of the two editions. It does not compare the content
  of the translation.
