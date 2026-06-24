# Channel (part 2)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

| Field | Type | Description |
| --- | --- | --- |
| allow? | string? | Bitwise value for every allowed permission; default is `"0"`. |
| deny? | string? | Bitwise value for every denied permission; default is `"0"`. |
| type | integer | Use 0 for a role and 1 for a member. |

### Get Channel Invites

  Return the channel's [invite](https://docs.discord.com/developers/resources/invite#invite-object) objects, including [invite metadata](https://docs.discord.com/developers/resources/invite#invite-metadata-object). This route works only for guild channels and requires `MANAGE_CHANNELS`.

### Create Channel Invite

  Create a channel [invite](https://docs.discord.com/developers/resources/invite#invite-object). The route is limited to guild channels and requires `CREATE_INSTANT_INVITE`. All JSON parameters are optional, but the body itself is required; send an empty JSON object (`{}`) when no fields are provided. The response is an [invite](https://docs.discord.com/developers/resources/invite#invite-object) object, and Discord fires an [Invite Create](https://docs.discord.com/developers/events/gateway-events#invite-create) Gateway event. `X-Audit-Log-Reason` is supported.

###### JSON/Form Params

| Field | Type | Description | Default |
| --- | --- | --- | --- |
| max_age | integer | Invite lifetime in seconds before expiration, or 0 for no expiration; valid range is 0 through 604800 (7 days). | 86400 (24 hours) |
| max_uses | integer | Maximum use count, or 0 for unlimited use; valid range is 0 through 100. | 0 |
| temporary | boolean | Whether the invite grants only temporary membership. | false |
| unique | boolean | When true, Discord should not reuse a similar invite; useful when generating many unique one-time-use invites. | false |
| target_type | integer | Target type for this voice channel invite. |  |
| target_user_id | snowflake | ID of the user whose stream should appear for this invite. Required when `target_type` is 1, and that user must be streaming in the channel. |  |
| target_application_id | snowflake | ID of the embedded application opened by this invite. Required when `target_type` is 2, and the application must include the `EMBEDDED` flag. |  |
| target_users_file?* | file | CSV file containing one column of user IDs for users allowed to accept the invite. |  |
| payload_json? | string | JSON-encoded non-file parameter body, used only with `multipart/form-data` requests. |  |
| role_ids?** | array of snowflakes | Role ID(s) for guild roles assigned to users who accept the invite. |  |

 * Use `multipart/form-data` as the content type, with other parameters included as form fields in the multipart body. Uploads containing invalid user IDs return 400 and identify those IDs. Duplicate user IDs in the file are ignored. ** Requires `MANAGE_ROLES`, and the sender cannot assign roles whose permissions exceed their own.

### Delete Channel Permission

  Delete a permission overwrite for a channel user or role. This applies only to guild channels and requires `MANAGE_ROLES`. Success returns an empty 204 response and fires a [Channel Update](https://docs.discord.com/developers/events/gateway-events#channel-update) Gateway event. See [permissions](https://docs.discord.com/developers/topics/permissions#permissions) for additional permission behavior. `X-Audit-Log-Reason` is supported.

### Follow Announcement Channel

  Follow an Announcement Channel so its messages are sent into a target channel. The target channel requires `MANAGE_WEBHOOKS`. The response is a [followed channel](https://docs.discord.com/developers/resources/channel#followed-channel-object) object, and Discord fires a [Webhooks Update](https://docs.discord.com/developers/events/gateway-events#webhooks-update) Gateway event for that target channel. `X-Audit-Log-Reason` is supported.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| webhook_channel_id | snowflake | id of target channel |

### Trigger Typing Indicator

  Show a typing indicator in the specified channel for 10 seconds. A successful call returns an empty 204 response and fires a [Typing Start](https://docs.discord.com/developers/events/gateway-events#typing-start) Gateway event. Bots generally should **not** call this route, except when a command response needs several seconds of processing and the user should see that work is underway.

### Group DM Add Recipient

  Add a recipient to a Group DM by using that user's access token.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| access_token | string | Access token for a user who granted the app the `gdm.join` scope. |
| nick | string | Nickname for the user being added. |

### Group DM Remove Recipient

  Remove a recipient from a Group DM.

### Start Thread from Message

  Create a thread from an existing message. Success returns a [channel](https://docs.discord.com/developers/resources/channel#channel-object); invalid parameters return 400 BAD REQUEST. Discord fires both [Thread Create](https://docs.discord.com/developers/events/gateway-events#thread-create) and [Message Update](https://docs.discord.com/developers/events/gateway-events#message-update) Gateway events. Calling from `GUILD_TEXT` creates a `PUBLIC_THREAD`; calling from `GUILD_ANNOUNCEMENT` creates an `ANNOUNCEMENT_THREAD`. It cannot be used on [`GUILD_FORUM`](https://docs.discord.com/developers/resources/channel#start-thread-in-forum-or-media-channel) or `GUILD_MEDIA`. The new thread's id matches the source message id, so each message can have only one thread. `X-Audit-Log-Reason` is supported.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name | string | 1-100 character channel name |
| auto_archive_duration? | integer | After `auto_archive_duration` minutes without activity, the thread no longer appears in the channel list. Allowed values are 60, 1440, 4320, and 10080. |
| rate_limit_per_user? | ?integer | Seconds a user must wait before posting another message (0-21600). |

### Start Thread without Message

  Create a thread that is not tied to an existing message. Success returns a [channel](https://docs.discord.com/developers/resources/channel#channel-object), while invalid parameters return 400 BAD REQUEST. Discord fires a [Thread Create](https://docs.discord.com/developers/events/gateway-events#thread-create) Gateway event. `X-Audit-Log-Reason` is supported.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name | string | 1-100 character channel name |
| auto_archive_duration? | integer | After `auto_archive_duration` minutes without activity, the thread no longer appears in the channel list. Allowed values are 60, 1440, 4320, and 10080. |
| type?* | integer | Thread type to create. |
| invitable? | boolean | Whether non-moderators may add other non-moderators to the thread; available only for private thread creation. |
| rate_limit_per_user? | ?integer | Seconds a user must wait before posting another message (0-21600). |

 * `type` currently defaults to `PRIVATE_THREAD` to preserve the behavior from the initial thread documentation release. A later API version will make `type` required and remove the default.

### Start Thread in Forum or Media Channel

  Create a thread in a forum or media channel and send its first message. Success returns a [channel](https://docs.discord.com/developers/resources/channel#channel-object) with a nested [message](https://docs.discord.com/developers/resources/message#message-object) object; invalid parameters return 400 BAD REQUEST. Discord fires [Thread Create](https://docs.discord.com/developers/events/gateway-events#thread-create) and [Message Create](https://docs.discord.com/developers/events/gateway-events#message-create) Gateway events.
- The created thread uses type `PUBLIC_THREAD`.
- For correct message layout, see [message formatting](https://docs.discord.com/developers/reference#message-formatting).
- The current user needs `SEND_MESSAGES`; `CREATE_PUBLIC_THREADS` is ignored.
- Message-send requests can be at most **25 MiB**.
- For embeds, every field can be set except `type` (always `rich` even if another value is attempted), `provider`, `video`, and image `height`, `width`, or `proxy_url` values.
- File upload examples are in [Uploading Files](https://docs.discord.com/developers/reference#uploading-files).
- Attach files with a `multipart/form-data` body as documented in [Uploading Files](https://docs.discord.com/developers/reference#uploading-files).
- A message send must include **at least one of** `content`, `embeds`, `sticker_ids`, `components`, or `files[n]`.
 Discord may remove characters from message content, including invalid unicode or characters that would create unexpected formatting. When inserting user-generated strings into message content, sanitize them to avoid unexpected behavior and use `allowed_mentions` to prevent unwanted mentions. `X-Audit-Log-Reason` is supported.

###### JSON/Form Params

| Field | Type | Description |
| --- | --- | --- |
| name | string | 1-100 character channel name |
| auto_archive_duration?* | integer | Minutes after recent activity before the thread automatically archives; allowed values are 60, 1440, 4320, and 10080. |
| rate_limit_per_user? | ?integer | Seconds a user must wait before posting another message (0-21600). |
| message | a forum thread message params object | First message contents for the forum or media thread. |
| applied_tags? | array of snowflakes | IDs for the tag set applied to a thread in a `GUILD_FORUM` or `GUILD_MEDIA` channel. |
| files[n]?* | file contents | Contents of the file being uploaded. See Uploading Files. |
| payload_json? | string | JSON-encoded non-file parameter body, used only with `multipart/form-data` requests. See Uploading Files. |

###### Forum and Media Thread Message Params Object

 When sending a message, apps need a value for **at least one of** `content`, `embeds`, `sticker_ids`, `components`, or `files[n]`.
| Field | Type | Description |
| --- | --- | --- |
| content?* | string | Message contents (up to 2000 characters) |
| embeds?* | array of embed objects | Up to 10 `rich` embeds, with up to 6000 characters. |
| allowed_mentions? | allowed mention object | Allowed mentions for the message |
| components?* | array of message component objects | Message components to include. |
| sticker_ids?* | array of snowflakes | IDs for up to 3 server stickers to send with the message. |
| attachments? | array of partial attachment objects | Attachment objects containing `filename` and `description`. See Uploading Files. |
| flags? | integer | Message flags combined as a bitfield; only `SUPPRESS_EMBEDS` and `SUPPRESS_NOTIFICATIONS` may be set. |

 * One of `content`, `embeds`, `sticker_ids`, `components`, or `files[n]` is required at minimum.

### Join Thread

  Add the current user to a thread. The thread must not be archived. Success returns an empty 204 response and fires [Thread Members Update](https://docs.discord.com/developers/events/gateway-events#thread-members-update) plus [Thread Create](https://docs.discord.com/developers/events/gateway-events#thread-create) Gateway events.

### Add Thread Member

  Add another member to a thread. The caller must be able to send messages there, and the thread must not be archived. Success, including the case where the member already belongs to the thread, returns an empty 204 response and fires a [Thread Members Update](https://docs.discord.com/developers/events/gateway-events#thread-members-update) Gateway event.

### Leave Thread

  Remove the current user from a thread. The thread must not be archived. Success returns an empty 204 response and fires a [Thread Members Update](https://docs.discord.com/developers/events/gateway-events#thread-members-update) Gateway event.

### Remove Thread Member

  Remove another member from a thread. The caller needs `MANAGE_THREADS`, unless they created the `PRIVATE_THREAD`. The thread also must not be archived. Success returns an empty 204 response and fires a [Thread Members Update](https://docs.discord.com/developers/events/gateway-events#thread-members-update) Gateway event.

### Get Thread Member

  Return a [thread member](https://docs.discord.com/developers/resources/channel#thread-member-object) object for the specified user when that user belongs to the thread; otherwise return 404. If `with_member` is `true`, the thread member object includes a `member` field containing a [guild member](https://docs.discord.com/developers/resources/guild#guild-member-object) object.

###### Query String Params

| Field | Type | Description |
| --- | --- | --- |
| with_member? | boolean | Whether the thread member should include a guild member object. |

### List Thread Members

  Beginning with API v11, this endpoint always returns paginated results. Before API v11, pagination can be opted into by setting `with_member` to `true`; see [the changelog](https://docs.discord.com/developers/change-log#thread-member-details-and-pagination). The response is an array of [thread members](https://docs.discord.com/developers/resources/channel#thread-member-object) for users in the thread. With `with_member` set to `true`, results are paginated and each thread member object includes a `member` field containing a [guild member](https://docs.discord.com/developers/resources/guild#guild-member-object) object. Access depends on whether the app has the `GUILD_MEMBERS` [Privileged Intent](https://docs.discord.com/developers/events/gateway#privileged-intents) enabled.

###### Query String Params

| Field | Type | Description |
| --- | --- | --- |
| with_member? | boolean | Whether each thread member should include a guild member object. |
| after? | snowflake | Return thread members after this user ID. |
| limit? | integer | Maximum thread members to return, from 1-100. Defaults to 100. |

### List Public Archived Threads

  Return public archived threads for the channel. On a `GUILD_TEXT` channel, the returned threads have [type](https://docs.discord.com/developers/resources/channel#channel-object-channel-types) `PUBLIC_THREAD`. On a `GUILD_ANNOUNCEMENT` channel, the returned threads have [type](https://docs.discord.com/developers/resources/channel#channel-object-channel-types) `ANNOUNCEMENT_THREAD`. Threads are sorted by descending `archive_timestamp`. Requires `READ_MESSAGE_HISTORY`.

###### Query String Params

| Field | Type | Description |
| --- | --- | --- |
| before? | ISO8601 timestamp | Return threads archived before this timestamp. |
| limit? | integer | Optional cap on the number of threads returned. |

###### Response Body

| Field | Type | Description |
| --- | --- | --- |
| threads | array of channel objects | the public, archived threads |
| members | array of thread members objects | Thread member object for each returned thread joined by the current user. |
| has_more | boolean | Whether another request may return more threads. |

### List Private Archived Threads

  Return archived threads in the channel with [type](https://docs.discord.com/developers/resources/channel#channel-object-channel-types) `PRIVATE_THREAD`. Threads are sorted by descending `archive_timestamp`. Requires both `READ_MESSAGE_HISTORY` and `MANAGE_THREADS`.

###### Query String Params

| Field | Type | Description |
| --- | --- | --- |
| before? | ISO8601 timestamp | Return threads archived before this timestamp. |
| limit? | integer | Optional cap on the number of threads returned. |

###### Response Body

| Field | Type | Description |
| --- | --- | --- |
| threads | array of channel objects | the private, archived threads |
| members | array of thread members objects | Thread member object for each returned thread joined by the current user. |
| has_more | boolean | Whether another request may return more threads. |

### List Joined Private Archived Threads

  Return archived [type](https://docs.discord.com/developers/resources/channel#channel-object-channel-types) `PRIVATE_THREAD` threads in the channel that the user has joined. Threads are sorted by descending `id`. Requires `READ_MESSAGE_HISTORY`.

###### Query String Params

| Field | Type | Description |
| --- | --- | --- |
| before? | snowflake | returns threads before this id |
| limit? | integer | Optional cap on the number of threads returned. |

###### Response Body

| Field | Type | Description |
| --- | --- | --- |
| threads | array of channel objects | Private archived threads joined by the current user. |
| members | array of thread members objects | Thread member object for each returned thread joined by the current user. |
| has_more | boolean | Whether another request may return more threads. |
