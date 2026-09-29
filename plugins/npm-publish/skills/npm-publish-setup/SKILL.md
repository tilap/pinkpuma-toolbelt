---
name: npm-publish-setup
description: >-
  Scaffold or update a GitHub Actions workflow for automated npm publish via Trusted
  Publishing (OIDC). Use when the user asks to set up npm CI publish, add a publish
  workflow, or wire Trusted Publisher for a package.
disable-model-invocation: true
metadata:
  plugin: npm-publish
---

# npm-publish-setup

Conventions: [../../CONVENTIONS.md](../../CONVENTIONS.md).
Template: [../../templates/publish-npm.yml](../../templates/publish-npm.yml).

Add or update a GitHub publish workflow so tagged releases publish to npm via OIDC.
Do **not** run `npm publish` or create/push tags unless the user explicitly asks.

## Gather parameters

Confirm with the user (or infer from the repo when obvious):

| Parameter             | Default                                                                                      |
| --------------------- | -------------------------------------------------------------------------------------------- |
| Package directory     | `.` if `package.json` is at the repo root; otherwise the folder that contains it             |
| Tag prefix            | Unscoped package name + `-` (e.g. `@acme/mycli` → `mycli-`) so tags look like `mycli-v1.2.3` |
| Workflow filename     | `publish-npm.yml` under `.github/workflows/`                                                 |
| Workflow display name | `Publish <package-name>`                                                                     |

Read `package.json` in the package directory for `name` and `version`.

## Write the workflow

1. Copy [../../templates/publish-npm.yml](../../templates/publish-npm.yml).
2. Replace placeholders:
   - `{{WORKFLOW_NAME}}` — display name
   - `{{TAG_PREFIX}}` — e.g. `mycli-`
   - `{{PACKAGE_DIR}}` — `.` or a subdirectory (GitHub Actions `working-directory`)
   - `{{WORKFLOW_FILE}}` — basename used in comments / Trusted Publisher docs
3. If `package.json` has no `scripts.test`, remove the **Test** step (or replace it with
   a no-op comment) so the job does not fail on a missing script.
4. Keep `npm publish --access public` **without** `--provenance` by default. Only add
   provenance if the user confirms the GitHub repo is **public** and wants it.
5. Write the file to `.github/workflows/<workflow-filename>`. If an existing publish
   workflow already covers this package, update it in place rather than adding a second
   conflicting trigger — say what you changed.

## After writing: print the human checklist

Print these exactly so the user can finish setup on npmjs.com and cut a release:

### Trusted Publisher (npmjs.com)

1. Ensure the package already exists on the registry. If it does not, one interactive
   bootstrap publish (or equivalent) is required before Trusted Publishing can be
   attached — see `CONVENTIONS.md`.
2. Package → **Settings** → **Trusted Publisher** → GitHub Actions:
   - Organization or user: `<github-owner>`
   - Repository: `<repo-name>`
   - Workflow filename: `<workflow-filename>` (exact basename)
   - Environment: leave empty unless the workflow uses a GitHub Environment
3. Allowed action: **`npm publish`** (not stage-only).

### First automated release

```bash
# 1. Bump version in package.json (and lockfile if needed)
# 2. Commit (prefer git-workflow commit skill if available)
git tag -a <prefix>vX.Y.Z -m "<package-name> X.Y.Z"
git push origin <prefix>vX.Y.Z
```

The tag commit must be on `main`. Pushing the tag runs the workflow; OIDC authenticates
to npm — no `NPM_TOKEN` secret required when Trusted Publishing is configured correctly.

## Notes

- Soft-dep: use `git-workflow` for commit/tag message style when that plugin is installed.
- Prefer running `npm-publish-audit` after setup so the user sees remaining Blocked /
  Human-only items.
- Never skip hooks, force-push tags, or publish from a branch that is not `main` unless
  the user explicitly overrides the conventions.
