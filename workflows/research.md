<required_reading>
Read references/trade-defaults.md for service lists per trade type.
</required_reading>

<objective>
Find [trade] businesses in [location] via Google Maps that have NO website listed.
Score by review count. Add to the local CRM at ~/Documents/development/website-factory/crm.json.
</objective>

<city_selection>
If no location is specified, read the `cities` array in crm.json. Pick the next city with `status: "not_started"`. After completing research for a city, update its status to `"research_done"` and set `prospectsFound`.
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
- No website → PROSPECT (add to CRM)
- Has website → SKIP

Do NOT fetch or visit any URLs. Do NOT assess website quality.
Skip businesses that are not actually the target trade (e.g. a "Repair service" when searching for electricians).

## Step 3: Score

Score = review count. More reviews = more established = better prospect.

**Tiers (maps to priority):**
- **hot:** 50+ reviews, no website. Established business losing leads.
- **warm:** 10-49 reviews, no website. Growing business.
- **cold:** Under 10 reviews, no website. May be too new/small.

## Step 4: Add to local CRM

Read `~/Documents/development/website-factory/crm.json`, then append new prospects to the `prospects` array.

**Each prospect object:**

```json
{
  "id": "[business-slug]",
  "businessName": "[from Google Maps]",
  "phone": "[from Google Maps]",
  "email": "[from Google Maps or empty]",
  "address": "[from Google Maps]",
  "city": "[city, state]",
  "trade": "[Electrician/Plumber/etc]",
  "placeId": "[from Google Maps]",
  "googleRating": [number],
  "googleReviews": [number],
  "ownerName": "",
  "pipelineStage": "lead",
  "outreachStatus": "not_contacted",
  "repo": "",
  "demoUrl": "",
  "vercelUrl": null,
  "emailSentDate": null,
  "lastContactDate": null,
  "notes": ""
}
```

Also update the city record in the `cities` array:
- `status`: `"research_done"`
- `prospectsFound`: count of prospects added

Write the updated crm.json back. **This is 1 read + 1 write — much cheaper than Notion.**

## Step 5: Report and stop

Tell the user:
> "Found {X} [trade] businesses in [location] without a website. {Y} Hot, {Z} Warm. Added to crm.json. Review at http://localhost:3456 and tell me when you're ready to build."

**Stop. Wait for user approval before building anything.**

</process>

<success_criteria>
- Prospects added to crm.json prospects array
- Each prospect has correct fields populated
- placeId saved for build phase enrichment
- City status updated in crm.json
- User told to review and approve before next phase
</success_criteria>
