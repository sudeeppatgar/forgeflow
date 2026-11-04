# ForgeFlow

**Reusable CI/CD building blocks for shipping software with confidence.**

ForgeFlow is a learning project and a growing collection of reusable GitHub Actions workflows. The goal is to build pipelines once, call them from many repositories, and understand each step along the way.

## What’s here

- `reusable-node-ci.yml` — checks out a Node.js project, installs dependencies, checks lint and types, runs tests and a build, and audits dependencies.
- `examples/node-ci.yml` — a consumer workflow showing how another repository calls it.
- `reusable-render-deploy.yml` — triggers a Render deployment using a deploy hook.
- `examples/backend-render.yml` — a backend pipeline that runs CI before deploying to Render.

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

## Continuous deployment: Render and Netlify

The backend example runs CI for pull requests and pushes to `main`. A production deploy runs only for a push to `main`, and only after the CI job passes.

### Backend to Render

1. Connect the backend repository to a Render Web Service and configure its build/start commands and deploy branch (`main`).
2. In Render, create a deploy hook for that service and copy its URL.
3. In the backend GitHub repository, add an Actions secret named `RENDER_DEPLOY_HOOK_URL` containing the hook URL.
4. Copy `examples/backend-render.yml` to `.github/workflows/deploy.yml` in the backend repository.

When CI passes on a push to `main`, ForgeFlow sends a POST request to the hook. Render then builds and deploys the latest commit on its configured branch. Keep Render automatic deploys disabled if you want this workflow to be the single deployment trigger.

## How reusable workflows work

1. A project triggers its workflow on a push or pull request.
2. That workflow calls ForgeFlow using `jobs.<job_id>.uses`.
3. GitHub runs ForgeFlow’s jobs against the caller’s repository.
4. A failed install, configured lint/type check, test, build, or dependency audit makes the check fail. Branch protection can require this check before merging.

## License

MIT. See [LICENSE](LICENSE).
