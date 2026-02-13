# Code Audit

Automated code audit that intelligently selects the best tools for each project. Analyzes security, secrets, dependencies, test coverage, code structure, import graphs, and AI-generated code patterns. Zero dependencies. Cross-platform.

## Install

### Claude Code

```
/plugin marketplace add variant-systems/skills
/plugin install code-audit@variant-systems-skills
```

### npx skills add

```bash
npx skills add variant-systems/skills --skill code-audit
```

### Claude.ai

Upload the `code-audit` folder via [Settings > Skills](https://support.claude.com/en/articles/12512180-using-skills-in-claude#h_a4222fa77b).

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

Requires **Node.js 18+**. Enhanced by optional tools like [Semgrep](https://semgrep.dev), [Trivy](https://trivy.dev), [TruffleHog](https://trufflesecurity.com), and [Gitleaks](https://gitleaks.io) — the agent will detect your ecosystem and recommend what to install.
