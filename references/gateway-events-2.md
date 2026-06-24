# Gateway Events (part 2)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

###### Channel Info Channel Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | The channel id |
| status? | ?string | The voice channel status |
| voice_start_time? | ?integer | Voice session start time as a Unix timestamp (in seconds) |

#### Voice Channel Status Update

 Emitted after the voice channel status changes.
| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | The channel id |
| guild_id | snowflake | The guild id |
| status | ?string | The new voice channel status |

#### Voice Channel Start Time Update

 Emitted after the voice channel start time changes.
| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | The channel id |
| guild_id | snowflake | The guild id |
| voice_start_time? | ?integer | Voice session start time as a Unix timestamp (in seconds) |

#### Thread Create

 Dispatched when a thread is created for the current user, becomes relevant to the current user, or gains the current user as a member. The payload body is a [channel](https://docs.discord.com/developers/resources/channel#channel-object) object.
- Newly created threads include an extra `newly_created` boolean field.
- Adding the current user to an existing private thread also includes a [thread member](https://docs.discord.com/developers/resources/channel#thread-member-object) object.

#### Thread Update

 Dispatched after a thread changes. The payload body is a [channel](https://docs.discord.com/developers/resources/channel#channel-object) object. Changes to `last_message_id` do not trigger this event; track last_message_id movement by listening for [Message Create](https://docs.discord.com/developers/events/gateway-events#message-create) events.

#### Thread Delete

 Dispatched when a thread visible to the current user is deleted. The payload body contains only the `id`, `guild_id`, `parent_id`, and `type` fields from the [channel](https://docs.discord.com/developers/resources/channel#channel-object) object.

#### Thread List Sync

 Dispatched after the current user *gains* access to a channel.

###### Thread List Sync Event Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild |
| channel_ids? | array of snowflakes | Parent channel IDs for the thread sync. If this field is absent, the sync covers the whole guild. The array can also include channel_ids without active threads, which signals that local data for those channels should be cleared. |
| threads | array of channel objects | Active threads in the specified channels that the current user can access |
| members | array of thread member objects | Thread member objects for the current user from synced threads, showing which threads include the current user |

#### Thread Member Update

 Dispatched when the current user's [thread member](https://docs.discord.com/developers/resources/channel#thread-member-object) object changes. The payload body is a [thread member](https://docs.discord.com/developers/resources/channel#thread-member-object) object plus an extra `guild_id` field. Most bots rarely need this event; for bots, it primarily confirms thread membership. See the [threads docs](https://docs.discord.com/developers/topics/threads) for more detail.

###### Thread Member Update Event Extra Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild |

#### Thread Members Update

 Dispatched when any user is added to or removed from a thread. Without the `GUILD_MEMBERS` [Gateway Intent](https://docs.discord.com/developers/events/gateway#gateway-intents), the current user receives it only for their own addition or removal.

###### Thread Members Update Event Fields

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | ID of the thread |
| guild_id | snowflake | ID of the guild |
| member_count | integer | Approximate thread member count, capped at 50 |
| added_members?* | array of thread member objects | Users added to the thread |
| removed_member_ids? | array of snowflakes | IDs of users removed from the thread |

 * In this gateway event, each added thread member's thread member object also carries the [guild member](https://docs.discord.com/developers/resources/guild#guild-member-object) object and nullable [presence](https://docs.discord.com/developers/events/gateway-events#presence) object.

#### Channel Pins Update

 Dispatched when a text channel message is pinned or unpinned. Deleting a pinned message does not send this event.

###### Channel Pins Update Event Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id? | snowflake | ID of the guild |
| channel_id | snowflake | ID of the channel |
| last_pin_timestamp? | ?ISO8601 timestamp | Timestamp for when the newest pinned message was pinned |

### Entitlements

#### Entitlement Create

 Note: `ENTITLEMENT_CREATE` behavior changed on October 1, 2024. See the [Change Log and Entitlement Migration Guide](https://docs.discord.com/developers/change-log#premium-apps-entitlement-migration-and-new-subscription-api) for details about the change. The event is dispatched when an entitlement is created, and its payload body is an [entitlement](https://docs.discord.com/developers/resources/entitlement#entitlement-object) object.

#### Entitlement Update

 Note: `ENTITLEMENT_UPDATE` behavior changed on October 1, 2024. See the [Change Log and Entitlement Migration Guide](https://docs.discord.com/developers/change-log#premium-apps-entitlement-migration-and-new-subscription-api) for details about the change. The event is dispatched when an entitlement is updated, and its payload body is an [entitlement](https://docs.discord.com/developers/resources/entitlement#entitlement-object) object. For subscription entitlements, it fires only when a user's subscription ends and includes an `ends_at` timestamp for the entitlement end.

#### Entitlement Delete

 Dispatched when an entitlement is deleted. The payload body is an [entitlement](https://docs.discord.com/developers/resources/entitlement#entitlement-object) object. Entitlement deletion is uncommon and happens when:
- Discord refunds a subscription
- Discord uses internal tooling to remove a user's entitlement
- Discord deletes an API-created app-managed entitlement
 Expiration does *not* delete entitlements.

### Guilds

#### Guild Create

 This event may arrive for three reasons:
1. During initial connection, the client uses it to lazily load and backfill all unavailable guilds listed by the [Ready](https://docs.discord.com/developers/events/gateway-events#ready) event. Guilds unavailable because of an outage instead produce a [Guild Delete](https://docs.discord.com/developers/events/gateway-events#guild-delete) event.
2. When a Guild becomes available to the client again.
3. When the current user joins a new Guild.
 If an outage is underway, the guild object in scenarios 1 and 3 can be flagged unavailable. The payload body can be:
- An available Guild: a [guild](https://docs.discord.com/developers/resources/guild#guild-object) object plus the extra fields listed below.
- An unavailable Guild: an [unavailable guild](https://docs.discord.com/developers/resources/guild#unavailable-guild-object) object.

###### Guild Create Extra Fields

| Field | Type | Description |
| --- | --- | --- |
| joined_at | ISO8601 timestamp | Time when this guild was joined |
| large | boolean | `true` when this is treated as a large guild |
| unavailable? | boolean | `true` when an outage makes this guild unavailable |
| member_count | integer | Total member count for this guild |
| voice_states | array of partial voice state objects | Current voice-channel member states; the `guild_id` key is not present |
| members | array of guild member objects | Users in the guild |
| channels | array of channel objects | Channels in the guild |
| threads | array of channel objects | Every active thread in the guild that the current user is allowed to view |
| presences | array of partial presence update objects | Member presences for the guild; when size exceeds `large threshold`, only non-offline members are included |
| stage_instances | array of stage instance objects | Stage instances in the guild |
| guild_scheduled_events | array of guild scheduled event objects | Scheduled events in the guild |
| soundboard_sounds | array of soundboard sound objects | Soundboard sounds in the guild |

 If your bot lacks the `GUILD_PRESENCES` [Gateway Intent](https://docs.discord.com/developers/events/gateway#gateway-intents), or the guild has more than 75k members, this event returns members and presences only for your bot and users in voice channels.

#### Guild Update

 Dispatched after a guild is updated. The payload body is a [guild](https://docs.discord.com/developers/resources/guild#guild-object) object.

#### Guild Delete

 Dispatched when a guild becomes unavailable, was already unavailable because of an outage, or the user leaves or is removed from a guild. The payload body is an [unavailable guild](https://docs.discord.com/developers/resources/guild#unavailable-guild-object) object. If `unavailable` is unset, the user was removed from the guild.

#### Guild Audit Log Entry Create

 Dispatched after a guild audit log entry is created. The payload body is an [Audit Log Entry](https://docs.discord.com/developers/resources/audit-log#audit-log-entry-object) object with an extra `guild_id` key. Only bots with the `VIEW_AUDIT_LOG` permission receive this event.

###### Guild Audit Log Entry Create Event Extra Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild |

#### Guild Ban Add

 Dispatched when a user is banned from a guild. Bots receive it only with the `BAN_MEMBERS` or `VIEW_AUDIT_LOG` permission.

###### Guild Ban Add Event Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild |
| user | a user object | User who was banned |

#### Guild Ban Remove

 Dispatched when a user is unbanned from a guild. Bots receive it only with the `BAN_MEMBERS` or `VIEW_AUDIT_LOG` permission.

###### Guild Ban Remove Event Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild |
| user | a user object | User who was unbanned |

#### Guild Emojis Update

 Dispatched after a guild's emojis are updated.

###### Guild Emojis Update Event Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild |
| emojis | array | Array of emojis |

#### Guild Stickers Update

 Dispatched after a guild's stickers are updated.

###### Guild Stickers Update Event Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild |
| stickers | array | Array of stickers |

#### Guild Integrations Update

 Dispatched after a guild integration is updated.

###### Guild Integrations Update Event Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild with updated integrations |

#### Guild Member Add

 When [Gateway Intents](https://docs.discord.com/developers/events/gateway#gateway-intents) are used, `GUILD_MEMBERS` is required for this event. It is dispatched when a user joins a guild, and can also arrive for users who already belong to the guild. The payload body is a [guild member](https://docs.discord.com/developers/resources/guild#guild-member-object) object plus an extra `guild_id` key:

###### Guild Member Add Extra Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild |

#### Guild Member Remove

 When [Gateway Intents](https://docs.discord.com/developers/events/gateway#gateway-intents) are used, `GUILD_MEMBERS` is required for this event. It is dispatched when a user leaves, is kicked from, or is banned from a guild.

###### Guild Member Remove Event Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild |
| user | a user object | User who was removed |

#### Guild Member Update

 When [Gateway Intents](https://docs.discord.com/developers/events/gateway#gateway-intents) are used, `GUILD_MEMBERS` is required for this event. It is dispatched when a guild member changes, including changes to that member's user object.

###### Guild Member Update Event Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild |
| roles | array of snowflakes | User role ids |
| user | a user object | User |
| nick? | ?string | User nickname inside the guild |
| avatar | ?string | Member’s guild avatar hash |
| banner | ?string | Member’s guild banner hash |
| joined_at | ?ISO8601 timestamp | Time when the user joined the guild |
| premium_since? | ?ISO8601 timestamp | Time when the user began boosting the guild |
| deaf? | boolean | Whether voice channels deafen the user |
| mute? | boolean | Whether voice channels mute the user |
| pending? | boolean | Whether the user still has not passed the guild's Membership Screening requirements |
| communication_disabled_until? | ?ISO8601 timestamp | Time when the user's timeout expires and guild communication is restored; null or a past time means the user is not timed out |
| avatar_decoration_data? | ?avatar decoration data | Member guild avatar decoration data |
| collectibles? | ?collectibles object | Member collectibles data |

#### Guild Members Chunk

 Dispatched in response to [Guild Request Members](https://docs.discord.com/developers/events/gateway-events#request-guild-members). Use `chunk_index` with `chunk_count` to determine how many chunks remain for the request.

###### Guild Members Chunk Event Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild |
| members | array of guild member objects | Set of guild members |
| chunk_index | integer | Index of this chunk among the expected response chunks (`0 <= chunk_index < chunk_count`) |
| chunk_count | integer | Total expected chunk count for this response |
| not_found? | array | Returned here when an invalid ID is passed to `REQUEST_GUILD_MEMBERS` |
| presences? | array of presence objects | Presences for returned members appear here when `true` is passed to `REQUEST_GUILD_MEMBERS` |
| nonce? | string | Nonce from the Guild Members Request |

#### Guild Role Create

 Dispatched after a guild role is created.

###### Guild Role Create Event Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild |
| role | a role object | Role that was created |

#### Guild Role Update

 Dispatched after a guild role is updated.

###### Guild Role Update Event Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild |
| role | a role object | Role that was updated |

#### Guild Role Delete

 Dispatched after a guild role is deleted.

###### Guild Role Delete Event Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild |
| role_id | snowflake | ID of the role |

#### Guild Scheduled Event Create

 Dispatched after a guild scheduled event is created. The payload body is a [guild scheduled event](https://docs.discord.com/developers/resources/guild-scheduled-event#guild-scheduled-event-object) object.

#### Guild Scheduled Event Update

 Dispatched after a guild scheduled event is updated. The payload body is a [guild scheduled event](https://docs.discord.com/developers/resources/guild-scheduled-event#guild-scheduled-event-object) object.

#### Guild Scheduled Event Delete

 Dispatched after a guild scheduled event is deleted. The payload body is a [guild scheduled event](https://docs.discord.com/developers/resources/guild-scheduled-event#guild-scheduled-event-object) object.

#### Guild Scheduled Event User Add

 Dispatched when a user subscribes to a guild scheduled event.

###### Guild Scheduled Event User Add Event Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_scheduled_event_id | snowflake | ID for the guild scheduled event |
| user_id | snowflake | ID of the user |
| guild_id | snowflake | ID of the guild |

#### Guild Scheduled Event User Remove

 Dispatched when a user unsubscribes from a guild scheduled event.

###### Guild Scheduled Event User Remove Event Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_scheduled_event_id | snowflake | ID for the guild scheduled event |
| user_id | snowflake | ID of the user |
| guild_id | snowflake | ID of the guild |

#### Guild Soundboard Sound Create

 Dispatched after a guild soundboard sound is created. The payload body is a [soundboard sound](https://docs.discord.com/developers/resources/soundboard#soundboard-sound-object) object.

#### Guild Soundboard Sound Update

 Dispatched after a guild soundboard sound is updated. The payload body is a [soundboard sound](https://docs.discord.com/developers/resources/soundboard#soundboard-sound-object) object.

#### Guild Soundboard Sound Delete

 Dispatched after a guild soundboard sound is deleted.

###### Guild Soundboard Sound Delete Event Fields

| Field | Type | Description |
| --- | --- | --- |
| sound_id | snowflake | ID of the deleted sound |
| guild_id | snowflake | ID of the guild that contained the sound |

#### Guild Soundboard Sounds Update

 Dispatched after multiple guild soundboard sounds are updated.

###### Guild Soundboard Sounds Update Event Fields

| Field | Type | Description |
| --- | --- | --- |
| soundboard_sounds | array of soundboard sound objects | The guild’s soundboard sounds |
| guild_id | snowflake | ID of the guild |

#### Soundboard Sounds

 Contains the guild's soundboard sound list. It is dispatched in response to [Request Soundboard Sounds](https://docs.discord.com/developers/events/gateway-events#request-soundboard-sounds).

###### Soundboard Sounds Event Fields

| Field | Type | Description |
| --- | --- | --- |
| soundboard_sounds | array of soundboard sound objects | The guild’s soundboard sounds |
| guild_id | snowflake | ID of the guild |

### Integrations

#### Integration Create

 Dispatched after an integration is created. The payload body is an [integration](https://docs.discord.com/developers/resources/guild#integration-object) object with `user` omitted and an added `guild_id` key:

###### Integration Create Event Additional Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild |

#### Integration Update

 Dispatched after an integration is updated. The payload body is an [integration](https://docs.discord.com/developers/resources/guild#integration-object) object with `user` omitted and an added `guild_id` key:

###### Integration Update Event Additional Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID of the guild |

#### Integration Delete

 Dispatched after an integration is deleted.

###### Integration Delete Event Fields

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | Integration ID |
| guild_id | snowflake | ID of the guild |
| application_id? | snowflake | ID of the bot/OAuth2 application for this Discord integration |

### Invites

 All [Invite](https://docs.discord.com/developers/resources/invite) events are delivered only to bot users that have the `MANAGE_CHANNELS` permission for the channel.

#### Invite Create

 Dispatched after a channel invite is created.

###### Invite Create Event Fields

| Field | Type | Description |
| --- | --- | --- |
| channel_id | snowflake | Channel the invite is for |
| code | string | Unique invite code |
| created_at | ISO8601 timestamp | Time when the invite was created |
| guild_id? | snowflake | Guild of the invite |
| inviter? | user object | User that created the invite |
| max_age | integer | Invite validity duration (in seconds) |
| max_uses | integer | Maximum use count for the invite |
| target_type? | integer | Target type for this voice channel invite |
| target_user? | user object | User whose stream should be shown for this voice channel stream invite |
| target_application? | partial application object | Embedded application opened by this voice channel embedded application invite |
| temporary | boolean | Whether the invite is temporary; invited users are kicked when they disconnect unless they receive a role |
| uses | integer | Number of times the invite has been used; always 0 |
| expires_at | ?ISO8601 timestamp | Expiration date for this invite |
| role_ids? | array of snowflakes | Role ID(s) assigned in the guild to users who accept this invite |

#### Invite Delete

 Dispatched after an invite is deleted.

###### Invite Delete Event Fields

| Field | Type | Description |
| --- | --- | --- |
| channel_id | snowflake | Channel of the invite |
| guild_id? | snowflake | Guild of the invite |
| code | string | Unique invite code |

### Messages

 Ephemeral messages differ from persistent messages because they go directly to the user and the sending bot instead of through the guild channel. Therefore, ephemeral messages depend on the [`DIRECT_MESSAGES` intent](https://docs.discord.com/developers/events/gateway#list-of-intents), and their message object omits `guild_id` and `member`.

#### Message Create

 Dispatched after a message is created. The payload body is a [message](https://docs.discord.com/developers/resources/message#message-object) object plus these extra fields:

###### Message Create Extra Fields

| Field | Type | Description |
| --- | --- | --- |
| guild_id? | snowflake | ID of the guild where the message was sent, except for ephemeral messages |
| member? | partial guild member object | Member properties for the message author; absent for ephemeral messages and webhook messages |
| mentions | array of user objects, optionally with an additional partial member field | Users explicitly mentioned by the message |
| channel_type? | integer | Channel type where the message was sent |

#### Message Update

 Dispatched after a message is updated. The payload body is a [message](https://docs.discord.com/developers/resources/message#message-object) object with the same extra fields as [MESSAGE_CREATE](https://docs.discord.com/developers/events/gateway-events#message-create). In message updates, `tts` is always false.

#### Message Delete

 Dispatched after a message is deleted.

###### Message Delete Event Fields

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | ID of the message |
| channel_id | snowflake | ID of the channel |
| guild_id? | snowflake | ID of the guild |

#### Message Delete Bulk

 Dispatched after multiple messages are deleted together.

###### Message Delete Bulk Event Fields

| Field | Type | Description |
| --- | --- | --- |
| ids | array of snowflakes | IDs of the messages |
| channel_id | snowflake | ID of the channel |
| guild_id? | snowflake | ID of the guild |

#### Message Reaction Add

 Dispatched after a user adds a reaction to a message.

###### Message Reaction Add Event Fields

| Field | Type | Description |
| --- | --- | --- |
| user_id | snowflake | ID of the user |
| channel_id | snowflake | ID of the channel |
| message_id | snowflake | ID of the message |
| guild_id? | snowflake | ID of the guild |
| member? | member object | Member that reacted, when the reaction occurred in a guild |
| emoji | a partial emoji object | Emoji used to react - example |
| message_author_id? | snowflake | ID of the user who wrote the message that received the reaction |
| burst | boolean | true when this is a super-reaction |
| burst_colors? | array of strings | Super-reaction animation colors in "#rrggbb" format |
| type | integer | The type of reaction |

#### Message Reaction Remove

 Dispatched after a user removes a reaction from a message.

###### Message Reaction Remove Event Fields

| Field | Type | Description |
| --- | --- | --- |
| user_id | snowflake | ID of the user |
| channel_id | snowflake | ID of the channel |
| message_id | snowflake | ID of the message |
| guild_id? | snowflake | ID of the guild |
| emoji | a partial emoji object | Emoji used to react - example |
| burst | boolean | true when this was a super-reaction |
| type | integer | The type of reaction |

#### Message Reaction Remove All
