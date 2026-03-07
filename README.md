# Claude Skills

Global skills for [Claude Code](https://claude.com/claude-code) that enforce coding conventions automatically.

## Skills included

| Skill | Description |
|-------|-------------|
| **clean-code** | Clean code principles — DRY, SRP, < 500 LOC, explicit naming, early returns, no dead code, colocation |
| **atomic-design** | Atomic Design pattern for UI components — atoms, molecules, organisms |
| **jsdoc** | JSDoc conventions — file headers with `@author`, exported functions with `@param` and `@returns` |

## Installation

Clone this repo into your Claude Code global skills directory:

```bash
# Remove existing skills directory if needed, then clone
git clone git@github.com:CorentinGC/claude-skills.git ~/.claude/skills
```

Or if you already have a `~/.claude/skills/` directory with other content:

```bash
# Clone somewhere temporary and copy
git clone git@github.com:CorentinGC/claude-skills.git /tmp/claude-skills
cp -r /tmp/claude-skills/*/ ~/.claude/skills/
```

Skills are automatically detected by Claude Code — no further configuration needed.

## Structure

```
~/.claude/skills/
├── clean-code/
│   └── SKILL.md
├── atomic-design/
│   └── SKILL.md
└── jsdoc/
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
