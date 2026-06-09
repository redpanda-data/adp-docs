# CLAUDE.md — adp-docs

Guidance for any Claude Code session — scheduled routine or interactive — that edits this repository. The scheduled ADP docs routines clone this repo, so they read this file too.

## About this repo

adp-docs is the Antora source for the Agentic Data Plane docs (component `agentic-data-plane`, single version). It is its own Antora component, separate from the docs.redpanda.com multi-repo system. Managed MCP server pages live under `modules/connect/pages/managed/`. Follow the redpanda-data/docs-team-standards style guide, terminology, and structure.

## Build and test

- `npm run build` — Antora build (output to `docs/`)
- `npm run serve` — serve the built site on port 5002
- `npm run start` — build + livereload dev server
- `npm run test-docs` — Doc Detective tests over `modules/`

## AsciiDoc and Antora conventions

- Use full Antora resource IDs with module prefixes in xrefs (for example `xref:connect:managed/servicenow.adoc[]`), not relative paths.
- Do NOT use inline ifdef syntax — it renders as raw markup. Use block-level `ifdef::[]` / `endif::[]` directives instead.

## Maturity status policy for managed MCP servers (HARD RULE)

**The docs never use "alpha" as a maturity label.** The only maturity tiers we publish are GA and beta. A server's maturity is grounded in — but not dictated by — the cloudv2 registration source; never infer it from a reviewer comment, a hunch, or "we haven't tested it internally yet."

- **Alpha-gated in source = human decision, not a docs label.** If a server's `apps/aigw/.../<server>/register_mcp.go` sets `FeatureGate: "alpha"` in cloudv2 (SharePoint does today), the server is early-access only and not GA-ready. Do NOT add, update, or label its page on your own, and do NOT write "alpha" anywhere as its maturity. Flag it for Michele or the documentation team, who decide whether to ship it documented as **beta** or leave it out of the docs entirely.
- **GA (no badge):** GitHub (Read), BambooHR, Aha, Google Calendar, Google Drive, Salesforce. This GA set is a product/policy decision confirmed with the team (adp-docs PR #74); treat the managed catalog (`modules/connect/pages/managed/managed-catalog.adoc`) as the rendered source of truth, and confirm with Michele or the documentation team before moving any server between tiers.
- **beta (default for everything we do ship):** `:page-beta: true` in the page header, and `badge:beta[label=beta]` on the server's row in the managed catalog.

Redpanda does NOT publish managed MCP servers labeled alpha. If a reviewer asks to "mark as alpha" because something is untested, that is a product decision, not a docs edit — see the next rule.

## Never auto-incorporate reviewer suggestions (HARD RULE)

When a human reviewer suggests a content change on a PR (for example "mark these as alpha," reword something, add or remove a section):

- Do NOT push a commit that incorporates it on your own.
- Do NOT reply as Michele accepting or agreeing to it, or otherwise speak for her.
- Surface the suggestion for Michele or the documentation team to decide. You may post a comment that quotes the suggestion and flags it for them, but do not act on it.
- This applies with special force to maturity/status (alpha/beta/GA), product positioning, support statements, pricing, roadmap, and anything policy-sensitive.

Routine factual corrections grounded in cloudv2 source (the kind the automated critic flags — wrong field names, defaults, CLI flags, endpoints) may still be applied. Reviewer *opinions* and product decisions may not.
