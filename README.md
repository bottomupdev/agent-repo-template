# Repository template for working with AI agents

A GitHub template for repositories where AI agents such as Claude Code and Codex write most of the code. It has three parts.

1. **Docs the agents read and work from** — shared rules (`AGENTS.md`), a `docs/` folder that keeps current state,
   decisions, questions, terms, architecture, and review apart, and a PR template.
2. **Read-only reviewers that check PRs against those docs** — the reviewer skills in `.claude/skills/`. They do not
   edit files, and they raise only findings that pass a set bar.
3. **Principles that tie the two together** — reason exhaustively and implement minimally, search before you write,
   and admit structure only with evidence.

[한국어판](https://github.com/bottomupdev/agent-repo-template-ko)

> This README describes the template. Once you create a repository from it, replace this with your project's README
> (install, run, deploy).

## Why

Agents forget context when a conversation ends, and they can read only so much at once. To avoid explaining everything
from scratch each time, three things must be clear inside the repository.

- **What we are building** — for whom, and what we are not doing (`docs/00-current-state.md`)
- **Which decisions are in force** — what we chose, why, and what not to do (`docs/01-decision-log.md`)
- **How we work** — task size, tests, doc updates, review, checks before pushing (`AGENTS.md`)

Agents also tend to overbuild. They rewrite functions that already exist, bring in abstractions with no caller and
version branches "for later", and add a new check for every rare failure. The principles in `AGENTS.md` are written to
push back on this. It collects the rules and tools the author refined while working with Claude Code and Codex every day.

## Layout

```
AGENTS.md                         Shared agent rules. Highest authority
CLAUDE.md                         Pointer that imports AGENTS.md
docs/
  README.md                       Guide by question, authority and evidence, which doc to update when, writing rules
  00-current-state.md             Now: what and for whom, scope, what works, gaps, next
  01-decision-log.md              Decisions (D1, D2, …). Never deleted; marked "superseded"
  02-open-questions.md            What is not decided. Settled questions stay as tombstones
  03-glossary.md                  Terms, synonyms not to use, numbering schemes
  04-architecture.md              Components and boundaries, data flow, source locations of high-risk primitives
  05-review.md                    Review routing: which reviewers for which PR
  adr/README.md                   When and how to write an ADR
  adr/0001-example.md             ADR example
  growing.md                      What to add as you grow, and the signal that justifies it
.claude/skills/
  README.md                       Reviewer overview, install, merging several reviewers' results
  finding-admission.md            What counts as a finding
  <reviewer>/SKILL.md             Reviewer skills
.github/pull_request_template.md  PR template
.github/workflows/scrub.yml       Banned-term check CI
scripts/scrub.mjs                 Banned-term check
.gitattributes                    Line-ending rule (* text=auto eol=lf)
```

Each doc in `docs/` starts with a one-line summary (`<!-- Summary: … -->`), and has italic guidance and examples drawn
from a fictional to-do app. Delete the guidance and examples once you fill a doc in.

`.gitattributes` is also an example of a principle: settings that affect behavior are versioned in the repository, not
left to each machine's configuration. If line endings differ between machines, you get tests that pass locally and fail
in CI (or the other way around).

| Reviewer | Looks at |
|---|---|
| `engineering-reviewer` | Architecture, reliability, efficiency, maintainability, merge readiness. Re-checks the PR's reuse check and architecture admission against the diff |
| `test-coverage-reviewer` | Whether tests would actually catch a regression in the changed behavior |
| `security-reviewer` | Security and privacy risks, including CI workflows and agent tool permissions |
| `backend-api-reviewer` | Authentication and authorization, ownership, data-access policies, lifecycle, storage, public tokens, error semantics |
| `docs-consistency-reviewer` | Whether docs, decision records, scope, and architecture descriptions match the code |
| `protocol-walkthrough-reviewer` | Follows agent rules, runbooks, and CI workflows literally to find where they break |
| `ai-pr-owner-review` | Not a gate. Explains an AI-written PR to its owner and separates merge blockers from later work |

## Quick start

1. On GitHub, create a new repository with **Use this template**.
2. Fill in the `<angle-bracket>` placeholders in `AGENTS.md` and `.github/pull_request_template.md` (project name,
   build and test commands).
3. **Fill in `docs/00-current-state.md` and `docs/01-decision-log.md` first.** What you are building, what you are not,
   and what is already decided make the biggest difference to an agent. You can leave the rest empty to start.
4. Tell your agent "Read `AGENTS.md` and start." Claude Code reads it automatically through `CLAUDE.md`.
5. Before a PR goes up, have the agent run the reviewers per `docs/05-review.md` and record the results in the PR
   description. This is already a rule in `AGENTS.md`. To run one yourself in Claude Code, call it like
   `/engineering-reviewer review this branch`.

After that, let the agent update docs on every PR following the table in `docs/README.md`. Empty docs fill in that way too.

## Using only the reviewers

The reviewer skills work without the rest of the template. Copy `.claude/skills/` into an existing repository.

```bash
mkdir -p .claude/skills
cp -r path/to/this-template/.claude/skills/. .claude/skills/
```

- **Claude Code**: finds the project's `.claude/skills/` automatically. To use them in every project, copy them to `~/.claude/skills/`.
- **Codex**: copy the skill folders to `~/.codex/skills/` and restart. Or skip installing and say
  "Read `.claude/skills/security-reviewer/SKILL.md` and follow it."
- For when to run which reviewer, start from the default routing in [.claude/skills/README.md](.claude/skills/README.md) and adapt it.

## Banned-term check and SCRUB_TERMS

Keeps pre-launch project names, customer names, and internal codenames out of a public repository.

**The banned-term list is not kept in the repository.** Writing the list into a public repository would itself be the
leak. The list lives only in the GitHub Actions secret `SCRUB_TERMS` (one comma-separated line).

- `scripts/scrub.mjs` checks every line of the tracked text files from `git ls-files` (case-insensitive substring match).
- On a hit it prints a number instead of the term, like `file:line: banned term #N`, and fails. The term does not end up in CI logs either.
- Without `SCRUB_TERMS` (a fork, or the secret not set) it skips and passes.

Setup: under **Settings → Secrets and variables → Actions → New repository secret**, create one named `SCRUB_TERMS`,
or run `gh secret set SCRUB_TERMS`. To make it block merges, mark `scrub` as a required check in the branch protection rules.

```bash
# Run locally. Keep the list in a file outside the repository and read it in (so it stays out of shell history).
SCRUB_TERMS="$(cat ~/.config/scrub-terms)" node scripts/scrub.mjs
```

Limits: it checks tracked files only (new files after `git add`). It does not see commit messages or PR descriptions,
so `AGENTS.md` states that as a separate rule. A term already committed stays in git history even after you delete it.
Because it matches substrings, short or common words produce false positives.

## As you grow

Start small. Things like a conflicts and reconciliation record, a project journal, an archive, context routing,
duplicate checks, and ratchet gates are added when the signal that you need them appears. The signals and smallest
forms are in [docs/growing.md](docs/growing.md).

## FAQ

**Can I use Claude Code and Codex together?**
Yes. Both follow the rules in `AGENTS.md`. Codex reads `AGENTS.md` directly; Claude Code reads `CLAUDE.md`, which
imports the same rules with `@AGENTS.md`. Rules are written in one place only, `AGENTS.md`.

**Isn't this too many docs?**
Delete the docs you do not need and update the tables in `docs/README.md`. But do not pile everything into one doc.
Agents need to be able to pick out only what they need.

**What if I want to change a decision?**
Do not delete the old entry. Add `Superseded by: D<m>` and write the new decision under a new number. Keeping why it
changed is what stops people from drifting back to the old way. When an agent thinks a decision needs changing, it asks
first instead of implementing.

**Does CI enforce the reviews?**
No. Reviewers are prompts; agents and people run them. The PR description's `## Review` records what was run.
Reviewers can be wrong, and a clean result does not prove the PR is correct.

**Why record "needs verification" separately?**
Agents readily write that something is done when they have not tried it. Keeping what was not verified apart from what
is done tells the next person what they can trust.

## Status

Published as-is. No support is promised. Issues and PRs may be read, but no reply or merge is promised. Forks are welcome.

## License

MIT. See [LICENSE](LICENSE).
