---
title: Community submissions
description: The on-site /submit/ page walks a contributor through three steps to a pre-filled GitHub pull request; a human always reviews and merges.
---

Grove's submission surface is the scaffolded `/submit/` page. There is no submission bot and no automated issue-to-PR pipeline in this repository — the page is a client-side draft generator that hands the contributor a pre-filled GitHub link, and every record still lands as a pull request a maintainer reviews.

## How the submit page works

The page is the registry's `src/pages/submit.astro`; the behaviour lives in `src/components/grove/submission-client.astro`. It is three numbered steps, shown as a stepper at the top that turns green as each one is done.

1. **Repository.** The contributor pastes a GitHub URL and presses **Fetch details** (or Enter). By default — the static-build path the scaffold ships — this is a **direct browser call** to `https://api.github.com/repos/<owner>/<repo>`: no server, no token. A `404` shows "Repository not found or not public."; a `403` or `429` shows "GitHub rate limit reached (60 requests/hour per visitor). Try again later or fill the fields manually."; a private repository is rejected ("Private repositories cannot be submitted."). On success a repository card appears with the owner avatar, stars, the detected licence and the last push.

   `SubmissionClient` also accepts an optional `githubProxyPath` prop so an SSR consumer can route the call through a server endpoint that reads a server-only `GITHUB_TOKEN` (see `packages/astro/src/server/github-repo.ts`). The scaffolded page does not pass it.

2. **Details.** The fetch fills in name, page address (slug), description, primary stack (guessed from language/topics, and only when the guess is a real taxonomy id), tags (up to 8 GitHub topics) and website. The contributor chooses category and platforms and can add their **GitHub username**, which becomes `submittedBy` and credits them on the page. Tags, licence, website and "Best for" sit behind **Add more detail**. Category, stack, platforms, tags and licence only appear when the site's `browse.facets` enables them.

3. **Preview and open the pull request.** A live card shows how the entry will look, and a checklist ticks off each requirement as it is met:
   - the repository was fetched and the slug is not already taken (`existingSlugs`),
   - the description is at least 40 characters,
   - category and stack (if enabled) come from the site's taxonomy,
   - at least one platform is checked (if enabled),
   - the GitHub username, if given, is a valid login.

   While anything fails, the buttons stay disabled and every issue is listed. Once valid, **Open pull request on GitHub** opens `<repoUrl>/new/main?filename=data/records/<slug>.yml&value=<yaml>` — GitHub's own "create new file" editor, pre-filled, which takes a contributor without push access through fork, commit and pull request. **Copy the file instead** copies the draft; the file itself is behind "Show the file".

The draft has this shape (fields present depend on which `fields.*` `getSubmissionPageModel` enables):

```yaml
kind: project
slug: ollama
addedAt: 2026-09-27
submittedBy: octocat
name: "Ollama"
description: "Get up and running with large language models locally."
category: ai
projectType: real-app
stack: "go"
platforms:
  - macos
  - linux
tags:
  - llm
  - local-llm
licenses:
  - mit
repoUrl: https://github.com/ollama/ollama
links:
  github: https://github.com/ollama/ollama
  website: https://ollama.com
bestFor:
  []
source:
  type: manual
  owner: ollama
  repo: ollama
curation:
  reviewed: false
  labels: []
  lenses: []
```

