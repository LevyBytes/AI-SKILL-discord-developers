# Sticker, Subscription

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: Sticker Object; Subscription Object.

## Sticker Object

### Sticker Object

 Describes a sticker that messages can include.

###### Sticker Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | id of the sticker |
| pack_id? | snowflake | for standard stickers, the ID of the source pack |
| name | string | name of the sticker |
| description | ?string | description of the sticker |
| tags* | string | sticker autocomplete/suggestion tags (max 200 characters) |
| type | integer | type of sticker |
| format_type | integer | type of sticker format |
| available? | boolean | whether the guild sticker is currently usable; can be false after Server Boost loss |
| guild_id? | snowflake | ID of the guild that owns the sticker |
| user? | user object | user who uploaded the guild sticker |
| sort_value? | integer | sort position of the standard sticker inside its pack |

 * Standard stickers commonly store a comma-separated keyword list in this field, but that is a convention rather than a strict format. When the client creates or updates a guild sticker, it always fills this field with a name derived from an emoji.

###### Sticker Types

| Type | Value | Description |
| --- | --- | --- |
| STANDARD | 1 | official sticker included in a pack |
| GUILD | 2 | guild-uploaded sticker available to that guild's members |

###### Sticker Format Types

| Type | Value |
| --- | --- |
| PNG | 1 |
| APNG | 2 |
| LOTTIE | 3 |
| GIF | 4 |

###### Example Sticker

```
{
  "id": "749054660769218631",
  "name": "Wave",
  "tags": "wumpus, hello, sup, hi, oi, heyo, heya, yo, greetings, greet, welcome, wave, :wave, :hello, :hi, :hey, hey, \ud83d\udc4b, \ud83d\udc4b\ud83c\udffb, \ud83d\udc4b\ud83c\udffc, \ud83d\udc4b\ud83c\udffd, \ud83d\udc4b\ud83c\udffe, \ud83d\udc4b\ud83c\udfff, goodbye, bye, see ya, later, laterz, cya",
  "type": 1,
  "format_type": 3,
  "description": "Wumpus waves hello",
  "pack_id": "847199849233514549",
  "sort_value": 12
}

```

### Sticker Item Object

 The minimal sticker data needed for rendering; effectively a partial sticker object.

###### Sticker Item Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | id of the sticker |
| name | string | name of the sticker |
| format_type | integer | type of sticker format |

### Sticker Pack Object

 Describes a standard sticker pack.

###### Sticker Pack Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | id of the sticker pack |
| stickers | array of sticker objects | the stickers in the pack |
| name | string | name of the sticker pack |
| sku_id | snowflake | ID of the pack SKU |
| cover_sticker_id? | snowflake | ID of the pack sticker used as the pack icon |
| description | string | description of the sticker pack |
| banner_asset_id? | snowflake | ID of the sticker pack banner image |

###### Example Sticker Pack

```
{
  "id": "847199849233514549",
  "stickers": [],
  "name": "Wumpus Beyond",
  "sku_id": "847199849233514547",
  "cover_sticker_id": "749053689419006003",
  "description": "Say hello to Wumpus!",
  "banner_asset_id": "761773777976819732"
}

```

