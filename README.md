# @pipeworx/mn-pfas-products

PFAS in consumer and commercial products as reported by manufacturers in Minnesota: which company reported which PFAS chemical, in which product component, for what function and at what concentration band.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2020+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `pfas_find_companies(query)` -- manufacturers with published reports, by name; returns company ids.
- `pfas_find_chemicals(query)` -- PFAS chemicals on the Minnesota reporting list, by name or CAS number.
- `pfas_search_disclosures(company?, chemical?, function?, brand?, component?, product_code?, concentration_range?, period?, limit?, offset?)` -- matching disclosure rows plus the total match count.
- `pfas_summary(group_by?, ...same filters)` -- disclosure counts grouped by chemical, function, manufacturer, concentration band or reporting period.

## Auth

Keyless.

## Data sources

- Minnesota Pollution Control Agency, PFAS in products reports (PRISM reporting system, operated for the agency by the Interstate Chemicals Clearinghouse). Reports appear only after agency staff review, so the published set is a subset of what manufacturers have submitted; trade-secret data is not shown. Every response carries `data_as_of`, the latest publication timestamp.

## Design and budget

Live proxy: each call goes to the publisher's public search and returns its answer. No store, no refresh job, no Cloudflare binding, no recurring cost. Budget: one-time $0, steady state $0 (D1/KV/R2/DO writes: none).

## Notes

- Manufacturer contact persons, their titles, emails, phones and mailing addresses are never read or served. Manufacturers are companies.
- Company and chemical filters are resolved against the small name tables first (fast), then applied by id. Brand, component and product-code filters scan the full disclosure table and take 3-8 seconds.
- Each disclosure row is one product component x one PFAS chemical.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "mn-pfas-products": {
      "url": "https://gateway.pipeworx.io/mn-pfas-products/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/mn-pfas-products/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 2020+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/pfas_find_companies \
  -H 'Content-Type: application/json' \
  -d '{"query":"Quaker"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/pfas_find_companies`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-mn-pfas-products` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/mn-pfas-products/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "mn-pfas-products": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-mn-pfas-products"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Mn Pfas Products data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
