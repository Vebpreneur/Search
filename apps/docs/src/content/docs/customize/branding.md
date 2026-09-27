---
title: Branding
description: Change your space's name, tagline, description, logo, and Open Graph image.
---

Site identity lives in `grove.config.ts` under `site`:

```ts
site: {
  name: "Open Apps",
  tagline: "Production-ready open-source applications.",
  description: "A curated, health-aware directory of open-source apps.",
  url: "https://openapps.example.com",
  repoUrl: "https://github.com/example/open-apps",
},
```

These fields flow into `<title>`, the home hero, `sitemap.xml`, `llms.txt`, the JSON-LD `WebSite` document, and the OG card.

## Logo and favicon

Drop the files in `public/`, then point at them from `site`:

```ts
site: {
  name: "Open Apps",
  logo: "/logo.svg",        // shown beside the site name in the header
  favicon: "/favicon.svg",  // browser tab icon
},
```

```
public/
├── logo.svg      # header logo
└── favicon.svg   # browser tab icon
```

Both fields are optional and there is no filename convention behind them — Grove renders exactly what you point it at (`site.logo`/`site.favicon` can be any path under `public/`). Leave `logo` unset and the header shows a neutral mark next to the site name (`src/layouts/header.astro`); leave `favicon` unset and `<link rel="icon">` gets a generated square data-URI tinted with your `theme.primaryColor` (`src/layouts/base-layout.astro`).

Grove only emits that one `<link rel="icon">` tag — it does not look for `favicon.ico` or `apple-touch-icon.png` by convention. Add those yourself under `public/` (and the matching `<link>` tags) if you want the legacy-browser and iOS home-screen affordances.

If you only ship one file, ship `logo.svg`.

## OG image

`grove check` (and every `astro dev`/`astro build`, since both run the same `prepareDirectory` pipeline) generates `public/og-image.svg` (1200×630) from your `site` block and `theme.primaryColor`. The file carries an ownership marker comment; edit the file yourself (or just delete the marker comment) and Grove stops regenerating it — your version wins on every future build (`packages/core/src/site-artifacts.ts`).

Every page also gets a PNG card: the build renders `public/og/**` with satori (home, records, collections, taxonomy pages), and the page models point `og:image` at the page's own card, falling back to `/og/default.png`. `/og-image.svg` is the site-wide fallback when no PNG exists. To use your own image on a page, pass it as the `image` prop to `BaseLayout`; dropping a file at `public/og-image.png` alone does nothing.

## What NOT to put in `site`

- No HTML — `name` is rendered as text, not parsed.
- No URLs in `name` — URLs belong in `site.url`.
- No marketing copy in `description` — it's quoted verbatim in `llms.txt`.

## Related

- [Theme](/customize/theme/) — colour tokens
- [Images and assets](/customize/assets/) — icons, favicons
- [SEO outputs](/outputs/seo/) — full OG + JSON-LD reference