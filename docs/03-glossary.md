<!-- Summary: Agreed terms, synonyms not to use, and how the numbering schemes differ. -->

# Glossary

*When the same thing goes by several names, agents treat them as different things and implement them separately. Pick
one name to use in code, UI text, and docs, and list the synonyms not to use.*

<!-- Template: italic text is guidance; "Example" blocks describe a fictional to-do app. Delete both once filled in. -->

## Terms

*Start with one table. If the code name and the on-screen name differ, list both.*

| Term | Meaning | Code name | Do not use |
|---|---|---|---|

## Numbering schemes

This repository has several kinds of numbers. Always say which kind, so they do not get mixed up.

| Written as | Refers to | Lives in |
|---|---|---|
| `D12` | Decision | [01-decision-log.md](01-decision-log.md) |
| `Tasks Question 3` (`<Area> Question <number>`) | Open question (a tombstone once settled) | [02-open-questions.md](02-open-questions.md) |
| `ADR 0001` | Architecture decision record | [adr/](adr/README.md) |
| `issue #<number>`, `PR #<number>`, `<owner>/<repo>#<number>` | GitHub issue or PR | GitHub |
| `item 3`, `step 3` | List or procedure number within one doc | That doc |

- Use `#<digits>` only for GitHub issues and PRs. GitHub turns `#<digits>` in issue bodies, PR descriptions, and commit
  messages into a link to this repository's issue or PR. On a decision or question, it links to an unrelated issue.
- Never reuse a number. Settled questions and superseded decisions keep their numbers.

---

**Example (a fictional to-do app)**

> | Term | Meaning | Code name | Do not use |
> |---|---|---|---|
> | Task | A one-line piece of work the user wrote down. Has a title, due date, and status | `Task` | todo, item, card |
> | Done | The status of a task marked as finished. Moves to the bottom of the list | `status: "done"` | checked, closed |
> | Trash | Where deleted tasks stay for 7 days (D2) | `trash` | bin, archive |
> | Due date | A date only, with no time. Interpreted in the user's device time zone | `dueDate` | deadline |
>
> We do not say "archive". It sounds like a place for keeping things on purpose, which contradicts the fact that items
> are removed after 7 days.
