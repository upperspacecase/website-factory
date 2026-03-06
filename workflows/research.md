<required_reading>
Read references/trade-defaults.md for service lists per trade type.
</required_reading>

<objective>
Find [trade] businesses in [location] via Google Maps that have NO website listed.
Score by review count. Save to Notion CRM.
</objective>

<process>

## Step 1: Discover businesses

**Preferred: Apify MCP** — Actor: `compass/crawler-google-places`
- Search: `[trade] in [location]`
- Extract: name, phone, website URL, address, Google rating, review count, email, placeId

**Fallback: Web search** — if Apify unavailable, use web search to find Google Maps listings.

## Step 2: Filter

**Simple rule: Does the business have a website URL in Google Maps?**
- No website → PROSPECT (add to database)
- Has website → SKIP

Do NOT fetch or visit any URLs. Do NOT assess website quality.
The only signal is: does Google Maps show a website link or not.

## Step 3: Score

Score = review count. More reviews = more established = better prospect.

**Tiers:**
- **HOT:** 50+ reviews, no website. Established business losing leads.
- **WARM:** 10-49 reviews, no website. Growing business.
- **COLD:** Under 10 reviews, no website. May be too new/small.

## Step 4: Create Notion database

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

## Step 5: Report and stop

Tell the user:
> "Found {X} [trade] businesses in [location] without a website. {Y} HOT, {Z} WARM. Check the Notion database and set Status to 'Approved' for the ones you want websites built for."

**Stop. Wait for user approval before building anything.**

</process>

<success_criteria>
- Notion database created with all no-website prospects
- Sorted by review count descending
- Each prospect has tier assigned
- User has been told to approve prospects before next phase
</success_criteria>
