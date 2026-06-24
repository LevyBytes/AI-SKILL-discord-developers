# User, Voice

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Topics: Usernames and Nicknames; Voice State Object.

## Usernames and Nicknames

A Discord User is the core account entity. Users can appear across the platform, join Guilds, take part in text and voice chat, and use many other features. Discord distinguishes "bot" users from "normal" users. Bot users behave similarly to normal users, but they are automated accounts "owned" by another user and are *not* capped on the number of Guilds they can join.

### Usernames and Nicknames

 Discord applies these username and nickname constraints:
1. Names may use most valid unicode characters, with limits on some zero-width and non-rendering characters.
2. Usernames must be 2 to 32 characters long.
3. Nicknames must be 1 to 32 characters long.
4. Names are sanitized, then leading, trailing, and excessive internal whitespace is trimmed.
 Usernames have these extra restrictions:
1. Usernames cannot include any of these substrings: `@`, `#`, `:`, `````, `discord`
2. Usernames cannot be: `everyone`, `here`
 Additional username rules are withheld to limit spam and abuse workarounds, and most users will not run into them. When editing or updating names, handle every error message returned by Discord.

### User Object

###### User Structure

| Field | Type | Description | Required OAuth2 Scope |
| --- | --- | --- | --- |
| id | snowflake | the user’s id | identify |
| username | string | the user's username; this value is not unique across the platform | identify |
| discriminator | string | the user’s Discord-tag | identify |
| global_name | ?string | the user's display name when one is configured | identify |
| avatar | ?string | the user’s avatar hash | identify |
| bot? | boolean | whether an OAuth2 application owns this user | identify |
| system? | boolean | whether the user is an Official Discord System user, used by the urgent message system | identify |
| mfa_enabled? | boolean | whether two factor authentication is enabled on the user's account | identify |
| banner? | ?string | the user’s banner hash | identify |
| accent_color? | ?integer | the user's banner color as an integer form of a hexadecimal color code | identify |
| locale? | string | the user's selected language option | identify |
| verified? | boolean | whether the account email has been verified | email |
| email? | ?string | the user’s email | email |
| flags? | integer | the flags attached to the user's account | identify |
| premium_type? | integer | the Nitro subscription type on the user's account | identify.premium |
| public_flags? | integer | the public flags exposed for the user's account | identify |
| avatar_decoration_data? | ?avatar decoration data object | avatar decoration data for the user | identify |
| collectibles? | ?collectibles object | collectibles data for the user | identify |
| primary_guild? | ?user primary guild object | the user’s primary guild | identify |

###### Example User

```
{
  "id": "80351110224678912",
  "username": "Nelly",
  "global_name": null,
  "discriminator": "1337",
  "avatar": "8342729096ea3675442027381ff50dfe",
  "verified": true,
  "email": "nelly@discord.com",
  "flags": 64,
  "banner": "06c16474723fe537c283b8efa61a30c8",
  "accent_color": 16711680,
  "premium_type": 0,
  "public_flags": 64,
  "avatar_decoration_data": {
    "sku_id": "1144058844004233369",
    "asset": "a_fed43ab12698df65902ba06727e20c0e"
  },
  "collectibles": {
    "nameplate": {
      "sku_id": "2247558840304243311",
      "asset": "nameplates/nameplates/twilight/",
      "label": "",
      "palette": "cobalt"
    }
  },
  "primary_guild": {
    "identity_guild_id": "1234647491267808778",
    "identity_enabled": true,
    "tag": "DISC",
    "badge": "7d1734ae5a615e82bc7a4033b98fade8"
  }
}

