# Issue & Contribution Conventions (portable)

> Canonical source: **`ima-jin/conventions`**. Consumed by `imajin-ai`, `imajin-app-template`, and every app that
> adopts the template. This is the **standalone-repo** subset — it deliberately excludes the monorepo fork-and-PR model
> and the org roadmap board (those are `imajin-ai`/org-specific). Apps import this as AGENTS.md **§7**.

## Labels

The label vocabulary is executable state, not prose — seed it with `scripts/init-taxonomy.sh <owner/repo>` (idempotent;
re-runnable). App repos get the **universal** set only; `imajin-ai` also carries the **platform** domain set. See
`taxonomy.json`.

Use labels for **type/topic**, not lifecycle status. Status/priority belong on a project board where one exists — not
as labels (avoids the drift that had `active`/`blocked`/`parked` + `priority:*` duplicating board fields).

## Issue lifecycle

- **`Closes #N` / `Fixes #N` in a PR is the only thing that auto-closes an issue.** `Phase N — #N:` or a body mention
  closes **nothing** — it just links. Don't rely on prose to close tickets.
- **Don't close-and-icebox real ideas.** A genuine idea that isn't being worked now stays **open** (shelved, no clock),
  not closed. Closing is for *done* or *genuinely dead/superseded*. ("Icebox" as a concept is rejected — shelved ≠
  abandoned.)
- **When you complete an issue** (fix committed + pushed), **close it** — `gh issue close <n>` or `Closes #N` in the
  commit/PR. Don't leave done work open.

## Issue relationships — use GitHub's NATIVE types, not body checklists

For epics/children and blocking, use first-class relationship types (GraphQL-only; `gh` has no CLI subcommand yet):

- **Sub-issues** (`addSubIssue`) — real parent/child hierarchy with a roll-up progress bar. Use for epic → children,
  **not** `- [ ]` body checklists.
- **Blocked-by / blocking** (`addBlockedBy`) — native dependency badge. Use instead of a `blocked` label or a body note.
  The arg is `blockingIssueId` (the issue doing the blocking), **not** `blockedByIssueId`.

```bash
# node IDs
gh api graphql -f query='{repository(owner:"OWNER",name:"REPO"){issue(number:N){id}}}' -q '.data.repository.issue.id'
# make CHILD a sub-issue of PARENT
gh api graphql -H "GraphQL-Features: sub_issues" \
  -f query='mutation{addSubIssue(input:{issueId:"<PARENT_ID>",subIssueId:"<CHILD_ID>"}){subIssue{number}}}'
# mark ISSUE blocked-by BLOCKER
gh api graphql -H "GraphQL-Features: issue_dependencies" \
  -f query='mutation{addBlockedBy(input:{issueId:"<ISSUE_ID>",blockingIssueId:"<BLOCKER_ID>"}){issue{number}}}'
```

Both mutations require their `GraphQL-Features` header or they error.

## Writing issue / PR bodies — ALWAYS file-based

**Never pass an issue or PR body as an inline shell string** (`--body "..."`, `--title "..."` with body content,
or a heredoc that the shell expands). Write the body to a temp `.md` file and pass `--body-file`:

```bash
cat > /tmp/body.md <<'EOF'
## Summary
... markdown with `code spans`, paths like C:\x, and \n escapes ...
EOF
gh pr create  --body-file /tmp/body.md ...
gh issue create --body-file /tmp/body.md ...
gh pr edit N  --body-file /tmp/body.md
```

**Why (this is not style — it is correctness):** inline strings get mangled by the shell. Backslashes in code
spans (`` `\r` ``, `` `\t` ``, `` `fetchKernel` `` after a stray `` ` ``) are interpreted as C-style escapes and
turn into literal control characters — CR, TAB, BEL (`\a` → `0x07`), form-feed (`\f`). The result is a PR body with
invisible garbage (`\^GttestationCid`, `\<TAB>oReceiptPayload`) that fails human review and can't be trusted. A
heredoc **must** be single-quoted (`<<'EOF'`) so the shell does not expand it. On Windows/PowerShell the quoting
rules differ again — the file-based pattern is the one form that is correct on every shell and OS.

- Use `--body-file` for `gh issue create`, `gh issue edit`, `gh pr create`, `gh pr edit`, `gh pr comment`,
  `gh pr review --body-file`.
- If `gh pr edit` errors on a repo with a classic Project attached (`Projects (classic) is being deprecated…`),
  set the body via REST instead: `gh api -X PATCH repos/OWNER/REPO/pulls/N -F body=@/tmp/body.md`.

## Cross-repo claims — verify against the target, cite the ref

When a PR/issue body (or a review, or a code comment) asserts a fact about code that lives in **another repo**
("the kernel already types this", "no change needed in `imajin-ai`", "endpoint X returns shape Y"):

- **Verify against the target repo's `main` (or its live `/spec`), never your own local clone.** Clones drift — a
  checkout that's a few days stale will confidently contradict merged work. (This is how PRs end up asserting a type
  is non-optional when the source repo actually types it optional.)
- **Cite the ref you checked** — `owner/repo@main path/to/file.ts:line`, or the `/spec` route. A claim with no
  citation is an assumption, not evidence.
- **"No change required in <other service>" must name how you confirmed it.** Reading your own consumer clone is not
  confirmation of the provider's contract.
- Prefer verifying an integration contract from the provider's `/spec` or `main` source over inferring it from how
  your side happens to call it.

## Commit / PR hygiene

- Feature branch → PR. `[skip ci]` only for iteration commits.
- Squash-merge for clean history where it matters.
- Issue/PR bodies are file-based (see above) — no inline `--body` strings.
- Cross-repo claims cite the ref they were verified against (see above).
