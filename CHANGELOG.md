# Changelog

Every release of apogee-http-client, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

## 0.2.0 (2026-11-25)

Needs grav 1.13.0 or newer.

- Retries honour a 429's or 503's `Retry-After` (seconds or an HTTP date), and
  exponential waits are random up to the doubled delay (std's `Retry`), so
  clients failing together do not come back together. `maxSleepMs` (30 s) caps
  every wait.

## 0.1.1 (2026-11-21)

- Laid out by `grav fmt` and tidied as grav 1.11's lints suggest (`isEmpty()`
  over `size() == 0`, compound assignments); no change in behaviour.

## 0.1.0 (2026-11-21)

The first release. Needs grav 1.10.1 or newer.

- `Fetch` and `PendingRequest`: JSON, form, multipart and raw bodies, query
  strings, headers, tokens, base URLs, timeouts, redirects, retries with
  backoff.
- `ClientResponse` with status helpers, JSON paths and `throwIfFailed()`.
- `Fetch.pool`: requests sent at the same time.
- Fakes: URL patterns, sequences, handlers, stray-request prevention, and
  assertions on what was sent.
