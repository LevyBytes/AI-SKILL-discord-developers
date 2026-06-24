# Channel (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

### Channel Object

 Describes a Discord channel, either in a guild or in DMs.

###### Channel Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | the id of this channel |
| type | integer | the type of channel |
| guild_id? | snowflake | guild ID; this can be absent on some channel objects delivered through gateway guild dispatches |
| position? | integer | channel ordering position; when positions match, channels are ordered by id |
| permission_overwrites? | array of overwrite objects | permission overwrites explicitly set for members and roles |
| name? | ?string | channel name; 1-100 characters |
| topic? | ?string | channel topic; 0-4096 characters for `GUILD_FORUM` and `GUILD_MEDIA`, and 0-1024 characters for other channel types |
| nsfw? | boolean | whether the channel has age restrictions enabled |
| last_message_id? | ?snowflake | ID of the most recent message in this channel, or thread for `GUILD_FORUM` or `GUILD_MEDIA`; it might not identify a valid existing message or thread |
| bitrate? | integer | voice channel bitrate, measured in bits per second |
| user_limit? | integer | maximum number of users allowed in the voice channel |
| rate_limit_per_user?* | integer | seconds a user must wait before sending another message, from 0-21600; bots and users with `BYPASS_SLOWMODE` are exempt |
| recipients? | array of user objects | the recipients of the DM |
| icon? | ?string | group DM icon hash |
| owner_id? | snowflake | ID of the user who created the group DM or thread |
| application_id? | snowflake | application ID for the group DM creator when a bot created it |
| managed? | boolean | for group DM channels, whether an application manages the channel through the `gdm.join` OAuth2 scope |
| parent_id? | ?snowflake | for guild channels, parent category ID, where each parent category can hold up to 50 channels; for threads, ID of the text channel where the thread was created |
| last_pin_timestamp? | ?ISO8601 timestamp | timestamp for the most recent pinned message; may be `null` in events such as `GUILD_CREATE` when no message is pinned |
| rtc_region? | ?string | voice channel region ID; null means automatic |
| video_quality_mode? | integer | voice channel camera video quality mode; value is 1 if omitted |
| message_count?** | integer | thread message count, excluding the first message and deleted messages |
| member_count? | integer | approximate thread user count, capped at 50 |
| thread_metadata? | a thread metadata object | thread-only fields that other channel types do not need |
| member? | a thread member object | thread member object for the current user when that user has joined; included only by some API endpoints |
| default_auto_archive_duration? | integer | default minutes copied to new threads for auto-archive behavior; after that inactive period, threads stop appearing in the channel list; allowed values are 60, 1440, 4320, 10080 |
| permissions? | string | permissions computed for the invoking user in this channel, including overwrites; present only inside `resolved` interaction data and does not include implicit permissions, which may need a separate check |
| flags? | integer | channel flags represented as a combined bitfield |
| total_message_sent? | integer | total number of messages ever sent in a thread; this resembles `message_count` at message creation time but does not decrease after deletion |
| available_tags? | array of tag objects | tags available for use in a `GUILD_FORUM` or `GUILD_MEDIA` channel |
| applied_tags? | array of snowflakes | IDs for tags applied to a thread in a `GUILD_FORUM` or `GUILD_MEDIA` channel |
| default_reaction_emoji? | ?default reaction object | emoji displayed on the add reaction button for a thread in a `GUILD_FORUM` or `GUILD_MEDIA` channel |
| default_thread_rate_limit_per_user? | integer | initial `rate_limit_per_user` assigned to newly created threads in a channel; the value is copied when the thread is created and does not update live |
| default_sort_order? | ?integer | default sort order type for posts in `GUILD_FORUM` and `GUILD_MEDIA`; defaults to `null`, meaning no channel admin has chosen a preferred sort order |
| default_forum_layout? | integer | default forum layout view for posts in `GUILD_FORUM`; defaults to `0`, meaning no channel admin has set a layout view |

 * `rate_limit_per_user` also controls thread creation. During each `rate_limit_per_user` interval, users may send one message and create one thread. ** For threads created before July 1, 2022, message counts above 50 are inaccurate.

###### Channel Types

 Types 10, 11, and 12 are available only in API v9 or later.
