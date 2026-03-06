---
name: website-factory
description: "Automated prospect-to-website pipeline for trades businesses. Runs in stages with user approval gates. Stage 1: Discover prospects via Google Maps + score leads + create Notion CRM. Stage 2: User reviews Notion, approves prospects. Stage 3: Build all websites using ui-ux-pro-max design, copywriting skill for copy, nano-banana for hero images, cold-email skill for outreach. Stage 4: User deploys via GitHub. Stage 5: User sends emails. Trigger when user says: 'run website-factory', 'find tradies', 'build websites for [trade]', 'prospect [trade] in [location]', or 'website factory'."
---

# Website Factory

Automated pipeline for finding trades businesses and building them websites to sell.

**Runs in 3 stages with approval gates — you stay in control.**

---

## Stage 1: RESEARCH (Agent runs, user reviews)

The agent does all the discovery, scoring, and database creation. Then stops for review.

### 1a. Discover Prospects

**Input:** `[trade]` (e.g., "electrician") and `[location]` (e.g., "Brisbane, Australia")

**Use Apify MCP** — Actor: `compass/crawler-google-places` (Google Maps Scraper)
- Search query: `[trade] in [location]`
- Extract per business: name, phone, website URL, address, Google rating, review count, hours, email, placeId

**If Apify is unavailable**, fall back to web searching and manually extracting details.

### 1b. Score Each Prospect

Score 0–100 based on readiness to buy a website.

| Signal | Points | Logic |
|--------|--------|-------|
| No website at all | +40 | `website === null` or domain dead |
| Bad website | +25 | Not mobile-friendly, outdated, slow, no HTTPS, Wix/Squarespace free tier, generic template |
| Decent/modern website | +0 | Skip — they don't need us |
| Google rating 4.5+ | +15 | Successful, likely to invest |
| 50+ reviews | +15 | Established business |
| 10–49 reviews | +10 | Growing business |
| Has email visible | +5 | Easy to reach |
| Has phone | +5 | Can be contacted |

**To assess website quality:**
- Use WebFetch to load their URL
- Check for: viewport meta tag, modern CSS, HTTPS, reasonable structure
- Old WordPress themes, Wix free tier, broken layouts = "bad website"

**Tiers:**
- **HOT (70–100):** No website + lots of reviews. Priority targets.
- **WARM (40–69):** Bad website or growing business. Good prospects.
- **COLD (0–39):** Already has a decent website. Skip.

### 1c. Scrape Existing Sites

For prospects with websites (even bad ones), extract:
- Business name, tagline, services, about text
- Phone, email, address, hours, license numbers
- Testimonials, team names
- Logo URL, hero images, gallery images

Use WebFetch. Store as structured data per prospect.

For prospects with **no website**, use only Google Maps data + flag for generated copy.

### 1d. Create Notion CRM

**Database name:** `[Trade] Prospects — [Location]`

**Columns:**

| Property | Type |
|----------|------|
| Business Name | Title |
| Score | Number |
| Tier | Select (HOT / WARM / COLD) |
| Phone | Phone |
| Email | Email |
| Current Website | URL |
| Address | Rich text |
| Google Rating | Number |
| Reviews | Number |
| Status | Select (New / Approved / Website Built / Email Drafted / Sent / Won / Lost) |
| New Website URL | URL |
| GitHub Repo | URL |
| Scraped Data | Rich text (JSON summary) |
| Email Draft | Rich text |
| Notes | Rich text |

Sort by score descending. Set all Status to "New".

### GATE 1: User Reviews Notion

**Stop here.** Tell the user:
> "Research complete. {X} prospects found, {Y} scored HOT, {Z} scored WARM. Check the Notion database and mark the ones you want websites built for by changing their Status to 'Approved'. Let me know when you're ready for Stage 2."

**Do NOT proceed until the user says to continue.**

---

## Stage 2: BUILD (Agent builds all approved sites + drafts emails)

For every prospect with Status = "Approved" in Notion, build a website and draft an outreach email.

### 2a. Generate Design System (ui-ux-pro-max)

Run once per batch (not per prospect — all sites in a batch share a design system).

