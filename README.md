Claude skills that might come in handy

## Skills

### `review-pr` — Python/Django PR Review

Reviews a GitHub pull request for Python 3.13 and Django issues, presents numbered findings by severity, and posts selected ones as a **pending** GitHub review for you to submit manually.

**Usage:** `/review-pr <PR-URL | PR-number> [repo]`

**Files:** [`pr-review/`](./pr-review/)

### `write-prd` — Write a Product Requirement Document

Reads the current session context and produces a concise PRD in Markdown, printed directly in the chat.

**Usage:** `/write-prd [optional title or one-liner]`

**Files:** [`write-prd/`](./write-prd/)

---

## Installation

Skills live in `~/.claude/skills/` (global) or `.claude/skills/` (project-local).

After cloning this repository, copy the skill directory you want to install:

```bash
# Global install (available in all projects)
cp -r pr-review ~/.claude/skills/

# Project-local install (available only in the current project)
cp -r pr-review /path/to/your/project/.claude/skills/
```

Claude Code will pick up the skill automatically — no restart needed. Invoke it with the `/` prefix matching the `name` field in the skill's frontmatter (e.g. `/review-pr`).
