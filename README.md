# @pipeworx/census-geocoder

The US Census Bureau's own geocoder: turns a US street address into a
TIGER/Line coordinate and the full Census geography stack for that point —
state, county, census tract, census block, place, congressional district,
school district and CBSA, each with its GEOID.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `census_geocode_address(address | street/city/state/zip, benchmark?, vintage?)` —
  forward geocode one address; returns coordinates plus the geography stack.
- `census_reverse_geocode(latitude, longitude, benchmark?, vintage?)` — every
  Census geography containing a coordinate.
- `census_geocode_batch(addresses[], benchmark?, vintage?)` — up to 100
  addresses in one call; returns match status, standardized address,
  coordinates and state/county/tract/block FIPS per row, plus the assembled
  tract and block GEOIDs.

## Auth

Keyless.

## Data sources

- <https://geocoding.geo.census.gov/geocoder/geographies/onelineaddress> — one-line forward geocode.
- <https://geocoding.geo.census.gov/geocoder/geographies/address> — structured forward geocode.
- <https://geocoding.geo.census.gov/geocoder/geographies/coordinates> — reverse geocode.
- <https://geocoding.geo.census.gov/geocoder/geographies/addressbatch> — multipart CSV batch.

Related packs: `census-tigerweb` describes the geographies themselves (list
every tract in a county); this pack answers "which tract is this address in".
The `geocodio` pack is a commercial geocoder and is not a substitute when the
answer has to be a Census GEOID.

## Traps

- `benchmark` and `vintage` must agree. `Public_AR_Current` pairs with
  `Current_Current`; a mismatched pair is a 400 with an unhelpful body.
- The `/geographies/*` endpoints return geography; the `/locations/*` ones
  return only coordinates. This pack always uses `/geographies`.
- The batch endpoint is a **multipart CSV POST that answers in CSV**, and it
  geocodes every line it is handed. Send a header row and you get a `No_Match`
  result for the literal word "street". This pack never sends one.
- Batch columns are positional (`id,street,city,state,zip`) so an unquoted
  comma inside a street field shifts every later column. Each cell is quoted.
- Congressional and state-legislative layer names carry the session number, so
  they are matched by pattern rather than by a name that expires.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "census-geocoder": {
      "url": "https://gateway.pipeworx.io/census-geocoder/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/census-geocoder/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/census_geocode_address \
  -H 'Content-Type: application/json' \
  -d '{"address":"1600 Pennsylvania Ave NW, Washington, DC 20500"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/census_geocode_address`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-census-geocoder` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/census-geocoder/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "census-geocoder": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-census-geocoder"]
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
ask_pipeworx({ question: "your question about Census Geocoder data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