| Type | ID | Description |
| --- | --- | --- |
| GUILD_TEXT | 0 | text channel in a server |
| DM | 1 | a direct message between users |
| GUILD_VOICE | 2 | voice channel in a server |
| GROUP_DM | 3 | direct message shared by multiple users |
| GUILD_CATEGORY | 4 | category used to organize up to 50 channels |
| GUILD_ANNOUNCEMENT | 5 | followable channel whose posts can be crossposted into another server; formerly news channels |
| ANNOUNCEMENT_THREAD | 10 | temporary sub-channel inside a GUILD_ANNOUNCEMENT channel |
| PUBLIC_THREAD | 11 | temporary sub-channel inside a GUILD_TEXT or GUILD_FORUM channel |
| PRIVATE_THREAD | 12 | temporary sub-channel inside a GUILD_TEXT channel, visible only to invited users and users with MANAGE_THREADS |
| GUILD_STAGE_VOICE | 13 | voice-style channel for audience events |
| GUILD_DIRECTORY | 14 | hub channel that contains the listed servers |
| GUILD_FORUM | 15 | channel that contains only threads |
| GUILD_MEDIA | 16 | channel that contains only threads and behaves similarly to `GUILD_FORUM` channels |

 * `GUILD_MEDIA` is still actively being developed. Do not build against behavior that is not documented here, because it can change without notice.

###### Video Quality Modes

| Mode | Value | Description |
| --- | --- | --- |
| AUTO | 1 | Discord selects the quality level for best performance |
| FULL | 2 | 720p |

###### Channel Flags

| Flag | Value | Description |
| --- | --- | --- |
| PINNED | `1 << 1` | thread is pinned at the top of its parent `GUILD_FORUM` or `GUILD_MEDIA` channel |
| REQUIRE_TAG | `1 << 4` | whether creating a thread in a `GUILD_FORUM` or `GUILD_MEDIA` channel requires specifying a tag; tags are supplied through `applied_tags` |
| HIDE_MEDIA_DOWNLOAD_OPTIONS | `1 << 15` | hides embedded media download options when set; supported only for media channels |

###### Sort Order Types

| Flag | Value | Description |
| --- | --- | --- |
| LATEST_ACTIVITY | 0 | Sort forum posts by activity |
| CREATION_DATE | 1 | order forum posts by creation time, newest first |

###### Forum Layout Types

| Flag | Value | Description |
| --- | --- | --- |
| NOT_SET | 0 | forum channel has no configured default |
| LIST_VIEW | 1 | Display posts as a list |
| GALLERY_VIEW | 2 | show posts as tile-style collection |

###### Example Guild Text Channel

```
{
  "id": "41771983423143937",
  "guild_id": "41771983423143937",
  "name": "general",
  "type": 0,
  "position": 6,
  "permission_overwrites": [],
  "rate_limit_per_user": 2,
  "nsfw": true,
  "topic": "24/7 chat about how to gank Mike #2",
  "last_message_id": "155117677105512449",
  "parent_id": "399942396007890945",
  "default_auto_archive_duration": 60
}

```

###### Example Guild Announcement Channel

 Bots with the required permissions can post or publish messages in this channel type.

```
{
  "id": "41771983423143937",
  "guild_id": "41771983423143937",
  "name": "important-news",
  "type": 5,
  "position": 6,
  "permission_overwrites": [],
  "nsfw": true,
  "topic": "Rumors about Half Life 3",
  "last_message_id": "155117677105512449",
  "parent_id": "399942396007890945",
  "default_auto_archive_duration": 60
}

```

###### Example Guild Voice Channel

```
{
  "id": "155101607195836416",
  "last_message_id": "174629835082649376",
  "type": 2,
  "name": "ROCKET CHEESE",
  "position": 5,
  "parent_id": null,
  "bitrate": 64000,
  "user_limit": 0,
  "rtc_region": null,
  "guild_id": "41771983423143937",
  "permission_overwrites": [],
  "rate_limit_per_user": 0,
  "nsfw": false,
}

```

###### Example DM Channel

```
{
  "last_message_id": "3343820033257021450",
  "type": 1,
  "id": "319674150115610528",
  "recipients": [
    {
      "username": "test",
      "discriminator": "9999",
      "id": "82198898841029460",
      "avatar": "33ecab261d4681afa4d85a04691c4a01"
    }
  ]
}

```

###### Example Group DM Channel

