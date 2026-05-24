# ADP Package 2 — Doc Verification & Update Plan

Source briefing: https://docs.google.com/document/d/1r5anOCrF3sqptzRX9T_4Wc5ryCMwU8sHi4tnGF0Rr1Y/edit
Scope of this pass: six features the briefing marks 🟢 Shipped or in-production: LLM Proxy, Managed MCP, Microsoft Teams integration, Cost Tracking, MCP OAuth Clients, and OAuth Provider / Token Vault. (LLM Proxy, Managed MCP, Cost Tracking, MCP OAuth Clients, and Claude Code updates shipped 2026-05-24 on branch `adp-package-2-briefing-doc-catchup`.)
Doc tree reviewed: `modules/ai-gateway`, `modules/mcp`, `modules/agents`, `modules/integrations`, `modules/governance`, `modules/reference`, `modules/ROOT/nav.adoc`.

---

## TL;DR

| Briefing feature | % per briefing | Docs state | Action needed |
|---|---|---|---|
| LLM Proxy | 95%+ | Mostly covered; two specific gaps | ~~Add transcript-logging section; finish Claude Code integration page; clarify Redpanda-managed model catalog~~ ✅ Done 2026-05-24 |
| Managed MCP | 95%+ | Well covered | ~~Light cleanup: add "Connection tab" naming, fill missing catalog deep-dives, optional codegen note~~ ✅ Connection-tab + deep-dive backlog flagged 2026-05-24 |
| Microsoft Teams integration | 95%+ | **Not documented at all** | New page + nav entry + update of `integration-overview.adoc`. Paused pending eng verification: see memory `project-adp-teams-integration-doc-gap` |
| Cost Tracking | ~90% | Well covered; two small gaps | ~~Distinguish v1-shipped vs WIP-in-Package-2; surface the API surface more clearly~~ ✅ Spending API how-to added + dashboard shipped-vs-WIP flagged 2026-05-24 |
| MCP OAuth Clients | ~70% | Well covered; two specific gaps | ~~Add Microsoft Copilot Studio to client list; surface "no DCR in Package 2" limitation~~ ✅ Done 2026-05-24 |
| OAuth Provider / Token Vault | 95%+ | **Most thoroughly covered feature.** One minor gap | Optional: new `my-connections.adoc` page so users have a discoverable home for connection management |

LLM Proxy, Managed MCP, Cost Tracking, MCP OAuth Clients, and Claude Code integration have all been updated. OAuth Provider / Token Vault is the best-documented Package 2 area and does not need significant changes in this pass. Microsoft Teams is the standing gap: a 95%+-shipped feature with zero user-facing documentation.

---

## 1. LLM Proxy

### What the briefing claims

1. Configure LLM providers and their API keys; centralized observability of inputs/outputs, token usage, and tool invocations.
2. Five provider types: OpenAI, Google AI Studio, Anthropic (API key **and** Enterprise/Personal passthrough), AWS Bedrock, OpenAI-compatible (vLLM, DeepSeek, self-hosted).
3. Connect tab on the provider detail page showing how to connect via, for example, Claude Code.
4. Built-in connectivity check.
5. Models list: Redpanda maintains the catalog of **available** models, IT Admin ticks checkboxes to enable.
6. Transcript logging is configured per LLM provider, so an admin can create a "sensitive LLM provider" where no data is logged.

### What the docs cover today

- `modules/ai-gateway/pages/overview.adoc` — provider types, capabilities, when-to-use.
- `modules/ai-gateway/pages/configure-provider.adoc` — full form reference for each of the five types, including Anthropic Auth passthrough, Bedrock inference profiles, model picker, Test Connection.
- `modules/ai-gateway/pages/connect-agent.adoc` — proxy URL anatomy, OIDC client-credentials grant, native SDK examples, and a passing mention that the provider detail page has Claude Code / Codex / Gemini client guides.
- `modules/ai-gateway/pages/bedrock-setup.adoc` — IAM walkthrough.

### Gaps to fix

