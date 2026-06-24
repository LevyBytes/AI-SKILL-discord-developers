# Permissions

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Discord permissions define which actions a user can perform. Role permissions start at the guild level: roles are configured with base capabilities, then assigned to users to grant or withhold privileges. Discord also supports per-channel permission overwrites for roles and individual members. [Application command permissions](https://docs.discord.com/developers/interactions/application-commands#permissions) can enable or disable commands for channels, roles, or users. The permission set is a variable-length integer, serialized as a string and evaluated with bitwise math. A permission value such as `123` is serialized as `"123"`. For durable implementations, deserialize permission strings with the Big Integer support in your language. Combine individual flags with OR (`|`), and test individual flags with AND (`&`). Starting with API v8, every permission value is serialized as a string, including overwrite fields such as `allow` and `deny`; new permission bits are folded into the base field. In [API v6 (now deprecated)](https://docs.discord.com/developers/reference), role and overwrite fields named `permissions`, `allow`, and `deny` are still numbers, but they will not exceed 31 bits. During the rest of API v6, new bits appear only in `permissions_new`, `allow_new`, and `deny_new`. Those `_new` fields exist for response serialization only; requests still use `permissions`, `allow`, and `deny`, accepting either string or number values.

```
# Permissions value that can Send Messages (0x800) and Add Reactions (0x40):
permissions = 0x40 | 0x800 # 2112

# Checking for flags that are set:
(permissions & 0x40) == 0x40   # True
(permissions & 0x800) == 0x800 # True

# Kick Members (0x2) was not set:
(permissions & 0x2) == 0x2 # False

```

Permission overwrites add more rules, covered later in this reference. For background on bit fields and bitwise flag handling, see [this page](https://en.wikipedia.org/wiki/Bit_field). The table below lists current permission flags, their hexadecimal values, their granted privilege, and the channel categories where they apply.

###### Bitwise Permission Flags

| Permission | Value | Description | Channel Type (Abbreviated) |
| --- | --- | --- | --- |
| CREATE_INSTANT_INVITE | `0x0000000000000001` `(1 << 0)` | Permits instant invite creation | T, V, S |
| KICK_MEMBERS * | `0x0000000000000002` `(1 << 1)` | Permits member kicks |  |
| BAN_MEMBERS * | `0x0000000000000004` `(1 << 2)` | Permits member bans |  |
| ADMINISTRATOR * | `0x0000000000000008` `(1 << 3)` | Grants every permission and ignores channel overwrites |  |
| MANAGE_CHANNELS * | `0x0000000000000010` `(1 << 4)` | Permits channel management and editing | T, V, S |
| MANAGE_GUILD * | `0x0000000000000020` `(1 << 5)` | Permits guild management and editing |  |
| ADD_REACTIONS | `0x0000000000000040` `(1 << 6)` | Permits adding new reactions to messages; existing reactions can still be used without this flag | T, V, S |
| VIEW_AUDIT_LOG | `0x0000000000000080` `(1 << 7)` | Permits audit log viewing |  |
| PRIORITY_SPEAKER | `0x0000000000000100` `(1 << 8)` | Permits priority speaker use in voice channels | V |
| STREAM | `0x0000000000000200` `(1 << 9)` | Permits going live | V, S |
| VIEW_CHANNEL | `0x0000000000000400` `(1 << 10)` | Permits seeing a channel, including reading text channels and joining voice channels | T, V, S |
| SEND_MESSAGES | `0x0000000000000800` `(1 << 11)` | Permits channel messages and forum thread creation, but not messages inside threads | T, V, S |
| SEND_TTS_MESSAGES | `0x0000000000001000` `(1 << 12)` | Permits `/tts` messages | T, V, S |
| MANAGE_MESSAGES * | `0x0000000000002000` `(1 << 13)` | Permits deleting messages sent by other users | T, V, S |
| EMBED_LINKS | `0x0000000000004000` `(1 << 14)` | Causes links from permitted users to auto-embed | T, V, S |
| ATTACH_FILES | `0x0000000000008000` `(1 << 15)` | Permits image and file uploads | T, V, S |
| READ_MESSAGE_HISTORY | `0x0000000000010000` `(1 << 16)` | Permits access to prior messages | T, V, S |
| MENTION_EVERYONE | `0x0000000000020000` `(1 << 17)` | Permits the `@everyone` and `@here` mentions for notifying all users or online users | T, V, S |
| USE_EXTERNAL_EMOJIS | `0x0000000000040000` `(1 << 18)` | Permits custom emoji use from other servers | T, V, S |
| VIEW_GUILD_INSIGHTS | `0x0000000000080000` `(1 << 19)` | Permits viewing guild insights |  |
| CONNECT | `0x0000000000100000` `(1 << 20)` | Permits joining voice channels | V, S |
| SPEAK | `0x0000000000200000` `(1 << 21)` | Permits speaking in voice channels | V |
| MUTE_MEMBERS | `0x0000000000400000` `(1 << 22)` | Permits muting members in voice channels | V, S |
| DEAFEN_MEMBERS | `0x0000000000800000` `(1 << 23)` | Permits deafening members in voice channels | V |
| MOVE_MEMBERS | `0x0000000001000000` `(1 << 24)` | Permits moving members between voice channels | V, S |
| USE_VAD | `0x0000000002000000` `(1 << 25)` | Permits voice-activity-detection in voice channels | V |
| CHANGE_NICKNAME | `0x0000000004000000` `(1 << 26)` | Permits changing one's own nickname |  |
| MANAGE_NICKNAMES | `0x0000000008000000` `(1 << 27)` | Permits changing nicknames for other users |  |
| MANAGE_ROLES * | `0x0000000010000000` `(1 << 28)` | Permits role management and editing | T, V, S |
| MANAGE_WEBHOOKS * | `0x0000000020000000` `(1 << 29)` | Permits webhook management and editing | T, V, S |
| MANAGE_GUILD_EXPRESSIONS * | `0x0000000040000000` `(1 << 30)` | Permits editing and deleting any user's emojis, stickers, and soundboard sounds |  |
| USE_APPLICATION_COMMANDS | `0x0000000080000000` `(1 << 31)` | Permits slash commands and context menu commands | T, V, S |
| REQUEST_TO_SPEAK | `0x0000000100000000` `(1 << 32)` | Permits requests to speak in stage channels | S |
| MANAGE_EVENTS | `0x0000000200000000` `(1 << 33)` | Permits editing and deleting scheduled events created by any user | V, S |
| MANAGE_THREADS * | `0x0000000400000000` `(1 << 34)` | Permits thread deletion, archiving, and access to all private threads | T |
| CREATE_PUBLIC_THREADS | `0x0000000800000000` `(1 << 35)` | Permits public and announcement thread creation | T |
| CREATE_PRIVATE_THREADS | `0x0000001000000000` `(1 << 36)` | Permits private thread creation | T |
| USE_EXTERNAL_STICKERS | `0x0000002000000000` `(1 << 37)` | Permits custom sticker use from other servers | T, V, S |
| SEND_MESSAGES_IN_THREADS | `0x0000004000000000` `(1 << 38)` | Permits messages inside threads | T |
| USE_EMBEDDED_ACTIVITIES | `0x0000008000000000` `(1 << 39)` | Permits Activities, meaning applications with the `EMBEDDED` flag | T, V |
| MODERATE_MEMBERS ** | `0x0000010000000000` `(1 << 40)` | Permits timing out users, blocking their chat, thread reaction, and voice or stage participation temporarily |  |
| VIEW_CREATOR_MONETIZATION_ANALYTICS * | `0x0000020000000000` `(1 << 41)` | Permits viewing role subscription insights |  |
| USE_SOUNDBOARD | `0x0000040000000000` `(1 << 42)` | Permits soundboard use in voice channels | V |
| CREATE_GUILD_EXPRESSIONS | `0x0000080000000000` `(1 << 43)` | Permits creating emojis, stickers, and soundboard sounds, plus editing and deleting items created by the current user |  |
| CREATE_EVENTS | `0x0000100000000000` `(1 << 44)` | Permits creating scheduled events, plus editing and deleting events created by the current user | V, S |
| USE_EXTERNAL_SOUNDS | `0x0000200000000000` `(1 << 45)` | Permits custom soundboard sound use from other servers | V |
| SEND_VOICE_MESSAGES | `0x0000400000000000` `(1 << 46)` | Permits voice message sending | T, V, S |
| SET_VOICE_CHANNEL_STATUS | `0x0001000000000000` `(1 << 48)` | Permits setting a voice channel status | V |
| SEND_POLLS | `0x0002000000000000` `(1 << 49)` | Permits poll sending | T, V, S |
| USE_EXTERNAL_APPS | `0x0004000000000000` `(1 << 50)` | Permits user-installed apps to send public responses; without it, users may still use those apps, but responses are ephemeral. This applies only to apps not installed on the server. | T, V, S |
| PIN_MESSAGES | `0x0008000000000000` `(1 << 51)` | Permits pinning and unpinning messages | T |
| BYPASS_SLOWMODE | `0x0010000000000000` `(1 << 52)` | Permits bypassing slowmode restrictions | T, V, S |

| Channel Type (Abbreviated) | Description | Channel Types |
| --- | --- | --- |
| T | Text | GUILD_TEXT, GUILD_ANNOUNCEMENT, GUILD_FORUM, GUILD_MEDIA |
| V | Voice | GUILD_VOICE |
| S | Stage | GUILD_STAGE_VOICE |

* These permissions require the owner account to use [two-factor authentication](https://docs.discord.com/developers/topics/oauth2#twofactor-authentication-requirement) when the guild has server-wide 2FA enabled. ** See [Permissions for Timed Out Members](https://docs.discord.com/developers/topics/permissions#permissions-for-timed-out-members) for temporary permission changes applied to timed out users. Client labels may differ from API permission names: "Manage Permissions" maps to MANAGE_ROLES, "Use Voice Activity" maps to USE_VAD, and "Timeout Members" maps to MODERATE_MEMBERS.

### Permission Hierarchy

Permission results can look simple until role position restrictions matter. Bots cannot perform some actions against targets whose highest role is too high relative to the bot's highest role. A user's or bot's highest role is the assigned role with the largest position value in the guild; the default @everyone role has position 0. These hierarchy rules apply:
- A bot can grant roles only when those roles sit below its own highest role.
- A bot can edit lower roles, and can grant only permissions the bot already has.
- A bot can reorder only roles below its highest role.
- A bot can kick, ban, or edit nicknames only for users whose highest role is below the bot's highest role.
Outside those cases, role order does not resolve permission conflicts. For example, a user may have role A denying `VIEW_CHANNEL` on #coolstuff and role B allowing `VIEW_CHANNEL` on #coolstuff. The user can view #coolstuff regardless of whether role A or role B has the higher position.

### Permission Overwrites

Overwrites apply selected permissions to roles or members at the channel level. The flag table marks channel support with **T** for text, **V** for voice, and **S** for stage. A user can receive conflicting instructions from guild roles, @everyone channel overwrites, role overwrites, and member overwrites. Discord resolves those inputs in this order:
1. Guild-level base permissions from @everyone are applied
2. Guild-level permissions from the member's roles are added
3. Channel-level @everyone denies are applied
4. Channel-level @everyone allows are applied
5. Channel-level denies for specific roles are applied
6. Channel-level allows for specific roles are applied
7. Channel-level member-specific denies are applied
8. Channel-level member-specific allows are applied
The following pseudocode shows the same order:

```
def compute_base_permissions(member, guild):
    if guild.is_owner(member):
        return ALL

    role_everyone = guild.get_role(guild.id)  # get @everyone role
    permissions = role_everyone.permissions

    for role in member.roles:
        permissions |= role.permissions

    if permissions & ADMINISTRATOR == ADMINISTRATOR:
        return ALL

    return permissions

def compute_overwrites(base_permissions, member, channel):
    # ADMINISTRATOR overrides any potential permission overwrites, so there is nothing to do here.
    if base_permissions & ADMINISTRATOR == ADMINISTRATOR:
        return ALL

    permissions = base_permissions
    overwrite_everyone = overwrites.get(channel.guild_id)  # Find (@everyone) role overwrite and apply it.
    if overwrite_everyone:
        permissions &= ~overwrite_everyone.deny
        permissions |= overwrite_everyone.allow

    # Apply role specific overwrites.
    overwrites = channel.permission_overwrites
    allow = NONE
    deny = NONE
    for role_id in member.roles:
        overwrite_role = overwrites.get(role_id)
        if overwrite_role:
            allow |= overwrite_role.allow
            deny |= overwrite_role.deny

    permissions &= ~deny
    permissions |= allow

    # Apply member specific overwrite if it exist.
    overwrite_member = overwrites.get(member.user_id)
    if overwrite_member:
        permissions &= ~overwrite_member.deny
        permissions |= overwrite_member.allow

    return permissions

def compute_permissions(member, channel):
    base_permissions = compute_base_permissions(member, channel.guild)
    return compute_overwrites(base_permissions, member, channel)

```

### Implicit Permissions

Some Discord permissions become ineffective because another required capability is missing. In text channels, the main examples are `VIEW_CHANNEL` and `SEND_MESSAGES`. Denying `VIEW_CHANNEL` to a user or role prevents meaningful use of other channel permissions, because the user cannot see the channel. A permission such as `SEND_MESSAGES` might not be explicitly denied in the calculation, but it has no effect without channel visibility. Denying `SEND_MESSAGES` similarly makes `MENTION_EVERYONE`, `SEND_TTS_MESSAGES`, `ATTACH_FILES`, and `EMBED_LINKS` unusable because the user cannot send messages. For voice and stage channels, denying `CONNECT` can also make other permissions, such as `MANAGE_CHANNEL`, ineffective. Discord may apply the same reasoning to other permission combinations when one action logically depends on another.

### Inherited Permissions (Threads)

Threads use permissions from their parent channel, except for message sending. `SEND_MESSAGES` does not carry into threads; a user needs `SEND_MESSAGES_IN_THREADS` to post there. That lets users participate in threads created under areas such as announcement channels. A user still needs `VIEW_CHANNEL` to see *any* thread in the parent channel, even after being mentioned in or added to that thread.

### Permission Syncing

Categories do not pass permissions to child channels through simple inheritance. Discord uses Permission Syncing instead. A child channel is "synced" when its permissions and overwrites match its parent category, including the absence of overwrites. Later changes to the **parent category** are reflected in synced child channels. Later changes made directly to a **child channel** desync it, so future category permission changes no longer update that child channel.

### Role Object

Roles are named permission bundles assigned to groups of users. They can carry colors, appear separately in the sidebar when pinned, and hold different permission settings for the guild context and channel context. The `@everyone` role uses the same ID as its guild.

###### Role Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | role id |
| name | string | role name |
| color* | integer | **Deprecated** integer form of the hexadecimal color code |
| colors | role colors object | colors assigned to the role |
| hoist | boolean | whether this role is pinned in the user list |
| icon? | ?string | role icon hash |
| unicode_emoji? | ?string | role unicode emoji |
| position | integer | role position; roles sharing a position are ordered by id |
| permissions | string | permission bit set |
| managed | boolean | whether an integration manages this role |
| mentionable | boolean | whether users can mention this role |
| tags? | role tags object | tags attached to this role |
| flags | integer | role flags represented as a bitfield |

Roles with no colors (`colors.primary_color == 0`) are ignored when Discord computes the final user-list color. * `color` is still returned by the API, but requests should use the `colors` field.

###### Role Tags Structure

Tags whose type is `null` behave as booleans. They are present with value `null` when true, and omitted when false.
| Field | Type | Description |
| --- | --- | --- |
| bot_id? | snowflake | id of the bot this role belongs to |
| integration_id? | snowflake | id of the integration this role belongs to |
| premium_subscriber? | null | whether this is the guild's Booster role |
| subscription_listing_id? | snowflake | id of this role's subscription sku and listing |
| available_for_purchase? | null | whether this role is available for purchase |
| guild_connections? | null | whether this is the guild's linked role |

###### Role Colors Object

The object always includes `primary_color`, matching the role's `color`. The other fields may be non-null only when the guild has the `ENHANCED_ROLE_COLORS` [guild feature](https://docs.discord.com/developers/resources/guild#guild-object-guild-features).
| Field | Type | Description |
| --- | --- | --- |
| primary_color | integer | primary role color |
| secondary_color | ?integer | secondary role color used with other supplied colors to make a gradient |
| tertiary_color | ?integer | tertiary role color that changes the gradient to a holographic style |

When sending `tertiary_color`, the API forces the role color into a holographic style with these values: `primary_color = 11127295`, `secondary_color = 16759788`, and `tertiary_color = 16761760`.

###### Default Role Colors Object

```
"colors": {
  "primary_color": 0,
  "secondary_color": null,
  "tertiary_color": null
}

```

###### Example Role

```
{
  "id": "41771983423143936",
  "name": "WE DEM BOYZZ!!!!!!",
  "color": 3447003,
  "colors": {
    "primary_color": 3447003,
    "secondary_color": null,
    "tertiary_color": null
  },
  "hoist": true,
  "icon": "cf3ced8600b777c9486c6d8d84fb4327",
  "unicode_emoji": null,
  "position": 1,
  "permissions": "66321471",
  "managed": false,
  "mentionable": false,
  "flags": 0
}

```

###### Role Flags

| Flag | Value | Description |
| --- | --- | --- |
| IN_PROMPT | `1 << 0` | role may be chosen by members in an onboarding prompt |

### Permissions For Timed Out Members

Timed out members temporarily keep only `VIEW_CHANNEL` and `READ_MESSAGE_HISTORY`. Owners and users with `ADMINISTRATOR` are not affected.
