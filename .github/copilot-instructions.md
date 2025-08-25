# Copilot instructions for this repo

This repository powers the GitHub profile page for `@rolandnyamo`. It’s a content repo (no app build) with a daily GitHub Actions job that generates a metrics SVG displayed in the profile README.

## Big picture
- Purpose: keep `README.md` and the generated `github-metrics.svg` up to date for the GitHub profile.
- Automation: a workflow runs every day and on manual dispatch to regenerate metrics via `lowlighter/metrics` and commits updates if the SVG changed.
- Output flow: `lowlighter/metrics` -> `github-metrics.svg` -> referenced in `README.md`.

## Key files
- `README.md`: Profile content. References `./github-metrics.svg` in the “Profile Metrics” section.
- `.github/workflows/metrics.yml`: Daily metrics generator.
  - schedule: `cron: "0 6 * * *"` (06:00 UTC daily)
  - manual: `workflow_dispatch`
  - concurrency: prevents overlapping runs (`group: metrics`)
  - action: `lowlighter/metrics@latest`
  - output: `filename: github-metrics.svg`

## Secrets / tokens
- Requires `secrets.METRICS_TOKEN` (a GitHub PAT) to avoid API limits and enable plugins.
  - Scopes for public repos: `public_repo`, `read:org` are typically sufficient for common plugins. Add more only if enabling plugins that need them.

## Conventions and patterns
- Keep the SVG path as `github-metrics.svg` or update both the workflow `filename` and the README `<img src="..." />` together.
- The commit step only pushes when there are changes (guarded by `git status --porcelain`).
- Action version is `@latest` for simplicity. Pin to a major tag if you need reproducibility (verify available tags first).
- Cron times are UTC. Scheduled workflows may pause after long inactivity; a push or manual dispatch resumes them.

## Making common changes (examples)
- Change run time to midnight UTC:
  - In `.github/workflows/metrics.yml`, set: `cron: "0 0 * * *"`.
- Avoid overlaps if you shorten intervals:
  - Keep `concurrency: { group: metrics, cancel-in-progress: true }`.
- Tweak displayed sections:
  - In the `with:` block, adjust plugins, e.g. `plugin_languages_limit: 12`, `plugin_isocalendar_duration: full-year`.
- Add/remove plugins:
  - Toggle booleans like `plugin_topics: yes/no`, or add plugin-specific keys per the lowlighter/metrics docs.

## External integrations referenced
- `lowlighter/metrics` GitHub Action generates the metrics SVG.
- Shields.io badges and `github-readme-stats` pins are embedded images in `README.md` and not generated here.

## Quick checklist for agents
- If updating metrics output name, update README reference too.
- Ensure `METRICS_TOKEN` exists in repo secrets before relying on the workflow.
- For reproducibility, consider pinning `lowlighter/metrics` to a known good version after testing.
- After edits to README or workflow, push to `main`; schedule runs only on the default branch.
