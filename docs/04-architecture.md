<!-- Summary: Components and boundaries, data flow, and the high-risk primitive families with each family's source location. -->

# Architecture

*Show at a glance how the code is divided now. This is the present, not a plan. Plans go in "Next" in
[00-current-state.md](00-current-state.md); decisions go in [01-decision-log.md](01-decision-log.md).*

<!-- Template: italic text is guidance; "Example" blocks describe a fictional to-do app. Delete both once filled in. -->

## Components and boundaries

*One line per app, service, or package: what it owns and what it does not. "What it does not own" is the boundary.
Say where a call that crosses a boundary must go through.*

## Data flow

*Where the main data is created, what it passes through, where it is stored, and when it is deleted. A short list or
one diagram is enough.*

## High-risk primitive families

*The reuse check in [AGENTS.md](../AGENTS.md) uses this table. Group primitives that silently drift apart when copied
into families, and choose one source of truth for each family. A PR that touches a family in this table finds the
source first and records it in the PR's `## Reuse check`. For a family with no source yet, write "none"; the first PR
that creates one fills in the location.*

*Common families: serialization and storage formats, ID generation, time parsing and formatting, error normalization,
input validation, state transitions, storage path and key rules, security and crypto primitives (tokens, hashes,
encodings), contracts shared by client and server. Delete families this repository does not have.*

| Family | Source location | Notes |
|---|---|---|

---

**Example (a fictional to-do app)**

> ## Components and boundaries
>
> - `apps/web` — screens and user input. Never reads or writes storage directly; calls only `src/storage/`.
> - `apps/web/src/storage/` — reading and writing browser storage, and the storage format. Knows nothing about screen state.
> - `apps/web/src/model/` — task rules (validation, state transitions). Knows neither screens nor storage.
> - There is no server ([ADR 0001](adr/0001-example.md)).
>
> ## Data flow
>
> Input → `model/validate` → `model/transitions` → serialized by `storage/codec` → browser storage.
> Tasks in the trash are deleted when the app opens, if 7 days have passed (D2).
>
> ## High-risk primitive families
>
> | Family | Source location | Notes |
> |---|---|---|
> | Serialization and storage format | `apps/web/src/storage/codec.ts` | Changing the format needs architecture admission |
> | ID generation | `apps/web/src/lib/id.ts` | Do not call `crypto.randomUUID()` directly anywhere else |
> | Date parsing and formatting | `apps/web/src/lib/date.ts` | Due dates are dates only, in the device time zone |
> | Input validation | `apps/web/src/model/validate.ts` | |
> | State transitions | `apps/web/src/model/transitions.ts` | open → done → trash → deleted |
> | Storage key rules | `apps/web/src/storage/keys.ts` | |
> | Error normalization | none | The first PR that creates one fills this in |
