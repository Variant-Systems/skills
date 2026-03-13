# Plugin Launch Content Plan

> Last updated: February 16, 2026
> Content plan for the Claude Code Audit Plugin launch — our open-source distribution play.

---

## The Strategy

The plugin is distribution disguised as tooling. Every developer who uses it sees Variant Systems' name, experiences our quality standards, and has a natural upgrade path to paid human audits.

**Funnel:** Free plugin → flags issues → "want a human to dig deeper?" → /get-audit → paid engagement

---

## Launch Channels

### 1. GitHub Repo

**README must include:**
- Clear one-liner: what it does, who it's for
- 30-second install + first run instructions
- Screenshot or GIF of output (show, don't tell)
- "What it checks" — concrete list of audit categories
- "What it doesn't check" — be honest, builds trust
- Link to /get-audit for deeper analysis
- Variant Systems branding (subtle, not salesy)

**Repo hygiene:**
- Clean commit history
- MIT license
- Contributing guide
- Issue templates
- GitHub Actions for CI

---

### 2. Launch Posts (Day 1)

**Hacker News:**
```
Show HN: Open-source code audit plugin for Claude Code

Built a plugin that runs automated quality checks on AI-generated
codebases directly from Claude Code. It flags security vulnerabilities,
architectural anti-patterns, code duplication, and missing test coverage.

Free, open source, MIT licensed.

Context: 41% of GitHub code is now AI-generated. We kept seeing
the same failure patterns in codebases we audit professionally,
so we automated the detection.

Repo: [link]

Feedback welcome — especially on what checks to add next.
```

**Key HN rules:**
- No marketing language
- Lead with the technical substance
- "Show HN" format — you're sharing work, not pitching
- Engage genuinely with every comment
- Don't mention pricing or paid services in the post (let them find it)

---

**Reddit (r/webdev, r/programming, r/SaaS):**
```
Title: I built an open-source plugin that audits AI-generated code
quality [Claude Code]

Been doing code audits professionally for a while and kept seeing
the same patterns in AI-generated codebases:

- Security vulnerabilities (2.7x higher than human-written code)
- Glue code that breaks when APIs change
- Duplicated logic instead of proper abstractions
- Missing error handling in critical paths

So I automated the detection into a Claude Code plugin. It's free
and open source.

[What it checks]
[Link]

Would love feedback, especially from anyone dealing with vibe-coded
codebases.
```

**Reddit rules:**
- Post in relevant subs, don't spam
- Flair as [Tool] or [Open Source] where available
- Reply to every comment, even negative ones
- Don't be defensive — learn from feedback

---

**Twitter/X Thread:**
```
1/ Built an open-source code quality plugin for Claude Code.

It runs automated checks on AI-generated codebases and flags
the patterns that break production systems.

Free. MIT licensed. Here's what it catches: 🧵

2/ Security vulnerabilities
AI-generated code has 2.74x more security issues than
human-written code. The plugin checks for the top patterns:
[list 3-4 specific checks]

3/ Architectural anti-patterns
The "glue code" problem — AI generates code that works in
isolation but creates hidden dependencies. The plugin maps these.

4/ Code duplication
AI tools love copy-pasting logic instead of abstracting.
GitClear found an 8x increase in duplicated code blocks
from AI tools. The plugin measures your duplication rate.

5/ Missing test coverage
AI generates features without tests for edge cases.
The plugin identifies untested critical paths.

6/ Why I built this:
I audit codebases professionally at @VariantSystems.
Same patterns kept appearing. So I automated detection
to help more teams catch issues early.

7/ Get it here: [link]
Feedback welcome. What checks would you add?

If the plugin flags something serious and you want a human
to dig deeper: variantsystems.io/get-audit
```

---

### 3. Follow-Up Content (Week 1–4)

| When | What | Where |
|------|------|-------|
| Day 3 | "Lessons from the first 100 installs" thread | Twitter/X |
| Day 7 | Blog post: "What our audit plugin found in 50 AI-generated codebases" | variantsystems.io/blog |
| Day 14 | "Most common AI code quality issues" — data from plugin usage | Reddit, HN |
| Day 21 | Blog post: "The 5 architectural anti-patterns in every vibe-coded app" | variantsystems.io/blog |
| Day 30 | Plugin v2 announcement with community-requested features | GitHub, Twitter, Reddit |

---

### 4. Ongoing Distribution

**Star farming (organic):**
- Pin repo on GitHub profile
- Add to "awesome" lists (awesome-claude, awesome-dev-tools, etc.)
- Reference in blog posts and community answers
- Include in email signature: "Creator of [plugin name]"

**Content loop:**
- Anonymize and aggregate findings from plugin usage
- Turn patterns into blog posts and social content
- Each post links back to plugin (top of funnel) and /get-audit (bottom of funnel)

**Community maintenance:**
- Respond to issues within 24 hours
- Ship community-requested features monthly
- Acknowledge contributors publicly

---

## Success Metrics

| Metric | Week 1 Target | Month 1 Target | Month 3 Target |
|--------|--------------|----------------|----------------|
| GitHub stars | 50 | 200 | 500 |
| Installs | 100 | 500 | 1,500 |
| /get-audit visits from plugin | 10 | 50 | 150 |
| Health check conversions from plugin | 1 | 5 | 15 |
| Community contributions | 2 | 10 | 25 |

---

## Key Messages

**For developers:** "Catch the AI code quality issues before they catch you."
**For founders:** "Know what's actually in your codebase before your next raise."
**For CTOs:** "Automated baseline for AI code governance."

---

*Update this doc as the launch progresses. Track what content performs and double down.*
