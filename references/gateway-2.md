# Gateway (part 2)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Stateful apps can become costly to maintain when a Gateway connection delivers large volumes of data, especially at scale. Gateway intents reduce that workload. Intents are bitwise values supplied in the `intents` parameter while [Identifying](https://docs.discord.com/developers/events/gateway#identifying), and each maps to a related group of events. For example, the event sent when a guild is created (`GUILD_CREATE`) and the event sent when a channel is updated (`CHANNEL_UPDATE`) both require the `GUILDS (1 << 0)` intent listed below. If an intent is omitted during identify, the app receives none of the Gateway events tied to that intent. Intents are optional on the v6 gateway and required starting with v8. There are two intent categories:
- **Standard intents** can be supplied without extra permissions or configuration.
- [**Privileged intents**](https://docs.discord.com/developers/events/gateway#privileged-intents) must be enabled for the app in the Developer Portal before they are passed. Verified apps, required for apps in 100+ guilds, need approval for the intent after verification before use. The [section below](https://docs.discord.com/developers/events/gateway#privileged-intents) explains privileged intents in more detail.
 Discord closes the app connection if it sends invalid intents ([`4013` close code](https://docs.discord.com/developers/topics/opcodes-and-status-codes#gateway-gateway-close-event-codes)) or sends a privileged intent that has not been configured or approved for the app ([`4014` close code](https://docs.discord.com/developers/topics/opcodes-and-status-codes#gateway-gateway-close-event-codes)).

### List of Intents

 The list below shows every intent and the Gateway events tied to it. Events not listed are not associated with an intent and are always sent to the app. The [Gateway events](https://docs.discord.com/developers/events/gateway-events) documentation lists all events, including those without an intent.

```
GUILDS (1 << 0)
  - GUILD_CREATE
  - GUILD_UPDATE
  - GUILD_DELETE
  - GUILD_ROLE_CREATE
  - GUILD_ROLE_UPDATE
  - GUILD_ROLE_DELETE
  - CHANNEL_CREATE
  - CHANNEL_UPDATE
  - CHANNEL_DELETE
  - CHANNEL_PINS_UPDATE
  - THREAD_CREATE
  - THREAD_UPDATE
  - THREAD_DELETE
  - THREAD_LIST_SYNC
  - THREAD_MEMBER_UPDATE
  - THREAD_MEMBERS_UPDATE *
  - STAGE_INSTANCE_CREATE
  - STAGE_INSTANCE_UPDATE
  - STAGE_INSTANCE_DELETE
  - VOICE_CHANNEL_STATUS_UPDATE
  - VOICE_CHANNEL_START_TIME_UPDATE

GUILD_MEMBERS (1 << 1) **
  - GUILD_MEMBER_ADD
  - GUILD_MEMBER_UPDATE
  - GUILD_MEMBER_REMOVE
  - THREAD_MEMBERS_UPDATE *

GUILD_MODERATION (1 << 2)
  - GUILD_AUDIT_LOG_ENTRY_CREATE
  - GUILD_BAN_ADD
  - GUILD_BAN_REMOVE

GUILD_EXPRESSIONS (1 << 3)
  - GUILD_EMOJIS_UPDATE
  - GUILD_STICKERS_UPDATE
  - GUILD_SOUNDBOARD_SOUND_CREATE
  - GUILD_SOUNDBOARD_SOUND_UPDATE
  - GUILD_SOUNDBOARD_SOUND_DELETE
  - GUILD_SOUNDBOARD_SOUNDS_UPDATE

GUILD_INTEGRATIONS (1 << 4)
  - GUILD_INTEGRATIONS_UPDATE
  - INTEGRATION_CREATE
  - INTEGRATION_UPDATE
  - INTEGRATION_DELETE

GUILD_WEBHOOKS (1 << 5)
  - WEBHOOKS_UPDATE

GUILD_INVITES (1 << 6)
  - INVITE_CREATE
  - INVITE_DELETE

GUILD_VOICE_STATES (1 << 7)
  - VOICE_CHANNEL_EFFECT_SEND
  - VOICE_STATE_UPDATE

GUILD_PRESENCES (1 << 8) **
  - PRESENCE_UPDATE

GUILD_MESSAGES (1 << 9)
  - MESSAGE_CREATE
  - MESSAGE_UPDATE
  - MESSAGE_DELETE
  - MESSAGE_DELETE_BULK

GUILD_MESSAGE_REACTIONS (1 << 10)
  - MESSAGE_REACTION_ADD
  - MESSAGE_REACTION_REMOVE
  - MESSAGE_REACTION_REMOVE_ALL
  - MESSAGE_REACTION_REMOVE_EMOJI

GUILD_MESSAGE_TYPING (1 << 11)
  - TYPING_START

DIRECT_MESSAGES (1 << 12)
  - MESSAGE_CREATE
  - MESSAGE_UPDATE
  - MESSAGE_DELETE
  - CHANNEL_PINS_UPDATE

DIRECT_MESSAGE_REACTIONS (1 << 13)
  - MESSAGE_REACTION_ADD
  - MESSAGE_REACTION_REMOVE
  - MESSAGE_REACTION_REMOVE_ALL
  - MESSAGE_REACTION_REMOVE_EMOJI

DIRECT_MESSAGE_TYPING (1 << 14)
  - TYPING_START

MESSAGE_CONTENT (1 << 15) ***

GUILD_SCHEDULED_EVENTS (1 << 16)
  - GUILD_SCHEDULED_EVENT_CREATE
  - GUILD_SCHEDULED_EVENT_UPDATE
  - GUILD_SCHEDULED_EVENT_DELETE
  - GUILD_SCHEDULED_EVENT_USER_ADD
  - GUILD_SCHEDULED_EVENT_USER_REMOVE

AUTO_MODERATION_CONFIGURATION (1 << 20)
  - AUTO_MODERATION_RULE_CREATE
  - AUTO_MODERATION_RULE_UPDATE
  - AUTO_MODERATION_RULE_DELETE

AUTO_MODERATION_EXECUTION (1 << 21)
  - AUTO_MODERATION_ACTION_EXECUTION

GUILD_MESSAGE_POLLS (1 << 24)
  - MESSAGE_POLL_VOTE_ADD
  - MESSAGE_POLL_VOTE_REMOVE

DIRECT_MESSAGE_POLLS (1 << 25)
  - MESSAGE_POLL_VOTE_ADD
  - MESSAGE_POLL_VOTE_REMOVE

```

 * [Thread Members Update](https://docs.discord.com/developers/events/gateway-events#thread-members-update) returns different data depending on the intents in use. ** Events under the `GUILD_PRESENCES` and `GUILD_MEMBERS` intents are **off by default on all API versions**. With **API v6**, you receive those events when authorized and when the intents are enabled in the Developer Portal; you do not need to send intents on API v6 for these events, only enable the flags. With **API v8** or above, intents are required and must be included when identifying. *** `MESSAGE_CONTENT` is not tied to individual events. It controls whether message-content fields are present on events that can include them. See the [message content intent](https://docs.discord.com/developers/events/gateway#message-content-intent) section for details.

### Caveats

 [Guild Member Update](https://docs.discord.com/developers/events/gateway-events#guild-member-update) is delivered for updates to the current user even if `GUILD_MEMBERS` is not set. [Guild Create](https://docs.discord.com/developers/events/gateway-events#guild-create) and [Request Guild Members](https://docs.discord.com/developers/events/gateway-events#request-guild-members) have special intent behavior; read those sections for details. [Thread Members Update](https://docs.discord.com/developers/events/gateway-events#thread-members-update) only includes current-user thread adds and removals by default. To get updates for other users, request the `GUILD_MEMBERS` [Gateway Intent](https://docs.discord.com/developers/events/gateway#gateway-intents).

### Privileged Intents

 Some intents have extra access requirements because of the data they expose. Those intents currently include:
- `GUILD_PRESENCES`
- `GUILD_MEMBERS`
- [`MESSAGE_CONTENT`](https://docs.discord.com/developers/events/gateway#message-content-intent)
 Apps that meet verification requirements **must** be approved for the privileged intent(s) before use. After verification, privileged intents can be requested from the app settings in the Developer Portal. Before adding any privileged intent to the `IDENTIFY` payload, enable the exact privileged intents the app needs in the Developer Portal. Only enable a privileged intent when the use case requires it and complies with the Developer Policy. The [You Might Not Need a Privileged Intent](https://docs.discord.com/developers/gateway/you-might-not-need-a-privileged-intent) guide explains what data these intents provide and what alternatives may work.

#### Enabling Privileged Intents

 Privileged intents must be enabled in the app settings before use. In the Developer Portal, open the app settings, then use the [**Bot** page](https://discord.com/developers/applications/select/bot) and its **Privileged Gateway Intents** section to toggle the needed intents. Only turn on privileged intents required for the bot to function.

#### Privileged Intent Access Review

 Apps with fewer than 10,000 users can use privileged intents by enabling them in the Developer Portal. When an app exceeds 10,000 unique users who can see it across all servers where it is installed, continued Privileged Intents access requires review. At that threshold, the app or team owner receives a system DM and/or email, and the Developer Portal shows a callout on the app. You must submit information about the app and how it uses Privileged Intents. If access is approved, the relevant intents can be enabled in the Developer Portal, and Discord notifies you annually to reapply. The [Getting Started with Privileged Intent Review](https://docs.discord.com/developers/gateway/getting-started-with-privileged-intent-review) guide covers the threshold, requirements, and review process.

#### Gateway Restrictions

 Privileged intents control which Gateway events the app may receive. With **API v8** and above, every intent, privileged or not, must be listed in the `intents` parameter when Identifying. If the app passes a privileged intent in `intents` without configuring it in the app settings or receiving verification approval, Discord closes the Gateway connection with a ([`4014` close code](https://docs.discord.com/developers/topics/opcodes-and-status-codes#gateway-gateway-close-event-codes)). With **API v6**, the app receives events for privileged intents it has configured and is authorized for *without* passing those intents in `intents` while Identifying. Events under `GUILD_PRESENCES` and `GUILD_MEMBERS` are disabled by default on every API version.

#### HTTP Restrictions

 Privileged intents can also limit which [HTTP API](https://docs.discord.com/developers/reference#http-api) endpoints the app may call and what data it receives. For example, using [List Guild Members](https://docs.discord.com/developers/resources/guild#list-guild-members) requires enabling `GUILD_MEMBERS`, with approval if the app is eligible for verification. HTTP API restrictions are separate from Gateway restrictions and do not depend on which intents the app sends in `intents` when Identifying.

#### Message Content Intent

 `MESSAGE_CONTENT (1 << 15)` is a special privileged intent that is not directly mapped to Gateway events. Instead, `MESSAGE_CONTENT` permits the app to receive message-content data across APIs. Documentation notes any fields affected by the message content intent. For example, the `content`, `embeds`, `attachments`, `components`, and `poll` fields in [message objects](https://docs.discord.com/developers/resources/message#message-object) all contain message content and need the intent. Like other privileged intents, `MESSAGE_CONTENT` requires approval. After verification, apply for the intent from the app settings in the Developer Portal. The [Help Center](https://support-dev.discord.com/hc/en-us/articles/5324827539479) explains the review policy. Apps **without** the intent receive empty values for fields containing user-entered content except for:
- Content in messages sent by the app
- Content in DMs with the app
- Content where the app is [mentioned](https://docs.discord.com/developers/reference#message-formatting-formats)
- Content of the message targeted by a [message context menu command](https://docs.discord.com/developers/interactions/application-commands#message-commands)

### Rate Limiting

 This section covers Gateway rate limits, not [HTTP API rate limits](https://docs.discord.com/developers/topics/rate-limits). Apps may send 120 [gateway events](https://docs.discord.com/developers/events/gateway-events) per [connection](https://docs.discord.com/developers/events/gateway#connections) every 60 seconds, which averages 2 commands per second. Exceeding the limit disconnects the app from the Gateway immediately. As with other rate limits, repeated violations can revoke API access. Apps also have a cap on [concurrent](https://docs.discord.com/developers/events/gateway#session-start-limit-object) [Identify](https://docs.discord.com/developers/events/gateway#identifying) requests per 5 seconds. If that limit is hit, the Gateway returns an [Invalid Session (opcode `9`)](https://docs.discord.com/developers/events/gateway-events#invalid-session).

### Encoding and Compression

 When [connecting](https://docs.discord.com/developers/events/gateway#connecting) to the Gateway, apps choose an `encoding` parameter for either plain-text JSON or binary [ETF](https://erlang.org/doc/apps/erts/erl_ext_dist.html). Use the encoding you prefer, but each has tradeoffs; JSON is generally recommended when you are unsure. Apps may enable compression to receive zlib-compressed or zstd-compressed packets. [Payload compression](https://docs.discord.com/developers/events/gateway#payload-compression) works only with JSON encoding, while [transport compression](https://docs.discord.com/developers/events/gateway#transport-compression) can be used with any encoding.

### Using JSON Encoding

 Plain-text JSON encoding allows apps to enable [payload compression](https://docs.discord.com/developers/events/gateway#payload-compression).

#### Payload Compression

 An app using payload compression cannot use [transport compression](https://docs.discord.com/developers/events/gateway#transport-compression). Payload compression optionally compresses some event packets sent by Discord over the connection. Compressed payloads use the zlib format described in [RFC1950 2.2](https://tools.ietf.org/html/rfc1950#section-2.2). To enable it, set `compress` to `true` in an [Identify (opcode `2`)](https://docs.discord.com/developers/events/gateway-events#identify) event. Not every payload is compressed after enabling this setting. The app or library *must* detect compressed payloads and decompress them to plain-text JSON before parsing. Payload compression does not use a shared compression context between events. It is disabled when [transport compression](https://docs.discord.com/developers/events/gateway#transport-compression) is used.

### Using ETF Encoding

 ETF (External Term Format) encoding has several specific behaviors:
- Snowflake IDs are sent as 64-bit integers or strings.
- The app cannot send compressed messages to the server.
- Outbound payloads must use string keys. Atom keys cause a [`4002`](https://docs.discord.com/developers/topics/opcodes-and-status-codes#gateway-gateway-close-event-codes) decode error.
 See [erlpack](https://github.com/discord/erlpack) for an ETF implementation example.

### Transport Compression

 Transport compression optionally compresses every packet that Discord sends over the connection. The available transport compression options are `zlib-stream` and `zstd-stream`.

#### zlib-stream

 With zlib transport compression, received data for a single Gateway connection must be processed through a shared zlib context. Each Gateway connection should still have its own unique zlib context. For transport-compressed data, append received data to a buffer until the 4-byte `Z_SYNC_FLUSH` suffix (`00 00 ff ff`) arrives. After the `Z_SYNC_FLUSH` marker is present, decompress the buffer.

###### Transport Compression Example

```
# Z_SYNC_FLUSH suffix
ZLIB_SUFFIX = b'\x00\x00\xff\xff'
# initialize a buffer to store chunks
buffer = bytearray()
# create a shared zlib inflation context to run chunks through
inflator = zlib.decompressobj()

# ...
def on_websocket_message(msg):
  # always push the message data to your cache
  buffer.extend(msg)

  # check if the last four bytes are equal to ZLIB_SUFFIX
  if len(msg) < 4 or msg[-4:] != ZLIB_SUFFIX:
    return

  # if the message *does* end with ZLIB_SUFFIX,
  # get the full message by decompressing the buffers
  # NOTE: the message is utf-8 encoded.
  msg = inflator.decompress(buffer)
  buffer = bytearray()

  # here you can treat `msg` as either JSON or ETF encoded,
  # depending on your `encoding` param

```

#### zstd-stream

 With zstd-stream transport compression, all received data must pass through a zstd decompression context that lives for the entire Gateway connection. Each websocket message maps to one Gateway message, but it does not close a zstd frame. Call ZSTD_decompressStream repeatedly until the frame data is processed; ZSTD_decompressStream does not always return 0. This [Erlang](https://github.com/silviucpp/ezstd/blob/f3f33b2f6b917f7e8aaa2b4d71338620537df81b/src/ezstd.erl#L151-L169) + [C++](https://github.com/silviucpp/ezstd/blob/f3f33b2f6b917f7e8aaa2b4d71338620537df81b/c_src/ezstd_nif.cc#L520-L568) implementation is a useful example.

### Tracking State

 Most client state arrives in the initial [Ready](https://docs.discord.com/developers/events/gateway#ready-event) event and later [Guild Create](https://docs.discord.com/developers/events/gateway-events#guild-create) events. As resources are created, changed, and removed, Gateway events notify the app and include relevant data. Cache the resource state the app needs so it can avoid excessive API calls, then update cached state as events arrive. Large apps can accumulate a very large client state, so keep only data required for operation. Some member data, such as roles or permissions, may not need separate caching because events like [MESSAGE_CREATE](https://docs.discord.com/developers/events/gateway-events#message-create) include the full member object. For a member-status tracker, the app receives guild member online status during the initial Gateway connection, including online, idle, dnd, or offline. It then parses [Presence Update](https://docs.discord.com/developers/events/gateway-events#presence-update) events and updates cached member objects as those updates arrive.

### Guild Availability

 When a bot user connects to the gateway, guilds the bot belongs to initially appear unavailable. The gateway automatically tries to reconnect for you. As guilds become available, the app receives [Guild Create](https://docs.discord.com/developers/events/gateway-events#guild-create) events.

### Sharding

 As apps grow into more guilds, developers may need to spread app operations across multiple processes. The Gateway supports user-controlled guild sharding, letting apps divide events across multiple Gateway connections. The app fully controls guild sharding, and separate connections do not need shared state to operate. Any app can enable sharding, but smaller apps usually do not need it. Each shard supports a maximum of 2500 guilds, and apps in 2500+ guilds *must* shard. To enable sharding on a connection, send the `shard` array in the [Identify](https://docs.discord.com/developers/events/gateway-events#identify) payload. The first array item is the zero-based integer for the current shard, and the second item is the total shard count. The [Get Gateway Bot](https://docs.discord.com/developers/events/gateway#get-gateway-bot) endpoint returns Discord's recommended shard count in the `shards` field. Use this formula to determine which shard receives each event:

###### Sharding Formula

```
shard_id = (guild_id >> 22) % num_shards

```

 For example, splitting the connection across three shards uses these `shard` values: `[0, 3]`, `[1, 3]`, and `[2, 3]`. Gateway events without a `guild_id` go only to the first shard (`shard_id: 0`), including Direct Message (DM), subscription, and entitlement events. `num_shards` does not describe or limit the total possible sessions; it is only for *routing* traffic. Sessions therefore do not need to be identified with an even distribution while sharding. You can open multiple sessions with the same `[shard_id, num_shards]`, or sessions using different `num_shards` values. That flexibility lets operators rebalance load or perform zero-downtime updates by preparing another session deployment with a different `num_shards` value before moving traffic to it.

###### Max Concurrency

 When multiple shards are used, you can start them concurrently according to the [`max_concurrency`](https://docs.discord.com/developers/events/gateway#session-start-limit-object) value returned at session start. The shards that may start together are assigned by a key for each shard. Compute the rate limit key for a shard with

```
rate_limit_key = shard_id % max_concurrency

```

 This places shards into buckets sized by `max_concurrency`. On bot startup, you can start up to `max_concurrency` shards at once, and the buckets must be started **in order**. For example, with 16 shards and `max_concurrency` set to 16:

```
shard_id: 0, rate limit key (0 % 16): 0
shard_id: 1, rate limit key (1 % 16): 1
shard_id: 2, rate limit key (2 % 16): 2
shard_id: 3, rate limit key (3 % 16): 3
shard_id: 4, rate limit key (4 % 16): 4
shard_id: 5, rate limit key (5 % 16): 5
shard_id: 6, rate limit key (6 % 16): 6
shard_id: 7, rate limit key (7 % 16): 7
shard_id: 8, rate limit key (8 % 16): 8
shard_id: 9, rate limit key (9 % 16): 9
shard_id: 10, rate limit key (10 % 16): 10
shard_id: 11, rate limit key (11 % 16): 11
shard_id: 12, rate limit key (12 % 16): 12
shard_id: 13, rate limit key (13 % 16): 13
shard_id: 14, rate limit key (14 % 16): 14
shard_id: 15, rate limit key (15 % 16): 15

```

 All 16 shards can start simultaneously because each `rate_limit_key` fills the bucket of 16 shards. Now consider 32 shards:
