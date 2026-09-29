---
name: npm-publish-audit
description: >-
  Audit an npm package for automated publish readiness (package.json, pack dry-run,
  GitHub workflow, Trusted Publishing). Use when the user asks whether a package can
  publish from CI, what is blocking npm release, or to check OIDC / Trusted Publisher
  setup.
disable-model-invocation: true
metadata:
  plugin: npm-publish
---

# npm-publish-audit

Conventions: [../../CONVENTIONS.md](../../CONVENTIONS.md).

Inspect a package directory and report a **pass/fail checklist**. Read-only unless the
user asks you to fix findings. Never invent npm org permissions or Trusted Publisher
state — call out what only a human can verify on npmjs.com.

## Resolve the package directory

1. Prefer an explicit path from the user.
2. Otherwise use the current working directory if it has a `package.json`.
3. If the cwd is a monorepo root, ask which package directory to audit (or use an obvious
   single publishable package if the user already named it).

## Checklist

Run through every item. Mark each **Pass**, **Fail**, or **Human-only** (cannot verify
from the repo alone).

### 1. package.json

Read `package.json` and check:

- `name` present; if scoped (`@scope/...`), note the required npm org
- `version` is valid semver
- `files` / `main` / `exports` look coherent for a published package
- `bin` (if any) points at an existing JS file suitable as a CLI entry
- `publishConfig.access` set when the package is scoped and should be public
- `repository.url` uses `git+https://github.com/<owner>/<repo>.git`; `repository.directory`
  set if the package is not at the repo root
- `engines` noted if present
- `scripts.test` present or absent (affects the workflow)

### 2. Pack dry-run

From the package directory:

```bash
npm pack --dry-run
```

Summarize what would land in the tarball. Flag missing dist files, accidental secrets,
or empty packages.

### 3. Git remote and branch

- Resolve the GitHub remote (`git remote -v`) and default branch (usually `main`).
- Note whether the repo is public or private if that can be inferred (`gh repo view`
  when available). Private repos must not use `--provenance`.

### 4. Existing publish workflow

Look under `.github/workflows/` for a publish-related workflow. If one exists, validate
against `CONVENTIONS.md`:

- `permissions.id-token: write`
- no `--provenance` when the repo is private
- Node **24** (or equally current) via `setup-node`
- tag pattern with a clear prefix and `v*` suffix
- tag version checked against `package.json`
- tagged commit required to be on `main`
- `working-directory` / package path correct for monorepos
- Trusted Publisher allowed action is `npm publish` (document this; cannot verify on npm)

If no workflow exists, mark **Fail** with “add via npm-publish-setup”.

### 5. Registry (best effort)

```bash
npm view <name> version
```

- Success → package exists; Trusted Publisher can be attached (human must confirm it is).
- Failure → may be unpublished, private, or auth/scope issue. Say so; do not claim the
  package is ready for CI-only publish until it exists and TP is configured.

### 6. Human-only items (always list)

Tell the user to verify on npmjs.com:

- Ownership of the `@scope` / package name
- Trusted Publisher entry: org, repo, **exact** workflow filename, environment (if any)
- Allowed action = **`npm publish`**

## Report format

End with a short verdict:

- **Ready** — repo side is green; only human npmjs checks remain (list them).
- **Blocked** — list each Fail with the exact fix (fix `bin`, add workflow, create
  package once, change Node version, remove provenance, etc.).

Do not run `npm publish`, create tags, or push unless the user explicitly asks.
