<!-- Summary: What is not decided yet. Numbered per area; settled questions stay as tombstones pointing to a decision number. -->

# Open questions

*Records what the owner has not decided yet. Agents do not guess the answer to a question here and implement it. Pause
only the work that depends on the question, and continue the rest.*

<!-- Template: italic text is guidance; "Example" blocks describe a fictional to-do app. Delete both once filled in. -->

## How to write questions

- Group questions by area (features, storage, operations, and so on) and number them within each area. Other docs refer
  to them as `<Area> Question <number>` (for example, `Tasks Question 2`). Do not add `#`.
- For each question, write: what needs deciding, the options, and the work blocked on the answer.
- **Never renumber.** Do not delete a settled question; leave a tombstone pointing to the decision number. The numbers
  that old docs and PRs point to must keep meaning the same question.
- Tombstone format: `~~<Area> Question <number>: <question>~~ → decided in D<n> (YYYY-MM-DD)`
- If a question stops mattering (its premise is gone), write `→ no longer asked: <reason>` on the tombstone.

## Questions

*Write them under a heading per area.*

---

**Example (a fictional to-do app)**

> ### Tasks
>
> 1. **How should recurring tasks be expressed?**
>    Options: (a) a few fixed rules such as "every Monday", (b) a calendar-standard (RRULE) string.
>    Blocked work: the whole recurring-tasks feature. The storage format changes, so it needs architecture admission.
> 2. ~~**Should deletion be undoable?**~~ → decided in D2 (2026-02-09)
>
> ### Storage
>
> 1. **When storage is full, what should be cleared first?**
>    Options: empty the trash first, ask the user.
>    Blocked work: the wording of the save-failure notice.
