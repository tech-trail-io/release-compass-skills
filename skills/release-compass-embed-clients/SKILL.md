---
name: release-compass-embed-clients
description: >-
  Integrates Release Compass changelogs and request boards into a customer app
  using @techtrail/release-compass-react, -angular, -next, or -core. Use when
  the user asks to embed a changelog, roadmap, feature board, upvote requests,
  or wire private boards with an API key.
---

# Release Compass embed clients

Wire a product to Release Compass using the open-source packages in
[release-compass-clients](https://github.com/tech-trail-io/release-compass-clients).
Product docs: [release-compass.app/docs/#embed](https://release-compass.app/docs/#embed).

Do **not** put a project API key (`rc_live_…`) in browser code, React/Angular
props, or `NEXT_PUBLIC_*` env vars.

## Choose the package

| Goal | Package | Auth |
|------|---------|------|
| Public changelog / public roadmap UI in React | `@techtrail/release-compass-react` | **owner** + **project** slug |
| Same in Angular | `@techtrail/release-compass-angular` | **owner** + **project** slug |
| Private board, create, vote, status from Next.js | `@techtrail/release-compass-next` | `RELEASE_COMPASS_API_KEY` on the **server** |
| Custom backend (Express, Nest, …) | `@techtrail/release-compass-core` `createServerClient` | API key on the **server** |
| Headless fetch only (any runtime) | `@techtrail/release-compass-core` `createPublicClient` | owner + project slug |
| Vue | Not available yet (**Soon**) | — |

Default API base: `https://api.release-compass.app/api/v1` (overridable with
`baseUrl` / `RELEASE_COMPASS_API_BASE_URL`).

Public API paths are `/api/v1/public/changelogs/{owner}/{project}`. The owner
segment is a user **username** or organization **slug**.

Ask the user for the **owner** (username or org slug) and **project slug**
(from the dashboard) before coding. For private boards or voting, also ask
where the API key will live (server env).

## Install

Prefer the package manager already used in the repo (`pnpm` / `npm` / `yarn`).
Packages may be consumed from the published npm names or from the clients
monorepo while unpublished — check
[release-compass-clients](https://github.com/tech-trail-io/release-compass-clients)
README if install fails.

```bash
pnpm add @techtrail/release-compass-react
# or
pnpm add @techtrail/release-compass-angular
# or (Next.js server)
pnpm add @techtrail/release-compass-next
# or (any server)
pnpm add @techtrail/release-compass-core
```

## Public React

1. Install `@techtrail/release-compass-react`.
2. Wrap the surface with `ReleaseCompassProvider` (`ownerSlug` or `owner`,
   `projectSlug`, optional `baseUrl`, optional `lang`).
3. Render `Changelog` and/or `RequestBoard`.
4. Style via your CSS targeting `data-rc` attributes (headless markup).

```tsx
import {
  ReleaseCompassProvider,
  Changelog,
  RequestBoard,
} from "@techtrail/release-compass-react";

export function ProductUpdates() {
  return (
    <ReleaseCompassProvider ownerSlug="acme" projectSlug="signals">
      <Changelog />
      <RequestBoard />
    </ReleaseCompassProvider>
  );
}
```

Public `RequestBoard` is **read-only**. Do not add in-browser voting with an
API key. Voting goes through a backend Route Handler / Server Action (Next) or
`createServerClient`.

## Public Angular

1. Install `@techtrail/release-compass-angular`.
2. `provideReleaseCompass({ ownerSlug, projectSlug })` in
   `bootstrapApplication` (or a route/providers array).
3. Use standalone `<rc-changelog />` and `<rc-request-board />`.

```ts
import {
  provideReleaseCompass,
  RcChangelogComponent,
  RcRequestBoardComponent,
} from "@techtrail/release-compass-angular";

bootstrapApplication(AppComponent, {
  providers: [
    provideReleaseCompass({ ownerSlug: "acme", projectSlug: "signals" }),
  ],
});
```

## Next.js (private boards and voting)

1. Install `@techtrail/release-compass-next` (imports `server-only`).
2. Set **server** env (never `NEXT_PUBLIC_`):

```bash
RELEASE_COMPASS_API_KEY=rc_live_…
# Either:
RELEASE_COMPASS_OWNER_SLUG=acme
RELEASE_COMPASS_PROJECT_SLUG=signals
# Or combined:
# RELEASE_COMPASS_PROJECT=acme/signals
```

3. Call helpers from Route Handlers, Server Actions, or RSC only:

| Helper | Use |
|--------|-----|
| `getChangelog` / `getRequestBoard` | With key → private-capable board list via API key routes; without key → public |
| `createRequest` | File a feature/bugfix |
| `voteRequest(id, voter)` | `voter` is an opaque id (user id, email, …) |
| `updateRequest` | Change status (`open`, `planned`, `in_progress`, `done`, `declined`) |
| `createReleaseCompassClient` | Shared `createServerClient` from env |

Example upvote route:

```ts
import { voteRequest } from "@techtrail/release-compass-next";

export async function POST(
  request: Request,
  context: { params: Promise<{ id: string }> },
) {
  const { id } = await context.params;
  const { voter } = (await request.json()) as { voter: string };
  return Response.json(await voteRequest(id, voter));
}
```

The browser UI should call **your** route (session/user id → `voter`), not the
Release Compass API with a key.

## Core (any backend)

```ts
import {
  createPublicClient,
  createServerClient,
} from "@techtrail/release-compass-core";

const publicClient = createPublicClient({
  ownerSlug: "acme",
  projectSlug: "signals",
});
await publicClient.getChangelog();
await publicClient.getRequestBoard();

// Server only
const server = createServerClient({
  ownerSlug: "acme",
  projectSlug: "signals",
  apiKey: process.env.RELEASE_COMPASS_API_KEY!,
});
await server.getRequestBoard(); // includes private boards
await server.createRequest({ kind: "feature", title: "…" });
await server.voteRequest("request-id", "user-42");
await server.updateRequest("request-id", { status: "planned" });
```

## Checklist before finishing

- [ ] Correct package for public vs server
- [ ] Owner (username or org slug) + project slug from the user (no invented values)
- [ ] No API key in client bundles or `NEXT_PUBLIC_*`
- [ ] Public roadmap UI has no secret-backed vote buttons; votes go through the consumer backend when needed
- [ ] Headless embeds: add minimal CSS or map `data-rc` to the product design system
- [ ] Point the user at the hosted page as a fallback:
  `https://release-compass.app/changelog/{owner}/{project}/` and
  `https://release-compass.app/roadmap/{owner}/{project}/`

## Out of scope for this skill

- Rewriting changelog **copy** → [customer-facing-changelog](../customer-facing-changelog/SKILL.md)
- Creating/publishing notes via MCP → [release-compass-mcp](../release-compass-mcp/SKILL.md)
- Vue package (not shipped)