```

###### User Flags

| Value | Name | Description |
| --- | --- | --- |
| `1 << 0` | STAFF | Discord Employee |
| `1 << 1` | PARTNER | Partnered Server Owner |
| `1 << 2` | HYPESQUAD | HypeSquad Events Member |
| `1 << 3` | BUG_HUNTER_LEVEL_1 | Bug Hunter Level 1 |
| `1 << 6` | HYPESQUAD_ONLINE_HOUSE_1 | House Bravery Member |
| `1 << 7` | HYPESQUAD_ONLINE_HOUSE_2 | House Brilliance Member |
| `1 << 8` | HYPESQUAD_ONLINE_HOUSE_3 | House Balance Member |
| `1 << 9` | PREMIUM_EARLY_SUPPORTER | Early Nitro Supporter |
| `1 << 10` | TEAM_PSEUDO_USER | User is a team |
| `1 << 14` | BUG_HUNTER_LEVEL_2 | Bug Hunter Level 2 |
| `1 << 16` | VERIFIED_BOT | Verified Bot |
| `1 << 17` | VERIFIED_DEVELOPER | Early Verified Bot Developer |
| `1 << 18` | CERTIFIED_MODERATOR | Moderator Programs Alumni |
| `1 << 19` | BOT_HTTP_INTERACTIONS | Bot communicates only through HTTP interactions and appears in the online member list |

###### Premium Types

 Premium types identify a user's premium level. See the [Nitro](https://discord.com/nitro) page for the current premium plans.
| Value | Name |
| --- | --- |
| 0 | None |
| 1 | Nitro Classic |
| 2 | Nitro |
| 3 | Nitro Basic |

###### User Primary Guild

| Field | Type | Description |
| --- | --- | --- |
| identity_guild_id | ?snowflake | the id of the user's primary guild |
| identity_enabled | ?boolean | whether the user is showing the primary guild's server tag. The value can be `null` when the system clears the identity, for example if the server stops supporting tags. It is `false` when the user manually removes the tag. |
| tag | ?string | the user's server tag text, limited to 4 characters |
| badge | ?string | the server tag badge hash |

### Avatar Decoration Data Object

 Information for the user's [avatar decoration](https://support.discord.com/hc/en-us/articles/13410113109911-Avatar-Decorations).

###### Avatar Decoration Data Structure

| Field | Type | Description |
| --- | --- | --- |
| asset | string | the avatar decoration hash |
| sku_id | snowflake | id of the SKU for the avatar decoration |

### Collectibles

 Collectibles held by the user, excluding Avatar Decorations and Profile Effects.

###### Collectible Structure

| Field | Type | Description |
| --- | --- | --- |
| nameplate? | object | object mapping of nameplate data |

### Nameplate

 The nameplate the user has.

###### Nameplate Structure

| Field | Type | Description |
| --- | --- | --- |
| sku_id | snowflake | id of the nameplate SKU |
| asset | string | path to the nameplate asset |
| label | string | the nameplate label. Currently unused |
| palette | string | nameplate background color; one of: `crimson`, `berry`, `sky`, `teal`, `forest`, `bubble_gum`, `violet`, `cobalt`, `clover`, `lemon`, `white` |

### Connection Object

 The connection object attached to the user.

###### Connection Structure

| Field | Type | Description |
| --- | --- | --- |
| id | string | id of the connection account |
| name | string | the username on the connected account |
| type | string | the service of this connection |
| revoked? | boolean | whether the connection is revoked |
| integrations? | array | an array containing partial server integrations |
| verified | boolean | whether the connection is verified |
| friend_sync | boolean | whether friend sync is enabled on this connection |
| show_activity | boolean | whether presence updates display activities tied to this connection |
| two_way_link | boolean | whether a matching third party OAuth2 token exists for this connection |
| visibility | integer | visibility of this connection |

###### Services

| Value | Name |
| --- | --- |
| amazon-music | Amazon Music |
| battlenet | Battle.net |
| bungie | Bungie.net |
| bluesky | Bluesky |
| crunchyroll | Crunchyroll |
| domain | Domain |
| ebay | eBay |
| epicgames | Epic Games |
| facebook | Facebook |
| github | GitHub |
| instagram * | Instagram |
| leagueoflegends | League of Legends |
| mastodon | Mastodon |
| paypal | PayPal |
| playstation | PlayStation Network |
| reddit | Reddit |
| riotgames | Riot Games |
| roblox | Roblox |
| spotify | Spotify |
| skype * | Skype |
| steam | Steam |
| tiktok | TikTok |
| twitch | Twitch |
| twitter | X (Twitter) |
| xbox | Xbox |
| youtube | YouTube |

 * Users can no longer add this service

###### Visibility Types

| Value | Name | Description |
| --- | --- | --- |
| 0 | None | hidden from everyone except the user |
| 1 | Everyone | visible to everyone |

### Application Role Connection Object

 The role connection object attached to a user by an application.

###### Application Role Connection Structure

| Field | Type | Description |
| --- | --- | --- |
| platform_name | ?string | the vanity name for the platform a bot connected (max 50 characters) |
| metadata | object | object that maps application role connection metadata keys to the user's `string`-ified value (max 100 characters) on the platform a bot connected |

### Get Current User

  Returns the requester account's [user](https://docs.discord.com/developers/resources/user#user-object) object. With OAuth2, the `identify` scope is required and returns the object *without* an email; the optional `email` scope returns the object *with* an email when the user has one.

### Get User

  Returns a [user](https://docs.discord.com/developers/resources/user#user-object) object for the supplied user ID.

### Modify Current User

  Updates the requester's user account settings and returns a [user](https://docs.discord.com/developers/resources/user#user-object) object on success. A successful update fires a [User Update](https://docs.discord.com/developers/events/gateway-events#user-update) Gateway event. Every parameter for this endpoint is optional.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| username | string | user's username; changing it may randomize the user's discriminator. |
| avatar | ?image data | when provided, changes the user's avatar |
| banner | ?image data | when provided, changes the user's banner |

### Get Current User Guilds

  Returns partial [guild](https://docs.discord.com/developers/resources/guild#guild-object) objects for guilds that include the current user. OAuth2 calls require the `guilds` scope.

###### Example Partial Guild

```
{
  "id": "80351110224678912",
  "name": "1337 Krew",
  "icon": "8342729096ea3675442027381ff50dfe",
  "banner": "bb42bdc37653b7cf58c4c8cc622e76cb",
  "owner": true,
  "permissions": "36953089",
  "features": ["COMMUNITY", "NEWS", "ANIMATED_ICON", "INVITE_SPLASH", "BANNER", "ROLE_ICONS"],
  "approximate_member_count": 3268,
  "approximate_presence_count": 784
}

