<required_reading>
Read these files before proceeding:
1. references/siteconfig-schema.md — full siteConfig structure to fill out
2. references/trade-defaults.md — default services per trade type
3. templates/cold-outreach.md — exact email template to use
</required_reading>

<objective>
For each prospect in the local CRM (~/Documents/development/website-factory/crm.json):
enrich with reviews, clone template, customize siteConfig, build, push to GitHub, draft outreach email.

**Each prospect build runs as a separate subagent to save tokens and prevent context bloat.**
**All data lives in crm.json — no Notion reads/writes needed.**
</objective>

<data_source>
## Local CRM

The single source of truth is `~/Documents/development/website-factory/crm.json`.

Structure:
- `emailTemplate` — the exact email copy to use (subject, body with {{variables}}, signer)
- `pricing` — sitePrice and monthlyManagement
- `cities` — target city list with progress tracking
- `prospects[]` — all prospect records

To read: `Read ~/Documents/development/website-factory/crm.json`
To update: `Write ~/Documents/development/website-factory/crm.json` (read first, modify, write back)

**This replaces Notion as the operational database. ~10x fewer tokens per update cycle.**
</data_source>

<process>

## Step 1: Read CRM and email template

Read `~/Documents/development/website-factory/crm.json` to get:
- The prospect list (filter by city or pipeline stage as needed)
- The email template (use EXACTLY as written — do not improvise copy)
- The signer name from `emailTemplate.signer`

No need to ask the user for their name — it's in the CRM.

## Step 2: Enrich prospects with review data

Before building, do a single targeted Apify scrape to get review text and opening hours.

Use `compass/crawler-google-places` with:
- `placeIds`: list of Google Maps placeIds from the CRM
- `scrapePlaceDetailPage: true` — for opening hours
- `maxReviews: 5` — enough for the reviews section + extracting owner name and services
- `maxImages: 0`
- `maxCrawledPlacesPerSearch: 1`
- Set timeout to 300000 (5 minutes max)

From the reviews, extract:
- **Owner/staff names** mentioned by customers
- **Services mentioned** → helps pick the right 6 services for siteConfig
- **Real review text** → use directly in the reviews section

## Step 3: Prepare build data for each prospect

After enrichment, compile a data object per prospect containing all data the subagent needs.

## Step 4: Launch subagents — one per prospect

For each prospect, launch a **background Agent**. Run up to 5 in parallel.

**Each subagent prompt must include ALL the data it needs.** The subagent only does: clone, customize, build, push, report back.

**IMPORTANT: Subagents do NOT create Gmail drafts or emails.** They only report back the email data. The main agent handles all email drafting to ensure the template is used exactly.

**Subagent prompt template:**

```
Build a website for a trades business. Follow these steps exactly:

## Business Data
- Name: [business name]
- Phone: [phone]
- Phone Href: [phoneHref]
- Email: [email or info@business-slug.com]
- Address: [address]
- City: [city]
- State: [state]
- PlaceId: [placeId]
- Google Rating: [rating]
- Review Count: [count]
- Year Established: [year or estimate]
- Opening Hours: [hours string]
- Owner Name: [if known]
- Trade: [electrician/plumber/etc]
- Credential: [e.g. Master Electrician]

## Real Reviews (use these in siteConfig.reviews.items)
[paste 3-5 real reviews with author, rating, date, text]

## Services to feature (6 total)
[list 6 services with Lucide icon names — use trade-defaults + review mentions]

## Nearby suburbs (12-16)
[list suburbs for service area]

## Steps

1. Clone the template:
   ```bash
   gh repo clone upperspacecase/adam-smith-electrician ~/Documents/development/[business-slug]
   cd ~/Documents/development/[business-slug]
   rm -rf .git
   ```

2. Edit `config/siteConfig.js` with ALL the business data above. Follow this structure exactly:
   - businessName, tagline, phone, phoneHref, smsHref, email, address, licenseNumber, yearEstablished, hoursOfOperation, emergencyAvailable
   - trustBar: only `credential` field (rating/reviews/years are derived automatically)
   - services: 6 items with title, description (benefit-focused, mentions location), icon (Lucide name)
   - about: headline + text (2-3 sentences, mentions location and experience)
   - reviews: rating, totalReviews, googleMapsUrl (use placeId), items (use the real reviews above)
   - serviceArea: mapEmbedUrl (Google Maps embed for [city, state]), suburbs array
   - contactForm: serviceOptions matching service titles + "Emergency Call-Out" + "Other"

   IMPORTANT: Do NOT include these fields (they are derived automatically):
   - trustBar.googleRating, trustBar.googleReviewCount, trustBar.yearsInBusiness
   - reviews.businessName
   - contactForm.recipientEmail

3. Run: `npm install && npx next build` — must pass with zero errors. If it fails, fix and retry.

4. Push to GitHub:
   ```bash
   git init && git add -A && git commit -m "Website for [Business Name]"
   gh repo create upperspacecase/[business-slug] --public --source=. --push
   ```

5. Report back with:
   - GitHub repo URL
   - Whether build passed
   - Any issues encountered

**DO NOT create Gmail drafts or send any emails. The main agent handles all outreach.**
```

