# Code Audit

Automated code audit that intelligently selects the best tools for each project. Analyzes security, secrets, dependencies, test coverage, code structure, import graphs, and AI-generated code patterns. Cross-platform. Zero dependencies baseline, enhanced by external tools.

## Try in Claude Code, Claude.ai, and the API

### Claude Code

You can register the Variant Systems marketplace and install the skill by running the following commands in Claude Code:

```
/plugin marketplace add variant-systems/skills
/plugin install code-audit@variant-systems-skills
```

After installing the plugin, you can use the skill by just mentioning it. For instance, ask Claude Code: "Audit this codebase" or "Check for security issues"

### npx skills add

You can also install using the [`skills` CLI](https://github.com/vercel-labs/skills). Supports Claude Code, Cursor, Codex, OpenCode, and [35+ more agents](https://github.com/vercel-labs/skills#supported-agents).

```bash
# Install directly
npx skills add variant-systems/code-audit

# Install to a specific agent
npx skills add variant-systems/code-audit -a claude-code

# Install globally (available across all projects)
npx skills add variant-systems/code-audit -g
```

### Claude.ai

To upload as a custom skill, follow the instructions in [Using skills in Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude#h_a4222fa77b).

## Usage

Ask your AI agent any of:

- "Audit this codebase"
- "Review the code quality"
- "Check for security issues"
- "Assess technical debt"
- "How healthy is this codebase?"

The agent will:

1. Analyze the project to detect languages, frameworks, and ecosystem
2. Check for installed security tools (Semgrep, Trivy, TruffleHog, etc.)
3. Offer to install missing tools that would improve the audit
4. Run 7 analyzers covering security, secrets, dependencies, structure, tests, imports, and AI patterns
5. Generate `CODE_AUDIT_REPORT.md` with findings, severity ratings, and remediation guidance

## What It Covers

| Area | What It Finds |
| --- | --- |
| **Secrets** | Hardcoded API keys, tokens, credentials, private keys, .env files |
| **Security** | Injection risks (SQL, XSS, command), eval(), weak crypto, CORS misconfig |
| **Dependencies** | Known CVEs, unpinned versions, deprecated packages, missing lockfiles |
| **Structure** | God files (>500 LOC), deep nesting, long functions, high complexity |
| **Tests** | Coverage gaps, weak assertions, snapshot-only tests, no CI integration |
| **Imports** | Circular dependencies, hub files, coupling hotspots |
| **AI Patterns** | Tool fingerprints, silent error handling, generic code, TODO accumulation |

## Requirements

- **Node.js 18+** (only requirement)
- Zero npm dependencies — uses only Node.js stdlib

## Optional External Tools

The audit runs without any external tools using regex-based analysis. Installing these tools provides deeper, more accurate results:

| Tool | What It Adds |
| --- | --- |
| [Semgrep](https://semgrep.dev) | 2000+ security rules, multi-language |
| [Trivy](https://trivy.dev) | Multi-ecosystem vulnerability scanning |
| [TruffleHog](https://trufflesecurity.com) | Verified secret detection |
| [Gitleaks](https://gitleaks.io) | 150+ secret patterns |
| [Madge](https://github.com/pahen/madge) | JS/TS circular dependency detection |

The agent will detect your ecosystem and recommend the most relevant tools.

## Running the Script Directly

```bash
node path/to/code-audit/scripts/audit.mjs /path/to/target
```

Outputs `CODE_AUDIT_REPORT.md` in the target directory.

## Severity Levels

- **Critical** — Must fix immediately. Exposed secrets, active security vulnerabilities.
- **High** — Fix soon. Missing security controls, no tests, significant debt.
- **Medium** — Plan to fix. Code quality issues, moderate complexity.
- **Low** — Nice to fix. Style issues, minor complexity.
- **Info** — Awareness only. Observations, not necessarily problems.

## License

MIT