```bash
python3 ~/.agents/skills/ui-ux-pro-max/scripts/search.py "[trade] service local business professional" --design-system -p "Website Factory - [Trade]"
```

Also run supplementary searches:
```bash
python3 ~/.agents/skills/ui-ux-pro-max/scripts/search.py "[trade] landing page" --domain landing -n 3
python3 ~/.agents/skills/ui-ux-pro-max/scripts/search.py "professional service trust" --domain typography -n 3
```

Use the design system for all sites in this batch. Adapt colors per trade:
- Electricians: blue/yellow accent
- Plumbers: blue/teal
- Builders: slate/warm neutrals
- Painters: green/earth tones
- HVAC: blue/red
- Landscapers: green/brown
- Roofers: dark slate/orange

### 2b. Generate Hero Image (nano-banana)

For each prospect, generate a unique hero image based on their **location**.

```bash
nano-banana "Professional [trade] working on site in [suburb/city], [location landmark or style], golden hour lighting, editorial photography style, shallow depth of field" -s 2K -a 16:9 -o [business-slug]-hero -d [project-directory]/public/images
```

**Prompt formula:** `"Professional [trade] in [specific location detail], [time of day], editorial photo, shallow DOF"`

Location details to include:
- Known landmarks or architectural style of the area
- Climate/vegetation cues (e.g., "Brisbane subtropical", "Sydney harbour", "Melbourne laneways")
- Use WebSearch to find a distinctive visual detail about the suburb if needed

If nano-banana is unavailable (no API key / quota), use a placeholder and note it.

### 2c. Write Website Copy (copywriting skill)

For each prospect, use the **copywriting** skill principles to write:

**Hero section:**
- Headline: Use formula `"{Outcome} without {pain point}"` or `"[City]'s Most Trusted [Trade]"`
- Subheadline: Specific, benefit-focused, mentions location
- CTA: Phone number as primary action

**Services section:**
- Use scraped services if available
- If not, generate trade-appropriate services from Google Maps categories
- Each service: clear name + one-sentence benefit (not feature)

**Social proof section:**
- Pull from Google reviews data
- Pick the 3 most compelling reviews (specific outcomes, emotional language)
- Include star rating and total review count

**About section (if content available):**
- Rewrite scraped about text using copywriting principles
- Focus on trust: years in business, local, licensed, family-owned

**CTA section:**
- Use **page-cro** principles: one clear action, urgency without being pushy
- "Call now for a free quote" style — low friction

**Apply copywriting skill rules:**
- Clarity over cleverness
- Benefits over features
- Customer language over company language
- Every sentence earns its place

### 2d. Build the Website

**For each approved prospect:**

1. Create directory: `~/Documents/development/[business-name-slug]/`
2. Clone or scaffold from the template structure
3. Tech stack: **Next.js + Tailwind CSS**
4. Use the Builder template design language from `~/.agents/skills/website-factory/templates/builder-template.html`:
   - Full-viewport hero with gradient overlay on nano-banana image
   - Clean nav with business name + section links
   - Service cards on light background
   - Social proof with real Google reviews
   - Strong CTA section with rounded card
   - Minimal footer

5. Create `config/siteConfig.js` with all business data
6. Apply ui-ux-pro-max design system (colors, typography, spacing)
7. Place nano-banana hero image in `/public/images/`
8. Build and verify: `npx next build`

### 2e. Push to GitHub

For each built site:
```bash
gh repo create [github-username]/[business-slug] --public --source=. --push
```

Update Notion:
- Set "GitHub Repo" to the repo URL
- Set "New Website URL" to `https://[business-slug].vercel.app` (or whatever the deploy URL will be)
- Set Status to "Website Built"

### 2f. Draft Outreach Email (cold-email skill)

For each prospect, draft a cold email using the **cold-email** skill principles.

**Use the cold-email skill rules:**
- Write like a peer, not a vendor
- Every sentence earns its place
- Lead with their world, not yours
- One ask, low friction
- Subject line: 2–4 words, lowercase, looks internal

**Two templates based on situation:**

