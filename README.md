# Release Compass skills

Reusable [Agent Skills](https://agentskills.io) for writing **customer-facing**
changelog notes and importing them with the
[Release Compass](https://release-compass.app) MCP.

This repository is a **Cursor plugin**. Skills can also be copied into any
Agent Skills folder. It contains **no product-specific changelogs**. Point the
MCP at **your** project API key.

## Skills

| Skill | When to use |
|-------|-------------|
| [customer-facing-changelog](skills/customer-facing-changelog/SKILL.md) | Rewrite Keep a Changelog / GitHub releases / commits for readers |
| [import-historical-releases](skills/import-historical-releases/SKILL.md) | Bulk-import already shipped notes with original `publishedAt` |
| [release-compass-mcp](skills/release-compass-mcp/SKILL.md) | List, create, publish, delete, and other MCP tools |
| [release-compass-translations](skills/release-compass-translations/SKILL.md) | Add languages on drafts or published notes |

## Install

### Cursor plugin (recommended)

1. In Cursor, open **Customize**.
2. Import this GitHub repository as a **team marketplace**.
3. Install **release-compass-skills**.
4. Set **Release Compass API key** (`rc_live_…`) when prompted. Create the key
   in the [dashboard](https://release-compass.app/dashboard/) (name it e.g.
   `MCP`).

The plugin MCP URL is `https://api.release-compass.app/api/v1/mcp`.

### Copy skills only

Use this when you are not installing the Cursor plugin. You still need to
configure MCP yourself.

```bash
git clone https://github.com/tech-trail-io/release-compass-skills.git
cp -R release-compass-skills/skills/* your-repo/.cursor/skills/
```

Or clone into `~/.cursor/skills/` (user-level). Cursor also loads
`.agents/skills/`.

```json
{
  "mcpServers": {
    "release-compass": {
      "url": "https://api.release-compass.app/api/v1/mcp",
      "headers": {
        "Authorization": "Bearer rc_live_…"
      }
    }
  }
}
```

## Typical flow

1. Rewrite the technical changelog (show a draft table first). Heading dates
   become `publishedAt` instants.
2. Import with `create_releases` `{ "releases": [ … ] }` and past
   `publishedAt`. Omit `locale` so the project default is used. Do not use
   `create_release`.
3. Attach other locales with `put_release_translation` (one MCP call per
   locale per note).
4. If a version is wrong, `delete_release` then bulk-create again — existing
   versions are never overwritten.

Docs: [release-compass.app/docs](https://release-compass.app/docs/) ·
[llms.txt](https://release-compass.app/llms.txt)

## License

[MIT](LICENSE)
