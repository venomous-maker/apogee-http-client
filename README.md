# apogee-http-client

Outgoing HTTP for [Apogee](https://github.com/venomous-maker) apps: a fluent
request builder over Grav's standard `HttpClient`, with JSON, form, multipart
and raw bodies, tokens, retries with backoff, timeouts, requests sent at the
same time, and fakes so tests never touch the network. The facade is `Fetch`
(the standard library already has an `Http` class).

```typescript
import "std"
import "apogee-http-client"
use apogee.*

let res = Fetch.withToken(token).acceptJson().retry(3, 100).get("https://api.example.com/users", {"page": "2"})
if (res.ok()) { for (u in res.json().get("data").asArray()) { ... } }
Fetch.post("https://api.example.com/users", {"name": "Ada"}).throwIfFailed()

let all = Fetch.pool(fn(p: HttpPool) {
    p.add("users").get("https://api.example.com/users")
    p.add("orders").get("https://api.example.com/orders")
})                                    // both at once
all.get("users").json()
```

## Requests

Start with `Fetch.request()` or any of these, and chain:

| | |
|---|---|
| `baseUrl(url)`, `withQuery(map)` | |
| `withHeader(n, v)`, `withHeaders(map)`, `withToken(t, kind = "Bearer")`, `withBasicAuth(u, p)`, `withUserAgent`, `accept(type)`, `acceptJson()` | |
| `asJson()` (the default), `asForm()`, `withBody(text, contentType)`, `attach(name, bytes, filename, type)` | bodies; `data` is JSON-plain (maps, arrays, scalars) |
| `timeoutMs(ms)`, `timeoutSeconds(s)`, `withoutRedirecting()`, `maxRedirects(n)` | |
| `retry(times, sleepMs, when = null, throwOnFailure = true, exponential = false)` | tries again after a connection error, a 5xx or a 429 (or when `when(response, exception)` says so); doubles the wait with `exponential`; throws `HttpClientException` when the last try fails |
| `get(url, query)`, `head`, `post(url, data)`, `put`, `patch`, `delete`, `send(method, url, data, query)` | send it |

`Fetch.withToken(...)`, `Fetch.get(...)` and the rest are shortcuts for
`Fetch.request()....`. `Fetch.instance().globalHeaders` go on every request.

## Responses

`status`, `body()`, `json()`, `jsonAt("data.0.name")`, `header(name)`,
`ok()`, `successful()`, `redirect()`, `failed()`, `clientError()`,
`serverError()`, `notFound()`, `unauthorized()`, `forbidden()`,
`tooManyRequests()`, `throwIfFailed()`, `elapsedMs`, and the `request` it
answers. A connection that fails throws `HttpClientException` (its `response`
is null).

## Pools

`Fetch.pool(build, concurrency = 16)` runs the requests `build` adds (`p.add(name)`,
then options and a verb) at the same time, and returns their responses in
order and by name; a request that could not be sent leaves a null response
and its exception in `errors`.

## Fakes

```typescript
Fetch.fake("api.example.com/users*", Fetch.response({"data": []}, 200))
Fetch.fake("api.example.com/flaky", Fetch.sequence().pushStatus(503).push("ok"))
Fetch.fakeUsing("api.example.com/echo*", fn(r: ClientRequest) -> ClientResponse { return Fetch.reply({"got": r.field("name")}, 201) })
Fetch.preventStrayRequests()                 // anything else throws
...
Fetch.assertSent(fn(r: ClientRequest) -> bool { return r.method == "POST" && r.field("name") == "Bo" })
Fetch.assertSentCount(3)
Fetch.reset()
```

Patterns use `*` for anything (a leading `*` is implied, so
`"api.example.com/*"` matches the https URL). `Fetch.fake()` alone answers
every request with an empty 200. A request records its method, URL, headers,
body and content type: `field(name)` reads a JSON or form body, `query(name)`
the query string.

## Tests

```bash
grav test tests/http_client.grav
```

They run a local server for the real requests.
