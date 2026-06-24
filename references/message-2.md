# Message (part 2)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

| Field | Type | Description |
| --- | --- | --- |
| url? | string | source url of video |
| proxy_url? | string | proxied URL for the video |
| height? | integer | height of video |
| width? | integer | width of video |
| content_type? | string | the video’s media type |
| placeholder? | string | thumbhash placeholder of the video |
| placeholder_version? | integer | version of the placeholder |
| description? | string | video description used as alt text |
| flags? | integer | embed media flags packed into a bitfield |

###### Embed Image Structure

| Field | Type | Description |
| --- | --- | --- |
| url | string | image source URL; supports only http(s) and attachments |
| proxy_url? | string | proxied URL for the image |
| height? | integer | height of image |
| width? | integer | width of image |
| content_type? | string | the image’s media type |
| placeholder? | string | thumbhash placeholder of the image |
| placeholder_version? | integer | version of the placeholder |
| description? | string | image description used as alt text |
| flags? | integer | embed media flags packed into a bitfield |

###### Embed Media Flags

| Flag | Value | Description |
| --- | --- | --- |
| IS_ANIMATED | `1 << 5` | this image is animated |

###### Embed Provider Structure

| Field | Type | Description |
| --- | --- | --- |
| name? | string | name of provider |
| url? | string | url of provider |

###### Embed Author Structure

| Field | Type | Description |
| --- | --- | --- |
| name | string | name of author |
| url? | string | author URL; supports only http(s) |
| icon_url? | string | author icon URL; supports only http(s) and attachments |
| proxy_icon_url? | string | proxied URL for the author icon |

###### Embed Footer Structure

| Field | Type | Description |
| --- | --- | --- |
| text | string | footer text |
| icon_url? | string | footer icon URL; supports only http(s) and attachments |
| proxy_icon_url? | string | proxied URL for the footer icon |

###### Embed Field Structure

| Field | Type | Description |
| --- | --- | --- |
| name | string | name of the field |
| value | string | value of the field |
| inline? | boolean | whether the field renders inline |

###### Embed Limits

 Rich embeds support rich content and therefore use limits separate from ordinary message content. Size caps still prevent overly large embeds. The table lists the limits. Each limit is inclusive, and leading or trailing whitespace is trimmed before counting.
| Field | Limit |
| --- | --- |
| title | 256 characters |
| description | 4096 characters |
| fields | Up to 25 field objects |
| field.name | 256 characters |
| field.value | 1024 characters |
| footer.text | 2048 characters |
| author.name | 256 characters |

 Across all embeds on a message, the total characters in `title`, `description`, `field.name`, `field.value`, `footer.text`, and `author.name` must be 6000 or fewer. Any violation returns `Bad Request`. Embeds with the same URL are deduplicated; only the first one is displayed.

