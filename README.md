# redis for ZZ

Production Redis client for ZZ applications: typed commands, pipelining,
transactions, connection pooling, pub/sub, and keyspace scanning behind
one stable API. Every fallible call returns a typed `RError` with a fix
hint, so failures stay visible and actionable instead of surfacing as
panics or silent retries.

Requires ZZ ≥ 0.3.0 and Redis 6.2+.

## Install

```sh
zz add redis
```

Pin a version:

```sh
zz add redis@0.1.0
```

From source (until the registry release lands):

```sh
git clone https://github.com/zaidejjo/redis-zz
zz add redis --path ./redis-zz
```

## 30-second quickstart

```zz
import redis

func main() -> Result<int, redis.RError> {
    c := redis.connect("redis://127.0.0.1:6379")?
    _ := c.set("greet", "hello", 0)?
    match c.get("greet") {
        .ok(v) => {
            match v {
                .some(s) => println("got: {s}")
                .none => println("miss")
            }
        }
        .err(e) => println("err: {redis.message(e)}")
    }
    _ := c.close()
    .ok(0)
}
```

```sh
zz run main.zz
# got: hello
```

A miss is a value (`.none`), not an error. Every fallible call returns
`Result<T, redis.RError>` — use `?` or `match`, and `redis.message(e)`
for a log-safe one-liner.

## Connect

```zz
// TCP
c := redis.connect("redis://127.0.0.1:6379")?
// TCP + auth + db
c := redis.connect("redis://:secret@db.internal:6379/1")?
// TLS (verified)
c := redis.connect("rediss://db.example.com:6380?tls_ca=/etc/ssl/ca.pem")?
// Unix socket
c := redis.connect("redis+unix:///run/redis/redis.sock?db=1")?
// Full options
o := redis.parse_url("redis://127.0.0.1:6379?pool=8&retries=3&timeout_ms=500")?
c := redis.connect_with(o)?
```

## Common commands

```zz
_ := c.set("k", "v", 0)?          // ttl seconds, 0 = no expiry
_ := c.set("sess", "abc", 3600)?
v := c.get("k")?                  # Option<str>
n := c.incr("hits")?              // int
d := c.del(["a", "b"])?           // int removed
e := c.exists("k")?               # 0 | 1
_ := c.expire("k", 60)?
t := c.ttl("k")?                  # seconds, -1 = no ttl, -2 = missing
vals := c.mget(["a", "b"])?       # [Option<str>], miss per slot
p := c.ping()?                    # "PONG"
```

Binary-safe paths: `c.set_bytes(key, blob)?`, `c.get_bytes(key)?`
(`Option<[int]>`), `c.do_bytes(cmd, parts)?`. Anything untyped goes
through the generic path: `r := c.do("JSON.GET", ["doc", "."])?`, with
`c.do_with(cmd, args, idempotent, timeout_ms)` for overrides.

## Pipeline & transactions

```zz
out := redis.pipe(c, [["SET", "a", "1"], ["INCR", "n"], ["GET", "a"]])?
_ := redis.watch(c, ["bal:a", "bal:b"])?
res := redis.multi(c, [["DECRBY", "bal:a", "10"], ["INCRBY", "bal:b", "10"]])?
```

Conflict-safe retry:

```zz
work := redis.retryable(|| {
    match c.incr("n") {
        .ok(v) => .ok(v)
        .err(e) => .err(e)
    }
}, 3)
n := work()?
```

Pipelines never retry after the first byte — partial failures report
`{sent, received}`, no silent halves. On a pool, WATCH/MULTI needs an
exclusive checkout: `idx := redis.pool_pinned(p, 1000)?`.

## Pool, pub/sub, scan

```zz
p := redis.pool_open(o)?
r := redis.pool_with(p, "GET", ["k"], true, 0)?
s := redis.pool_stats(p)?          // {hits, misses, size, in_use, waits}
_u := redis.pool_close(p)
```

```zz
import redis.pubsub as pubsub
sub := pubsub.subscribe(o, ["events"])?
loop := true
while loop {
    match pubsub.next(sub) {
        .ok(out) => {
            println("{out.msg.channel}={out.msg.payload} gap={out.msg.gap}")
            sub = out.sub
            loop = out.msg.payload != "bye"
        }
        .err(e) => {
            println("sub err: {redis.message(e)}")
            loop = false
        }
    }
}
_u := pubsub.close(sub)
```

```zz
import redis.scan as scan
keys := scan.scan_all(c, "user:*", 500, true)?
```

## Errors

```zz
match c.get("k") {
    .ok(v) => println("{v}")
    .err(e) => println("{redis.message(e)}")
}
```

Kinds: `url`, `conn_refused`, `timeout`, `closed`, `protocol`, `redis`
(server `CODE: msg`), `moved`/`ask`, `auth`, `tls`, `conflict`,
`ambiguous` (bytes sent, no reply — **verify state before resending**),
`pool_exhausted`. Full table with fix hints: [DOCS.md](DOCS.md#error-table).

## Testing your app

```zz
import redis.mock as mock
m := mock.new()
_ := mock.set(m, "k", "v", 0)?
_u := mock.fail_next(m, "timeout")   # next op fails closed
```

Subset fake (no server needed) with fault injection. It may differ from
real Redis in edge cases — run live tests before release.

## Examples & docs

| | |
|---|---|
| [DOCS.md](DOCS.md) | full guide: retry policy, recipes, TLS setup, large values |
| [SECURITY.md](SECURITY.md) | guarantees, reporting |
| [BENCH.md](BENCH.md) | measured baseline + CI gates |
| [SCORECARD.md](SCORECARD.md) | 97-pt evidence checklist |
| [CHANGELOG.md](CHANGELOG.md) | release notes |
| `examples/` | `quickstart`, `rate_limiter`, `sessions`, `queue`, `cache_aside`, `binary_blob`, `all_transports` |

```sh
zz test tests/     # VM; live suites skip cleanly without a server
```

## Compatibility

| | |
|---|---|
| ZZ toolchain | ≥ 0.3.0 |
| Redis server | 6.2, 7.x (CI runs `redis:7`) |
| Engines | VM + native, byte-identical output |

## License

MIT — see [CHANGELOG.md](CHANGELOG.md) for release history.
