# Contributing

Issues and pull requests are welcome, especially real-world budget numbers
from Astro sites in production and audit assertions worth enabling by
default.

## The bar for a change

- **Framework-honest.** The goal is a budget a well-built static Astro site
  can meet, not one that is impossible to pass. If you tighten a number, say
  what kind of site you measured it on.
- **No invented numbers.** Do not add scores, timings or "typical" figures
  you cannot point to. If a threshold is a judgement call, say so in
  [`BUDGETS.md`](BUDGETS.md).
- **Keep the two configs in step.** A new assertion in `lighthouserc.json`
  usually belongs in `lighthouserc.mobile.json` too, and in the tables in
  `README.md` and `BUDGETS.md`.
- **British English, no em-dashes.** Use commas or spaced hyphens.

## Check your change

CI ([`.github/workflows/ci.yml`](.github/workflows/ci.yml)) runs both budgets
against the static pages in `test/fixture/` and checks that a deliberate
regression fails. To run the same thing locally (Chrome or Chromium needed):

```bash
npx @lhci/cli@0.15.x autorun --config=./lighthouserc.json \
  --collect.staticDistDir=./test/fixture --upload.target=filesystem --upload.outputDir=./lhci_reports/desktop
npx @lhci/cli@0.15.x autorun --config=./lighthouserc.mobile.json \
  --collect.staticDistDir=./test/fixture --upload.target=filesystem --upload.outputDir=./lhci_reports/mobile
```

If you assert on a new audit, check it exists in the Lighthouse version that
`@lhci/cli@0.15.x` runs (Lighthouse 12); CI will fail if it does not.

## Licence

By contributing you agree that your contribution is licensed under the MIT
licence, the same as the rest of the project. Security reports go through
[`SECURITY.md`](SECURITY.md), not a public issue.
