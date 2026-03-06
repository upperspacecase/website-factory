---
name: website-factory
description: "Two-phase system for building and selling websites to trades businesses. Phase 1 (Research): Discover Google Maps businesses without websites, rank by review count, create Notion CRM. Phase 2 (Build): Clone template repo, update siteConfig with business data, push to GitHub, draft outreach email. Trigger: 'website factory', 'find tradies', 'prospect [trade] in [location]', 'build websites for [trade]'."
---

# Website Factory

Two-skill system for finding trades businesses that need websites, then building and selling them.

---

## SKILL 1: RESEARCH

**Trigger:** "find [trade] in [location]", "prospect [trade]", "research [trade]"

### What it does:
1. Find all [trade] businesses in [location] via Google Maps
2. Filter to ONLY businesses with **no website**
3. Rank by Google review count (more reviews = more established = more likely to buy)
4. Create a Notion database with all prospects

### Discovery

**Use Apify MCP** — Actor: `compass/crawler-google-places`
- Search: `[trade] in [location]`
- Extract: name, phone, website URL, address, Google rating, review count, email, placeId

**If Apify unavailable**, use web search to find Google Maps listings.

### Filtering

**Simple rule: Does the business have a website URL in Google Maps?**
- No website → PROSPECT (add to database)
- Has website → SKIP

**Do NOT fetch or assess website quality.** Do not visit any URLs. The only signal is: does Google Maps show a website link or not.

### Scoring

Score is just the review count. More reviews = more established business = better prospect.

**Tiers:**
- **HOT:** 50+ reviews, no website. Established business losing leads.
- **WARM:** 10-49 reviews, no website. Growing business.
- **COLD:** Under 10 reviews, no website. May be too new/small.

### Notion Database

**Database name:** `[Trade] — [Location]`

| Column | Type |
|--------|------|
| Business Name | Title |
| Reviews | Number |
| Rating | Number |
| Tier | Select (HOT / WARM / COLD) |
| Phone | Phone |
| Email | Email |
| Address | Rich text |
| Status | Select (New / Approved / Built / Email Drafted) |
| GitHub Repo | URL |
| Deploy URL | URL |
| Email Draft | Rich text |

Sort by reviews descending. Status = "New" for all.

### Output

Tell the user:
> "Found {X} [trade] businesses in [location] without a website. {Y} HOT, {Z} WARM. Check the Notion database and set Status to 'Approved' for the ones you want websites built for."

**Stop. Wait for user approval before building anything.**

---

## SKILL 2: BUILD

**Trigger:** "build websites", "build for [business name]", "run website factory build"

### For each approved prospect:

#### Step 1: Clone template repo

```bash
gh repo clone upperspacecase/adam-smith-electrician ~/Documents/development/[business-slug]
cd ~/Documents/development/[business-slug]
rm -rf .git
```

The template repo is a complete, working Next.js + Tailwind site. Everything is already set up — fonts, CSS, components, build config. **Do not scaffold from scratch. Do not use create-next-app.**

#### Step 2: Update siteConfig.js

Edit `config/siteConfig.js` with the prospect's real data from the Notion database. The template has a full siteConfig structure — update ALL fields:

```javascript
const siteConfig = {
  businessName: "[from Google Maps]",
  tagline: "[generate: Licensed [Trade] Serving [Location] Since [year]]",
  phone: "[from Google Maps]",
  phoneHref: "tel:[formatted]",
  email: "[from Google Maps or generate]",
  address: "[from Google Maps]",
  licenseNumber: "[from Google Maps or leave generic e.g. 'Fully Licensed']",
  yearEstablished: [year or estimate],
  hoursOfOperation: "Mon-Fri: 7am - 6pm | Sat: 8am - 2pm",
  emergencyAvailable: true,

  trustBar: {
    googleRating: [from Google Maps],
    googleReviewCount: [from Google Maps],
    yearsInBusiness: [calculated],
    credential: "[e.g. Master Electrician, Licensed Plumber]",
  },

  services: [
    // 6 services with title, description, icon (Lucide icon name)
    // Use trade defaults below for service titles
  ],

  about: {
    headline: "[generate: trust-focused headline]",
    text: "[generate: 2-3 sentences about the business, location, experience]",
    image: null,
  },

  reviews: {
    businessName: "[from Google Maps]",
    rating: [from Google Maps],
    totalReviews: [from Google Maps],
    googleMapsUrl: "[Google Maps review URL]",
    items: [
      // 3-5 reviews with author, rating, date, text, avatar: null
      // Use real Google reviews if available, otherwise generate realistic ones
    ],
  },

  serviceArea: {
    mapEmbedUrl: "[Google Maps embed URL for the location]",
    suburbs: [
      // 12-16 nearby suburbs/neighborhoods
    ],
  },

  contactForm: {
    serviceOptions: [
      // Match the service titles + "Emergency Call-Out" + "Other"
    ],
    recipientEmail: "[same as email above]",
  },
};
```