**Gap 1 — Transcript logging per provider is undocumented.** The briefing highlights this as a featured capability (screenshot of "Transcript logging is configured on LLM provider level… IT Admin is in charge"), including the "sensitive provider with no logging" use case. `configure-provider.adoc` has no section for it. A grep for "transcript" across `modules/ai-gateway` returns zero hits.

> *Plan:* Add a "Transcript logging" subsection to `configure-provider.adoc`, ideally between "Select models" and "Save and verify". Cover: where the toggle lives on the create/edit form, the default state, the data captured when enabled (inputs, outputs, tool calls, token counts) and the data still captured when disabled (request metadata, token totals — confirm with eng), and the canonical "sensitive provider" pattern (one provider with logging off for regulated workloads, alongside the default provider). Cross-link to `observability:transcripts.adoc` and `observability:concepts.adoc`.

**Gap 2 — The "Connect tab" / Claude Code page is a stub.** `modules/integrations/pages/claude-code.adoc` contains only `// TODO: Add content`. The detail-page snippets the briefing showcases (Claude Code, Codex, Gemini desktop) are only referenced in a NOTE in `connect-agent.adoc`.

> *Plan:* Fill `modules/integrations/pages/claude-code.adoc`. The two existing partials (`modules/integrations/partials/integrations/claude-code-admin.adoc` and `…-user.adoc`) carry the substance — but they describe the **old** AI Gateway v1 surface (separate "AI Gateway > Models" page, `…/ai-gateway/v1` endpoint shape, model IDs like `anthropic/claude-opus-4.6-5`) and need a rewrite against the current per-provider model selection and the `aigw.<cluster-id>.clusters.rdpa.co/llm/v1/providers/<name>/…` URL shape. Same retrofit applies to `cursor.adoc`, `continue.adoc`, `cline.adoc`, `copilot.adoc` — flag for a follow-up sweep, out of scope for this pass.

**Gap 3 — "Available models are Redpanda-maintained" is implicit, not stated.** `configure-provider.adoc#select-models` says "the form shows a picker backed by the provider's catalog" but doesn't say that Redpanda curates that catalog and adds new models within a day or two of their upstream release. That's a featured product claim in the briefing.

> *Plan:* Add a one-paragraph clarification near the model picker description: "The available-models list is maintained by Redpanda; new models published by the upstream provider typically appear in the picker within a day or two. Models you tick become the catalog this provider exposes; everything else stays gated." Light edit.

### Items the briefing implies but does not require a doc change for

- "OpenAI compatibility is a fallback that allows anything (DeepSeek, self-hosted vLLM)" — already covered in `configure-provider.adoc` and `overview.adoc`.
- Anthropic Enterprise / Max passthrough — already covered as "Auth passthrough" in `configure-provider.adoc` and `connect-agent.adoc`.
- Built-in connectivity check — already documented as "Test Connection" in the "Save and verify" section.

---

## 2. Managed MCP

### What the briefing claims

1. Redpanda-hosted, high-quality, response-optimized MCP servers with form-based config.
2. Per-user OBO (on-behalf-of) authentication so each end-user only sees what their own upstream account allows.
3. AI-driven codegen harness (vibe-coded servers via `protoc-gen-go-mcp`, Claude Skills for authoring/optimization/review).
4. 14 MCP servers shipped by Alex Gallego; full catalog with both managed and proxied servers shown together.
5. Marketplace-style Create page; autoform-rendered configuration (e.g., BambooHR subdomain field).
6. Connection tab on detail page (external-client connection URL).
7. Inspector tab for testing.

### What the docs cover today

- `modules/mcp/pages/overview.adoc` — managed vs. self-managed, 36 default types referenced.
- `modules/mcp/pages/managed/managed-catalog.adoc` — every managed type listed (categorized: AI, AWS, Communication, Database, Google, Streaming, Utility), with deep-dive links where they exist.
- `modules/mcp/pages/create-server.adoc` — marketplace picker, autoform-from-protobuf, five auth modes (None / Static key / Token passthrough / Service-account OAuth / User-delegated OAuth), code mode, CLI flow.
- `modules/mcp/pages/test-tools.adoc` — Inspector tab with Tools/Resources/Prompts/Session panels.
- `modules/mcp/pages/user-delegated-oauth.adoc` — full OBO flow with token vault, scope upgrade, refresh, contrast with service-account auth.
- `modules/mcp/pages/oauth-providers.adoc`, `…/github-oauth-tutorial.adoc` — OAuth provider configuration.
- Deep-dives: BambooHR, Ironclad, Jira, Kafka, Metabase, NetSuite, OpenAPI, Ramp, Slack, SQL, Workday, Zendesk.