## Step 5: Collect subagent results

As each subagent completes, collect:
- GitHub repo URL
- Build success/failure
- Any issues

## Step 6: Draft outreach emails (main agent ONLY)

**Use the EXACT template from crm.json emailTemplate.** Do not improvise or modify the copy.

Read the template from CRM, fill in variables:
- `{{first_name}}` — owner's first name, or "there" if unknown
- `{{demo_site_url}}` — the Vercel demo URL
- `{{your_name}}` — from emailTemplate.signer

**Use Gmail MCP to create drafts:**
```
Tool: mcp__claude_ai_Gmail__gmail_create_draft
Parameters:
  to: [prospect email]
  subject: [from emailTemplate.subject]
  body: [filled template — EXACT copy, no changes]
```

## Step 7: Update local CRM

Read crm.json, update each prospect with:
- `repo`: GitHub repo URL
- `demoUrl`: Vercel demo URL
- `pipelineStage`: "demo_built"
- `outreachStatus`: "email_drafted"
- `ownerName`: if discovered during enrichment

Also update the city record:
- `status`: "done"
- `sitesBuilt`: count of successful builds

Write the updated crm.json back. **This is 1 read + 1 write = ~2,000 tokens total.**

## Step 8: Report to user

Tell the user:
> "Built {X}/{total} websites. {Y} emails drafted. Review the drafts in Gmail and send when ready."
>
> Failures (if any): [list businesses that failed and why]
>
> Dashboard: http://localhost:3456

**Stop. User reviews drafts in Gmail and sends manually.**

</process>

<subagent_guidelines>

**Why subagents:** Each website build consumes ~30+ tool calls. Subagents isolate this, keeping the main conversation lean.

**Parallelism:** Launch up to 5 subagents at a time.

**Data handoff:** The main agent does ALL research (CRM reads, Apify enrichment) and passes complete data to each subagent. Subagents should NOT need to call the CRM, Notion, Apify, or Gmail — they only clone, customize, build, and push.

**Email copy:** Subagents NEVER draft emails. The main agent handles all Gmail drafts using the exact template from crm.json. This prevents copy drift.

**Error handling:** If a subagent reports a build failure, the main agent can retry once with corrected data.

</subagent_guidelines>

<success_criteria>
- Enrichment scrape completed in under 5 minutes
- Each prospect has a GitHub repo with working site
- siteConfig.js has all fields populated with real data
- Real review text used where available
- `next build` passes with zero errors
- Gmail drafts use EXACT copy from crm.json emailTemplate
- crm.json updated with repo URLs, demo URLs, pipeline stages
- City progress updated in crm.json
- Dashboard accessible at http://localhost:3456
- Main conversation context stays lean
</success_criteria>
