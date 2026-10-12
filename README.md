# redis — pure-ZZ Redis client (sync, both engines)

Sync Redis client with zero native deps. One `do(cmd, args)` core, typed
sugar on top. VM and native produce identical output.

```zz
import redis

func main() -> Result<int, redis.RError> {
    c := redis.connect("redis://127.0.0.1:6379")?
    _ := c.set("k", "v", 0)?
    m := c.get("k")?
    match m {
        .some(s) => println(s)
        .none => println("miss")
    }
    _ := c.close()
    .ok(0)
}
```

Transports: `redis://` (TCP), `rediss://` (TLS, verify on by default),
`redis+unix://` (Unix sockets). Every public fn returns `Result`; a miss
is `.none`, never an error; ambiguous outcomes are typed errors, never
silent retries. See DOCS.md, SECURITY.md, BENCH.md, SCORECARD.md.
