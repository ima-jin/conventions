# ima-jin/conventions

Canonical, portable Imajin repo conventions — consumed by `imajin-ai`, `imajin-app-template`, and every app that
adopts the template. **Nobody forks anybody; everyone consumes this.**

Two halves, each in its right form:

| Half | File | Form |
|------|------|------|
| **Labels** | [`taxonomy.json`](./taxonomy.json) | executable state — seed with the init script |
| **Issue/PR rules** | [`ISSUE-CONVENTIONS.md`](./ISSUE-CONVENTIONS.md) | prose — imported as AGENTS.md **§7** |

## Seed a repo's labels

```bash
# universal set (any repo)
scripts/init-taxonomy.sh ima-jin/some-app

# monorepo also gets the platform domain set
scripts/init-taxonomy.sh ima-jin/imajin-ai --set universal --set platform

# preview only
scripts/init-taxonomy.sh ima-jin/some-app --dry-run
```

Idempotent (`gh label create --force` upserts): safe to re-run, updates colour/description, never duplicates.

## Label sets

- **universal** — type/topic/status vocabulary with no domain coupling (`bug`, `enhancement`, `epic`, `rfc`,
  `tech-debt`, `documentation`, `work-order`, `exploration`, `question`, `help wanted`, `invalid`, `wontfix`).
  Every consuming repo gets these.
- **platform** — `imajin-ai`-only domain labels (`bus`, `fair`, `mjn-protocol`, `events`). **Not** for app repos.

## Consumers

- **`ima-jin/imajin-ai`** — runs the init script (universal + platform); AGENTS.md §7 points here.
- **`ima-jin/imajin-app-template`** — ships a copy of the init script; new apps seed the universal set on init.
- **apps** (xprize, scorecard, fixready, karaoke, …) — seed universal set; inherit §7 via the template sync.

## Scope

Deliberately **excludes** the monorepo fork-and-PR model and the org roadmap board (Status/Priority/Vertical fields) —
those are `imajin-ai`/org-specific, not portable. Board wiring may become a v2 opt-in for repos that enter a roadmap
sprint. v1 is **labels-only**.
