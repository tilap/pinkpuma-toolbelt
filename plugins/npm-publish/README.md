# npm-publish

Audit an npm package for automated release readiness, and scaffold GitHub Actions plus
npm Trusted Publishing (OIDC) so publishes run unattended from a version tag.

See [`CONVENTIONS.md`](CONVENTIONS.md) for the shared rules.

## Skills

| Skill                                                  | Use when                                                                |
| ------------------------------------------------------ | ----------------------------------------------------------------------- |
| [npm-publish-audit](skills/npm-publish-audit/SKILL.md) | Checking whether a package is ready for automated npm publish           |
| [npm-publish-setup](skills/npm-publish-setup/SKILL.md) | Adding or updating the publish workflow and Trusted Publisher checklist |