### Gaps to fix

**Gap 1 — Detail-page "Connection" tab is not described by name for MCP.** `configure-provider.adoc` explicitly walks through the Connection card on the LLM-provider detail page. The MCP equivalent — Connection tab showing the MCP URL plus client-connection snippets — isn't called out as a discrete UI surface in `create-server.adoc` (it just says "the *Overview* tab shows the *API URL*").

> *Plan:* In `create-server.adoc` "Save and verify", split the post-create UI into the actual tabs visible on the detail page: *Overview*, *Connection* (URL + connection-snippet generator), *Inspector*, *Connect* (or whatever the live label is). Or, if there is no separate Connection tab and the URL really does sit on Overview, leave as-is but verify against `adp-production` before signing off — the briefing's screenshot caption "Connection tab. Shows how to connect" implies a real tab. **Open question to confirm with eng.**

**Gap 2 — Catalog deep-dive coverage is incomplete.** 12 of the 36 types have deep-dive pages. Untouched: Discord, GitHub (Read), Elasticsearch, MongoDB, Qdrant, Redis, GCP Pub/Sub, Google Calendar, Google Drive, NATS, Azure AD, BILL, DocuSign, Greenhouse, Morningstar Portfolio Analytics, Morningstar Securities, Okta, Salesforce, Text Chunker, AWS Bedrock-as-MCP, Cohere, OpenAI-as-MCP, AWS SNS, AWS SQS. The briefing called out 14 ships by Alex Gallego plus Marc Millstone's response-optimization work — the team can self-prioritize which ones get deep-dives next.

> *Plan:* Out of scope for "Package 2 verify" specifically. Surface as a separate tracking issue: which managed types need deep-dive pages before GA, and which can ride on the catalog row alone.

**Gap 3 — Optional: the AI-driven codegen story.** The briefing makes a point that managed MCP servers are vibe-coded via Claude Skills, and the harness uses `protoc-gen-go-mcp`. This is product-marketing material but might warrant a one-paragraph callout under "How managed servers stay current" — useful for prospects evaluating quality/cadence. Likely not user-doc territory, more a blog/case-study angle. **Defer unless product asks.**

### Items the briefing implies but does not require a doc change for

- OBO semantics / per-user upstream identity — already heavily documented in `user-delegated-oauth.adoc` and `create-server.adoc`.
- Inspector tab — already documented in `test-tools.adoc`.
- Marketplace picker + autoform — already documented in `create-server.adoc`.
- Managed + remote shown together — already in `overview.adoc`.

---

## 3. Microsoft Teams Integration — **MAJOR GAP**

### What the briefing claims

1. Implemented as an ask from GF; hacked at a Dresden workwith, then product-ized in 10 days.
2. A Managed Agent can be configured to be exposed via Microsoft Teams (screenshot of the toggle/config in the agent detail page).
3. End-users interact with the agent inside MS Teams (screenshot of the Teams chat).
4. Currently wired to **Agents v1**; will be ported to v2 (Agent Registry) when that lands.

### What the docs cover today

Nothing. A `\bTeams\b` grep across `modules/` returns only generic usages ("teams adopting LLMs"). There is no Teams integration page, no nav entry, and no mention in `integration-overview.adoc`. The three integration scenarios listed there are: agent invokes MCP, pipeline invokes agent, external system invokes agent — nothing for "chat platform invokes agent" / "agent surfaced in a chat platform".

### Gaps to fix

**Gap 1 — No page, no nav entry, no concept.**

