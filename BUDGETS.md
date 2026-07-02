# Budget reference

Every number in `lighthouserc.json` is a deliberate line in the sand for a
fast Astro marketing site. This page explains what each threshold means, why it
sits where it does, and how to move it when a page genuinely needs headroom.

Treat a broken budget as a review conversation, not an emergency. The point of a
budget is that the regression shows up in the pull request, before it reaches a
visitor, so you can decide on purpose whether to raise the ceiling or fix the
page.

## How the assertions work

Each entry is `[level, options]`:

- `level` is `error` (fails CI) or `warn` (prints, does not fail).
- `minScore` applies to Lighthouse category scores, from 0 to 1.
- `maxNumericValue` applies to metric audits, in milliseconds for timings,
  unitless for layout shift, and bytes for `resource-summary` audits.
- `maxLength` applies to opportunity and diagnostic audits, and asserts on the
  number of flagged items. `maxLength: 0` means the audit must be clean.

## Category scores

| Category | Desktop | Mobile | Why |
| --- | --- | --- | --- |
| Performance | 0.95 | 0.90 | A brochure or landing page on Astro ships almost no client JavaScript, so a high bar is realistic. Mobile is one notch lower to absorb CPU throttling. |
| Accessibility | 0.95 | 0.95 | Colour contrast, labels, and landmarks are cheap to get right and expensive to retrofit. |
| Best practices | 0.95 | 0.95 | Catches mixed content, console errors, and insecure requests early. |
| SEO | 0.95 | 0.95 | Marketing pages live or die on this. Missing meta descriptions and non-crawlable links fail here. |

## Core Web Vitals and timing metrics

| Metric | Desktop budget | Mobile budget | Notes |
| --- | --- | --- | --- |
| First Contentful Paint | 1500 ms | 2200 ms | First pixel of content. Static Astro output should beat this comfortably. |
| Largest Contentful Paint | 2000 ms | 2800 ms | Usually the hero image or headline. Preload the hero and set explicit dimensions. |
| Cumulative Layout Shift | 0.05 | 0.10 | Reserve space for images, embeds, and web fonts to keep this near zero. |
| Total Blocking Time | 150 ms | 250 ms | Proxy for main-thread work. Islands architecture keeps this low by default. |
| Speed Index | 2500 ms | 3400 ms | How quickly the page fills in visually. |
| Time to Interactive | 3000 ms | 3800 ms | When the page reliably responds to input. |

Google's own "good" thresholds are LCP 2500 ms and CLS 0.1. These budgets are
tighter on purpose, because a static marketing site has no excuse to sit at the
edge of the acceptable band.

## Resource size and count budgets

Sizes are asserted in bytes in `lighthouserc.json` and in kilobytes in the
Lighthouse-native `budget.json`. They describe the same ceilings.

| Resource | Ceiling | Reasoning |
| --- | --- | --- |
| Document (HTML) | 30 KB | A single marketing page rarely needs more once compressed. |
| Script | 120 KB | Astro ships zero JS by default; this budget still leaves room for a couple of small islands. |
| Stylesheet | 60 KB | Scoped component styles plus a small global sheet. |
| Font | 120 KB | Roughly two `woff2` families with a couple of weights each. Subset aggressively. |
| Image | 350 KB | Per-page image weight. Serve AVIF or WebP through Astro's image pipeline. |
| Third party | 5 requests | Every tag is someone else's performance and privacy problem on your page. |
| Total | 600 KB | The whole-page ceiling. If you are near it, something is wrong. |

## Diagnostic and opportunity budgets

These audits fail the build when they find work left on the table:

- `unminified-javascript` and `unminified-css` must be clean. The Astro
  production build minifies both, so a hit here means an asset bypassed the
  pipeline.
- `uses-text-compression` must be clean. Turn on Brotli or gzip at the CDN.
- `font-display` must be clean. Set `font-display: swap` (or `optional`) so text
  is never invisible while a font loads.
- `errors-in-console` must be clean. No uncaught errors in production.
- `render-blocking-resources`, `modern-image-formats`, `unused-javascript`, and
  `unused-css-rules` are `warn` by default. Promote them to `error` once your
  baseline is clean and you want to hold the line.

## Adjusting a budget

1. Confirm the regression is intentional. Read the failing audit in the report
   link the workflow prints.
2. If the page legitimately needs the weight (a genuine hero video, an embedded
   map), raise only the specific ceiling, and leave a one-line comment in
   `lighthouserc.json` explaining why.
3. Prefer path-scoped overrides in `budget.json` over loosening the global
   number, so a single heavy page does not relax the budget for the whole site.
4. Never delete an assertion to make CI green. Downgrade `error` to `warn` if
   you must, and open an issue to earn it back.
