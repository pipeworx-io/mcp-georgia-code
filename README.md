# @pipeworx/georgia-code

Official Code of Georgia Annotated (O.C.G.A.) — Georgia state statutes by citation
and by topic search, with historical versions and legislative/amendment history.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1879+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `ga_code_section(citation)` — the current text of a Code section by citation
  (any natural form: `"16-5-1"`, `"O.C.G.A. § 44-7-34"`, `"Ga. Code Ann. §
  16-5-1"`, `"Title 16, Chapter 5, Section 1"`). Returns status
  (active/repealed/reserved), the legislative history line (the Acts that
  enacted and amended it), editor's notes and cross references, and every
  stored historical version when a section has more than one — e.g. a
  delayed effective date.
- `ga_code_search(query, limit?, headings_only?)` — full-text topic search
  over the Code's body text and catchlines. Returns ranked citations with a
  highlighted snippet; pass the citation to `ga_code_section` for the full
  text.

## Auth

Keyless. No signup, no key, on either side of the call.

## Data sources

- <https://api.georgiacommons.org/api/code/sections/{citation}> — section lookup.
- <https://api.georgiacommons.org/api/code/search?q=> — full-text search (`in=headings`
  narrows to headings/catchlines only).
- Documented at <https://georgiacommons.org/api> and
  <https://georgiacommons.org/api/endpoints>.

**Why this isn't a Lexis call.** Georgia's own publishing arrangement — the Code
Revision Commission, for the General Assembly (legis.ga.gov) — contracts the
OCGA's *online* text out to LexisNexis (`advance.lexis.com`), which is a
login-shaped surface this pack does not touch. The statutory *text* itself
carries no copyright: *Georgia v. Public.Resource.Org, Inc.*, 590 U.S. 255
(2020) held the OCGA (text and annotations) is an edict of government. Georgia
Commons — an independent project of Georgia Civic Data, explicitly "not
affiliated with, operated by, or endorsed by the State of Georgia" — publishes
that public-domain text as a CC0-1.0, keyless, no-signup JSON API, current
through the 2025 Regular Session, extracted from the state's own officially
released volumes (each record's `source_link` points back to the original
legis.ga.gov PDF). This pack calls that API live, per request.

**Four capabilities, one response shape.** `GET /api/code/sections/{citation}`
carries all four at once:
- citation lookup — the top-level fields (`catchline`, `status`, `heading_path`).
- topic/full-text search — the separate `/search` endpoint.
- historical version — the `versions[]` array; `version_count` is 1 for most
  sections and >1 for ones with a delayed or superseded effective date.
- amendments/history — the `history` string (Acts that enacted/amended the
  section) and `notes` (editor's notes, cross references, law review articles).

**Rate limits** (documented, no key): 120 requests/min, 2,000/hr, shared across
Georgia Commons' bills/opinions/code collections. One request per tool call
here; a 429 carries `Retry-After`.

**Citation parsing is upstream's job, not ours.** The API itself accepts
`"16-5-1"`, `"O.C.G.A. § 16-5-1"`, `"Ga. Code Ann. § 16-5-1"`, `"Code Section
16-5-1"`, and `"Title 16, Chapter 5, Section 1"` — this pack passes the caller's
string through URL-encoded rather than re-parsing it, and lets the upstream's
own 404 (which names the nearest sections and the accepted forms) do the error
handling.

**Scope note.** The upstream API also serves General Assembly bills, Supreme
Court of Georgia opinions, and both the Georgia and U.S. constitutions under
the same `/api/{collection}` grammar — out of scope for this pack, which is
statutes only.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "georgia-code": {
      "url": "https://gateway.pipeworx.io/georgia-code/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/georgia-code/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1879+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/ga_code_section \
  -H 'Content-Type: application/json' \
  -d '{"citation":"16-5-1"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/ga_code_section`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "georgia-code": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-georgia-code"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-georgia-code
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Georgia Code data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
