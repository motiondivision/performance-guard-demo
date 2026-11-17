# Motion Performance Guard demo

![A pull request's checks on GitHub: Motion Performance Guard fails because /pricing dropped from A to B](docs/guard-check.png)

[Performance Guard](https://motion.dev/docs/performance-guard) catches regressions in animation performance before they ship to users.

It performs a runtime audit on your selected URLs with Motion's [MotionScore system](https://score.motion.dev/methodology). This detects things like layout thrashing, too many animations, scroll animation drivers and more, and then blocks PRs when URLs regress or fails your minimum performance grade.

Every PR is tagged with prompts for your agent to automatically fix.

## See it in action

- **[Slow animations](https://github.com/motiondivision/performance-guard-demo/pull/2)**: This PR swaps a smooth CSS animation for 300 boxes animated from JavaScript, with a forced layout read on every frame. Guard fails the check, and its comment shows the page dropping from S to F, names the problem (frame thrashing) and the element, and gives a fix prompt you can either paste into your agent or a monitoring agent can fix automatically. The regression was then accepted with the `performance-ok` label and `/guard accept`, you can use either to skip a check.
- **[Simple text change](https://github.com/motiondivision/performance-guard-demo/pull/3)**: This PR passes as no text changes were made.

## How it works

1. Pushes to `main` record a baseline grade for each page.
2. Each pull request is compared with that baseline. The Motion Performance app posts one comment and a check named **Motion Performance Guard**.
3. Make the check required, and a regression can't merge. A maintainer can still accept it with the `performance-ok` label or a `/guard accept` comment.

This is the whole workflow in this repo. It serves the `site/` folder on the runner, so it needs no preview deploys:

```yaml
name: Motion Performance Guard
on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:
permissions:
  id-token: write
  contents: read
jobs:
  guard:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - id: guard
        uses: motiondivision/motionscore-guard@v2
        with:
          url: http://localhost:4173
          pages: /
          serve: npx --yes serve@14 -l 4173 site
```

Want to use a preview deploy instead? Guard can audit those instead. The [docs](https://motion.dev/docs/performance-guard) detail both setups.

## Add Guard to your repo

1. Open the [Guard dashboard](https://motion.dev/dashboard/guard) and install the Motion Performance app on your GitHub organisation.
2. Add the workflow.
3. Open a pull request. Guard’s first run sets the baseline, and each push to `main` updates it.

The full guide is in the [Performance Guard docs](https://motion.dev/docs/performance-guard).

## Guard is part of Motion Cloud

Performance Guard comes with [Motion Cloud](https://motion.dev/cloud). Each person who opens pull requests needs a seat. Pull requests from bots, such as Dependabot, use the seat that installed the app.