### Get Sticker

  Fetches the [sticker](https://docs.discord.com/developers/resources/sticker#sticker-object) object for the specified sticker ID.

### List Sticker Packs

  Fetches the available sticker packs.

###### Response Structure

| Field | Type |
| --- | --- |
| sticker_packs | array of sticker pack objects |

### Get Sticker Pack

  Fetches the [sticker pack](https://docs.discord.com/developers/resources/sticker#sticker-pack-object) object for the specified sticker pack ID.

### List Guild Stickers

  Fetches an array of [sticker](https://docs.discord.com/developers/resources/sticker#sticker-object) objects for the specified guild. The response includes `user` fields when the bot has `CREATE_GUILD_EXPRESSIONS` or `MANAGE_GUILD_EXPRESSIONS`.

### Get Guild Sticker

  Fetches the [sticker](https://docs.discord.com/developers/resources/sticker#sticker-object) object for the specified guild ID and sticker ID. The response includes `user` when the bot has `CREATE_GUILD_EXPRESSIONS` or `MANAGE_GUILD_EXPRESSIONS`.

### Create Guild Sticker

  Creates a guild sticker. Use a `multipart/form-data` request and the `CREATE_GUILD_EXPRESSIONS` permission. On success, the endpoint returns the created [sticker](https://docs.discord.com/developers/resources/sticker#sticker-object) object and emits a [Guild Stickers Update](https://docs.discord.com/developers/events/gateway-events#guild-stickers-update) Gateway event. Guilds start with five free sticker slots, and Boost levels add more. The endpoint accepts the `X-Audit-Log-Reason` header. Lottie uploads are allowed only in guilds with the `VERIFIED` and/or `PARTNERED` [guild feature](https://docs.discord.com/developers/resources/guild#guild-object-guild-features). Uploaded animated stickers may be no longer than 5 seconds and must fit 320 x 320 pixels.

###### Form Params

| Field | Type | Description |
| --- | --- | --- |
| name | string | name of the sticker (2-30 characters) |
| description | string | sticker description, either empty or 2-100 characters |
| tags | string | sticker autocomplete/suggestion tags (max 200 characters) |
| file | file contents | file upload for the sticker; must be PNG, APNG, GIF, or Lottie JSON, max 512 KiB |

### Modify Guild Sticker

  Updates the specified sticker. If the current user created the sticker, either `CREATE_GUILD_EXPRESSIONS` or `MANAGE_GUILD_EXPRESSIONS` is required. For stickers created by someone else, `MANAGE_GUILD_EXPRESSIONS` is required. A successful call returns the updated [sticker](https://docs.discord.com/developers/resources/sticker#sticker-object) object and emits a [Guild Stickers Update](https://docs.discord.com/developers/events/gateway-events#guild-stickers-update) Gateway event. Every parameter is optional. The endpoint accepts the `X-Audit-Log-Reason` header.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name | string | name of the sticker (2-30 characters) |
| description | ?string | description of the sticker (2-100 characters) |
| tags | string | sticker autocomplete/suggestion tags (max 200 characters) |

### Delete Guild Sticker

  Deletes the specified sticker. If the current user created it, either `CREATE_GUILD_EXPRESSIONS` or `MANAGE_GUILD_EXPRESSIONS` is required. For stickers created by another user, `MANAGE_GUILD_EXPRESSIONS` is required. Success returns `204 No Content` and emits a [Guild Stickers Update](https://docs.discord.com/developers/events/gateway-events#guild-stickers-update) Gateway event. The endpoint accepts the `X-Audit-Log-Reason` header.

## Subscription Object

A Discord subscription represents recurring payments by a user for one or more SKUs over time. Successful payment gives the user access to the entitlements attached to the SKU.

### Subscription Object

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | ID of the subscription |
| user_id | snowflake | ID of the subscribed user |
| sku_ids | array of snowflakes | List of SKUs subscribed to |
| entitlement_ids | array of snowflakes | entitlements provided by this subscription |
| renewal_sku_ids | ?array of snowflakes | SKUs the user will remain subscribed to on renewal |
| current_period_start | ISO8601 timestamp | start timestamp for the active subscription period |
| current_period_end | ISO8601 timestamp | end timestamp for the active subscription period |
| status | SubscriptionStatus | Current status of the subscription |
| canceled_at | ?ISO8601 timestamp | When the subscription was canceled |
| country? | string | ISO3166-1 alpha-2 country code for the payment source used for purchase; absent unless a private OAuth scope was used for the query |

 The subscription ID determines when the subscription began. Renewal updates the current period. If the user cancels, the subscription moves to `ENDING`, and `canceled_at` records the cancellation time.

### Subscription Example

```
{
  "id": "1278078770116427839",
  "user_id": "1088605110638227537",
  "sku_ids": ["1158857122189168803"],
  "entitlement_ids": [],
  "renewal_sku_ids": null,
  "current_period_start": "2024-08-27T19:48:44.406602+00:00",
  "current_period_end": "2024-09-27T19:48:44.406602+00:00",
  "status": 0,
  "canceled_at": null
}

```

### Subscription Statuses

| Type | Value | Description |
| --- | --- | --- |
| ACTIVE | 0 | Subscription is active and will renew. |
| INACTIVE | 1 | Subscription is inactive and charges are not occurring. |
| ENDING | 2 | Subscription remains active but is not set to renew. |

 Do not use subscription status itself to decide perk access. Use [entitlements](https://docs.discord.com/developers/resources/entitlement#entitlement-object) to determine whether a user should access a given SKU. For more context, see [Implementing App Subscriptions](https://docs.discord.com/developers/monetization/implementing-app-subscriptions). During the current period, subscriptions can begin in or move among any of these statuses. A subscription may be `ACTIVE` outside its current period or `INACTIVE` during it. Examples include:
- During retry for a failed payment, the subscription can stay `ACTIVE` until payment succeeds or the system marks it unrecoverable.
- A refund or chargeback within the current period can make the subscription `INACTIVE`.

### List SKU Subscriptions

  Fetches subscriptions for the SKU, optionally filtered by user. The response is a list of [subscription](https://docs.discord.com/developers/resources/subscription#subscription-object) objects.

### Query String Params

| Field | Type | Description | Default |
| --- | --- | --- | --- |
| before? | snowflake | List subscriptions before this ID | absent |
| after? | snowflake | List subscriptions after this ID | absent |
| limit? | integer | Number of results to return (1-100) | 50 |
| user_id? | snowflake | User ID whose subscriptions should be returned. Required unless the query is OAuth-based. | absent |

### Get SKU Subscription

  Fetches one subscription by ID and returns a [subscription](https://docs.discord.com/developers/resources/subscription#subscription-object) object.
