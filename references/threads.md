# Threads

> Discord Developers reference. Original prose; identifiers preserved verbatim.

[Threads](https://docs.discord.com/developers/resources/channel#channel-object) are temporary sub-channels under an existing channel, useful for separating conversations in busy places. Their design closely follows [channel](https://docs.discord.com/developers/resources/channel#channel-object) objects, and this topic collects thread-related details so migration is easier.

### Backwards Compatibility

 Threads require API v9 or later. Bots that stay below API v9 will miss most thread gateway events and events that occur inside threads, such as [Message Create](https://docs.discord.com/developers/events/gateway-events#message-create). API v8 bots still receive gateway events for Interactions. Events that may be omitted include, but are not limited to:
- MESSAGE_CREATE
- MESSAGE_DELETE
- MESSAGE_DELETE_BULK
- MESSAGE_REACTION_ADD
- MESSAGE_REACTION_REMOVE
- MESSAGE_REACTION_REMOVE_ALL
- MESSAGE_REACTION_REMOVE_EMOJI
- MESSAGE_UPDATE
- THREAD_CREATE
- THREAD_UPDATE
- THREAD_DELETE
- THREAD_MEMBER_UPDATE
- THREAD_MEMBERS_UPDATE

### Thread Fields

 Threads reuse or reinterpret several fields from the [channel object](https://docs.discord.com/developers/resources/channel#channel-object):
- `id`, `guild_id`, `type`, `name`, `last_message_id`, `last_pin_timestamp`, and `rate_limit_per_user` are reused
- `owner_id` now stores the ID of the user who created the thread
- `parent_id` now stores the ID of the parent `GUILD_TEXT` or `GUILD_ANNOUNCEMENT` channel
 Threads also expose several fields that ordinary channels do not:
- `member_count` is an approximate member count capped at 50; it is used only by Discord UI and is usually not useful for bots
- `message_count` and `total_message_sent` both count thread messages. Deleting a message decreases `message_count` but does not decrease `total_message_sent`. For threads created before July 1, 2022 both counters stop at 50.
- `thread_metadata` contains thread-only fields: `archived`, `archive_timestamp`, `auto_archive_duration`, and `locked`. `archive_timestamp` changes when a thread is created, archived, unarchived, or has `auto_archive_duration` changed.

### Public & Private Threads

 Public threads are visible to users who can view the parent channel. They must start from an existing message, though they may become orphaned if that message is removed. The thread and starter message share an ID. The parent channel [type](https://docs.discord.com/developers/resources/channel#channel-object-channel-types) determines the resulting thread [type](https://docs.discord.com/developers/resources/channel#channel-object-channel-types): `GUILD_TEXT` channels [create](https://docs.discord.com/developers/resources/channel#start-thread-from-message) `PUBLIC_THREAD`, and `GUILD_ANNOUNCEMENT` channels [create](https://docs.discord.com/developers/resources/channel#start-thread-from-message) `ANNOUNCEMENT_THREAD`. Private threads are guild-scoped and behave like Group DMs. They are always [created](https://docs.discord.com/developers/resources/channel#start-thread-without-message) as `GUILD_PRIVATE_THREAD` [type](https://docs.discord.com/developers/resources/channel#channel-object-channel-types) and can only be created inside `GUILD_TEXT` channels.

### Locked Threads

 Locked threads limit users, including bot users, that lack `MANAGE_THREADS`. Such users cannot create or edit messages in locked threads or change properties such as title or tags. Some actions, including deleting messages and adding or removing reactions, are allowed in locked threads only when the thread is also active, meaning unarchived. Users or bot users with `MANAGE_THREADS` can still change the thread and its messages.

### Active & Archived Threads

 A thread is either active or archived; moving from archived to active is unarchiving. If `locked` is true, only a user with `MANAGE_THREADS` can unarchive it. Archiving reduces UI clutter and limits the active thread set that must be kept available, because archived thread counts can become large. Guilds therefore cap the number of active threads. Mutation is limited to active threads: archived threads do not allow users to edit messages, add reactions, use application commands, or join; deleting messages is the only expected operation inside one. Sending a message automatically unarchives a thread unless a moderator locked it. The gateway is built so bots can accurately track active threads, but archived threads are not synced up front. Threads do not count toward a guild's max-channels limit, though guilds do have a maximum active-thread count. Threads auto-archive after inactivity. As a server nears the max thread limit, the timer may shorten, usually not below `auto_archive_duration`; very busy channels with 7 day auto archive can archive sooner to avoid the server becoming full. Activity means sending a message, unarchiving, or changing the auto-archive time. `auto_archive_duration` previously controlled how long a thread stayed active, but now controls how long it remains in the channel list. Channels may also define `default_auto_archive_duration`, which clients use to preselect an `auto_archive_duration` when creating a thread.

### Permissions

 Threads usually inherit permissions from the parent channel; for example, permission to add reactions in the parent generally applies in the thread. Three bits are thread-specific: `CREATE_PUBLIC_THREADS`, `CREATE_PRIVATE_THREADS`, and `SEND_MESSAGES_IN_THREADS`. `SEND_MESSAGES` does nothing inside threads; speaking in a thread requires `SEND_MESSAGES_IN_THREADS`. Private threads work like guild-scoped Group DMs: viewing or participating requires an invite or moderator status through `MANAGE_THREADS`. The gateway also treats threads differently from channels, because clients are not told about threads they cannot view.

### Gateway Events

- [Guild Create](https://docs.discord.com/developers/events/gateway-events#guild-create) includes `threads`, an array of channel objects representing all active threads the current user can view in the guild.
- Creating, updating, or deleting a thread sends [Thread Create](https://docs.discord.com/developers/events/gateway-events#thread-create), [Thread Update](https://docs.discord.com/developers/events/gateway-events#thread-update), or [Thread Delete](https://docs.discord.com/developers/events/gateway-events#thread-delete). As with channel events, the payload contains the thread.
- Because the gateway syncs only active visible threads, gaining access to a channel may require syncing that channel's active threads. Discord sends a [Thread List Sync](https://docs.discord.com/developers/events/gateway-events#thread-list-sync) event for that case; the [Thread List Sync](https://docs.discord.com/developers/events/gateway-events#thread-list-sync) payload identifies the affected channels.

### Thread Membership

 Threads maintain explicit membership records. This data mainly supports two cases:
- Clients use *their own* [thread member](https://docs.discord.com/developers/resources/channel#thread-member-object) to derive read state and notification settings. Bots rarely need this directly, but it drives some of the sync behavior described here.
- Listing all users present in a thread.
 Membership is represented as an array of [thread member](https://docs.discord.com/developers/resources/channel#thread-member-object) objects. Each object has `id` for the thread ID, `user_id`, `join_timestamp`, and `flags`. The current `flags` only cover notification settings, though additional flags may be introduced later.

### Syncing for the current user

- Discord always emits [Thread Members Update](https://docs.discord.com/developers/events/gateway-events#thread-members-update) for current-user membership changes, including being added to a thread or removed from one.
- [Thread Member Update](https://docs.discord.com/developers/events/gateway-events#thread-member-update) is sent whenever the current user's [thread member](https://docs.discord.com/developers/resources/channel#thread-member-object) object changes.
- Some API calls, including archived-thread listing and search, return an array of [thread member](https://docs.discord.com/developers/resources/channel#thread-member-object) objects for returned threads that include the current user. Other calls, such as getting a channel, return the current user's [thread member](https://docs.discord.com/developers/resources/channel#thread-member-object) object as a property on the channel when applicable.
- [Guild Create](https://docs.discord.com/developers/events/gateway-events#guild-create) includes a [thread member](https://docs.discord.com/developers/resources/channel#thread-member-object) object on returned threads where the current user is a member.
- [Thread Create](https://docs.discord.com/developers/events/gateway-events#thread-create) includes a [thread member](https://docs.discord.com/developers/resources/channel#thread-member-object) object on the thread when the current user is a member and recently gained access to the parent channel.
- [Thread List Sync](https://docs.discord.com/developers/events/gateway-events#thread-list-sync) includes [thread member](https://docs.discord.com/developers/resources/channel#thread-member-object) objects for returned threads where the current user is a member.

### Syncing for other users

 These operations require the `GUILD_MEMBERS` [Gateway Intent](https://docs.discord.com/developers/events/gateway#gateway-intents).
- An API `GET` request to [`/channels/<channel_id>/thread-members`](https://docs.discord.com/developers/resources/channel#list-thread-members) returns an array of [thread member](https://docs.discord.com/developers/resources/channel#thread-member-object) objects.
- [Thread Members Update](https://docs.discord.com/developers/events/gateway-events#thread-members-update) includes users added to or removed from a thread by an action.

### Editing & Deleting Threads

 Threads use the existing channel edit and delete endpoints with `PATCH` and `DELETE`.
- Deleting a thread requires `MANAGE_THREADS`.
- Editing a thread to set `archived` to `false` only requires that the current user is already in the thread. If `locked` is true, the user must have created the thread or have `MANAGE_THREADS`.
- Editing `name`, `archived`, or `auto_archive_duration` requires `MANAGE_THREADS` or current-user ownership of the thread.
- Editing `rate_limit_per_user` or `locked` requires `MANAGE_THREADS`.

### Age-Restricted Threads

 Threads do not set `nsfw` directly. A thread inside an [age-restricted channel](https://support.discord.com/hc/en-us/articles/115000084051) inherits that setting.

### New Message Types

 Threads add some [message types](https://docs.discord.com/developers/resources/message#message-object-message-types) and reuse others:
- `RECIPIENT_ADD` and `RECIPIENT_REMOVE` are also sent when another user adds or removes someone from a thread
- `CHANNEL_NAME_CHANGE` is reused for thread name changes
- `THREAD_CREATED` is a new message in the parent `GUILD_TEXT` channel that tells users a thread was created. It is currently sent only when a `PUBLIC_THREAD` is created from an older message; the exact age remains TBD but is currently very small. The message has a [message reference](https://docs.discord.com/developers/resources/message#message-reference-structure) containing the thread's `guild_id` and `channel_id`, and its `content` is the thread `name`.
- `THREAD_STARTER_MESSAGE` is the first message in threads started from an existing parent-channel message. It contains only a [message reference](https://docs.discord.com/developers/resources/message#message-reference-structure) to the starter message.

### Enumerating threads

 Four `GET` routes enumerate threads for a channel or guild:
- [`/guilds/<guild_id>/threads/active`](https://docs.discord.com/developers/resources/guild#list-active-guild-threads) returns accessible active public and private threads in a guild
- [`/channels/<channel_id>/users/@me/threads/archived/private`](https://docs.discord.com/developers/resources/channel#list-joined-private-archived-threads) returns archived private threads in a channel where the current user is a member, sorted by descending thread ID
- [`/channels/<channel_id>/threads/archived/public`](https://docs.discord.com/developers/resources/channel#list-public-archived-threads) returns archived public threads in a channel, sorted by descending archive timestamp
- [`/channels/<channel_id>/threads/archived/private`](https://docs.discord.com/developers/resources/channel#list-private-archived-threads) returns archived private threads in a channel, sorted by descending archive timestamp

### Webhooks

 Webhooks can target threads by passing the `thread_id` query parameter. See [execute webhook](https://docs.discord.com/developers/resources/webhook#execute-webhook) for details.

### Details about Thread Access and Syncing

 Thread syncing resembles channel syncing, but [Thread List Sync](https://docs.discord.com/developers/events/gateway-events#thread-list-sync) and [Thread Create](https://docs.discord.com/developers/events/gateway-events#thread-create) have two important differences:
1. The [Gateway](https://docs.discord.com/developers/events/gateway) syncs only threads the app may view
2. The Gateway syncs threads only after the app has subscribed to the guild. In Discord's official clients, this subscription occurs when the user opens a channel in that guild.
 Those differences create several special cases.

### Thread Access

#### Gaining Access to Private Threads

 When an app is added to a private thread, it often does not already have that thread in memory because it lacked permission to view it. Private threads are synced only for members and moderators. When a user is added to a private thread, the Gateway sends [Thread Create](https://docs.discord.com/developers/events/gateway-events#thread-create), ensuring the client has a non-null thread value. `THREAD_CREATE` is also sent for moderators, even though they may already have the channel in memory.

#### Gaining Access to Public Threads

 Apps automatically subscribe to thread events and active threads when connecting to the Gateway. For non-app users, being added to a public thread before subscribing may leave the thread absent from memory, which Discord clients require. Therefore the Gateway sends [Thread Create](https://docs.discord.com/developers/events/gateway-events#thread-create) whenever a user is added to *any* thread, even if apps do not strictly need that event.

### Channel Access

#### Gaining Access to Channels

 When an app gains access to a channel, such as by receiving a moderator role, it may not have that channel's threads in memory because the Gateway only syncs visible threads. Discord sends [Thread List Sync](https://docs.discord.com/developers/events/gateway-events#thread-list-sync) to address this. The [Thread List Sync](https://docs.discord.com/developers/events/gateway-events#thread-list-sync) event includes `channel_ids`, the IDs of channels whose threads are being synced. Use it to clear active threads whose `parent_id` is in `channel_ids`, then ingest the event's thread list.

#### Losing Access to Channels

 When an app loses channel access, the Gateway does **not** send [Thread Delete](https://docs.discord.com/developers/events/gateway-events#thread-delete) or an equivalent thread event. Instead, the app receives the permission-changing event. Tracking lost thread access is possible but difficult because every permission-change path must be handled correctly. Common causes are [Guild Role Update](https://docs.discord.com/developers/events/gateway-events#guild-role-update), [Guild Member Update](https://docs.discord.com/developers/events/gateway-events#guild-member-update), and [Channel Update](https://docs.discord.com/developers/events/gateway-events#channel-update). Discord clients check permissions *before* acting so stale data does not cause invalid actions. If a user or app loses channel access, it is not removed from the thread and remains reported as a thread member, but it receives no new Gateway events unless removed from the thread, which sends [Thread Members Update](https://docs.discord.com/developers/events/gateway-events#thread-members-update).

### Unarchiving a Thread

 When a thread is unarchived, an app may lack the thread or its membership state in memory. The Gateway therefore sends two events in this order:
1. [Thread Update](https://docs.discord.com/developers/events/gateway-events#thread-update), containing the complete channel object.
2. [Thread Member Update](https://docs.discord.com/developers/events/gateway-events#thread-member-update), sent to every member of the unarchived thread. Discord clients load only active threads at startup, so this event is sent even when many apps may not need it.

### Forums

 A `GUILD_FORUM` (type `15`) channel behaves like a `GUILD_TEXT` channel except that it allows only threads. Unless a detail says otherwise, forum-channel threads use the same endpoints and receive the same Gateway events as text-channel threads. Messages cannot be sent directly to forum channels. For how forum channels appear in Discord, see the [Forum Channels FAQ](https://support.discord.com/hc/en-us/articles/6208479917079-Forum-Channels-FAQ#h_01G69FJQWTWN88HFEHK7Z6X79N).

### Media Channels

 A `GUILD_MEDIA` (type `16`) channel resembles `GUILD_FORUM` because only threads can be created there. Unless otherwise specified, media-channel threads behave like forum-channel threads, using the same endpoints and receiving the same Gateway events. See the [media channels Help Center Article](https://creator-support.discord.com/hc/en-us/articles/14346342766743) for user-facing behavior. `GUILD_MEDIA` channels are beta and still under active development, so API behavior and technical details may change.

### Creating Threads in Forum and Media Channels

 In thread-only channels, threads appear as posts. Create them with [`POST /channels/<channel_id>/threads`](https://docs.discord.com/developers/resources/channel#start-thread-in-forum-or-media-channel), using [parameters](https://docs.discord.com/developers/resources/channel#start-thread-in-forum-or-media-channel-jsonform-params) that differ slightly from text-channel thread creation. For example, creating a thread in a thread-only channel also creates a message with the same ID as the thread, so the request must include both thread and message parameters. These threads inherit parent-channel permissions like text-channel threads, except creating one in a thread-only channel requires only `SEND_MESSAGES`.

### Forum and Media Channel Fields

 Several fields specific to thread-only channels are worth noting:
- `last_message_id` is the ID of the newest thread created in that channel. As with messages, Discord does not send `CHANNEL_UPDATE` when it changes, so apps should update it from [Thread Create](https://docs.discord.com/developers/events/gateway-events#thread-create).
- `topic` is displayed as the Guidelines section in the Discord client.
- `rate_limit_per_user` controls how often threads can be created. Thread-only channels also have `default_thread_rate_limit_per_user`, which limits how frequently messages may be sent *inside a thread* and is copied to `rate_limit_per_user` when the thread is created.
- `available_tags` can be set during channel creation or update and determines which tags may be placed on individual threads through `applied_tags`.
- `flags` contains [channel flags](https://docs.discord.com/developers/resources/channel#channel-object-channel-flags) for a thread-only channel. Currently `REQUIRE_TAG` is the only supported value; it requires choosing a tag from `available_tags` when creating a thread.
 All channel fields, including fields for thread-only channels, are documented in the [Channel Object](https://docs.discord.com/developers/resources/channel#channel-object).

### Forum and Media Channel Thread Fields

 A thread can be pinned in a thread-only channel, represented by the [`PINNED` flag](https://docs.discord.com/developers/resources/channel#channel-object-channel-flags) in the thread's `flags` field. A pinned thread has `(1 << 1)` set, and archiving it unsets that flag. Pinned threads do *not* auto-archive. For threads in thread-only channels, `message_count` and `total_message_sent` increase on `MESSAGE_CREATE` and decrease on `MESSAGE_DELETE` and `MESSAGE_DELETE_BULK`. No dedicated `CHANNEL_UPDATE` reports those counter changes, so apps should update them from the corresponding events. The full thread field set for thread-only channels is in the [channel resources documentation](https://docs.discord.com/developers/resources/channel#start-thread-in-forum-or-media-channel-jsonform-params).

### Forum and Media Channel Formatting

 In thread-only channels, both the first thread message and the channel topic may use markdown for headings and bulleted lists, unlike text channels.
