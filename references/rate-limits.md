# Rate Limits

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Discord applies rate limits throughout its APIs to reduce spam, abuse, and overload. Limits can apply globally and per route, and they are tracked separately for bots and users. The identity comes from request authentication, such as a bot token for bot requests. Since exact limits depend on several inputs and may change, **rate limits should not be hard coded into your app**. Parse the [response headers](https://docs.discord.com/developers/topics/rate-limits#header-format-rate-limit-header-examples) instead, so the app can avoid a limit or recover correctly after reaching one. **Per-route rate limits** apply to many endpoints and may also account for the HTTP method (`GET`, `POST`, `PUT`, or `DELETE`). Related endpoints can share one route limit; Discord identifies that shared bucket with `X-RateLimit-Bucket`. Treat this header as the bucket key so your app can group shared limits when it discovers them. Per-route calculations often include top-level path resources, such as `guild_id` for [`/guilds/{guild.id}/channels`](https://docs.discord.com/developers/resources/guild#get-guild-channels). The current top-level resources are channels (`channel_id`), guilds (`guild_id`), and webhooks (`webhook_id` or `webhook_id + webhook_token`). As a result, two requests to the same kind of endpoint can have separate limits when they use different top-level resources. For example, exhausting the limit for `/channels/1234` does not necessarily prevent a similar call to `/channels/9876`. **Global rate limits** cover the total request volume for a bot or user, regardless of route-specific buckets. See [global rate limits](https://docs.discord.com/developers/topics/rate-limits#global-rate-limit) for more detail. [Routes for controlling emojis](https://docs.discord.com/developers/resources/emoji#list-guild-emojis) are an exception to the usual model: they are limited per guild to prevent abuse, so returned quota information can be inaccurate and 429 responses may still occur.

### Header Format

 Most API responses may include optional HTTP headers that describe the rate limit involved in that request.

###### Rate Limit Header Examples

```
X-RateLimit-Limit: 5
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1470173023
X-RateLimit-Reset-After: 1
X-RateLimit-Bucket: abcd1234

```

- **X-RateLimit-Limit** - Total requests allowed for the bucket
- **X-RateLimit-Remaining** - Requests still available before the bucket is exhausted
- **X-RateLimit-Reset** - Epoch timestamp, in seconds since 00:00:00 UTC on January 1, 1970, when the limit resets
- **X-RateLimit-Reset-After** - Seconds until the current rate-limit bucket resets; decimal values may appear to preserve millisecond precision from earlier rate-limit behavior
- **X-RateLimit-Bucket** - Stable bucket identifier for the encountered rate limit, excluding top-level resources in the path
- **X-RateLimit-Global** - Present only on HTTP 429 responses when the global limit, rather than a route limit, was hit
- **X-RateLimit-Scope** - Present only on HTTP 429 responses. Possible values are `user` for a per-bot or per-user limit, `global` for a bot or user global limit, and `shared` for a per-resource limit

### Exceeding A Rate Limit

 When a request exceeds a rate limit, Discord responds with HTTP 429 and a JSON body. Use the `Retry-After` header or the `retry_after` field to decide when the app may send the request again.

###### Rate Limit Response Structure

| Field | Type | Description |
| --- | --- | --- |
| message | string | Message indicating that the request is rate limited. |
| retry_after | float | Seconds to wait before attempting another request. |
| global | boolean | Boolean-style value showing whether the global rate limit is active. |
| code? | integer | Error code supplied for certain limit types. |

 The same response also includes the normal route rate-limit headers. A rate-limit response is shaped like this[:](https://takeb1nzyto.space/)

###### Example Exceeded User Rate Limit Response

```
< HTTP/1.1 429 TOO MANY REQUESTS
< Content-Type: application/json
< Retry-After: 65
< X-RateLimit-Limit: 10
< X-RateLimit-Remaining: 0
< X-RateLimit-Reset: 1470173023.123
< X-RateLimit-Reset-After: 64.57
< X-RateLimit-Bucket: abcd1234
< X-RateLimit-Scope: user
{
  "message": "You are being rate limited.",
  "retry_after": 64.57,
  "global": false
}

```

###### Example Exceeded Resource Rate Limit Response

```
< HTTP/1.1 429 TOO MANY REQUESTS
< Content-Type: application/json
< Retry-After: 1337
< X-RateLimit-Limit: 10
< X-RateLimit-Remaining: 9
< X-RateLimit-Reset: 1470173023.123
< X-RateLimit-Reset-After: 64.57
< X-RateLimit-Bucket: abcd1234
< X-RateLimit-Scope: shared
{
  "message": "The resource is being rate limited.",
  "retry_after": 1336.57,
  "global": false
}

```

###### Example Exceeded Global Rate Limit Response

```
< HTTP/1.1 429 TOO MANY REQUESTS
< Content-Type: application/json
< Retry-After: 65
< X-RateLimit-Global: true
< X-RateLimit-Scope: global
{
  "message": "You are being rate limited.",
  "retry_after": 64.57,
  "global": true
}

```

### Global Rate Limit

 Each bot can send up to 50 requests per second to the Discord API. Requests without an authorization header are limited by IP address. This global ceiling is separate from any endpoint-specific route limit. A bot with enough traffic, depending on its behavior, may be unable to remain under 50 requests per second in normal operation. Global-limit trouble often appears as repeated bans from the Discord API during bot startup. Occasional temporary Cloudflare bans are usually **not** caused by the global rate limit; they more often mean a burst of unhandled errors crossed the invalid-request threshold. If normal bot operation repeatedly triggers Cloudflare bans from the Discord API, contact support to ask whether the bot qualifies for higher global limits. Use [https://dis.gd/rate-limit](https://dis.gd/rate-limit). [Interaction endpoints](https://docs.discord.com/developers/interactions/receiving-and-responding#endpoints) do not count against the bot's Global Rate Limit.

### Invalid Request Limit aka Cloudflare bans

 Discord automatically applies temporary API restrictions to IP addresses that generate too many invalid HTTP requests. The current threshold is **10,000 per 10 minutes**. Responses with **401**, **403**, or **429** status count as invalid requests. Applications should make practical efforts to avoid them. For example:
- Avoid **401** responses by sending a valid authorization token when one is required, and stop retrying after a token is known to be invalid
- Avoid **403** responses by checking role and channel permissions before attempting requests the caller cannot perform
- Avoid **429** responses by reading the rate-limit headers above and waiting for exhausted buckets to reset before issuing more requests. *429 errors returned with `X-RateLimit-Scope: shared` are not counted against you.*
 Large applications, especially ones that might reach 10,000 requests per 10 minutes, or roughly 16 to 17 requests per second, should log and monitor invalid-request rates so they do not hit this fixed limit. Apps are also expected to handle other invalid statuses reasonably. For example, after a webhook returns **404**, do not keep calling it; repeated attempts can cause a temporary restriction.
