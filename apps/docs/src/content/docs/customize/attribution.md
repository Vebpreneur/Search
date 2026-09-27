---
title: Attribution and credit
description: Make the projects you list see your directory in their analytics, give maintainers a README badge, credit the people who submit entries, and show where your site was featured.
---

A directory sends readers to other people's projects and takes submissions from other people. Grove makes both visible: listed projects can tell the traffic came from you, maintainers get a badge that links back, contributors are credited on the pages they added, and your own press shows on the home page.

## Outbound links carry your site

Every link that leaves a Grove site uses `rel="noopener"`, never `noreferrer`, so the browser sends your site as the referrer. That is what GitHub's **Insights → Traffic → Referring sites** and most analytics tools read.

Links to a record's own site — the homepage button on a record page, "Website" on collection picks — also carry `ref=<your host>`. Plausible, GA, Umami and similar tools read `ref` as the source when the browser sends no referrer:

```
https://anarlog.so/download?ref=openappscout.com
```

Forges, app stores and package registries are never tagged (GitHub, GitLab, Codeberg, the App Store, Google Play, F-Droid, Flathub, Snapcraft, npm, PyPI, crates.io, Docker Hub, browser extension stores). A URL that already has `ref` or `utm_source` is left as it is.

The value defaults to the host of `site.url`. Change or turn it off in `grove.config.ts`:

```ts title="grove.config.ts"
outbound: {
  ref: "example.dev",            // or false to add nothing
  skipHosts: ["docs.example.com"], // extra hosts (and their subdomains) to leave alone
},
```

In your own components, use the same rule through `withRef` from `@grove-dev/core`, with the resolved settings from `site-config.json`:

```astro
---
import siteConfig from "@grove/generated/site-config.json";
import { withRef } from "@grove-dev/core";
---
<a href={withRef(record.links?.website, siteConfig.outbound)} rel="noopener" target="_blank">Website</a>
```

## A README badge for maintainers

Every build writes two SVGs, `public/badges/featured.svg` and `public/badges/featured-dark.svg`, reading "Featured on <site name>". Like `robots.txt`, they start with a `grove-generated` marker line: remove it and the file is yours, and Grove stops rewriting it.

The record page's sidebar shows the badge with **Copy Markdown** and **Copy HTML** buttons. The snippet links to the record with `?ref=badge`, so badge clicks are visible in your analytics, and the HTML version switches to the dark badge through `<picture>`, which GitHub READMEs respect:

```markdown
[![Ollama on Example Directory](https://example.dev/badges/featured.svg)](https://example.dev/projects/ollama/?ref=badge)
```

The component is the registry's `src/components/site/featured-badge.astro`.

## Credit for submitters

A record can name the GitHub login of the person who submitted it:

```yaml
submittedBy: octocat   # a leading @ is accepted and stripped
```

The value must be a valid GitHub login, or `grove check` fails. The `/submit/` form writes it when the contributor fills in their username.

- The record page shows **Submitted by @octocat** with the avatar, linking to that person on `/contributors/`.
- `/contributors/` gets an **Added by the community** section: one card per person with the entries they added, most first. Each card's `id` is the lowercased login, so `/contributors/#octocat` scrolls to it.

The section comes from `getSubmissionsBySubmitter(siteConfig)` in `@grove-dev/astro/server`, which groups visible records by `submittedBy`.

## Where your site was featured

List press mentions under `site.press`, newest first. Only list one you can link to:

```ts title="grove.config.ts"
site: {
  name: "Example Directory",
  press: [
    {
      outlet: "Astro",
      title: "What's new in Astro — August 2026",
      url: "https://astro.build/blog/whats-new-august-2026/",
      date: "2026-08",                               // YYYY-MM
      label: "Featured in Astro's August 2026 roundup", // optional hero text
      logo: "/icons/brands/astro.svg",                // optional, under public/
    },
  ],
},
```

The home page's `HeroProof` block (`src/components/grove/hero-proof.astro`) puts the newest mention above the headline, and shows contributor faces, GitHub stars and the mention under the call to action. `/about` lists every mention. With an empty list the press parts are simply left out.

## Related

- [Config reference → `outbound`](/reference/config/#outbound) and [`site`](/reference/config/#site)
- [Record schema → `submittedBy`](/reference/record-schema/#submittedby)
- [Community submissions](/automation/submissions/)
- [Outputs overview](/outputs/overview/) — the badge SVGs among the generated files
