# Gateway Events (part 3)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Emitted after a user deliberately clears every reaction from a message.

###### Message Reaction Remove All Event Fields

| Field | Type | Description |
| --- | --- | --- |
| channel_id | snowflake | ID of the channel |
| message_id | snowflake | ID of the message |
| guild_id? | snowflake | ID of the guild |

#### Message Reaction Remove Emoji

 Emitted when a bot clears all reactions using a particular emoji from a message.

###### Message Reaction Remove Emoji Event Fields

| Field | Type | Description |
| --- | --- | --- |
| channel_id | snowflake | ID of the channel |
| guild_id? | snowflake | ID of the guild |
| message_id | snowflake | ID of the message |
| emoji | partial emoji object | Emoji that was removed |

### Presence

#### Presence Update

 When using [Gateway Intents](https://docs.discord.com/developers/events/gateway#gateway-intents), Presence Update events require the `GUILD_PRESENCES` intent. Presence represents a user's current guild state, and this event fires when that state or user information such as name or avatar changes. The embedded user object may be partial; only `id` is guaranteed, and all other fields are optional. No field is required beyond that, and field types are **not** validated, so clients must tolerate any field/type combination.

###### Presence Update Event Fields

| Field | Type | Description |
| --- | --- | --- |
| user | user object | User record for the presence change. |
| guild_id | snowflake | ID of the guild |
| status | string | One of "idle", "dnd", "online", or "offline". |
| activities | array of activity objects | User’s current activities |
| client_status | client_status object | User’s platform-dependent status |

#### Client Status Object

 For active sessions, each platform reports an "online", "idle", or "dnd" string. Offline or invisible users omit the matching platform field.
| Field | Type | Description |
| --- | --- | --- |
| desktop? | string | Status for the user's active desktop application session on Windows, Linux, or Mac. |
| mobile? | string | Status for the user's active mobile application session on iOS or Android. |
| web? | string | Status for the user's active web application session, including browser and bot user sessions. |

#### Activity Object

###### Activity Structure

| Field | Type | Description |
| --- | --- | --- |
| name | string | Activity’s name |
| type | integer | Activity type |
| url? | ?string | Stream URL; validation applies when type is 1. |
| created_at | integer | Unix timestamp in milliseconds for when the activity entered the user's session. |
| timestamps? | timestamps object | Unix timestamp values for the game's start and/or end. |
| application_id? | snowflake | Application ID for the game |
| status_display_type? | ?integer | Status display type that chooses which field appears in the user's member-list status text. |
| details? | ?string | The player's current activity. |
| details_url? | ?string | URL attached to clicks on the details text. |
| state? | ?string | Current party status for the user, or custom-status text. |
| state_url? | ?string | URL attached to clicks on the state text. |
| emoji? | ?emoji object | Emoji for a custom status. |
| party? | party object | Details about the player's current party. |
| assets? | assets object | Presence images and the hover text for those images. |
| secrets? | secrets object | Rich Presence secrets used for joining and spectating. |
| instance? | boolean | Whether the activity represents an instanced game session. |
| flags? | integer | Activity flags combined with `OR`; they describe the payload contents. |
| buttons? | array of buttons | Rich Presence custom buttons, up to 2. |

 Bot users can set only `name`, `state`, `type`, and `url`.

###### Activity Types

| ID | Name | Format | Example |
| --- | --- | --- | --- |
| 0 | Playing | Playing `{name}` | ”Playing Rocket League” |
| 1 | Streaming | Streaming `{details}` | ”Streaming Rocket League” |
| 2 | Listening | Listening to `{name}` | ”Listening to Spotify” |
| 3 | Watching | Watching `{name}` | ”Watching YouTube Together” |
| 4 | Custom | `{emoji}` `{state}` | ”:smiley: I am cool” |
| 5 | Competing | Competing in `{name}` | ”Competing in Arena World Champions” |

 Streaming activity currently works only with Twitch and YouTube. Valid URLs must start with `https://twitch.tv/` or `https://youtube.com/` .

###### Status Display Types

| ID | Name | Example |
| --- | --- | --- |
| 0 | Name | ”Listening to Spotify” |
| 1 | State | ”Listening to Rick Astley” |
| 2 | Details | "Listening to Never Gonna Give You Up" |

 The rule covers every activity type. The examples use "Listening" only as a stable illustration of how the fields can be populated.

###### Activity Timestamps

| Field | Type | Description |
| --- | --- | --- |
| start? | integer | Unix time in milliseconds when the activity began. |
| end? | integer | Unix time in milliseconds when the activity finishes. |

 Listening and Watching activities can provide both start and end timestamps so clients show a time bar.

###### Activity Emoji

| Field | Type | Description |
| --- | --- | --- |
| name | string | Name of the emoji |
| id? | snowflake | ID of the emoji |
| animated? | boolean | Whether the emoji is animated |

###### Activity Party

| Field | Type | Description |
| --- | --- | --- |
| id? | string | ID of the party |
| size? | array of two integers (current_size, max_size) | Displays the party's current size and maximum size. |

###### Activity Assets

| Field | Type | Description |
| --- | --- | --- |
| large_image? | string | See Activity Asset Image |
| large_text? | string | Hover text for the activity's large image. |
| large_url? | string | URL opened from clicks on the large image. |
| small_image? | string | See Activity Asset Image |
| small_text? | string | Hover text for the activity's small image. |
| small_url? | string | URL opened from clicks on the small image. |
| invite_cover_image? | string | See Activity Asset Image. Shown as the banner on a Game Invite. |

###### Activity Asset Image

 Activity asset image values are arbitrary strings, commonly snowflake IDs or prefixed image IDs. Handle this field defensively because users can supply it and Discord does not sanitize it. To send an external image through media proxy, put the URL in the field value. Gateway payloads expose only the `mp:` prefix.
| Type | Format | Image URL |
| --- | --- | --- |
| Application Asset | `{application_asset_id}` | See Application Asset Image Formatting |
| Media Proxy Image | `mp:{image_id}` | `https://media.discordapp.net/{image_id}` |

 **Uploaded assets** using the Application Asset type allow PNG, JPEG, and WebP, but not animation. **External URL assets** using the Media Proxy Image type can point at any publicly reachable image URL, including GIF, animated WebP, and AVIF.

###### Activity Secrets

| Field | Type | Description |
| --- | --- | --- |
| join? | string | Secret for joining a party |
| spectate? | string | Secret for spectating a game |
| match? | string | Secret tied to a specific instanced match. |

###### Activity Flags

| Name | Value |
| --- | --- |
| INSTANCE | `1 << 0` |
| JOIN | `1 << 1` |
| SPECTATE | `1 << 2` |
| JOIN_REQUEST | `1 << 3` |
| SYNC | `1 << 4` |
| PLAY | `1 << 5` |
| PARTY_PRIVACY_FRIENDS | `1 << 6` |
| PARTY_PRIVACY_VOICE_CHANNEL | `1 << 7` |
| EMBEDDED | `1 << 8` |

###### Activity Buttons

 Gateway payloads represent `buttons` as an array of strings containing button labels. Bots cannot read activity button URLs for a user. When sending `buttons`, provide an array of this object:
| Field | Type | Description |
| --- | --- | --- |
| label | string | Button label text, from 1-32 characters. |
| url | string | Button click target URL, from 1-512 characters. |

###### Example Activity

```
{
  "details": "24H RL Stream for Charity",
  "state": "Rocket League",
  "name": "Twitch",
  "type": 1,
  "url": "https://www.twitch.tv/discord"
}

```

###### Example Activity with Rich Presence

```
{
  "name": "Rocket League",
  "type": 0,
  "application_id": "379286085710381999",
  "state": "In a Match",
  "details": "Ranked Duos: 2-1",
  "timestamps": {
    "start": 15112000660000
  },
  "party": {
    "id": "9dd6594e-81b3-49f6-a6b5-a679e6a060d3",
    "size": [2, 2]
  },
  "assets": {
    "large_image": "351371005538729000",
    "large_text": "DFH Stadium",
    "small_image": "351371005538729111",
    "small_text": "Silver III"
  },
  "secrets": {
    "join": "025ed05c71f639de8bfaa0d679d7c94b2fdce12f",
    "spectate": "e7eb30d2ee025ed05c71ea495f770b76454ee4e0",
    "match": "4b2fdce12f639de8bfa7e3591b71a0d679d7c93f"
  }
}

```

 A client can change its game status no more than 5 times in 20 seconds.

#### Typing Start

 Emitted when a user begins typing in a channel.

###### Typing Start Event Fields

| Field | Type | Description |
| --- | --- | --- |
| channel_id | snowflake | ID of the channel |
| guild_id? | snowflake | ID of the guild |
| user_id | snowflake | ID of the user |
| timestamp | integer | Unix time in seconds when the user began typing. |
| member? | member object | Member that began typing, included when the channel is in a guild. |

#### User Update

 Emitted when properties on the current bot user change. The inner payload is a [user](https://docs.discord.com/developers/resources/user#user-object) object.

### Voice

#### Voice Channel Effect Send

 Emitted when someone sends an effect, for example an emoji reaction or soundboard sound, in a voice channel joined by the current user.

###### Voice Channel Effect Send Event Fields

| Field | Type | Description |
| --- | --- | --- |
| channel_id | snowflake | ID for the channel where the effect was sent. |
| guild_id | snowflake | ID for the guild where the effect was sent. |
| user_id | snowflake | ID for the user who sent the effect. |
| emoji? | ?emoji object | Emoji payload for emoji reaction and soundboard effects. |
| animation_type? | ?integer | Emoji animation type for emoji reaction and soundboard effects. |
| animation_id? | integer | Emoji animation ID for emoji reaction and soundboard effects. |
| sound_id? | snowflake or integer | Soundboard sound ID for soundboard effects. |
| sound_volume? | double | Soundboard sound volume, from 0 to 1, for soundboard effects. |

###### Animation Types

| Type | Value | Description |
| --- | --- | --- |
| PREMIUM | 0 | A playful animation sent by a Nitro subscriber. |
| BASIC | 1 | The standard animation |

#### Voice State Update

 Emitted when someone joins, leaves, or moves between voice channels. The inner payload is a [voice state](https://docs.discord.com/developers/resources/voice#voice-state-object) object.

#### Voice Server Update

 Emitted when a guild voice server changes. It appears during initial voice connection and when the active voice instance fails over to another server. If `endpoint` is null, the allocated voice server disappeared and Discord is reallocating it. Disconnect from the current voice server, then wait for a newly allocated voice server before reconnecting.

###### Voice Server Update Event Fields

| Field | Type | Description |
| --- | --- | --- |
| token | string | Voice connection token |
| guild_id | snowflake | Guild associated with this voice server update. |
| endpoint | ?string | Voice server host |

###### Example Voice Server Update Payload

```
{
  "token": "my_token",
  "guild_id": "41771983423143937",
  "endpoint": "sweetwater-12345.discord.media:2048"
}

```

### Webhooks

#### Webhooks Update

 Emitted after a guild channel webhook is created, modified, or deleted.

###### Webhooks Update Event Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild |
| channel_id | snowflake | ID of the channel |

### Interactions

#### Interaction Create

 Emitted when a user invokes an [Application Command](https://docs.discord.com/developers/interactions/application-commands) or uses a [Message Component](https://docs.discord.com/developers/components/reference#component-object). The inner payload is an [Interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure).

### Stage Instances

#### Stage Instance Create

 Emitted when a [Stage instance](https://docs.discord.com/developers/resources/stage-instance) is created, meaning the Stage is now "live". The inner payload is a [Stage instance](https://docs.discord.com/developers/resources/stage-instance#stage-instance-object).

#### Stage Instance Update

 Emitted when a [Stage instance](https://docs.discord.com/developers/resources/stage-instance) is updated. The inner payload is a [Stage instance](https://docs.discord.com/developers/resources/stage-instance#stage-instance-object).

#### Stage Instance Delete

 Emitted when a [Stage instance](https://docs.discord.com/developers/resources/stage-instance) is deleted, meaning the Stage closed. The inner payload is a [Stage instance](https://docs.discord.com/developers/resources/stage-instance#stage-instance-object).

### Subscriptions

#### Subscription Create

 Do not use subscription status to award perks. Use [entitlements](https://docs.discord.com/developers/resources/entitlement#entitlement-object) to decide whether a user should receive access to a given SKU. The [Implementing App Subscriptions](https://docs.discord.com/developers/monetization/implementing-app-subscriptions) guide has more detail. This event fires when a Premium App [Subscription](https://docs.discord.com/developers/resources/subscription) is created. The inner payload is a [Subscription](https://docs.discord.com/developers/resources/subscription#subscription-object). At receipt time, a Subscription's `status` may be **inactive** or **active**. Later `SUBSCRIPTION_UPDATE` events arrive if `status` changes to **active**. Best practice is to wait for entitlements before giving users any perks.

#### Subscription Update

 Emitted when a Premium App [Subscription](https://docs.discord.com/developers/resources/subscription) is updated. The inner payload is a [Subscription](https://docs.discord.com/developers/resources/subscription#subscription-object) object.

#### Subscription Delete

 Emitted when a Premium App [Subscription](https://docs.discord.com/developers/resources/subscription) is deleted. The inner payload is a [Subscription](https://docs.discord.com/developers/resources/subscription#subscription-object) object.

### Polls

#### Message Poll Vote Add

 Emitted when a user casts a poll vote. For polls with multiple selections, Discord sends one event for each answer.

###### Message Poll Vote Add Fields

| Field | Type | Description |
| --- | --- | --- |
| user_id | snowflake | ID of the user |
| channel_id | snowflake | ID of the channel |
| message_id | snowflake | ID of the message |
| guild_id? | snowflake | ID of the guild |
| answer_id | integer | ID of the answer |

#### Message Poll Vote Remove

 Emitted when a user withdraws a poll vote. For polls that allow multiple selections, Discord sends one event for each answer.

###### Message Poll Vote Remove Fields

| Field | Type | Description |
| --- | --- | --- |
| user_id | snowflake | ID of the user |
| channel_id | snowflake | ID of the channel |
| message_id | snowflake | ID of the message |
| guild_id? | snowflake | ID of the guild |
| answer_id | integer | ID of the answer |

### Rate Limits

#### Rate Limited

 Emitted when an app hits a gateway event rate limit, such as for [Request Guild Members](https://docs.discord.com/developers/events/gateway-events#request-guild-members). The changelog entry [Introducing Rate Limit When Requesting All Guild Members](https://docs.discord.com/developers/change-log#introducing-rate-limit-when-requesting-all-guild-members) explains the rate limit and rollout timing.

###### Rate Limited Fields

| Field | Type | Description |
| --- | --- | --- |
| opcode | integer | Gateway opcode for the rate-limited event. |
| retry_after | float | Seconds to wait before sending another request. |
| meta | Rate Limit Metadata for Opcode | Metadata describing the rate-limited event. |

###### Rate Limit Metadata for Opcode Structure

| Opcode | Type |
| --- | --- |
| 8 | Request Guild Member Rate Limit Metadata |

###### Request Guild Member Rate Limit Metadata Structure

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild whose members should be fetched. |
| nonce? | string | nonce used to match the Guild Members Chunk response. |
