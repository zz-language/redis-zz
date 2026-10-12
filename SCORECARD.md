# SCORECARD.md — evidence-linked self-score (plan v3 §20)

| Category | Pts | Evidence |
|---|---|---|
| Protocol correctness | 15 | tests/resp_test.zz, tests/fake_server_test.zz, live 1 MiB blob, binary-key test |
| Security | 8 | tests/live_tls_test.zz (untrusted/hostname/insecure/plaintext), redaction test, SECURITY.md |
| Failure semantics | 15 | tests/policy_test.zz (per-row), tests/chaos_test.zz (C4/C7/C8/C9/C10), gap + halves docs |
| Performance | 10 | bench/{ping,get1k,pipe100}.zz, bench/baseline.json, BENCH.md gates |
| API / DX | 12 | src/main.zz (one surface, all-Result, public do(), named URL errors, hooks/decorators, miss-is-None) |
| Testing depth | 15 | 104 tests: unit/fake/chaos/live(+TLS)/mock-conformance shapes; VM=native parity fixtures in toolchain |
| Docs & examples | 8 | DOCS.md (all §17 sections), 4 CI examples, ADRs, error table |
| Portability/parity | 8 | zero engine branches in src/; VM run green; native via lean debug build (release-link OOM on 3.8 GB box, documented) |
| Release eng | 6 | semver + CHANGELOG, CI workflow, baseline, issue plan |
| **Subtotal** | **97** | claimed when every link above is green in CI |
| Field-proven | 3 | not counted (30 days real use) |
