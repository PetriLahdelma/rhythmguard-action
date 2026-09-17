# Rhythmguard Audit

A GitHub Action that measures off-scale spacing in CSS, SCSS and Tailwind class strings with [stylelint-plugin-rhythmguard](https://github.com/PetriLahdelma/stylelint-plugin-rhythmguard), annotates the diff, posts the report as a pull-request comment, and fails on new drift against a committed baseline.

```yaml
name: Spacing
on: [pull_request]
permissions:
  contents: read
  pull-requests: write
jobs:
  spacing:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: PetriLahdelma/rhythmguard-action@v1
        with:
          directory: src
```

That is enough for a first run: the scale is inferred from your own tokens (`--space-*`, `--spacing-*`, Sass maps, Tailwind `@theme`, installed token packages), every off-scale value becomes an annotation on the diff, and the summary lands in the job summary and as one comment on the pull request, updated in place on later pushes.

## Gate only new drift

Legacy codebases have drift. Write a baseline once, commit it, and the action fails only when a change adds a finding that is not in it:

```bash
npx stylelint-plugin-rhythmguard audit src --scale auto --write-baseline
git add .rhythmguard-baseline.json
```

With the file present at the default path the action compares against it and fails on new drift. Every report leads with `Since baseline: N resolved, M new`. Baselines key findings by content, not by line, so moving code does not produce noise.

## Inputs

| Input | Default | Meaning |
| --- | --- | --- |
| `directory` | `.` | Directory to audit |
| `scale` | `auto` | Comma-separated values, or `auto` to infer from the project's tokens |
| `baseline` | `.rhythmguard-baseline.json` | Baseline to compare against when the file exists |
| `fail-on-new-drift` | `true` | Fail when the comparison finds new drift |
| `max-findings` | | Fail when total findings exceed this number |
| `min-cleanliness` | | Fail when scale cleanliness is below this percent |
| `comment` | `true` | Post and update the pull-request comment (needs `pull-requests: write`) |
| `badge` | | Write a shields.io endpoint document to this path |
| `version` | `latest` | `stylelint-plugin-rhythmguard` version to run |
| `github-token` | `${{ github.token }}` | Token for the comment |

## Outputs

`findings`, `cleanliness`, `scale`, `scale-source`, `new-findings`, `resolved-findings`, `report` (path of the JSON report, schema 2.0).

## Badge

With `badge: badges/spacing.json` the action writes a shields.io endpoint document. Commit it to a branch GitHub Pages serves, or push it to a gist, then:

```md
![spacing drift](https://img.shields.io/endpoint?url=https://<your-pages-host>/badges/spacing.json)
```

## What it does not do

It does not install anything into your project and it does not need a Stylelint config; it runs `npx stylelint-plugin-rhythmguard audit`. For enforcement in the editor and in `--fix`, add the plugin to your Stylelint config: see the [main README](https://github.com/PetriLahdelma/stylelint-plugin-rhythmguard#readme).

## License

MIT.

The comment is posted once per pull request and updated in place on every later push, so the thread never fills with stale reports.

