<div align="center">

# Jeff's Brain

**Polyglot memory for LLM agents. Spec-driven, local-first, hosted-optional.**

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![TS CI](https://github.com/jeffs-brain/memory/actions/workflows/ci.yml/badge.svg)](https://github.com/jeffs-brain/memory/actions/workflows/ci.yml)
[![Go CI](https://github.com/jeffs-brain/memory/actions/workflows/go.yml/badge.svg)](https://github.com/jeffs-brain/memory/actions/workflows/go.yml)
[![Eval Smoke](https://github.com/jeffs-brain/memory/actions/workflows/eval-smoke.yml/badge.svg)](https://github.com/jeffs-brain/memory/actions/workflows/eval-smoke.yml)

[Docs](https://docs.jeffsbrain.com) · [Spec](https://github.com/jeffs-brain/memory/tree/main/spec) · [Protocol](https://github.com/jeffs-brain/memory/blob/main/spec/PROTOCOL.md) · [MCP tools](https://github.com/jeffs-brain/memory/blob/main/spec/MCP-TOOLS.md)

</div>

---

## What we build

A cross-language memory and hybrid retrieval library for LLM agents. A single language-neutral [`spec/`](https://github.com/jeffs-brain/memory/tree/main/spec) defines the HTTP protocol, storage contract, retrieval algorithms, query DSL, and MCP tool surface. The TypeScript, Go, and Python SDKs each ship a full implementation of that spec, conformance-tested against the same fixtures.

- **Four-stage memory pipeline.** `extract`, `reflect`, `consolidate`, `recall`, plus session buffers, episodes, and a feedback loop.
- **Hybrid retrieval.** SQLite FTS5 BM25 plus vector search, fused with Reciprocal Rank Fusion (`k=60`), a five-rung retry ladder, intent reweight, and cross-encoder rerank.
- **Pluggable stores.** Filesystem, Git, in-memory, and HTTP (wire-protocol client). Postgres + pgvector via an adapter.
- **Pluggable auth.** RBAC out of the box, OpenFGA adapter when you need relationship-based access.
- **Offline by default.** Runs entirely locally against Ollama and a built-in hash embedder, or against OpenAI, Anthropic, or TEI when you want quality.
- **MCP native.** Three wire-compatible stdio servers (TS, Go, Python) exposing the same 11 canonical `memory_*` tools to Claude Code, Claude Desktop, Cursor, Windsurf, and Zed.

## Status

| Language | Package | Install |
| --- | --- | --- |
| TypeScript | [`@jeffs-brain/memory`](https://www.npmjs.com/package/@jeffs-brain/memory) | `npm i -g @jeffs-brain/memory` |
| TypeScript | [`@jeffs-brain/memory-mcp`](https://www.npmjs.com/package/@jeffs-brain/memory-mcp) | `npx -y @jeffs-brain/memory-mcp` |
| TypeScript | [`@jeffs-brain/memory-postgres`](https://www.npmjs.com/package/@jeffs-brain/memory-postgres) | `npm i @jeffs-brain/memory-postgres` |
| TypeScript | [`@jeffs-brain/memory-openfga`](https://www.npmjs.com/package/@jeffs-brain/memory-openfga) | `npm i @jeffs-brain/memory-openfga` |
| TypeScript | [`@jeffs-brain/install`](https://www.npmjs.com/package/@jeffs-brain/install) | `npx @jeffs-brain/install` |
| Go | `github.com/jeffs-brain/memory/sdks/go` | `go install github.com/jeffs-brain/memory/go/cmd/memory@latest` |
| Go | MCP wrapper | `go install github.com/jeffs-brain/memory/go/cmd/memory-mcp@latest` |
| Python | [`jeffs-brain-memory`](https://pypi.org/project/jeffs-brain-memory/) | `uv add jeffs-brain-memory` |
| Python | [`jeffs-brain-memory-mcp`](https://pypi.org/project/jeffs-brain-memory-mcp/) | `uvx jeffs-brain-memory-mcp` |

**Conformance**: 28/29 HTTP cases green on every SDK. **Tri-SDK smoke benchmark**: 19/20 on every SDK against Ollama `gemma3:latest`. **LongMemEval**: full replay and agentic ingest harness ships in `eval/`.

## Quick start

### As a CLI

```bash
npm i -g @jeffs-brain/memory
memory init
memory ingest ./docs
memory search "what did we decide about auth?"
memory ask --brain default --question "who owns billing?"
```

Equivalent flows in Go (`go install .../cmd/memory@latest`) and Python (`uv tool install jeffs-brain-memory`). Every SDK's `memory serve` speaks the same wire protocol, so clients in any language can drive any daemon.

### Wire it into every agent at once

```bash
npx @jeffs-brain/install
```

Detects Claude Code, Claude Desktop, Cursor, Windsurf, and Zed, and installs the MCP server into each. Pick `local` for a zero-config filesystem brain at `~/.jeffs-brain`, or `hosted` to point at the platform with a `JB_TOKEN`.

### Embed in your agent

```ts
import { createMemory, createMemStore, createHashEmbedder, createMemoryLifecycle } from '@jeffs-brain/memory'

const mem = createMemory({
  store: createMemStore(),
  embedder: createHashEmbedder(),
  provider,
  scope: 'project',
  actorId: 'me',
})

const lifecycle = createMemoryLifecycle({ memory: mem })
await lifecycle.beforeTurn({ message })
await lifecycle.afterTurn({ messages, sessionId })
await lifecycle.endSession({ messages, sessionId, consolidate: true })
```

## Repositories

| Repo | What it is |
| --- | --- |
| [`memory`](https://github.com/jeffs-brain/memory) | OSS monorepo: spec, TS/Go/Python SDKs, MCP wrappers, installer, eval harness, docs site |
| `platform` | Private platform: backend API, dashboard, CLI, MCP bridge, marketing site, product docs |

### OSS layout (`jeffs-brain/memory`)

```
spec/               Protocol, storage, algorithms, query DSL, MCP tools, fixtures, conformance
sdks/
  ts/memory         Core TS SDK, CLI, HTTP daemon, query DSL, retrieval, memory pipeline
  ts/memory-postgres Postgres + pgvector adapter (tsvector BM25, halfvec cosine, RRF)
  ts/memory-openfga OpenFGA authorisation adapter
  go/               Full Go SDK, pure-Go or sqlite-vec, LongMemEval replay runner
  py/               Full Python SDK, FTS5 + sqlite-vec + trigram fuzzy fallback
mcp/
  ts                @jeffs-brain/memory-mcp stdio server
  go                memory-mcp binary
  py                jeffs-brain-memory-mcp package
install/            Cross-agent MCP installer (@jeffs-brain/install)
examples/           Runnable hello-world per language
eval/               LongMemEval runner, cross-SDK scorer, datasets, results
docs/               docs.jeffsbrain.com (Astro)
```

### Hosted platform

Closed-source monorepo sitting on top of the OSS packages:

- `apps/backend` (Hono) — auth, billing, tenant management, HTTP API.
- `apps/dashboard` (React + Vite) — brains, tenants, usage, keys.
- `apps/mcp` — hosted MCP bridge.
- `apps/cli` — operational CLI.
- `apps/website` (Astro) — marketing at jeffsbrain.com.
- `apps/docs` (Astro) — product docs at docs.jeffsbrain.com.

## Design principles

1. **The spec wins.** When a behaviour is not in [`spec/`](https://github.com/jeffs-brain/memory/tree/main/spec), it is not part of the contract. When an SDK disagrees with the spec, the SDK is wrong.
2. **No lock-in.** Start local, swap to Postgres, swap to hosted, swap back. Same data shape, same CLI, same MCP tools, same HTTP wire.
3. **Boring where it counts.** SQLite, Postgres, pgvector, HTTP, JSON. Novel work goes into retrieval quality and the memory pipeline, not the plumbing.
4. **Three languages, one contract.** Every SDK runs the same HTTP conformance suite and the same cross-SDK retrieval benchmark in `eval/`.
5. **Agents as first-class clients.** The 11 `memory_*` MCP tools are a product surface, not a side project, and they are identical across TS, Go, and Python.

## Contributing

Contributions are welcome on the OSS repo. Start with [`CONTRIBUTING.md`](https://github.com/jeffs-brain/memory/blob/main/CONTRIBUTING.md) for dev setup, commit style, and the DCO. Security reports go via [`SECURITY.md`](https://github.com/jeffs-brain/memory/blob/main/SECURITY.md). Community standards live in [`CODE_OF_CONDUCT.md`](https://github.com/jeffs-brain/memory/blob/main/CODE_OF_CONDUCT.md).

## Licence

OSS repo is Apache-2.0. Bundled third-party components are listed in [`NOTICE`](https://github.com/jeffs-brain/memory/blob/main/NOTICE). The hosted platform is proprietary.
