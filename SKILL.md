---
name: website-factory
description: "Two-phase system for building and selling websites to trades businesses. Phase 1 (Research): Discover Google Maps businesses without websites, rank by review count, create Notion CRM. Phase 2 (Build): Clone template repo, update siteConfig with business data, push to GitHub, draft outreach email. Trigger: 'website factory', 'find tradies', 'prospect [trade] in [location]', 'build websites for [trade]'."
---

<essential_principles>

This skill finds trades businesses that need websites, builds sites for them, and drafts cold outreach emails to sell them.

**Two phases, always in order:**
1. **Research** — find prospects, score them, save to Notion
2. **Build** — clone template, customize, push to GitHub, draft email

**Hard rules:**
- Never assess website quality. Only signal: does Google Maps show a website URL or not.
- Never fetch prospect URLs. Binary check only.
- Never scaffold from scratch. Always clone the template repo.
- Never send emails. Only create Gmail drafts. User sends manually.
- Always stop between phases. User must approve prospects before building.

**Dependencies:**
- Apify MCP (preferred) or web search for Google Maps discovery
- Notion MCP for CRM database
- Gmail MCP for email drafts (fallback: save to Notion)
- GitHub CLI (`gh auth status`)
- Template repo: `upperspacecase/adam-smith-electrician`
- Node.js and npm

</essential_principles>

<routing>

What would you like to do?

| Intent | Trigger phrases | Workflow |
|--------|----------------|----------|
| Research | "find [trade] in [location]", "prospect", "research" | workflows/research.md |
| Build | "build websites", "build for [name]", "run build" | workflows/build.md |

**If user provides a trade and location** → go directly to workflows/research.md, skip asking.
**If user says "build"** → go directly to workflows/build.md, skip asking.
**If unclear** → ask: "Research new prospects, or build websites for approved ones?"

</routing>
