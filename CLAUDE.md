# CLAUDE.md — adp-docs

Guidance for any Claude Code session — scheduled routine or interactive — that edits this repository. The scheduled ADP docs routines clone this repo, so they read this file too.

## Maturity status policy for managed MCP servers (HARD RULE)

A managed MCP server's maturity in the docs MUST be grounded in the cloudv2 registration source. Never infer it from a reviewer comment, a hunch, or "we haven't tested it internally yet."

- **Alpha:** ONLY when the server's `apps/aigw/.../<server>/register_mcp.go` sets `FeatureGate: "alpha"` in cloudv2. Such a server is early-access gated and creatable only on deployments where early-access features are enabled. Today only **SharePoint** qualifies. If `FeatureGate` is absent, the server is **not** alpha.
- **GA (no badge):** GitHub (Read), BambooHR, Aha, Google Calendar, Google Drive, Salesforce. This GA set is a product/policy decision confirmed with the team (adp-docs PR #74); treat the managed catalog (`modules/connect/pages/managed/managed-catalog.adoc`) as the rendered source of truth, and confirm with Michele before moving any server between tiers.
- **Beta (default for everything else):** `:page-beta: true` in the page header, and `badge:beta[label=beta]` on the server's row in the managed catalog.

Redpanda does NOT ship managed MCP servers as alpha as a maturity signal. If a reviewer asks to "mark as alpha" because something is untested, that is a product decision, not a docs edit — see the next rule.

## Never auto-incorporate reviewer suggestions (HARD RULE)

When a human reviewer suggests a content change on a PR (for example "mark these as alpha," reword something, add or remove a section):

- Do NOT push a commit that incorporates it on your own.
- Do NOT reply as Michele accepting or agreeing to it, or otherwise speak for her.
- Surface the suggestion for Michele to decide. You may post a comment that quotes the suggestion and flags it for her, but do not act on it.
- This applies with special force to maturity/status (alpha/beta/GA), product positioning, support statements, pricing, roadmap, and anything policy-sensitive.

Routine factual corrections grounded in cloudv2 source (the kind the automated critic flags — wrong field names, defaults, CLI flags, endpoints) may still be applied. Reviewer *opinions* and product decisions may not.