`addedAt` is always today's date (it orders "Recently added"); `submittedBy` appears only when the contributor gave a username. `projectType: real-app` is hardcoded — the form doesn't expose other project types. A site that keeps its records as Markdown (`content/records/<slug>.md`) can adapt the client to write that file instead; [Open App Scout](https://github.com/tortuvshin/open-apps) does.

## The submission copy

`grove.config.ts`'s `submission` block drives the page's headline, description, and the "good submissions" / "please avoid" lists (`copy.good` / `copy.avoid`), consumed via `getSubmissionPageModel` in `packages/astro/src/server/models.ts`:

```ts
submission: {
  eyebrow: "AI project submission",
  title: "Add an open-source AI project",
  description:
    "Generate a Grove record from a public GitHub repository, review the AI taxonomy, then open a pull request.",
  good: [
    "A usable open-source AI tool, agent framework, interface, or infrastructure project",
    "A public repository with a clear license and enough documentation to evaluate",
    "A category, stack, and tags chosen from this directory's taxonomy",
  ],
  avoid: [
    "Closed-source AI products or marketing-only landing pages",
    "Prompt collections, tutorials, snippets, or duplicate entries",
    "Abandoned experiments without documentation or a verifiable license",
  ],
},
```

If any field is omitted, `submit.astro` falls back to generic copy hardcoded in the page itself, not to anything from `@grove-dev/core`.

## The freeform issue template

`grove init` writes no `.github/` directory. The example site in the Grove repo has `apps/example/.github/ISSUE_TEMPLATE/record_submission.md` — copy it if you want one. It is a plain Markdown issue template (not a GitHub Issue Forms schema) for contributors who'd rather describe a suggestion than fill out the on-site form. It asks for the same broad shape of information — name, description, category, stack, platforms, project type, links, and a rationale — as free-text fields under Markdown headings, plus a small checklist (public repo, OSI license, maintained in the last 12 months, author-disclosure).

Nothing automated reads this template. Opening an issue with it does not generate a YAML draft, does not comment back, and does not open a PR — a maintainer reads the issue and, if it's in scope, either writes the record by hand or asks the submitter to use `/submit/` instead. `.github/ISSUE_TEMPLATE/bug_report.md` and `feature_request.md` are separate, unrelated templates for site bugs and feature requests.

## Review flow

Once a PR exists — opened from `/submit/`, hand-written from a copied draft, or opened directly — the example site's `apps/example/.github/workflows/ci.yml` (copy it; `grove init` writes no workflows) runs on every PR and push to `main`:

1. `pnpm install --frozen-lockfile`
2. `pnpm exec grove check` — schema validation against every record (including the new one), regenerating artifacts, and (internally) running `astro check`
3. `pnpm build` — the full Astro build

A red CI run blocks merge in the usual GitHub sense (branch protection has to be configured for that; the workflow itself just reports status). Once merged, the new record has no GitHub data until the next [`grove sync github`](/automation/sync-github/) run writes its cache entry (`data/cache/github/<slug>.json`); the example workflow runs weekly. A `health` entry is *not* filled in by anything; see [Maintain health signals](/content/health-classification/).

## What's not automated

- No bot writes, comments on, or closes issues.
- No bot merges PRs. A human always reviews and merges.
- No auto-labeling beyond the static `labels: ["submission"]` on the issue template's own frontmatter.
- `CODEOWNERS` is not part of the scaffolded `apps/example/` site — this repository's own `.github/CODEOWNERS` covers the Grove monorepo itself, not a site built with Grove. If you want required review on `data/records/`, add a `CODEOWNERS` entry yourself.

## Spam and quality gates

- The issue template's `labels: ["submission"]` frontmatter tags every issue opened from it, which you can use to filter or triage.
- `grove check` in CI rejects a PR whose record YAML doesn't match the schema — malformed submissions fail the build rather than merging silently.
- Branch protection requiring the CI check to pass (a GitHub repo setting, not something Grove configures) is the mechanism that actually blocks a bad PR from merging.
- A `CODEOWNERS` entry for `data/records/` (a plain GitHub feature) restricts who can approve changes there — add it if you want it; it isn't shipped by default.

## Credit after merge

The record page shows "Submitted by @login" from `submittedBy`, and `/contributors/` lists what each person added. Maintainers of the listed project get a "Featured on" README badge from the record's sidebar. See [Attribution and credit](/customize/attribution/).

## Related

- [Record schema](/reference/record-schema/) — every field a contributor's draft needs to satisfy
- [`grove check`](/automation/check/) — what the CI gate validates
- [Sync GitHub metadata](/automation/sync-github/) — the enrichment that runs after a record merges
- [Contributing](/maintainers/contributing/) — the contributor-facing walkthrough of this same flow
