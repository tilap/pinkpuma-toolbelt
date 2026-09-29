# npm publish conventions

Shared by every skill in this plugin. Read this once; each `SKILL.md` links back here
instead of repeating it.

Goal: **unattended** CI publish of an npm package — no interactive 2FA at publish time,
no human `npm stage approve` step.

## Auth: Trusted Publishing (OIDC)

Prefer [npm Trusted Publishing](https://docs.npmjs.com/trusted-publishers) over
long-lived tokens.

- GitHub Actions workflow must set `permissions.id-token: write` (and usually
  `contents: read`).
- On npmjs.com, attach a Trusted Publisher to the package: GitHub org, repo, workflow
  filename, and environment if the workflow uses one.
- Allowed action on the Trusted Publisher must be **`npm publish`**, not a stage-only
  action. Stage-only forces a human approve step and breaks full automation.
- Bypass-2FA / classic “automation” tokens that skip 2FA are deprecated. Do not set
  them up for new packages.

## Provenance

Do **not** pass `--provenance` when the GitHub repository is **private**. The registry
rejects provenance for private source repos (`E422`).

Use `npm publish --access public` (or the package’s `publishConfig.access`) without
`--provenance` unless the repo is public and provenance is explicitly desired.

## Node and npm on the runner

- Use **Node 24** in `actions/setup-node` (or at least a Node version that ships a
  current enough npm for Trusted Publishing).
- Do **not** run `npm i -g npm@latest` on an older Node image: the latest npm may require
  a newer Node than the runner provides, and the install fails mid-job.

## Version and tags

- Publish is triggered by a **git tag**, not by a push to `main` alone.
- Tag form: `<prefix>v<semver>` (example: `mycli-v1.2.3`). The prefix defaults to the
  unscoped package name.
- The tag’s version suffix **must** equal `package.json` `"version"`.
- The tagged commit must be reachable from `main` (ancestor check in CI). Refuse to
  publish from a tag that is not on the release branch.

## package.json requirements

- `name` — scoped packages need an npm org/scope the publisher owns. Missing scope rights
  often surface as `E404` (looks like “not found”, actually auth/ownership).
- `version` — must match the release tag.
- `bin` — if present, must point at a valid JS entry (e.g. `bin/<name>.js` that
  `require`s or re-exports the real CLI). Invalid bin paths are stripped or break
  install.
- `repository.url` — use `git+https://github.com/<owner>/<repo>.git`. If the package
  lives in a subdirectory, set `repository.directory` to that path.
- `publishConfig.access` — typically `"public"` for scoped packages that should be
  public.
- `files` / `main` / `exports` — confirm the tarball contains what users need
  (`npm pack --dry-run`).

## First publish vs Trusted Publisher

The package must **exist** on the registry before a Trusted Publisher can be attached.
The very first version may need one interactive `npm publish` (or another bootstrap
path). After the package exists and Trusted Publishing is configured, later versions
publish from CI alone.

## Soft dependency

When creating commits or annotated tags for a release, use the `git-workflow` plugin
conventions if it is available. Do not invent a parallel commit style.
