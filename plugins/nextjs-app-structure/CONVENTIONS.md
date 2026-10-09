# Next.js app structure

Next.js App Router only. The root of this layout is the package whose `package.json`
lists `next` (dependencies or devDependencies), or that contains `next.config.*`. In a
monorepo that package is the root — not the git root. Any other package keeps its own
layout.

A Pages Router app (`pages/` or `src/pages/`, and no `app/` directory) is out of scope.
Stop and ask. A hybrid app keeps its `pages/` tree as it is; these rules cover the App
Router side.

This is the whole layout. A file that matches nothing here: stop and ask. Do not invent
a top-level folder.

```
src/
  app/
    <route>/
      page.tsx
      _components/  components used only by this route
  components/
    ui/             presentation only, shared
    …               every other shared component
  config/           configuration
  env.ts            the only process.env reader
```

## `src/app/`

Files the App Router reserves (`page`, `layout`, `loading`, `error`, `route`,
`template`, `not-found`, and the rest of that set).

A component used by a single route lives in that route's `_components/` folder,
next to its `page.tsx`:

```
src/app/page123/_components/
```

The underscore keeps the folder out of the URL. Do not name it `components` inside
`app/`. A component used by two routes does not go here — it goes under
`src/components/`.

## `src/components/`

Shared components only.

- **`ui/`** — presentation only. Props in, markup out. No store, no data fetching, no
  knowledge of a page or a route.
- **Outside `ui/`** — components that compose other components, use a store, or belong
  to a feature, and that more than one route uses.

A file that exists only for one component (a helper, a style, a test) sits in that
component's folder.

## `src/env.ts`

The only file that reads `process.env`. Everything else imports this module.

## `src/config/`

Configuration: values derived from `src/env.ts`, or hardcoded constants. Import
`env.ts`. Do not read `process.env` here.
