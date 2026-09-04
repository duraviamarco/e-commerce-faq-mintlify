# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A [Mintlify](https://mintlify.com) documentation site: the "Centro Assistenza 3DZ" support/FAQ center for **3DZ Shop** (Italian 3D-printing e-commerce). Content is MDX pages with YAML frontmatter; site structure and nav live in `docs.json`.

**This content is a clone of the live FAQ site https://help.3dzshop.com/** — that site is the source of truth for what content should exist and what it should say. When adding, verifying, or restructuring a section, check the corresponding page on help.3dzshop.com rather than assuming existing repo content is accurate; several pages here were previously fabricated by an earlier agent session and had to be removed (e.g. `prodotti/catalogo-prodotti`, `prodotti/certificazioni`, `prodotti/compatibilita-materiali` do not exist on the source and were deleted).

## Commands

Install the Mintlify CLI and preview locally (run from the repo root, where `docs.json` lives):

```bash
npm i -g mint
mint dev            # preview at http://localhost:3000
mint update          # if the dev server won't start / CLI seems stale
```

There is no build/lint/test suite — this is a content-only repo. The only "validation" is that `docs.json` is well-formed JSON matching the [Mintlify schema](https://mintlify.com/docs.json) and that every page listed in its navigation exists on disk.

## Deployment

- **GitHub repo**: `duraviamarco/e-commerce-faq-mintlify`, deploy branch `main`.
- **Mintlify deployment**: subdomain `3-dz-spa`, live at `https://3-dz-spa.mintlify.app`.
- Pushing to `main` triggers an automatic deploy. There is no PR/staging step required — a push that produces invalid `docs.json` (bad JSON, or config schema violations like duplicate `redirects` sources) simply fails to deploy and the site stays on the last good build.
- This repo can also be edited through Mintlify's web dashboard editor (which works via sessions/branches and PRs against this repo). **That pipeline has proven unreliable on this deployment**: saving from the editor has repeatedly produced a corrupted `docs.json` (the `Centro Assistenza` navigation group duplicated dozens of times, phantom references to pages/directories that don't exist, `tabs` and `groups` both present at the top level). For any nontrivial navigation restructuring, prefer cloning the repo locally, editing files with normal git, and pushing directly to `main` — then re-fetch `docs.json` from GitHub afterward to confirm it's still clean before trusting a deploy.

## Navigation architecture (`docs.json`)

Single tab (`Centro Assistenza`) containing 6 groups, in this order: `Home`, `Informazioni Generali`, `Ordini, Spedizioni e Resi`, `Preventivi`, `Prodotti e Schede Sicurezza`, `Problemi Tecnici`. Each group's `pages` array is an ordered list of MDX paths (no extension, no leading slash).

Icons follow the Lucide icon set by convention (e.g. `circle-info`, `truck`, `box`) — no custom icon library is configured.

There is no `snippets/` directory in this repo. Do not add `import ... from "/snippets/..."` to any page — earlier content inherited from another branch referenced `/snippets/Header3DZ.mdx` and `/snippets/Footer3DZ.mdx` that never existed here, which would break the build; those imports were stripped.

### The index-page-per-section pattern

Four of the five content sections (`informazioni-generali`, `ordini`, `preventivi`, `problemi-tecnici`) are a **decomposition of a single FAQ page from help.3dzshop.com into one MDX file per question**, plus an `index.mdx` for the section that renders a `<CardGroup>` linking to every sibling page in that folder. On the source site each of these sections is one page with expandable accordions; here each accordion answer became its own page for Mintlify's navigation model. When you add or remove a question page in one of these folders, you must also:

1. Add/remove it from that section's `index.mdx` `<CardGroup>` (title copied verbatim from the source site's accordion question, matching frontmatter `title`).
2. Add/remove it from the corresponding group in `docs.json`, keeping the section's own `index` as the **first** entry in the group's `pages` array (so the section's overview/index page is what a reader lands on first).

**`prodotti/schede-sicurezza.mdx` is the exception** — on the source site this section is not FAQ-style; it's a single page with an intro and four cards that link straight to *external* SharePoint URLs (one per brand: 3D Systems, 3D Systems - Resine, Markforged, Formlabs). It has no sub-pages and its `docs.json` group only ever lists that one path. Do not add a `catalogo`, `certificazioni`, or `compatibilita` page under `prodotti/` unless help.3dzshop.com actually has one — as of this writing it doesn't (and the `/compatibilita.html` link on the homepage is a dead 404 even on the source site, so that section is intentionally omitted here too).

## Editing with the Mintlify MCP / GitHub MCP

- The Mintlify Admin MCP's `checkout` → edit → `save` flow works for small, isolated content edits but has shown the corruption behavior described above for structural nav changes (adding/removing/reordering groups or pages) — its in-session node tree can silently desync from what's actually committed on `main`. Always re-read `docs.json` from `main` after a `save`/merge to confirm the result.
- If using the GitHub MCP's `create_or_update_file` directly: pass the file content as **plain text**, not pre-base64-encoded — the tool encodes it itself. Double-encoding was a real bug that landed a base64 string as the literal committed content of `docs.json`.
