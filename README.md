# skills

Agent Skills (per the [Agent Skills Specification](https://agentskills.io/specification)) used by
Dynamic's internal agents, loaded live at runtime via `embabel-agent-skills`'s
`GitHubSkillDefinitionLoader` — not compiled into any application jar. A new or corrected skill
ships via a PR merge to this repo, not an application deploy.

Each skill is a directory with a `SKILL.md` file (YAML frontmatter + Markdown instructions) plus
optional `scripts/`, `references/`, `assets/` subdirectories. The frontmatter's `name` must match
the directory name.

Skills are classified two levels deep: **subject area** first (the business domain the skill
belongs to, e.g. `ledger-document`), then **purpose** (what kind of task within that domain, e.g.
`extraction` or `matching`). Named by domain and purpose, not by whichever agent/class currently
calls them — a caller that loads "every skill under a path" should only ever see skills relevant to
its own concern, and the classification should hold even if the calling code is later renamed or a
second caller in the same domain shows up.

## Skills

- `ledger-document/extraction/ipfs-funding-report/` — PFC disbursement statement parsing for
  IPFS/Plus's "Funding Report" format.
- `ledger-document/matching/ledger-reconciliation-matching/` — Criteria for matching a confirmed
  disbursement statement line item against an open receivable LedgerEntry before creating
  reconciliation links.

Both are currently used by gottlieb-suite's realm ledger statement ingestion agent.
