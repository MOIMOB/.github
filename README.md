# .github

MOIMOB org defaults: profile README, shared CI, workflow templates.

## Shared CI

`.github/workflows/ci.yml`: reusable workflow. App repo `.github/workflows/ci.yml`:

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
permissions:
  contents: read
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
jobs:
  ci:
    uses: MOIMOB/.github/.github/workflows/ci.yml@v1
    with:
      lint: false # remove parts the app doesn't have yet
```

Jobs and toggles (defaults = standard):

| Job | Inputs |
|---|---|
| `build` (always) | `fastlane-metadata` (off), `format-check`, `lint`, `test` + `test-args`, `build` + `build-script`, `post-build` |
| `database` | `database` (off), `database-commands`, `database-types-check`, `database-supabase-exclude` |
| `e2e` | `e2e`: `none` / `playwright` / `cypress`, `e2e-supabase`, `e2e-command` (Playwright), `e2e-start` + `e2e-wait-on` (Cypress) |
| all | `mise-tools` (default `node`), `supabase-cli-version` (empty = `npx supabase`) |

Extra checks: add your own job next to `ci:` and reuse the setup action (mise + `npm ci`):

```yaml
  android:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: MOIMOB/.github/actions/setup@v1
        with: { mise-tools: node java }
      - run: npm run build && npx cap sync android
```

## Versioning

Callers pin `@v1`. Non-breaking changes: merge to `main`, move the tag (`git tag -f v1 && git push -f origin v1`). Breaking changes (renamed/removed inputs, new default-on steps): tag `v2`.
