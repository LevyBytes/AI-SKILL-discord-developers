# Webhook

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Webhooks provide a simple way to send messages into Discord channels without a bot user or authentication. Apps may also receive webhook events, also called outgoing webhooks, for events that occur *in* Discord; see [Webhook Events](https://docs.discord.com/developers/events/webhook-events).

### Webhook Object

 Used to represent a webhook.

###### Webhook Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | the id of the webhook |
| type | integer | the type of the webhook |
| guild_id? | ?snowflake | guild id associated with this webhook, when present |
| channel_id | ?snowflake | channel id associated with this webhook, when present |
| user? | user object | user that created the webhook; omitted when retrieving a webhook by token |
| name | ?string | webhook default name |
| avatar | ?string | webhook default avatar hash |
| token? | string | secure webhook token, returned for Incoming Webhooks |
| application_id | ?snowflake | bot or OAuth2 application that created the webhook |
| source_guild? * | partial guild object | guild for the followed channel, returned for Channel Follower Webhooks |
| source_channel? * | partial channel object | channel followed by this webhook, returned for Channel Follower Webhooks |
| url? | string | URL for executing the webhook, returned by the webhooks OAuth2 flow |

 * These fields are missing if the creator of the webhook no longer has access to the guild containing the followed channel

###### Webhook Types

 These webhook types exclude [webhook events](https://docs.discord.com/developers/events/webhook-events), which are outgoing webhooks Discord sends to your app. See [Webhook Events](https://docs.discord.com/developers/events/webhook-events).
| Value | Name | Description |
| --- | --- | --- |
| 1 | Incoming | Incoming Webhooks use a generated token to send messages into channels |
| 2 | Channel Follower | Channel Follower Webhooks are internal Channel Following webhooks that repost new messages into channels |
| 3 | Application | Application webhooks are used for Interactions |

###### Example Incoming Webhook

```
{
  "name": "test webhook",
  "type": 1,
  "channel_id": "199737254929760256",
  "token": "3d89bb7572e0fb30d8128367b3b1b44fecd1726de135cbe28a41f8b2f777c372ba2939e72279b94526ff5d1bd4358d65cf11",
  "avatar": null,
  "guild_id": "199737254929760256",
  "id": "223704706495545344",
  "application_id": null,
  "user": {
    "username": "test",
    "discriminator": "7479",
    "id": "190320984123768832",
    "avatar": "b004ec1740a63ca06ae2e14c5cee11f3",
    "public_flags": 131328
  }
}

```

###### Example Channel Follower Webhook

```
{
  "type": 2,
  "id": "752831914402115456",
  "name": "Guildy name",
  "avatar": "bb71f469c158984e265093a81b3397fb",
  "channel_id": "561885260615255432",
  "guild_id": "56188498421443265",
  "application_id": null,
  "source_guild": {
    "id": "56188498421476534",
    "name": "Guildy name",
    "icon": "bb71f469c158984e265093a81b3397fb"
  },
  "source_channel": {
    "id": "5618852344134324",
    "name": "announcements"
  },
  "user": {
    "username": "test",
    "discriminator": "7479",
    "id": "190320984123768832",
    "avatar": "b004ec1740a63ca06ae2e14c5cee11f3",
    "public_flags": 131328
  }
}

```

###### Example Application Webhook

```
{
  "type": 3,
  "id": "658822586720976555",
  "name": "Clyde",
  "avatar": "689161dc90ac261d00f1608694ac6bfd",
  "channel_id": null,
  "guild_id": null,
  "application_id": "658822586720976555"
}

```

### Create Webhook

  Creates a webhook and returns a [webhook](https://docs.discord.com/developers/resources/webhook#webhook-object) object if successful. The caller needs `MANAGE_WEBHOOKS`. Discord emits a [Webhooks Update](https://docs.discord.com/developers/events/gateway-events#webhooks-update) Gateway event. Invalid webhook `name` values produce an error. A name is valid when:
- It does not include `clyde` or `discord`, ignoring case
- It follows the [Usernames and Nicknames](https://docs.discord.com/developers/resources/user#usernames-and-nicknames) nickname rules, except webhook names may be up to 80 characters
 This endpoint accepts `X-Audit-Log-Reason`.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name | string | name of the webhook (1-80 characters) |
| avatar? | ?image data | default avatar image for the webhook |

### Get Channel Webhooks

  Returns the channel's [webhook](https://docs.discord.com/developers/resources/webhook#webhook-object) objects. Requires `MANAGE_WEBHOOKS`.

### Get Guild Webhooks

  Returns [webhook](https://docs.discord.com/developers/resources/webhook#webhook-object) objects for the guild. Requires `MANAGE_WEBHOOKS`.

### Get Webhook

  Returns the [webhook](https://docs.discord.com/developers/resources/webhook#webhook-object) object for the supplied id. The request needs `MANAGE_WEBHOOKS` unless the requesting application owns that webhook. [(see: webhook.application_id)](https://docs.discord.com/developers/resources/webhook#webhook-object)

### Get Webhook with Token

  Works like the previous call, but it uses the token, requires no authentication, and omits user from the webhook object.

### Modify Webhook

  Updates a webhook and returns the updated [webhook](https://docs.discord.com/developers/resources/webhook#webhook-object) object on success. Requires `MANAGE_WEBHOOKS`. Discord emits a [Webhooks Update](https://docs.discord.com/developers/events/gateway-events#webhooks-update) Gateway event. Every parameter is optional. This endpoint supports `X-Audit-Log-Reason`.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name | string | webhook default name |
| avatar | ?image data | default avatar image for the webhook |
| channel_id | snowflake | new channel id to move this webhook into |

### Modify Webhook with Token

  Works like the previous update call, but it requires no authentication, does not accept `channel_id` in the body, and omits user from the webhook object.

### Delete Webhook

  Permanently deletes a webhook. Requires `MANAGE_WEBHOOKS`. A successful request returns `204 No Content`. Discord emits a [Webhooks Update](https://docs.discord.com/developers/events/gateway-events#webhooks-update) Gateway event. This endpoint supports `X-Audit-Log-Reason`.

### Delete Webhook with Token

  Works like Delete Webhook, but no authentication is required.

### Execute Webhook

  See [Uploading Files](https://docs.discord.com/developers/reference#uploading-files) for attachment handling and `multipart/form-data` requests. Depending on `wait`, the response is either a message or `204 No Content`. A send request must include at **least one of** `content`, `embeds`, `components`, `file`, or `poll`. For forum or media channels, provide `thread_id` in query string params or `thread_name` in JSON/form params. `thread_id` sends the message into that thread. `thread_name` creates a thread with that name in the channel. Discord may remove invalid unicode characters or characters that unexpectedly affect formatting from message content. Sanitize user-generated message content when needed, and use `allowed_mentions` to avoid unintended mentions.

###### Query String Params

| Field | Type | Description | Required |
| --- | --- | --- | --- |
| wait | boolean | waits for Discord to confirm message creation before responding, and returns the created message body; defaults to `false`; when `false`, an unsaved message does not return an error | false |
| thread_id | snowflake | Send the message to a specific thread inside the webhook's channel; Discord automatically unarchives that thread | false |
| with_components | boolean | whether Discord should honor the request `components` field. When enabled, application-owned webhooks may use all components, and non-owned webhooks may use non-interactive components. Defaults to `false` | false |

###### JSON/Form Params

| Field | Type | Description | Required |
| --- | --- | --- | --- |
| content | string | message content, up to 2000 characters | one of content, file, embeds, poll |
| username | string | default webhook username override | false |
| avatar_url | string | default webhook avatar override | false |
| tts | boolean | true when the message is TTS | false |
| embeds | array of up to 10 embed objects | embedded `rich` content | one of content, file, embeds, poll |
| allowed_mentions | allowed mention object | allowed mentions for the message | false |
| components * | array of message component | components included in the message | false |
| files[n] ** | file contents | file contents being sent | one of content, file, embeds, poll |
| payload_json ** | string | JSON-encoded body for non-file parameters | `multipart/form-data` only |
| attachments ** | array of partial attachment objects | attachment objects with filenames and descriptions | false |
| flags *** | integer | message flags as a combined bitfield; only `SUPPRESS_EMBEDS`, `SUPPRESS_NOTIFICATIONS`, and `IS_COMPONENTS_V2` are allowed | false |
| thread_name | string | thread name to create; requires a webhook channel that is a forum or media channel | false |
| applied_tags | array of snowflakes | array of tag ids to attach to the thread; requires a webhook channel that is a forum or media channel | false |
| poll | poll request object | A poll! | one of content, file, embeds, poll |

 * Application-owned webhooks may always send components. Non-application-owned webhooks cannot send interactive components, and Discord ignores `components` unless `with_components` is set. ** See [Uploading Files](https://docs.discord.com/developers/reference#uploading-files) for details. *** With `IS_COMPONENTS_V2`, a webhook message may contain only `components`; including `content`, `embeds`, `files[n]`, or `poll` causes a 400 BAD REQUEST. For webhook embed objects, all fields are allowed except `type` (always `rich` even if supplied), `provider`, `video`, and image `height`, `width`, or `proxy_url` values.

### Execute Slack-Compatible Webhook

  See [Slack's documentation](https://api.slack.com/incoming-webhooks) for more information. Discord does not support Slack's `channel`, `icon_emoji`, `mrkdwn`, or `mrkdwn_in` properties.

###### Query String Params

| Field | Type | Description | Required |
| --- | --- | --- | --- |
| thread_id | snowflake | thread id where the message should be sent | false |
| wait | boolean | waits for Discord to confirm message send before responding; defaults to `true`; when `false`, an unsaved message does not return an error | false |

### Execute GitHub-Compatible Webhook

  [Add a new webhook](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks) in the GitHub repository settings and set this endpoint as the “Payload URL.” To choose which events reach the Discord channel, select “Let me select individual events” and then pick events for the webhook. Supported [events](https://docs.github.com/en/webhooks/webhook-events-and-payloads) are `commit_comment`, `create`, `delete`, `fork`, `issue_comment`, `issues`, `member`, `public`, `pull_request`, `pull_request_review`, `pull_request_review_comment`, `push`, `release`, `watch`, `check_run`, `check_suite`, `discussion`, and `discussion_comment`.

###### Query String Params

| Field | Type | Description | Required |
| --- | --- | --- | --- |
| thread_id | snowflake | thread id where the message should be sent | false |
| wait | boolean | waits for Discord to confirm message send before responding; defaults to `true`; when `false`, an unsaved message does not return an error | false |

### Get Webhook Message

  Fetches a webhook message previously sent with the same token. Success returns a [message](https://docs.discord.com/developers/resources/message#message-object) object.

###### Query String Params

| Field | Type | Description | Required |
| --- | --- | --- | --- |
| thread_id | snowflake | id of the thread containing the message | false |

### Edit Webhook Message

  Edits a webhook message previously sent with the same token and returns a [message](https://docs.discord.com/developers/resources/message#message-object) object on success. Editing `content` rebuilds `mentions`, `mention_roles`, and `mention_everyone` from the new content. If `IS_COMPONENTS_V2` is set, those fields are rebuilt from edited `components` content. The edit request `allowed_mentions` field controls parsing. Without explicit `allowed_mentions`, Discord parses content using default allowances, regardless of whether the original create request included `allowed_mentions`. See [Uploading Files](https://docs.discord.com/developers/reference#uploading-files) for attachment and `multipart/form-data` details. New files are **appended** to the message. To remove or replace files, send `attachments` with the files that should remain. Starting in API v10, `attachments` must list every attachment that should exist after the edit, including retained attachments and new files in the request body. All endpoint parameters are optional and nullable.

###### Query String Params

| Field | Type | Description | Required |
| --- | --- | --- | --- |
| thread_id | snowflake | id of the thread containing the message | false |
| with_components | boolean | whether Discord should honor the request `components` field. When enabled, application-owned webhooks may use all components, and non-owned webhooks may use non-interactive components. Defaults to `false` | false |

###### JSON/Form Params

| Field | Type | Description |
| --- | --- | --- |
| content | string | message content, up to 2000 characters |
| embeds | array of up to 10 embed objects | embedded `rich` content |
| flags * | integer | message flags as a combined bitfield; only `SUPPRESS_EMBEDS` and `IS_COMPONENTS_V2` are allowed |
| allowed_mentions | allowed mention object | allowed mentions for the message |
| components ** | array of message component | components included in the message |
| files[n] *** | file contents | file contents being sent or edited |
| payload_json *** | string | JSON-encoded body for non-file parameters, only with multipart/form-data |
| attachments *** | array of partial attachment objects | attached files to retain, plus possible descriptions for new files |
| poll **** | poll request object | A poll! |

 * `SUPPRESS_EMBEDS` may be set or unset, while `IS_COMPONENTS_V2` may only be set. With `IS_COMPONENTS_V2`, any used `content`, `embeds`, `files[n]`, or `poll` values in the original message must first be set to `null`; otherwise Discord returns 400 BAD REQUEST. ** Application-owned webhooks may always send components. Non-application-owned webhooks cannot send interactive components, and Discord ignores `components` unless `with_components` is set. *** See [Uploading Files](https://docs.discord.com/developers/reference#uploading-files). **** Polls can be added only while editing a deferred interaction response.

### Delete Webhook Message

  Deletes a webhook-created message. A successful request returns `204 No Content`.

###### Query String Params

| Field | Type | Description | Required |
| --- | --- | --- | --- |
| thread_id | snowflake | id of the thread containing the message | false |
