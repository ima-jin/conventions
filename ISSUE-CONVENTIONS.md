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

## Commit / PR hygiene

- Feature branch → PR. `[skip ci]` only for iteration commits.
- Squash-merge for clean history where it matters.
