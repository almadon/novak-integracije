# novak-integracije

Integrations for [Novak](https://github.com/almadon/novak) — the MCP servers and adapters that
give the assistant its capabilities.

The default extension catalog for Novak.

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

If an integration is multi-user, **the caller must not be able to name the
user.**

This isn't a style preference. A model chooses tool arguments based on text it
has read, including documents and email it didn't write. If a user or namespace
were a parameter, a note saying *"look up user alice"* would be a working
attack.

Hindsight is the worked example of doing it right: the memory bank is part of
the MCP URL, each connection is scoped to one, and no tool takes a bank
argument. There is nothing to poison.

An integration that fails this test needs a wrapper here that enforces it — or,
better, a different upstream. `memory-mcp` used to live in this repo doing
exactly that for a backend that lacked it. It was deleted when the backend
changed, which is the outcome to want: the shim was never the goal.

## Contents

| Integration | What it does | Status |
|---|---|---|
| _(none)_ | Everything currently used speaks MCP natively | — |

That table being empty is the healthy state, not a gap. Outline, Tududi and
Hindsight all serve MCP themselves and are catalogued in the core registry as
external entries. Code belongs here only when an upstream leaves no choice.

## Tududi: a worked example of "prefer no code"

Tududi was added for tasks, and needed **nothing in this repo**. It ships its
own MCP server at `/api/mcp` (streamable HTTP, bearer token), so it is
catalogued as an external entry in the core registry and clients point straight
at it. No adapter, no wrapper container, no code to maintain.

That is the outcome to aim for. Check for a native endpoint before writing
anything.

Vikunja stays. Tududi is an addition, not a replacement — other people use this
too, and removing a working integration because it is not the one you happen to
prefer is not an upgrade.

**Worth knowing:** Tududi also has its own optional AI assistant, separate from
its MCP server. It is off by default and its `LLM_BASE_URL` can point at oMLX,
so it can use Novak's own model rather than a cloud provider. Using MCP does
not require enabling it.

## Status

No integrations currently need code. If one does, it gets its own directory, a
job in `.github/workflows/ci.yml`, and an image published to ghcr.