> *Plan:* Add a new page. Two viable homes:
>
> - **Option A (recommended):** `modules/agents/pages/integrations/teams.adoc`. Pairs with the existing `integration-overview.adoc`, treats Teams as an agent frontend pattern. Add an "Agent surfaced in a chat platform" row to the scenarios table in `integration-overview.adoc` and link out.
> - **Option B:** `modules/integrations/pages/teams.adoc`. Lives alongside Claude Code, Cursor, etc. Risk: those integrations connect *clients* to ADP's LLM proxy, not chat platforms to *agents*. Conflating the two would confuse readers.
>
> Recommend Option A.
>
> Page should cover: prerequisites (Teams app permissions / bot registration in Azure AD or Entra; ADP-side managed-agent already created); configure-the-agent flow (the toggle the briefing screenshots show — exact label TBD with eng); end-user flow (where the bot shows up in Teams, how a user starts a conversation, auth boundary — whose identity Teams forwards, how that maps to ADP's OBO model); known limits (currently Agents v1 only; will be ported to Agents v2 when the registry ships — flag this **explicitly** so customers don't build long-term integrations on the v1 path); troubleshooting.
>
> Add nav entry to `modules/ROOT/nav.adoc` under the Agents section, after `integration-overview.adoc`.

**Gap 2 — `integration-overview.adoc` scenario table is incomplete.**

> *Plan:* Add a fourth scenario row: "User chats with agent from a chat platform" → "Chat-platform-initiated, interactive, end-user-facing" → link to the new Teams page. Phrase it generically so Slack-as-frontend (if/when it ships) can slot in later.

**Gap 3 — Agents v1 vs. v2 caveat.** The briefing is explicit that Teams is wired to v1 today and will be ported to v2. Docs need to surface this honestly so customers building on Teams know what's portable.

> *Plan:* In the new Teams page, include a "Roadmap and current limitations" admonition noting v1-only today and the eventual port. Cross-link to the Agent Registry RFC or relevant agents v2 doc once that exists.

### Items to confirm with eng before writing

- Exact label of the "expose via Teams" toggle/section in the agent detail page.
- Whether Teams users authenticate via the same Token Vault / OAuth Provider mechanism as MCP servers do, or via a Teams-specific path.
- Whether multiple agents can be exposed in the same Teams tenant simultaneously, and how the user picks among them.
- Tenant/admin install steps — who installs the Teams app, what permissions are required, is it App-Source-listed or sideloaded.

---

## Suggested PR breakdown

To keep reviews focused, split the work into four PRs:

1. **LLM Proxy: transcript logging + Redpanda-maintained models clarification.** Touches `configure-provider.adoc` only. Smallest blast radius. Ships first.
2. **Claude Code integration page rewrite.** Touches `modules/integrations/pages/claude-code.adoc` and its two partials. Flag the parallel pages (Cursor / Continue / Cline / Copilot) for a follow-up — same stale-architecture pattern in all of them.
3. **Microsoft Teams integration page (new).** Touches `modules/agents/pages/integrations/teams.adoc` (new), `modules/agents/pages/integration-overview.adoc`, `modules/ROOT/nav.adoc`. Blocked on eng confirmation of toggle label, auth model, and v1/v2 messaging.
4. **MCP Connection-tab naming check + catalog deep-dive backlog.** Touches `create-server.adoc` if eng confirms the tab exists; otherwise just opens a tracking issue for missing deep-dives.

## Open questions for eng before writing

- LLM Proxy: where in the create-LLM-provider form does the transcript-logging toggle live, and what exactly does it gate (capture of `gen_ai.prompt.*` / `gen_ai.completion.*` content fields only, or also `gen_ai.usage.*` token counts)?
- MCP: is there a real "Connection" tab on the MCP-server detail page, or does the API URL just sit on the Overview tab?
- Teams: needs the answers in the "Items to confirm with eng" subsection above.
- Engineering source of truth: `projects/cloudv2` (per project instructions) — verify the toggle/tab labels against the actual UI before committing copy.

---

## 4. Cost Tracking

### What the briefing claims

1. AI Gateway tracks every token used and writes it to Redpanda (Kafka topic-backed cost pipeline).
2. ADP provides an **API** and **UI** to query for usage.
3. Some "minor parts" still to do — explicitly: grouping by user / agent and drill-down into agent and user. Eng expects this to land before Jun 15.

