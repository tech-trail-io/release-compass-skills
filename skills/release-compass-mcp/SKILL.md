---
name: release-compass-mcp
description: >-
  Uses the Release Compass Streamable HTTP MCP to list, create, update,
  translate, publish, unpublish, schedule, or delete changelog notes. Use when
  the user mentions Release Compass, changelog MCP, create_releases, drafts, or
  public changelog import.
---

# Release Compass MCP

Configured server: Streamable HTTP at
`https://api.release-compass.app/api/v1/mcp` with
`Authorization: Bearer rc_live_…` (project API key). The key **selects the
project**. Handshake (`initialize`, `tools/list`) does not count against quota;
**every `tools/call` does** (free plan: 100/month, shared with API-key REST).

Always **inspect tool schemas** on the live server before calling. Prefer MCP
over REST unless the user asks for HTTP. The MCP schemas and
[docs](https://release-compass.app/docs/) /
[llms.txt](https://release-compass.app/llms.txt) are enough.

## Choose a tool

| Intent | Tool |
|--------|------|
| Project name, slug, default language | `get_project` |
| See what exists | `list_releases` / `get_release` |
| New note, not shipped yet | `create_release` → draft. No `publishedAt` |
| Already shipped / backfill | `create_releases` → **published**. Optional past `publishedAt` |
| Edit draft / scheduled customer copy | `update_release` |
| AI rewrite from source (BYOK) | `rewrite_release_customer_copy` |
| Rewrite **published** default-locale copy | `put_release_translation` with the project default locale (sets `editedAt`, no email) |
| Add or replace a **language** (including published) | `put_release_translation` |
| Ship a draft now | `publish_release` (emails matching subscribers) |
| Hold for later | `schedule_release` (`scheduledAt` future ISO-8601, **`timeZone` IANA required**) |
| Pull off the public page | `unpublish_release` → draft. No email |
| Remove forever | `delete_release` (cannot undo). Prefer unpublish when unsure |

Feature requests (`create_request`, `vote_request`, `approve_request`, …) are
separate from changelog notes.

## create_release vs create_releases

- **`create_release`**: one draft. Source fields freeze. Release manager edits
  customer copy, then publishes. **Rejects `publishedAt`.**
- **`create_releases`**: `{ "releases": [ … ] }`, 1–50 items, each **published
  immediately**. Past `publishedAt` allowed. Existing **versions returned as-is**
  (no overwrite, no merge). **Does not email.** One quota call per batch.

Never use `create_release` + `publish_release` to fake a historical date.

## GitHub-shaped notes already in the project

`list_releases` first. Action / release-please drafts often freeze GitHub
source as customer copy (title is `v0.6.0`, body still has `feat:` / PR links).
Rewrite with [customer-facing-changelog](../customer-facing-changelog/SKILL.md),
then:

1. Default locale: `update_release` (draft / scheduled) or
   `put_release_translation` with the default locale (published).
2. Other locales: `put_release_translation` **before** `publish_release` if the
   user asked for translations.
3. `publish_release` only for drafts the user asked to ship (emails
   subscribers). Do not unpublish a live note just to edit copy.

## Body and changes

Send Markdown `body` with recognised headings (`Features`, `Fixes`,
`Maintenance`, …). The server parses list items into `customerChanges`.

If a call fails with a marshalling / nested-object error on `changes`, **retry
without the `changes` array** and keep the lists in `body`.

## Locales

Call `get_project` for `defaultLocale`, `slug`, `name`, and `description`. An
empty `list_releases` has no slug. Prefer omitting `locale` on `create_*` so the
server uses the project default, or pass the value from `get_project`. Do not
guess `en`.

- If you pass a non-default locale, default-locale fields are still filled
  with the same text. Then call `put_release_translation` for the real default
  language (and any others).
- `put_release_translation` on the **default** locale updates
  `customerTitle` / `customerBody` and may set `editedAt`. On another locale it
  only updates `translations.{locale}`. One quota call per locale per note.
- Readers use `?lang=` on the public changelog; missing copy falls back to
  default.

Full workflow: [release-compass-translations](../release-compass-translations/SKILL.md).

## Delete and recreate

`create_releases` will not fix a bad `publishedAt` or body on an existing
version. `unpublish_release` then `publish_release` sets `publishedAt` to now.

To replace a version: `delete_release` → `create_releases` with the intended
`publishedAt` and body → `put_release_translation` for other languages.

## Safety

- Confirm with the user before `delete_release` or bulk `create_releases`.
- Do not print API keys or JWT.
- Public changelog JSON (use `slug` from the MCP payload):
  `GET https://api.release-compass.app/api/v1/public/changelogs/{slug}`
