# Invite, Lobby

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: Invite Object; Lobby Object.

## Invite Object

### Invite Object

 Represents an invite code; using it makes someone join either a guild or a group DM channel.

###### Invite Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | the type of invite |
| code | string | the invite code (unique ID) |
| guild? | partial guild object | guild targeted by this invite |
| channel | ?partial channel object | channel targeted by this invite |
| inviter? | user object | user that created the invite |
| target_type? | integer | target type for this voice channel invite |
| target_user? | user object | user stream shown for this voice channel stream invite |
| target_application? | partial application object | embedded application opened by this voice channel embedded application invite |
| approximate_presence_count? | integer | approximate number of online members; returned by `GET /invites/<code>` when `with_counts` is `true` |
| approximate_member_count? | integer | approximate total member count; returned by `GET /invites/<code>` when `with_counts` is `true` |
| expires_at | ?ISO8601 timestamp | expiration timestamp for this invite |
| guild_scheduled_event? | guild scheduled event object | guild scheduled event data; included only when `guild_scheduled_event_id` is a valid guild scheduled event id |
| flags? | integer | guild invite flags for guild invites |
| roles?* | array of partial role objects | roles assigned after the user accepts the invite |

 * This partial role object includes only `id`, `name`, `position`, `color`, `colors`, `icon`, and `unicode_emoji`.

###### Invite Types

| Type | Value |
| --- | --- |
| GUILD | 0 |
| GROUP_DM | 1 |
| FRIEND | 2 |

###### Invite Target Types

| Type | Value |
| --- | --- |
| STREAM | 1 |
| EMBEDDED_APPLICATION | 2 |

###### Guild Invite Flags

| Flag | Value | Description |
| --- | --- | --- |
| IS_GUEST_INVITE | `1 << 0` | guest invite for a voice channel |

###### Example Invite Object

```
{
  "type": 0,
  "code": "0vCdhLbwjZZTWZLD",
  "guild": {
    "id": "165176875973476352",
    "name": "CS:GO Fraggers Only",
    "splash": null,
    "banner": null,
    "description": "Very good description",
    "icon": null,
    "features": ["NEWS", "DISCOVERABLE"],
    "verification_level": 2,
    "vanity_url_code": null,
    "nsfw_level": 0,
    "premium_subscription_count": 5
  },
  "channel": {
    "id": "165176875973476352",
    "name": "illuminati",
    "type": 0
  },
  "inviter": {
    "id": "115590097100865541",
    "username": "speed",
    "avatar": "deadbeef",
    "discriminator": "7653",
    "public_flags": 131328
  },
  "target_type": 1,
  "target_user": {
    "id": "165176875973476352",
    "username": "bob",
    "avatar": "deadbeef",
    "discriminator": "1234",
    "public_flags": 64
  }
}

```

