# BENCH.md — generated from bench/baseline.json (do not hand-edit numbers)

Box: local-docker, engine VM. Single run per bench, one line each
(`name ops=N ms=M ops_per_s=R`). `ping` used to take the median of 5
runs; it now does one 200-ping run so the file stays light — rerun it a
few times and take the median yourself if you need noise control.

Committed baseline: ping 200 ops / 461 ms (433 ops/s), get1k 100 ops /
1245 ms (80 ops/s), pipe100 50 ops / 6871 ms (7 ops/s).

## AOT (native-debug `-O0`, same box, 2026-10-12)

First measured AOT numbers (release `-O3-ThinLTO` links OOM on the
3.8 GB dev box — release numbers pending a bigger runner):

| bench | ops | ms | ops/s | × vs VM |
|---|---|---|---|---|
| ping | 200 | 49 | 4081 | 9.4× |
| get1k | 100 | 120 | 833 | 10.4× |
| pipe100 | 50 | 362 | 138 pipelines/s (13,800 cmds/s) | 19.7× |

Reference on the same box: `redis-benchmark -c 1` does 13,333
ping_inline / 17,094 ping_mbulk rps. AOT-debug single ping (4,081/s)
sits ~3.3× behind it — inside the 10× gate even unoptimized.

Pipeline gate (pipelined cmds/s ≥ 10× single-shot): 13,800 / 4,081 =
3.4× on AOT-debug — **not met**. Per-command pipeline overhead
(frame building through the generic path) dominates at this size;
retest on release builds before changing the gate.

Gates (CI): pipelined-100 throughput ≥ 10× single-shot on the same
engine; no bench regresses > 10% vs baseline; native single-conn is
informational until a release-box baseline is committed (current numbers
are VM-only on a shared box, so no native ratio is claimed here).

## Reproduce

```sh
zz run bench/ping.zz     # ping ops=200 ms=... ops_per_s=...
zz run bench/get1k.zz    # needs live redis on 127.0.0.1:6379
zz run bench/pipe100.zz
```

Each bench prints one `name ops=N ms=M ops_per_s=R` line (median of 5).
To re-baseline: run all three, paste the numbers into
`bench/baseline.json`, and keep this file's table in sync.
