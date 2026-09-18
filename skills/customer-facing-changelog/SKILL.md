---
name: customer-facing-changelog
description: >-
  Rewrites technical CHANGELOG.md files, GitHub release notes, and conventional
  commits into customer-facing changelog copy. Use when importing notes into
  Release Compass, drafting public release notes, or when the user asks to hide
  internals and write for customers.
---

# Customer-facing changelog

Turn engineer-facing source notes into copy a product user can understand.

Do **not** create or publish anything until the user has seen a draft table
(or equivalent) and asked you to proceed.

## Goal

Each note is a short headline plus Markdown body. Readers should learn what
**they** can do now — not how it was built.

## What to keep

- User-visible features, fixes, and reliability improvements
- New settings, languages, or channels they can choose
- Dependency/package bumps, described only as **Security maintenance**
- Versions that have no customer-facing change: still include them, with title
  and body **Internal changes only.**

## What to hide

- Implementation details (frameworks, tables, queues, flags, file paths)
- Security architecture, auth internals, crypto, WAF, IP lists
- Business secrets (pricing formulas, margins, FX, supplier names, quotas)
- CI, Docker, lint, test-only, and refactor PRs
- Ticket IDs, PR numbers, and GitHub handles unless the user asks for them
- Raw commit subjects (`feat(api): …`)

If a version is **only** the items in “What to hide”, use **Internal changes only.**

## Voice

- Second person (“you can…”)
- Product language, not git language
- One short intro paragraph, then headed lists
- No hype, no internal codenames
- No ship date in the body (`Released 3 March 2026`, `Megjelent …`). The public
  page already shows `publishedAt`.

## Markdown shape

Recognised headings become grouped change lists on the public changelog:

`Features`, `Added`, `Changed`, `Bug fixes`, `Fixes`, `Improvements`,
`Breaking`, `Chore`, `Maintenance`

```markdown
Short intro of what changed for you.

## Features
- Export invoices as CSV

## Fixes
- Sign-in after an idle session

## Maintenance
- Security maintenance
```

Optional detail after an em dash becomes a description:

```markdown
- Billing export — CSV for finance
```

Keep English heading names even when the body is another language, so the
parser can group lists. Put translated bullets under those headings.

## Title

A short customer headline, not the version string (`v0.6.0`) and not a commit
subject.

| Source | Title |
|--------|--------|
| `feat: add CSV invoice export` | Invoice export |
| `chore(deps): bump lodash` | Security maintenance |
| infra-only release | Internal changes only. |

## Version list

Walk the source changelog **newest first**. Include every version the user
asked to import. Do not skip internal versions. Keep one version prefix
(`v1.4.0` or `1.4.0`, not mixed).

`publishedAt` is an ISO-8601 **instant**. Prefer git tag `creatordate`, then a
Keep a Changelog / GitHub heading date (`## 1.4.0 (2026-03-03)` →
`2026-03-03T12:00:00.000Z`). Ask the user only if neither exists. **Omit
`publishedAt` only when the user wants “now”.**

## Draft table (required before create)

Show the user a table, then wait:

| Version | Title | Summary | publishedAt (UTC) | Notes |
|---------|-------|---------|-------------------|-------|
| v1.4.0 | Invoice export | CSV for finance | 2026-03-03T12:00:00.000Z | |
| v1.3.1 | Internal changes only. | Internal changes only. | 2026-02-18T09:12:00.000Z | internal |

After approval, follow [import-historical-releases](../import-historical-releases/SKILL.md)
and [release-compass-mcp](../release-compass-mcp/SKILL.md).

## Notes already in Release Compass

`list_releases` first. GitHub Action / release-please often freeze the GitHub
release as `customerTitle` / `customerBody`. Rewrite when any of these are true:

- `customerTitle` is the version string (`v0.6.0`)
- Body still has `feat:`, PR URLs, `@handles`, or `## What's Changed`
- Body repeats a ship date the UI already shows

Then write customer copy as usual and save it:

| Status | Default locale | Other locales |
|--------|----------------|---------------|
| Draft / scheduled | `update_release` | `put_release_translation` |
| Published | `put_release_translation` with the **default** locale | `put_release_translation` |

Do not unpublish to rewrite copy. If the user wants extra languages on drafts,
attach them **before** `publish_release`. Confirm before publishing (emails
subscribers). See [release-compass-translations](../release-compass-translations/SKILL.md).

## Examples

See [examples.md](examples.md).
