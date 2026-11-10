# Security Policy

This repository ships configuration files and a GitHub Actions workflow that
people copy into their own projects. Please report privately, rather than in
a public issue, if you find for example:

- something in [`templates/lighthouse.yml`](templates/lighthouse.yml) that
  grants more permissions than it needs, or that could run untrusted code
  with access to secrets;
- a default that exposes more than the README says (the configs upload
  reports to Lighthouse CI's temporary public storage, which the README
  states).

**Contact:** email [hello@ticketwavehq.com](mailto:hello@ticketwavehq.com)
and say in the subject that it is a security report. TicketWave HQ's
disclosure policy, which covers this repository, is at
<https://ticketwavehq.com/security>.

We will acknowledge your report and say what we intend to do. We do not
publish a response time. Please give us a reasonable window to fix the
issue before you publish it.

Out of scope: vulnerabilities in Lighthouse, Lighthouse CI, Astro or GitHub
Actions themselves. Report those to their maintainers.
