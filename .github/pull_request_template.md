## What and why

<!-- Briefly: what changed and why. Name any related decision (D<n>), question, or issue. If this closes an issue, write "Closes #<issue number>". -->

## Docs

<!-- Keep one of the two lines. The table is "Which doc to update when" in docs/README.md. -->

docs: updated docs (for example, 00-current-state, 04-architecture)
docs: none (reason)

## Reuse check

<!-- AGENTS.md section 4. The high-risk families are listed in docs/04-architecture.md. If no family was touched, keep only the first line as "none — reason". -->

Family: <families touched | none — reason>
Searched for: <search terms>
Paths inspected: <paths and symbols of existing implementations>
Decision: <reuse | extend | replace (old path deleted, or follow-up issue #<number> with exit condition) | new — the concrete boundary where reuse was wrong>

## Architecture admission

<!-- AGENTS.md section 3. Triggers: durable state, format or protocol version, runtime compatibility branch, platform or provider abstraction. For a removal or simplification only, write Trigger: none. -->

Trigger: <every trigger that applies | none>
Route: <Route A | Route B | N/A>
Proof: <Route A: the decision or ADR cited and the reachable producer-to-consumer path / Route B: the simpler implementation, the target, and reproducible evidence>
Lifecycle: <migration, deletion, rebuild, downgrade, or removal of the compatibility branch | N/A — reason>

## Review

<!-- Follow the routing in docs/05-review.md. List every reviewer in the table once; for any reviewer you did not run, add "N/A - reason". -->

Tier: <low-risk docs/tests | ordinary code | critical surfaces>

- [ ] engineering-reviewer
- [ ] test-coverage-reviewer
- [ ] backend-api-reviewer
- [ ] security-reviewer
- [ ] docs-consistency-reviewer
- [ ] protocol-walkthrough-reviewer

Must fix dispositions: <finding title — fixed `commit` / tracked issue #<number> / risk accepted (by whom, why) / declined (reason)>. If none, "none".
Left open: <Can track after merge findings and observations not fixed in this PR, with reasons>. If none, "none".

## Verified

<!-- What you verified and how: commands, screens, links. -->

- [ ] `node scripts/scrub.mjs`
- [ ] `<build command>`
- [ ] `<test command>`

## Not verified

<!-- What you could not verify, and why. If nothing, "none". -->