**For services:** Use standard services for the trade type. See trade defaults below. Each service needs a Lucide icon name (e.g. House, Building2, Siren, PlugZap, BatteryCharging, Lightbulb, Wrench, Droplets, Flame, Hammer).

**For reviews:** Use Google Maps review data if available from the discovery phase, otherwise generate 3-5 realistic reviews mentioning the location and specific services.

**For copy:** Apply the **copywriting** skill principles:
- Headline: specific, benefit-focused, mentions location
- Subheadline: addresses the customer's problem
- CTA: phone number, low friction

#### Step 3: Generate hero image (optional)

If nano-banana is available:
```bash
nano-banana "Professional [trade] working in [location], editorial photography, golden hour" -s 2K -a 16:9 -o hero -d [project]/public/images
```

If unavailable, skip. The template works without a hero image.

#### Step 4: Build and verify

```bash
npm install && npx next build
```

Must pass with zero errors before pushing.

#### Step 5: Push to GitHub

```bash
git init && git add -A && git commit -m "Website for [Business Name]"
gh repo create [github-username]/[business-slug] --public --source=. --push
```

Update Notion: set GitHub Repo URL, Status = "Built".

#### Step 6: Draft outreach email in Gmail

Use the **cold-email** skill principles. Create the email as a **draft in Gmail** using the Gmail MCP `create_draft` tool.

**The email should:**
- Be under 100 words
- Lead with the live site URL (the value)
- Sound like a human, not a sales machine
- Have one clear CTA (reply or call)
- Subject line: 2-4 words, lowercase
- To: prospect's email from Google Maps data

**Use Gmail MCP:**
```
Tool: mcp__gmail__create_draft
Parameters:
  to: [prospect email]
  subject: built you a site
  body: [email body below]
```

**Email template:**
```
Hi {{first_name}},

97% of your future customers will Google you and look at your website before they call.

So I built you a website: {{demo_site_url}}

It's yours for $1,000 USD.

Optional: I'll manage the leads for $200/month after that.

Take a look. If it's not for you, no worries.

{{your_name}}
```

**Variables:**
- `{{first_name}}` — Prospect's first name (from Google Maps business owner or just "there")
- `{{demo_site_url}}` — The deployed site URL
- `{{your_name}}` — User's name (ask once, reuse for all emails)

**If Gmail MCP is not authenticated**, save the draft to Notion "Email Draft" field instead and tell the user to authenticate Gmail MCP (`claude.ai > Settings > Connected Apps > Gmail`).

Update Notion Status = "Email Drafted".

**Stop. User reviews drafts in Gmail and sends manually.**

---

## Trade Defaults

| Trade | Services |
|-------|----------|
| Electrician | Residential, Commercial, Emergency 24/7, Switchboard Upgrades, EV Chargers, Lighting |
| Plumber | General Plumbing, Hot Water, Blocked Drains, Gas Fitting, Bathroom Renos, Emergency |
| Builder | New Builds, Renovations, Extensions, Decks & Patios, Project Management, Commercial |
| Painter | Interior, Exterior, Commercial, Strata, Colour Consulting, Wallpaper |
| HVAC | Split Systems, Ducted, Repairs & Service, Commercial, Ventilation, Refrigeration |
| Landscaper | Design, Turf & Gardens, Retaining Walls, Irrigation, Paving, Maintenance |
| Roofer | Repairs, Replacement, Gutters, Metal Roofing, Tile, Inspections |

---

## Skills Used

| Skill | Where | Purpose |
|-------|-------|---------|
| **copywriting** | Build Step 2 | Website headlines, CTAs, service descriptions |
| **cold-email** | Build Step 6 | Outreach email drafts |
| **ui-ux-pro-max** | Template design | Template already designed with this. Run for custom colors if needed. |
| **nano-banana** | Build Step 3 | Hero image generation (optional) |

---

## Dependencies

- [ ] Apify MCP or web search for discovery
- [ ] Notion MCP for CRM
- [ ] Gmail MCP for email drafts (authenticate at claude.ai > Settings > Connected Apps > Gmail)
- [ ] GitHub CLI (`gh auth status`)
- [ ] Template repo at `upperspacecase/adam-smith-electrician`
- [ ] Node.js and npm
