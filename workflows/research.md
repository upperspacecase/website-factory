<required_reading>
Read references/trade-defaults.md for service lists per trade type.
Read references/target-cities.md for the pre-researched list of target cities.
</required_reading>

<objective>
Find [trade] businesses in [location] via Google Maps that have NO website listed.
Score by review count. Add to the existing Tradie Website CRM in Notion.
</objective>

<city_selection>
If no location is specified, pull the next city from the **Target Cities** Notion database (data source ID: `de8405d4-507e-443a-80a6-3ae604a621d2`). Pick the city with the lowest Priority number that has Status = "Not Started". After completing research for a city, update its Status to "Research Done" and set the "Prospects Found" count.
</city_selection>

<process>

## Step 1: Discover businesses (basic scrape)

**Preferred: Apify MCP** — Actor: `compass/crawler-google-places`
- Search: `[trade] in [location]`
- Use `website: "withoutWebsite"` filter to only get businesses without a website
- `scrapePlaceDetailPage: false` — keep it fast
- `maxReviews: 0` — no reviews needed yet
- `maxImages: 0`
- Set timeout to 300000 (5 minutes max)
- Extract: name, phone, address, Google rating, review count, email, placeId

**This scrape should be fast (1-2 minutes). We only need basic listing data at this stage.**

**Fallback: Web search** — if Apify unavailable, use web search to find Google Maps listings.

## Step 2: Filter

**Simple rule: Does the business have a website URL in Google Maps?**
- No website → PROSPECT (add to database)
- Has website → SKIP

Do NOT fetch or visit any URLs. Do NOT assess website quality.
Skip businesses that are not actually the target trade (e.g. a "Repair service" when searching for electricians).

## Step 3: Score

Score = review count. More reviews = more established = better prospect.

**Tiers (maps to Priority field):**
- **Hot:** 50+ reviews, no website. Established business losing leads.
- **Warm:** 10-49 reviews, no website. Growing business.
- **Cold:** Under 10 reviews, no website. May be too new/small.

## Step 4: Add to existing Notion CRM

**IMPORTANT: Do NOT create a new database. Always add rows to the existing Tradie Website CRM.**

Database: `Tradie Website CRM`
Data source ID: `80c5d3e2-b35f-4d8a-a19b-bed4b11b9f2b`

Use `mcp__claude_ai_Notion__notion-create-pages` with parent `{"data_source_id": "80c5d3e2-b35f-4d8a-a19b-bed4b11b9f2b"}`.

**Fields to populate for each prospect:**

| Field | Value |
|-------|-------|
| Business Name | From Google Maps |
| Google Reviews | Review count from Google Maps |
| Google Rating | Rating from Google Maps |
| Priority | Hot / Warm / Cold (based on tier) |
| Phone | From Google Maps |
| City | From Google Maps |
| State/Region | From Google Maps |
| Country | US / UK / Australia / Canada / New Zealand |
| Trade | Electrician / Plumber / Builder / etc. |
| Source | Google Maps |
| Pipeline Stage | Lead |
| Outreach Status | Not Contacted |
| Has Website | __NO__ |
| Website Quality | None |
| Email | From Google Maps if available |
| Owner Name | From Google Maps if available |
| Notes | Include the Google Maps placeId for use in the build phase |

## Step 5: Report and stop

Tell the user:
> "Found {X} [trade] businesses in [location] without a website. {Y} Hot, {Z} Warm. Added to the Tradie Website CRM. Set Pipeline Stage to 'Researched' for the ones you want websites built for."

**Stop. Wait for user approval before building anything.**

</process>

<success_criteria>
- Prospects added to the existing Tradie Website CRM (NOT a new database)
- Each prospect has Priority tier assigned
- Pipeline Stage set to "Lead" for all new entries
- placeId saved in Notes for build phase
- User has been told to approve prospects before next phase
</success_criteria>
