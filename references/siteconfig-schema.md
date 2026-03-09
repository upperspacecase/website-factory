<siteconfig_schema>

Full structure of `config/siteConfig.js` in the template repo.
Update ALL fields for each prospect.

**IMPORTANT — Data deduplication rules:**
- `trustBar` only contains `credential`. Rating, review count, and years in business are derived automatically from `reviews.rating`, `reviews.totalReviews`, and `yearEstablished`.
- `reviews` does NOT contain `businessName`. The component reads from top-level `businessName`.
- `contactForm` does NOT contain `recipientEmail`. The API route reads from top-level `email`.
- Do NOT duplicate these values or the site will show stale data.

```javascript
const siteConfig = {
  // ── Business Info ──
  businessName: "[from Google Maps]",
  tagline: "[generate: Licensed [Trade] Serving [Location] Since [year]]",
  phone: "[from Google Maps]",
  phoneHref: "tel:[formatted with country code]",
  smsHref: "sms:[formatted with country code]",
  email: "[from Google Maps or generate: info@businessslug.com]",
  address: "[from Google Maps]",
  licenseNumber: "[from Google Maps or 'Fully Licensed']",
  yearEstablished: [year or estimate from reviews/age],
  hoursOfOperation: "Mon-Fri: 7am - 6pm | Sat: 8am - 2pm",
  emergencyAvailable: true,

  // ── Trust Bar ──
  // Only credential here. Rating, reviews, and years are derived automatically.
  trustBar: {
    credential: "[e.g. Master Electrician, Licensed Plumber]",
  },

  // ── Services ──
  // 6 services. Use references/trade-defaults.md for titles.
  services: [
    {
      title: "[service name]",
      description: "[1-2 sentences, benefit-focused, mentions location]",
      icon: "[Lucide icon name]",
    },
    // ... 5 more
  ],

  // ── About ──
  about: {
    headline: "[generate: trust-focused, e.g. 'Trusted. Local. Licensed.']",
    text: "[generate: 2-3 sentences about the business, location, years of experience]",
    image: "/images/team.jpg", // or null if no image
  },

  // ── Reviews ──
  // No businessName here — derived from top-level businessName.
  reviews: {
    rating: [from Google Maps],
    totalReviews: [from Google Maps],
    googleMapsUrl: "https://search.google.com/local/writereview?placeid=[PLACE_ID]",
    items: [
      {
        author: "[name]",
        rating: 5,
        date: "[e.g. '2 months ago']",
        text: "[realistic review mentioning location + specific service]",
        avatar: null,
      },
      // 3-5 reviews total
    ],
  },

  // ── Service Area ──
  serviceArea: {
    mapEmbedUrl: "[Google Maps embed URL for the location]",
    suburbs: [
      // 12-16 nearby suburbs/neighborhoods
    ],
  },

  // ── Contact Form ──
  // No recipientEmail here — derived from top-level email.
  contactForm: {
    serviceOptions: [
      // Match service titles + "Emergency Call-Out" + "Other"
    ],
  },
};

export default siteConfig;
```

</siteconfig_schema>
