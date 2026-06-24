# Poll, Sku, Soundboard, Stage Instance

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: Poll Endpoints; SKU Object; Soundboard Sound Object; Stage Instance Object.

## Poll Endpoints

A poll stores the data for a poll message. (Figure: Example message containing a poll)

### Poll Object

 The poll object contains several nested layers. It is structured for future extension, so some fields are more elaborate than the current feature set requires.

###### Poll Object Structure

| Field | Type | Description |
| --- | --- | --- |
| question | Poll Media Object | The poll question. Only `text` is supported. |
| answers | List of Poll Answer Objects | The list of answer options for the poll. |
| expiry | ?IS08601 timestamp | Timestamp when the poll closes. |
| allow_multiselect | boolean | Whether users may choose more than one answer |
| layout_type | integer | The poll layout type |
| results? | Poll Results Object | The results of the poll |

 `expiry` is nullable so non-expiring polls can be supported later, although every poll currently has an expiry.

### Poll Create Request Object

 This request object is used to create polls across endpoints. It resembles the main [poll object](https://docs.discord.com/developers/resources/poll#poll-object-poll-object-structure), but it is not identical. The key difference is `duration`, which later becomes `expiry`.

###### Poll Create Request Object Structure

| Field | Type | Description |
| --- | --- | --- |
| question | Poll Media Object | The poll question. Only `text` is supported. |
| answers | List of Poll Answer Objects | Available poll answers, with a maximum of 10 |
| duration? | integer | How many hours the poll remains open, capped at 32 days. Defaults to 24 |
| allow_multiselect? | boolean | Whether users may select multiple answers. Defaults to false |
| layout_type? | integer | The poll layout type. Defaults to DEFAULT. |

### Layout Type

 Additional poll layouts may exist later. Currently this value is 1.
| Type | ID | Description |
| --- | --- | --- |
| DEFAULT | 1 | The, uhm, default layout type. |

### Poll Media Object

 The poll media object is shared by questions and answers. It leaves room for additional display formats later. At present, `question` supports only `text`, and answers may include optional `emoji`.

###### Poll Media Object Structure

| Field | Type | Description |
| --- | --- | --- |
| text? | string | The text of the field |
| emoji? | partial emoji | The emoji of the field |

 `text` is expected to be non-null for questions and answers today, but do not rely on that forever. `text` is limited to 300 characters on the question and 55 characters on each answer. When an answer uses an emoji, send only `id` for a custom emoji or only `name` for a default emoji.

### Poll Answer Object

 `answer_id` is the numeric label for an answer. The current implementation assigns 1 to the first answer and then increments sequentially, but callers should not rely on that ordering. Polls currently allow up to 10 answers.

###### Poll Answer Object Structure

| Field | Type | Description |
| --- | --- | --- |
| answer_id* | integer | The ID of the answer |
| poll_media | Poll Media Object | The data of the answer |

 * Sent only in responses from Discord's API/Gateway.

### Poll Results Object

 This object contains vote counts per answer. The `results` field can be absent in responses where Discord's backend did not fetch poll results; treat that as unknown results, not zero results. Previously received results can still be used. While a poll is active, counts are usually close but not guaranteed exact because counting at scale has edge cases. After the poll ends, a background job performs the final accurate tally, and that process is complete once `is_finalized` is `true`. Ended polls always include results. If `answer_counts` lacks a row for an answer, that answer has no votes.

###### Poll Results Object Structure

| Field | Type | Description |
| --- | --- | --- |
| is_finalized | boolean | Whether vote counting has been finalized precisely |
| answer_counts | List of Poll Answer Count Object | The counts for each answer |

###### Poll Answer Count Object Structure

| Field | Type | Description |
| --- | --- | --- |
| id | integer | The `answer_id` |
| count | integer | Vote count for this answer |
| me_voted | boolean | Whether the current user selected this answer |

### Poll Endpoints

 Create polls with [Create Message](https://docs.discord.com/developers/resources/message#create-message). A poll message is locked after creation, and apps cannot vote in polls.

### Get Answer Voters

  Return the users who voted for a particular answer.

###### Query String Params

| Field | Type | Description | Default |
| --- | --- | --- | --- |
| after? | snowflake | Return users after this user ID | absent |
| limit? | integer | Maximum users to return (1-100) | 25 |

###### Response Body

| Field | Type | Description |
| --- | --- | --- |
| users | array of user | Users that voted for this answer |

### End Poll

  End the poll immediately. A user cannot end another user's poll. Returns a [message](https://docs.discord.com/developers/resources/message#message-object) object and emits a [Message Update](https://docs.discord.com/developers/events/gateway-events#message-update) Gateway event.

## SKU Object

In Discord, SKUs (stock-keeping units) are premium offerings that an application can make available to users or guilds.

### SKU Object

###### SKU Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | ID of SKU |
| type | integer | Type of SKU |
| application_id | snowflake | ID of the parent application |
| name | string | User-facing name for the premium offering |
| slug | string | System-created URL slug derived from the SKU's name |
| flags | integer | SKU flags represented as a combined bitfield |

###### SKU Example

```
{
  "id": "1088510058284990888",
  "type": 5,
  "dependent_sku_id": null,
  "application_id": "788708323867885999",
  "manifest_labels": null,
  "access_type": 1,
  "name": "Test Premium",
  "features": [],
  "release_date": null,
  "premium": false,
  "slug": "test-premium",
  "flags": 128,
  "show_age_gate": false
}

```

###### SKU Types

 Subscription SKUs use either `SUBSCRIPTION` with `type: 5` or `SUBSCRIPTION_GROUP` with `type:6`. Current integrations should use the SKU with `type: 5`. Discord automatically creates a `SUBSCRIPTION_GROUP` for each `SUBSCRIPTION` SKU, and those group SKUs are not currently used.
| Type | Value | Description |
| --- | --- | --- |
| DURABLE | 2 | Durable one-time purchase |
| CONSUMABLE | 3 | Consumable one-time purchase |
| SUBSCRIPTION | 5 | Represents a recurring subscription |
| SUBSCRIPTION_GROUP | 6 | System-created group for every SUBSCRIPTION SKU |

###### SKU Flags

 Subscription offerings support two access models:
- **Guild Subscriptions**: A user buys the subscription for one server, granting premium benefits to everyone in that server.
- **User Subscriptions**: A user buys the subscription for their own account, granting premium benefits to that user across every server.
 Use the `flags` field with a bitwise `&` operator to distinguish user subscriptions from server subscriptions.
| Type | Value | Description |
| --- | --- | --- |
| AVAILABLE | `1 << 2` | SKU is available for purchase |
| GUILD_SUBSCRIPTION | `1 << 7` | Recurring SKU bought by a user and applied to one server. Grants access to all users in that server. |
| USER_SUBSCRIPTION | `1 << 8` | Recurring SKU bought by a user for that user. Grants access to the purchaser in every server. |

### List SKUs

 GET/applications/[{application.id}](https://docs.discord.com/developers/resources/application#application-object)/skus Returns every SKU for the application. Because of Discord's SKU and subscription model, a subscription offering appears as two SKUs. For subscription entitlement integration and testing, use the SKU whose `type: 5`.

```
[
  {
    "id": "1088510053843210999",
    "type": 6,
    "dependent_sku_id": null,
    "application_id": "788708323867885999",
    "manifest_labels": null,
    "access_type": 1,
    "name": "Test Premium",
    "features": [],
    "release_date": null,
    "premium": false,
    "slug": "test-premium",
    "flags": 128,
    "show_age_gate": false
  },
  {
    "id": "1088510058284990888",
    "type": 5,
    "dependent_sku_id": null,
    "application_id": "788708323867885999",
    "manifest_labels": null,
    "access_type": 1,
    "name": "Test Premium",
    "features": [],
    "release_date": null,
    "premium": false,
    "slug": "test-premium",
    "flags": 128,
    "show_age_gate": false
  }
]

```

## Soundboard Sound Object

Playing a soundboard sound in a voice channel emits the Gateway event [Voice Channel Effect Send](https://docs.discord.com/developers/events/gateway-events#voice-channel-effect-send) for connected users. Discord supplies [default sounds](https://docs.discord.com/developers/resources/soundboard#list-default-soundboard-sounds) to everyone. Guild soundboard sounds can be [created in a guild](https://docs.discord.com/developers/resources/soundboard#create-guild-soundboard-sound); guild users can use those sounds there, and Nitro subscribers can use them across all guilds. Use [Request Soundboard Sounds](https://docs.discord.com/developers/events/gateway-events#request-soundboard-sounds) to retrieve soundboard sounds for a set of guilds over the Gateway.

### Soundboard Sound Object

###### Soundboard Sound Structure

| Field | Type | Description |
| --- | --- | --- |
| name | string | the name of this sound |
| sound_id | snowflake | the id of this sound |
| volume | double | volume for this sound, from 0 to 1 |
| emoji_id | ?snowflake | custom emoji id for this sound |
| emoji_name | ?string | standard emoji unicode character for this sound |
| guild_id? | snowflake | guild id containing this sound |
| available | boolean | whether the sound is usable; can be false if Server Boosts are lost |
| user? | user object | user who created the sound |

###### Example Default Soundboard Sound

```
{
  "name": "quack",
  "sound_id": "1",
  "volume": 1.0,
  "emoji_id": null,
  "emoji_name": "🦆",
  "available": true
}

```

###### Example Guild Soundboard Sound

```
{
  "name": "Yay",
  "sound_id": "1106714396018884649",
  "volume": 1,
  "emoji_id": "989193655938064464",
  "emoji_name": null,
  "guild_id": "613425648685547541",
  "available": true
}

```

### Sound Files

 A soundboard sound can be fetched as MP3 or Ogg from this URL:

```
https://cdn.discordapp.com/soundboard-sounds/{sound_id}

```

### Send Soundboard Sound

  Plays a soundboard sound in a voice channel where the user is connected. This emits Gateway event [Voice Channel Effect Send](https://docs.discord.com/developers/events/gateway-events#voice-channel-effect-send). Requires `SPEAK` and `USE_SOUNDBOARD`; for sounds from another server, `USE_EXTERNAL_SOUNDS` is required too. The user must be connected to the voice channel with a [voice state](https://docs.discord.com/developers/resources/voice#voice-state-object) where `deaf`, `self_deaf`, `mute`, and `suppress` are disabled.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| sound_id | snowflake | soundboard sound id to play |
| source_guild_id? | snowflake | guild id that owns the soundboard sound; required for sounds from other servers |

### List Default Soundboard Sounds

  Returns objects for [soundboard sound](https://docs.discord.com/developers/resources/soundboard#soundboard-sound-object) entries available to every user.

### List Guild Soundboard Sounds

  Returns the guild's soundboard sound list. Includes `user` fields when the bot has `CREATE_GUILD_EXPRESSIONS` or `MANAGE_GUILD_EXPRESSIONS`.

###### Response Structure

| Field | Type |
| --- | --- |
| items | array of soundboard sound objects |

### Get Guild Soundboard Sound

  Returns the [soundboard sound](https://docs.discord.com/developers/resources/soundboard#soundboard-sound-object) object for the specified sound id. Includes `user` when the bot has `CREATE_GUILD_EXPRESSIONS` or `MANAGE_GUILD_EXPRESSIONS`.

### Create Guild Soundboard Sound

  Creates a guild soundboard sound. Requires `CREATE_GUILD_EXPRESSIONS`. On success, returns the created [soundboard sound](https://docs.discord.com/developers/resources/soundboard#soundboard-sound-object) object; Discord emits Gateway event [Guild Soundboard Sound Create](https://docs.discord.com/developers/events/gateway-events#guild-soundboard-sound-create). Uploads for soundboard sounds are limited to 512kb and 5.2 seconds. This endpoint accepts the `X-Audit-Log-Reason` header.

###### JSON

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name | string | soundboard sound name (2-32 characters) |
| sound | data uri | base64-encoded mp3 or ogg sound data, similar to image data |
| volume? | ?double | soundboard sound volume; valid range 0 to 1 and default 1 |
| emoji_id? | ?snowflake | custom emoji id for the soundboard sound |
| emoji_name? | ?string | standard emoji unicode character for the soundboard sound |

### Modify Guild Soundboard Sound

  Updates an existing soundboard sound. Sounds created by the current user require `CREATE_GUILD_EXPRESSIONS` or `MANAGE_GUILD_EXPRESSIONS`; sounds from other users require `MANAGE_GUILD_EXPRESSIONS`. On success, returns the updated [soundboard sound](https://docs.discord.com/developers/resources/soundboard#soundboard-sound-object) object and emits the [Guild Soundboard Sound Update](https://docs.discord.com/developers/events/gateway-events#guild-soundboard-sound-update) Gateway event. Every parameter is optional. This endpoint accepts the `X-Audit-Log-Reason` header.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name | string | soundboard sound name (2-32 characters) |
| volume | ?double | soundboard sound volume, from 0 to 1 |
| emoji_id | ?snowflake | custom emoji id for the soundboard sound |
| emoji_name | ?string | standard emoji unicode character for the soundboard sound |

### Delete Guild Soundboard Sound

  Deletes a soundboard sound. Sounds created by the current user require `CREATE_GUILD_EXPRESSIONS` or `MANAGE_GUILD_EXPRESSIONS`; sounds from other users require `MANAGE_GUILD_EXPRESSIONS`. Success returns `204 No Content`; Discord emits Gateway event [Guild Soundboard Sound Delete](https://docs.discord.com/developers/events/gateway-events#guild-soundboard-sound-delete). The endpoint accepts the `X-Audit-Log-Reason` header.

## Stage Instance Object

A Stage Instance describes a live stage.

### Stage Instance Object

###### Stage Instance Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | id of this Stage instance |
| guild_id | snowflake | guild id for the related Stage channel |
| channel_id | snowflake | id of the related Stage channel |
| topic | string | Stage instance topic (1-120 characters) |
| privacy_level | integer | Stage instance privacy level |
| discoverable_disabled | boolean | whether Stage Discovery is disabled (deprecated) |
| guild_scheduled_event_id | ?snowflake | scheduled event id for this Stage instance |

###### Privacy Level

| Level | Value | Description |
| --- | --- | --- |
| PUBLIC | 1 | The Stage instance is publicly visible. (deprecated) |
| GUILD_ONLY | 2 | The Stage instance is visible only to guild members. |

###### Example Stage Instance

```
{
  "id": "840647391636226060",
  "guild_id": "197038439483310086",
  "channel_id": "733488538393510049",
  "topic": "Testing Testing, 123",
  "privacy_level": 1,
  "discoverable_disabled": false,
  "guild_scheduled_event_id": "947656305244532806"
}

```

### Definitions

 These terms describe stage behavior.
- **Liveness:** A Stage channel is *live* when a stage instance exists for it. It is *not live* when no stage instance exists.
- **Speakers:** A Stage participant is a *speaker* when the participant's [voice state](https://docs.discord.com/developers/resources/voice#voice-state-object) is not `suppress`ed and does not include `request_to_speak_timestamp`.
- **Moderators**: A guild member is a Stage channel *moderator* when they have all of these [permissions](https://docs.discord.com/developers/topics/permissions):
  - `MANAGE_CHANNELS`
  - `MUTE_MEMBERS`
  - `MOVE_MEMBERS`

- **Topic**: The short description shown below the channel's name and in related surfaces.
- **Public**: `privacy_level` set to `PUBLIC` makes the Stage instance public. During that state:
  - Users in that Stage may have it appear in their [activities](https://docs.discord.com/developers/events/gateway-events#presence).
  - Stage channel [Invites](https://docs.discord.com/developers/resources/invite#invite-object) include the `stage_instance` field.

### Auto Closing

 If a Stage channel remains without speakers for several minutes, Discord automatically deletes the Stage instance.

### Create Stage Instance

 POST/stage-instances Creates a Stage instance tied to a Stage channel. Returns the created [Stage instance](https://docs.discord.com/developers/resources/stage-instance#stage-instance-object-stage-instance-structure). The Gateway event [Stage Instance Create](https://docs.discord.com/developers/events/gateway-events#stage-instance-create) is emitted. Requires a moderator user for the Stage channel. This endpoint accepts the `X-Audit-Log-Reason` header.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| channel_id | snowflake | id of the Stage channel |
| topic | string | topic for the Stage instance (1-120 characters) |
| privacy_level? | integer | privacy level for the Stage instance (default GUILD_ONLY) |
| send_start_notification? * | boolean | whether to notify @everyone when a Stage instance starts |
| guild_scheduled_event_id? | snowflake | guild scheduled event tied to this Stage instance |

 * The stage moderator needs `MENTION_EVERYONE` for Discord to send this notification.

### Get Stage Instance

 GET/stage-instances/[{channel.id}](https://docs.discord.com/developers/resources/channel#channel-object) Returns the stage instance for the Stage channel, when one exists.

### Modify Stage Instance

 PATCH/stage-instances/[{channel.id}](https://docs.discord.com/developers/resources/channel#channel-object) Changes fields on an existing Stage instance. Returns the changed [Stage instance](https://docs.discord.com/developers/resources/stage-instance#stage-instance-object-stage-instance-structure). The Gateway event [Stage Instance Update](https://docs.discord.com/developers/events/gateway-events#stage-instance-update) is emitted. Requires moderator status in the Stage channel. This endpoint accepts the `X-Audit-Log-Reason` header.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| privacy_level? | integer | privacy level for the Stage instance |

### Delete Stage Instance

 DELETE/stage-instances/[{channel.id}](https://docs.discord.com/developers/resources/channel#channel-object) Removes the Stage instance. Returns `204 No Content`. The Gateway event [Stage Instance Delete](https://docs.discord.com/developers/events/gateway-events#stage-instance-delete) is emitted. Requires moderator status in the Stage channel. This endpoint accepts the `X-Audit-Log-Reason` header.