```

 By default this endpoint returns 200 guilds, matching the maximum number of guilds a non-bot user can join. Integrations that only need the user's guild list do **not need** pagination.

###### Query String Params

| Field | Type | Description | Required | Default |
| --- | --- | --- | --- | --- |
| before | snowflake | return guilds before this guild ID | false | absent |
| after | snowflake | return guilds after this guild ID | false | absent |
| limit | integer | maximum number of guilds to return (1-200) | false | 200 |
| with_counts | boolean | include approximate member and presence counts in the response | false | false |

### Get Current User Guild Member

  Returns the current user's [guild member](https://docs.discord.com/developers/resources/guild#guild-member-object) object. Requires the `guilds.members.read` OAuth2 scope.

### Leave Guild

  Makes the current user leave a guild. On success, returns an empty 204 response and fires both a [Guild Delete](https://docs.discord.com/developers/events/gateway-events#guild-delete) Gateway event and a [Guild Member Remove](https://docs.discord.com/developers/events/gateway-events#guild-member-remove) Gateway event.

### Create DM

  Creates a DM channel with a user and returns the [DM channel](https://docs.discord.com/developers/resources/channel#channel-object) object; if the channel already exists, the existing one is returned. Do not use this endpoint to message every server member. A DM should usually follow a user action, and opening many DMs quickly can cause the bot to be rate limited or blocked from opening more.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| recipient_id | snowflake | the recipient user for the DM channel |

### Create Group DM

  Creates a group DM channel with multiple users and returns a [DM channel](https://docs.discord.com/developers/resources/channel#channel-object) object. The endpoint was designed for the now-deprecated GameBridge SDK. It fires a [Channel Create](https://docs.discord.com/developers/events/gateway-events#channel-create) Gateway event and is capped at 10 active group DMs.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| access_tokens | array of strings | access tokens for users who granted your app the `gdm.join` scope |
| nicks | dict | a dictionary mapping user ids to their nicknames |

### Get Current User Connections

  Returns a list of [connection](https://docs.discord.com/developers/resources/user#connection-object) objects. Requires the `connections` OAuth2 scope.

### Get Current User Application Role Connection

  Returns the user's [application role connection](https://docs.discord.com/developers/resources/user#application-role-connection-object) for the application named in the path. Requires an OAuth2 access token with the `role_connections.write` scope.

### Update Current User Application Role Connection

  Updates the user's [application role connection](https://docs.discord.com/developers/resources/user#application-role-connection-object) for the application named in the path, then returns it. Requires an OAuth2 access token with the `role_connections.write` scope.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| platform_name? | string | the vanity name for the platform a bot connected (max 50 characters) |
| platform_username? | string | the username on the platform a bot connected (max 100 characters) |
| metadata? | object | object that maps application role connection metadata keys to the user's `string`-ified value (max 100 characters) on the platform a bot connected |

### Delete Current User Application Role Connection

  Deletes the user's [application role connection](https://docs.discord.com/developers/resources/user#application-role-connection-object) for the application named in the path. Requires an OAuth2 access token with the `role_connections.write` scope.

## Voice State Object

### Voice State Object

 Represents a user's voice connection status.

###### Voice State Structure

| Field | Type | Description |
| --- | --- | --- |
| guild_id? | snowflake | the guild id for this voice state |
| channel_id | ?snowflake | the channel id the user is connected to |
| user_id | snowflake | the user id for this voice state |
| member? | guild member object | the guild member tied to this voice state |
| session_id | string | the session id associated with this voice state |
| deaf | boolean | whether the server deafened this user |
| mute | boolean | whether the server muted this user |
| self_deaf | boolean | whether this user deafened locally |
| self_mute | boolean | whether this user muted locally |
| self_stream? | boolean | whether this user is streaming with "Go Live" |
| self_video | boolean | whether this user's camera is enabled |
| suppress | boolean | whether this user's permission to speak is denied |
| request_to_speak_timestamp | ?ISO8601 timestamp | when the user requested to speak |

###### Example Voice State

```
{
  "channel_id": "157733188964188161",
  "user_id": "80351110224678912",
  "session_id": "90326bd25d71d39b9ef95b299e3872ff",
  "deaf": false,
  "mute": false,
  "self_deaf": false,
  "self_mute": true,
  "suppress": false,
  "request_to_speak_timestamp": "2021-03-31T18:45:31.297561+00:00"
}

