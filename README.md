# Claude Skills

Global skills for [Claude Code](https://claude.com/claude-code) that enforce coding conventions automatically.

## Skills included

| Skill | Description |
|-------|-------------|
| **clean-code** | Clean code principles — DRY, SRP, < 500 LOC, explicit naming, early returns, no dead code, colocation |
| **atomic-design** | Atomic Design pattern for UI components — atoms, molecules, organisms |
| **jsdoc** | JSDoc conventions — file headers with `@author`, exported functions with `@param` and `@returns` |
| **security-audit** | Full security audit for web apps (Next.js, React, Node.js, API) — OWASP Top 10, XSS, CSRF, injection, auth, headers, secrets, dependencies. Invoke with `/security-audit` |
| **perf-audit** | Web performance audit — bundle size, lazy loading, memoization, images, Core Web Vitals, Server vs Client Components. Invoke with `/perf-audit` |
| **db-query-review** | Database query review — N+1, missing indexes, select *, pagination, transactions, Prisma/Drizzle. Invoke with `/db-query-review` |
| **a11y** | Web accessibility — ARIA roles, keyboard navigation, contrast, labels, focus management, semantic HTML |
| **dockerfile** | Dockerfile best practices — multi-stage builds, layer caching, non-root user, .dockerignore, health checks |
| **smart-contract-audit** | Security audit for EVM smart contracts (Solidity/Vyper) — reentrancy, access control, overflow, flash loans, oracle manipulation, proxy patterns, gas optimization. Invoke with `/smart-contract-audit` |
| **project-memory** | Automatic per-project MEMORY.md — stack, architecture, patterns, conventions, structure. Applied at session start and after architecture/pattern changes |
| **beads** | Workflow quotidien beads (bd) — detection automatique dans les projets `.beads/`, claim/create/close de taches. Remplace `TodoWrite` |
| **beads-setup** | Installe et configure beads (bd + dolt) — issue tracker graphique pour agents IA. Invoke avec `/beads-setup` |

## Installation

### Skills directory location

The skills directory depends on your environment and OS:

| Environment | macOS | Linux | Windows |
|-------------|-------|-------|---------|
| **Claude Code (CLI)** | `~/.claude/skills/` | `~/.claude/skills/` | `%USERPROFILE%\.claude\skills\` |
| **VS Code extension** | `~/.claude/skills/` | `~/.claude/skills/` | `%USERPROFILE%\.claude\skills\` |
| **Cursor extension** | `~/.claude/skills/` | `~/.claude/skills/` | `%USERPROFILE%\.claude\skills\` |

> All environments share the same global skills directory — install once, available everywhere.

### macOS / Linux

```bash
# Fresh install
git clone git@github.com:CorentinGC/claude-skills.git ~/.claude/skills

# If you already have a ~/.claude/skills/ directory
git clone git@github.com:CorentinGC/claude-skills.git /tmp/claude-skills
cp -r /tmp/claude-skills/*/ ~/.claude/skills/
rm -rf /tmp/claude-skills
```

### Windows (PowerShell)

```powershell
# Fresh install
git clone git@github.com:CorentinGC/claude-skills.git "$env:USERPROFILE\.claude\skills"

# If you already have a skills directory
git clone git@github.com:CorentinGC/claude-skills.git "$env:TEMP\claude-skills"
Copy-Item -Recurse "$env:TEMP\claude-skills\*" "$env:USERPROFILE\.claude\skills\" -Exclude ".git"
Remove-Item -Recurse -Force "$env:TEMP\claude-skills"
```

### Windows (Git Bash / WSL)

```bash
git clone git@github.com:CorentinGC/claude-skills.git ~/.claude/skills
```

Skills are automatically detected by Claude Code — no further configuration needed.

## Structure

```
~/.claude/skills/
├── clean-code/
│   └── SKILL.md
├── atomic-design/
│   └── SKILL.md
├── jsdoc/
│   └── SKILL.md
├── security-audit/
│   └── SKILL.md
├── perf-audit/
│   └── SKILL.md
├── db-query-review/
│   └── SKILL.md
├── a11y/
│   └── SKILL.md
└── dockerfile/
    └── SKILL.md
```

Each skill is defined by a `SKILL.md` file with a YAML frontmatter (`name`, `description`) and markdown instructions that Claude follows when writing or modifying code.

## Creating a new skill

1. Create a folder: `mkdir ~/.claude/skills/my-skill`
2. Add a `SKILL.md` file with this format:

```markdown
---
name: my-skill
description: Short description shown in Claude's skill list.
---

# Skill : My Skill

Instructions for Claude to follow...
```

3. Commit and push.

## License

MIT
