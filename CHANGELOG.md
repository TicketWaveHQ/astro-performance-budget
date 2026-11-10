# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

Nothing yet.

## [1.0.0] - 2026-11-10

First tagged release. The budget has been public since 2026-07-02.

### Contents

- `lighthouserc.json` (desktop) and `lighthouserc.mobile.json` (mobile):
  category scores, Core Web Vitals, resource-size ceilings and clean-audit
  assertions.
- `budget.json`: the same ceilings in Lighthouse's budget format.
- `templates/lighthouse.yml`: the workflow to copy into an Astro project.
- `BUDGETS.md`: the reasoning behind every threshold.
- `examples/package.scripts.json`: scripts for running the budgets locally.

### Breaking

- The drop-in workflow moved from `.github/workflows/lighthouse.yml` to
  `templates/lighthouse.yml`. In this repository, which has no Astro site,
  it could never pass: `actions/setup-node` stopped at "Dependencies lock
  file is not found". Copy it from the new path (the README's commands
  do). Your own copy keeps working as it is.
- The optional pull-request comment step is gone, and the workflow asks only
  for `contents: read`. The comment said only "see the job logs", and
  needed `pull-requests: write`.

### Added

- `.github/workflows/ci.yml`: this repository's own check. It runs both
  budgets against two static pages in `test/fixture/`, then checks that a
  page with a console error fails the desktop budget. Reports stay on the
  runner.
- README: what the repo is, who it is for, and a 60-second start; how to
  choose pages; how to loosen one page with `assertMatrix`; the defaults
  worth knowing (optimistic aggregation, public report storage).
- `CONTRIBUTING.md`, `SECURITY.md` and this changelog.

### Changed

- `@lhci/cli@0.15.x` (was `0.14.x` in the workflow, `^0.14.0` in the example
  scripts). 0.15.1 runs Lighthouse 12.6.1, in which every asserted audit
  exists. LHCI 0.16 moves to Lighthouse 13 and has not been tested here.
- `actions/checkout@v7`, `actions/setup-node@v7`, Node 24 (Node 20 reached end
  of life in April 2026).
- The README's quick-start URLs point at `TicketWaveHQ/astro-performance-budget`;
  they used the old `sansware/` path, which only worked through GitHub's
  redirect.
- The author's GitHub handle is `@eu-jordangilbert` (`@sansware` is an
  organisation account). The practice link is named "Digital practice".

### Fixed

- The README said results were the median of three runs. LHCI's default is
  `optimistic`: each assertion uses the most favourable run. The README and
  `BUDGETS.md` now say so and show how to switch to `median-run`.
- The README and `BUDGETS.md` recommended path-scoped overrides in
  `budget.json`. Nothing in the shipped setup reads `budget.json`, and LHCI
  refuses to combine it with assertions. The README now explains how to use
  it on its own, and recommends `assertMatrix` for per-page ceilings.
- The README's table showed a third-party request budget for mobile; the
  mobile config does not assert one.
- The workflow's comment said it tested "every changed page"; LHCI tests up
  to five pages it finds in `dist/`.

[Unreleased]: https://github.com/TicketWaveHQ/astro-performance-budget/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/TicketWaveHQ/astro-performance-budget/releases/tag/v1.0.0
