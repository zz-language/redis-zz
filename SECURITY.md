# SECURITY.md

Supported: latest `redis` 0.1.x on Redis 6.2 / 7.x.

- TLS verify ON by default (hostname + SNI); `tls_insecure=1` fires one
  `on_insecure` hook event. No silent downgrade: `rediss://` failures
  stay `tls(...)`, never retried in plaintext.
- No secrets in errors, hook events, or Client/Options display
  (`redacted()` masks passwords; live test greps error strings).
- Parser limits before allocation: 64 MiB single-read cap, oversized
  declarations fail closed. Length-prefixed encoding is injection-safe
  (binary-key test covers `"a\r\nFLUSHALL\r\n"` as data).
- Plaintext + password to non-loopback fires `on_insecure`.

Out of scope: cluster routing (MOVED/ASK typed only), client-side
caching, EVALSHA helpers. Report issues with the `security` label;
no live credentials in reports.