### What the docs cover today

- `modules/governance/pages/budgets.adoc` — what ADP records automatically (input/output/cached tokens, cost in microcents, request count, provider/model/user/org context), how spend events flow through Kafka, per-request pricing variations (Anthropic fast mode, Gemini context-tier pricing), per-model rate-card overrides for chargeback.
- `modules/governance/pages/dashboard/overview.adoc` — KPI tabs (Total Spend / Requests / Tokens), date-range presets, chart filters (provider, model, cost type, token type, user), Agents table, Top users panel with heatmap.
- `modules/ai-gateway/pages/configure-provider.adoc` — *Cost & Usage* tab on the provider detail page: time-series charts, group-by (provider / model / token type), filter by provider/model/cost type/token type/user, sharable custom-range URLs.
- `modules/governance/pages/guardrails/cost-tracking.adoc` — per-evaluator cost shape (PII / Toxicity / Custom webhook), where evaluator cost surfaces, capping knobs.

This is the most thoroughly documented Package 2 area. Coverage is in good shape.

### Gaps to fix

**Gap 1 — The "what's shipped vs. what's WIP" line is blurry.** The briefing flags grouping by user/agent and per-agent / per-user drill-down as still-to-land work. The docs already describe a *Top users* panel and a *user* filter on the chart, and `budgets.adoc` references `GetSpendingBreakdown` with `user_id`. Either (a) those features are now live and the briefing is stale, (b) the docs got ahead of eng and document something that's not yet shipped, or (c) the docs describe a partial implementation. This needs reconciliation before customers find a feature gap.

> *Plan:* Send eng a short list of features the docs claim and ask which are live in Package 2 today vs. coming Jun 15: per-user filter on the dashboard chart, the *Top users* panel with heatmap, `SpendingFilter.user_id` on `GetSpendingBreakdown`, the *Agents* table, an Agent drill-down view from the dashboard, and the *user* filter on the AI Gateway *Cost & Usage* tab. Add a "Currently available" admonition in `dashboard/overview.adoc` covering what's live in Package 2 vs. what's coming. Avoid documenting unshipped surfaces — if the user-drill-down or agent-drill-down view isn't live, the *Top users* panel description should be marked as "coming in Package 2" or deferred entirely.

**Gap 2 — The API surface is under-promoted.** The briefing says ADP exposes an API to query usage. `budgets.adoc` mentions `SpendingService.GetSpendingBreakdown` and AIP-160 `filter` expressions in passing, but there's no how-to with a worked example (auth, endpoint URL, request body, response shape). The `modules/reference/pages/api.adoc` page is a `// TODO: Add content` stub. Customers comparing ADP to LangSmith / Helicone / etc. will look for "can I pull spend programmatically" — the answer is yes, but the docs don't show it.

> *Plan:* Add a new "Query spend programmatically" section in `budgets.adoc` (or a dedicated `modules/governance/pages/spending-api.adoc` if it grows past a section) with: the gRPC + Connect/REST shape of `GetSpendingBreakdown`, the canonical `SpendingFilter` body, a worked cURL example pulling per-user-per-day spend for the last 7 days, a worked Python example using the generated client, and the AIP-160 `filter` syntax. Cross-link from `dashboard/overview.adoc` "Next steps" and from the *Cost & Usage* tab section of `configure-provider.adoc`. **Out of scope:** the full `api.adoc` reference page rewrite — that's a separate project.

### Items the briefing implies but does not require a doc change for

- Every-token capture via Kafka topic — covered in `budgets.adoc`.
- Microcents unit and `total_cost_microcents` conversion — covered.
- Per-request pricing variations (Anthropic fast mode, Gemini context-tier) — covered.
- Per-model rate-card overrides for chargeback — covered.
- Anonymous-vs-identified user behavior in the Top users panel — covered.

### Open questions for eng

