# skills

Agent Skills (per the [Agent Skills Specification](https://agentskills.io/specification)) used by
Dynamic's internal agents, loaded live at runtime via `embabel-agent-skills`'s
`GitHubSkillDefinitionLoader` — not compiled into any application jar. A new or corrected skill
ships via a PR merge to this repo, not an application deploy.

Each top-level directory is one skill: a `SKILL.md` file (YAML frontmatter + Markdown
instructions) plus optional `scripts/`, `references/`, `assets/` subdirectories. The frontmatter's
`name` must match the directory name.

## Skills

- `ipfs-funding-report/` — PFC disbursement statement parsing for IPFS/Plus's "Funding Report"
  format. Used by gottlieb-suite's PFC statement ingestion agent.
- `ledger-reconciliation-matching/` — Criteria for matching a confirmed disbursement statement line
  item against an open receivable LedgerEntry before creating reconciliation links. Used by
  gottlieb-suite's realm ledger statement ingestion agent.
