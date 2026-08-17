# novak-integracje

Integrations for [Novak](https://github.com/almadon/novak) — the MCP servers and adapters that
give the assistant its capabilities.

> **Naming:** the repo is `novak-integracje`, following the `srz` / `konzol`
> pattern. The local working directory may be spelled `integracije` — the
> directory name is arbitrary and nothing depends on it.

## What belongs here

Anything that gives the assistant a new capability, and that the core stack
should not have to know the internals of:

- **First-party MCP servers** we wrote (`memory-mcp/`)
- **Adapters** that wrap a service which doesn't speak MCP itself
- Eventually, **third-party integrations** — this is the "store"

## What does NOT belong here

- **The registry** (`srz/registry/mcp-servers.yaml`) — that records what *your*
  install runs and at what risk level. It's per-deployment, not a catalog.
- **The reconciler** — the only thing with power over containers stays in core.
- **Anything that speaks MCP already.** If a service has its own MCP endpoint,
  catalogue it as `kind: external` in the registry and write no code. Outline
  is the worked example: it serves `/mcp` natively, so it needs nothing here.
  **Wrapping is the fallback, not the default.**

## The bar for adding an integration

An integration is a dependency your assistant will trust with data. Before
adding one:

1. **Prefer no code.** Native MCP endpoint? Use it directly.
2. **Record it** in [`novak/docs/credits.md`](https://github.com/almadon/novak/blob/HEAD/docs/credits.md) with a link
   and licence.
3. **Set a risk level** in the registry. `standard` reads or writes one service
   you already run; `elevated` can act outward irreversibly; `dangerous` can
   change the machine or the stack. Anything above standard needs a written
   acceptance before it will start.
4. **Pin versions.** Several existing servers run `npx -y` at container start,
   which fetches the newest release every time and fails when the network is
   down. Don't add more of those.
5. **Never take a credential as a tool argument.** Secrets come from the
   environment; the model must never see one. Identity must not be a tool
   argument either — see below.

## The identity rule

If an integration is multi-user, the caller must not be able to name the user.

This isn't a style preference. A model chooses tool arguments based on text it
has read, including documents and email it didn't write. If `user_id` were a
parameter, a note saying *"look up user alice"* would be a working attack.

`memory-mcp/` is the reference implementation: identity comes from a token
bound to the connection, and the MCP server object is rebuilt per request with
that identity closed over, so the tools have no way to refer to anyone else.

## Contents

| Integration | What it does | Status |
|---|---|---|
| [`memory-mcp/`](memory-mcp/README.md) | MCP front end for Mem0, with per-user scoping | **never run** |

## Wanted

- **Tududi** — replacing Vikunja for tasks. Needs either a native MCP endpoint
  (check first) or an adapter here. Vikunja stays until this lands; this is an
  addition, not a swap-and-delete.

## Status

Nothing here has been built or run. `memory-mcp` has no lockfile yet, so its
Docker build and CI both fail until `npm install` is run and the result
committed.
