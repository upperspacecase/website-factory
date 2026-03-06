# Cold Outreach Email Template

## The Email

**Subject:** built you a site

```
Hi {{first_name}},

97% of your future customers will Google you and look at your website before they call.

So I built you a website: {{demo_site_url}}

It's yours for $1,000 USD.

Optional: I'll manage the leads for $200/month after that.

Take a look. If it's not for you, no worries.

{{your_name}}
```

## Variables

- `{{first_name}}` — Prospect's first name from Google Maps data. If unknown, use "there".
- `{{demo_site_url}}` — The deployed website URL.
- `{{your_name}}` — Your name. Ask once at start of batch, reuse for all.

## Delivery

Create as a **Gmail draft** using Gmail MCP `create_draft` tool.
If Gmail MCP unavailable, store in Notion "Email Draft" field.

## Rules

- Do not modify the template. Use it exactly as written.
- Subject line is always: `built you a site`
- No follow-up sequence unless the user asks for one.
