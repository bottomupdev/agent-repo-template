<!-- Summary: Decisions made, why, and what not to do. Never deleted; superseded entries stay in place. -->

# Decision log

*Records what is already decided and why, so agents do not re-ask the same question or drift back to an approach that
was dropped. The owner makes decisions. Agents may record the owner's decisions but do not make decisions on their own.*

<!-- Template: italic text is guidance; "Example" blocks describe a fictional to-do app. Delete both once filled in. -->

## How to write entries

- Number entries `D1`, `D2`, … one at a time. Never reuse or skip a number. If two PRs claim the same number, the one
  that merges later renumbers.
- Entry format:

  ```markdown
  ### D<n>: <the decision in one sentence>

  - Date: YYYY-MM-DD
  - Source: <link to the issue, PR, or comment where the decision was recorded>
  - Decision: <what we decided to do>
  - Why: <the reason; one line on alternatives considered, if any>
  - Do not: <what we do not do because of this decision. This is the part agents break most often>
  ```

- **Never delete.** When a decision changes, write it under a new number and add one line directly below the old
  entry's title: `Superseded by: D<m> (YYYY-MM-DD)`. Leave the old body as it is. The reason for the change has to stay
  visible so nobody reverts to the old approach.
- If only part of a decision changes, write `Partly superseded by: D<m> — <the changed part>` on the old entry.
- To correct a factual error, do not rewrite the body either; add a dated correction line.
- If there were several alternatives and the context will still matter months later, write an [ADR](adr/README.md)
  too. Keep a one-line summary and the ADR link in the decision log.
- Point a question this settles in [02-open-questions.md](02-open-questions.md) to this number.

## Decisions

*Add new decisions at the bottom.*

---

**Example (a fictional to-do app)**

> ### D1: Delete tasks immediately
>
> Superseded by: D2 (2026-02-09)
>
> - Date: 2026-01-12
> - Source: [<owner>/<repo>#N](https://github.com/<owner>/<repo>/issues/N)
> - Decision: Pressing delete removes the task from storage right away.
> - Why: Keep the first version simple. Revisit undo if someone asks for it.
> - Do not: Do not add a delete confirmation dialog.
>
> ### D2: Keep deleted tasks in the trash for 7 days
>
> - Date: 2026-02-09
> - Source: [<owner>/<repo>#N](https://github.com/<owner>/<repo>/issues/N) — three requests from people who deleted by mistake
> - Decision: Deleting moves a task to the trash, and it is removed after 7 days. Tasks can be restored from the trash. Supersedes D1.
> - Why: A confirmation dialog costs an extra tap every time; a trash costs something only when you make a mistake.
> - Do not: Do not add a delete confirmation dialog (carried over from D1). Do not make the trash period a user setting.
>   If evidence that it is needed appears, handle it as a new decision.
