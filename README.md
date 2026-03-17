# Skills

Agent skills by [Variant Systems](https://variantsystems.io). Works with Claude Code, Cursor, Codex, and [35+ more agents](https://github.com/vercel-labs/skills#supported-agents).

## Install

### Claude Code

```
/plugin marketplace add variant-systems/skills
/plugin install code-audit@variant-systems-skills
/plugin install postbox@variant-systems-skills
```

### npx skills add

```bash
npx skills add variant-systems/skills --skill code-audit
npx skills add variant-systems/skills --skill postbox
```

### Claude.ai

Upload the skill folder via [Settings > Skills](https://support.claude.com/en/articles/12512180-using-skills-in-claude#h_a4222fa77b).

## Available Skills

| Skill | Description |
| --- | --- |
| [code-audit](./code-audit) | Automated code audit — security, secrets, dependencies, test coverage, code structure, and AI-generated code patterns |
| [postbox](./postbox) | Collect structured data, create forms, and set up submission endpoints — contact, feedback, signup, waitlist, and more. Webhooks, notifications, and spam protection via Postbox API |
