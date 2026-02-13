# Skills

Skills are folders of instructions, scripts, and resources that Claude loads dynamically to improve performance on specialized tasks. Skills teach Claude how to complete specific tasks in a repeatable way.

For more information, check out:
- [What are skills?](https://support.claude.com/en/articles/12512176-what-are-skills)
- [Using skills in Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude)
- [How to create custom skills](https://support.claude.com/en/articles/12512198-creating-custom-skills)
- [Agent Skills specification](https://agentskills.io)

## About This Repository

This repository contains skills built by [Variant Systems](https://variantsystems.io). Each skill is self-contained in its own folder with a `SKILL.md` file containing the instructions and metadata that Claude uses.

### Available Skills

| Skill | Description |
| --- | --- |
| [code-audit](./code-audit) | Automated code audit — security, secrets, dependencies, test coverage, code structure, import graphs, and AI-generated code patterns |

## Try in Claude Code, Claude.ai, and the API

### Claude Code

You can register this repository as a Claude Code Plugin marketplace by running the following command in Claude Code:

```
/plugin marketplace add variant-systems/skills
```

Then, to install a specific skill:

```
/plugin install code-audit@variant-systems-skills
```

After installing the plugin, you can use the skill by just mentioning it. For instance, ask Claude Code: "Audit this codebase" or "Check for security issues"

### npx skills add

You can also install skills using the [`skills` CLI](https://github.com/vercel-labs/skills). Supports Claude Code, Cursor, Codex, OpenCode, and [35+ more agents](https://github.com/vercel-labs/skills#supported-agents).

```bash
# List available skills
npx skills add variant-systems/skills --list

# Install a specific skill
npx skills add variant-systems/skills --skill code-audit

# Install to a specific agent
npx skills add variant-systems/skills --skill code-audit -a claude-code

# Install globally (available across all projects)
npx skills add variant-systems/skills --skill code-audit -g
```

### Claude.ai

To upload custom skills, follow the instructions in [Using skills in Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude#h_a4222fa77b).

## Creating a Skill

Skills are simple to create — just a folder with a `SKILL.md` file containing YAML frontmatter and instructions. See [SKILLS_SPEC_REFERENCE.md](./SKILLS_SPEC_REFERENCE.md) for the full specification.

```markdown
---
name: my-skill-name
description: A clear description of what this skill does and when to use it
---

# My Skill Name

[Add your instructions here that Claude will follow when this skill is active]

## Examples
- Example usage 1
- Example usage 2

## Guidelines
- Guideline 1
- Guideline 2
```

The frontmatter requires only two fields:
- `name` — A unique identifier for your skill (lowercase, hyphens for spaces)
- `description` — A complete description of what the skill does and when to use it

## License

MIT
