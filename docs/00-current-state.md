<!-- Summary: What we are building now and for whom, what works, what is missing, and what comes next. -->

# Current state

*This doc is a one-page summary of "now". There is no separate status or roadmap doc; update it here, in one place.
Git history and merged PRs remember the past. Keep only what is true now.*

<!-- Template: italic text is guidance; "Example" blocks describe a fictional to-do app. Delete both once filled in. -->

## What and for whom

*Two or three sentences: who, in what situation, gets what. If more background is needed, link to an ADR.*

## Scope and out of scope

*List what is in and what is out side by side. For anything out of scope, say whether it is "not yet" or "decided
against", and give the decision number for anything decided against.*

## What works now

*Write what users can actually do. Code existing is not the same as usable. Write anything not verified as "needs
verification".*

## Known gaps

*Known defects, missing pieces, and accepted risks. Link the tracking issue if there is one.*

## Next

*The next few things to do, in order. Link anything that needs the owner's answer to its question in
[02-open-questions.md](02-open-questions.md).*

## Milestones (optional)

*Keep only if you want to. Write the date and PR link only, not commit hashes (squash merges drop them). Fill in the
PR number after opening the PR, with one more commit on the same PR. Do not open a separate PR after merge just to edit
this doc.*

---

**Example (a fictional to-do app)**

> ## What and for whom
>
> An app for people who work alone to write down today's tasks and finish them, in a single browser tab. No sign-up needed.
>
> ## Scope and out of scope
>
> - In scope: add, complete, and delete tasks; due dates; trash.
> - Out of scope (decided against): server sync, accounts. Data stays in browser storage ([ADR 0001](adr/0001-example.md)).
> - Out of scope (not yet): recurring tasks, reminders.
>
> ## What works now
>
> - Add tasks, mark them done, sort by due date.
> - Deleted tasks stay in the trash for 7 days (D2). Automatic emptying after 7 days needs verification.
>
> ## Known gaps
>
> - When storage is full, a failed save is not shown on screen.
>
> ## Next
>
> 1. Save-failure notice.
> 2. Recurring tasks — needs a decision on how to express recurrence rules (Tasks Question 1).
>
> ## Milestones
>
> - 2026-01-20 First release — [<owner>/<repo>#N](https://github.com/<owner>/<repo>/pull/N)
