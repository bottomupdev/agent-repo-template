<!-- Summary: (Example) The to-do app stores data only in browser storage, with no server. -->

# ADR 0001: Store data only in browser storage, with no server

<!-- Template: a format example written for a fictional to-do app. When you write your first ADR, delete this file and update the docs that link to it. -->

- Status: Amended
- Date: 2026-01-08
- Decision: D<n> (not included in the decision log example)
- Amended: 2026-02-09, D2 — trash retention counted toward storage usage

## Context

The first users work alone, on one device. Using the app right away without signing up matters most. A server would
bring accounts, authentication, backups, and personal-data handling along with it.

## Decision

Tasks live only in browser storage. No server, accounts, or sync. The storage format is handled in one place only,
`apps/web/src/storage/codec.ts`.

## Consequences

- Usable right away with no sign-up. No server to operate.
- Tasks do not follow you to a new device. Clearing browser data deletes them too.
- Storage space is limited. What happens when it is full is an open question (Storage Question 1).

## Rejected alternatives

- **Server and accounts**: nobody has asked for multi-device use yet. If evidence appears, handle it as a new decision.
- **Export and import files as the default storage**: requires pressing save every time, which makes the first-use flow heavy.

## Amendment history

- 2026-02-09 (D2): deleted tasks now stay in the trash for 7 days, so the trash counts toward storage usage. Everything else is unchanged.
