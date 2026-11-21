# @pipeworx/switzerland-law

Swiss federal legislation from Fedlex, the Confederation's publication platform
(Federal Chancellery): search acts by title, abbreviation or Classified
Compilation (SR) number, then read an act's record and its consolidated text,
whole or one article at a time. German, French, Italian, and English where an
English version exists.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2020+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `ch_law_search(query?, abbreviation?, sr_number?, lang?, in_force_only?, page?)`
  — find federal acts by title words in the chosen language (for example
  `Datenschutz` in `de`, `protection des donnees` in `fr`), by short title
  (`DSG`, `ZGB`) or by SR number (`235.1`). Returns the SR number, the Official
  Compilation (AS) reference, short title, adoption and entry-into-force dates,
  status and a link. Only acts in force today unless `in_force_only=false`.
  25 per page; `next_page` says whether to continue.
- `ch_law_get(id, article?, lang?, max_chars?)` — one act by SR number
  (`235.1`) or AS reference (`2022/491`): title, short title, SR and AS
  references, dates, in-force status, the date of the consolidated version
  returned (the one applicable today) and the text. `article` returns one
  article (`6`, `66a`, `7bis`); otherwise the text is capped at `max_chars`
  with a list of chapters.

## Auth

Keyless.

## Data sources

- Switzerland, Federal Chancellery, Fedlex — Classified Compilation of Federal
  Legislation (SR) and Official Compilation (AS): act records, consolidated
  texts and in-force status.

The consolidated text is the informative version; the legally binding text is
the one in the Official Compilation. Federal law only: no cantonal law, and EU
law is in the EUR-Lex pack. `data_as_of` is the date of the live read. Read
live on each call; nothing is copied. A search or lookup that finds nothing
returns `empty_reason`, not an empty success. Where one SR number has a
repealed act and its successor, the act in force is returned; if that is still
ambiguous the matches are listed and the AS reference picks one.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "switzerland-law": {
      "url": "https://gateway.pipeworx.io/switzerland-law/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/switzerland-law/mcp` returns the tools in the table
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
curl -X POST https://gateway.pipeworx.io/v1/tools/ch_law_search \
  -H 'Content-Type: application/json' \
  -d '{"query":"Datenschutz","lang":"de"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/ch_law_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-switzerland-law` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/switzerland-law/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "switzerland-law": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-switzerland-law"]
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
ask_pipeworx({ question: "your question about Switzerland Law data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
