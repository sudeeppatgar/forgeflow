# ForgeFlow

**Reusable CI/CD building blocks for shipping software with confidence.**

ForgeFlow is a learning project and a growing collection of reusable GitHub Actions workflows. The goal is to build pipelines once, call them from many repositories, and understand each step along the way.

## What’s here

- `reusable-node-ci.yml` — checks out a Node.js project, installs dependencies, checks lint and types, runs tests and a build, and audits dependencies.
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

The called workflow expects a `package-lock.json`. It runs `npm run lint --if-present`, `npm run typecheck --if-present`, `npm test --if-present`, and `npm run build --if-present`: define these scripts in `package.json` to activate those checks. If a script is absent, npm skips it, so a project without tests will pass this step without running any tests. The dependency audit fails for high or critical vulnerabilities by default; set `run-audit: false` only if you intentionally want to disable it.

For example, a project can define scripts like these in `package.json` (choose tools and commands that match that project):

```json
{
  "scripts": {
    "lint": "eslint .",
    "typecheck": "tsc --noEmit",
    "test": "vitest run",
    "build": "vite build"
  }
}
```

Pin the `uses` reference to a release tag or commit SHA for production use; `@main` is convenient while learning and iterating.

## How reusable workflows work

1. A project triggers its workflow on a push or pull request.
2. That workflow calls ForgeFlow using `jobs.<job_id>.uses`.
3. GitHub runs ForgeFlow’s jobs against the caller’s repository.
4. A failed install, configured lint/type check, test, build, or dependency audit makes the check fail. Branch protection can require this check before merging.

## License

MIT. See [LICENSE](LICENSE).