- Which of the following are live in Package 2 today, and which are WIP? Per-user filter on the dashboard chart; *Top users* panel with heatmap; agent drill-down from the *Agents* table; `SpendingFilter.user_id` in `GetSpendingBreakdown`; per-agent spend grouping.
- Is `user_id` populated automatically from OIDC claims today, or does the calling app need to inject it? (Also flagged in `budgets.adoc` TODO at line 75.)
- For the API how-to: confirm the canonical endpoint (gRPC reflection? Connect-Go? HTTP/JSON gateway?) and the authentication model (same OIDC service-account flow as `connect-agent.adoc`?).
- Is `organization_id` multi-tenant-aware in the public ADP API, or internal-only? (Same as `budgets.adoc` TODO at line 97.)

---

## 5. MCP OAuth Clients

### What the briefing claims

1. Lets external MCP clients connect *to* ADP-hosted MCP servers via incoming OAuth.
2. ADP runs an embedded Identity Provider so MCP clients can connect with no extra hacks like headers or local rpk proxy.
3. Works with **any compliant MCP client** — claude.ai, Claude Desktop, Claude Code, **Microsoft Copilot Studio**, etc.
4. Generic, based on the OAuth2 spec.
5. **Dynamic Client Registration (DCR) is explicitly out of scope for Package 2.**

### What the docs cover today

- `modules/integrations/pages/remote-mcp-clients.adoc` — comprehensive page covering:
  - When to use vs. when not to use (line 17–28).
  - Three-resource architecture (MCP server / OAuth Provider / OAuth Client) with a GitHub worked example (line 30–54).
  - Register Client form (Name, Grant Types, Redirect URIs) and Client ID + Client Secret retrieval (line 65–93).
  - Claude.ai / Claude Desktop wire-up walkthrough (line 95–125).
  - Brief mentions of ChatGPT, Gemini, Cursor (line 127–137).
  - Two-step OAuth flow with consent screen content (line 139–177).
  - Manage and rotate, including `rpk ai oauth-client revoke-tokens` (line 179–216).
  - Troubleshooting and Limitations (line 218–251).
- Embedded IdP reference: "Auth0 today, Zitadel later" at line 149.
- `modules/integrations/pages/claude-code.adoc` (newly written 2026-05-24) — covers Claude Code specifically, including MCP server attachment.

This is the second-most-thoroughly-documented Package 2 area.

### Gaps to fix

**Gap 1 — Microsoft Copilot Studio is missing from the named-clients list.** The briefing's OAuth Clients screenshot caption explicitly names "Claude.ai, Microsoft Copilot Studio, …" as examples. Grep across `modules/` returns zero hits for "Copilot Studio" or "copilot studio". This matters because Copilot Studio is the Microsoft-side equivalent of Claude's custom connector and is what unlocks Microsoft 365 / Teams users — a meaningful adjacency given the Teams integration GF asked for.

> *Plan:* Add a "Microsoft Copilot Studio" bullet to the "Wire up other chat clients" section in `remote-mcp-clients.adoc` (currently a three-bullet list of ChatGPT desktop / Gemini apps / Cursor at line 127–137). Cover: where Copilot Studio's custom-connector setup lives (Power Platform admin center? Copilot Studio agent designer?), the redirect URIs Microsoft requires (these need eng / a manual test to capture), and the same fields-required pattern (connector name, MCP URL, Client ID, Client Secret). Flag with a "// TODO" until a real walk-through is captured. **Open question:** does Copilot Studio actually pass MCP custom-connector tests against ADP today, or is it on the roadmap? The briefing's caption is illustrative — Michele should verify with eng that the integration is actually exercised before promising it in docs.

**Gap 2 — DCR-out-of-scope is not surfaced.** The briefing makes a point of calling out: "Dynamic Client Registration (DCR) is out of scope for Package 2." This is a meaningful limitation: some MCP clients (notably newer Anthropic builds and some experimental Copilot connectors) expect DCR (RFC 7591) for auto-registration. Without DCR, an admin must manually register the OAuth Client first — exactly what `remote-mcp-clients.adoc` walks through. Today the docs implicitly require manual registration but never explain *why*, so a customer reading the page might wonder why their MCP client's "register with this MCP server" auto-flow isn't supported.

> *Plan:* Add a one-bullet entry to the "Limitations" section at the end of `remote-mcp-clients.adoc` (currently line 245–251) explicitly stating that AI Gateway does not support OAuth 2.0 Dynamic Client Registration (RFC 7591) today, and that all OAuth Clients must be registered manually through the UI or `rpk ai oauth-client` CLI. One sentence. Cross-link to the manual registration section. Optionally add a sentence in the architecture overview at the top stating "Each external MCP client must be registered up-front" so readers form the right model immediately.

### Items the briefing implies but does not require a doc change for

- Embedded IdP (Auth0 → Zitadel migration) — covered at line 149.
- OAuth2-spec generic / works with any compliant client — covered throughout, including the explicit Limitations subsection telling readers to confirm the latest menu paths for ChatGPT, Gemini, Cursor.
- No extra hacks like headers or local rpk proxy — implicit; the doc only shows the clean flow.
- `claude.ai` and Claude Desktop wire-up — covered thoroughly.
- Claude Code — covered in the new `claude-code.adoc` page (2026-05-24).
- Revocation flows (UI button + `rpk ai oauth-client revoke-tokens`) — covered, including the access-vs-refresh-token distinction.

### Open questions for eng

- Microsoft Copilot Studio: has anyone successfully wired ADP as a Copilot Studio connector end-to-end? If yes, capture the menu path and required redirect URIs. If no, leave it off the named-clients list.
- DCR: any plan for when DCR support lands (Package 3? Beyond?), so the Limitations bullet can reference a successor doc?
- ChatGPT / Gemini / Cursor: the doc has `// TODO: confirm and document the ChatGPT, Gemini, and Cursor menu paths once each integration ships.` at line 137. Are any of these now in a state where a real walk-through can replace the placeholder?

---

## Updated suggested PR breakdown

Five PRs total, in dependency order:

1. ✅ **LLM Proxy: transcript logging + Redpanda-maintained models clarification.** Shipped 2026-05-24. Touches `configure-provider.adoc`.
2. ✅ **Claude Code integration page.** Shipped 2026-05-24. Touches `modules/integrations/pages/claude-code.adoc` and adds an anchor to `connect-agent.adoc`.
3. ✅ **MCP Connection-tab naming flag.** Shipped 2026-05-24. Touches `modules/mcp/pages/create-server.adoc` with a TODO.
4. **Cost Tracking: shipped-vs-WIP reconciliation + spending API how-to.** Blocked on eng confirming which dashboard surfaces are live in Package 2. Touches `dashboard/overview.adoc` and adds a section to `budgets.adoc` (or a new `governance/pages/spending-api.adoc`).
5. **MCP OAuth Clients: Microsoft Copilot Studio + DCR limitation.** Touches `modules/integrations/pages/remote-mcp-clients.adoc`. The Copilot Studio addition is blocked on eng confirmation that the integration actually works; the DCR limitation can ship immediately.
6. **Microsoft Teams integration page (new).** Paused: see memory `project-adp-teams-integration-doc-gap`.

---

## 6. OAuth Provider / Token Vault

### What the briefing claims

1. OAuth Providers register external systems (Slack, Jira, GitHub, Salesforce, Workday, and similar) that AI Gateway can authenticate against.
2. OBO (on-behalf-of) semantics: each user of an MCP server can only do what they can do in the upstream system anyway, with their own permission level. No shared credentials.
3. Token Vault captures user tokens during OAuth login in the browser and refreshes them automatically up to the upstream's max expiry.
4. Two creation paths: fully customizable custom providers and pre-built "easy" template providers per popular upstream.
5. GitHub template as the canonical example, pre-filled with most fields.
6. MCP server configuration attaches one configured OAuth Provider; the auth lives on the provider once and is shared across many MCP servers (IT-configures-once, builders-attach-many).
7. Claude.ai-driven UX: expired connection presents a login URL, user authorizes, MCP becomes invokable seamlessly.

Briefing acknowledges one product-side polish item not yet shipped: "we will bake a custom experience per template oauth provider type in the future." Custom-flow UX is functional but rough today.

### What the docs cover today

This is the most thoroughly documented Package 2 feature. Coverage spans five pages plus the glossary:

- `modules/mcp/pages/oauth-providers.adoc`: ~285 lines. Prerequisites, the OAuth-Providers-vs-OAuth-Clients disambiguation, full permission set (`_create`, `_get`, `_update`, `_delete`, `_attach`), Browse / Register-in-UI / Register-from-CLI flows, the Category template picker with 10 categories, Browser Consent vs JWT Bearer grant types, client-secret-basic vs client-secret-post vs none auth methods, edit and rotate, delete with consequences, troubleshooting.
- `modules/mcp/pages/user-delegated-oauth.adoc`: ~80 lines. The OBO concept, the user consent flow with `authorize_url`, token vault storage under user identity, transparent refresh, `OAuthTokenExpired` failure mode, scope-upgrade flow, service-account-OAuth contrast.
- `modules/mcp/pages/github-oauth-tutorial.adoc`: ~360+ lines. End-to-end tutorial: create the upstream GitHub OAuth app, register the provider in ADP, test through *My Connections*, create the user-delegated GitHub MCP server, troubleshoot scope mismatches and token expiry.
- `modules/mcp/pages/create-server.adoc`: section *Configure authentication* documents the five auth modes, with User-delegated OAuth pulling from a registered OAuth Provider.
- `modules/ROOT/pages/index.adoc`: surfaces the "on-behalf-of authorization model" at the top of the product overview.
- `modules/reference/pages/glossary.adoc`: registers `OAuth provider`, `OAuth client`, `OAuth connection`, and `token vault` as terms with `glossterm:...` macros so they hyperlink consistently across the site.

### Gaps to fix

**Gap 1: `My Connections` is referenced but never has its own page.** Six pages mention *My Connections* as a sidebar entry where users see and revoke their OAuth connections (`user-delegated-oauth.adoc`, `github-oauth-tutorial.adoc`, `overview.adoc`, `create-server.adoc`, `test-tools.adoc`, `ai-gateway/overview.adoc`). A user trying to "find their connections to revoke one" reaches the page itself through trial, then has to piece together the behavior from references on three or four other pages.

> *Plan (optional):* Add `modules/mcp/pages/my-connections.adoc` describing: where the page lives in the sidebar, what each row represents (provider + scopes + last-used + status), how to revoke a single connection, how the consent flow re-runs after revocation, and the contrast with admin-side OAuth provider management. Cross-link from the six pages that already reference it. Low priority because the references work; nice to have for discoverability.

### Items the briefing implies but do not need a doc change

- OAuth Providers for popular upstreams: covered by the Category picker in `oauth-providers.adoc` (Identity, Source control, Productivity, Storage, Communication, CRM, Data warehouse, Monitoring, Security, Business categories, with named example providers in each).
- OBO semantics: surfaced in `oauth-providers.adoc` opening prose, dedicated treatment in `user-delegated-oauth.adoc`, and a top-level product-overview callout in `ROOT/pages/index.adoc`.
- Token Vault as a named concept: glossary term plus consistent inline use across pages.
- Custom provider creation: full form reference in `oauth-providers.adoc`.
- GitHub template: full tutorial.
- IT-configures-once, builders-attach-many: covered by the separate `_create` / `_update` / `_delete` vs `_attach` permission split and the NOTE at `oauth-providers.adoc` line 59. The opening prose also says "any MCP server that talks to that upstream can attach to it instead of carrying its own credentials."
- Token refresh up to max expiry: covered in `user-delegated-oauth.adoc` (transparent refresh + re-consent on refresh failure).
- Claude.ai expired-connection re-login flow: covered in `remote-mcp-clients.adoc` two-step OAuth flow section.

### Open questions for engineering

None blocking. Two soft questions:

- Is there a planned `My Connections` redesign that would change what the page shows (for example, surfacing token TTL, last upstream call timestamp, or per-MCP-server attribution)? If yes, defer the dedicated page until the redesign lands.
- The briefing's "we will bake a custom experience per template oauth provider type" implies the Category picker today pre-fills endpoints but does not deeply customize the form fields. Confirm whether the current per-template differences are limited to endpoint pre-fill, or whether scope vocabularies and additional fields already vary per template; the docs should reflect what actually differs.