### Invite Metadata Object

 Additional invite information that extends the [invite](https://docs.discord.com/developers/resources/invite#invite-object) object.

###### Invite Metadata Structure

| Field | Type | Description |
| --- | --- | --- |
| uses | integer | how many times this invite has been used |
| max_uses | integer | maximum number of allowed uses for this invite |
| max_age | integer | duration in seconds before the invite expires |
| temporary | boolean | whether the invite grants only temporary membership |
| created_at | ISO8601 timestamp | when this invite was created |

###### Example Invite Metadata

```
{
  "uses": 0,
  "max_uses": 0,
  "max_age": 0,
  "temporary": false,
  "created_at": "2016-03-31T19:15:39.954000+00:00"
}

```

### Invite Stage Instance Object

 This is deprecated.

###### Invite Stage Instance Structure

| Field | Type | Description |
| --- | --- | --- |
| members | array of partial guild member objects | members currently speaking in the Stage |
| participant_count | integer | user count in the Stage |
| speaker_count | integer | count of users currently speaking in the Stage |
| topic | string | Stage instance topic (1-120 characters) |

###### Example Invite Stage Instance

```
{
  "topic": "The debate is over: diet is better than regular",
  "participant_count": 200,
  "speaker_count": 5 ,
  "members": [
    {
      "roles": [],
      "nick": "NOT API SUPPORT",
      "avatar": null,
      "premium_since": null,
      "joined_at": "2015-04-26T06:26:56.936000+00:00",
      "pending": false,
      "user": {}
    }
  ]
}

```

### Get Invite

 GET/invites/[{invite.code}](https://docs.discord.com/developers/resources/invite#invite-object) Returns the [invite](https://docs.discord.com/developers/resources/invite#invite-object) object for that code.

###### Query String Params

| Field | Type | Description |
| --- | --- | --- |
| with_counts? | boolean | whether approximate member counts should be included on the invite |
| guild_scheduled_event_id? | snowflake | guild scheduled event to include on the invite |

### Delete Invite

 DELETE/invites/[{invite.code}](https://docs.discord.com/developers/resources/invite#invite-object) Deletes an invite. Requires `MANAGE_CHANNELS` on the invite's channel, or `MANAGE_GUILD` to delete any invite in the guild. On success, returns an [invite](https://docs.discord.com/developers/resources/invite#invite-object) object. Discord emits the [Invite Delete](https://docs.discord.com/developers/events/gateway-events#invite-delete) Gateway event. The endpoint accepts `X-Audit-Log-Reason`.

### Get Target Users

 GET/invites/[{invite.code}](https://docs.discord.com/developers/resources/invite#invite-object)/target-users Returns users allowed to view and accept the invite. The response is a CSV with header `user_id`; each subsequent line contains a user ID from the file submitted to [invite create](https://docs.discord.com/developers/resources/channel#create-channel-invite). Caller must be the inviter, have `MANAGE_GUILD`, or have `VIEW_AUDIT_LOG`.

### Update Target Users

 PUT/invites/[{invite.code}](https://docs.discord.com/developers/resources/invite#invite-object)/target-users Replaces the users allowed to view and accept the invite. If the uploaded file has invalid user IDs, the response is a 400 that lists those invalid IDs. The caller must be the inviter or have `MANAGE_GUILD`.

###### Form Params

| Field | Type | Description |
| --- | --- | --- |
| target_users_file | file | CSV file with one user ID column listing every user allowed to accept this invite |

###### Example Error Response

```
{
  "target_users_file": [
    "Line 5: invalid user ID - must be a valid Discord snowflake",
    "Line 6: invalid user ID format"
  ]
}

```

### Get Target Users Job Status

 GET/invites/[{invite.code}](https://docs.discord.com/developers/resources/invite#invite-object)/target-users/job-status CSV processing for target users during invite creation or update is asynchronous. This endpoint reports that job's status. The caller must be the inviter, have `MANAGE_GUILD`, or have `VIEW_AUDIT_LOG`.

###### Example Response

```
{
  "status": 3,
  "total_users": 100,
  "processed_users": 41,
  "created_at": "2025-01-08T12:00:00.000000+00:00",
  "completed_at": null,
  "error_message": "Failed to parse CSV file"
}

```

###### Status Codes

| Code | Name | Description |
| --- | --- | --- |
| 0 | UNSPECIFIED | The default value. |
| 1 | PROCESSING | The job is currently processing. |
| 2 | COMPLETED | The job completed successfully. |
| 3 | FAILED | The job failed; inspect `error_message` for details. |

## Lobby Object

### Lobby Object

 Represents a Discord lobby. See [Managing Lobbies](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-lobbies) for details.

###### Lobby Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | the id of this channel |
| application_id | snowflake | application that created the lobby |
| metadata | ?dict<string, string> | string key/value dictionary; maximum combined length 1000. |
| members | array of lobby member objects | members of the lobby |
| linked_channel? | channel object | guild channel connected to the lobby |

### Lobby Member Object

 Represents a lobby member, including optional metadata and flags.

###### Lobby Member Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | the id of the user |
| metadata? | ?dict<string, string> | string key/value dictionary; maximum combined length 1000. |
| flags? | integer | lobby member flags represented as a combined bitfield |

###### Lobby Member Flags

| Flag | Value | Description |
| --- | --- | --- |
| CanLinkLobby | `1<<0` | user has permission to link a lobby text channel |

###### Example Lobby Object

```
{
  "id": "96008815106887111",
  "application_id": "41771983429993937",
  "metadata": {
    "topic": "we need more redstone",
  },
  "members": [
    {
      "id": "41771983429993000",
      "metadata": null,
      "flags": 1
    }
  ]
}

```

### Create Lobby

  Creates a lobby and optionally adds listed members. On success, returns a [lobby](https://docs.discord.com/developers/resources/lobby#lobby-object) object. Lobbies created with this API cannot be joined or left by [Discord Social SDK](https://docs.discord.com/developers/discord-social-sdk/overview) clients, including through [`Client::CreateOrJoinLobby`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a8b4e195555ecaa89ccdfc0acd28d3512). See [Managing Lobbies](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-lobbies#server-side-lobby-management) for more details.

### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| metadata? | ?dict<string, string> | optional string key/value dictionary; maximum combined length 1000. |
| members? | array of lobby member objects | optional array containing up to 25 users to add to the lobby |
| idle_timeout_seconds? | integer | seconds to wait before shutting down an idle lobby. Allowed range is 5 to 604800 (7 days). See `LobbyHandle` for behavior details. |

#### Lobby Member JSON Params

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | Discord user id for the lobby member to add |
| metadata? | ?dict<string, string> | optional string key/value dictionary; maximum combined length 1000. |
| flags? | integer | lobby member flags represented as a combined bitfield |

### Create or Join Lobby

  Creates a lobby for the application identified by `secret`; if that same secret already has a lobby, the caller joins it. Joining updates lobby metadata and the caller's member metadata. Authorization uses a `Bearer` token with [`sdk.social_layer`](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes#default-communication-scopes) scope. A [lobby](https://docs.discord.com/developers/resources/lobby#lobby-object) object is returned.

### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| secret | string | secret that identifies the lobby. If this application already has a lobby with the secret, the caller joins it; otherwise Discord creates one. Maximum 250 characters. |
| idle_timeout_seconds? | integer | seconds to wait before shutting down an idle lobby. Allowed range is 5 to 604800 (7 days). See `LobbyHandle` for behavior details. |
| lobby_metadata? | ?dict<string, string> | optional string key/value dictionary for lobby metadata. Maximum combined length 1000. Replaces existing lobby metadata. |
| member_metadata? | ?dict<string, string> | optional string key/value dictionary for the calling user's lobby member. Maximum combined length 1000. |

### Get Lobby

  Returns the requested [lobby](https://docs.discord.com/developers/resources/lobby#lobby-object) object when the specified lobby id exists.

### Modify Lobby

  Updates the specified lobby with any provided values. Returns the updated [lobby](https://docs.discord.com/developers/resources/lobby#lobby-object) object.

### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| metadata? | ?dict<string, string> | optional string key/value dictionary. Maximum combined length 1000. Replaces existing metadata. |
| members? | array of lobby member objects | optional replacement list of up to 25 users for lobby membership. Members absent from this list are removed from the lobby. |
| idle_timeout_seconds? | integer | seconds to wait before shutting down an idle lobby. Allowed range is 5 to 604800 (7 days). See `LobbyHandle` for behavior details. |

### Delete Lobby

  Deletes the specified lobby when it exists. Calling this on an already deleted lobby is safe. Returns nothing.

### Add a Member to a Lobby

  Adds the given user to the specified lobby. If the user is already a lobby member, fields such as that user's metadata are updated instead. Returns the [lobby member](https://docs.discord.com/developers/resources/lobby#lobby-member-object) object.

### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| metadata? | ?dict<string, string> | optional string key/value dictionary; maximum combined length 1000. |
| flags? | integer | lobby member flags represented as a combined bitfield |

### Bulk Update Lobby Members

  In one request, add, update, or remove up to 25 members from a lobby. A member with `remove_member: false` uses the default upsert behavior: add when absent, or update metadata and flags when present. A member with `remove_member: true` is removed. The response includes [lobby member](https://docs.discord.com/developers/resources/lobby#lobby-member-object) objects for upserts only and omits removed members. Unknown Discord users produce 404 UNKNOWN_USER. Permission failures, or users at the per-application lobby maximum who are not already in this lobby, are silently left out of the upsert results.

### JSON Params

 Member object array. Minimum 1, maximum 25.
| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | Discord user id for the lobby user to add, update, or remove |
| metadata? | ?dict<string, string> | optional string key/value dictionary; maximum combined length 1000. |
| flags? | integer | lobby member flags represented as a combined bitfield |
| remove_member? | boolean | when `true`, remove the user from the lobby instead of upserting. Default `false`. |

### Remove a Member from a Lobby

  Removes a provided user from the lobby. Calling this for a former member is safe; a missing lobby makes the request fail. No response body is returned.

### Leave Lobby

  Removes the caller from the lobby. Calling this after the caller has left is safe, but the request fails if the lobby does not exist. Authorization uses a `Bearer` token. Returns nothing.

### Link Channel to Lobby

  Connects a lobby with an existing text channel. See [Linked Channels](https://docs.discord.com/developers/discord-social-sdk/development-guides/linked-channels) for details. Authorization uses a `Bearer` token, and the operation requires a lobby member holding the `CanLinkLobby` [lobby member flag](https://docs.discord.com/developers/resources/lobby#lobby-member-object-lobby-member-flags). A [lobby](https://docs.discord.com/developers/resources/lobby#lobby-object) object containing a linked channel is returned.

### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| channel_id? | snowflake | channel id to link with the lobby. If omitted, any current linked channel is removed from the lobby. |

### Unlink Channel from Lobby

  Clears any channels linked to the specified lobby. Send this endpoint an empty body to remove current links. Authorization uses a `Bearer` token, and the operation requires a lobby member holding the `CanLinkLobby` [lobby member flag](https://docs.discord.com/developers/resources/lobby#lobby-member-object-lobby-member-flags). The returned [lobby](https://docs.discord.com/developers/resources/lobby#lobby-object) object has no linked channel.

### Send Lobby Message

  Posts into the specified lobby. The caller must be a lobby member. The created lobby message object is returned. Authorization uses a `Bearer` token with the [`sdk.social_layer`](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes#default-communication-scopes) scope. When the lobby includes [a linked channel](https://docs.discord.com/developers/discord-social-sdk/development-guides/linked-channels), Discord forwards the message there too. If forwarding fails, for example because of AutoMod, other lobby members still receive the lobby message.

### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| content | string | message content. Must not be empty. |
| metadata? | ?dict<string, string> | optional string key/value dictionary sent with the message to active clients through the Social SDK. It is not stored on the linked channel message. |
| flags? | integer | optional message flags represented as a combined bitfield. Only Social SDK-creatable flags are accepted. |

### Lobby Message Object

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | id of the message |
| type | integer | message type |
| content | string | message content |
| lobby_id | snowflake | id of the lobby where this message was sent |
| channel_id | snowflake | included for message-interface compatibility; equal to `lobby_id` |
| author | user object | user who sent the message |
| metadata? | ?dict<string, string> | dispatch-only metadata sent with the message |
| moderation_metadata? | ?dict<string, string> | moderation metadata set through Update Lobby Message Moderation Metadata |
| flags | integer | message flags bitfield |
| application_id | snowflake | application that sent the message |

### Get Lobby Messages

  Returns the newest messages in the specified lobby. The caller must be a lobby member. Authorization uses a `Bearer` token with the [`sdk.social_layer`](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes#default-communication-scopes) scope. For the object shape, see [Send Lobby Message](https://docs.discord.com/developers/resources/lobby#send-lobby-message); the response contains lobby message objects.

### Query Params

| Field | Type | Description |
| --- | --- | --- |
| limit? | integer | maximum returned messages (1-200). Default 50. |

### Update Lobby Message Moderation Metadata

  Sets moderation metadata for a lobby message. Metadata is scoped to the app and reaches active game clients through the Social SDK in realtime through a message update. The complete moderation flow is in [Integrate Moderation](https://docs.discord.com/developers/discord-social-sdk/how-to/integrate-moderation#server-side-chat-moderation). Authorization uses a `Bot` token. Success returns `HTTP 204: No Content`.

### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| * | string | Moderation metadata as free-form key-value pairs. Limit: 5 keys maximum; keys can be <= 1024 characters and values can be <= 2000 characters. |

### Create Lobby Channel Invite for Self

  Generates a single-use guild invite to the lobby's [linked channel](https://docs.discord.com/developers/discord-social-sdk/development-guides/linked-channels), for the calling user. The lobby needs a linked channel; membership is required for the caller. The invite expires after one hour. A [lobby invite object](https://docs.discord.com/developers/resources/lobby#lobby-invite-object) is returned. Authorization uses a `Bearer` token with the [`sdk.social_layer`](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes#default-communication-scopes) scope.

### Create Lobby Channel Invite for User

  Generates a single-use guild invite to the lobby's [linked channel](https://docs.discord.com/developers/discord-social-sdk/development-guides/linked-channels) for an application and targets the specified user. The lobby needs a linked channel. After one hour, the invite expires. Authorization uses a `Bot` token. Returns a [lobby invite object](https://docs.discord.com/developers/resources/lobby#lobby-invite-object).

###### Lobby Invite Object

| Field | Type | Description |
| --- | --- | --- |
| code | string | invite code for the lobby's linked channel |

### Development Rate Limits

 Applications that do not have [increased rate limits for production releases](https://docs.discord.com/developers/discord-social-sdk/core-concepts/communication-features#applying-for-increased-rate-limits-for-production-releases) use these application-wide development limits:
| Endpoint | Limit |
| --- | --- |
| `POST /lobbies` | 100 per 2 hours |
| `PUT /lobbies` | 100 per 2 hours |
| `PATCH /lobbies/{lobby.id}` | 100 per 2 hours |
| `PUT /lobbies/{lobby.id}/members/{user.id}` | 100 per 2 hours |
| `DELETE /lobbies/{lobby.id}/members/{user.id}` | 100 per 2 hours |
| `POST /lobbies/{lobby.id}/members/bulk` | 100 per 2 hours |
| `POST /lobbies/{lobby.id}/members/@me/invites` | 100 per 2 hours |
| `POST /lobbies/{lobby.id}/members/{user.id}/invites` | 100 per 2 hours |
| `PATCH /lobbies/{lobby.id}/channel-linking` | 20 per 2 hours |
| `POST /lobbies/{lobby.id}/messages` | 100 per 2 hours |

 These limits apply per application, not per user. They are intended to support development, testing, and small demos while protecting system stability.