#### Embed Fields by Embed Type

 Some embed types drive special UIs and store extra key-value data in [fields](https://docs.discord.com/developers/resources/message#embed-object-embed-field-structure). The list below shows possible fields for those embed types.

###### Poll Result Embed Fields

| Field | Description |
| --- | --- |
| poll_question_text | text of the original poll question |
| victor_answer_votes | vote count for the top answer(s) |
| total_votes | total poll vote count |
| victor_answer_id? | id for the winning answer |
| victor_answer_text? | text for the winning answer |
| victor_answer_emoji_id? | emoji id attached to the winning answer |
| victor_answer_emoji_name? | emoji name attached to the winning answer |
| victor_answer_emoji_animated? | whether the winning answer emoji is animated |

### Attachment Object

###### Attachment Structure

 In Message Create/Edit requests, only `id` is required inside the `attachments` array.
| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | attachment id |
| filename | string | name of file attached |
| title? | string | the title of the file |
| description? | string | file description used as alt text (max 1024 characters) |
| content_type? | string | the attachment’s media type |
| size | integer | size of file in bytes |
| url | string | source url of file |
| proxy_url | string | a proxied url of file |
| height? | ?integer | file height, for images or videos |
| width? | ?integer | file width, for images or videos |
| placeholder? | string | thumbhash placeholder for images or videos |
| placeholder_version? | integer | placeholder version for images or videos |
| ephemeral? * | boolean | whether this attachment is ephemeral |
| duration_secs? | float | audio duration, currently used for voice messages |
| waveform? | string | base64 encoded bytearray for a sampled waveform, currently used for voice messages |
| flags? | integer | attachment flags packed into a bitfield |
| clip_participants? | array of user objects | for Clips, users present in the stream |
| clip_created_at? | ISO8601 timestamp | for Clips, the clip creation time |
| application? | ?application | for Clips, the stream application when recognized |

 * Ephemeral attachments are removed automatically after a set time. On messages, ephemeral attachments remain available for at least as long as the message exists.

###### Attachment Flags

| Flag | Value | Description |
| --- | --- | --- |
| IS_CLIP | `1 << 0` | attachment is a stream Clip |
| IS_THUMBNAIL | `1 << 1` | attachment is a media-channel thread thumbnail shown in the grid but not on the message |
| IS_REMIX | `1 << 2` | attachment was edited with the deprecated mobile remix feature |
| IS_SPOILER | `1 << 3` | attachment is spoiler-marked and blurred until clicked |
| IS_ANIMATED | `1 << 5` | attachment is an animated image |

### Channel Mention Object

###### Channel Mention Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | id of the channel |
| guild_id | snowflake | guild id for the channel's guild |
| type | integer | the type of channel |
| name | string | the name of the channel |

### Allowed Mentions Object

 The `allowed_mentions` field controls whether mentions in message content or attached component content notify users. Discord validates it against your permissions and against mentions actually present in the message, preventing phantom pings where a user is notified without a visible mention. To ping everyone, for example, listing it in `allowed_mentions` is insufficient; `@everyone` must appear in the message content or components. This field **does not** guarantee a push notification, because other factors can change delivery:
- Role mentions notify members only when the role's `mentionable` field is `true` or the bot has `MENTION_EVERYONE` permission
- Mentions of `@everyone` and `@here` require the bot to have `MENTION_EVERYONE` permission
- Sending a message with `SUPPRESS_NOTIFICATIONS` disables push notifications and leaves only a notification badge
- User notification settings in the Discord app may limit delivery to a notification badge with no push notification

###### Allowed Mention Types

| Type | Value | Description |
| --- | --- | --- |
| Role Mentions | ”roles” | Controls role mentions |
| User Mentions | ”users” | Controls user mentions |
| Everyone Mentions | ”everyone” | Controls @everyone and @here mentions |

###### Default Settings for Allowed Mentions

 When the request body omits `allowed_mentions`, the default depends on context. In **regular messages**, every mention type is parsed, equivalent to this payload:

```
{
  "parse": ["users", "roles", "everyone"]
}

```

 In **interactions** and **webhooks**, the default parses only user mentions, equivalent to this payload:

```
{
  "parse": ["users"]
}

```

| Field | Type | Description |
| --- | --- | --- |
| parse? | array of allowed mention types | allowed mention types to parse from the content |
| roles? | array of snowflakes | role id array to mention, max 100 |
| users? | array of snowflakes | user id array to mention, max 100 |
| replied_user? | boolean | for replies, controls whether the referenced message author is mentioned; defaults to false |

###### Allowed Mentions Examples

 `allowed_mentions` behavior has several edge cases. In this example, a regular message is sent **without** `allowed_mentions`, so every included mention is parsed.

```
{
  "content": "@here Hello <@&1234> and <@5678> 👋"
}

```

 To suppress every mention in the message, set `allowed_mentions` as shown above:

```
{
  "content": "@here Hello <@&1234> and <@5678> 👋",
  "allowed_mentions": {
    "parse": []
  }
}

```

 `parse` is **mutually exclusive** with the other fields. In the example below, only role `1234` and user `5678` are parsed; the leading `@here` is not. Passing a falsy value such as `null` or an empty array to `users` is still valid.

```
{
  "content": "@here Hello <@&1234> and <@5678> 👋",
  "allowed_mentions": {
    "parse": ["users", "roles"],
    "users": []
  }
}

```

 This example parses **only** `@everyone`, plus users `1234` and `5678` if those users suppressed `@everyone` mentions in their settings.

```
{
  "content": "@everyone <@1234> <@5678> <@&789> 👋",
  "allowed_mentions": {
    "parse": ["everyone"],
    "users": ["1234", "5678"]
  }
}

```

 Some configurations are rejected because they are ambiguous. This *invalid* example includes both `parse` and `users`, which are mutually exclusive, so validation fails.

```
{
  "content": "@everyone <@1234> <@5678> <@9012> <@&200>",
  "allowed_mentions": {
    "parse": ["users"],
    "users": ["1234", "5678"]
  }
}

```

 Any entity whose id appears in the field can be mentioned, but the API silently ignores ids in `allowed_mentions` that do not appear in the message content or components. In the example below, user 123 is parsed because it appears in `content`; user 456 is absent from `content`, so no notification is sent.

```
{
  "content": "<@123> Time for some memes 🤠",
  "allowed_mentions": {
    "users": ["123", "456"]
  }
}

```

### Role Subscription Data Object

###### Role Subscription Data Object Structure

| Field | Type | Description |
| --- | --- | --- |
| role_subscription_listing_id | snowflake | sku and listing id the user subscribed to |
| tier_name | string | tier name for the user's subscription |
| total_months_subscribed | integer | total number of months the user has subscribed |
| is_renewal | boolean | whether the notification represents a renewal instead of a new purchase |

### Message Pin Object

###### Message Pin Object Struture

| Field | Type | Description |
| --- | --- | --- |
| pinned_at | ISO8601 timestamp | timestamp when the message was pinned |
| message | message object | the pinned message |

### Shared Client Theme Object

###### Shared Client Theme Object Struture

| Field | Type | Description |
| --- | --- | --- |
| colors | array of strings | hexadecimal-encoded theme colors (max of 5) |
| gradient_angle | integer | theme color direction (max of 360) |
| base_mix | integer | theme color intensity (max of 100) |
| base_theme? | ?base theme | the mode of the theme |

###### Base Theme Types

| Type | Value |
| --- | --- |
| UNSET [1] | 0 |
| DARK | 1 |
| LIGHT | 2 |
| DARKER | 3 |
| MIDNIGHT | 4 |

 [1] Equivalent to the `DARK` type.

###### Example Shared Client Theme

```
{
  "shared_client_theme": {
    "colors": ["5865F2", "7258F2", "9858F2", "BE58F2", "E558F2"],
    "gradient_angle": 0,
    "base_mix": 58,
    "base_theme": 1
  }
}

```

### Get Channel Messages

  Gets messages from a channel. On success, returns an array of [message](https://docs.discord.com/developers/resources/message#message-object) objects ordered newest to oldest. For a guild channel, the current user needs `VIEW_CHANNEL`; for a voice channel, the user needs `CONNECT` as well. Without `READ_MESSAGE_HISTORY`, the channel returns no messages. The `before`, `after`, and `around` parameters are mutually exclusive; send only one.

###### Query String Params

| Field | Type | Description | Default |
| --- | --- | --- | --- |
| around? | snowflake | message ID to center results around | absent |
| before? | snowflake | message ID to fetch messages before | absent |
| after? | snowflake | message ID to fetch messages after | absent |
| limit? | integer | maximum messages to return (1-100) | 50 |

### Search Guild Messages

  Returns guild messages matching a search query, without the `reactions` key. Requires `READ_MESSAGE_HISTORY`. Availability is limited by whether the application's `MESSAGE_CONTENT` [Privileged Intent](https://docs.discord.com/developers/events/gateway#privileged-intents) is enabled. If the searched entity is not indexed yet, the endpoint returns a 202 accepted response with no search results, using a body similar to an error response:

```
{
  "message": "Index not yet available. Try again later",
  "code": 110000,
  "documents_indexed": 0,
  "retry_after": 2
}

```

Retry after the duration in `retry_after`. If `retry_after` is `0`, wait a short time before retrying. Search optimizations can return slightly fewer messages than the requested limit for messages that have not been accessed recently, so do not use the `messages` array length for pagination. When messages are being created or deleted, `total_results` may be inaccurate.

###### Query String Params

| Field | Type | Description |
| --- | --- | --- |
| limit? | integer | maximum messages to return (1-25, default 25) |
| offset? | integer | offset for returned messages (max 9975) |
| max_id? | snowflake | message ID to search before |
| min_id? | snowflake | message ID to search after |
| slop? | integer | maximum skipped words between matching tokens in search `content` (max 100, default 2) |
| content? | string | content filter for messages (max 1024 characters) |
| channel_id? | array of snowflakes | channel filter for messages (max 500) |
| author_type? | array of strings | Filter messages by author type |
| author_id? | array of snowflakes | author filter for messages (max 100) |
| mentions? | array of snowflakes | filter for messages mentioning these users (max 100) |
| mentions_role_id? | array of snowflakes | filter for messages mentioning these roles (max 100) |
| mention_everyone? | boolean | filter by whether messages mention @everyone |
| replied_to_user_id? | array of snowflakes | filter for messages replying to these users (max 100) |
| replied_to_message_id? | array of snowflakes | filter for messages replying to these messages (max 100) |
| pinned? | boolean | filter by pinned or unpinned messages |
| has? | array of strings | filter by whether messages contain selected content types |
| embed_type? | array of strings | Filter messages by embed type |
| embed_provider? | array of strings | embed provider filter, case-sensitive, e.g. `Tenor` (max 256 characters, max 100) |
| link_hostname? | array of strings | link hostname filter, e.g. `discordapp.com` (max 256 characters, max 100) |
| attachment_filename? | array of strings | attachment filename filter (max 1024 characters, max 100) |
| attachment_extension? | array of strings | attachment extension filter, e.g. `txt` (max 256 characters, max 100) |
| sort_by? [1] | string | The sorting algorithm to use |
| sort_order? [1] | string | sort direction (`asc` or `desc`, default `desc`) |
| include_nsfw? | boolean | whether age-restricted channel results are included (default false) |

 [1] Relevance sorting does not honor sort order.

###### Author Types

 Prefix any type with `-` to negate it, excluding messages that match that type.
| Type | Description |
| --- | --- |
| user | Return messages from user accounts |
| bot | Return messages from bot accounts |
| webhook | Return messages sent by webhooks |

###### Search Has Types

 Prefix any type with `-` to negate it, excluding messages that match that type.
| Type | Description |
| --- | --- |
| image | Return messages containing an image |
| sound | Return messages containing a sound attachment |
| video | Return messages containing a video |
| file | Return messages containing an attachment |
| sticker | Return messages containing a sent sticker |
| embed | Return messages containing an embed |
| link | Return messages containing a link |
| poll | Return messages containing a poll |
| snapshot | Return messages containing a forwarded message |

###### Search Embed Types

 These values are broader than actual [embed types](https://docs.discord.com/developers/resources/message#embed-object-embed-types) and do not map 1:1 to them.
| Type | Description |
| --- | --- |
| image | Return messages containing an image embed |
| video | Return messages containing a video embed |
| gif [1] | Return messages containing a gifv embed |
| sound | Return messages containing a sound embed |
| article | Return messages containing an article embed |

 [1] Messages sent before February 24, 2026 may not be indexed correctly under the `gif` embed type.

###### Search Sort Modes

| Type | Description |
| --- | --- |
| timestamp | Sort by message creation time (default) |
| relevance | Sort by search-query relevance |

###### Response Body

| Field | Type | Description |
| --- | --- | --- |
| doing_deep_historical_index | boolean | whether the guild is running a deep historical indexing operation |
| documents_indexed? | integer | documents indexed during the current index operation, if one is running |
| total_results | integer | total results matching the query |
| messages [1] | array of message object arrays | nested message arrays matching the query |
| threads? | array of channel objects | threads containing the returned messages |
| members? | array of thread member objects | thread member object for each returned thread joined by the current user |

 [1] The nested array formerly supplied surrounding search-result context. Surrounding context is no longer returned.

### Get Channel Message

  Gets one message from the channel. On success, returns a [message](https://docs.discord.com/developers/resources/message#message-object) object. In guild channels, the current user needs `VIEW_CHANNEL` and `READ_MESSAGE_HISTORY`; in voice channels, `CONNECT` is required too.

### Create Message

  Discord may remove characters from message content, including invalid unicode or characters that trigger unexpected formatting. Sanitize user-generated strings before placing them in content, and use `allowed_mentions` to prevent unwanted mentions. This endpoint posts a message to a guild text or DM channel, returns a [message](https://docs.discord.com/developers/resources/message#message-object) object, and fires a [Message Create](https://docs.discord.com/developers/events/gateway-events#message-create) Gateway event. See [message formatting](https://docs.discord.com/developers/reference#message-formatting) for formatting rules. To create a reply or forwarded message, include [`message_reference`](https://docs.discord.com/developers/resources/message#message-reference-structure) with the required fields. Attach files with a `multipart/form-data` body as described in [Uploading Files](https://docs.discord.com/developers/reference#uploading-files).

###### Limitations

- In guild channels, the current user needs `SEND_MESSAGES`.
- Sending a message with `tts` (text-to-speech) set to `true` requires `SEND_TTS_MESSAGES`.
- Replies require the current user to have `READ_MESSAGE_HISTORY`.
  - The referenced message must exist and must not be a system message.

- Message send requests can be at most **25 MiB**.
- For the embed object, every field can be set except `type` (it remains `rich` even if another value is attempted), `provider`, `video`, and image `height`, `width`, or `proxy_url` values.

###### JSON/Form Params

| Field | Type | Description |
| --- | --- | --- |
| content?* | string | Message contents (up to 2000 characters) |
| nonce? | integer or string | Verifies that a message was sent, up to 25 characters; appears in the Message Create event. |
| tts? | boolean | `true` for a TTS message |
| embeds?* | array of embed objects | Up to 10 `rich` embeds, up to 6000 characters |
| allowed_mentions? | allowed mention object | Allowed mentions for the message |
| message_reference?* | message reference | Use to make the message a reply or forward |
| components?* | array of message component objects | message components to include |
| sticker_ids?* | array of snowflakes | IDs for up to 3 server stickers to send |
| files[n]?* | file contents | file contents to send; see Uploading Files |
| payload_json? | string | JSON-encoded non-file request body, only for `multipart/form-data`; see Uploading Files |
| attachments? | array of partial attachment objects | attachment objects with filenames and descriptions; see Uploading Files |
| flags?** | integer | message flags packed into a bitfield; only `SUPPRESS_EMBEDS`, `SUPPRESS_NOTIFICATIONS`, `IS_VOICE_MESSAGE`, and `IS_COMPONENTS_V2` can be set |
| enforce_nonce? | boolean | When true and nonce is present, Discord checks whether the same author used that nonce in the past few minutes. If a matching message exists, Discord returns it and does not create another message. |
| poll? | poll request object | A poll! |
| shared_client_theme? | shared client theme object | custom client-side theme shared through the message |
