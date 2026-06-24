# Emoji, Entitlement

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: Emoji Object; Entitlement Object.

## Emoji Object

Emoji management routes use different rate limit behavior from the standard convention. They are constrained per guild to reduce abuse, so the limits reported by the API can be wrong and 429 responses may still occur.

### Emoji Object

###### Emoji Structure

| Field | Type | Description |
| --- | --- | --- |
| id | ?snowflake | emoji id |
| name | ?string (null is allowed only for reaction emoji objects). | emoji name |
| roles? | array of role object ids | roles that may use this emoji. |
| user? | user object | user that created this emoji |
| require_colons? | boolean | whether this emoji requires surrounding colons. |
| managed? | boolean | whether this emoji is managed |
| animated? | boolean | whether this emoji is animated |
| available? | boolean | whether this emoji is usable; this can be false after Server Boosts are lost. |

###### Premium Emoji

 Roles whose `integration_id` tag is the guild's guild_subscription integration count as subscription roles. One emoji cannot mix subscription roles with non-subscription roles. Emoji assigned to subscription roles are premium emoji and use a separate limit of 25. Once created, an emoji cannot be changed between normal and premium.

###### Emoji Formats

 Emoji uploads may use JPEG, PNG, GIF, WebP, or AVIF. Regardless of upload format, emoji can be served as WebP. Discord strongly recommends fetching emoji with the `.webp` extension for best performance and compatibility, matching the Discord client, which displays all in-app emoji as WebP. Still WebP emoji use the `.webp` extension. Animated WebP emoji also use `.webp` with the `?animated=true` query parameter.

###### Application-Owned Emoji

 An application may own as many as 2000 emojis that are usable only by that app. App emojis can be administered through the API with a bot token or through the application's portal settings. Using app emojis does not require `USE_EXTERNAL_EMOJIS`. For an app emoji object, the `user` field is the team member who uploaded it through settings, or the bot user when it was uploaded through the API.

###### Emoji Example

```
{
  "id": "41771983429993937",
  "name": "LUL",
  "roles": ["41771983429993000", "41771983429993111"],
  "user": {
    "username": "Luigi",
    "discriminator": "0002",
    "id": "96008815106887111",
    "avatar": "5500909a3274e1812beb4e8de6631111",
    "public_flags": 131328
  },
  "require_colons": true,
  "managed": false,
  "animated": false
}

```

###### Standard Emoji Example

```
{
  "id": null,
  "name": "🔥"
}

```

###### Custom Emoji Examples

 `MESSAGE_REACTION_ADD`, `MESSAGE_REACTION_REMOVE`, and `MESSAGE_REACTION_REMOVE_EMOJI` gateway events include `animated` for animated emoji. In `MESSAGE_REACTION_ADD` and `MESSAGE_REACTION_REMOVE`, `name` can be `null` when custom emoji data cannot be retrieved, such as after the emoji was deleted from the guild.

```
{
  "id": "41771983429993937",
  "name": "LUL",
  "animated": true
}

```

```
{
  "id": "41771983429993937",
  "name": null
}

```

