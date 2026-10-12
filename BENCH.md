# BENCH.md — generated from bench/baseline.json (do not hand-edit numbers)

Box: local-docker, engine VM. Median of 5 runs per bench, one line each
(`name ops=N ms=M ops_per_s=R`).

Committed baseline: ping 200 ops / 461 ms (433 ops/s), get1k 100 ops /
1245 ms (80 ops/s), pipe100 50 ops / 6871 ms (7 ops/s).

Gates (CI): pipelined-100 throughput ≥ 10× single-shot on the same
engine; no bench regresses > 10% vs baseline; native single-conn is
informational until a release-box baseline is committed (current numbers
are VM-only on a shared box, so no native ratio is claimed here).