```
{
  "name": "Some test channel",
  "icon": null,
  "recipients": [
    {
      "username": "test",
      "discriminator": "9999",
      "id": "82198898841029460",
      "avatar": "33ecab261d4681afa4d85a04691c4a01"
    },
    {
      "username": "test2",
      "discriminator": "9999",
      "id": "82198810841029460",
      "avatar": "33ecab261d4681afa4d85a10691c4a01"
    }
  ],
  "last_message_id": "3343820033257021450",
  "type": 3,
  "id": "319674150115710528",
  "owner_id": "82198810841029460"
}

```

###### Example Channel Category

```
{
  "permission_overwrites": [],
  "name": "Test",
  "parent_id": null,
  "nsfw": false,
  "position": 0,
  "guild_id": "290926798629997250",
  "type": 4,
  "id": "399942396007890945"
}

```

###### Example Thread Channel

 [Threads](https://docs.discord.com/developers/topics/threads) may be `archived` or `active`. Archived threads are generally not mutable; before sending a message or adding a reaction, the thread must be unarchived. The API automatically unarchives a thread when a message is sent in that thread. Unlike ordinary channel updates, thread updates are synced only to users who can view the affected threads. A [guild create](https://docs.discord.com/developers/events/gateway-events#guild-create) payload includes only active threads visible to the current user. Threads in private channels remain private to members of that channel. Therefore, when access to a channel is gained, the API sends a [thread list sync](https://docs.discord.com/developers/events/gateway-events#thread-list-sync) containing all active threads in that channel. Threads also have membership: users must be added before they can send messages there. Sending a message in a thread automatically adds the user to that thread. Guilds limit both active thread count and thread member count. After those limits are reached, new threads cannot be created or unarchived, and additional users cannot be added. Threads do not count toward the per-guild channel limit. The [threads](https://docs.discord.com/developers/topics/threads) topic provides additional detail.

```
{
  "id": "41771983423143937",
  "guild_id": "41771983423143937",
  "parent_id": "41771983423143937",
  "owner_id": "41771983423143937",
  "name": "don't buy dota-2",
  "type": 11,
  "last_message_id": "155117677105512449",
  "message_count": 1,
  "member_count": 5,
  "rate_limit_per_user": 2,
  "thread_metadata": {
    "archived": false,
    "auto_archive_duration": 1440,
    "archive_timestamp": "2021-04-12T23:40:39.855793+00:00",
    "locked": false
  },
  "total_message_sent": 1
}

```

### Followed Channel Object

###### Followed Channel Structure

| Field | Type | Description |
| --- | --- | --- |
| channel_id | snowflake | source channel id |
| webhook_id | snowflake | created target webhook id |

### Overwrite Object

 For details on the `allow` and `deny` fields, see [permissions](https://docs.discord.com/developers/topics/permissions#permissions).

###### Overwrite Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | role or user id |
| type | int | either 0 (role) or 1 (member) |
| allow | string | permission bit set |
| deny | string | permission bit set |

### Thread Metadata Object

 The thread metadata object holds channel fields that apply only to threads.

###### Thread Metadata Structure

| Field | Type | Description |
| --- | --- | --- |
| archived | boolean | whether the thread is archived |
| auto_archive_duration | integer | after `auto_archive_duration` minutes with no activity, the thread stops appearing in the channel list; supported values are 60, 1440, 4320, 10080 |
| archive_timestamp | ISO8601 timestamp | timestamp of the most recent archive status change, used to calculate recent activity |
| locked | boolean | whether the thread is locked; locked threads can be unarchived only by users with MANAGE_THREADS |
| invitable? | boolean | whether non-moderators may add other non-moderators to the thread; available only for private threads |
| create_timestamp? | ?ISO8601 timestamp | thread creation timestamp; populated only for threads created after 2022-01-09 |

### Thread Member Object

 A thread member object describes a user who has joined a thread.

###### Thread Member Structure

| Field | Type | Description |
| --- | --- | --- |
| id? * | snowflake | ID of the thread |
| user_id? * | snowflake | ID of the user |
| join_timestamp | ISO8601 timestamp | time when the user most recently joined the thread |
| flags | integer | user-specific thread settings; currently used only for notifications |
| member? * ** | guild member object | Additional information about the user |

 * These fields are not included on the member embedded in each thread in the [GUILD_CREATE](https://docs.discord.com/developers/events/gateway-events#guild-create) event. ** The `member` field appears only when `with_member` is `true` for [List Thread Members](https://docs.discord.com/developers/resources/channel#list-thread-members) or [Get Thread Member](https://docs.discord.com/developers/resources/channel#get-thread-member).

### Default Reaction Object

 Object specifying the emoji used as the default reaction option for a forum post. Exactly one of `emoji_id` or `emoji_name` must be provided.

###### Default Reaction Structure

| Field | Type | Description |
| --- | --- | --- |
| emoji_id | ?snowflake | ID of a guild custom emoji |
| emoji_name | ?string | unicode emoji character |

### Forum Tag Object

 Object representing a tag that can be applied to a thread in a `GUILD_FORUM` or `GUILD_MEDIA` channel.

###### Forum Tag Structure

 When updating a `GUILD_FORUM` or `GUILD_MEDIA` channel, each tag object in `available_tags` only needs `name`.
| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | the id of the tag |
| name | string | tag name; 0-20 characters |
| moderated | boolean | whether only members with `MANAGE_THREADS` can add or remove this tag on threads |
| emoji_id | ?snowflake | ID of a guild custom emoji * |
| emoji_name | ?string | unicode emoji character * |

 * No more than one of `emoji_id` and `emoji_name` may contain a non-null value.

### Get Channel

  Retrieve the channel identified by the supplied ID. The response is a [channel](https://docs.discord.com/developers/resources/channel#channel-object) object, with a [thread member](https://docs.discord.com/developers/resources/channel#thread-member-object) object included when the target is a thread.

### Modify Channel

  Change a channel's settings. On success, returns a [channel](https://docs.discord.com/developers/resources/channel#channel-object); invalid parameters return 400 BAD REQUEST. Every parameter for this endpoint is optional. This endpoint supports the `X-Audit-Log-Reason` header.

###### JSON Params (Group DM)

 Triggers a [Channel Update](https://docs.discord.com/developers/events/gateway-events#channel-update) Gateway event.
| Field | Type | Description |
| --- | --- | --- |
| name | string | 1-100 character channel name |
| icon | binary | base64 encoded icon |

###### JSON Params (Guild channel)

 Requires `MANAGE_CHANNELS` in the guild. Triggers a [Channel Update](https://docs.discord.com/developers/events/gateway-events#channel-update) Gateway event. If the modified channel is a category, each changed child channel also receives its own [Channel Update](https://docs.discord.com/developers/events/gateway-events#channel-update) event. Modifying permission overwrites requires `MANAGE_ROLES`. A bot can allow or deny only permissions it already has in the guild or parent channel, when applicable, unless it has a `MANAGE_ROLES` overwrite in the channel.
| Field | Type | Description | Channel Type |
| --- | --- | --- | --- |
| name | string | 1-100 character channel name | All |
| type | integer | channel type; only text-to-announcement or announcement-to-text conversion is supported, and only in guilds with the "NEWS" feature | Text, Announcement |
| position | ?integer | channel position in the left-side list; matching positions are sorted by id | All |
| topic | ?string | channel topic with 0-1024 characters, or 0-4096 characters for `GUILD_FORUM` and `GUILD_MEDIA` channels | Text, Announcement, Forum, Media |
| nsfw | ?boolean | whether the channel has age restrictions enabled | Text, Voice, Announcement, Stage, Forum, Media |
| rate_limit_per_user | ?integer | seconds a user must wait before another message, from 0-21600; bots and users with `BYPASS_SLOWMODE` are exempt | Text, Voice, Stage, Forum, Media |
| bitrate* | ?integer | voice or stage channel bitrate in bits per second; minimum 8000 | Voice, Stage |
| user_limit | ?integer | voice or stage channel user limit; maximum 99 for voice channels and 10,000 for stage channels, with 0 meaning unlimited | Voice, Stage |
| permission_overwrites** | ?array of partial overwrite objects | channel or category-specific permissions | All |
| parent_id | ?snowflake | ID of the channel's new parent category | Text, Voice, Announcement, Stage, Forum, Media |
| rtc_region | ?string | voice channel region ID; null means automatic | Voice, Stage |
| video_quality_mode | ?integer | camera video quality mode for the voice channel | Voice, Stage |
| default_auto_archive_duration | ?integer | default client-side duration, in minutes, used for newly created threads in the channel before auto-archiving after recent activity | Text, Announcement, Forum, Media |
| flags | integer | channel flags as a combined bitfield. Currently, `GUILD_FORUM` and `GUILD_MEDIA` support only `REQUIRE_TAG` (`1 << 4`). `HIDE_MEDIA_DOWNLOAD_OPTIONS` (`1 << 15`) is supported only by `GUILD_MEDIA` channels | Forum, Media |
| available_tags | array of tag objects | tags usable in a `GUILD_FORUM` or `GUILD_MEDIA` channel; maximum 20 | Forum, Media |
| default_reaction_emoji | ?default reaction object | emoji shown on the add reaction button for a thread in a `GUILD_FORUM` or `GUILD_MEDIA` channel | Forum, Media |
| default_thread_rate_limit_per_user | integer | initial `rate_limit_per_user` assigned to newly created threads in a channel; the value is copied when the thread is created and does not update live | Text, Forum, Media |
| default_sort_order | ?integer | default sort order type for posts in `GUILD_FORUM` and `GUILD_MEDIA` channels | Forum, Media |
| default_forum_layout | integer | default forum layout type for displaying posts in `GUILD_FORUM` channels | Forum |

 * Normal servers can set voice bitrate up to 96000, Boost level 1 servers up to 128000, Boost level 2 servers up to 256000, and Boost level 3 servers, or servers with the `VIP_REGIONS` [guild feature](https://docs.discord.com/developers/resources/guild#guild-object-guild-features), up to 384000. Stage channel bitrate can be up to 64000. ** In each overwrite object, `allow` and `deny` may be omitted or set to `null`; both cases default to `"0"`.

###### JSON Params (Thread)

 Setting `archived` to `false` requires only `SEND_MESSAGES` when `locked` is also `false`; otherwise `MANAGE_THREADS` is required. Triggers a [Thread Update](https://docs.discord.com/developers/events/gateway-events#thread-update) Gateway event. The thread must already have `archived` set to `false`, or the request must set it to `false`.
| Field | Type | Description |
| --- | --- | --- |
| name | string | 1-100 character channel name |
| archived | boolean | whether the thread is archived |
| auto_archive_duration | integer | after `auto_archive_duration` minutes without activity, the thread stops appearing in the channel list; allowed values are 60, 1440, 4320, 10080 |
| locked | boolean | whether the thread is locked; locked threads can be unarchived only by users with MANAGE_THREADS |
| invitable | boolean | whether non-moderators may add other non-moderators to the thread; available only for private threads |
| rate_limit_per_user | ?integer | seconds a user must wait before sending another message, from 0-21600; bots and users with `BYPASS_SLOWMODE` are exempt |
| flags? | integer | channel flags as a combined bitfield; `PINNED` can be set only on threads in forum and media channels |
| applied_tags? | array of snowflakes | IDs of tags applied to a thread in a `GUILD_FORUM` or `GUILD_MEDIA` channel; maximum 5 |

### Set Voice Channel Status

  Set the status for a voice channel. Requires `SET_VOICE_CHANNEL_STATUS`; if the current user is not connected to that voice channel, `MANAGE_CHANNELS` is also required. Triggers a [Voice Channel Status Update](https://docs.discord.com/developers/events/gateway-events#voice-channel-status-update) Gateway event. This endpoint supports the `X-Audit-Log-Reason` header.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| status | ?string | new voice channel status, up to 500 characters |

### Delete/Close Channel

  Remove a channel, or close a private message conversation. Guild channels require `MANAGE_CHANNELS`; threads require `MANAGE_THREADS`. When a category is removed, its child channels stay in place, lose their `parent_id`, and each emits a [Channel Update](https://docs.discord.com/developers/events/gateway-events#channel-update) Gateway event. A successful call returns a [channel](https://docs.discord.com/developers/resources/channel#channel-object) object and emits [Channel Delete](https://docs.discord.com/developers/events/gateway-events#channel-delete), or [Thread Delete](https://docs.discord.com/developers/events/gateway-events#thread-delete) for a thread target. Guild channel removal is irreversible, so use it carefully. Private messages can be restored by opening the conversation with the recipient again. The Rules or Guidelines channel and Community Updates channel in Community guilds are protected from deletion. This endpoint supports the `X-Audit-Log-Reason` header.

### Edit Channel Permissions

  Edit permission overwrites for a user or role in a channel. This is available only for guild channels and requires `MANAGE_ROLES`. A bot can allow or deny only permissions it has in the guild or parent channel, if applicable, unless it has a `MANAGE_ROLES` overwrite in the channel. Success returns an empty 204 response. Triggers a [Channel Update](https://docs.discord.com/developers/events/gateway-events#channel-update) Gateway event. For permission details, see [permissions](https://docs.discord.com/developers/topics/permissions#permissions). This endpoint supports the `X-Audit-Log-Reason` header.

###### JSON Params
