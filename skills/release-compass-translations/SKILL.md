---
name: release-compass-translations
description: >-
  Adds or replaces customer-facing changelog translations on Release Compass
  notes, including after publish. Use when the user asks for Hungarian, English,
  German, or other locales, ?lang= on the public changelog, or
  put_release_translation.
---

# Release Compass translations

Extra languages are **translations of customer title, body, and change lists** —
not separate versions. Version strings stay unique per project.

## When to use which tool

| Note status | Default locale copy | Other locales |
|-------------|---------------------|----------------|
| Draft / scheduled | `update_release` (`locale` optional) | `put_release_translation` |
| Published | `put_release_translation` with the **default** locale (sets `editedAt`) | `put_release_translation` |

Do **not** unpublish just to add a language. Do **not** send a second
`create_releases` item with the same `version` and a different `locale` —
existing versions are returned as-is.

## put_release_translation

Required: `id`, `locale` (BCP 47, e.g. `hu`, `en`, `de`), `title`, `body`.

Optional `changes`. If nested `changes` errors, omit it; keep headed lists in
`body` so the parser can fill change lists.

Does not publish. Does not email subscribers. One MCP quota call per note per
locale — bulk-create or rewrite first, then translate.

## Before publish

If the user wants extra languages on **drafts**, call `put_release_translation`
**before** `publish_release` so the first public view and subscriber email
include them. Adding a language after publish is also supported and does not
email. `publishedAt` stays the original instant.

When rewriting **published** notes, call the default locale (updates
`customerTitle` / `customerBody`) and each other locale. Do not unpublish.

## Copy rules

Follow [customer-facing-changelog](../customer-facing-changelog/SKILL.md) in
**each** language. Translate meaning, not commit subjects.

Keep recognised English headings (`## Features`, `## Fixes`, `## Maintenance`)
so grouping still works; translate the bullets and intro.

Internal versions: translate the same phrase, e.g. English `Internal changes only.`
and the equivalent in the other language.

## Verify

Use `slug` from the MCP release payload (do not invent it):

```text
GET https://api.release-compass.app/api/v1/public/changelogs/{slug}?lang=hu
GET https://api.release-compass.app/api/v1/public/changelogs/{slug}?lang=en
```

Check `availableLocales`, each item’s `locales` array, and that titles **differ**
when you wrote different copy. Fallback to the default locale if a language is
missing is expected.

`publishedAt` must stay the original instant after adding translations.
