# astro-performance-budget

A drop-in **Lighthouse CI performance budget** and **GitHub Actions workflow** for fast [Astro](https://astro.build) marketing sites. Copy two files into your project and every pull request is checked against real Core Web Vitals and resource-size budgets, so a performance regression fails review before it ever reaches a visitor.

No dashboard to sign up for, no service to pay for, no telemetry. It is a `lighthouserc.json`, a mobile companion config, and a `lighthouse.yml` workflow that you own and can edit.

- **`lighthouserc.json`** - desktop budget: category scores, Core Web Vitals, and per-resource size ceilings.
- **`lighthouserc.mobile.json`** - the same discipline under mobile CPU and network throttling.
- **`.github/workflows/lighthouse.yml`** - builds your Astro site, serves the static output, and asserts both budgets on push and pull request.
- **`budget.json`** - the same ceilings in Lighthouse's native budget format, for path-scoped overrides.
- **[`BUDGETS.md`](./BUDGETS.md)** - every number explained, with guidance on when and how to move it.

Built and maintained by [TicketWave HQ](https://ticketwavehq.com). [VerySEOly](https://veryseoly.com) holds the sites it builds to this budget, so a regression fails the pull request rather than reaching a visitor.

## Why a performance budget

A great Lighthouse score on launch day is easy. Keeping it after twenty commits, a new analytics tag, an unoptimised hero image, and a font swap is the hard part. A performance budget turns "we should keep the site fast" into an assertion that either passes or fails, on every change, in the pull request where you can still do something about it.

Astro is an unusually good fit. Because it ships zero client-side JavaScript by default and hydrates only the interactive islands you ask for, a well-built Astro marketing page starts far inside these budgets. That means the budget is not fighting the framework, it is protecting the head start the framework gives you.

## Quick start

From the root of your Astro project:

```bash
# 1. Copy the budget configs and the workflow into your repo.
curl -O https://raw.githubusercontent.com/sansware/astro-performance-budget/main/lighthouserc.json
curl -O https://raw.githubusercontent.com/sansware/astro-performance-budget/main/lighthouserc.mobile.json
mkdir -p .github/workflows
curl -o .github/workflows/lighthouse.yml \
  https://raw.githubusercontent.com/sansware/astro-performance-budget/main/.github/workflows/lighthouse.yml

# 2. Install the Lighthouse CI runner locally (optional, for running it yourself).
npm install -D @lhci/cli
```

Add a couple of scripts to your `package.json` (see [`examples/package.scripts.json`](./examples/package.scripts.json)):

```json
{
  "scripts": {
    "lhci": "lhci autorun --config=./lighthouserc.json --collect.staticDistDir=./dist",
    "perf": "astro build && npm run lhci"
  }
}
```

Then run it locally before you push:

```bash
npm run perf
```

Commit the two config files and the workflow, open a pull request, and the **Lighthouse CI** check appears alongside your other status checks. If a budget is broken, the check fails and the job log links to the full report.

## What it checks

The desktop budget in `lighthouserc.json` asserts, among others:

| Signal | Budget (desktop) | Budget (mobile) |
| --- | --- | --- |
| Performance score | 0.95 | 0.90 |
| Accessibility score | 0.95 | 0.95 |
| Best practices score | 0.95 | 0.95 |
| SEO score | 0.95 | 0.95 |
| Largest Contentful Paint | 2000 ms | 2800 ms |
| Cumulative Layout Shift | 0.05 | 0.10 |
| Total Blocking Time | 150 ms | 250 ms |
| First Contentful Paint | 1500 ms | 2200 ms |
| Total page weight | 600 KB | 600 KB |
| Script weight | 120 KB | 120 KB |
| Third-party requests | 5 (warn) | 5 (warn) |

It also fails the build on unminified CSS or JavaScript, missing text compression, missing `font-display`, and any uncaught console error in production. The full list, with the reasoning behind every threshold, is in [`BUDGETS.md`](./BUDGETS.md).

These defaults are deliberately tighter than Google's "good" Core Web Vitals thresholds (LCP 2500 ms, CLS 0.1). A static Astro marketing page has the headroom, so the budget holds it to a higher standard than the bare minimum.

## How the workflow runs

`lighthouse.yml` does the following on every push to `main` and every pull request:

1. Checks out the repository and sets up Node 20 with npm caching.
2. Runs `npm ci` and `npm run build` to produce the production `dist/` output.
3. Installs `@lhci/cli` and runs `lhci autorun` against the static build, three times per URL, taking the median.
4. Asserts the desktop budget, then the mobile budget.
5. Uploads the reports to Lighthouse CI's temporary public storage and links them in the job log.

Because it serves the built `dist/` directory with LHCI's own static server, there is nothing else to spin up and no flaky `dev` server timing to fight. The run is deterministic and self-contained.

## Configuring for your site

**Test specific pages.** By default the workflow measures the built site's routes as served. To pin exact URLs, add a `url` array under `collect` in `lighthouserc.json`, or point `staticDistDir` at your build output and let LHCI discover the HTML files.

**Loosen a single page, not the whole site.** If one page genuinely needs more weight, for example a landing page with a background video, use a path-scoped entry in `budget.json` rather than raising the global ceiling. That keeps the rest of the site honest.

**Promote warnings to errors.** Several diagnostics ship as `warn` so you can adopt the budget without an immediate red build. Once your baseline is clean, switch `render-blocking-resources`, `unused-javascript`, and `modern-image-formats` to `error` to lock in the gains.

**Turn off the PR comment.** The workflow includes an optional step that comments a one-line summary on the pull request. Delete that step, or drop the `pull-requests: write` permission, if you do not want the bot to post.

## Tuning tips for Astro marketing sites

These are the changes that most often move a real Astro page back inside budget:

- **Serve modern image formats through Astro's image pipeline.** Use the `<Image />` component or `getImage()` so hero and content images ship as AVIF or WebP with correct dimensions, which protects both the image budget and Cumulative Layout Shift.
- **Preload the LCP image and give it explicit width and height.** The hero is almost always the Largest Contentful Paint element on a marketing page.
- **Subset and self-host fonts, and set `font-display: swap`.** Two `woff2` families with a couple of weights each fit comfortably inside the 120 KB font budget. The budget fails if text is invisible while a font loads.
- **Keep islands small and few.** Every `client:*` directive adds JavaScript. Prefer `client:visible` or `client:idle` over `client:load` where you can, and question whether a component needs to hydrate at all.
- **Move third-party tags behind consent or defer them.** Each tag counts against the third-party request budget and usually against Total Blocking Time. Cookie-free, privacy-respecting analytics keep this clean.
- **Enable Brotli or gzip at the edge.** The `uses-text-compression` audit is an `error`, so uncompressed HTML, CSS, or JS fails the build.

## Requirements

- An Astro project with a `build` script that outputs to `dist/` (the Astro default).
- Node 18 or newer. The workflow uses Node 20.
- A GitHub repository with Actions enabled.

There is no runtime dependency on this repository. You copy the files in and they are yours; update them from here whenever you want the latest defaults.

## Related work

- **[VerySEOly](https://veryseoly.com)** - websites built and run for UK businesses, held to this budget as standard rather than as an afterthought.
- **[TicketWave HQ, Growth & Digital](https://ticketwavehq.com/practices/growth-and-digital)** - the practice this budget belongs to.

## Contributing

Issues and pull requests are welcome, especially real-world budget numbers from Astro sites in production and additional audit assertions worth enabling by default. Keep changes framework-honest: the goal is a budget that a well-built static site can meet, not one that is impossible to pass.

## License

[MIT](./LICENSE) © Sansware. Use it, fork it, ship it.
