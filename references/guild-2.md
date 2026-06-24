# Guild (part 2)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

```
{
  "guild_id": "960007075288915998",
  "prompts": [
    {
      "id": "1067461047608422473",
      "title": "What do you want to do in this community?",
      "options": [
        {
          "id": "1067461047608422476",
          "title": "Chat with Friends",
          "description": "",
          "emoji": {
            "id": "1070002302032826408",
            "name": "chat",
            "animated": false
          },
          "role_ids": [],
          "channel_ids": [
            "962007075288916001"
          ]
        },
        {
          "id": "1070004843541954678",
          "title": "Get Gud",
          "description": "We have excellent teachers!",
          "emoji": {
            "id": null,
            "name": "😀",
            "animated": false
          },
          "role_ids": [
            "982014491980083211"
          ],
          "channel_ids": []
        }
      ],
      "single_select": false,
      "required": false,
      "in_onboarding": true,
      "type": 0
    }
  ],
  "default_channel_ids": [
    "998678771706110023",
    "998678693058719784",
    "1070008122577518632",
    "998678764340912138",
    "998678704446263309",
    "998678683592171602",
    "998678699715067986"
  ],
  "enabled": true
}

```

### Membership Screening Object

 For guilds that enable [Membership Screening](https://support.discord.com/hc/en-us/articles/1500000466882), a joining member still triggers [Guild Member Add](https://docs.discord.com/developers/events/gateway-events#guild-member-add), starts with limited server access, and has `pending` set to true on the [member object](https://docs.discord.com/developers/resources/guild#guild-member-object). After that member finishes screening, Discord emits [Guild Member Update](https://docs.discord.com/developers/events/gateway-events#guild-member-update) and changes `pending` to false. Discord is substantially revising the Membership Screening API for reading and editing the Membership Screening object, so the old documentation was removed. The pending-member behavior described here is **not changing**.

### Incidents Data Object

###### Incidents Data Structure

| Field | Type | Description |
| --- | --- | --- |
| invites_disabled_until | ?ISO8601 timestamp | when invites get enabled again |
| dms_disabled_until | ?ISO8601 timestamp | timestamp for once direct messages get enabled again |
| dm_spam_detected_at? | ?ISO8601 timestamp | timestamp for once the dm spam was detected |
| raid_detected_at? | ?ISO8601 timestamp | when the raid was detected |

###### Example Incidents Data

```
{
  "invites_disabled_until": "2023-09-01T14:48:02.222000+00:00",
  "dms_disabled_until": null
}

```

### Get Guild

  Responds using the [guild](https://docs.discord.com/developers/resources/guild#guild-object) object record for the supplied id. Once `with_counts` is set to `true`, this route can also provide `approximate_member_count` and `approximate_presence_count` for the guild.

###### Query String Params

| Field | Type | Description | Required | Default |
| --- | --- | --- | --- | --- |
| with_counts? | boolean | timestamp for once `true`, returns approximate member and presence counts for the guild | false | false |

###### Example Response

```
{
  "id": "2909267986263572999",
  "name": "Mason's Test Server",
  "icon": "389030ec9db118cb5b85a732333b7c98",
  "description": null,
  "splash": "75610b05a0dd09ec2c3c7df9f6975ea0",
  "discovery_splash": null,
  "approximate_member_count": 2,
  "approximate_presence_count": 2,
  "features": [
    "INVITE_SPLASH",
    "VANITY_URL",
    "COMMERCE",
    "BANNER",
    "NEWS",
    "VERIFIED",
    "VIP_REGIONS"
  ],
  "emojis": [
    {
      "name": "ultrafastparrot",
      "roles": [],
      "id": "393564762228785161",
      "require_colons": true,
      "managed": false,
      "animated": true,
      "available": true
    }
  ],
  "banner": "5c3cb8d1bc159937fffe7e641ec96ca7",
  "owner_id": "53908232506183680",
  "application_id": null,
  "region": null,
  "afk_channel_id": null,
  "afk_timeout": 300,
  "system_channel_id": null,
  "widget_enabled": true,
  "widget_channel_id": "639513352485470208",
  "verification_level": 0,
  "roles": [
    {
      "id": "2909267986263572999",
      "name": "@everyone",
      "permissions": "49794752",
      "position": 0,
      "color": 0,
      "colors": {
        "primary_color": 0,
        "secondary_color": null,
        "tertiary_color": null
      },
      "hoist": false,
      "managed": false,
      "mentionable": false
    }
  ],
  "default_message_notifications": 1,
  "mfa_level": 0,
  "explicit_content_filter": 0,
  "max_presences": null,
  "max_members": 250000,
  "max_video_channel_users": 25,
  "vanity_url_code": "no",
  "premium_tier": 0,
  "premium_subscription_count": 0,
  "system_channel_flags": 0,
  "preferred_locale": "en-US",
  "rules_channel_id": null,
  "public_updates_channel_id": null,
  "safety_alerts_channel_id": null
}

```

### Get Guild Preview

  Responds using the [guild preview](https://docs.discord.com/developers/resources/guild#guild-preview-object) object record for the supplied id. Once the user account is not within the guild, then that server must be [discoverable](https://docs.discord.com/developers/resources/guild#guild-object-guild-features).

### Modify Guild

  Updates a guild’s settings. The caller needs the `MANAGE_GUILD` permission. Responds using the updated [guild](https://docs.discord.com/developers/resources/guild#guild-object) object record after a successful request. Emits a [Guild Update](https://docs.discord.com/developers/events/gateway-events#guild-update) Gateway event. Every parameter on this route is optional. The `X-Audit-Log-Reason` header is accepted by this endpoint. Adding or removing the `COMMUNITY` [guild feature](https://docs.discord.com/developers/resources/guild#guild-object-guild-features) needs the `ADMINISTRATOR` permission.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name | string | guild name |
| region | ?string | guild voice region id (deprecated) |
| verification_level | ?integer | verification level |
| default_message_notifications | ?integer | default message notification level |
| explicit_content_filter | ?integer | explicit content filter level |
| afk_channel_id | ?snowflake | id for afk channel |
| afk_timeout | integer | afk timeout in seconds, may be set to: 60, 300, 900, 1800, 3600 |
| icon | ?image data | Base64 1024x1024 png/jpeg/gif image for that server icon (can be animated gif timestamp for once the server has the `ANIMATED_ICON` feature) |
| splash | ?image data | Base64 16:9 png/jpeg image for that server splash (timestamp for once the server has the `INVITE_SPLASH` feature) |
| discovery_splash | ?image data | Base64 16:9 png/jpeg image for that server discovery splash (timestamp for once the server has the `DISCOVERABLE` feature) |
| banner | ?image data | Base64 16:9 png/jpeg image for that server banner (timestamp for once the server has the `BANNER` feature; may be animated gif timestamp for once the server has the `ANIMATED_BANNER` feature) |
| system_channel_id | ?snowflake | Channel id used for server notices such as welcome messages and boost events are posted |
| system_channel_flags | integer | system channel flags |
| rules_channel_id | ?snowflake | Channel id used for Community guilds display rules and/or guidelines |
| public_updates_channel_id | ?snowflake | Channel id used for admins and moderators of Community guilds get notices from Discord |
| preferred_locale | ?string | Preferred locale of a Community server used in server discovery and notices from Discord; default is “en-US” |
| features | array of guild feature strings | enabled guild features |
| description | ?string | the description for the guild |
| premium_progress_bar_enabled | boolean | Indicates whether the guild’s boost progress bar must be enabled |
| safety_alerts_channel_id | ?snowflake | Channel id used for admins and moderators of Community guilds get safety alerts from Discord |

### Get Guild Channels

  Responds with a list of server [channel](https://docs.discord.com/developers/resources/channel#channel-object) objects. Thread records are excluded.

### Create Guild Channel

  Creates a new [channel](https://docs.discord.com/developers/resources/channel#channel-object) object record for the guild. The caller needs the `MANAGE_CHANNELS` permission. Once setting permission overwrites, only permissions your bot has within that server may be allowed/denied. Setting `MANAGE_ROLES` permission in channel records is only possible for server administrators. Responds using the new [channel](https://docs.discord.com/developers/resources/channel#channel-object) object record after a successful request. Emits a [Channel Create](https://docs.discord.com/developers/events/gateway-events#channel-create) Gateway event. Every parameter on this route is optional and nullable excluding `name`. The `X-Audit-Log-Reason` header is accepted by this endpoint.

###### JSON Params

| Field | Type | Description | Channel Type |
| --- | --- | --- | --- |
| name | string | channel name (1-100 characters) | All |
| type | integer | the type of channel | All |
| topic | string | channel topic (0-1024 characters) | Text, Announcement, Forum, Media |
| bitrate* | integer | Voice or stage bitrate, measured in bits per second; min 8000 | Voice, Stage |
| user_limit | integer | User limit for the voice channel | Voice, Stage |
| rate_limit_per_user | integer | amount of seconds one user has to wait before sending another message (0-21600); bots, as a well as user accounts using the permission `BYPASS_SLOWMODE`, are unaffected | Text, Voice, Stage, Forum, Media |
| position | integer | Sort position for that channel (channels using the same position are sorted by id) | All |
| permission_overwrites** | array of partial overwrite objects | the channel’s permission overwrites | All |
| parent_id | snowflake | ID for the parent category for a channel | Text, Voice, Announcement, Stage, Forum, Media |
| nsfw | boolean | Indicates whether that channel is age-restricted | Text, Voice, Announcement, Stage, Forum |
| rtc_region | string | Voice region ID for voice or stage channels; `null` selects automatic routing | Voice, Stage |
| video_quality_mode | integer | Camera video quality mode for the voice channel | Voice, Stage |
| default_auto_archive_duration | integer | Client-side default archive duration, in minutes, for new threads created within the channel, in minutes, to automatically archive the thread after recent activity | Text, Announcement, Forum, Media |
| default_reaction_emoji | default reaction object | Emoji displayed by the add reaction button for a thread in a `GUILD_FORUM` or a `GUILD_MEDIA` channel | Forum, Media |
| available_tags | array of tag objects | Set of tags that may be used in a `GUILD_FORUM` or a `GUILD_MEDIA` channel | Forum, Media |
| default_sort_order | integer | Default sort order type used for order posts in `GUILD_FORUM` and `GUILD_MEDIA` channels | Forum, Media |
| default_forum_layout | integer | Default forum layout view used for display posts in `GUILD_FORUM` channels | Forum |
| default_thread_rate_limit_per_user | integer | Initial `rate_limit_per_user` to set on newly created threads in one channel. this field is copied to that thread at creation time and does not live update. | Text, Announcement, Forum, Media |

 * For voice channels, normal servers can set bitrate up to 96000, servers using Boost level 1 can set up to 128000, servers using Boost level 2 can set up to 256000, plus servers with Boost level 3 or this `VIP_REGIONS` [guild feature](https://docs.discord.com/developers/resources/guild#guild-object-guild-features) may set up to 384000. For stage channels, bitrate may be set up to 64000. ** In each overwrite object, this `allow` and `deny` keys may be omitted or set to `null`, which both default to `"0"`.

### Modify Guild Channel Positions

  Updates the positions of a set of [channel](https://docs.discord.com/developers/resources/channel#channel-object) object records for the guild. The caller needs `MANAGE_CHANNELS` permission. Responds with a 204 empty response after a successful request. Emits multiple [Channel Update](https://docs.discord.com/developers/events/gateway-events#channel-update) Gateway events. Only channel records to be modified are required. This route takes a JSON list of parameters within the following format:

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | channel id |
| position? | ?integer | Sort position for that channel (channels using the same position are sorted by id) |
| lock_permissions? | ?boolean | Synchronizes permission overwrites using the new parent, once moving to a new category |
| parent_id? | ?snowflake | New parent ID for that channel that is moved |

### List Active Guild Threads

  Responds with all active threads within the guild, covering public plus private threads. Threads are ordered by their `id`, in descending order.

###### Response Body

| Field | Type | Description |
| --- | --- | --- |
| threads | array of channel objects | the active threads |
| members | array of thread members objects | Thread member object record for each provided thread the current user has joined |

### Get Guild Member

  Responds with a [guild member](https://docs.discord.com/developers/resources/guild#guild-member-object) object record for the requested user.

### List Guild Members

  Responds with a list of [guild member](https://docs.discord.com/developers/resources/guild#guild-member-object) object records that are member records for the guild. This route needs the `GUILD_MEMBERS` [Privileged Intent](https://docs.discord.com/developers/events/gateway#privileged-intents). Every parameter on this route is optional.

###### Query String Params

| Field | Type | Description | Default |
| --- | --- | --- | --- |
| limit | integer | max count of member records to provide (1-1000) | 1 |
| after | snowflake | Highest user id within the previous page | 0 |

### Search Guild Members

  Responds with a list of [guild member](https://docs.discord.com/developers/resources/guild#guild-member-object) object records whose username or nickname starts with a provided string. Every parameter to this route except for `query` are optional

###### Query String Params

| Field | Type | Description | Default |
| --- | --- | --- | --- |
| query | string | Query string to match username(s) and nickname(s) against. |  |
| limit | integer | max count of member records to provide (1-1000) | 1 |

### Add Guild Member

  Adds one user to that guild, provided you have a valid oauth2 access token for the user account using the `guilds.join` scope. Responds with a 201 Created using the [guild member](https://docs.discord.com/developers/resources/guild#guild-member-object) as this body, or 204 No Content once the user account is already one member for the guild. Emits one [Guild Member Add](https://docs.discord.com/developers/events/gateway-events#guild-member-add) Gateway event. For guilds using [Membership Screening](https://docs.discord.com/developers/resources/guild#membership-screening-object) enabled, this route can default to adding new member records as `pending` within the [guild member object](https://docs.discord.com/developers/resources/guild#guild-member-object). Members that are `pending` can have to complete membership screening before they become full member records that can talk. Every parameter to this route except for `access_token` are optional. This Authorization header must be a Bot token (belonging to that same application used for authorization), and the bot must be one member for that server with `CREATE_INSTANT_INVITE` permission.

###### JSON Params

| Field | Type | Description | Permission |
| --- | --- | --- | --- |
| access_token | string | Oauth2 access token granted using the `guilds.join` to that bot’s application for the user account you want to add to that guild |  |
| nick | string | Value assigned to user’s nickname to | MANAGE_NICKNAMES |
| roles | array of snowflakes | List of role ids that member is assigned | MANAGE_ROLES |
| mute | boolean | Indicates whether the user account is muted in voice channels | MUTE_MEMBERS |
| deaf | boolean | Indicates whether the user account is deafened in voice channels | DEAFEN_MEMBERS |

### Modify Guild Member

  Updates attributes of a [guild member](https://docs.discord.com/developers/resources/guild#guild-member-object). Responds with a 200 OK using the [guild member](https://docs.discord.com/developers/resources/guild#guild-member-object) as the body. Emits a [Guild Member Update](https://docs.discord.com/developers/events/gateway-events#guild-member-update) Gateway event. Once the `channel_id` is set to null, this can force the target user to be disconnected from voice. Every parameter on this route is optional and nullable. Once moving member records to channels, the API user *must* have permissions to both connect to that channel and have the `MOVE_MEMBERS` permission. The `X-Audit-Log-Reason` header is accepted by this endpoint.

###### JSON Params

| Field | Type | Description | Permission |
| --- | --- | --- | --- |
| nick | string | Value assigned to user’s nickname to | MANAGE_NICKNAMES |
| roles | array of snowflakes | List of role ids that member is assigned | MANAGE_ROLES |
| mute | boolean | Indicates whether the user account is muted in voice channels. Will throw a 400 error once the user account is not in a voice channel | MUTE_MEMBERS |
| deaf | boolean | Indicates whether the user account is deafened in voice channels. Will throw a 400 error once the user account is not in a voice channel | DEAFEN_MEMBERS |
| channel_id | snowflake | ID for channel to move user to (if they are connected to voice) | MOVE_MEMBERS |
| communication_disabled_until | ISO8601 timestamp | timestamp for once the user’s timeout can expire and the user account can communicate within that server again (up to 28 days within this future), set to null to remove timeout. Will throw a 403 error once the user account has the ADMINISTRATOR permission or is the owner for the guild | MODERATE_MEMBERS |
| flags | integer | guild member flags | MANAGE_GUILD or MANAGE_ROLES or (MODERATE_MEMBERS and KICK_MEMBERS and BAN_MEMBERS) |

### Modify Current Member

  Updates the current member in a guild. Responds with a 200 using the updated member object record after a successful request. Emits a [Guild Member Update](https://docs.discord.com/developers/events/gateway-events#guild-member-update) Gateway event. The `X-Audit-Log-Reason` header is accepted by this endpoint.

###### JSON Params

| Field | Type | Description | Permission |
| --- | --- | --- | --- |
| nick? | ?string | Value assigned to user’s nickname to | CHANGE_NICKNAME |
| banner? | ?string | data URI base64 encoded banner image |  |
| avatar? | ?string | data URI base64 encoded avatar image |  |
| bio? | ?string | guild member bio |  |

### Modify Current User Nick

  Deprecated in favor of [Updates Current Member](https://docs.discord.com/developers/resources/guild#modify-current-member). Updates the nickname for the current user in a guild. Responds with a 200 using the nickname after a successful request. Emits a [Guild Member Update](https://docs.discord.com/developers/events/gateway-events#guild-member-update) Gateway event. The `X-Audit-Log-Reason` header is accepted by this endpoint.

###### JSON Params

| Field | Type | Description | Permission |
| --- | --- | --- | --- |
| nick? | ?string | Value assigned to user’s nickname to | CHANGE_NICKNAME |

### Add Guild Member Role

  Adds one role to a [guild member](https://docs.discord.com/developers/resources/guild#guild-member-object). The caller needs the `MANAGE_ROLES` permission. Responds with a 204 empty response after a successful request. Emits a [Guild Member Update](https://docs.discord.com/developers/events/gateway-events#guild-member-update) Gateway event. The `X-Audit-Log-Reason` header is accepted by this endpoint.

### Remove Guild Member Role

  Removes one role from a [guild member](https://docs.discord.com/developers/resources/guild#guild-member-object). The caller needs the `MANAGE_ROLES` permission. Responds with a 204 empty response after a successful request. Emits a [Guild Member Update](https://docs.discord.com/developers/events/gateway-events#guild-member-update) Gateway event. The `X-Audit-Log-Reason` header is accepted by this endpoint.

### Remove Guild Member

  Removes one member from a guild. The caller needs `KICK_MEMBERS` permission. Responds with a 204 empty response after a successful request. Emits a [Guild Member Remove](https://docs.discord.com/developers/events/gateway-events#guild-member-remove) Gateway event. The `X-Audit-Log-Reason` header is accepted by this endpoint.

### Get Guild Bans

  Responds with a list of [ban](https://docs.discord.com/developers/resources/guild#ban-object) object records for the user accounts banned from this guild. The caller needs the `BAN_MEMBERS` permission.

###### Query String Params

| Field | Type | Description | Default |
| --- | --- | --- | --- |
| limit? | number | Count of user accounts to provide (up to maximum 1000) | 1000 |
| before? * | snowflake | consider only user accounts before given user id | null |
| after? * | snowflake | consider only user accounts after given user id | null |

 * Provide one user id to `before` and `after` for pagination. Users can always be provided in ascending order by `user.id`. Once both `before` and `after` are provided, only `before` is respected.

### Get Guild Ban

  Responds with a [ban](https://docs.discord.com/developers/resources/guild#ban-object) object record for the supplied user or a 404 not found once the ban cannot be found. The caller needs the `BAN_MEMBERS` permission.

### Create Guild Ban

  Creates a server ban, and optionally delete previous messages emitted by the banned user. The caller needs the `BAN_MEMBERS` permission. Responds with a 204 empty response after a successful request. Emits a [Guild Ban Add](https://docs.discord.com/developers/events/gateway-events#guild-ban-add) Gateway event. The `X-Audit-Log-Reason` header is accepted by this endpoint.

###### JSON Params

| Field | Type | Description | Default |
| --- | --- | --- | --- |
| delete_message_days? | integer | Count of days to delete messages for (0-7) (deprecated) | 0 |
| delete_message_seconds? | integer | Count of seconds to delete messages for, between 0 and 604800 (7 days) | 0 |

### Remove Guild Ban

  Remove the ban for a user. The caller needs the `BAN_MEMBERS` permissions. Responds with a 204 empty response after a successful request. Emits a [Guild Ban Remove](https://docs.discord.com/developers/events/gateway-events#guild-ban-remove) Gateway event. The `X-Audit-Log-Reason` header is accepted by this endpoint.

### Bulk Guild Ban

  Ban up to 200 user accounts from a guild, and optionally delete previous messages emitted by the banned users. Needs both the `BAN_MEMBERS` and `MANAGE_GUILD` permissions. Responds with a 200 response after a successful request, covering the fields `banned_users` using the IDs for the banned user accounts plus `failed_users` with IDs that could not be banned or were already banned. The `X-Audit-Log-Reason` header is accepted by this endpoint.

###### JSON Params

| Field | Type | Description | Default |
| --- | --- | --- | --- |
| user_ids | array of snowflakes | List of user ids to ban (max 200) |  |
| delete_message_seconds? | integer | Count of seconds to delete messages for, between 0 and 604800 (7 days) | 0 |

###### Bulk Ban Response

 On success, this route returns a 200 success response using the following body.
| Field | Type | Description |
| --- | --- | --- |
| banned_users | array of snowflakes | List of user ids, that were successfully banned |
| failed_users | array of snowflakes | List of user ids, that were not banned |

 Once none for the user accounts could be banned, an error code response `500000: Failed to ban users` is provided instead.

### Get Guild Roles

  Responds with a list of [role](https://docs.discord.com/developers/topics/permissions#role-object) object records for the guild.

### Get Guild Role

  Responds with a [role](https://docs.discord.com/developers/topics/permissions#role-object) object record for the requested role.

### Get Guild Role Member Counts

  Responds with a map of role IDs to that count of member records using the role. Does not include the @everyone role.

###### Example Response

```
{
  "613425648685547541": 1337,
  "1409696176629878905": 2,
  "697138785317814292": 67
}

```

### Create Guild Role

  Creates a new [role](https://docs.discord.com/developers/topics/permissions#role-object) for the guild. The caller needs the `MANAGE_ROLES` permission. Responds using the new [role](https://docs.discord.com/developers/topics/permissions#role-object) object record after a successful request. Emits a [Guild Role Create](https://docs.discord.com/developers/events/gateway-events#guild-role-create) Gateway event. Every JSON parameter is optional. The `X-Audit-Log-Reason` header is accepted by this endpoint.

###### JSON Params
