<!-- Summary: When to write an ADR, its status values, and how to record changes instead of overwriting. -->

# ADRs (architecture decision records)

An ADR (Architecture Decision Record) captures a durable decision's context, the choice made, the alternatives
rejected, and the consequences in one document. Every decision goes into the [decision log](../01-decision-log.md)
first. Only some of them also get an ADR.

## When a decision log entry is enough, and when you need an ADR

| Decision log only | ADR too |
|---|---|
| The reason fits in a sentence or two | There were several alternatives, and why they were rejected will matter later |
| Easy to reverse | Expensive to reverse, such as a storage format, a boundary, or a security model |
| Affects one area | Affects several components or many future PRs |

When you write an ADR, keep a one-line summary and the ADR link in the decision log entry, and put the decision number
in the ADR.

## File name and format

- File name: `NNNN-short-title.md` (for example, `0002-trash-retention.md`). Numbers are four digits and go up one at a time.
- The first line, as in other docs, is `<!-- Summary: one-line summary -->`.
- Header list: `Status`, `Date`, `Decision` (decision log number), and, where they apply, `Amended` and `Superseded by`.
- Body sections: `Context`, `Decision`, `Consequences`, `Rejected alternatives`. Shorter is better. See [0001-example.md](0001-example.md).

## Status

| Status | Meaning |
|---|---|
| `Proposed` | A proposal. No authority. Do not implement from this ADR alone |
| `Accepted` | Accepted. In force now |
| `Amended` | Partly changed after acceptance. The changes are in the `Amended` line and the "Amendment history" at the end of the body |
| `Superseded` | Replaced by another ADR. Kept as a record only |

## Record changes

- Do not overwrite the body of an accepted ADR. Add what changed to an `Amended: <date>, D<n>` line in the header and to
  an "Amendment history" section at the end of the body, and change the status to `Amended`.
- If the decision changes entirely, write a new ADR, change the old ADR's status to `Superseded`, and add
  `Superseded by: ADR <new number>`.
- Typos and broken links can simply be fixed. Record only edits that change meaning.

Why: when what changed, when, and why is visible, nobody revives an old approach on the strength of an old sentence.
