# ForgeFlow

**Reusable CI/CD building blocks for shipping software with confidence.**

ForgeFlow is a learning project and a growing collection of reusable GitHub Actions workflows. The goal is to build pipelines once, call them from many repositories, and understand each step along the way.

## What’s here

- `reusable-node-ci.yml` — checks out a Node.js project, installs dependencies, and runs its tests.
- `examples/node-ci.yml` — a consumer workflow showing how another repository calls it.

## Try the Node.js workflow

In a Node.js project, create `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
  push:
    branches: [main]

jobs:
  test:
    uses: sudeeppatgar/forgeflow/.github/workflows/reusable-node-ci.yml@main
    with:
      node-version: '22'
```

The called workflow expects a `package-lock.json`. It runs `npm test --if-present`: if the project defines a `test` script in `package.json`, npm runs it; otherwise npm skips it. A skipped test step passes without actually running tests. Pin the `uses` reference to a release tag or commit SHA for production use; `@main` is convenient while learning and iterating.

## How reusable workflows work

1. A project triggers its workflow on a push or pull request.
2. That workflow calls ForgeFlow using `jobs.<job_id>.uses`.
3. GitHub runs ForgeFlow’s jobs against the caller’s repository.
4. A failed install or test makes the check fail. Branch protection can require this check before merging.

## License

MIT. See [LICENSE](LICENSE).
