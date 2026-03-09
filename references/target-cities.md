# Target Cities

The list of 20 target US cities is stored in the `cities` array in `~/Documents/development/website-factory/crm.json`.

## How to use

When running Phase 1 (Research) with no location specified:
1. Read crm.json
2. Find the next city with `status: "not_started"`
3. Run the research workflow for that city
4. Update the city's `status` to `"research_done"` and set `prospectsFound`
5. After build phase, update `sitesBuilt`

## Selection criteria (for adding more cities later)

Cities were selected based on:
- **Population:** 100k-225k (mid-size — enough electricians, few with websites)
- **Growth:** High population growth rate, construction booms
- **Construction activity:** Building permits, new housing starts
- **Trades demand:** Blue-collar economies, military bases, post-disaster rebuild
- **Geographic spread:** Avoid clustering in one region

Sources used: Census Bureau population data, NAHB building permit data, construction employment trends, regional growth analyses (March 2026).

## Current progress

- **Cities researched:** 1 / 20
- **Last city completed:** Huntsville, AL (20 sites built)
- **Next city:** Boise, ID
