# Message (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

### Message Object

 Describes a message posted in a Discord channel.

###### Message Structure

 The [Gateway documentation](https://docs.discord.com/developers/events/gateway-events#message-create) lists fields that are specific to `MESSAGE_CREATE` and `MESSAGE_UPDATE`. If an app has not configured or been approved for the [`MESSAGE_CONTENT` privileged intent (`1 << 15`)](https://docs.discord.com/developers/events/gateway#message-content-intent), Discord sends empty `content`, `embeds`, `attachments`, and `components` values and omits `poll`.
| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | id of the message |
| channel_id | snowflake | channel id where the message was sent |
| author [1] | user object | message author; not always a valid user, see the note below |
| content [2] | string | contents of the message |
| timestamp | ISO8601 timestamp | when this message was sent |
| edited_timestamp | ?ISO8601 timestamp | edit timestamp, or null when the message has never been edited |
| tts | boolean | whether the message used TTS |
| mention_everyone | boolean | whether this message mentions everyone |
| mentions | array of user objects | users explicitly mentioned by the message |
| mention_roles | array of role object ids | roles explicitly mentioned by the message |
| mention_channels? [3] | array of channel mention objects | channels explicitly mentioned by the message |
| attachments [2] | array of attachment objects | attached files not referenced from embeds or components |
| embeds [2] | array of embed objects | any embedded content |
| reactions? | array of reaction objects | reactions to the message |
| nonce? | integer or string | nonce used to confirm that a message was sent |
| pinned | boolean | whether this message is pinned |
| webhook_id? | snowflake | webhook id when a webhook generated the message |
| type | integer | type of message |
| activity? | message activity object | included with Rich Presence-related chat embeds |
| application? | partial application object | included with Rich Presence-related chat embeds |
| application_id? | snowflake | application id for an Interaction or application-owned webhook message |
| flags? | integer | message flags encoded as a bitfield |
| message_reference? | message reference object | metadata identifying the source message for a crosspost, channel follow add, pin, or reply |
| message_snapshots? [5] | array of message snapshot objects | message tied to the `message_reference`; only a minimal field subset is included, such as excluding `author` |
| referenced_message? [4] | ?message object | message linked by message_reference |
| interaction_metadata? | message interaction metadata object | included when an interaction produced the message |
| interaction? | message interaction object | **Deprecated in favor of `interaction_metadata`**; included when the message responds to an interaction |
| thread? | channel object | thread created from this message, including the thread member object |
| components? [2] | array of message components | included when the message includes components such as buttons, action rows, or other interactive elements |
| sticker_items? | array containing message sticker item objects | included when the message includes stickers |
| stickers? | array of sticker objects | **Deprecated** stickers included with the message |
| position? | integer | approximate thread position, represented by a generally increasing integer that can contain gaps or duplicates; compare it with `total_message_sent` on the parent thread to estimate relative order |
| role_subscription_data? | role subscription data object | role subscription purchase or renewal data that triggered this ROLE_SUBSCRIPTION_PURCHASE message |
| resolved? | resolved data | resolved data for users, members, channels, and roles referenced by the message |
| poll? [2] | poll object | A poll! |
| call? | message call object | call linked to the message |
| shared_client_theme? | shared client theme object | custom client-side theme shared in the message |

 [1] The author object uses the user object shape, but it is a valid user only when a user or bot user created the message. For webhook-generated messages, the author object maps to the webhook id, username, and avatar; check `webhook_id` to identify those messages. [2] Without configuration or approval for the [`MESSAGE_CONTENT` privileged intent (`1 << 15`)](https://docs.discord.com/developers/events/gateway#message-content-intent), apps receive empty `content`, `embeds`, `attachments`, and `components` values and no `poll`. [3] `mention_channels` does not include every channel mention. It can include only textual channels visible to everyone in a public guild, and `mention_channels` currently appears only on crossposted messages through Channel Following. If no mention qualifies, Discord omits the field. [4] This field is returned only for messages whose `type` is `19` (REPLY), `21` (THREAD_STARTER_MESSAGE), or `23` (CONTEXT_MENU_COMMAND). If such a message lacks `referenced_message`, the backend did not try to fetch the replied-to message, so the state is unknown. If the field exists and is null, the referenced message was deleted. [5] See [message reference types](https://docs.discord.com/developers/resources/message#message-reference-types)

###### Message Types

 Types `19` and `20` exist only in API v8 and later. In v6 they appear as type `0`. Type `21` requires API v9 or later.
| Type | Value | Deletable |
| --- | --- | --- |
| DEFAULT | 0 | true |
| RECIPIENT_ADD | 1 | false |
| RECIPIENT_REMOVE | 2 | false |
| CALL | 3 | false |
| CHANNEL_NAME_CHANGE | 4 | false |
| CHANNEL_ICON_CHANGE | 5 | false |
| CHANNEL_PINNED_MESSAGE | 6 | true |
| USER_JOIN | 7 | true |
| GUILD_BOOST | 8 | true |
| GUILD_BOOST_TIER_1 | 9 | true |
| GUILD_BOOST_TIER_2 | 10 | true |
| GUILD_BOOST_TIER_3 | 11 | true |
| CHANNEL_FOLLOW_ADD | 12 | true |
| GUILD_DISCOVERY_DISQUALIFIED | 14 | true |
| GUILD_DISCOVERY_REQUALIFIED | 15 | true |
| GUILD_DISCOVERY_GRACE_PERIOD_INITIAL_WARNING | 16 | true |
| GUILD_DISCOVERY_GRACE_PERIOD_FINAL_WARNING | 17 | true |
| THREAD_CREATED | 18 | true |
| REPLY | 19 | true |
| CHAT_INPUT_COMMAND | 20 | true |
| THREAD_STARTER_MESSAGE | 21 | false |
| GUILD_INVITE_REMINDER | 22 | true |
| CONTEXT_MENU_COMMAND | 23 | true |
| AUTO_MODERATION_ACTION | 24 | true* |
| ROLE_SUBSCRIPTION_PURCHASE | 25 | true |
| INTERACTION_PREMIUM_UPSELL | 26 | true |
| STAGE_START | 27 | true |
| STAGE_END | 28 | true |
| STAGE_SPEAKER | 29 | true |
| STAGE_TOPIC | 31 | true |
| GUILD_APPLICATION_PREMIUM_SUBSCRIPTION | 32 | true |
| GUILD_INCIDENT_ALERT_MODE_ENABLED | 36 | true |
| GUILD_INCIDENT_ALERT_MODE_DISABLED | 37 | true |
| GUILD_INCIDENT_REPORT_RAID | 38 | true |
| GUILD_INCIDENT_REPORT_FALSE_ALARM | 39 | true |
| PURCHASE_NOTIFICATION | 44 | true |
| POLL_RESULT | 46 | true |

 * Deletion requires the `MANAGE_MESSAGES` permission

###### Message Activity Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | type of message activity |
| party_id? | string | Rich Presence event party_id |

###### Message Activity Types

| Type | Value |
| --- | --- |
| JOIN | 1 |
| SPECTATE | 2 |
| LISTEN | 3 |
| JOIN_REQUEST | 5 |

###### Message Flags

| Flag | Value | Description |
| --- | --- | --- |
| CROSSPOSTED | `1 << 0` | message has been published to subscribed channels through Channel Following |
| IS_CROSSPOST | `1 << 1` | message was created from a message in another channel through Channel Following |
| SUPPRESS_EMBEDS | `1 << 2` | omit all embeds when serializing this message |
| SOURCE_MESSAGE_DELETED | `1 << 3` | source message for this crosspost was deleted through Channel Following |
| URGENT | `1 << 4` | message came from the urgent message system |
| HAS_THREAD | `1 << 5` | message has a thread with the same id as the message |
| EPHEMERAL | `1 << 6` | message is visible only to the user who invoked the Interaction |
| LOADING | `1 << 7` | message is an Interaction Response while the bot is thinking |
| FAILED_TO_MENTION_SOME_ROLES_IN_THREAD | `1 << 8` | message could not mention some roles and add their members to the thread |
| SUPPRESS_NOTIFICATIONS | `1 << 12` | message does not trigger push or desktop notifications |
| IS_VOICE_MESSAGE | `1 << 13` | message is a voice message |
| HAS_SNAPSHOT | `1 << 14` | message includes a snapshot through Message Forwarding |
| IS_COMPONENTS_V2 * | `1 << 15` | allows creation of messages driven entirely by components |

 * After a message is sent with this flag, the flag can't be removed from that message.

###### Example Message

```
{
  "reactions": [
    {
      "count": 1,
      "count_details": {
         "burst": 0,
         "normal": 1
      },
      "me": false,
      "me_burst": false,
      "emoji": {
         "id": null,
         "name": "🔥"
      },
      "burst_colors": []
    }
  ],
  "attachments": [],
  "tts": false,
  "embeds": [],
  "timestamp": "2017-07-11T17:27:07.299000+00:00",
  "mention_everyone": false,
  "id": "334385199974967042",
  "pinned": false,
  "edited_timestamp": null,
  "author": {
    "username": "Mason",
    "discriminator": "9999",
    "id": "53908099506183680",
    "avatar": "a_bab14f271d565501444b2ca3be944b25"
  },
  "mention_roles": [],
  "content": "Supa Hot",
  "channel_id": "290926798999357250",
  "mentions": [],
  "type": 0
}

```

###### Example Crossposted Message

```
{
  "reactions": [
    {
      "count": 1,
      "count_details": {
         "burst": 0,
         "normal": 1
      },
      "me": false,
      "me_burst": false,
      "emoji": {
         "id": null,
         "name": "🔥"
      },
      "burst_colors": []
    }
  ],
  "attachments": [],
  "tts": false,
  "embeds": [],
  "timestamp": "2017-07-11T17:27:07.299000+00:00",
  "mention_everyone": false,
  "id": "334385199974967042",
  "pinned": false,
  "edited_timestamp": null,
  "author": {
    "username": "Mason",
    "discriminator": "9999",
    "id": "53908099506183680",
    "avatar": "a_bab14f271d565501444b2ca3be944b25"
  },
  "mention_roles": [],
  "mention_channels": [
    {
      "id": "278325129692446722",
      "guild_id": "278325129692446720",
      "name": "big-news",
      "type": 5
    }
  ],
  "content": "Big news! In this <#278325129692446722> channel!",
  "channel_id": "290926798999357250",
  "mentions": [],
  "type": 0,
  "flags": 2,
  "message_reference": {
    "type": 0,
    "channel_id": "278325129692446722",
    "guild_id": "278325129692446720",
    "message_id": "306588351130107906"
  }
}

```

### Message Interaction Metadata Object

 Interaction metadata records the interaction source plus relevant server and user IDs. It is one of [Application Command Interaction Metadata](https://docs.discord.com/developers/resources/message#message-interaction-metadata-object-application-command-interaction-metadata-structure), [Message Component Interaction Metadata](https://docs.discord.com/developers/resources/message#message-interaction-metadata-object-message-component-interaction-metadata-structure), or [Modal Submit Interaction Metadata](https://docs.discord.com/developers/resources/message#message-interaction-metadata-object-modal-submit-interaction-metadata-structure).

###### Application Command Interaction Metadata Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | ID of the interaction |
| type | interaction type | Type of interaction |
| user | user object | User who triggered the interaction |
| authorizing_integration_owners | dictionary keyed by application integration types | installation context IDs for an interaction; details are in Authorizing Integration Owners Object |
| original_response_message_id? | snowflake | original response message ID, present only on follow-up messages |
| target_user? | user object | user targeted by the command, present only for user command interactions |
| target_message_id? | snowflake | message ID targeted by the command, present only for message command interactions. The original response message also has `message_reference` and `referenced_message` pointing to that message. |

###### Message Component Interaction Metadata Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | ID of the interaction |
| type | interaction type | Type of interaction |
| user | user object | User who triggered the interaction |
| authorizing_integration_owners | dictionary keyed by application integration types | installation context IDs for an interaction; details are in Authorizing Integration Owners Object |
| original_response_message_id? | snowflake | original response message ID, present only on follow-up messages |
| interacted_message_id | snowflake | ID of the message that held the interactive component |

###### Modal Submit Interaction Metadata Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | ID of the interaction |
| type | interaction type | Type of interaction |
| user | user object | User who triggered the interaction |
| authorizing_integration_owners | dictionary keyed by application integration types | installation context IDs for an interaction; details are in Authorizing Integration Owners Object |
| original_response_message_id? | snowflake | original response message ID, present only on follow-up messages |
| triggering_interaction_metadata | Application Command Interaction Metadata object or Message Component Interaction Metadata object | metadata for the interaction that opened the modal |

### Message Call Object

 Details for the call attached to a private channel message.

###### Message Call Object Structure

| Field | Type | Description |
| --- | --- | --- |
| participants | array of snowflakes | array of user object ids for users who joined the call |
| ended_timestamp? | ?ISO8601 timestamp | time when call ended |

### Message Reference Object

#### Message Reference Structure

| Field | Type | Description |
| --- | --- | --- |
| type? * | integer | type of reference. |
| message_id? | snowflake | id of the originating message |
| channel_id? ** | snowflake | id of the originating message's channel |
| guild_id? | snowflake | id of the originating message's guild |
| fail_if_not_exists? | boolean | when sending, whether to return an error if the referenced message is missing instead of sending a normal non-reply message; default true |

 * If `type` is omitted, treat it as `DEFAULT` to preserve behavior from before message references had types. Future API versions will require this field. `channel_id` is optional when creating a reply, but it is always present in received events or responses that include this model. **Required for forwards.**

#### Message Reference Types

 Defines how the associated data is filled in.
| Type | Value | Coupled Message Field | Description |
| --- | --- | --- | --- |
| DEFAULT | 0 | `referenced_message` | Standard reference used for replies. |
| FORWARD | 1 | `message_snapshot` | Reference that targets a message snapshot at a specific moment. |

 `FORWARD` is limited to basic messages, meaning messages without strong bindings to a non global entity. Discord supports only messages with type `DEFAULT`, `REPLY`, `CHAT_INPUT_COMMAND`, or `CONTEXT_MENU_COMMAND`; messages with a poll, call, or activity are not supported. This may change later. Applications must be able to read a message's content before forwarding it. If an application tries to forward unreadable content, Discord returns `160014` and explains that unreadable message content cannot be forwarded.

#### Message Reference Content Attribution

 Message references provide general attribution on a message. Several message types include a `message_reference` object.

###### Crosspost messages

- Crosspost messages started in another channel and carry the IS_CROSSPOST flag.
- These messages include all three fields, each pointing back to the original crossposted message.

###### Channel Follow Add messages

- Channel Follow Add messages are automatic messages created when a followed channel posts into the current channel (type 12).
- These messages include `channel_id` and `guild_id`, pointing to the followed announcement channel.

###### Pin messages

- Pin messages are automatic messages created when a message is pinned (type 6).
- These messages include `message_id` and `channel_id`, plus `guild_id` when the pin is in a guild, pointing to the pinned message.

###### Forwards

- Forwards capture a snapshot of another message.
- These messages include an array of [`message_snapshot`](https://docs.discord.com/developers/resources/message#message-snapshot-object) objects containing a copy of the original message. The copy follows the message structure but returns only the minimal fields needed for context and rendering.
  - Notably, `author` is not included

- Detect a forwarded message by checking its `message_reference.type` field
  - `message_snapshots` contains the message data for the forward. Currently only 1 snapshot is supported.
  - prevents spoofing forwarded data
  - `message_snapshots` are captured when the forward is created and are **immutable**; later edits to the original message are not copied over.

- Create a forward by sending a message with a message_reference whose type is `FORWARD`.
  - Required fields: `type`, `message_id`, `channel_id`
  - the requester must have `VIEW_CHANNEL` permissions

###### Replies

- Replies are messages responding to an earlier message (type 19).
- These messages include `message_id` and `channel_id`, plus `guild_id` when the reply is in a guild, pointing to the replied-to message. The channel_id and guild_id match the reply.
- Create replies by including a message_reference when sending a message. When sending, only `message_id` is required.
- The referenced message may be resolved into the `referenced_message` field.

###### Thread Created messages

- Thread Created messages are automatic messages created when a public thread is made from an older message or without a message (type 18).
- These messages include `channel_id` and `guild_id`, pointing to the created thread channel.

###### Thread starter messages

- Thread starter messages are the first messages in public threads created from messages. They point to the parent-channel message that started the thread. (type 21)
- These messages include `message_id`, `channel_id`, and `guild_id`.
- The referenced message may be resolved into the `referenced_message` field.
- These messages never include content, embeds, or attachments; they mainly contain `message_reference` and `referenced_message`.

###### Poll result messages

- Poll result messages are automatic messages sent after a poll ends and the results are finalized. (type 46)
- These messages include `message_id` and `channel_id`, pointing to the original poll message. The `channel_id` matches the poll's channel.
- The author matches the poll author and is mentioned.
- These messages include a [`poll_result` embed](https://docs.discord.com/developers/resources/message#embed-fields-by-embed-type-poll-result-embed-fields)

###### Context Menu Command messages

- Context Menu Command messages are sent when a user runs a context menu Application Command. (type 23)
- If the command was a [message command](https://docs.discord.com/developers/interactions/application-commands#message-commands), the message includes `message_id` and `channel_id`, plus `guild_id` when it is in a guild, pointing to the message used by the command. The channel_id and guild_id match the new message.
- The referenced message may be resolved into the `referenced_message` field.

#### Voice Messages

 Voice messages are messages carrying the `IS_VOICE_MESSAGE` flag. They have these properties.
- They cannot be edited.
- Exactly 1 audio attachment is allowed. Content, stickers, and similar extra payloads are not allowed.
- The [attachment](https://docs.discord.com/developers/resources/message#attachment-object) includes extra fields: `duration_secs` and `waveform`. The attachment `Content-Type` must start with `audio/` for these fields to apply.
 `waveform` previews the full voice message with 1 byte per datapoint encoded in base64. Clients sample at most once every 100 milliseconds and downsample so the waveform has no more than 256 datapoints. As of 2023-04-14, clients upload a 1 channel, 48000 Hz, 32kbps Opus stream in an OGG container. Encoding and waveform details are implementation details that may change without warning or documentation.

### Message Snapshot Object

#### Message Snapshot Structure

| Field | Type | Description |
| --- | --- | --- |
| message* | partial message object | minimal field subset from the forwarded message |

 * The current message-field subset is: `type`, `content`, `embeds`, `attachments`, `timestamp`, `edited_timestamp`, `flags`, `mentions`, `mention_roles`, `stickers`, `sticker_items`, and `components`. Message snapshots can support nested snapshots, but the current nesting depth limit is 1.

### Reaction Object

###### Reaction Structure

| Field | Type | Description |
| --- | --- | --- |
| count | integer | total uses of this emoji as a reaction, including super reacts |
| count_details | object | Reaction count details object |
| me | boolean | whether the current user reacted with this emoji |
| me_burst | boolean | whether the current user used a super reaction with this emoji |
| emoji | partial emoji object | emoji information |
| burst_colors | array | HEX colors used by the super reaction |

### Reaction Count Details Object

 Reaction count details break down normal and super reaction counts for the related emoji.

###### Reaction Count Details Structure

| Field | Type | Description |
| --- | --- | --- |
| burst | integer | Count of super reactions |
| normal | integer | Count of normal reactions |

### Embed Object

###### Embed Structure

| Field | Type | Description |
| --- | --- | --- |
| title? | string | title of embed |
| type? | string | embed type; webhook embeds always use rich |
| description? | string | description of embed |
| url? | string | url of embed |
| timestamp? | ISO8601 timestamp | timestamp of embed content |
| color? | integer | color code of the embed |
| footer? | embed footer object | footer information |
| image? | embed image object | image information |
| thumbnail? | embed image object | thumbnail information |
| video? | embed video object | video information |
| provider? | embed provider object | provider information |
| author? | embed author object | author information |
| fields? | array of embed field objects | fields information, max of 25 |
| flags? | integer | embed flags encoded as a bitfield |

###### Embed Types

| Type | Description |
| --- | --- |
| rich | generic embed produced from embed attributes |
| image | image embed |
| video | video embed |
| gifv | animated gif image embed displayed as a video embed |
| article | article embed |
| link | link embed |
| poll_result | poll result embed |

###### Embed Flags

| Flag | Value | Description |
| --- | --- | --- |
| IS_CONTENT_INVENTORY_ENTRY | `1 << 5` | embed is a fallback for replying to an activity card |

###### Embed Video Structure
