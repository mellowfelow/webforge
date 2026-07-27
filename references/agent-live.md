# WebForge — Live agent layer reference (v8, Vercel only)

Sections A–N of the skill are the **declaration layer** (static files that describe capabilities — both targets ship them). This is the **capability layer**: endpoints agents actually call. Requires API routes → **Vercel only**.

## Rule 10 — the target decides what may be claimed

`scripts/gen-agent-files.mjs` reads `SITE.target` and emits **different `.well-known` content**:

| File | `target: 'vercel'` | `target: 'static'` (Cloudflare) |
|---|---|---|
| `mcp/server-card.json` | `transport: {type:'streamable-http', endpoint:'https://[DOMAIN]/api/mcp'}` + full `tools[]` | `transport: {type:'none'}`, **no `tools`**, note: `human_ordering_only` |
| `api-catalog` | real JSON endpoints, `service-desc` links | HTML resources only |
| `acp.json` / `ucp` | live endpoint URLs | descriptive; `ordering: 'human-assisted-chat'` |
| `agent-skills/index.json` | skills → API URLs | skills → page URLs |

A static site advertising an MCP server it cannot host is a lie. **Crosscheck B2 fails it.** Never generate the Vercel variant for a Cloudflare build — this is structural, not a reminder.

---

## `/api/mcp` — MCP server (Streamable HTTP)

**Transport: Streamable HTTP** (MCP spec 2025-03-26, retained in 2025-11-25). It replaced HTTP+SSE. **Never implement HTTP+SSE** — two endpoints (`/sse` + `/messages`) is the deprecated design.

- **One endpoint path** — `/api/mcp` — supporting **POST** (client JSON-RPC) and **GET** (optional server SSE stream).
- Response mode follows the client's `Accept`: `application/json` → one JSON body; `text/event-stream` → SSE. **A product catalog is always fast: return JSON. Do not implement streaming without a reason.**
- **Stateless.** No session store. `Mcp-Session-Id` unnecessary for a read-only catalog.
- Handle `initialize`, `tools/list`, `tools/call`. Proper JSON-RPC errors: `-32700` parse, `-32600` invalid request, `-32601` method not found, `-32602` invalid params.
- `OPTIONS` → 204 with CORS headers.

```js
// src/app/api/mcp/route.js
const PROTOCOL_VERSION = '2025-11-25'
const CORS = {
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Methods': 'POST, GET, OPTIONS',
  'Access-Control-Allow-Headers': 'Content-Type, Accept, Mcp-Session-Id',
}
export async function OPTIONS() { return new Response(null, { status: 204, headers: CORS }) }

export async function POST(req) {
  const msg = await req.json()
  const reply = (result) => Response.json({ jsonrpc: '2.0', id: msg.id, result }, { headers: CORS })
  const fail  = (code, message) => Response.json({ jsonrpc: '2.0', id: msg.id ?? null, error: { code, message } }, { headers: CORS })

  switch (msg.method) {
    case 'initialize':
      return reply({
        protocolVersion: PROTOCOL_VERSION,
        capabilities: { tools: {} },
        serverInfo: { name: SITE.brand, version: '1.0.0' },
      })
    case 'tools/list':
      return reply({ tools: TOOLS })            // TOOLS = the single source, also read by gen-agent-files
    case 'tools/call': {
      const tool = TOOLS.find(t => t.name === msg.params?.name)
      if (!tool) return fail(-32602, `Unknown tool: ${msg.params?.name}`)
      const out = await run(tool.name, msg.params.arguments ?? {})
      return reply({ content: [{ type: 'text', text: JSON.stringify(out) }] })
    }
    default:
      return fail(-32601, `Unknown method: ${msg.method}`)
  }
}
```

**`TOOLS` is defined once and imported by BOTH the route and `gen-agent-files.mjs`.** That is what makes `server-card.json` structurally incapable of drifting from `tools/list` (crosscheck B8).

### Standard tools

| Tool | Input | Returns |
|---|---|---|
| `search_products` | `query`, `category?`, `max_price?` | slug, name, price, currency, category, short, url |
| `get_product` | `slug` | full product + images + availability + url |
| `list_categories` | — | slug, name, description, count |
| `get_policies` | — | shipping, returns, payment methods, min order, free-ship threshold |
| `get_wholesale_info` | — | tiers + how to enquire *(only if wholesale = Yes)* |
| `create_order_draft` | `items[]`, `notes?` | prefilled order URL (chat deep-link or `/order/`) + totals — **no payment** |

Adapt names to the vertical. **Never invent a tool for a capability the business doesn't have.**

### Hard rules

- **Read-only plus draft. A human always completes the order.** No payment capture, no card data, no state mutation. (Skill Rule 13.)
- **Tool descriptions carry the site's compliance language.** Banned words are banned here too — the crosscheck scans tool descriptions (B7). This is the easiest place to leak a forbidden term, because it doesn't look like page copy.
- All data derives from `src/config/site.js`. The MCP server never holds its own catalog copy.
- Prices as numbers + explicit `currency`. Every item carries an absolute `url` from `SITE.domain`.
- Descriptions are written **for an agent deciding whether to call**: say what it returns and when to use it, not how it works.

---

## JSON API

| Route | Returns |
|---|---|
| `GET /api/products` | catalog; `?category=` `?q=` `?limit=` |
| `GET /api/products/[slug]` | one product; 404 if unknown |
| `GET /api/categories` | categories + counts |
| `GET /api/search?q=` | products + posts (same index as `/search/`) |

All: `Content-Type: application/json`, `Access-Control-Allow-Origin: *`, `Cache-Control: public, max-age=300`.

Shape products identically everywhere — MCP tools, JSON API and `/search/` all serialize through one `toPublicProduct(p)` helper. Three subtly different product shapes is how agents get confused.

## Live ACP / UCP

`.well-known/acp.json` → `/api/acp/catalog`; `.well-known/ucp` → `/api/ucp/services`. Both return live JSON. Ordering stays `human-assisted` in both protocols, consistent with `create_order_draft`.

## Crosscheck (B8 + V5)

On a Vercel build, boot locally and assert:
- `POST /api/mcp` with `Accept: application/json` answers `initialize` **and** `tools/list`
- every tool in `server-card.json` appears in the live `tools/list` — **and vice versa** (drift both ways = FAIL)
- every URL in `api-catalog` returns 200
- `/api/products`, `/api/search` return valid JSON with correct content types
- no endpoint mutates state; none accepts payment fields
- no banned compliance term appears in any tool description or API response

## Why this matters

A static site's agent files are a brochure about an API. `server-card.json` advertising tools that don't exist is the agent-era equivalent of keyword stuffing — it will be discovered and discounted. The live layer is the difference between **declaring** agent-readiness and **having** it.

For Cloudflare targets: the declaration layer alone is honest and correct, and caps around 79 on isitagentready's free plan. That is the plan's ceiling, not a defect — **do not add Pages Functions purely to chase points.**
