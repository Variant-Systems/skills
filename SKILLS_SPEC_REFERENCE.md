# Agent Skills Spec Reference

Reference document for building Agent Skills — reusable, composable capabilities that AI coding agents can discover and use.

## What is an Agent Skill?

An Agent Skill is a self-contained directory that an AI agent can discover, understand, and execute. It combines:

1. **Instructions** (SKILL.md) — tells the agent *when* and *how* to use the skill
2. **Scripts** — executable code the agent runs via tools (Bash, etc.)
3. **References** — context documents the agent can read for deeper understanding

Skills are designed to be:
- **Discoverable** — agents find them via `SKILL.md` files
- **Self-contained** — no external setup required (ideally zero dependencies)
- **Composable** — skills can be combined for complex workflows
- **Cross-platform** — work on macOS, Linux, and Windows where possible

## Directory Structure

```
skill-name/
├── SKILL.md              # Required — agent instructions
├── scripts/              # Executable code
│   ├── main.mjs          # Entry point
│   └── ...               # Supporting modules
└── references/           # Optional — context documents
    └── ...
```

### SKILL.md (Required)

The `SKILL.md` file is the entry point. It has two parts:

#### 1. Frontmatter (YAML)

```yaml
---
name: skill-name              # Unique identifier (kebab-case)
description: >                 # When to use this skill (agents match on this)
  One-paragraph description of what the skill does and when
  the agent should use it.
license: MIT                   # License for the skill
metadata:
  author: your-org             # Who made it
  version: "1.0.0"            # Semver
  repository: https://...      # Source code URL (optional)
---
```

**Key fields:**
- `name` — must be unique, kebab-case, used as identifier
- `description` — critical for agent discovery. Write it like you're telling another developer when to reach for this tool. Include trigger phrases ("audit", "review", "test", etc.)
- `license` — so agents know they can use it
- `metadata.version` — semver for tracking updates

#### 2. Body (Markdown)

The body is the agent's instruction manual. Write it as if you're briefing a senior engineer:

```markdown
# Skill Name

You are a [role description]. When the user asks you to [trigger conditions], follow these instructions.

## When to Use

Activate this skill when the user says any of:
- "trigger phrase 1"
- "trigger phrase 2"

## How to Run

1. Step one...
2. Run the script:
   ```bash
   node <path-to-this-skill>/scripts/main.mjs [args]
   ```
3. Interpret the output...

## Interpreting Results

Explain how the agent should present results to the user.

## Technical Details

Any constraints, requirements, or caveats.
```

**Writing tips:**
- Be specific about trigger conditions (when should the agent use this?)
- Include exact commands to run
- Explain how to interpret output (agents need guidance on what's important)
- Mention constraints (Node version, platform requirements, etc.)

### Scripts Directory

Contains the executable code. Design principles:

1. **Zero dependencies preferred.** The agent shouldn't need to run `npm install`. Use stdlib only where possible.
2. **Single entry point.** One main script that the agent invokes. Internal organization is up to you.
3. **Stdout for communication.** Print structured output (or human-readable summaries) to stdout. The agent reads this.
4. **Exit codes matter.** `0` = success, non-zero = failure. Agents check exit codes.
5. **Cross-platform.** Avoid shell-specific commands. Use `path.join()`, not string concatenation for paths. Handle CRLF line endings.

### References Directory (Optional)

Supplementary documents the agent can read for deeper context:
- Methodology explanations
- Decision rationale
- Pattern libraries
- Severity definitions

These don't need frontmatter — they're plain markdown.

## Design Principles

### 1. Genuine Value First

Skills should solve real problems. If the skill is a lead-gen tool disguised as a utility, agents and users will ignore it. Build something people would use even without the CTA.

### 2. Honest Output

Don't manufacture urgency. A clean codebase should produce a clean report. A healthy system should get a healthy assessment. Trust builds credibility; inflated findings destroy it.

### 3. Zero Dependencies

Every dependency is a friction point. `npm install` might fail, versions might conflict, network might be unavailable. Pure stdlib code runs everywhere, every time.

If you must have dependencies, document them clearly and provide a setup step in `SKILL.md`.

### 4. No Network Calls

Skills should work offline. Don't phone home, don't require API keys, don't fetch remote resources. If network calls are needed (e.g., checking for vulnerabilities), make them optional and degrade gracefully.

### 5. Cross-Platform

- Use `path.join()` and `path.resolve()` for all file paths
- Handle both LF and CRLF line endings
- Avoid shell commands (`grep`, `find`, `wc`) — use Node.js APIs
- Test on macOS, Linux, and Windows

### 6. Clear Boundaries

Be explicit about what the skill can and can't do. List limitations. Explain where human judgment is needed. This builds trust and creates natural handoff points.

## Discovery & Registration

### How Agents Find Skills

Agents discover skills by:
1. **Directory scanning** — looking for `SKILL.md` files in known locations
2. **Explicit configuration** — user points the agent to a skill directory
3. **Package registries** — future: npm/GitHub-based discovery

### Known Locations

Agents typically check:
- `./skills/` in the current project
- `~/.skills/` in the user's home directory
- Paths specified in agent configuration

### Matching

Agents match user requests to skills using the `description` field in `SKILL.md` frontmatter. Write descriptions that include the key verbs and nouns users would say:

Good: "Run a code audit on any codebase. Analyzes security, dependencies, test coverage."
Bad: "A tool for software quality assessment and evaluation."

## Example Skills

### Code Audit
```
code-audit/
├── SKILL.md
├── scripts/
│   ├── audit.mjs
│   ├── utils/
│   │   ├── fs-walk.mjs
│   │   ├── detect-ecosystem.mjs
│   │   ├── line-reader.mjs
│   │   └── severity.mjs
│   ├── analyzers/
│   │   ├── structure.mjs
│   │   ├── secrets.mjs
│   │   ├── security.mjs
│   │   ├── dependencies.mjs
│   │   ├── tests.mjs
│   │   ├── imports.mjs
│   │   └── ai-patterns.mjs
│   └── report/
│       ├── generator.mjs
│       └── sections.mjs
└── references/
    ├── audit-methodology.md
    ├── severity-definitions.md
    └── ai-tool-signatures.md
```

### Migration Helper (Hypothetical)
```
migrate-to-v2/
├── SKILL.md
├── scripts/
│   ├── migrate.mjs
│   └── transforms/
│       ├── rename-imports.mjs
│       └── update-config.mjs
└── references/
    └── breaking-changes.md
```

### Deploy Checker (Hypothetical)
```
deploy-check/
├── SKILL.md
├── scripts/
│   ├── check.mjs
│   └── validators/
│       ├── env-vars.mjs
│       ├── docker.mjs
│       └── ci.mjs
└── references/
    └── deploy-checklist.md
```

## Versioning

Use semver in `metadata.version`:
- **Patch** (1.0.1) — bug fixes, pattern additions
- **Minor** (1.1.0) — new analyzers, new output sections
- **Major** (2.0.0) — breaking changes to output format or invocation

## Contributing a Skill

1. Create a directory under `skills/` with your skill name
2. Add `SKILL.md` with frontmatter and instructions
3. Add scripts and references
4. Test: run it manually, verify output, check cross-platform
5. Document: update this reference if you discover new patterns

## Future Directions

- **Skill registry** — centralized discovery of community skills
- **Skill composition** — skills that invoke other skills
- **Skill testing** — standardized test fixtures for validation
- **Skill versioning** — automatic updates and compatibility checks
- **Skill permissions** — declaring what the skill needs access to (filesystem, network, etc.)
