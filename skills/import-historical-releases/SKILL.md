---
name: import-historical-releases
description: >-
  Imports an existing changelog into Release Compass as already-published
  historical notes with original publishedAt timestamps. Use when backfilling
  a public changelog, migrating Keep a Changelog or GitHub releases, or when
  the user asks to bulk-create published notes (not drafts).
---

# Import historical releases

Backfill notes that **already shipped**. They must be published with the original
dates, not opened as drafts dated “now”.

## Preconditions

1. Rewrite copy with [customer-facing-changelog](../customer-facing-changelog/SKILL.md).
2. Show the draft table. **Do not create until the user approves.**
3. Discover MCP tools on the configured Release Compass server
   (`list_releases`, `create_releases`, `delete_release`,
   `put_release_translation`). See [release-compass-mcp](../release-compass-mcp/SKILL.md).
4. `list_releases` first. Existing **versions are returned as-is** and are not
   overwritten.

## Create path

Use MCP **`create_releases`**. The argument is a **`releases` array**, not a
flat item. Do **not** use `create_release` — that opens a **draft** and rejects
`publishedAt`.

```json
{
  "releases": [
    {
      "version": "v1.4.0",
      "title": "Invoice export",
      "body": "You can export invoices as CSV.\n\n## Features\n- Invoice export as CSV",
      "publishedAt": "2026-03-03T12:00:00.000Z"
    }
  ]
}
```

Each item:

| Field | Required | Notes |
|-------|----------|--------|
| `version` | yes | Unique per project. Keep **one** prefix (`v1.4.0` **or** `1.4.0`) |
| `title` | yes | Customer headline |
| `body` | yes | Markdown. Headings parse into change lists. Do **not** include a “Released …” date line; `publishedAt` is shown on the public page |
| `channel` | no | If omitted, inferred from the version (`-rc` → `rc`, else often `stable`). Prereleases stay off the default public list |
| `locale` | no | **Omit** on the first import so the server uses the project default. Do not guess `en` |
| `publishedAt` | no | ISO-8601 **instant** (not date-only `2026-03-03`). Past allowed. Omit = now. Future rejected |
| `url` | no | Source URL if you have one |
| `changes` | no | Prefer Markdown lists. If nested objects error, **omit `changes`** |

Limits: **1–50** items per call. No subscriber email. Each `create_releases`
call is **one** quota use; `put_release_translation` is **one call per locale
per note**.

## publishedAt

Convert the real ship time to a UTC instant.

1. git tag `creatordate`
2. Keep a Changelog / GitHub heading date: `## 1.4.0 (2026-03-03)` →
   `2026-03-03T12:00:00.000Z` (or a more precise time if you have one)
3. Ask the user
4. **Omit only if the user wants “now”**

```bash
git for-each-ref --sort=-creatordate \
  --format='%(refname:short) %(creatordate:iso-strict)' refs/tags
```

Example: `2026-09-08T09:45:36+02:00` → `2026-09-08T07:45:36.000Z`

## After import

1. `list_releases` and check `status`, `publishedAt`, and `translations`. Use
   `ownerSlug`, project `slug`, and `changelogUrl` from the payload — do not
   invent paths.
2. Public JSON (includes `project.defaultLocale`):
   `GET https://api.release-compass.app/api/v1/public/changelogs/{owner}/{project}?lang={locale}`
3. Add other languages with **`put_release_translation`** (works on published
   notes). Do not unpublish just to add a language.
4. Wrong `publishedAt` on a version: **`delete_release`**, then `create_releases`
   again with the correct instant. Re-calling `create_releases` for an existing
   version does nothing. `publish_release` after a draft sets `publishedAt` to
   **now**.

## Do not

- Import as drafts when the user asked for historical published notes
- Stack a second item with the same `version` expecting a translation
- Guess `locale` or omit `publishedAt` for notes that already shipped
- Leak API keys or dump full XML/JSON payloads
- Email-test by publishing live notes
