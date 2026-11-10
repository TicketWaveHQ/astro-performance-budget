# astro-performance-budget

**What it is:** a drop-in **Lighthouse CI performance budget** and **GitHub Actions workflow** for fast [Astro](https://astro.build) marketing sites. Copy three files into your project and every pull request is checked against Core Web Vitals and resource-size budgets, so a performance regression fails review before it ever reaches a visitor.

**Who it is for:** teams and freelancers who build static or mostly static Astro sites (brochure sites, landing pages, small marketing sites) and want "keep it fast" to be a check that passes or fails, not a good intention.

No dashboard to sign up for and no service to pay for. It is a `lighthouserc.json`, a mobile companion config, and a workflow file that you own and can edit.

## 60-second start

From the root of your Astro project (one that builds to `dist/` with `npm run build` and has a `package-lock.json`):

```bash
curl -O https://raw.githubusercontent.com/TicketWaveHQ/astro-performance-budget/main/lighthouserc.json
curl -O https://raw.githubusercontent.com/TicketWaveHQ/astro-performance-budget/main/lighthouserc.mobile.json
mkdir -p .github/workflows
curl -o .github/workflows/lighthouse.yml \
  https://raw.githubusercontent.com/TicketWaveHQ/astro-performance-budget/main/templates/lighthouse.yml

# Optional: run the desktop budget locally first, keeping the reports on disk.
npm run build && npx @lhci/cli@0.15.x autorun --config=./lighthouserc.json \
  --collect.staticDistDir=./dist --upload.target=filesystem --upload.outputDir=./lhci_reports
```

Commit the three files and open a pull request. The **Assert performance budget** check appears alongside your other status checks; if a budget is broken, it fails and the job log links to the full report.

## What is in this repo

- **`lighthouserc.json`** - desktop budget: category scores, Core Web Vitals, and per-resource size ceilings.
- **`lighthouserc.mobile.json`** - the same discipline under mobile CPU and network throttling.
- **`templates/lighthouse.yml`** - the workflow to copy into your project: builds your Astro site, serves the static output, and asserts both budgets on push and pull request.
- **`budget.json`** - the same ceilings in Lighthouse's budget format, as an alternative to the assertions (see [Using budget.json instead](#using-budgetjson-instead)).
- **`examples/package.scripts.json`** - `package.json` scripts for running the budgets locally.
- **[`BUDGETS.md`](./BUDGETS.md)** - every number explained, with guidance on when and how to move it.
- **`test/fixture/`** and **`.github/workflows/ci.yml`** - how this repository tests itself (see [How this repository is tested](#how-this-repository-is-tested)).

Built and maintained by [TicketWave HQ](https://ticketwavehq.com). [VerySEOly](https://veryseoly.com) holds the sites it builds to this budget, so a regression fails the pull request rather than reaching a visitor.

## Why a performance budget

A great Lighthouse score on launch day is easy. Keeping it after twenty commits, a new analytics tag, an unoptimised hero image, and a font swap is the hard part. A performance budget turns "we should keep the site fast" into an assertion that either passes or fails, on every change, in the pull request where you can still do something about it.

Astro is an unusually good fit. Because it ships zero client-side JavaScript by default and hydrates only the interactive islands you ask for, a well-built Astro marketing page starts far inside these budgets. That means the budget is not fighting the framework, it is protecting the head start the framework gives you.

## Running it locally

Add a couple of scripts to your `package.json` (see [`examples/package.scripts.json`](./examples/package.scripts.json)):

```json
{
  "scripts": {
    "lhci": "lhci autorun --config=./lighthouserc.json --collect.staticDistDir=./dist",
    "perf": "astro build && npm run lhci"
  }
}
```

Then install the runner and run it before you push:

```bash
npm install -D @lhci/cli@0.15.x
npm run perf
```

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
| Third-party requests | 5 (warn) | not asserted |

It also fails the build on unminified CSS or JavaScript, missing text compression, missing `font-display`, and any uncaught console error in production. The full list, with the reasoning behind every threshold, is in [`BUDGETS.md`](./BUDGETS.md).

These defaults are deliberately tighter than Google's "good" Core Web Vitals thresholds (LCP 2500 ms, CLS 0.1). A static Astro marketing page has the headroom, so the budget holds it to a higher standard than the bare minimum.

## How the workflow runs

`templates/lighthouse.yml` does the following on every push to `main`, every pull request, and on demand:

1. Checks out the repository and sets up Node 24 with npm caching.
2. Runs `npm ci` and `npm run build` to produce the production `dist/` output.
3. Installs `@lhci/cli@0.15.x` and runs `lhci autorun` against the static build, three times per URL.
4. Asserts the desktop budget, then the mobile budget.
5. Uploads the reports to Lighthouse CI's temporary public storage and links them in the job log.

Because it serves the built `dist/` directory with LHCI's own static server, there is nothing else to spin up and no flaky `dev` server timing to fight.

Two defaults worth knowing:

- **Each assertion uses the most favourable of the three runs.** That is LHCI's default `optimistic` aggregation, which keeps one noisy run from failing the build. To assert on the median run instead, add `"aggregationMethod": "median-run"` under `assert` in each config.
- **Reports are public.** Anyone with the link can open a report in temporary public storage. If your pages are not public, set `"target": "filesystem"` under `upload` and keep the reports on the runner.

The workflow pins `@lhci/cli@0.15.x`, which runs Lighthouse 12. Every audit these configs assert exists in that version; this repository's CI checks that on every change. Newer LHCI releases move to a newer Lighthouse, so test the configs before you upgrade.

## Configuring for your site

**Choose the pages.** With `staticDistDir` and no `url` list, LHCI tests up to five HTML files it finds in `dist/`, at most two folders deep. To choose pages, add a `url` array under `collect` with paths such as `"/"` and `"/pricing/"`; LHCI serves them from `dist/` and fills in the port. To test more pages without listing them, raise `maxAutodiscoverUrls` (`0` removes the limit).

**Loosen a single page, not the whole site.** If one page genuinely needs more weight, for example a landing page with a background video, replace `assertions` with an `assertMatrix` of two entries rather than raising the global ceiling. Entries apply independently, so the strict entry must exclude the heavy page:

```json
"assert": {
  "assertMatrix": [
    {
      "matchingUrlPattern": "^(?!.*/video-landing/).*$",
      "assertions": { "resource-summary:total:size": ["error", { "maxNumericValue": 600000 }] }
    },
    {
      "matchingUrlPattern": "/video-landing/",
      "assertions": { "resource-summary:total:size": ["error", { "maxNumericValue": 1500000 }] }
    }
  ]
}
```

Put the full set of assertions in each entry; an `assertMatrix` cannot sit alongside a top-level `assertions` block.

**Promote warnings to errors.** Several diagnostics ship as `warn` so you can adopt the budget without an immediate red build. Once your baseline is clean, switch `render-blocking-resources`, `unused-javascript`, and `modern-image-formats` to `error` to lock in the gains.

**Using pnpm or Yarn.** Change `cache: npm` and `npm ci` in the workflow to your package manager's equivalents.

### Using budget.json instead

`budget.json` holds the same ceilings in Lighthouse's budget format. LHCI can assert with it **instead of** the `assertions` in `lighthouserc.json`, never alongside them (it stops with "Cannot use both budgets AND assertions"). To try it, run LHCI without the rc file:

```bash
npx @lhci/cli@0.15.x collect --no-lighthouserc --staticDistDir=./dist
npx @lhci/cli@0.15.x assert --no-lighthouserc --budgetsFile=./budget.json
```

The workflow uses the assertions, which cover more than `budget.json` can express (category scores, console errors, `font-display`, minification).

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
- npm with a committed `package-lock.json`, or the workflow adjusted for your package manager.
- A Node.js version your Astro release supports. The workflow uses Node 24.
- A GitHub repository with Actions enabled.

There is no runtime dependency on this repository. You copy the files in and they are yours; update them from here whenever you want the latest defaults.

## How this repository is tested

This repository has no Astro site, so its own CI ([`.github/workflows/ci.yml`](.github/workflows/ci.yml)) runs both budgets against the two hand-written static pages in [`test/fixture/`](test/fixture/), with `@lhci/cli@0.15.x`. It then adds a console error to a copy of one page and checks that the desktop budget fails. That proves the configs load, every asserted audit exists, and the assertions bite. It does not measure an Astro build.

## Related work

- **[VerySEOly](https://veryseoly.com)** - websites built and run for UK businesses, held to this budget as standard rather than as an afterthought.
- **[TicketWave HQ, Digital practice](https://ticketwavehq.com/practices/digital)** - the practice this budget belongs to.
- **[TicketWaveHQ/website-care-checklist](https://github.com/TicketWaveHQ/website-care-checklist)** - the monthly care checklist, including a Lighthouse run against the live site.

## Contributing

Issues and pull requests are welcome, especially real-world budget numbers from Astro sites in production and additional audit assertions worth enabling by default. Keep changes framework-honest: the goal is a budget that a well-built static site can meet, not one that is impossible to pass. See [`CONTRIBUTING.md`](./CONTRIBUTING.md); security reports go through [`SECURITY.md`](./SECURITY.md).

## Author

Written and maintained by [Jordan Gilbert](https://veryseoly.com/jordan) ([@eu-jordangilbert](https://github.com/eu-jordangilbert)), and published by [TicketWave HQ Ltd](https://ticketwavehq.com) ([@TicketWaveHQ](https://github.com/TicketWaveHQ)).

## License

[MIT](./LICENSE) © TicketWave HQ Ltd. Use it, fork it, ship it.
