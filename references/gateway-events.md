# Gateway Events (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Gateway connections use WebSockets; each WebSocket supports bidirectional communication, so either endpoint may send events. The event set falls into two categories:
- **Send events** are Gateway events an app transmits to Discord, such as identifying with the Gateway.
- **Receive events** are Gateway events Discord delivers to an app. They usually describe activity in a server where the app is installed, such as a channel update.
 Every Gateway event is wrapped in a [Gateway event payload](https://docs.discord.com/developers/events/gateway-events#payload-structure). See the [Gateway documentation](https://docs.discord.com/developers/events/gateway) for details on working with the Gateway. Fields not documented for Gateway events should be treated as unsupported for apps, and their format or data can change without notice.

### Event Names

 Event names use UPPER-CASE words separated with under_scores. For example, [Channel Create](https://docs.discord.com/developers/events/gateway-events#channel-create) maps to `CHANNEL_CREATE`, and [Voice State Update](https://docs.discord.com/developers/events/gateway-events#voice-state-update) maps to `VOICE_STATE_UPDATE`. The documentation generally displays event names in Title Case for readability.

### Payload Structure

 Gateway event payloads share one outer format, while the event-specific data in `d` changes by event type.
| Field | Type | Description |
| --- | --- | --- |
| op | integer | Gateway opcode that identifies the payload type |
| d | ?mixed (any JSON value) | Event data |
| s | ?integer * | Event sequence number used when resuming sessions and sending heartbeats |
| t | ?string * | Event name |

 * `s` and `t` are `null` unless `op` is `0` ([Gateway Dispatch opcode](https://docs.discord.com/developers/topics/opcodes-and-status-codes#gateway-gateway-opcodes)).

###### Example Gateway Event Payload

```
{
  "op": 0,
  "d": {},
  "s": 42,
  "t": "GATEWAY_EVENT_NAME"
}

```

### Send Events

 Send events are Gateway events contained in an [event payload](https://docs.discord.com/developers/events/gateway-events#payload-structure) and delivered from an app to Discord over a Gateway connection. Older material called Gateway send events commands.
| Name | Description |
| --- | --- |
| Identify | Starts the initial gateway handshake |
| Resume | Resumes a dropped gateway connection |
| Heartbeat | Maintains an active gateway connection |
| Request Guild Members | Requests members for a guild |
| Request Soundboard Sounds | Fetches soundboard sounds for selected guilds |
| Request Channel Info | Fetches temporary channel data for channels in a guild |
| Update Voice State | Connects, relocates, or disconnects the app from a voice channel |
| Update Presence | Updates an app’s presence |

#### Identify

 Starts the initial gateway handshake. The [Gateway documentation](https://docs.discord.com/developers/events/gateway#identifying) covers the identifying process.

###### Identify Structure

| Field | Type | Description | Default |
| --- | --- | --- | --- |
| token | string | Authentication token | - |
| properties | object | Connection properties | - |
| compress? | boolean | Whether packet compression is supported by this connection | false |
| large_threshold? | integer | Value between 50 and 250; the gateway stops including offline members in the guild member list once the total member count reaches this value | 50 |
| shard? | array of two integers (shard_id, num_shards) | Used for Guild Sharding | - |
| presence? | update presence object | Presence structure containing the starting presence data | - |
| intents | integer | Gateway Intents requested for delivery | - |

###### Identify Connection Properties

| Field | Type | Description |
| --- | --- | --- |
| os | string | Your operating system |
| browser | string | Your library name |
| device | string | Your library name |

 These fields once used prefixed names, such as `prefixed(i.e:‘browser`, but [this syntax is deprecated](https://docs.discord.com/developers/change-log#updated-connection-property-field-names). The older names may still function today, but non-prefixed fields are recommended.

###### Example Identify

```
{
  "op": 2,
  "d": {
    "token": "my_token",
    "properties": {
      "os": "linux",
      "browser": "disco",
      "device": "disco"
    },
    "compress": true,
    "large_threshold": 250,
    "shard": [0, 1],
    "presence": {
      "activities": [{
        "name": "Cards Against Humanity",
        "type": 0
      }],
      "status": "dnd",
      "since": 91879201,
      "afk": false
    },
    // These intents represent 1 << 0 for GUILDS and 1 << 2 for GUILD_MODERATION
    // This connection will only receive the events defined in those two intents
    "intents": 5
  }
}

```

#### Resume

 Replays events missed by a disconnected client that is resuming. The [Gateway documentation](https://docs.discord.com/developers/events/gateway#resuming) explains resume behavior.

###### Resume Structure

| Field | Type | Description |
| --- | --- | --- |
| token | string | Session token |
| session_id | string | Session ID |
| seq | integer | Last sequence number received |

###### Example Resume

```
{
  "op": 6,
  "d": {
    "token": "randomstring",
    "session_id": "evenmorerandomstring",
    "seq": 1337
  }
}

```

#### Heartbeat

 Keeps the gateway connection active. Send it every `heartbeat_interval` milliseconds after receiving the [Opcode 10 Hello](https://docs.discord.com/developers/events/gateway-events#hello) payload. The nested `d` value is the most recent sequence number, `s`, received by the client; send `null` if no sequence number has arrived yet. Heartbeat behavior is described in the [Gateway documentation](https://docs.discord.com/developers/events/gateway#sending-heartbeats).

###### Example Heartbeat

```
{
  "op": 1,
  "d": 251
}

```

#### Request Guild Members

 Requests every member for one guild or for multiple guilds. During initial connection, if `GUILD_PRESENCES` [Gateway Intent](https://docs.discord.com/developers/events/gateway#gateway-intents) is absent, or if the guild has more than 75k members, Discord sends only voice members plus the connecting user's member object. Otherwise, for guilds above the `large_threshold` value set in [Gateway Identify](https://docs.discord.com/developers/events/gateway-events#identify), Discord limits the initial member set to online members, role-bearing members, nicknamed members, and voice participants; guilds below `large_threshold` send all members. Clients that need additional members must request them with this operation. Discord replies with [Guild Members Chunk](https://docs.discord.com/developers/events/gateway-events#guild-members-chunk) events containing up to 1000 members per chunk until every requested matching member has been sent. Privacy and infrastructure requirements impose these limits:
- `GUILD_PRESENCES` intent is required for `presences = true`; without it, the value is always false.
- `GUILD_MEMBERS` intent is required to request the full member list—`(query=‘’, limit=0<=n)`.
- Each request is limited to 1 `guild_id`.
- A prefix request using the `query` parameter can return at most 100 members.
- Requests using `user_ids` remain capped at 100 returned members.
 A new rate limit is being added to the Request Guild Members opcode. See [Introducing Rate Limit When Requesting All Guild Members](https://docs.discord.com/developers/change-log#introducing-rate-limit-when-requesting-all-guild-members) for timing and details.

###### Request Guild Members Structure

| Field | Type | Description | Required |
| --- | --- | --- | --- |
| guild_id | snowflake | ID of the guild whose members are being requested | true |
| query? | string | username prefix to match, or an empty string when returning all members | one of query or user_ids |
| limit | integer | maximum count of members to send for the `query`; use a limit of `0` with an empty string `query` to request every member | true when specifying query |
| presences? | boolean | sets whether presences should be included for matched members | false |
| user_ids? | snowflake or array of snowflakes | identifies the specific users to fetch | one of query or user_ids |
| nonce? | string | nonce used to match the Guild Members Chunk response | false |

 A nonce may be no larger than 32 bytes. If an invalid nonce is sent, Discord ignores it and the response member_chunk(s) omit the nonce.

###### Example Request Guild Members

```
{
  "op": 8,
  "d": {
    "guild_id": "41771983444115456",
    "query": "",
    "limit": 0
  }
}

```

#### Request Soundboard Sounds

 Requests soundboard sounds for a set of guilds. Discord replies with [Soundboard Sounds](https://docs.discord.com/developers/events/gateway-events#soundboard-sounds) events for each guild.

###### Request Soundboard Sounds Structure

| Field | Type | Description |
| --- | --- | --- |
| guild_ids | array of snowflakes | IDs of guilds whose soundboard sounds should be returned |

###### Example Request Soundboard Sounds

```
{
  "op": 31,
  "d": {
    "guild_ids": ["613425648685547541", "81384788765712384"]
  }
}

```

### Request Channel Info

 Requests temporary channel data for channels in a guild. Discord responds with a [Channel Info](https://docs.discord.com/developers/events/gateway-events#channel-info) event.

###### Request Channel Info Structure

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | guild id for the channel info request |
| fields | array of strings | Fields to request; currently available values are `status` and `voice_start_time`. |

###### Example Request Channel Info

```
{
  "guild_id": "613425648685547541",
  "fields": ["status", "voice_start_time"]
}

```

#### Update Voice State

 Sent by a client to join, change, or leave a voice channel.

###### Gateway Voice State Update Structure

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild |
| channel_id | ?snowflake | ID of the voice channel the client wants to enter, or null when disconnecting |
| self_mute | boolean | Whether the client is muted |
| self_deaf | boolean | Whether the client deafened |

###### Example Gateway Voice State Update

```
{
  "op": 4,
  "d": {
    "guild_id": "41771983423143937",
    "channel_id": "127121515262115840",
    "self_mute": false,
    "self_deaf": false
  }
}

```

#### Update Presence

 Sent by the client to report a presence or status change.

###### Gateway Presence Update Structure

| Field | Type | Description |
| --- | --- | --- |
| since | ?integer | Unix time in milliseconds when the client became idle, or null when the client is active |
| activities | array of activity objects | User’s activities |
| status | string | User’s new status |
| afk | boolean | Whether the client is afk |

###### Status Types

| Status | Description |
| --- | --- |
| online | Online |
| dnd | Do Not Disturb |
| idle | AFK |
| invisible | Invisible and shown as offline |
| offline | Offline |

###### Example Gateway Presence Update

```
{
  "op": 3,
  "d": {
    "since": 91879201,
    "activities": [{
      "name": "Save the Oxford Comma",
      "type": 0
    }],
    "status": "online",
    "afk": false
  }
}

```

### Receive Events

 Receive events are Gateway events carried in an [event payload](https://docs.discord.com/developers/events/gateway-events#payload-structure) and sent from Discord to an app through the Gateway connection. They describe activity in Discord servers where the app is installed.
| Name | Description |
| --- | --- |
| Hello | Defines the heartbeat interval |
| Ready | Contains the initial state information |
| Resumed | Response to Resume |
| Reconnect | Server is shutting down; the client should reconnect to the gateway and resume |
| Rate Limited | App hit a rate limit for a gateway opcode |
| Invalid Session | Identify or Resume failed, or the active session is invalid |
| Application Command Permissions Update | Application command permission was updated |
| Auto Moderation Rule Create | Auto Moderation rule was created |
| Auto Moderation Rule Update | Auto Moderation rule was updated |
| Auto Moderation Rule Delete | Auto Moderation rule was deleted |
| Auto Moderation Action Execution | Auto Moderation triggered a rule and ran an action, such as blocking a message |
| Channel Create | New guild channel created |
| Channel Update | Channel was updated |
| Channel Delete | Channel was deleted |
| Channel Info | Response to Request Channel Info |
| Channel Pins Update | Message was pinned or unpinned |
| Thread Create | Thread was created; also delivered when the user is added to a private thread |
| Thread Update | Thread was updated |
| Thread Delete | Thread was deleted |
| Thread List Sync | Delivered when access to a channel is gained, with all active threads in that channel |
| Thread Member Update | Thread member data for the current user changed |
| Thread Members Update | One or more users joined or left a thread |
| Entitlement Create | Entitlement was created |
| Entitlement Update | Entitlement was updated or renewed |
| Entitlement Delete | Entitlement was deleted |
| Guild Create | Unavailable guild was lazy-loaded, a guild became available, or the user joined a new guild |
| Guild Update | Guild was updated |
| Guild Delete | Guild became unavailable, or the user left or was removed from it |
| Guild Audit Log Entry Create | A guild audit log entry was added |
| Guild Ban Add | A user was banned from a guild |
| Guild Ban Remove | A user ban was removed from a guild |
| Guild Emojis Update | Guild emojis were updated |
| Guild Stickers Update | Guild stickers were updated |
| Guild Integrations Update | Guild integration was updated |
| Guild Member Add | New user joined a guild |
| Guild Member Remove | A user was removed from a guild |
| Guild Member Update | Guild member was updated |
| Guild Members Chunk | Response to Request Guild Members |
| Guild Role Create | Guild role was created |
| Guild Role Update | Guild role was updated |
| Guild Role Delete | Guild role was deleted |
| Guild Scheduled Event Create | Guild scheduled event was created |
| Guild Scheduled Event Update | Guild scheduled event was updated |
| Guild Scheduled Event Delete | Guild scheduled event was deleted |
| Guild Scheduled Event User Add | A user subscribed to a guild scheduled event |
| Guild Scheduled Event User Remove | A user unsubscribed from a guild scheduled event |
| Guild Soundboard Sound Create | Guild soundboard sound was created |
| Guild Soundboard Sound Update | Guild soundboard sound was updated |
| Guild Soundboard Sound Delete | Guild soundboard sound was deleted |
| Guild Soundboard Sounds Update | Guild soundboard sounds were updated |
| Soundboard Sounds | Response to Request Soundboard Sounds |
| Integration Create | Guild integration was created |
| Integration Update | Guild integration was updated |
| Integration Delete | Guild integration was deleted |
| Interaction Create | A user used an interaction, such as an Application Command |
| Invite Create | A channel invite was created |
| Invite Delete | A channel invite was deleted |
| Message Create | Message was created |
| Message Update | Message was edited |
| Message Delete | Message was deleted |
| Message Delete Bulk | Several messages were deleted together |
| Message Reaction Add | User reacted to a message |
| Message Reaction Remove | A user removed a reaction from a message |
| Message Reaction Remove All | Every reaction was explicitly cleared from a message |
| Message Reaction Remove Emoji | Every reaction for a specific emoji was explicitly cleared from a message |
| Presence Update | User was updated |
| Stage Instance Create | Stage instance was created |
| Stage Instance Update | Stage instance was updated |
| Stage Instance Delete | Stage instance was deleted or closed |
| Subscription Create | Premium App Subscription was created |
| Subscription Update | Premium App Subscription was updated |
| Subscription Delete | Premium App Subscription was deleted |
| Typing Start | A user began typing in a channel |
| User Update | Properties about the user changed |
| Voice Channel Effect Send | An effect was sent in a voice channel where the current user is connected |
| Voice Channel Start Time Update | Voice channel start time was updated |
| Voice Channel Status Update | Voice channel status was updated |
| Voice State Update | A user joined, left, or moved between voice channels |
| Voice Server Update | Guild’s voice server was updated |
| Webhooks Update | Guild channel webhook was created, updated, or deleted |
| Message Poll Vote Add | User voted on a poll |
| Message Poll Vote Remove | A user removed a poll vote |

#### Hello

 Sent when the websocket connection opens. It supplies the heartbeat interval the app should use.

###### Hello Structure

| Field | Type | Description |
| --- | --- | --- |
| heartbeat_interval | integer | Interval in milliseconds for app heartbeats |

###### Example Hello

```
{
  "op": 10,
  "d": {
    "heartbeat_interval": 45000
  }
}

```

#### Ready

 The ready event is dispatched after a client completes the initial gateway handshake for a new session. It can be the gateway's largest and most complex event because it provides the state a client needs before interacting with the rest of the platform. `guilds` lists the guilds where the bot is a member. They are unavailable at connection time, and the bot receives [Guild Create](https://docs.discord.com/developers/events/gateway-events#guild-create) events as each guild becomes available.

###### Ready Event Fields

| Field | Type | Description |
| --- | --- | --- |
| v | integer | API version |
| user | user object | User information, including email |
| guilds | array of Unavailable Guild objects | Guilds the user is in |
| session_id | string | Used for resuming connections |
| resume_gateway_url | string | Gateway URL for resuming connections |
| shard? | array of two integers (shard_id, num_shards) | Shard details for this session, when supplied during identify |
| application | partial application object | Contains `id` and `flags` |

#### Resumed

 The resumed event is dispatched when a client sends a [resume payload](https://docs.discord.com/developers/events/gateway-events#resume) to the gateway for an existing session.

#### Reconnect

 The reconnect event tells a client to reconnect to the gateway and resume an existing session when available. It may arrive at any stage of the gateway connection lifecycle, including before or instead of a [Hello](https://docs.discord.com/developers/events/gateway-events#hello) event. The server may close the connection a few seconds after dispatching reconnect.

###### Example Gateway Reconnect

```
{
  "op": 7,
  "d": null
}

```

#### Invalid Session

 Sent for at least three different conditions:
- the gateway could not start a session after receiving an [Opcode 2 Identify](https://docs.discord.com/developers/events/gateway-events#identify)
- the gateway could not resume an earlier session after receiving an [Opcode 6 Resume](https://docs.discord.com/developers/events/gateway-events#resume)
- the gateway invalidated an active session and needs the client to respond
 The nested `d` value is a boolean showing whether the session may be resumable. See [Connecting](https://docs.discord.com/developers/events/gateway#connecting) and [Resuming](https://docs.discord.com/developers/events/gateway#resuming) for details.

###### Example Gateway Invalid Session

```
{
  "op": 9,
  "d": false
}

```

### Application Commands

#### Application Command Permissions Update

 `APPLICATION_COMMAND_PERMISSIONS_UPDATE` event, delivered when permissions for an application command change. The nested payload is an [application command permissions](https://docs.discord.com/developers/interactions/application-commands#application-command-permissions-object-guild-application-command-permissions-structure) object.

### Auto Moderation

 Events related to [Auto Moderation](https://docs.discord.com/developers/resources/auto-moderation) are sent only to bot users that hold the `MANAGE_GUILD` permission.

#### Auto Moderation Rule Create

 Sent when a rule is created. The nested payload is an [auto moderation rule](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object) object.

#### Auto Moderation Rule Update

 Sent when a rule changes. The nested payload is an [auto moderation rule](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object) object.

#### Auto Moderation Rule Delete

 Sent when a rule is removed. The nested payload is an [auto moderation rule](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object) object.

#### Auto Moderation Action Execution

 Sent after a rule triggers and its action runs, such as when a message is blocked.

###### Auto Moderation Action Execution Event Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild where the action ran |
| action | auto moderation action object | Action which was executed |
| rule_id | snowflake | ID of the rule associated with the action |
| rule_trigger_type | trigger_type | Trigger type for the rule that fired |
| user_id | snowflake | ID of the user whose content caused the rule to fire |
| channel_id? | snowflake | ID of the channel where the user content appeared |
| message_id? | snowflake | ID of the user message containing the content, when any exists * |
| alert_system_message_id? | snowflake | ID of the system auto moderation message posted because of this action ** |
| content *** | string | User-generated text content |
| matched_keyword | ?string | Rule-configured word or phrase that matched |
| matched_content *** | ?string | Substring from the content that matched the rule |

 * `message_id` is absent when [Auto Moderation](https://docs.discord.com/developers/resources/auto-moderation) blocked the message or when the content was not part of a message. ** `alert_system_message_id` is absent when this event is unrelated to an action of type `SEND_ALERT_MESSAGE`. *** The `MESSAGE_CONTENT` (`1 << 15`) [gateway intent](https://docs.discord.com/developers/events/gateway#gateway-intents) is required for the `content` and `matched_content` fields.

### Channels

#### Channel Create

 Sent when a new guild channel relevant to the current user is created. The nested payload is a [channel](https://docs.discord.com/developers/resources/channel#channel-object) object.

#### Channel Update

 Sent when a channel changes. The nested payload is a [channel](https://docs.discord.com/developers/resources/channel#channel-object) object. This event is not sent for changes to `last_message_id`; track last_message_id changes through [Message Create](https://docs.discord.com/developers/events/gateway-events#message-create) events, or [Thread Create](https://docs.discord.com/developers/events/gateway-events#thread-create) events for `GUILD_FORUM` and `GUILD_MEDIA` channels. The event may mention roles or guild members that no longer exist in the guild.

#### Channel Delete

 Sent when a channel relevant to the current user is deleted. The nested payload is a [channel](https://docs.discord.com/developers/resources/channel#channel-object) object.

#### Channel Info

 Contains temporary data for channels in a guild. It is sent as the response to [Request Channel Info](https://docs.discord.com/developers/events/gateway-events#request-channel-info).

###### Channel Info Structure

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | The guild id |
| channels | array of channel info objects | Temporary data for channels in the guild |
