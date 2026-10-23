# chargebee-mcp

Chargebee MCP Service — a stateless HTTP MCP server wrapping the [Chargebee REST API v2](https://apidocs.chargebee.com/docs/api/) for account/customer management use cases (company records, personnel/contacts, subscription lookups, and financial reporting lookups).

**Tech stack:** Python 3.12 + uv + FastMCP (Starlette/Uvicorn)

<div align="center">

## MSPbots Connector

**The MCP Gateway built for MSPs: visible, controlled, and your data stays where it belongs.**

[**Get started at connect.mspbots.ai →**](https://connect.mspbots.ai)

</div>

---

MSPbots Connector gives your AI clients one MCP URL to reach your entire MSP stack (RMM, PSA, documentation, security, finance), and gives you control over who can call which tool and a record of everything that was called.

**Hosted service: [connect.mspbots.ai](https://connect.mspbots.ai)** · Free to use.

### Why Connector

![MSPbots Connector](https://stgoss.mspbots.ai/files/mcp/mspbots-mcp.png)

| | |
|---|---|
| **Visible** | Every tool call is logged: who, which vendor, when. Usage dashboard, CSV export, SIEM streaming. |
| **Controlled** | Role-based, per-tool allowlist. Deny by default. |
| **Protected** | Credentials encrypted per record. AI clients never see your vendor passwords or tokens. |
| **MSP-native** | Built for the MSP stack only: RMM, PSA, documentation, security, finance. |

### Quick start

1. **Sign up** at [connect.mspbots.ai](https://connect.mspbots.ai).
2. **Connect your vendors.** Pick from 150+ connectors and authorize each one on the vendor's own page, or bring your own API keys. No extra vendor accounts needed.
3. **Set access.** Create roles and decide which tools each role may call. Nothing is allowed until you allow it.
4. **Add your MCP URL to your AI client.** Copy the URL from your Connector dashboard into Claude, ChatGPT, Cursor, VS Code, Gemini, Codex, or any MCP-capable client.
5. **Ask your AI to do real work**, and watch every call appear in the audit log.

### Features

**Unified gateway**
- One MCP URL for all your AI clients
- 150+ connectors across the MSP stack
- Custom MCP servers supported, free
- Authorization happens on each vendor's own page; the AI never sees passwords or tokens
- Bring your own credentials: use the vendor API keys you already have
- Multiple accounts per app; connect once, reuse across AI clients
- Tool search: finds the right action for the task
- One sign-in; vendor credentials are injected behind the scenes and OAuth tokens refresh automatically

**Visibility**
- Audit log for every tool call (who, which vendor, when)
- Usage dashboard: call volume, top vendors, error rate, per-user activity
- Export audit logs to CSV, or stream to Loki, Graylog, or LogScale (SIEM)
- Optional recording of tool arguments and result summaries (off by default, enabled per organization)

**Control**
- Per-role, per-tool allowlist, deny by default
- owner / admin / member roles
- SCIM provisioning with Entra ID, Okta, JumpCloud, and Google Workspace

**Security**
- Credentials encrypted one record at a time (AES-256-GCM, per-record key)
- AI clients authenticate with OAuth 2.1 + PKCE
- Vendor responses are scanned for prompt-injection patterns and returned as structured data. This is one layer of defense, not a guarantee against every attack.
- Static egress IP for customer firewall and API allowlisting

### Pricing

**Free.**

Links: [connect.mspbots.ai](https://connect.mspbots.ai) · [mspbots.ai](https://mspbots.ai)

---

## When would an agent use this

Chargebee is MSPbots' own subscription billing/revenue platform. An agent should reach for this MCP for requests like:

- "Look up this customer's billing account / create a new customer record" → `chargebee_retrieve_customer` / `chargebee_create_customer`
- "Update this company's billing email or address" → `chargebee_update_customer`
- "Who are the contacts under this account?" → `chargebee_list_customer_contacts`
- "What subscription plan is this customer on, is it active?" → `chargebee_list_subscriptions` (filter by `customer_id`) or `chargebee_retrieve_subscription`
- "Cancel this customer's subscription" → `chargebee_cancel_subscription`
- "Pull this customer's recent invoices / did this transaction go through?" → `chargebee_list_invoices` (what was billed) / `chargebee_list_transactions` (whether payment settled)

## Scope

Chargebee's official [MCP Server](https://www.chargebee.com/docs/billing/2.0/ai-in-chargebee/chargebee-mcp) offering (the "Data Lookup MCP Server") was evaluated first and found unsuitable as a replacement: it is read-only, covers roughly a dozen resource categories, and is missing Coupons, the Item/Item Price/Item Family product-catalog resources, and Upcoming Invoice Estimates entirely — none of which can be added on top of it. This service instead wraps the full Chargebee REST API directly.

Out of Chargebee's ~438 REST API operations across 77 resources, this service started at 30 tools selected for account/company/personnel/report management use cases, then was trimmed down to the **10 core tools** below (user-confirmed reduction — dropped delete/merge/payment-role/hierarchy on customers, all contact write ops, subscription create/update/pause/resume/reactivate/term-end/scheduled-changes, all payment-source tools, and invoice-retrieve/credit-notes from reports):

| Category | Count | Resources |
|---|---|---|
| Company (Customer) | 4 | create, retrieve, update, list |
| Personnel (Customer Contacts) | 1 | list contacts under a customer |
| Account (Subscription) | 3 | list, retrieve, cancel |
| Report (Invoices/Transactions) | 2 | list invoices, list transactions |

## Quick Start

```powershell
# Install dependencies
cd D:\claude\project\chargebee-mcp
uv sync

# Run in stdio mode (for Claude Desktop)
$env:CHARGEBEE_SITE="your-site"
$env:CHARGEBEE_API_KEY="your_api_key"
uv run chargebee-mcp
```

## Configuration

Copy `.env.example` to `.env` and fill in your values:

| Variable | Default | Description |
|----------|---------|--------------|
| `CHARGEBEE_SITE` | — | Chargebee site name (the subdomain in `https://{site}.chargebee.com`) |
| `CHARGEBEE_API_KEY` | — | Chargebee API key |
| `AUTH_MODE` | `gateway` | `gateway` = site + API key per-request via headers (SOP-compliant); `env` = shared credentials from env vars (local dev only) |
| `MCP_TRANSPORT` | `stdio` | `stdio` (Claude Desktop) or `http` (gateway) |
| `MCP_HTTP_PORT` | `8080` | HTTP server port |

Get your API key: Chargebee Billing → Settings → Configure Chargebee → API Keys and Webhooks → API Keys tab.

## HEADER Authentication Parameters

In Gateway mode, every request must carry the following two HTTP Headers:

| Header | Type | Required | Default | Allowed values | Description | Example |
|---|---|---|---|---|---|---|
| `X-Chargebee-Site` | string | Yes | None | None | Chargebee site name (i.e. the subdomain part of `https://{site}.chargebee.com`) | `acme-test` |
| `X-Chargebee-Api-Key` | string | Yes | None | None | Chargebee API Key (generated on the Settings → Configure Chargebee → API Keys and Webhooks page); this service uses it as the HTTP Basic Auth username with the password left empty | `test_XXXXXXXXXXXXXXXXXXXXXXXXXXXXX` |

## Claude Desktop Setup

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "chargebee": {
      "command": "uv",
      "args": ["run", "--directory", "D:/claude/project/chargebee-mcp", "chargebee-mcp"],
      "env": {
        "CHARGEBEE_SITE": "your-site",
        "CHARGEBEE_API_KEY": "your_api_key"
      }
    }
  }
}
```

## Transport Modes

### stdio (Claude Desktop / CLI)
```powershell
$env:CHARGEBEE_SITE="your-site"
$env:CHARGEBEE_API_KEY="your_api_key"
uv run chargebee-mcp
```

### HTTP — single-tenant
```powershell
$env:CHARGEBEE_SITE="your-site"
$env:CHARGEBEE_API_KEY="your_api_key"
$env:MCP_TRANSPORT="http"
$env:AUTH_MODE="env"
uv run chargebee-mcp
```

### HTTP — gateway / multi-tenant
```powershell
$env:MCP_TRANSPORT="http"
$env:AUTH_MODE="gateway"
uv run chargebee-mcp
# Each request must include: X-Chargebee-Site and X-Chargebee-Api-Key headers
```

## Available Tools (10)

### Company (Customer) — 4

| Tool | Description | Parameters |
|---|---|---|
| `chargebee_list_customers` | List customers (companies) | `limit`, `offset`, `include_deleted`, `filters` (dict of Chargebee compound filter keys) |
| `chargebee_create_customer` | Create a customer (company) | `first_name`, `last_name`, `email`, `company`, `phone`, `id`, `auto_collection`, `taxability`, `locale`, `meta_data`, `billing_address` |
| `chargebee_retrieve_customer` | Retrieve a customer by ID | `customer_id` |
| `chargebee_update_customer` | Update a customer | `customer_id`, `first_name`, `last_name`, `email`, `company`, `phone`, `auto_collection`, `taxability`, `locale`, `invoice_notes`, `meta_data` |

### Personnel (Customer Contacts) — 1

| Tool | Description | Parameters |
|---|---|---|
| `chargebee_list_customer_contacts` | List contacts under a customer | `customer_id`, `limit`, `offset` |

### Account (Subscription) — 3

| Tool | Description | Parameters |
|---|---|---|
| `chargebee_list_subscriptions` | List subscriptions | `limit`, `offset`, `include_deleted`, `filters` |
| `chargebee_retrieve_subscription` | Retrieve a subscription by ID | `subscription_id` |
| `chargebee_cancel_subscription` | Cancel a subscription | `subscription_id`, `cancel_option`, `end_of_term`, `cancel_at`, `cancel_reason_code`, `credit_option_for_current_term_charges`, `unbilled_charges_option` |

### Report (Invoices / Transactions) — 2

| Tool | Description | Parameters |
|---|---|---|
| `chargebee_list_invoices` | List invoices | `limit`, `offset`, `include_deleted`, `filters` |
| `chargebee_list_transactions` | List payment/refund transactions | `limit`, `offset`, `include_deleted`, `filters` |

### Filter parameters

Chargebee's list endpoints use compound filter query keys with an operator suffix, e.g. `email[is]=a@b.com`, `created_at[after]=1700000000`, `status[in]=["active","paused"]`. Rather than expanding every field × operator combination into separate function parameters, list tools accept an optional `filters: dict[str, str]` argument — pass the literal Chargebee query key (including the `[operator]` suffix) as the dict key. Supported fields per resource are documented in each tool's docstring; supported operators are `[is]`, `[is_not]`, `[starts_with]`, `[in]`, `[not_in]`, `[between]`, `[after]`, `[before]`, `[on]`, `[none]`, `[is_present]` (availability varies by field type — see [Chargebee's filter documentation](https://apidocs.chargebee.com/docs/api/customers?prod_cat_ver=2#list_customers)).

### Request encoding

Chargebee's REST API uses `application/x-www-form-urlencoded` request bodies, not JSON. Nested object parameters (`billing_address`, `meta_data`) are accepted as plain Python `dict` and flattened server-side into Chargebee's bracket-notation form fields (`billing_address[line1]=...`) — see `_flatten_form()` in `api_client.py`. `_flatten_form()` also supports flattening lists and "array of hashes" fields (Chargebee's `field[subfield][index]=value` convention), but none of the current 10 tools take such a parameter — that code path is currently unexercised.

## Known Gaps

- **Verified against a live Chargebee account.** Using a real production API key + site: `chargebee_list_customers`, `chargebee_retrieve_customer`, `chargebee_list_customer_contacts`, `chargebee_list_subscriptions`, `chargebee_list_invoices`, and `chargebee_list_transactions` all returned real data (200) through the running service. Gateway 401 gating was re-confirmed with a fresh MCP session + an invalid API key (correctly rejected by Chargebee with `api_authentication_invalid_key`).
- **Write operations (`chargebee_create_customer`, `chargebee_update_customer`, `chargebee_cancel_subscription`) have not been exercised end-to-end** — only verified structurally (schema, request construction). This was a deliberate choice during self-test: the only available credentials are for MSPbots' own live production Chargebee site, and mutating real billing/subscription data to self-test was avoided. If write-path verification is needed, test against a Chargebee **test site** (not a live/production API key).
- Nested fields (`billing_address`, `meta_data`) are accepted as generic `dict` rather than fully-typed sub-schemas — callers must know Chargebee's field names for these substructures (documented in each tool's docstring where practical, otherwise see the [Chargebee API reference](https://apidocs.chargebee.com/docs/api/)).
- List-endpoint filters are not expanded into named parameters (see "Filter parameters" above) — this keeps the tool count/signature size manageable but pushes filter-key correctness onto the caller.
- Scope is limited to the 10 operations above (company/personnel/account/report management, user-trimmed down from an initial 30). Everything else — Coupons, Items/Item Prices/Item Families, Estimates, Orders, Payment Sources, Credit Notes, subscription create/update/pause/resume/reactivate, customer delete/merge/hierarchy, contact write ops, etc. — is out of scope per user-confirmed selection.

## API Reference

- [Chargebee API Documentation](https://apidocs.chargebee.com/docs/api/)
- [Chargebee OpenAPI Specification](https://github.com/chargebee/openapi) (`chargebee_api_v2_pc_v2_spec.json` — API v2 + Product Catalog v2, used as the source of truth for this service)
- [Chargebee Official MCP Servers](https://www.chargebee.com/docs/billing/2.0/ai-in-chargebee/chargebee-mcp) (evaluated and found insufficient — see Scope above)
