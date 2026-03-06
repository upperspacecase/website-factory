<required_reading>
Read these files before proceeding:
1. references/siteconfig-schema.md — full siteConfig structure to fill out
2. references/trade-defaults.md — default services per trade type
3. templates/cold-outreach.md — exact email template to use
</required_reading>

<objective>
For each prospect with Status = "Approved" in Notion:
clone template, customize siteConfig, build, push to GitHub, draft outreach email.
</objective>

<process>

## Step 1: Get user's name

Ask once: "What name should I sign the outreach emails with?"
Reuse for all emails in this batch.

## Step 2: For each approved prospect, repeat steps 3-8

## Step 3: Clone template repo

```bash
gh repo clone upperspacecase/adam-smith-electrician ~/Documents/development/[business-slug]
cd ~/Documents/development/[business-slug]
rm -rf .git
```

**Do not scaffold from scratch. Do not use create-next-app.**
The template is a complete, working Next.js + Tailwind site.

## Step 4: Update siteConfig.js

Edit `config/siteConfig.js` with the prospect's real data from Notion.
See references/siteconfig-schema.md for the full structure — update ALL fields.

**For services:** Use references/trade-defaults.md. Each service needs a Lucide icon name.

**For reviews:** Use Google Maps review data if available, otherwise generate 3-5 realistic reviews mentioning the location and specific services.

**For copy:** Apply the **copywriting** skill:
- Headline: specific, benefit-focused, mentions location
- Subheadline: addresses the customer's problem
- CTA: phone number, low friction

## Step 5: Generate hero image (optional)

If nano-banana is available:
```bash
nano-banana "Professional [trade] working in [location], editorial photography, golden hour" -s 2K -a 16:9 -o hero -d [project]/public/images
```
If unavailable, skip. Template works without a hero image.

## Step 6: Build and verify

```bash
npm install && npx next build
```
Must pass with zero errors before pushing.

## Step 7: Push to GitHub

```bash
git init && git add -A && git commit -m "Website for [Business Name]"
gh repo create [github-username]/[business-slug] --public --source=. --push
```
Update Notion: set GitHub Repo URL, Status = "Built".

## Step 8: Draft outreach email

Read templates/cold-outreach.md for the exact email template.

**Use Gmail MCP:**
```
Tool: mcp__gmail__create_draft
Parameters:
  to: [prospect email]
  subject: built you a site
  body: [filled template from templates/cold-outreach.md]
```

**If Gmail MCP is not authenticated**, save the draft to Notion "Email Draft" field instead.
Tell user to authenticate: `claude.ai > Settings > Connected Apps > Gmail`

Update Notion Status = "Email Drafted".

## Step 9: After all prospects are done

Tell the user:
> "Built {X} websites and drafted {X} emails. Review the drafts in Gmail and send when ready."

**Stop. User reviews drafts in Gmail and sends manually.**

</process>

<success_criteria>
- Each approved prospect has a GitHub repo with working site
- siteConfig.js has all fields populated with real data
- `next build` passes with zero errors
- Gmail draft created (or saved to Notion as fallback)
- Notion Status updated for each prospect
- User told to review and send manually
</success_criteria>
