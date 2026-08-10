# skills

Agent Skills (per the [Agent Skills Specification](https://agentskills.io/specification)) used by
Dynamic's internal agents, loaded live at runtime via `embabel-agent-skills`'s
`GitHubSkillDefinitionLoader` — not compiled into any application jar. A new or corrected skill
ships via a PR merge to this repo, not an application deploy.

Each skill is a directory with a `SKILL.md` file (YAML frontmatter + Markdown instructions) plus
optional `scripts/`, `references/`, `assets/` subdirectories. The frontmatter's `name` must match
the directory name.

Skills are grouped into top-level folders by purpose, not left flat at the repo root -- a caller
that loads "every skill under a path" (gottlieb-suite's statement extraction step does this, since
it doesn't know in advance which vendor format applies) should only ever see skills relevant to
its own concern, not unrelated ones that happen to live in the same repo.

## Skills

- `extraction/ipfs-funding-report/` — PFC disbursement statement parsing for IPFS/Plus's "Funding
  Report" format. Used by gottlieb-suite's statement extraction step.
- `ledger-reconciliation-matching/` — Criteria for matching a confirmed disbursement statement line
  item against an open receivable LedgerEntry before creating reconciliation links. Loaded by exact
  path (not folder-scanned), so it isn't grouped under a purpose folder.
