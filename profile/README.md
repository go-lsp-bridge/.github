<p align="center"><img src="https://raw.githubusercontent.com/go-lsp-bridge/brand/main/social/go-lsp-bridge.png" alt="go-lsp-bridge" width="640"></p>

<h1 align="center">go-lsp-bridge</h1>
<p align="center"><strong>A pure-Go WebSocket↔stdio bridge for JSON-RPC LSP traffic — drop a real language server behind a web editor, no cgo.</strong></p>

<p align="center">
  🌐 <a href="https://go-lsp-bridge.github.io">Website</a> ·
  📚 <a href="https://go-lsp-bridge.github.io/docs/">Documentation</a>
</p>

<p align="center">
  <a href="https://go-lsp-bridge.github.io/docs/"><img alt="Docs" src="https://img.shields.io/badge/docs-mkdocs--material-4F46E5?style=flat-square"></a>
  <a href="https://github.com/go-lsp-bridge/lspbridge/blob/main/LICENSE"><img alt="License: BSD-3-Clause" src="https://img.shields.io/badge/license-BSD--3--Clause-blue?style=flat-square"></a>
  <img alt="Go 1.26.4+" src="https://img.shields.io/badge/go-1.26.4%2B-00ADD8?style=flat-square&logo=go&logoColor=white">
  <img alt="Coverage 100%" src="https://img.shields.io/badge/coverage-100%25-1a7f37?style=flat-square">
</p>

---

go-lsp-bridge is a **pure-Go (no cgo)** bridge that relays JSON-RPC
[Language Server Protocol](https://microsoft.github.io/language-server-protocol/)
traffic between a **WebSocket** and a language-server subprocess's **stdio**. It
owns the `Content-Length` framing on the stdio side so the WebSocket carries the
bare JSON object — exactly what browser LSP clients such as
`codemirror-languageserver` and `@open-rpc/codemirror-lsp` expect.

The result: a web editor gets real completions, hovers, diagnostics and
go-to-definition from an actual language server (texlab, gopls, pyright,
typescript-language-server, rust-analyzer, …) behind a single HTTP handler —
with per-connection subprocesses and both global and per-user concurrency caps.

## Repositories

| Repo | What it is |
|------|------------|
| [**lspbridge**](https://github.com/go-lsp-bridge/lspbridge) | the bridge: `HandleWS`, the `Servers` registry + `DefaultServers`, `WithSubject` concurrency caps, `AvailableLanguages`, and the bundled `cmd/fake-lsp` test stub |
| [**docs**](https://github.com/go-lsp-bridge/docs) | MkDocs Material documentation, versioned with [mike], served at [/docs/](https://go-lsp-bridge.github.io/docs/) |
| [**go-lsp-bridge.github.io**](https://github.com/go-lsp-bridge/go-lsp-bridge.github.io) | the Hugo landing page |
| [**brand**](https://github.com/go-lsp-bridge/brand) | logos and brand assets |

## Principles

- **Pure Go, zero cgo.** Cross-compiles and embeds anywhere; a static binary by
  default.
- **A bridge, not a client.** It transports JSON-RPC untouched, only translating
  the transport framing (WebSocket text frames ↔ stdio `Content-Length`); the
  editor and the language server stay the sources of truth.
- **Reusable by construction.** The launcher registry is a plain map with neutral
  `LSPBRIDGE_*` environment overrides — no host-application naming baked in.
- **Safe under load.** Per-connection subprocesses plus a global and a per-user
  concurrency ceiling keep one runaway editor from saturating the host.
- **100% test coverage** is the target, enforced as a CI gate — including every
  error branch — with a hermetic WS→subprocess→WS round-trip against a bundled
  stub.

## Status

**Complete and released.** `HandleWS` relays JSON-RPC in both directions with
correct `Content-Length` framing; `DefaultServers()` ships launchers for latex /
go / python / typescript / javascript / rust with neutral env overrides;
`WithSubject` drives the per-user cap; `AvailableLanguages()` reports resolvable
servers; and `EncodeError` surfaces setup failures as JSON-RPC error frames. The
bundled `cmd/fake-lsp` stub drives a hermetic round-trip test at **100% coverage
including error branches**, `gofmt` + `go vet` clean, CI green across the six
64-bit Go targets (amd64, arm64, riscv64, loong64, ppc64le, s390x) — the
exec/WebSocket tests are gated off the qemu cross-arch lanes while the pure
framing / registry / concurrency logic still runs there.

BSD-3-Clause.

[mike]: https://github.com/jimporter/mike
