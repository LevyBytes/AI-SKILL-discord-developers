# Change Log (part 6)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

[Activities](https://docs.discord.com/developers/activities/overview) can now be launched as an interaction response through the `LAUNCH_ACTIVITY` (type `12`) [interaction callback type](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-response-object-interaction-callback-type). `LAUNCH_ACTIVITY` is valid in responses to `APPLICATION_COMMAND`, `MESSAGE_COMPONENT`, and `MODAL_SUBMIT` [interaction types](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-type). August 26, 2024 Activities Interactions

### Entry Point Commands

Apps with [Activities](https://docs.discord.com/developers/activities/overview) enabled can create [Entry Point commands](https://docs.discord.com/developers/interactions/application-commands#entry-point-commands) using the `PRIMARY_ENTRY_POINT` (type `4`) [command type](https://docs.discord.com/developers/interactions/application-commands#application-command-object-application-command-types). Each app can have one globally-scoped Entry Point command, shown in the App Launcher. When creating or updating an Entry Point command, set an [Entry Point handler](https://docs.discord.com/developers/interactions/application-commands#application-command-object-entry-point-command-handler-types) with the [`handler` field](https://docs.discord.com/developers/interactions/application-commands#application-command-object-application-command-structure). The `handler` field controls whether your app handles the interaction response itself:
- `DISCORD_LAUNCH_ACTIVITY` (`2`) lets Discord handle the interaction itself and post the follow-up message in the channel that used the Entry Point command.
- `APP_HANDLER` (`1`) sends your app an interaction token and makes your app responsible for responding. You can then launch the Activity with the `LAUNCH_ACTIVITY` (type `12`) [interaction callback](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-response-object-interaction-callback-type).
Read the [Application Commands documentation](https://docs.discord.com/developers/interactions/application-commands#entry-point-commands) and the [Activity development guide](https://docs.discord.com/developers/activities/development-guides/user-actions#setting-up-an-entry-point-command) for more about Entry Point commands.

### Default Entry Point Commands

When Activities are enabled in an [app's settings](http://discord.com/developers/applications), Discord now creates a default [Entry Point command](https://docs.discord.com/developers/interactions/application-commands#default-entry-point-command) named "Launch" for the app. You can customize or delete it like any other command when changing the name or handler type. August 13, 2024 Voice

### Voice Encryption Modes

Documentation now covers voice [encryption modes](https://docs.discord.com/developers/topics/voice-connections#transport-encryption-modes) `aead_aes256_gcm_rtpsize` and `aead_xchacha20_poly1305_rtpsize`, and announces deprecation for all `xsalsa20_poly1305*` variants plus `aead_aes256_gcm`. Deprecated encryption modes are scheduled for discontinuation on November 18th, 2024. August 13, 2024 Gateway Voice

### Voice Gateway Version 8 and Deprecation of Versions < 4

Discord is deprecating very old voice gateway versions (> 7 years ago).
- Voice gateway version 8 adds buffered resume support, which re-sends lost messages. Use voice gateway version 8 and see [Buffered Resume](https://docs.discord.com/developers/topics/voice-connections#buffered-resume).
- The default voice gateway version option has been removed. After deprecation, callers must pass a voice gateway version.
Callers will need to pass a voice gateway version, and deprecated voice gateway versions are scheduled for discontinuation on November 18th, 2024. See [Voice Gateway Versioning](https://docs.discord.com/developers/topics/voice-connections#voice-gateway-versioning) for details. August 12, 2024 HTTP API

### Get Guild Role Endpoint

To fetch a single role instead of the full role list, use the new [Get Guild Role](https://docs.discord.com/developers/resources/guild#get-guild-role) endpoint with a role ID. August 09, 2024 User Apps

### User App Install Count

The [Application object](https://docs.discord.com/developers/resources/application#application-object) now includes an approximate user install count for user-installable apps. Install counts are visible in the developer portal too. August 05, 2024 Voice HTTP API

### Voice State Endpoints

Voice states are now available through the HTTP API. Apps can call [Get Current User Voice State](https://docs.discord.com/developers/resources/voice#get-current-user-voice-state) and [Get User Voice State](https://docs.discord.com/developers/resources/voice#get-user-voice-state) to fetch a user's voice state without using a Gateway connection. July 25, 2024

### Supported Activity Types for SET_ACTIVITY

The [`SET_ACTIVITY` RPC command](https://docs.discord.com/developers/topics/rpc#setactivity) now supports 3 additional [activity types](https://docs.discord.com/developers/events/gateway-events#activity-object-activity-types): Listening (`2`), Watching (`3`), and Competing (`5`). Previously it accepted only Playing (`0`). The [Game SDK](https://docs.discord.com/developers/developer-tools/game-sdk#activities) still has not been updated to support setting [`ActivityType`](https://docs.discord.com/developers/developer-tools/game-sdk#activitytype-enum), and remains read-only for handling events received from Discord. July 18, 2024

### Application Emoji

Applications can upload emojis in [app's settings](https://discord.com/developers/applications) and use them as custom emojis across Discord.
- Up to 2000 emojis per app
- Support for user-installable apps
- Application emojis can be [managed via the API](https://docs.discord.com/developers/resources/emoji#create-application-emoji) with a bot token
 July 17, 2024ActivitiesEmbedded App SDKBreaking Changes

### Activities Proxy CSP Update

Outdated notice: Discord has since updated the Activities Proxy CSP, and `/.proxy/` is no longer required. See [this changelog](https://docs.discord.com/developers/change-log#remove-proxy-from-discord-activity-proxy-path) for current guidance. This change was scheduled to roll out to all existing applications on **August 28, 2024**. Discord planned an updated Content Security Policy (CSP) for the Activities Domain (`https://<application_id>.discordsays.com`). Because this was a **breaking change** for **all Activities**, a migration plan was provided. The CSP update was described as follows:
- all requests must go through `https://<application_id>.discordsays.com/.proxy/`; requests to other paths on the `discordsays.com` domain would be blocked.
- requests to `https://discord.com/api/` would be allowed, while other paths on the `discord.com` domain would be blocked.
- Only allowed paths on `cdn.discordapp.com` and `media.discordapp.net` would be permitted, including `/attachments/`, `/icons/`, and `/avatars/`.
- nested child iframes must mount paths with the `/.proxy/` prefix
As of [embedded-app-sdk v1.4.0](https://github.com/discord/embedded-app-sdk/releases/tag/v1.4.0) `patchUrlMappings` automatically routes requests through `/.proxy/`, so updating the SDK and calling `patchUrlMappings` is a good first step. If `patchUrlMappings` is unfamiliar, see the [documentation](https://docs.discord.com/developers/activities/development-guides/networking#using-external-resources). All Application IDs created after `07/17/2024 12:00:00` UTC (applicationID greater than `1263102905548800000`) automatically received the new CSP. Testing production code on a new application created after that date was the suggested compliance check. July 16, 2024

### Guild Member Banners

Apps can access guild member profile banners through the API and Gateway. The [guild member object](https://docs.discord.com/developers/resources/guild#guild-member-object) now includes a `banner` field that can be used to build the [guild member banner URL](https://docs.discord.com/developers/reference#image-formatting). July 15, 2024

### Message Forwarding rollout

Message forwarding is rolling out gradually. When callers create a message with `message_reference.type = FORWARD`, the API can attach a `message_snapshot` to that new message. The feature has [some limitations](https://docs.discord.com/developers/resources/message#message-reference-types); the snapshot is a compact version of a standard `MessageObject` that still preserves the key message content. The resulting message looks like:

```
{
  "id": "1255957733279273083",
  "message_reference": {
    "type": 1, // Forward
    ...
  }
  "message_snapshots": [
    {
      "message": {
        "content": "original message",
        "embeds": [...],
        "attachments": [...],
        ...
      }
    }
  ],
  ...
}

```

Stricter rate limits were applied to this feature based on:
- number of forwards sent by the user
- total attachment size

###### API Updates since preview

This was [previously announced](https://discord.com/channels/613425648685547541/697138785317814292/1233463756160503859) but the final API differs from the first preview in several ways:
- [`message snapshot`](https://docs.discord.com/developers/resources/message#message-snapshot-object) objects no longer include a `guild` field because `message_reference` already carries that information
- forwarded messages now use the distinct `message_reference` type `FORWARD`
 July 09, 2024

### Banners in Get Current User Guilds

[`GET /users/@me/guilds`](https://docs.discord.com/developers/resources/user#get-current-user-guilds) now returns each guild's `banner` field, letting OAuth2 apps with the `guilds` scope display guild banners. June 27, 2024 Breaking Changes

### User-Installed Apps General Availability

The [user-installed apps beta](https://docs.discord.com/developers/change-log#userinstallable-apps-preview) announced in March has reached general availability after developer and modmin feedback. User-installed apps can now be used in all servers regardless of size. This update includes API and behavior changes for user-installed apps.

###### API Updates

- `user_id` was removed from the `interaction_metadata` field on messages. Use the `id` field in the nested `user` object instead. See the [Message Interaction Metadata Object](https://docs.discord.com/developers/resources/message#message-interaction-metadata-object) for details.
- User-installed apps can create at most 5 [follow-ups](https://docs.discord.com/developers/interactions/receiving-and-responding#followup-messages) while responding to interactions. This affects only the [Create Followup Message endpoint](https://docs.discord.com/developers/interactions/receiving-and-responding#create-followup-message), not apps installed to the server.
- On [Interactions](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure), `authorizing_integration_owners` is now serialized correctly as a string. The `"0"` value used to be serialized incorrectly as a number.
- `app_permissions` on [Interactions](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure) now correctly reflects permissions for user-installed apps. The previous value was wrong for user-installed apps.
- Updating a message can return `400` when AutoMod blocks the message content, which matters especially for [deferred messages](https://docs.discord.com/developers/interactions/receiving-and-responding#responding-to-an-interaction).
- Interaction responses are no longer forced ephemeral in servers with over 25 members.

###### New `Use External Apps` Permission

Discord added the [`USE_EXTERNAL_APPS` (`1 << 50`) permission](https://docs.discord.com/developers/topics/permissions#permissions-bitwise-permission-flags) and enables it for servers by default. This permission lets modmins control whether user-installed apps may publish non-ephemeral replies in a server. When `Use External Apps` is disabled and your app is *not* installed to that server, your app's responses appear as ephemeral messages to the end user. Read more in the [Moderating Apps on Discord Help Center article](https://support.discord.com/hc/en-us/articles/23957313048343-Moderating-Apps-on-Discord#h_01HZQQQEADYVN2CM4AX4EZGKHM).

###### Updated Defaults for New Apps

- Newly-created apps now default to both "User Install" *and* "Guild Install" [installation contexts](https://docs.discord.com/developers/resources/application#installation-context) enabled. Change this in the **Installation** tab in an [app's settings](https://discord.com/developers/applications).
- Newly-created apps now default to the "Discord Provided Link" [install link](https://docs.discord.com/developers/resources/application#install-links). Change this in the **Installation** tab in an [app's settings](https://discord.com/developers/applications).
- When Discord Provided Link is selected as the install link type, the `application.commands` scope is added to both installation contexts.
 June 17, 2024

### Premium Apps: New Premium Button Style & Deep Linking URL Schemes

**New Premium Button Style** adds a new `premium` [button style](https://docs.discord.com/developers/components/reference#button-button-styles) for use with a `sku_id` that points to an active [SKU](https://docs.discord.com/developers/resources/sku#sku-object). Developers can return targeted subscription products or one-time purchases to shape premium flows. See [button components with interactions](https://docs.discord.com/developers/components/reference#button) for implementation details. Interaction Response Type 10 is deprecated: `PREMIUM_REQUIRED (10)` now gives way to custom premium buttons. It still works for now, but may lose support later, so bots should move to the [premium button component](https://docs.discord.com/developers/components/reference#button-button-styles). See [gating features with premium interactions](https://docs.discord.com/developers/monetization/implementing-app-subscriptions#prompting-users-to-subscribe) for subscription gating patterns. **Deep Linking URL Schemes for SKUs and Store** adds two direct Application Directory url schemes. When posted in chat, these links render as rich embeds that let users open an app store page or a SKU detail modal.
- New [Store URL Scheme](https://docs.discord.com/developers/monetization/managing-skus#linking-to-your-store): `https://discord.com/application-directory/:appID/store`
- New [SKU URL Scheme](https://docs.discord.com/developers/monetization/managing-skus#linking-to-a-specific-sku): `https://discord.com/application-directory/:appID/store/:skuID`
 May 31, 2024

### Auto Moderation Member Profile Rule

- Added Auto Moderation `MEMBER_PROFILE` rule [trigger_type](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-trigger-types). This rule checks whether a member's profile contains disallowed keywords.
- Added Auto Moderation `BLOCK_MEMBER_INTERACTION` [action type](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-action-object-action-types) for the `MEMBER_PROFILE` rule [trigger_type](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-trigger-types). The action partially "quarantines" the member and blocks most interactions in a specific server.
 April 24, 2024

### Premium Apps: One-Time Purchases and Store

Premium Apps gained two new features: One-Time Purchases and Stores. **One-Time Purchases**
- **Durable Items**: A one-time purchase that is permanent and not consumed or renewed, such as lifetime access to an app's premium features.
- **Consumable Items**: A one-time, non-renewable purchase that grants access, such as a temporary power-up or game boost.
See [Implementing One-Time Purchases](https://docs.discord.com/developers/monetization/implementing-one-time-purchases) for purchase implementation guidance. **A Store for Your Premium App** adds a Store where your Premium App can showcase app subscriptions and one-time purchase items. You can create a unique Store page in the developer portal and add published subscription SKUs or one-time purchase SKUs to the store view, allowing users to buy them from your App Directory or Bot User Profile. For eligibility, setup, and monetization details, see the [Monetization Overview](https://docs.discord.com/developers/monetization/overview). **API Documentation Updates** added these items to public Monetization documentation:
- New [SKU Object Types](https://docs.discord.com/developers/resources/sku#sku-object-sku-types)
- New [Entitlement Object Types](https://docs.discord.com/developers/resources/entitlement#entitlement-object-entitlement-types)
- [Consume an Entitlement](https://docs.discord.com/developers/resources/entitlement#consume-an-entitlement) API endpoint
- `consumed` field on the [Entitlement](https://docs.discord.com/developers/resources/entitlement) resource
 April 23, 2024

### Modify Guild Member flags field permissions

Permissions required to modify the `flags` field on the [Modify Guild Member](https://docs.discord.com/developers/resources/guild#modify-guild-member) endpoint were updated. April 02, 2024

### CSV Export for Premium App Analytics

Apps with [Monetization](https://docs.discord.com/developers/monetization/overview) enabled can export SKU analytics to CSV. The export supports reporting on premium offerings in external data tools. Find it at the bottom of the `Monetization → Analytics` tab for the app; it exports monthly data points such as `sales_count`, `sales_amount`, `sales_currencies`, `cancellation_count`, `refund_amount`, and `refund_count`, aggregated by offering. March 18, 2024 Embedded App SDK Activities

### Discord Activities: Developer Preview of the Embedded App SDK

Discord Developers can now build Activities. Activities are interactive multiplayer experiences running in an iframe inside Discord. The Embedded App SDK was introduced to support communication between the app and Discord client.
- New [Discord Activities](https://docs.discord.com/developers/activities/overview) developer docs include a tutorial, code samples, development guides, and design principles.
- The Embedded App SDK is available through [npm](https://npmjs.com/package/@discord/embedded-app-sdk) and [GitHub](http://github.com/discord/embedded-app-sdk).
- The [Embedded App SDK Reference](https://docs.discord.com/developers/developer-tools/embedded-app-sdk) is available.
To get started building an Activity, see the [Activities Overview](https://docs.discord.com/developers/activities/overview). March 18, 2024 User Apps

### User-Installable Apps Preview

Apps can now install to individual user accounts, which makes them easier to install, find, and use throughout Discord. A user-installed app works in all servers the user belongs to, in the user's (G)DMs, and in DMs with the app's bot user. During app creation or updates, set the supported installation types from the **Installation** page in [app's settings](https://discord.com/developers/applications). To start quickly, follow the new [Developing a User-Installable App tutorial](https://docs.discord.com/developers/tutorials/developing-a-user-installable-app) or read the details below. This change adds new API concepts and fields that apps will encounter.

###### API Changes