### List Guild Emojis

  Returns [emoji](https://docs.discord.com/developers/resources/emoji#emoji-object) objects for the specified guild. The response includes `user` fields when the bot has `CREATE_GUILD_EXPRESSIONS` or `MANAGE_GUILD_EXPRESSIONS`.

### Get Guild Emoji

  Returns the [emoji](https://docs.discord.com/developers/resources/emoji#emoji-object) object for the supplied guild and emoji IDs. The `user` field is included when the bot has `MANAGE_GUILD_EXPRESSIONS`, or when the bot created the emoji and has `CREATE_GUILD_EXPRESSIONS`.

### Create Guild Emoji

  Creates a guild emoji. This requires `CREATE_GUILD_EXPRESSIONS` and returns the created [emoji](https://docs.discord.com/developers/resources/emoji#emoji-object) object when successful. It also emits a [Guild Emojis Update](https://docs.discord.com/developers/events/gateway-events#guild-emojis-update) Gateway event. Emoji and animated emoji uploads are limited to 256 KiB. Larger uploads fail with 400 Bad Request and an error message, but without a [JSON status code](https://docs.discord.com/developers/topics/opcodes-and-status-codes#json). Discord strongly recommends fetching emoji with `.webp` for performance and compatibility; see [Emoji Formats](https://docs.discord.com/developers/resources/emoji#emoji-object-emoji-formats). This endpoint accepts the `X-Audit-Log-Reason` header.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name | string | name of the emoji |
| image | image data | the 128x128 emoji image |
| roles | array of snowflakes | roles that may use this emoji. |

### Modify Guild Emoji

  Updates the specified emoji. If the current user created the emoji, either `CREATE_GUILD_EXPRESSIONS` or `MANAGE_GUILD_EXPRESSIONS` is required. For emojis created by someone else, `MANAGE_GUILD_EXPRESSIONS` is required. A successful request returns the updated [emoji](https://docs.discord.com/developers/resources/emoji#emoji-object) object and emits a [Guild Emojis Update](https://docs.discord.com/developers/events/gateway-events#guild-emojis-update) Gateway event. Every parameter is optional. This endpoint accepts the `X-Audit-Log-Reason` header.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name | string | name of the emoji |
| roles | ?array of snowflakes | roles that may use this emoji. |

### Delete Guild Emoji

  Deletes the specified emoji. For emojis created by the current user, either `CREATE_GUILD_EXPRESSIONS` or `MANAGE_GUILD_EXPRESSIONS` is required; other emojis require `MANAGE_GUILD_EXPRESSIONS`. Success returns `204 No Content` and emits a [Guild Emojis Update](https://docs.discord.com/developers/events/gateway-events#guild-emojis-update) Gateway event. This endpoint accepts the `X-Audit-Log-Reason` header.

### List Application Emojis

  Returns an object whose `items` key contains [emoji](https://docs.discord.com/developers/resources/emoji#emoji-object) objects for the application. Each item includes a `user` object for the team member who uploaded it through application settings or the bot user that uploaded it through the API.

```
{
  "items": [
    {
      "id": "41771983429993937",
      "name": "LUL",
      "roles": [],
      "user": {
        "username": "Luigi",
        "discriminator": "0002",
        "id": "96008815106887111",
        "avatar": "5500909a3274e1812beb4e8de6631111",
        "public_flags": 131328
      },
      "require_colons": true,
      "managed": false,
      "animated": false
    }
  ]
}

```

### Get Application Emoji

  Returns the [emoji](https://docs.discord.com/developers/resources/emoji#emoji-object) object for the given application and emoji IDs, including `user`.

### Create Application Emoji

  Creates an emoji for the application and returns the new [emoji](https://docs.discord.com/developers/resources/emoji#emoji-object) object on success. Emoji and animated emoji files are limited to 256 KiB. Uploads above the limit fail with 400 Bad Request and an error message, but no [JSON status code](https://docs.discord.com/developers/topics/opcodes-and-status-codes#json). Discord strongly recommends `.webp` when fetching emoji for performance and compatibility. See [Emoji Formats](https://docs.discord.com/developers/resources/emoji#emoji-object-emoji-formats) for details.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name | string | name of the emoji |
| image | image data | the 128x128 emoji image |

### Modify Application Emoji

  Updates the specified emoji and returns the updated [emoji](https://docs.discord.com/developers/resources/emoji#emoji-object) object when successful.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name | string | name of the emoji |

### Delete Application Emoji

  Deletes the specified emoji. Success returns `204 No Content`.

## Entitlement Object

In Discord, an entitlement records that a user or guild can access a premium offering for an application. See the [Monetization Overview](https://docs.discord.com/developers/monetization/overview) for guidance on using Entitlements in an app.

### Entitlement Object

###### Entitlement Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | ID of the entitlement |
| sku_id | snowflake | ID of the SKU |
| application_id | snowflake | ID of the parent application |
| user_id? | snowflake | ID of the user who receives access to the entitlement's sku. |
| type | integer | Type of entitlement |
| deleted | boolean | Entitlement was deleted |
| starts_at | ?ISO8601 timestamp | Date and time when the entitlement becomes valid. |
| ends_at | ?ISO8601 timestamp | Date and time when the entitlement stops being valid. |
| guild_id? | snowflake | ID of the guild that receives access to the entitlement's sku. |
| consumed? | boolean | For consumable items, indicates whether the entitlement is already consumed. |

###### Entitlement Example

```
{
  "id": "1019653849998299136",
  "sku_id": "1019475255913222144",
  "application_id": "1019370614521200640",
  "user_id": "771129655544643584",
  "promotion_id": null,
  "type": 8,
  "deleted": false,
  "gift_code_flags": 0,
  "consumed": false,
  "starts_at": "2022-09-14T17:00:18.704163+00:00",
  "ends_at": "2022-10-14T17:00:18.704163+00:00",
  "guild_id": "1015034326372454400",
  "subscription_id": "1019653835926409216"
}

```

###### Entitlement Types

| Type | Value | Description |
| --- | --- | --- |
| PURCHASE | 1 | Entitlement was purchased by user |
| PREMIUM_SUBSCRIPTION | 2 | Entitlement for Discord Nitro subscription |
| DEVELOPER_GIFT | 3 | Entitlement was gifted by developer |
| TEST_MODE_PURCHASE | 4 | Entitlement bought by a developer while the application is in test mode. |
| FREE_PURCHASE | 5 | Entitlement issued because the SKU was free. |
| USER_GIFT | 6 | Entitlement given as a gift by another user. |
| PREMIUM_PURCHASE | 7 | Entitlement claimed at no charge by a Nitro Subscriber. |
| APPLICATION_SUBSCRIPTION | 8 | Entitlement bought as an app subscription. |

### List Entitlements

 GET/applications/[{application.id}](https://docs.discord.com/developers/resources/application#application-object)/entitlements Returns every entitlement for the specified app, including active and expired entitlements.

###### Query String Params

| param | type | description |
| --- | --- | --- |
| user_id? | snowflake | User ID used to filter entitlements. |
| sku_ids? | comma-delimited set of snowflakes | Optional SKU ID list used when checking entitlements. |
| before? | snowflake | Return entitlements earlier than this entitlement ID. |
| after? | snowflake | Return entitlements later than this entitlement ID. |
| limit? | integer | Number of entitlements to return, from 1-100; default 100. |
| guild_id? | snowflake | Guild ID used to filter entitlements. |
| exclude_ended? | boolean | Whether ended entitlements are excluded. Defaults to false, so ended entitlements are returned unless omitted. |
| exclude_deleted? | boolean | Whether deleted entitlements are excluded. Defaults to true, so deleted entitlements are omitted unless requested. |

```
[
  {
    "id": "1019653849998299136",
    "sku_id": "1019475255913222144",
    "application_id": "1019370614521200640",
    "user_id": "771129655544643584",
    "promotion_id": null,
    "type": 8,
    "deleted": false,
    "gift_code_flags": 0,
    "consumed": false,
    "starts_at": "2022-09-14T17:00:18.704163+00:00",
    "ends_at": "2022-10-14T17:00:18.704163+00:00",
    "guild_id": "1015034326372454400",
    "subscription_id": "1019653835926409216"
  }
]

```

### Get Entitlement

 GET/applications/[{application.id}](https://docs.discord.com/developers/resources/application#application-object)/entitlements/[{entitlement.id}](https://docs.discord.com/developers/resources/entitlement#entitlement-object) Returns a single entitlement.

```
{
  "id": "1019653849998299136",
  "sku_id": "1019475255913222144",
  "application_id": "1019370614521200640",
  "user_id": "771129655544643584",
  "promotion_id": null,
  "type": 8,
  "deleted": false,
  "gift_code_flags": 0,
  "consumed": false,
  "starts_at": "2022-09-14T17:00:18.704163+00:00",
  "ends_at": "2022-10-14T17:00:18.704163+00:00",
  "guild_id": "1015034326372454400",
  "subscription_id": "1019653835926409216"
}

```

### Consume an Entitlement

 POST/applications/[{application.id}](https://docs.discord.com/developers/resources/application#application-object)/entitlements/[{entitlement.id}](https://docs.discord.com/developers/resources/entitlement#entitlement-object)/consume For One-Time Purchase consumable SKUs, marks the user's entitlement as consumed. Afterward, [List Entitlements](https://docs.discord.com/developers/resources/entitlement#list-entitlements) shows `consumed: true` for that entitlement. Success returns `204 No Content`.

### Create Test Entitlement

 POST/applications/[{application.id}](https://docs.discord.com/developers/resources/application#application-object)/entitlements Creates a test entitlement for a given SKU and guild or user. Discord treats that user or guild as entitled to the premium offering. The response is a partial entitlement object and does **not** include `subscription_id`, `starts_at`, or `ends_at`, because the test entitlement remains valid indefinitely. Reload the Discord client after creation; the server or user will then show premium access.

###### JSON Params

| param | type | description |
| --- | --- | --- |
| sku_id | string | ID of the SKU that receives the entitlement grant. |
| owner_id | string | ID of the guild or user receiving the entitlement. |
| owner_type | integer | `1` for a guild subscription, `2` for a user subscription. |

```
{
  "sku_id": "999184799365857331",
  "owner_id": "847184799365857999",
  "owner_type": 1
}

```

### Delete Test Entitlement

 DELETE/applications/[{application.id}](https://docs.discord.com/developers/resources/application#application-object)/entitlements/[{entitlement.id}](https://docs.discord.com/developers/resources/entitlement#entitlement-object) Deletes an active test entitlement. Discord then treats that user or guild as *no longer having* entitlement to the premium offering. Success returns `204 No Content`.