```

### Voice Region Object

###### Voice Region Structure

| Field | Type | Description |
| --- | --- | --- |
| id | string | unique ID for the region |
| name | string | name of the region |
| optimal | boolean | true for the single server nearest to the current user's client |
| deprecated | boolean | whether this voice region is deprecated; avoid switching to deprecated regions |
| custom | boolean | whether this is a custom voice region, such as one used for events |

### List Voice Regions

  Returns an array of [voice region](https://docs.discord.com/developers/resources/voice#voice-region-object) objects usable when setting a voice or stage channel's [`rtc_region`](https://docs.discord.com/developers/resources/channel#channel-object-channel-structure).

### Get Current User Voice State

  Returns the current user's [voice state](https://docs.discord.com/developers/resources/voice#voice-state-object) in the guild.

### Get User Voice State

  Returns the specified user's [voice state](https://docs.discord.com/developers/resources/voice#voice-state-object) in the guild. If that user is connected to a voice channel, the current user needs permission to connect to that channel.

### Modify Current User Voice State

  Updates the current user's voice state. On success, returns `204 No Content` and fires a [Voice State Update](https://docs.discord.com/developers/events/gateway-events#voice-state-update) Gateway event.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| channel_id? | snowflake | the id of the channel the user is currently in |
| suppress? | boolean | toggles the user's suppress state |
| request_to_speak_timestamp? | ?ISO8601 timestamp | sets the user's request to speak |

###### Caveats

 Current caveats for this endpoint:
- `channel_id` must point to a stage channel.
- The current user must already be joined to `channel_id`.
- You need the `MUTE_MEMBERS` permission to unsuppress yourself. You can always suppress yourself.
- You need the `REQUEST_TO_SPEAK` permission to request to speak. You can always clear your own request to speak.
- `request_to_speak_timestamp` may be set to any present or future time.

### Modify User Voice State

  Updates another user's voice state. On success, returns `204 No Content` and fires a [Voice State Update](https://docs.discord.com/developers/events/gateway-events#voice-state-update) Gateway event.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| channel_id? | snowflake | the id of the channel the user is currently in |
| suppress? | boolean | toggles the user's suppress state |

###### Caveats

 Current caveats for this endpoint:
- `channel_id` must point to a stage channel.
- The user must already be joined to `channel_id`.
- You need the `MUTE_MEMBERS` permission, because suppression is currently the only available operation.
- When unsuppressed, non-bot users get `request_to_speak_timestamp` set to the current time. Bot users do not.
- When suppressed, the user's `request_to_speak_timestamp` is removed.
