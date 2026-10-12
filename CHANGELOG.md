# CHANGELOG.md

## 0.1.1 (unreleased)

- Package metadata: description + MIT license (registry warnings fixed).

## 0.1.0 (unreleased)

- Core: errors/url/resp/transport/conn/client/cmds — TCP + TLS + Unix,
  HELLO/AUTH/SELECT handshake, typed commands over public `do()`,
  binary-safe `*_bytes` paths, 1 MiB round trip.
- v0.2: fixed `pool`, `pipe` (pipeline + MULTI/EXEC/WATCH, no silent
  halves), `retry` (conflict-only `@retryable`, pure `backoff_ms`,
  `@timed`), `hooks` metrics.
- v0.3: `pubsub` (resubscribe + gap), `scan` family (dedupe opt-in),
  `mock` fake (subset + fault injection).
- Tests: 104 green (VM) — pure tables, scripted fake-server framing,
  chaos deaths, env-gated live TCP/TLS/Unix.
- Docs: README/DOCS/SECURITY/BENCH/SCORECARD/CHANGELOG, ADRs, 4 CI
  examples, measured `bench/baseline.json`.

Frozen at 1.0: `do()`, `Options`, `RError` variants, `Value` accessors,
`connect`/`connect_with`, pool `with`/`pinned`.
