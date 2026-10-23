# AdsCove MCP Server

AdsCove is an approval-gated advertising-operations server that connects AI agents to Shopify, Meta Ads, Google Ads, and TikTok Ads.

## Endpoint

- URL: `https://adscove.com/api/mcp`
- Transport: Streamable HTTP. Clients should send `Accept: application/json, text/event-stream`. JSON-only clients are also supported.
- Authentication:
  - OAuth, for clients that support remote-MCP OAuth.
  - API key, for other clients: `Authorization: Bearer sk_live_...`. Generate a key at https://adscove.com/keys.
- Catalog modes:
  - Default (progressive core): `tools/list` stays small (connections, reporting, capabilities, `discover_adscove_tools`, approvals). Call `discover_adscove_tools` to find platform tool names, then `run_adscove_tool`.
  - `https://adscove.com/api/mcp?tools=expanded`: the previous fat compact list.
  - `https://adscove.com/api/mcp?tools=full`: every schema.

Setup: connect your platforms at https://adscove.com/connections, then add the endpoint as a remote MCP server in your AI client.

## What it does

Platforms:

- Shopify (optional)
- Meta Ads
- Google Ads
- TikTok Ads

The launch-available registry has 329 tools (182 read, 147 write), as of the 2026-10-20 source review.

Capabilities:

- Reporting and diagnosis
  - Daily business report across Shopify and ad accounts.
  - Cross-platform campaign performance in one schema.
  - Attribution-gap analysis, wasted-spend and search-term analysis, anomaly explanations, creative fatigue, and tracking-URL audits.
  - One-call business insights for "how are my accounts doing" questions.
- Creatives
  - Upload images and videos to the Meta, Google, and TikTok ad libraries.
  - Inspect existing ad creatives and assets.
  - Replace Shopify product images.
- Launch and optimize (writes, approval-gated)
  - Create paused Meta campaigns, ad sets, and ads, and paused Google Search, Performance Max, Demand Gen, and Display drafts.
  - Create disabled TikTok campaigns and ad groups.
  - Change status, budgets, bids, targeting, and negative keywords.
  - Create and manage native Meta and TikTok automated rules.
  - Edit Shopify products, pages, collections, discounts, and inventory.

Blended ROAS in AdsCove is Shopify GMV divided by combined Meta, Google, and TikTok spend. Because it uses total store GMV, it is closer to MER than to campaign-level attribution ROAS. See https://adscove.com/guides/shopify-meta-google-roas for the formulas and limits.

## Safety model

- Every real write needs explicit approval. A write returns a pending action with an `execution_id`. The agent runs it with `approve_action` only after the user consents in the conversation, or drops it with `reject_action`.
- New ads start paused. Meta and Google creation tools create PAUSED resources. TikTok creation tools use DISABLE. Activating a resource is a separate approved action.
- Dry runs come first where they apply. Shopify media and file uploads default to a dry run. `google_launch_campaign` lists exactly what it will enable before approval. Automation rules are created disabled, pass static checks and a Dry Run, and are activated separately.
- Errors include `error.code`, `category`, `retryable`, `user_action`, and `operator_action`. Clients should branch on `error.code` first.
- Unconnected platforms return `not_connected`. Tools without a live adapter return `capability_not_implemented`, never a sample success.
- Hiding an account from lists does not pause ads or disable rules or schedules.
- The security documentation covers how reads are separated from writes, how platform tokens are encrypted, how duplicate writes are prevented, and how audit history is recorded: https://adscove.com/docs/security
- AdsCove is not an autonomous agent. It does not change ads or spend budget without approval.
- AdsCove is in beta and does not provide an uptime SLA. Keep the native platform safeguards enabled.

## Quick start

Connect your platforms at https://adscove.com/connections first. Then add the endpoint in your client. To check the connection, ask the agent to call `list_connections`.

### Claude (web, desktop)

1. Open Settings, then Connectors, then Add custom connector.
2. Paste `https://adscove.com/api/mcp` as the URL.
3. Sign in with OAuth when prompted. Setup guide: https://adscove.com/clients/claude

### Claude Code

```sh
claude mcp add --transport http adscove https://adscove.com/api/mcp
```

Setup guide: https://adscove.com/clients/claude-code

### Cursor

Add this to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "adscove": {
      "url": "https://adscove.com/api/mcp"
    }
  }
}
```

Setup guide: https://adscove.com/clients/cursor

### Codex

Add this to `~/.codex/config.toml`:

```toml
[mcp_servers.adscove]
url = "https://adscove.com/api/mcp"
```

Setup guide: https://adscove.com/clients/codex

### ChatGPT

Custom MCP apps in ChatGPT are currently web-only. Add AdsCove as a custom MCP app on the ChatGPT web app, using `https://adscove.com/api/mcp`. Setup guide: https://adscove.com/clients/chatgpt

### Gemini CLI

Add this to your Gemini CLI `settings.json`:

```json
{
  "mcpServers": {
    "adscove": {
      "httpUrl": "https://adscove.com/api/mcp"
    }
  }
}
```

Setup guide: https://adscove.com/clients/gemini

The `examples/` folder contains these snippets as separate files.

## Example prompts

- "Compare Meta-reported ROAS with Shopify revenue for the last 7 days."
- "How are my accounts doing, and what should I optimize first?"
- "Which Google search terms are wasting spend? Suggest negative keywords, but don't apply them yet."
- "Check whether any Meta ads are disapproved or pending review, and tell me why."
- "Prepare a paused Meta test campaign for this product and show me the pending changes. Wait for my approval before running them."

## Links

- Website: https://adscove.com
- Product introduction: https://adscove.com/docs
- Quickstart: https://adscove.com/docs/quickstart
- Security and approval model: https://adscove.com/docs/security
- Pricing and account binding limits: https://adscove.com/docs/pricing
- Live tool reference: https://adscove.com/docs/tools
- REST API: https://adscove.com/docs/api
- Full page index: https://adscove.com/llms-full.txt
- Guides: https://adscove.com/guides
  - Meta Ads MCP setup: https://adscove.com/guides/meta-ads-mcp-setup
  - Google Ads MCP setup: https://adscove.com/guides/google-ads-mcp-setup
  - Shopify, Meta, and Google ROAS: https://adscove.com/guides/shopify-meta-google-roas
  - Meta ROAS vs Shopify: https://adscove.com/guides/meta-roas-vs-shopify
  - Approval-gated ad changes: https://adscove.com/guides/approval-gated-ad-changes
  - AI ad operations safety: https://adscove.com/guides/ai-ad-operations-safety
  - Claude Desktop setup: https://adscove.com/guides/claude-desktop-ecommerce-mcp
  - Cursor and Claude install: https://adscove.com/guides/cursor-claude-ecommerce-mcp-install
- Pricing: https://adscove.com/pricing

## License

The server is a hosted service operated by AdsCove. This repository contains documentation and configuration examples only (MIT).
