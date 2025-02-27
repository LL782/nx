# Configuring CI Using GitHub Actions and Nx

Below is an example of an GitHub Actions setup, building and testing only what is affected.

```yaml {% fileName=".github/workflows/ci.yml" %}
name: CI
on:
  push:
    branches:
      # Change this if your primary branch is not main
      - main
  pull_request:

# Needed for nx-set-shas when run on the main branch
permissions:
  actions: read
  contents: read

jobs:
  main:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      # Cache node_modules
      - uses: actions/setup-node@v3
        with:
          node-version: 20
          cache: 'npm'
      - run: npx nx-cloud start-ci-run --distribute-on="5 linux-medium-js" --stop-agents-after="build" # this line enables distribution
      - run: npm ci
      - uses: nrwl/nx-set-shas@v3
      # This line is needed for nx affected to work when CI is running on a PR
      - run: git branch --track main origin/main

      - run: npx nx-cloud record -- nx format:check
      - run: npx nx affected -t lint test build --parallel=3
```

### Get the Commit of the Last Successful Build

`GitHub` can track the last successful run on the `main` branch and use this as a reference point for the `BASE`. The `nrwl/nx-set-shas` provides a convenient implementation of this functionality which you can drop into your existing CI config.
To understand why knowing the last successful build is important for the affected command, check out the [in-depth explanation in Actions's docs](https://github.com/marketplace/actions/nx-set-shas#background).
