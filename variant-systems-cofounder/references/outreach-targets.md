# Outreach Targets

> Last updated: February 16, 2026
> Update this file as new target segments are identified or existing ones are refined.

---

## Target Profiles

### Tier 1: Post-Raise Startups (Highest Priority)

**Who:** VC-backed startups that raised $1–5M in the last 3–6 months, built fast with AI tools, now need to scale or pass technical review.

**Why they need us:**
- 25% of YC W2025 batch had 95%+ AI-generated codebases
- They shipped fast to hit milestones, now face code quality reckoning
- Investors often require technical due diligence before follow-on rounds
- 75% of tech decision-makers projected to face moderate-to-severe technical debt by 2026

**Where to find them:**
- Crunchbase: Filter by funding date (last 3–6 months), stage (Seed/Series A), industry (SaaS, AI, FinTech)
- AngelList / Wellfound: Recently funded startups hiring engineers (signal: they're scaling and the codebase matters)
- X/Twitter: Founders announcing raises — engage genuinely before outreaching
- LinkedIn: Founders and CTOs at companies matching the profile
- Product Hunt: Recent launches by small teams (often vibe-coded MVPs)
- Fundraise Insider: Weekly lists of newly funded startups with contact info

**Signal they need us:**
- Hiring senior engineers after shipping with a small team or solo founder
- Complaining about "tech debt" or "scaling challenges" publicly
- Posting about migrating frameworks or rewriting systems
- Just raised and now need to demonstrate technical maturity to investors

**Entry point:** "AI Code Health Check" — $1,500–$2,500, no sales call needed, risk-tiered pricing.

---

### Tier 2: SMBs with AI-Built Products in Production

**Who:** Small-to-medium businesses running AI-built products that are breaking, slow, or insecure in production.

**Why they need us:**
- They have revenue and customers depending on the product
- AI-generated code is "functional but lacking architectural judgment" (Ox Security, 2025)
- 46% of developers actively distrust AI tool output — but shipped it anyway
- These founders often can't debug what they didn't write

**Where to find them:**
- Reddit: r/webdev, r/SaaS, r/startups — look for posts about production issues, scaling problems, "my app is slow/broken"
- Indie Hackers: Founders posting about technical challenges
- Twitter/X: Founders of bootstrapped products sharing frustrations
- Upwork/Freelancer: People posting rescue/fix jobs (engage, don't bid — redirect to Variant Systems)

**Signal they need us:**
- Production outages or performance complaints from their users
- "Looking for a senior developer to fix..." posts
- Using terms like "spaghetti code," "tech debt," "need to rewrite"
- Shipping features that keep breaking other features

**Entry point:** Rescue engagement — "It's broken, we fix it." Fast, fixed-scope, proves value immediately.

---

### Tier 3: Enterprises with AI-Generated Internal Tools

**Who:** Companies using AI to build internal tools (admin dashboards, data pipelines, automation) that now have compliance or security concerns.

**Why they need us:**
- AI-generated code has 2.74x higher security vulnerability rates
- Regulatory pressure increasing (financial services, healthcare, critical infrastructure)
- Internal tools built by non-engineering staff using vibe coding are accumulating hidden risk

**Where to find them:**
- LinkedIn: CTOs, VP Engineering at mid-size companies posting about AI adoption challenges
- Industry conferences and webinars on AI governance
- Security/compliance forums and Slack communities

**Entry point:** Audit engagement — "Is your AI-generated code safe?" Frames it as risk management, not criticism.

---

### Tier 4: PE Firms / VCs / Acquirers (Year 2–3 Play)

**Who:** Private equity firms, venture capitalists, and companies doing acquisitions who need technical due diligence.

**Why they need us:**
- Need independent assessment of code quality before writing checks
- AI-generated codebases are a new risk factor they don't have frameworks for
- High-margin, relationship-driven work — one engagement can be worth 10 rescue jobs

**Where to find them:**
- Build reputation through Tier 1–3 work first
- Network through founders we've helped (referrals)
- Publish due diligence content that ranks (already have 3 DD blog posts)
- LinkedIn thought leadership targeting PE/VC audiences

**Entry point:** Not yet — build credibility through rescue and audit case studies first. Revisit when we have 5+ completed engagements and published case studies.

---

## Sourcing Rhythm

| Activity | Frequency | Owner |
|----------|-----------|-------|
| Crunchbase scan for new raises | Weekly | Claude (research) + Vipul (outreach) |
| Reddit/Indie Hackers monitoring | Daily (15 min) | Vipul (engage) |
| LinkedIn content + connection requests | 3x/week | Vipul (post) + Claude (draft) |
| X/Twitter engagement on AI code quality threads | Daily (10 min) | Vipul |
| Review and refresh target list | Biweekly | Claude + Vipul |

---

## Tracking

| Date | Target | Channel | Status | Notes |
|------|--------|---------|--------|-------|
| (fill as outreach happens) | | | | |

---

---

## Reddit Subreddit Targets

> Added: February 22, 2026
> Strategy: Lurk and contribute first, post own content after building activity in each sub.

### Tier 1 — Our people are here right now

| Subreddit | Why |
|-----------|-----|
| r/vibecoding (87K+) | Ground zero. People building with AI tools whose code will break. Every post is a potential lead. |
| r/SaaS | Founders building products, many with AI tools. Care about code quality when scaling. |
| r/webdev (6M+) | Developers who've seen bad AI code. "I inherited a vibe-coded codebase" posts are gold. |
| r/cursor | Cursor users hitting walls. Our fix guides target these exact people. |

### Tier 2 — Founders who don't know they need us yet

| Subreddit | Why |
|-----------|-----|
| r/startups | Early-stage founders discussing MVPs, scaling, technical decisions |
| r/indiehackers | Solo builders, bootstrapped. Build fast with AI, hit walls later. |
| r/entrepreneur | Broader, but non-technical founders using AI tools show up here |
| r/nocode | Non-technical builders using Lovable, Bolt, v0. They'll need us first. |

### Tier 3 — Developer credibility plays

| Subreddit | Why |
|-----------|-----|
| r/programming (6M+) | Show HN energy. Good for the audit plugin, not for pitching. |
| r/ExperiencedDevs | Senior engineers. Credibility, not leads. |
| r/devops | Relevant now that we have 66 infrastructure pages. |

### Reddit Engagement Rules
- Be useful 3-4 times in a sub before posting own content
- End posts with questions to invite discussion
- Never mention paid services in posts — let them find variantsystems.io through GitHub
- Reddit punishes self-promoters and rewards contributors

---

## Ready-to-Post Drafts

### r/vibecoding — Code Audit Skill Launch

**Status:** DRAFT — waiting for Vipul to build activity in the sub first

**Title:** I open-sourced a code audit skill that checks what AI coding tools leave behind

**Body:**

Been doing code audits professionally for a while. Same patterns kept showing up in every AI-generated codebase we reviewed — leaked secrets, missing tests, dead imports, no real architecture.

So we automated parts of our audit process and gave it away.

It's a skill (works with Claude Code, and any tool that supports the skills protocol). Point it at any codebase, it runs:

→ Secrets & credentials scanning
→ Security vulnerability checks
→ Dependency analysis (outdated/vulnerable)
→ Project structure & architecture review
→ Test coverage gaps
→ Dead imports & unused code
→ AI-generated code pattern detection

One command, full report with severity ratings.

No signup, no paywall. Just clone and run.

GitHub: github.com/Variant-Systems/skills

Curious what patterns you all are hitting in your vibe-coded projects. What would you want an audit tool to check for?

---

*When updating: add new targets, mark contacted ones, note what worked and what didn't. Patterns compound into playbooks.*
