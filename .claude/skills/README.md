# Reviewer skills

Read-only AI code-review skills for Claude Code and Codex. Each skill is one
specialist reviewer with a narrow domain, a checklist, and a shared rule for
what counts as a finding. Run one, or run several on the same PR and merge
their output.

| Skill | Reviews |
| --- | --- |
| `engineering-reviewer` | Architecture, reliability, efficiency, scalability, maintainability, merge readiness. Also checks for speculative mechanisms and duplicated sources of truth. |
| `test-coverage-reviewer` | Whether tests would actually catch a regression in the changed behavior. |
| `security-reviewer` | Concrete security and privacy risks, including CI workflows and agent tool permissions. |
| `backend-api-reviewer` | Auth, authorization, ownership, data-access policies, lifecycle transitions, storage and signed URLs, public tokens, error semantics. |
| `docs-consistency-reviewer` | Whether docs, decision records, scope, and architecture claims match what the code does. |
| `protocol-walkthrough-reviewer` | Agent prompts, runbooks, and CI workflows, executed literally, one persona at a time, to find where they break. |
| `ai-pr-owner-review` | Not a gate. Explains an AI-authored PR to its owner and separates blockers from follow-ups, with optional teach-back questions. |

## Layout

```text
.claude/skills/<name>/SKILL.md            the skill (frontmatter: name, description)
.claude/skills/<name>/agents/openai.yaml  Codex interface metadata (display name, default prompt)
.claude/skills/finding-admission.md       the shared admission policy (canonical copy)
.claude/skills/README.md                  this file
```

