<p align="center">
  <img src="assets/logo.svg" alt="Release Compass" width="120">
</p>

# Release Compass Skills

Reusable [Agent Skills](https://agentskills.io) for writing **customer-facing**
changelog notes, importing them with the
[Release Compass](https://release-compass.app) MCP, and embedding changelogs
or request boards with the
[client libraries](https://github.com/tech-trail-io/release-compass-clients).

This repository is a **Cursor plugin** and a **Claude Code plugin**. Skills can
also be copied into any Agent Skills folder. It contains **no product-specific
changelogs**. Point the MCP at **your** project API key.

## Skills

| Skill                                                                          | When to use                                                                               |
| ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------- |
| [customer-facing-changelog](skills/customer-facing-changelog/SKILL.md)         | Rewrite Keep a Changelog / GitHub releases / commits for readers                          |
| [import-historical-releases](skills/import-historical-releases/SKILL.md)       | Bulk-import already shipped notes with original `publishedAt`                             |
| [release-compass-mcp](skills/release-compass-mcp/SKILL.md)                     | List, create, publish, delete, and other MCP tools                                        |
| [release-compass-translations](skills/release-compass-translations/SKILL.md)   | Add languages on drafts or published notes                                                |
| [release-compass-embed-clients](skills/release-compass-embed-clients/SKILL.md) | Embed public React/Angular changelogs and boards; Next/core for private boards and voting |

In Claude Code, skills are namespaced:
`/release-compass-skills:customer-facing-changelog`, and so on.

## Install

Create a project API key in the
[dashboard](https://release-compass.app/dashboard/) (name it e.g. `MCP`). The
plugin MCP URL is `https://api.release-compass.app/api/v1/mcp`.

### Cursor plugin

1. In Cursor, open **Customize**.
2. Import this GitHub repository as a **team marketplace**.
3. Install **release-compass-skills**.
4. Set **Release Compass API key** (`rc_live_…`) when prompted.

### Claude Code plugin

```text
/plugin marketplace add tech-trail-io/release-compass-skills
/plugin install release-compass-skills@release-compass-skills
```

Set the project API key when prompted. Reload with `/reload-plugins` if MCP
tools do not appear. You can also load a local clone:

```bash
claude --plugin-dir /path/to/release-compass-skills
```

### Claude Desktop / claude.ai

There is no plugin marketplace there. Add a custom connector to
`https://api.release-compass.app/api/v1/mcp` with
`Authorization: Bearer rc_live_…`. Copy skills into `~/.claude/skills/` if you
want the rewrite/import workflows too.

### Copy skills only

Use this when you are not installing a plugin. You still need to configure MCP
yourself.

```bash
git clone https://github.com/tech-trail-io/release-compass-skills.git
cp -R release-compass-skills/skills/* your-repo/.cursor/skills/
# Claude Code project skills:
cp -R release-compass-skills/skills/* your-repo/.claude/skills/
```

Or clone into `~/.cursor/skills/` or `~/.claude/skills/` (user-level). Cursor
also loads `.agents/skills/`.

```json
{
  "mcpServers": {
    "release-compass": {
      "type": "http",
      "url": "https://api.release-compass.app/api/v1/mcp",
      "headers": {
        "Authorization": "Bearer rc_live_…"
      }
    }
  }
}
```

## Typical flow

1. `list_releases`. If customer copy is still GitHub source (version-as-title,
   `feat:`, PR links), rewrite it — drafts with `update_release`, published
   notes with `put_release_translation` on the default locale. Do not put a
   “Released …” line in the body; the page already shows `publishedAt`.
2. For a new import, rewrite the technical changelog (show a draft table
   first). Heading dates become `publishedAt` instants.
3. Import with `create_releases` `{ "releases": [ … ] }` and past
   `publishedAt`. Omit `locale` so the project default is used. Do not use
   `create_release`.
4. Attach other locales with `put_release_translation` (one MCP call per
   locale per note) **before** `publish_release` when the notes are still
   drafts.
5. If a version is wrong, `delete_release` then bulk-create again — existing
   versions are never overwritten.

Docs: [release-compass.app/docs](https://release-compass.app/docs/) ·
[llms.txt](https://release-compass.app/llms.txt)

## License

[MIT](LICENSE)
