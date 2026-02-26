# Oracle Vault Report — Agent Instructions

You are working with the Oracle Vault Report generator. This tool creates an OLED dark-mode HTML dashboard showing the state of an Oracle knowledge system.

## Context

The **Oracle** system stores knowledge as `.md` files inside `ψ/` (psi) directories in git repositories. Repos are managed with [ghq](https://github.com/x-motemen/ghq).

There are three types of ψ/ connections:
- **Symlinked** → points to a central vault repo (already in sync)
- **Real directory** → local .md files that need rsync to the vault
- **Worktree** → git worktree copy, needs merge-mode rsync (no `--delete`)

## How to generate a report

```bash
node generate.mjs
```

This produces:
- `index.html` — self-contained OLED dashboard (Tailwind CDN, no build step)
- `data.json` — machine-readable metrics

## How to deploy

```bash
node generate.mjs --push
```

This commits both files and pushes to `main`. GitHub Actions automatically deploys to GitHub Pages.

## Auto-detection

The generator finds everything automatically:

| Source | Detection |
|--------|-----------|
| Repos | `ghq list -p` → all managed repos |
| Vault | `$ORACLE_VAULT_PATH` env var, or ghq repo matching `*oracle-vault` |
| Skills | `~/.claude/skills/*/SKILL.md` |
| Worktrees | Path contains `.wt-` |

No configuration needed. Works with or without a central vault.

## What's in the report

| Section | Description |
|---------|-------------|
| Hero stats | .md count, projects, orgs, skills, vault size |
| Connection Breakdown | Symlinked vs real vs worktree repos |
| Org Distribution | Files per GitHub org with auto-colored bars |
| Top 20 Projects | Horizontal bar chart by .md count |
| Rsync Eligible | Repos needing sync to vault |
| Skills Inventory | Installed Oracle skills |

## Data safety

Only public metadata is included: repo names, file counts, byte sizes, skill names. No file contents, secrets, or absolute paths are exposed.

## Common tasks

| Task | Command |
|------|---------|
| Generate report | `node generate.mjs` |
| Generate + deploy | `node generate.mjs --push` |
| View locally | `open index.html` |
| Read metrics | `cat data.json \| jq .` |
| Check .md count | `cat data.json \| jq .totalMdInVault` |
| List projects | `cat data.json \| jq '.vaultProjects[].project'` |

## Modifying the generator

`generate.mjs` is a single-file Node.js script with zero dependencies. It:

1. Scans ghq repos for ψ/ directories
2. Scans the vault for project/org stats
3. Scans ~/.claude/skills/ for skill metadata
4. Generates self-contained HTML with inline Tailwind CSS
5. Writes data.json with raw metrics

Key functions:
- `orgColor(org)` — deterministic color from org name hash
- `countMd(dir)` — count .md files recursively
- `dirSize(dir)` — directory size in bytes
- `generateHTML()` — builds the full HTML string

To add a new section: add data collection after the scan phase, then add HTML in `generateHTML()`.
