# Gateway (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Gateway API connections let apps use secure WebSocket connections with Discord and receive events for server or guild activity, such as channel updates or role creation. Apps also use Gateway connections in a few cases to request or update resources, such as updating voice state. For most resource operations, use the [HTTP API](https://docs.discord.com/developers/reference#http-api) instead of the Gateway API. The Gateway is Discord's real-time communication channel for clients, including apps, so it includes behavior and data that may not matter to app developers. Gateway work is complex, and [community-built libraries](https://docs.discord.com/developers/developer-tools/community-resources#libraries) can handle much of the difficult logic. If you build a custom implementation, read this documentation end to end so the relevant Gateway details are understood.

### Gateway Events

 Gateway events are [payloads](https://docs.discord.com/developers/events/gateway-events#payload-structure) transmitted over a [Gateway connection](https://docs.discord.com/developers/events/gateway#connections) in either direction between an app and Discord. Apps usually [*send* events](https://docs.discord.com/developers/events/gateway#sending-events) while establishing and managing the Gateway connection, and [*receive* events](https://docs.discord.com/developers/events/gateway#receiving-events) while observing server activity. Every Gateway event is wrapped in a [Gateway event payload](https://docs.discord.com/developers/events/gateway-events#payload-structure). The [Gateway events documentation](https://docs.discord.com/developers/events/gateway-events) lists all events and their details.

###### Example Gateway Event

```
{
  "op": 0,
  "d": {},
  "s": 42,
  "t": "GATEWAY_EVENT_NAME"
}

```

 See the [Gateway events documentation](https://docs.discord.com/developers/events/gateway-events#payload-structure) for Gateway event payload details.

### Sending Events

 When an app sends a Gateway event, such as during [performing an initial handshake](https://docs.discord.com/developers/events/gateway-events#identify) or [updating presence](https://docs.discord.com/developers/events/gateway-events#update-presence), it must send an [event payload object](https://docs.discord.com/developers/events/gateway-events#payload-structure) containing a valid opcode (`op`) and inner data object (`d`). Sent events have specific rate limits described in [Rate Limiting](https://docs.discord.com/developers/events/gateway#rate-limiting). Gateway event payloads sent over a connection:
1. Must be serialized as [plain-text JSON or binary ETF](https://docs.discord.com/developers/events/gateway#encoding-and-compression).
2. Must be no larger than 4096 bytes. If a payload exceeds 4096 bytes, Discord closes the connection with close event code [`4002`](https://docs.discord.com/developers/topics/opcodes-and-status-codes#gateway-gateway-close-event-codes).
 See [Gateway event documentation](https://docs.discord.com/developers/events/gateway-events#send-events) for every event an app can send on a connection.

### Receiving Events

 Receiving Gateway events from Discord, such as [a reaction is added to a message](https://docs.discord.com/developers/events/gateway-events#message-reaction-add), is more common and somewhat more involved than sending events. Some events are delivered automatically, but most require the app to specify intents during [Identifying](https://docs.discord.com/developers/events/gateway#identifying). Intents are bitwise flags; combine them with OR (`|`) to choose which events, or event groups, Discord sends to the app. The [intents section](https://docs.discord.com/developers/events/gateway#gateway-intents) maps intents to events. Apps can also configure how events are delivered, including [encoding and compression](https://docs.discord.com/developers/events/gateway#encoding-and-compression) and whether [sharding should be enabled](https://docs.discord.com/developers/events/gateway#sharding)). All receivable connection events are listed in the [Gateway event documentation](https://docs.discord.com/developers/events/gateway-events#receive-events).

#### Dispatch Events

 [Dispatch (opcode `0`)](https://docs.discord.com/developers/topics/opcodes-and-status-codes#gateway-gateway-opcodes) is the event type apps receive most often. Most Gateway events representing guild actions arrive as Dispatch events. When parsing one:
- Use the `t` field to identify which [Gateway event](https://docs.discord.com/developers/events/gateway-events#receive-events) the payload contains and what data should appear in `d`.
- The `s` field is the event sequence number, representing relative event order. Cache the latest non-null `s` value for heartbeats and for [Resuming](https://docs.discord.com/developers/events/gateway#resuming) a connection.

### Connections

 Gateway connections are long-lived WebSockets and are more complex than HTTP requests or interaction responses such as Slash Commands. Apps using the Gateway must open the initial connection, keep it alive, and respond correctly to disconnects.

### Connection Lifecycle

 The overview below omits some nuances; the sections that follow describe each step and event in detail. The Gateway connection sequence is summarized here: (Figure: Gateway connection lifecycle overview)
1. The app fetches and caches a WSS URL with [Get Gateway](https://docs.discord.com/developers/events/gateway#get-gateway) or [Get Gateway Bot](https://docs.discord.com/developers/events/gateway#get-gateway-bot), then opens a Gateway connection.
2. Discord sends [Hello (opcode `10`)](https://docs.discord.com/developers/events/gateway#hello-event), which includes the heartbeat interval in milliseconds. **Read the section on [Connecting](https://docs.discord.com/developers/events/gateway#connecting)**
3. Begin the Heartbeat interval. The app sends a [Heartbeat (opcode `1`)](https://docs.discord.com/developers/events/gateway-events#heartbeat) event and continues sending heartbeats at each interval until the connection closes. **Read the section on [Sending Heartbeats](https://docs.discord.com/developers/events/gateway#sending-heartbeats)**
  - Discord replies to each Heartbeat with [Heartbeat ACK (opcode `11`)](https://docs.discord.com/developers/events/gateway#sending-heartbeats). If no Heartbeat ACK arrives, the app should close the connection, reconnect, and continue appropriately.
  - Discord can also send the app a [Heartbeat (opcode `1`)](https://docs.discord.com/developers/events/gateway-events#heartbeat) event; the app should answer with a Heartbeat immediately.

4. The app sends [Identify (opcode `2`)](https://docs.discord.com/developers/events/gateway-events#identify) to complete the initial Gateway handshake. **Read the section on [Identifying](https://docs.discord.com/developers/events/gateway#identifying)**
5. Discord sends [Ready (opcode `0`)](https://docs.discord.com/developers/events/gateway-events#ready), confirming that the handshake succeeded and the connection is established. Ready includes `resume_gateway_url`, which the app should store as the WebSocket URL for Resume. **Read the section on [the Ready event](https://docs.discord.com/developers/events/gateway#ready-event)**
6. The connection can drop at any time for many reasons. Whether the app can [Resume](https://docs.discord.com/developers/events/gateway#resuming) or must identify again depends on factors such as the received [opcode](https://docs.discord.com/developers/topics/opcodes-and-status-codes#gateway-gateway-opcodes) and [close code](https://docs.discord.com/developers/topics/opcodes-and-status-codes#gateway-gateway-close-event-codes). **Read the section on [Disconnecting](https://docs.discord.com/developers/events/gateway#disconnecting)**
7. If the app **can** resume or reconnect, it opens a new connection using `resume_gateway_url` with matching version and encoding, then sends [Resume (opcode `6`)](https://docs.discord.com/developers/events/gateway-events#resume). If it **cannot** resume or reconnect, it opens a new connection from the cached URL in step #1 and repeats the Gateway cycle. *Yipee!* **Read the section on [Resuming](https://docs.discord.com/developers/events/gateway#resuming)**

### Connecting

 Before opening a Gateway connection, the app should call [Get Gateway](https://docs.discord.com/developers/events/gateway#get-gateway) or [Get Gateway Bot](https://docs.discord.com/developers/events/gateway#get-gateway-bot). Either endpoint returns a payload with a `url` field containing the WSS URL for the WebSocket connection. [Get Gateway Bot](https://docs.discord.com/developers/events/gateway#get-gateway-bot) also returns recommended shard count and session start limit information for the app. On the first call to either [Get Gateway](https://docs.discord.com/developers/events/gateway#get-gateway) or [Get Gateway Bot](https://docs.discord.com/developers/events/gateway#get-gateway-bot), cache `url` and reuse it for later Gateway reconnects. When connecting, explicitly pass API version and [encoding](https://docs.discord.com/developers/events/gateway#encoding-and-compression) as query parameters. You may also include whether Discord should [compress](https://docs.discord.com/developers/events/gateway#encoding-and-compression) data sent to the app. `wss://gateway.discord.gg/?v=10&encoding=json` is an example WSS URL for a Gateway connection.

###### Gateway URL Query String Params

| Field | Type | Description | Accepted Values |
| --- | --- | --- | --- |
| v | integer | API Version to use | API version |
| encoding | string | Encoding used for received gateway packets | `json` or `etf` |
| compress? | string | Optional transport compression for gateway packets | `zlib-stream` or `zstd-stream` |

#### Hello Event

 After connecting to the Gateway, the app receives [Hello (opcode `10`)](https://docs.discord.com/developers/events/gateway#hello-event) with the connection's heartbeat interval (`heartbeat_interval`). This interval is the number of milliseconds used to schedule Heartbeat events that keep the connection active. Heartbeat behavior is described in [Sending Heartbeats](https://docs.discord.com/developers/events/gateway#sending-heartbeats).

###### Example Hello Event

```
{
  "op": 10,
  "d": {
    "heartbeat_interval": 45000
  }
}

```

### Sending Heartbeats

 Heartbeats are pings that indicate to Discord that the app is still using a Gateway connection. After the connection opens, the app should send heartbeats in a background process as described below until the Gateway connection closes.

#### Heartbeat Interval

 When the app opens a Gateway connection, [Hello (opcode `10`)](https://docs.discord.com/developers/events/gateway#hello-event) provides `heartbeat_interval`, a millisecond duration. After Hello, wait `heartbeat_interval * jitter`, where `jitter` is a random value from 0 to 1, then send the first [Heartbeat (opcode `1`)](https://docs.discord.com/developers/events/gateway-events#heartbeat). Continue sending a heartbeat every `heartbeat_interval` milliseconds until the connection closes. If the app misses the heartbeat deadline, Discord closes the connection and the app must [Resume](https://docs.discord.com/developers/events/gateway#resuming). Each heartbeat must include the latest sequence number received in the `d` field. That sequence number appears in `s` on the [event payload](https://docs.discord.com/developers/events/gateway-events#payload-structure). If no event has been received, send `null` in `d`. The first heartbeat's `jitter` spreads clients over the interval from 0 to `heartbeat_interval` to avoid mass reconnect traffic. Heartbeats may be sent before `heartbeat_interval` elapses, but avoid doing so unless required. `heartbeat_interval` already includes tolerance for network latency, so no extra latency adjustment is needed. Discord answers each Heartbeat with [Heartbeat ACK (opcode `11`)](https://docs.discord.com/developers/events/gateway#heartbeat-interval-example-heartbeat-ack):

###### Example Heartbeat ACK

```
{
  "op": 11
}

```

 During a service outage where the app remains connected to the Gateway, keep sending heartbeats and receiving heartbeat ACKs. The Gateway eventually responds and issues a session when able. If a client sends a heartbeat and does not receive an ACK before the next send attempt, the connection may have failed or become "zombied." The client should immediately close the connection with any close code except `1000` or `1001`, then reconnect and try to [Resume](https://docs.discord.com/developers/events/gateway#resuming).

#### Heartbeat Requests

 Discord may request extra heartbeats by sending [Heartbeat (opcode `1`)](https://docs.discord.com/developers/events/gateway-events#heartbeat) in addition to the normal interval. When that event arrives, send a Heartbeat immediately rather than waiting for the current interval to finish. Discord responds with [Heartbeat ACK (opcode `11`)](https://docs.discord.com/developers/events/gateway#heartbeat-interval-example-heartbeat-ack), just as it does for interval heartbeats.

### Identifying

 Once the connection is open and heartbeats are running, send [Identify (opcode `2`)](https://docs.discord.com/developers/events/gateway-events#identify). Identify is the required initial Gateway handshake before most Gateway events can be sent or received. Apps are constrained by maximum concurrency, shown as `max_concurrency` in the [session start limit object](https://docs.discord.com/developers/events/gateway#session-start-limit-object), when identifying. Exceeding the limit causes Discord to return [Invalid Session (opcode `9`)](https://docs.discord.com/developers/events/gateway-events#invalid-session). After a valid Identify payload, Discord sends [Ready](https://docs.discord.com/developers/events/gateway-events#ready), indicating a successfully connected Gateway state. Ready is delivered as [Dispatch (opcode `0`)](https://docs.discord.com/developers/topics/opcodes-and-status-codes#gateway-gateway-opcodes). Clients have a global limit of 1000 `IDENTIFY` calls to the websocket per 24 hours across all shards; `RESUME` calls are excluded. Reaching that limit terminates all active app sessions, resets the bot token, and emails the owner. The owner must then update the application with the new token.

###### Example Identify Payload

 The following is a minimal `IDENTIFY` payload. `IDENTIFY` can include additional session properties such as payload compression and initial presence. See [Identify Structure](https://docs.discord.com/developers/events/gateway-events#identify-identify-structure) for the full event shape.

```
{
  "op": 2,
  "d": {
    "token": "my_token",
    "intents": 513,
    "properties": {
      "os": "linux",
      "browser": "my_library",
      "device": "my_library"
    }
  }
}

```

#### Ready event

 After a valid Identify payload, Discord sends [Ready](https://docs.discord.com/developers/events/gateway-events#ready) to the app. Ready includes state such as the guilds the app belongs to, which lets the app begin interacting with the platform. It also includes fields that must be cached for later [Resume](https://docs.discord.com/developers/events/gateway#resuming) attempts. Two fields are especially important:
- `resume_gateway_url` is the WebSocket URL to use for Resume after a disconnect. Use `resume_gateway_url` instead of the URL [used when connecting](https://docs.discord.com/developers/events/gateway#connecting).
- `session_id` identifies the Gateway session for the new connection. It is needed to know which event stream was tied to the disconnected connection.
 The [Gateway events documentation](https://docs.discord.com/developers/events/gateway-events) contains the complete Ready event details.

### Disconnecting

 Gateway disconnects can occur for many reasons and can be started either by Discord or by the app.

#### Handling a Disconnect

 Because of Discord's architecture, disconnects are normal enough that apps should expect and handle them. A disconnect typically includes a [close code](https://docs.discord.com/developers/topics/opcodes-and-status-codes#gateway-gateway-close-event-codes), which helps decide whether to reconnect and [Resume](https://docs.discord.com/developers/events/gateway#resuming) or start over with a new Identify. After making that decision, choose one of these paths:
- If the app *can* reconnect and resume the previous session, reconnect with `resume_gateway_url` and `session_id` from the [Ready](https://docs.discord.com/developers/events/gateway-events#ready) event. See [Resuming](https://docs.discord.com/developers/events/gateway#resuming) for when and how to do this.
- If the app *cannot* reconnect **or the reconnect fails**, open a new connection with the URL from the initial [Get Gateway](https://docs.discord.com/developers/events/gateway#get-gateway) or [Get Gateway Bot](https://docs.discord.com/developers/events/gateway#get-gateway-bot) call. When resume is not possible, the app must identify again after opening the new connection.
 The full set of close codes is listed in [Response Codes](https://docs.discord.com/developers/topics/opcodes-and-status-codes#gateway-gateway-close-event-codes).

#### Initiating a Disconnect

 Closing the gateway connection with close code `1000` or `1001` invalidates the session and makes the bot appear offline. If the app only closes the TCP connection or uses another close code, the session remains active and times out after a few minutes. That can help when [Resuming](https://docs.discord.com/developers/events/gateway#resuming) the previous session.

### Resuming

 When an app disconnects, Discord provides a reconnect and resume process that can replay lost events from the last received sequence number. After Resume succeeds, the app receives missed events as if the connection had stayed open. Unlike a new connection, Resume does **not** require Identify again. Apps should try to resume in these cases:
1. It receives a [Reconnect (opcode `7`)](https://docs.discord.com/developers/events/gateway-events#reconnect) event
2. It disconnects with a close code that permits reconnecting. The [Opcodes and Status Codes](https://docs.discord.com/developers/topics/opcodes-and-status-codes#gateway-gateway-close-event-codes) documentation lists close codes.
3. It disconnects without receiving *any* close code.
4. It receives [Invalid Session (opcode `9`)](https://docs.discord.com/developers/events/gateway-events#invalid-session) with `d` set to `true`. This is uncommon but possible.

#### Preparing to Resume

 To send [Resume (opcode `6`)](https://docs.discord.com/developers/events/gateway-events#resume), the app needs three values: `session_id`, `resume_gateway_url` from [Ready](https://docs.discord.com/developers/events/gateway#ready-event), and the sequence number (`s`) from the most recent Dispatch (opcode `0`) event received before disconnect. After the old connection closes, open a new connection using `resume_gateway_url`, not the original connection URL, while keeping the same query parameters from the initial [Connection](https://docs.discord.com/developers/events/gateway#connecting). Apps that ignore `resume_gateway_url` see more disconnects than normal. After the new connection opens, send [Gateway Resume](https://docs.discord.com/developers/events/gateway-events#resume) with `session_id` and the last sequence number. The `session_id` field name stays the same, while the last sequence number is sent as `seq` inside the data object (`d`). Do not send Identify after opening a Resume connection. If Resume works, the Gateway replays missed events in order and ends with [Resumed](https://docs.discord.com/developers/events/gateway-events#resumed), meaning replay is complete and later events are new. When using `resume_gateway_url`, supply the same version and encoding as the first connection. If the app does not reconnect quickly enough to Resume, it may receive [Invalid Session (opcode `9`)](https://docs.discord.com/developers/events/gateway-events#invalid-session). If `d` is `false`, which is the usual case, the app should disconnect. Then it should open a new connection with the cached URL from [Get Gateway](https://docs.discord.com/developers/events/gateway#get-gateway) or [Get Gateway Bot](https://docs.discord.com/developers/events/gateway#get-gateway-bot), and send [Identify (opcode `2`)](https://docs.discord.com/developers/events/gateway-events#identify).

###### Example Gateway Resume Event

```
{
  "op": 6,
  "d": {
    "token": "my_token",
    "session_id": "session_id_i_stored",
    "seq": 1337
  }
}

```

### Gateway Intents
