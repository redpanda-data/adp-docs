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
- `ifdef` is a preprocessor directive and must start its own line. Prefer the block form (`ifdef::[]` / `endif::[]`); the single-line form `ifdef::attr[text]` also works at line start. Do NOT use `ifdef` mid-sentence (truly inline in prose) — there it renders as raw markup.

## Maturity status policy for managed MCP servers (HARD RULE)

**The docs never use "alpha" as a maturity label.** The only maturity tiers we publish are GA and Preview. A server's maturity is grounded in — but not dictated by — the cloudv2 registration source; never infer it from a reviewer comment, a hunch, or "we haven't tested it internally yet."

- **Alpha-gated in source = human decision, not a docs label.** If a server's `apps/aigw/.../<server>/register_mcp.go` sets `FeatureGate: "alpha"` in cloudv2, the server is early-access only and not GA-ready. <!-- e.g. SharePoint as of 2026-06-09 --> Do NOT add, update, or label its page on your own, and do NOT write "alpha" anywhere as its maturity. Flag it for Michele or the documentation team, who decide whether to ship it documented as **Preview** or leave it out of the docs entirely.
- **GA (no badge):** a managed MCP server is GA when its row in the managed catalog (`modules/connect/pages/managed/managed-catalog.adoc`) carries no Preview badge; that catalog is the source of truth. As of 2026-06-09 the GA set is GitHub (Read), BambooHR, Aha, Google Calendar, Google Drive, Salesforce. <!-- GA snapshot 2026-06-09 — re-verify against managed-catalog.adoc --> Confirm with Michele or the documentation team before moving any server between tiers.
- **Preview (default for everything we do ship):** `:page-preview: true` in the page header, and `badge:preview[label=Preview]` on the server's row in the managed catalog. The visible label is "Preview" (never "beta" or "alpha").

Redpanda does NOT publish managed MCP servers labeled alpha. If a reviewer asks to "mark as alpha" because something is untested, that is a product decision, not a docs edit — see the next rule.

## Never auto-incorporate reviewer suggestions (HARD RULE)

When a human reviewer suggests a content change on a PR (for example "mark these as alpha," reword something, add or remove a section):

- Do NOT push a commit that incorporates it on your own.
- Do NOT reply as Michele accepting or agreeing to it, or otherwise speak for her.
- Surface the suggestion for Michele or the documentation team to decide. You may post a comment that quotes the suggestion and flags it for them, but do not act on it.
- This applies with special force to maturity/status (alpha/beta/Preview/GA), product positioning, support statements, pricing, roadmap, and anything policy-sensitive.

Routine factual corrections grounded in cloudv2 source (the kind the automated critic flags — wrong field names, defaults, CLI flags, endpoints), plus unambiguous typo, grammar, and broken-link fixes that don't change meaning, may still be applied. Reviewer *opinions* and product/maturity decisions may not.

## Public-surface rule for automated routines (HARD RULE)

This repository is public. Automated routines must not post review findings, source analysis, commit provenance, or internal status to any public surface of this repo: no PR comments or reviews, and no internal references (private repo names or commit SHAs, source file paths, feature flag or gate names, internal tickets, unshipped or reverted features, session links) in PR titles, PR descriptions, commit messages, or branch names. PR descriptions and commit messages describe only the documentation changes themselves. All other routine output goes to the docs team's internal channel. If the internal channel is unreachable, fail the run; never fall back to posting the content here. For automated routines, this rule supersedes the comment allowance in the previous section: flag reviewer suggestions through the internal channel, not a PR comment.
