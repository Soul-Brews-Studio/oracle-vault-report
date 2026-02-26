---
name: vault-report
description: Generate Oracle Vault dashboard — scan ψ/ repos, vault stats, skills inventory → OLED HTML report with GitHub Pages deploy.
---

# /vault-report — Oracle Vault Dashboard

Generate an OLED dark-mode dashboard showing your Oracle ψ/ memory structure, vault stats, and skills inventory.

## Usage

```
/vault-report              # Generate report (index.html + data.json)
/vault-report --deploy     # Generate + commit + push (updates GitHub Pages)
/vault-report --open       # Generate + open in browser
```

## Prerequisites

- **Node.js** 18+ (or Bun)
- **ghq** — repo manager (`ghq root` must work)
- Repos with `ψ/` directories (Oracle memory structure)
- Optional: central vault repo matching `*oracle-vault`

## Action

### Step 0: Locate the generator

```bash
# Find oracle-vault-report repo
REPORT_REPO="$(ghq list -p | grep oracle-vault-report | head -1)"
if [ -z "$REPORT_REPO" ]; then
  echo "oracle-vault-report not found. Cloning..."
  ghq get -u https://github.com/Soul-Brews-Studio/oracle-vault-report
  REPORT_REPO="$(ghq list -p | grep oracle-vault-report | head -1)"
fi
echo "Report repo: $REPORT_REPO"
```

### Step 1: Generate the report

```bash
cd "$REPORT_REPO" && node generate.mjs
```

This scans:
1. All ghq repos for `ψ/` directories (symlinked vs real)
2. Central vault (auto-detected via `$ORACLE_VAULT_PATH` or ghq)
3. Oracle skills in `~/.claude/skills/`

Outputs:
- `index.html` — OLED dark-mode dashboard (Tailwind CDN, glassmorphism)
- `data.json` — raw metrics for programmatic access

### Step 2: Open or deploy

**Open locally:**
```bash
open "$REPORT_REPO/index.html"
```

**Deploy to GitHub Pages (`--deploy`):**
```bash
cd "$REPORT_REPO" && node generate.mjs --push
```

This commits `index.html` + `data.json` and pushes to main → GitHub Actions deploys to Pages automatically.

### Step 3: Show summary

After generating, read `data.json` and present a summary:

```markdown
## Vault Report Generated

| Metric | Value |
|--------|-------|
| .md Files | [totalMdInVault] |
| Projects | [totalVaultProjects or totalWithPsi] |
| Orgs | [totalVaultOrgs] |
| Skills | [skills count] |
| Repos with ψ/ | [totalWithPsi] |
| Symlinked | [totalSymlinked] |
| Needs rsync | [totalNeedSync] |

**Report**: [path to index.html]
**Live**: https://soul-brews-studio.github.io/oracle-vault-report/
**JSON**: [path to data.json]
```

## What the report shows

- **Hero stats** — .md file count, projects, orgs, skills, vault size
- **ψ/ Connection Breakdown** — symlinked vs real repos vs worktrees
- **Org Distribution** — .md files per GitHub org (auto-colored by name hash)
- **Top 20 Projects** — horizontal bar chart by .md count
- **Rsync Eligible** — repos with real ψ/ dirs that need syncing to vault
- **Oracle Skills** — installed skills with descriptions

## Data safety

No secrets are included — only repo names, file counts, sizes, and skill names. No file contents, absolute paths, or credentials.

## For any AI agent

This skill works with any AI coding agent (Claude Code, Cursor, Windsurf, Copilot, etc). The prompt below can be given to any agent:

---

### Universal Agent Prompt

```
You are helping manage an Oracle knowledge vault.

The Oracle system uses ψ/ (psi) directories inside git repos to store .md knowledge files.
These can be:
- Symlinked to a central vault repo (already synced)
- Real directories (need rsync to vault)
- Worktree copies (need merge-mode rsync)

To generate a vault dashboard:

1. Find the oracle-vault-report repo:
   ghq list -p | grep oracle-vault-report

2. If not found, clone it:
   ghq get -u https://github.com/Soul-Brews-Studio/oracle-vault-report

3. Run the generator:
   cd <repo-path> && node generate.mjs

4. Open the result:
   open index.html

The generator auto-detects:
- ghq root (all managed repos)
- Central vault ($ORACLE_VAULT_PATH or ghq repo matching *oracle-vault)
- Oracle skills (~/.claude/skills/)

Output: index.html (OLED dashboard) + data.json (raw metrics)

To deploy to GitHub Pages:
   node generate.mjs --push
```
