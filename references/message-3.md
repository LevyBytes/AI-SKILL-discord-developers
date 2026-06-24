# Message (part 3)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

* Provide at least one of `content`, `embeds`, `sticker_ids`, `components`, `files[n]`, `poll`, or `shared_client_theme`. For message forwarding, `message_reference` alone is sufficient. ** If `IS_COMPONENTS_V2` is set, the message may contain only `components`; including `content`, `embeds`, `sticker_ids`, `poll`, or `shared_client_theme` causes a 400 BAD REQUEST response.

###### Example Request Body (application/json)

```
{
  "content": "Hello, World!",
  "tts": false,
  "embeds": [{
    "title": "Hello, Embed!",
    "description": "This is an embedded message."
  }]
}

```

 See [Uploading Files](https://docs.discord.com/developers/reference#uploading-files) for file upload examples.

### Crosspost Message

  Publishes an Announcement Channel message to follower channels. If the current user authored the message, `SEND_MESSAGES` is required; for messages from other users, the current user also needs `MANAGE_MESSAGES`. A successful call returns a [message](https://docs.discord.com/developers/resources/message#message-object) object and emits a [Message Update](https://docs.discord.com/developers/events/gateway-events#message-update) Gateway event.

### Create Reaction

  Adds a reaction to the message. The current user must have `READ_MESSAGE_HISTORY`. If this emoji has not already been used as a reaction on the message, the current user also needs `ADD_REACTIONS`. Success returns a 204 empty response and emits a [Message Reaction Add](https://docs.discord.com/developers/events/gateway-events#message-reaction-add) Gateway event. The `emoji` value must be [URL Encoded](https://en.wikipedia.org/wiki/Percent-encoding), or Discord returns `10014: Unknown Emoji`. Custom emoji must be encoded as `name:id`, using the emoji name and emoji id.

### Delete Own Reaction

  Removes the current user's reaction from the message. Success returns a 204 empty response and emits a [Message Reaction Remove](https://docs.discord.com/developers/events/gateway-events#message-reaction-remove) Gateway event. The `emoji` value must be [URL Encoded](https://en.wikipedia.org/wiki/Percent-encoding), or Discord returns `10014: Unknown Emoji`. Custom emoji must be encoded as `name:id`, using the emoji name and emoji id.

### Delete User Reaction

  Removes a reaction made by another user. The current user must have `MANAGE_MESSAGES`. Success returns a 204 empty response and emits a [Message Reaction Remove](https://docs.discord.com/developers/events/gateway-events#message-reaction-remove) Gateway event. The `emoji` value must be [URL Encoded](https://en.wikipedia.org/wiki/Percent-encoding), or Discord returns `10014: Unknown Emoji`. Custom emoji must be encoded as `name:id`, using the emoji name and emoji id.

### Get Reactions

  Fetches users who reacted with the specified emoji. A successful response is an array of [user](https://docs.discord.com/developers/resources/user#user-object) objects. The `emoji` value must be [URL Encoded](https://en.wikipedia.org/wiki/Percent-encoding), or Discord returns `10014: Unknown Emoji`. Custom emoji must be encoded as `name:id`, using the emoji name and emoji id.

###### Query String Params

| Field | Type | Description | Default |
| --- | --- | --- | --- |
| type? | integer | The type of reaction | 0 |
| after? | snowflake | Return users after this user ID | absent |
| limit? | integer | Maximum users to return (1-100) | 25 |

###### Reaction Types

| Type | Value |
| --- | --- |
| NORMAL | 0 |
| BURST | 1 |

### Delete All Reactions

  Removes every reaction from a message. The current user must have `MANAGE_MESSAGES`. This emits a [Message Reaction Remove All](https://docs.discord.com/developers/events/gateway-events#message-reaction-remove-all) Gateway event.

### Delete All Reactions for Emoji

  Removes all reactions for one emoji on a message. The current user must have `MANAGE_MESSAGES`. This emits a [Message Reaction Remove Emoji](https://docs.discord.com/developers/events/gateway-events#message-reaction-remove-emoji) Gateway event. The `emoji` value must be [URL Encoded](https://en.wikipedia.org/wiki/Percent-encoding), or Discord returns `10014: Unknown Emoji`. Custom emoji must be encoded as `name:id`, using the emoji name and emoji id.

### Edit Message

  Updates a message that was previously sent. The message author may edit `content`, `embeds`, `flags`, and `components`. Other users may edit only `flags`, and only with `MANAGE_MESSAGES` in the relevant channel. When sending `flags`, include all existing flag bits along with the bits being changed. Users may modify only the `flags` listed below; unsupported flag edits are currently ignored without an error. Editing `content` rebuilds the message object's `mentions`, `mention_roles`, and `mention_everyone` values from the new content. If `IS_COMPONENTS_V2` is set, those reconstructed values come from edited content in the `components` array. The edit request's `allowed_mentions` controls that parsing. Without an explicit `allowed_mentions` in the edit request, Discord parses content with *default* allowances, regardless of whether the original create request included `allowed_mentions`. A successful call returns a [message](https://docs.discord.com/developers/resources/message#message-object) object and emits a [Message Update](https://docs.discord.com/developers/events/gateway-events#message-update) Gateway event. See [Uploading Files](https://docs.discord.com/developers/reference#uploading-files) for attachment and `multipart/form-data` details. Files supplied in the edit are **appended**. To remove or replace files, provide the `attachments` field listing the files that should remain on the message after the edit. Starting with API v10, the `attachments` array must list every attachment that should exist after editing, including **retained and new** attachments sent in the request body. All parameters for this endpoint are optional and nullable.

###### JSON/Form Params

| Field | Type | Description |
| --- | --- | --- |
| content | string | Message contents (up to 2000 characters) |
| embeds | array of embed objects | Up to 10 `rich` embeds, with up to 6000 characters |
| flags* | integer | Change message flags (`SUPPRESS_EMBEDS` and `IS_COMPONENTS_V2` only) |
| allowed_mentions | allowed mention object | Allowed mentions for the message |
| components | array of message component | Components that should be included with the message |
| files[n] | file contents | File contents being sent or edited. See Uploading Files |
| payload_json | string | JSON-encoded non-file parameters for `multipart/form-data` requests only. See Uploading Files |
| attachments | array of attachment objects | Existing attachments to retain plus possible descriptions for new files. See Uploading Files |

 * `SUPPRESS_EMBEDS` may be set or unset, but `IS_COMPONENTS_V2` may only be set. When setting `IS_COMPONENTS_V2`, any used `content`, `embeds`, `sticker_ids`, or `poll` values must be cleared. Use `null` for `content` and `poll`; use `[]` for `embeds` and `sticker_ids`. Otherwise the request fails with 400 BAD REQUEST.

### Delete Message

  Deletes a message. In a guild channel, deleting a message sent by someone other than the current user requires `MANAGE_MESSAGES`. Success returns a 204 empty response and emits a [Message Delete](https://docs.discord.com/developers/events/gateway-events#message-delete) Gateway event. The endpoint accepts the `X-Audit-Log-Reason` header.

### Bulk Delete Messages

  Deletes multiple messages in one request. This endpoint is limited to guild channels and requires `MANAGE_MESSAGES`. Success returns a 204 empty response and emits a [Message Delete Bulk](https://docs.discord.com/developers/events/gateway-events#message-delete-bulk) Gateway event. Message IDs that are invalid or do not exist still count toward the minimum and maximum message counts, currently 2 and 100. The request fails with 400 BAD REQUEST if any supplied message is older than 2 weeks or if the same message ID appears more than once. The endpoint accepts the `X-Audit-Log-Reason` header.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| messages | array of snowflakes | array of message ids to delete (2-100) |

### Get Channel Pins

  Gets pins for a channel. The current user needs `VIEW_CHANNEL`. If that user lacks `READ_MESSAGE_HISTORY` in the channel, the response contains no pins.

###### Query String Params

| Field | Type | Description | Default |
| --- | --- | --- | --- |
| before? | ISO8601 timestamp | Return messages pinned before this timestamp | absent |
| limit? | integer | Maximum pins to return (1-50) | 50 |

###### Response Structure

| Field | Type |
| --- | --- |
| items | array of message pin objects |
| has_more | boolean |

###### Example

 To retrieve 100 pins, send these two requests: `GET /channels/:id/messages/pins?limit=50` `GET /channels/:id/messages/pins?limit=50&before={pins[pins.len() - 1].pinned_at}`

### Pin Message

  Pins a message in a channel. The current user needs `PIN_MESSAGES`. This emits a [Channel Pins Update](https://docs.discord.com/developers/events/gateway-events#channel-pins-update) Gateway event. The endpoint accepts the `X-Audit-Log-Reason` header.

### Unpin Message

  Removes a pin from a channel message. The current user needs `PIN_MESSAGES`. Success returns a 204 empty response and emits a [Channel Pins Update](https://docs.discord.com/developers/events/gateway-events#channel-pins-update) Gateway event. The endpoint accepts the `X-Audit-Log-Reason` header.

### Get Pinned Messages (deprecated)

  Returns the first 50 pinned messages in a channel as an array of [message](https://docs.discord.com/developers/resources/message#message-object) objects. This endpoint is deprecated; use [Get Channel Pins](https://docs.discord.com/developers/resources/message#get-channel-pins) instead.

### Pin Message (deprecated)

  This endpoint is deprecated; use [Pin Message](https://docs.discord.com/developers/resources/message#pin-message) instead.

### Unpin Message (deprecated)

  This endpoint is deprecated; use [Unpin Message](https://docs.discord.com/developers/resources/message#unpin-message) instead.