Which reviewer runs on which PR is project policy, not skill policy. In a repo
made from this template it lives in [docs/05-review.md](../../docs/05-review.md).
Without that file, start from [Default routing](#default-routing) below.

## Review philosophy

- **Read-only.** A reviewer reads and reports. It does not edit files, commit,
  or post comments unless you explicitly ask. As a subagent, it returns
  proposed comment text to whoever publishes.
- **Findings first.** Output starts with findings ordered by severity. Summary
  and "what I inspected" notes come after, and stay short.
- **Two axes on every finding.** Severity (`Must fix before merge` or
  `Can track after merge`) says what happens if it is real. Confidence
  (`High`, `Medium`, `Low`) says how sure the reviewer is.
- **Construct the failure.** The engineering and security reviewers try to
  build the breaking sequence (a race, a crash between two writes, an abuse
  case, an injection path) and report it as evidence.
- **Execute the protocol.** When agents run a project, prompts and workflow
  files are code. `protocol-walkthrough-reviewer` reads them as a
  hostile-literal interpreter and reports the persona and step where they break.
- **Delta-focused re-review.** A re-review checks prior findings and the new
  delta, not the whole PR again.

## Finding admission, in short

The full policy is [finding-admission.md](finding-admission.md); each
`SKILL.md` restates the minimum so a single skill works on its own.

- A finding must meet one of eight tests: it violates an accepted requirement
  or invariant, creates a reproducible failure path, creates a security,
  privacy, authorization, data-loss or irreversible-lifecycle risk, blocks a
  binding release gate, regresses the base branch, is the second occurrence of
  the same operational failure, leaves a changed high-risk behavior without a
  load-bearing test, or misses an accepted measurable target.
- It carries a traced failure sequence, evidence or a falsification step,
  severity, confidence, and the smallest proportionate fix.
- Low-confidence hypotheses are admitted only for security, privacy, data
  loss, irreversible authority changes, and concurrent-writer violations.
- Style, naming, speculative scaling, and "more tests would be nice" go in a
  separate list of observations with no obligation attached.
- An empty finding list is a valid result.
- Every admitted finding ends in one disposition: fixed at a commit, tracked in
  a named issue, risk accepted by someone with authority, or declined with a
  reason.

## Install

### Claude Code

In a repo made from this template, nothing to do: Claude Code discovers
project skills under `.claude/skills/` automatically. It ignores
`agents/openai.yaml`, `README.md`, and `finding-admission.md`.

To make them available in every project, copy them to your user directory:

```sh
mkdir -p ~/.claude/skills
cp -r .claude/skills/*/ ~/.claude/skills/
```

### Codex

Sync the skill folders into the Codex skills directory (`$CODEX_HOME/skills`,
default `~/.codex/skills`) and restart Codex:

```sh
mkdir -p ~/.codex/skills
cp -r .claude/skills/*/ ~/.codex/skills/
```

Re-run after you edit a skill. Or skip installing: any agent can follow a skill
by reading its file ("read `.claude/skills/security-reviewer/SKILL.md` and
follow it").

### Copy into an existing repo

The skills have no dependency on the rest of this template.

```sh
mkdir -p .claude/skills
cp -r <path to this template>/.claude/skills/. .claude/skills/
```

Then add a routing table to your repo (copy [Default routing](#default-routing)
or `docs/05-review.md` and edit it), and a line in your agent instructions
(`AGENTS.md`, `CLAUDE.md`) saying when to run the reviewers.

## Invoke

Claude Code:

```text
/engineering-reviewer review PR 42 at head <sha>
```

or in plain language ("run a security review of this branch"). Codex:

```text
Use $security-reviewer to review this PR for security and privacy risks only.
```

Always give the exact target: PR URL or number, branch, or diff range, and
ideally the head SHA, so a stale checkout does not skew the result.

## Running several reviewers on one PR

1. **Pick a tier and route.** Choose one tier, then take the union of matching
   routing rows. Do not add unrelated reviewers just because the tier is broad.
2. **Run them independently.** Start each reviewer as its own subagent with the
   same target and head SHA, so one reviewer's framing does not anchor another.
   They are read-only, so they can run in parallel. In Claude Code: "Run
   engineering-reviewer and security-reviewer on this branch as separate
   subagents, then merge the findings."
3. **Merge the findings.**
   - Group findings with the same root cause, even when worded differently.
     Keep one stable title per group and note which reviewers raised it.
   - Keep the highest severity in the group. For confidence, keep the strongest
     evidence (a constructed breaking sequence beats a pattern match).
   - Re-apply the admission policy to the merged list. Items that fail it,
     including items from hosted review bots, are recorded once as not
     admitted with a one-line reason.
   - Give each admitted finding one disposition and record it in the PR body
     (the `## Review` section of the PR template).
4. **Re-review the delta.** After fixes, send each reviewer the previous head,
   the current head, and the findings addressed (prompt below). A reviewer newly
   routed at this head, or one whose PR base changed, gives a full review.
5. **Stop.** One clean round ends review at that head. As a ceiling, two
   consecutive rounds with no `Must fix before merge` against repository files
   means the PR is eligible to ship at that head.

## Default routing

A default for when you copy only the skills. If `docs/05-review.md` exists, it
is the source of truth. `always` runs on every implementation PR; other rows
are conditional.

| Reviewer | Route when |
| --- | --- |
| `engineering-reviewer` | always |
| `test-coverage-reviewer` | production code or a critical surface changed; or tests changed |
| `backend-api-reviewer` | API routes, auth, authorization, data-access policies, repositories, storage adapters, public token or link access, or resource lifecycle transitions changed |
| `security-reviewer` | privacy, security, auth, access policies, tokens, signed URLs, storage, logging, analytics, or public links changed; a boundary carrying user data changed; or a CI or agent-workflow surface changed |
| `docs-consistency-reviewer` | docs, decision records, scope, or architecture claims changed; a code change required a docs update; or agent prompts, skills, or CI workflows changed |
| `protocol-walkthrough-reviewer` | agent prompts, skill files, runbooks, CI workflows, or any other procedure an agent or runner executes changed |

| Tier | Classification | Required review |
| --- | --- | --- |
| `low-risk docs/tests` | Docs or tests only; no behavior, critical surface, procedure, or permission change | `engineering-reviewer`; `test-coverage-reviewer` if tests changed; `docs-consistency-reviewer` if docs changed |
| `ordinary code` | Production code with regression risk, no critical surface | `engineering-reviewer`, `test-coverage-reviewer`, every matched specialist |
| `critical surfaces` | Privacy, auth, storage, deletion, migrations, permissions, concurrency or recovery, cross-platform contracts, agent or CI authority | same as `ordinary code` |

`ai-pr-owner-review` is not a gate; run it when the owner wants a PR explained.
The engineering reviewer checks that the declared tier and routing match the
diff. Keep one routing table per repo as the source of truth; other files point
to it. Treat removing `engineering-reviewer`, `test-coverage-reviewer`, or
`security-reviewer` as a deliberate, reviewed change.

## Request prompts

```text
Review <PR URL, branch, or diff range>.

Head SHA: <full SHA>
Linked issue or requirement: <URL or "none">
Review tier: <tier>

Read first: the repository's agent instructions (AGENTS.md, CLAUDE.md), the
exact current head and diff, the linked issue and docs it names, and the named
reviewer skill.

Keep the review read-only. Do not modify files or post comments.
Output findings first, each with a stable short title, admitted under
.claude/skills/finding-admission.md. List out-of-policy items separately as
non-finding observations. Include suggested PR comment text for each finding.
```

Re-review:

```text
Re-review <PR URL>.

Previously reviewed head: <full SHA, or "none - newly required">
Current head: <full SHA>
Base change since that review: <none, or old base -> new base>
Exact delta range: <prev SHA..current SHA, or "N/A">
Addressed findings:
1. <finding title>
Invariants materially affected by this delta:
- <invariant>

Classify each prior finding as Resolved, Still valid, or Needs follow-up.
Verify the delta and the named invariants at the current head; do not restart
a whole-PR discovery pass. Admit a new finding only for a fix-introduced
regression, a corrected review input still in violation, a previously missed
critical invariant with concrete evidence, or a security, privacy, or
data-loss risk exposed by this head. If the previous head is "none - newly
required", or the base changed, give a full review of the current diff.
```

## Adding a reviewer

Add a domain reviewer when a boundary keeps slipping past the general ones
(mobile/native code, payments, migrations). Copy an existing skill folder, keep
its `## Formal Finding Admission` block, write a new checklist and a
one-paragraph domain application, and add a row to your routing table. Remove
reviewers you never route to.

## Limitations

- These are prompts, not tools. Quality depends on the model, the context the
  reviewer can reach, and how precisely you name the target.
- Reviewers can be wrong in both directions. A clean result does not prove a
  correct PR.
- Several checklists lean toward web backends and agent-run repositories,
  because that is where they were used. Adapt the examples to your stack.
- PR reviews assume access to PR metadata (a GitHub connector or the `gh` CLI).
  Without that, give the diff directly.
- Nothing here enforces routing in CI. That is up to your process.
