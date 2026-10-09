---
name: nextjs-app-structure
description: >-
  Place and check files in a Next.js App Router app using the house layout under src/:
  routes in src/app/, a component used by one route in that route's _components/,
  shared presentation components in src/components/ui/, other shared components in
  src/components/, configuration in src/config/, and environment variables only in
  src/env.ts. Use when adding, moving, or reviewing a file in a Next.js app. Do not
  use it for the Pages Router or any other stack.
metadata:
  plugin: nextjs-app-structure
---

# Next.js app structure

Conventions: [../../CONVENTIONS.md](../../CONVENTIONS.md). Read it before placing a
file. The rules live there. This skill only decides whether they apply, then follows
them.

## Gate

1. Find the Next.js package: the directory whose `package.json` lists `next`, or that
   contains `next.config.*`. In a monorepo this is not necessarily the git root.
   Several matches: ask which app. No match: stop. Do not propose this layout.
2. App Router only. `src/app/` or a root `app/` qualifies. Pages Router alone
   (`pages/` or `src/pages/`, no `app/` directory): stop and ask. A hybrid app keeps
   its `pages/` tree untouched.
3. The house layout is `src/` inside that package. A root `app/` and no `src/` does
   not match — report it. Do not invent a second layout beside `src/`.

## Place

Match the new or moved file to one rule in `CONVENTIONS.md` and write it there.
Nothing matches: stop and ask. Do not add a top-level folder. Do not restructure
unrelated files in the same task.

## Check

When asked to review the layout, list each violation with its path and the rule it
breaks. Move a file only when the user asked to fix the layout, or when that file is
already part of this task. Say what moved.

Paths only. Routing, caching, and server or client components stay in the project's
Next.js docs.
