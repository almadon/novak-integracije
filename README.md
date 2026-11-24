# novak-integracije

**The home for tools and adapters that give [Novak](https://github.com/almadon/novak) a capability its core stack should not have to know the internals of.**

> **Status: in testing.** The repository is set up and its CI runs, but it holds no integration at the moment. Everything Novak uses today speaks MCP itself.
> This project comes with no warranty or guarantee of fitness for any purpose. Use it at your own risk.
>
> **LLM use.** Built and maintained with substantial AI assistance (Claude Code). Design and review are by a human. See the conventions in [novak's AGENTS.md](https://github.com/almadon/novak/blob/HEAD/AGENTS.md).

## What this does

Novak reaches every capability (memory, notes, tasks, the house) through an MCP server. Most services already ship one, and for those nothing is needed here: the core registry lists them as `kind: external` and clients point straight at them. This repository exists for the other case, where an upstream has no MCP endpoint or gets identity wrong, and a small adapter has to be written and published as an image.

## Use cases

- You want to connect a service that has an API but no MCP server, and you need somewhere to keep the adapter and its image build.
- You are checking whether a new integration belongs here at all. The answer is usually no, and the bar below says why.

## Getting started

There is nothing to build today. When an integration is added it gets its own directory, a job in `.github/workflows/ci.yml` and an image on ghcr. The core stack uses these as prebuilt images, and the registry in the Novak repo records what a deployment runs.

## The bar for adding an integration

An integration is a dependency your assistant will trust with data. Before adding one:

1. **Prefer no code.** If the service has a native MCP endpoint, use it directly.
2. **Record it** in [docs/credits.md](https://github.com/almadon/novak/blob/HEAD/docs/credits.md) with a link and licence.
3. **Set a risk level** in the registry. `standard` reads or writes one service you already run. `elevated` can act outward and irreversibly. `dangerous` can change the machine or the stack. Anything above standard needs a written acceptance before it will start.
4. **Pin versions.** Some existing servers run `npx -y` at container start, which fetches the newest release every time and fails when the network is down. Don't add more.
5. **Never take a credential as a tool argument.** Secrets come from the environment, so the model never sees one. Identity must not be an argument either.

### The identity rule

If an integration serves more than one person, the caller must not be able to name the person. A model picks tool arguments from text it has read, including documents and email it did not write. If a user or namespace were a parameter, a note saying "look up user alice" would be a working attack.

Hindsight does this right. The memory bank is part of the MCP URL, each connection is scoped to one bank, and no tool takes a bank argument, so there is nothing to poison. An integration that fails this test needs a wrapper here that enforces it, or better, a different upstream. A first-party `memory-mcp` lived here for a backend that lacked it. It was deleted when the backend changed, which is the outcome to want: the shim was never the goal.

## Contents

| Integration | What it does | Status |
|---|---|---|
| None at the moment | Outline, Tududi and Hindsight all serve MCP themselves and are catalogued in the core registry as external entries | |

An empty table is the healthy state. Code belongs here only when an upstream leaves no choice.

### A worked example of "prefer no code"

Tududi was added for tasks and needed nothing here. It ships its own MCP server at `/api/mcp` (streamable HTTP, bearer token), so it is an external entry in the core registry and clients point straight at it. Check for a native endpoint before writing anything.

Tududi also has its own optional AI assistant, separate from its MCP server. It is off by default, and its `LLM_BASE_URL` can point at Novak's own model. Using the MCP server does not require enabling it. Vikunja stays supported as well, because removing a working integration for one you happen to prefer is not an upgrade.

## Documentation

This repository is small enough that the Novak docs cover it.

| Read | For |
|---|---|
| [novak README](https://github.com/almadon/novak#readme) | What the whole stack is |
| [docs/architecture.md](https://github.com/almadon/novak/blob/HEAD/docs/architecture.md) | Where integrations sit |
| [docs/identity.md](https://github.com/almadon/novak/blob/HEAD/docs/identity.md) | Why identity cannot be an argument |

## Licence

[MIT](LICENSE).
