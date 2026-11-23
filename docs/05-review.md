<!-- Summary: Which reviewers run on which PR (routing table, tiers) and how findings are handled. -->

# Review routing

Decides which reviewers run on which PR. Reviewers are the read-only skills in [.claude/skills/](../.claude/skills/README.md),
and what counts as a finding follows [finding-admission.md](../.claude/skills/finding-admission.md). The routing table
lives here and only here; [AGENTS.md](../AGENTS.md) and the PR template only point to this doc.

<!-- Template: italic text is guidance; "Example" blocks describe a fictional to-do app. Delete both once filled in. -->

## Routing table

*Say when each reviewer runs. Start from the generic conditions below and replace them with this repository's real
paths (for example, "API routes" → `src/server/routes/**`). Delete the row of any reviewer that will never run, and
remove it from the PR template too. Reviewer names are skill folder names; do not rename them.*

| Reviewer | When |
|---|---|
| `engineering-reviewer` | Always |
| `test-coverage-reviewer` | Production code or a critical surface changed, or tests changed |
| `backend-api-reviewer` | API routes, authentication or authorization, data-access policies, the repository layer, storage adapters, public token or link access, or resource lifecycle transitions changed |
| `security-reviewer` | Privacy, security, authentication, access policies, tokens, signed URLs, storage, logging, analytics, or public links changed. A boundary that user data crosses (client/server, runtime) changed. The CI or agent workflow surface (workflow permissions and triggers, agent tool allow-lists, prompt-injection paths, CI secrets) changed |
| `docs-consistency-reviewer` | Docs, decision records, scope, or architecture descriptions changed. A code change needs a docs update. AGENTS.md, reviewer skills, or CI workflows changed |
| `protocol-walkthrough-reviewer` | AGENTS.md, `.claude/skills/**`, `.github/workflows/**`, or runbooks — procedures and rules that an agent or runner executes as written — changed |

`ai-pr-owner-review` is not a gate. Run it when the owner wants an AI-written PR explained.

## What engineering-reviewer checks separately

`engineering-reviewer` always runs, and along with code quality it re-checks two sections of the PR description against
the actual diff.

- **Reuse check**: if it says "none", whether the PR really touches none of the high-risk families in
  [04-architecture.md](04-architecture.md). If it touches some, whether a better-suited source already exists, whether
  there are now two sources, whether a retained duplicate has a real boundary, and whether a replaced old path was
  deleted or a follow-up issue linked. A missing section is `Must fix` in itself.
- **Architecture admission**: whether `Trigger: none` is right; if there is a trigger, whether the Route A path is
  really reachable, whether the Route B evidence is reproducible, and whether lifecycle hygiene is recorded. The
  standard is section 3 of [AGENTS.md](../AGENTS.md).
- **Tier and reviewers**: whether the tier the PR chose and the reviewers it ran match the actual diff.

## Tiers

*Pick one tier per PR, then add every row of the routing table whose condition matches. Do not add unrelated reviewers
just because the tier is broad. Say what counts as a "critical surface" in this repository. Tier names are identifiers;
keep them as they are.*

| Tier | Classification | Reviewers to run |
|---|---|---|
| `low-risk docs/tests` | Only docs or tests changed. Product behavior, critical surfaces, procedures and workflows, and permissions are unchanged | `engineering-reviewer`. `test-coverage-reviewer` if tests changed, `docs-consistency-reviewer` if docs changed |
| `ordinary code` | Production code with regression risk. No critical surface | `engineering-reviewer`, `test-coverage-reviewer`, and every matching reviewer in the routing table |
| `critical surfaces` | Privacy, authentication, storage, deletion, migrations, permissions, concurrency and recovery, cross-platform contracts, agent and CI authority | Same as `ordinary code` |

## Recording it in the PR description

*Record the tier and the reviewers in the PR description's `## Review`. List every reviewer in the table once, and add
`N/A - reason` to any reviewer you did not run.*

## Handling findings

- Run each reviewer separately (one subagent each) and merge findings with the same root cause into one. How to merge
  them, and the request wording, are in [.claude/skills/README.md](../.claude/skills/README.md).
- Record a disposition for each admitted finding: **fixed** (commit), **tracked** (issue), **risk accepted** (who,
  why), or **declined** (one line on which test it fails).
- Do not merge with a `Must fix` that has no disposition. A `Can track after merge` finding gets an issue, or a reason
  for moving on.
- A finding's severity is evidence, not permission to widen scope. Escalate a finding that would need wider scope to fix
  to an owner decision.
- After fixes, re-review only what changed. One clean round ends it; the ceiling is two consecutive rounds with no
  `Must fix`.
- A clean review does not prove the PR is correct. Leave anything not verified in the PR's `## Not verified`.

## Changing this table

- When paths change or reviewers are added or removed, update this doc and the [PR template](../.github/pull_request_template.md)
  in the same PR.
- Do not drop `engineering-reviewer`, `test-coverage-reviewer`, or `security-reviewer` lightly. To drop one, give the
  reason in the PR.
- If general reviewers keep missing a boundary (mobile/native, payments, migrations, and so on), add a domain reviewer.
  How to build one is in [.claude/skills/README.md](../.claude/skills/README.md). Do not add one after a single miss.

---

**Example (a fictional to-do app)**

> | Reviewer | When |
> |---|---|
> | `engineering-reviewer` | Always |
> | `test-coverage-reviewer` | `apps/web/src/**` or `**/*.test.ts` changed |
> | `security-reviewer` | Browser storage reads or writes (`apps/web/src/storage/**`) changed, an external script was added, or `.github/workflows/**` changed |
> | `docs-consistency-reviewer` | `docs/**` or `AGENTS.md` changed, or a change matches the table in [docs/README.md](README.md#which-doc-to-update-when) |
> | `protocol-walkthrough-reviewer` | `AGENTS.md`, `.claude/skills/**`, or `.github/workflows/**` changed |
>
> There is no server ([ADR 0001](adr/0001-example.md)), so `backend-api-reviewer` was removed.
>
> A PR that adds a `priority` field to tasks and moves existing tasks stored on the device to the new format is
> `critical surfaces` (migration of stored data). The storage format changes, so it has an architecture admission trigger.
>
> ```markdown
> ## Review
>
> Tier: critical surfaces
>
> - [x] engineering-reviewer
> - [x] test-coverage-reviewer
> - [x] security-reviewer
> - [x] docs-consistency-reviewer
> - [ ] protocol-walkthrough-reviewer N/A - no procedure or workflow change
>
> Must fix dispositions: old data deleted if the move fails partway — fixed `commit`
> Left open: polish the migration-failure message — tracked issue #<number>
> ```
