# Examples

Generic product notes. Do not copy these into a real project unchanged.

## Feature + maintenance

**Source**

```markdown
## 1.4.0 (2026-03-03)
* feat(billing): CSV invoice export (#88)
* chore(deps): bump lodash and actions (#91)
```

**Customer**

Title: `Invoice export`

```markdown
Finance can download invoices as CSV.

## Features
- Invoice export as CSV

## Maintenance
- Security maintenance
```

## Internal only

**Source**

```markdown
## 1.3.1 (2026-02-18)
* ci: pin runners to ubuntu-24.04
* refactor: extract mapper helpers
```

**Customer**

Title: `Internal changes only.`

```markdown
Internal changes only.
```

## Hide secrets and internals

**Source** (do not leak this)

```markdown
* feat: per-customer margin cascade and FX rates
* feat: Redis session store for SSO
```

**Customer**

Talk about what the user sees (prices that apply to them, sign-in), never
margins, FX engines, or session stores.

Title: `Company prices and sign-in`

```markdown
Catalog prices match your company. Sign-in is more reliable.

## Features
- Catalog prices for your company
- Sign-in reliability
```

## Multi-user / languages

**Source**

```markdown
* feat: invite colleagues to the same org
* feat: i18n for en and de
```

**Customer**

Title: `Team invites and more languages`

```markdown
Invite colleagues to the same company account. Use the product in English or German.

## Features
- Several users per company account
- English and German
```
