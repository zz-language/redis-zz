# DOCS.md — redis package guide

## Quickstart

```zz
import redis

func main() -> Result<int, redis.RError> {
    c := redis.connect("redis://127.0.0.1:6379")?
    _ := c.set("greet", "hello", 0)?
    m := c.get("greet")?
    match m {
        .some(s) => println(s)
        .none => println("miss")
    }
    _ := c.close()
    .ok(0)
}
```

Custom options via `redis.connect_with(o)` with
`o := redis.parse_url("redis://...?pool=8&retries=3&timeout_ms=500&connect_timeout_ms=1000&proto=2&name=my-app")?`.

## URL matrix

`redis://host:port[/db]` (TCP), `redis://:pass@host:port/1` (+AUTH/SELECT),
`rediss://host:6380?tls_ca=/ca.pem` (verified TLS),
`?tls_insecure=1` (unverified, fires hook event),
`?tls_sni=name` (hostname override), `redis+unix:///tmp/redis.sock?db=1`.
Defaults: port 6379, db 0, pool 8, retries 3, proto 2. Error text never
contains secrets (`import redis.url as url`, then `url.redacted(o)` to
log a masked URL; the live redaction test enforces it).

## Error table

`url` (bad segment, fix named part), `conn_refused` (start server),
`timeout:connect|write|read|handshake` (raise matching timeout),
`closed:idle|command|pipeline|subscribe` (reconnect; retry only
idempotent), `protocol` (peer not Redis; connection closed),
`redis:CODE: msg` (WRONGTYPE/OOM/BUSY/LOADING/READONLY/NOSCRIPT — fix
data/command), `moved/ask` (cluster signal; routing in v2), `auth`
(check credentials; never retried), `tls:handshake_failed|
cert_untrusted|cert_hostname|refused_tls|timeout` (check ca/hostname; no
plaintext fallback), `conflict` (WATCH tripped; retry transaction),
`ambiguous:cmd` (VERIFY state before resending), `pool_exhausted`
(raise pool=, shorten holds).

## Idempotency (auto-retry only if unsent, or idempotent + IO error)

Retry: GET MGET EXISTS TTL PTTL TYPE STRLEN HGET HGETALL SMEMBERS ZRANGE
SCAN* PING, plain 2-arg SET, DEL HSET SADD SREM ZADD PERSIST EXPIREAT
SELECT, EXPIRE (relative: idempotent-with-drift). Never auto-retried
(→ `ambiguous`): INCR DECR APPEND LPUSH/RPUSH LPOP/RPOP BRPOP XADD
PUBLISH GETDEL GETSET option-SET EVAL/EVALSHA, unknown commands (unless
`idempotent=true` explicit). Pipelines/MULTI-EXEC never retry after the
first byte; partial failures report `{sent, received}`.

## Retry/backoff, pipeline, transactions, pub/sub, scan, TLS, mock

Backoff `min(1000, 20·2^attempt)` ms; `redis.backoff_ms(a)` is pure.
`@retryable(tries=3)` re-runs a thunk on `conflict` only.
Pipeline: `redis.pipe(c, [[...], ...])?` (one round trip, ordered).
Transactions: `watch` then `multi` (WATCH pins the connection — on a
pool, `checkout` exclusively, then `checkin`). Pub/sub owns a dedicated
connection; `next` returns message + updated subscriber; `gap` counts
reconnects since last delivery. SCAN is at-least-once (`scan_all` with
`dedupe=true` collapses repeats). TLS: verified by default; without
`tls_ca` self-signed fails `cert_untrusted`; wrong SNI fails
`cert_hostname`. `mock` fake is subset-only, single-threaded, fault
injectable (`fail_next/delay_ms/trip_watch`); live tests stay mandatory.
Ambiguous = VERIFY before resending, never blind-retry. Single reads cap
64 MiB — use GETRANGE / chunked frames for huge values.