**No existing website:**
```
Subject: built you a site

Hey [name/there],

[BusinessName] has [X] reviews at [rating] stars on Google — your reputation is solid. But without a website, you're invisible to everyone who searches before they call.

I put one together for you: [liveUrl]

Has your real info, reviews, and services. Mobile-friendly, loads fast.

If you want it on your own domain, happy to chat. No contracts.

[senderName]
[senderPhone]
```

**Bad existing website:**
```
Subject: fresh site for [businessName]

Hey [name/there],

Had a look at [currentWebsite] — with [X] Google reviews your work speaks for itself, but your site isn't keeping up.

Built a modern version to show what's possible: [liveUrl]

Same info, just presented so it actually converts visitors into calls.

Worth a look?

[senderName]
[senderPhone]
```

**Customize each email** with specific details from their business (suburb, services, review highlights). Store the draft in the Notion "Email Draft" field. Set Status to "Email Drafted".

### GATE 2: User Reviews and Deploys

**Stop here.** Tell the user:
> "All websites built and emails drafted for {X} prospects. Check the Notion database — each has a GitHub repo link, website URL, and email draft. Deploy them through GitHub (Vercel auto-deploys on push). Review the email drafts and let me know if you want changes."

**Do NOT send emails. The user deploys and sends manually.**

---

## Stage 3: SEND (User does this)

The user:
1. Reviews each site via the deployed URL
2. Requests any tweaks
3. Sends emails manually from their own email
4. Updates Notion Status to "Sent"
5. Tracks responses in Notion (Responded / Won / Lost)

The agent can help with:
- Tweaking individual sites
- Rewriting email drafts
- Writing follow-up sequences (use **cold-email** skill's follow-up cadence)
- Updating Notion status in bulk

---

## Running Individual Steps

Users can run any part independently:

- `"discover electricians in Brisbane"` → Stage 1a only
- `"score these leads"` → Stage 1b only
- `"create the notion database"` → Stage 1d only
- `"generate a design system for plumbers"` → Stage 2a only
- `"generate a hero image for Smith Electrical in Paddington"` → Stage 2b only
- `"write copy for [business name]"` → Stage 2c only
- `"build a website for [business name]"` → Stage 2d only
- `"draft an email for [business name]"` → Stage 2f only
- `"write a follow-up for [business name]"` → Follow-up sequence

---

## Skills Used

| Skill | Used In | Purpose |
|-------|---------|---------|
| **ui-ux-pro-max** | Stage 2a | Design system: colors, typography, layout patterns, style |
| **nano-banana** | Stage 2b | AI-generated hero images unique to each prospect's location |
| **copywriting** | Stage 2c | Website copy: headlines, CTAs, service descriptions, about text |
| **page-cro** | Stage 2c | Conversion optimization: CTA placement, trust signals, structure |
| **cold-email** | Stage 2f | Outreach email drafts and follow-up sequences |
| **copy-editing** | Stage 2f | Polish email drafts before user sends |

---

## Trade-Specific Defaults

| Trade | Primary Color | Accent | Hero Image Cue |
|-------|--------------|--------|----------------|
| Electrician | #1E40AF (blue) | #F59E0B (amber) | Switchboard, wiring, modern home |
| Plumber | #0891B2 (cyan) | #10B981 (emerald) | Clean bathroom, copper pipes |
| Builder | #374151 (slate) | #D97706 (amber) | Construction site, new home frame |
| Painter | #059669 (emerald) | #8B5CF6 (violet) | Fresh painted wall, color swatches |
| HVAC | #1D4ED8 (blue) | #DC2626 (red) | AC unit, comfortable home |
| Landscaper | #15803D (green) | #92400E (brown) | Manicured garden, outdoor living |
| Roofer | #1F2937 (dark) | #EA580C (orange) | Rooftop work, aerial suburb view |

---

## Dependencies Checklist

Before running, verify:
- [ ] Apify MCP connected (`claude mcp list`)
- [ ] Notion MCP connected and authenticated
- [ ] GitHub CLI authenticated (`gh auth status`)
- [ ] nano-banana set up with Gemini API key (`~/.nano-banana/.env`)
- [ ] ui-ux-pro-max skill installed (`~/.agents/skills/ui-ux-pro-max/`)
- [ ] Node.js and npm available
- [ ] Template exists at `~/.agents/skills/website-factory/templates/builder-template.html`
