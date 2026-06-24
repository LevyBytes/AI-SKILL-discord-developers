# Change Log (part 7)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

**Concepts:**
- [Installation context](https://docs.discord.com/developers/resources/application#installation-context) records whether an app was installed to a user, a guild (server), or both. Apps currently default to the guild installation context only, though the default can change later.
- Commands may support one or both installation contexts. At creation time, a command defaults to the app's supported installation context(s).
- [Interaction context](https://docs.discord.com/developers/interactions/application-commands#interaction-contexts) identifies where a command can run in Discord: guilds, DMs with the app's bot user, and/or group DMs and DMs not with the app's bot user.
- The app installation flow now lets users choose installation to their account or to a server.
**API Fields:**
- [Applications](https://docs.discord.com/developers/resources/application#application-object) now have `integration_types_config`, which contains default scopes and permissions for the app's supported installation contexts.
- [Commands](https://docs.discord.com/developers/interactions/application-commands#application-command-object-application-command-structure) now have `integration_types` and `contexts` for supported [installation](https://docs.discord.com/developers/interactions/application-commands#installation-context) and [interaction](https://docs.discord.com/developers/interactions/application-commands#interaction-contexts) contexts, respectively. See [command contexts](https://docs.discord.com/developers/interactions/application-commands#contexts) docs for details.
- [Interactions](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure) now include `context`, indicating the [interaction context](https://docs.discord.com/developers/interactions/application-commands#interaction-contexts) where the interaction was triggered.
- [Interactions](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure) now include `authorizing_integration_owners`, a mapping from authorized installation contexts to related snowflakes. See [Authorizing Integration Owners Object](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-authorizing-integration-owners-object).
- `app_permissions` is always serialized on interactions to show the app's available [permissions](https://docs.discord.com/developers/topics/permissions#permissions-bitwise-permission-flags) in the response context. For (G)DMs with other users, it includes `ATTACH_FILES | EMBED_LINKS | MENTION_EVERYONE`; for DMs with the app's bot user, it includes `USE_EXTERNAL_EMOJIS` for the bot's DM.
- [Messages](https://docs.discord.com/developers/resources/message#message-object) created by interaction responses or follow-ups now include `interaction_metadata`; see [Message Interaction Metadata Object](https://docs.discord.com/developers/resources/message#message-interaction-metadata-object).
- The `dm_permission` field on [Commands](https://docs.discord.com/developers/interactions/application-commands#application-command-object-application-command-structure) is deprecated. Use `contexts` instead.
- The `interaction` field on [Messages](https://docs.discord.com/developers/resources/message#message-object) is deprecated. Use `interaction_metadata` instead.

###### Limitations and Known Issues

- During preview, interaction responses for the user installation context are forced ephemeral in servers with over 25 members. The client enforces this, so apps do not need to track server size and receive no API errors.
- [follow-up messages](https://docs.discord.com/developers/interactions/receiving-and-responding#followup-messages) in DMs are currently forced ephemeral
- A bug prevents follow-up messages from respecting user permissions correctly
 March 15, 2024Breaking Changes

### Guild Prune Requiring

[Get Guild Prune Count](https://docs.discord.com/developers/resources/guild#get-guild-prune-count) and [Begin Guild Prune](https://docs.discord.com/developers/resources/guild#begin-guild-prune) now require `MANAGE_GUILD` along with the existing `KICK_MEMBERS` requirement. February 12, 2024

### Enforced Nonces on Create Message Endpoint

[Create message](https://docs.discord.com/developers/resources/message#create-message) now supports `enforce_nonce`. When true, Discord deduplicates messages from the same sender within a few minutes. If the same nonce already produced a message, Discord returns that message instead of creating a new one. A future API version will make this behavior the default. December 19, 2023

### Limit Number of Fields in Embeds

[Embed objects](https://docs.discord.com/developers/resources/message#embed-object) now enforce the 25 [embed fields](https://docs.discord.com/developers/resources/message#embed-object-embed-field-structure) limit more explicitly. Sending more than 25 entries in an embed's `fields` property returns an error. Before this change, Discord displayed only the first 25 fields and returned no error. December 15, 2023

### Clarification on Permission Splits for Expressions and Events

Behavior for `MANAGE_GUILD_EXPRESSIONS` and `MANAGE_EVENTS` is **not changing**. Bot users can continue to create, update, and delete expressions/events with these permissions. No action is required if you will keep using them. To add finer controls for expressions and events, Discord added new [permissions](https://docs.discord.com/developers/topics/permissions) for users and roles in July 2023:
- `CREATE_GUILD_EXPRESSIONS`: `1 << 43`
- `CREATE_EVENTS`: `1 << 44`
These permissions allow creating expressions and events, plus editing and deleting ones created by the current user. They rolled out to users and roles in July 2023 and appear in developer documentation, but **are not yet available to app developers**. Discord will post an update when apps can use them. December 01, 2023 Voice

### Experimenting with End-to-End Encryption for Voice & Video

#### What’s Happening?

As described in [a blog post earlier this year](https://discord.com/blog/encryption-for-voice-and-video-on-discord), Discord is testing e2ee for calls that use voice or video channels. The goal is that only participants in a call can decrypt its contents. One protocol under test is Messaging Layer Security, which Discord believes can provide encryption at large scale. Intermediaries, including Discord, cannot read communications protected by that design.

#### How do I prepare for the changes?

Developers do not need to support end-to-end encryption during this test period. When a bot user joins, voice channels automatically downgrade to documented non-e2ee protocols. This is transparent to the client connecting, but audio may arrive after a small delay.

#### What is planned for the future?

Testing will continue, with developer documentation and sample code published when ready. After publication, Discord will give developers a substantial implementation window for end-to-end encryption in voice and video. November 29, 2023 Premium Apps

### Premium App Subscriptions: New Ways for Testing App Subscriptions

Based on Premium App Subscriptions feedback, Discord made app subscription testing easier. The goal is more flexibility during testing without requiring live payment methods.
- Team members automatically receive a 100% discount on an app subscription, allowing end-to-end payment-flow testing
- Developers can create and delete [test entitlements](https://docs.discord.com/developers/resources/entitlement#create-test-entitlement) to toggle access to premium features for an application
Read [Testing your App Subscriptions Implementation](https://docs.discord.com/developers/monetization/implementing-app-subscriptions#testing-your-app-subscription-implementation) for details. November 01, 2023

### Fix Message Edit Interaction Response Permissions

Message edit interaction response actions, including [updating interaction responses](https://docs.discord.com/developers/interactions/receiving-and-responding#edit-original-interaction-response) and [sending follow-up messages](https://docs.discord.com/developers/interactions/receiving-and-responding#followup-messages), now follow the bot user's permissions. Previously, some of these actions used default permissions instead. October 19, 2023 Premium Apps

### Premium App Subscriptions Now Available in the EU and UK

Starting today, eligible EU and UK developers can monetize verified apps with App Subscriptions. [App Subscriptions](https://docs.discord.com/developers/monetization/overview) let apps charge users for premium functionality through a recurring, monthly subscription. New Premium App Subscriptions features are documented in the [App Subscriptions overview](https://docs.discord.com/developers/monetization/overview) and [the changelog for the previous App Subscriptions release](https://docs.discord.com/developers/change-log#premium-app-subscriptions-available-in-the-us). To learn eligibility and enablement details, see the [Monetization Overview](https://docs.discord.com/developers/monetization/overview). October 17, 2023

### Global Rate Limit added to discordapp.com/*

A global rate limit now applies to API requests to `discordapp.com/*`, and further restrictions may be added later. To reduce impact, call `discord.com/*`. This does **not** affect `cdn.discordapp.com`. The [API Reference](https://docs.discord.com/developers/reference) lists the URL(s) to use for REST API work.
- [February 14, 2022 Change Log](https://docs.discord.com/developers/change-log#api-v10): v10 and higher requests are no longer supported on `discordapp.com` (this does not affect `cdn.discordapp.com`)
- [May 4, 2020 #api-announcements](https://discord.com/channels/613425648685547541/697138785317814292/706944540971630662)
 September 26, 2023Premium Apps

### Premium App Subscriptions Available in the US

Starting today, eligible US-based developers can monetize verified apps with App Subscriptions. [App Subscriptions](https://docs.discord.com/developers/monetization/overview) let apps charge users for premium functionality through a recurring, monthly subscription.
- Manage subscription SKUs in the Developer Portal
- View monetization analytics in the Developer Portal
- Team owners can set up and manage payouts in Developer Portal
- New [SKUs](https://docs.discord.com/developers/resources/sku) and [Entitlements](https://docs.discord.com/developers/resources/entitlement) endpoints are available:
  - [List SKUs](https://docs.discord.com/developers/resources/sku#list-skus) `GET /applications/<application.id>/skus`
  - [List Entitlements](https://docs.discord.com/developers/resources/entitlement#list-entitlements) `GET /applications/<application.id>/entitlements`
  - [Create Test Entitlement](https://docs.discord.com/developers/resources/entitlement#create-test-entitlement) `POST /applications/<application.id>/entitlements`
  - [Delete Test Entitlement](https://docs.discord.com/developers/resources/entitlement#delete-test-entitlement) `DELETE /applications/<application.id>/entitlements/<entitlement.id>`

- [Gateway Events](https://docs.discord.com/developers/events/gateway-events#entitlements) for entitlements include `ENTITLEMENT_CREATE`, `ENTITLEMENT_UPDATE`, and `ENTITLEMENT_DELETE`
- New [`PREMIUM_REQUIRED (10)` interaction response type](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-response-object-interaction-callback-type) can prompt users to upgrade
- Interaction data payloads for [receiving and responding to interactions](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure) now include `entitlements`, an array of [entitlement](https://docs.discord.com/developers/resources/entitlement) objects
See the [Monetization Overview](https://docs.discord.com/developers/monetization/overview) for eligibility details and monetization setup. September 22, 2023

### Default Value in Auto-populated Select Menus

`default_values` was added for user (`5`), role (`6`), mentionable (`7`), and channel (`8`) [select menu components](https://docs.discord.com/developers/components/reference). `default_values` is a list of [default value objects](https://docs.discord.com/developers/components/reference#user-select-select-default-value-structure), each with an `id` snowflake and a `type` of `"user"`, `"role"`, or `"channel"`. August 23, 2023

### Team Member Roles

Teams can now invite or configure members with roles other than admin. A team member can be owner, admin, developer, or read-only; see [role types](https://docs.discord.com/developers/topics/teams#team-member-roles-team-member-role-types). The team member object now includes [`role` field](https://docs.discord.com/developers/topics/teams#data-models-team-member-object), a string for the member's current role. See updated [Teams documentation](https://docs.discord.com/developers/topics/teams#team-member-roles). August 10, 2023

### Embed Debugger

The new [Embed Debugger tool](https://discord.com/developers/embeds) previews how a URL's metadata is parsed and rendered as a Discord link embed. Use it to preview site embeds or diagnose link embed rendering problems. August 08, 2023

### Activity State for Bot Users

Bot users can now set `state` in [activity objects](https://docs.discord.com/developers/events/gateway-events#activity-object) when [updating presence](https://docs.discord.com/developers/events/gateway-events#update-presence). If the [activity's `type`](https://docs.discord.com/developers/events/gateway-events#activity-object-activity-types) is `4`, `state` appears as a custom status for the bot user; for other activity types, it appears as additional data under the activity name. August 02, 2023

### Public Preview of OpenAPI 3.1 Specification

Discord is introducing an [OpenAPI 3.1 spec](https://github.com/discord/discord-api-spec) in public preview to make HTTP API development easier and more reliable. Unlike manually reviewed developer documentation, the OpenAPI spec is generated from source code and captures more Discord API detail. The public preview may break without advance notice and should not be used in production. If something looks wrong or improvable, [open an issue](https://github.com/discord/discord-api-spec/issues). The public spec is in the [`discord-api-spec` repository on GitHub](https://github.com/discord/discord-api-spec). August 01, 2023

### New GUILD_MEDIA channel type

- Added the [`GUILD_MEDIA` (16) channel type](https://docs.discord.com/developers/resources/channel#channel-object-channel-types). `GUILD_MEDIA` channels support only threads, similar to `GUILD_FORUM` channels.
See the [media channel topic](https://docs.discord.com/developers/topics/threads#media-channels) for APIs and technical details, or the [media channel Help Center Article](https://creator-support.discord.com/hc/en-us/articles/14346342766743) for feature information. May 05, 2023

### Add Join Raid and Mention Raid fields

- Added Auto Moderation `mention_raid_protection_enabled` [trigger_metadata](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-trigger-metadata) for the `MENTION_SPAM` [trigger_type](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-trigger-types). Enabling the field together with its parent `MENTION_SPAM` rule turns on basic detection for sudden mention surges that can signal mention raids.
- Added `safety_alerts_channel_id` [guild](https://docs.discord.com/developers/resources/guild#guild-object) field and [`RAID_ALERTS_DISABLED` guild feature flag](https://docs.discord.com/developers/resources/guild#guild-object-guild-features), both tied to join raid protection
 May 03, 2023

### Unique usernames on Discord

Bot users remain on the legacy username system for now; see the [Developer Help Center article](https://dis.gd/app-usernames). Discord's username system is changing: discriminators are being removed, and unique usernames plus display names are being introduced. The [general Help Center article](https://dis.gd/usernames) explains the impact for non-bot users, while the [Developer Help Center article](https://dis.gd/app-usernames) explains bot-user impact. This changelog covers only technical app-code changes.

### Identifying migrated users

The new username system rolls out to users gradually. A user has migrated when the [`discriminator` field](https://docs.discord.com/developers/resources/user#user-object-user-structure) is exactly `"0"`. Migrated users do *not* use a 4-digits discriminator (`"0"` is not `"0000"`). The `username` field becomes the migrated user's unique username. After all users migrate, `discriminator` may be removed.

#### Example migrated user

```
{
  "id": "80351110224678912",
  "username": "nelly",
  "discriminator": "0",
  "global_name": "Nelly",
  "avatar": "8342729096ea3675442027381ff50dfe",
  "verified": true,
  "email": "nelly@discord.com",
  "flags": 64,
  "banner": "06c16474723fe537c283b8efa61a30c8",
  "accent_color": 16711680,
  "premium_type": 1,
  "public_flags": 64
}

```

### Display names

Under the new username system, standard Discord users can set a non-unique display name. This is exposed as `global_name`, with a max length of 32 characters. If no display name is set, `global_name` is null.

### Default avatars

For migrated accounts, default avatar URLs are based on user ID instead of discriminator. Calculate the URL with `(user_id >> 22) % 6`. Legacy username accounts continue to use `discriminator % 5`. April 14, 2023

### Bot users added to all new apps

Starting today, [bot users](https://docs.discord.com/developers/topics/oauth2#bot-vs-user-accounts) are added to every newly created app. Bot user settings and configuration remain on the **Bot** page in the [app's settings](https://discord.com/developers/applications). If an app does not need or want a bot user, do not include the [`bot` scope](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-scopes) during installation. April 06, 2023

### Interaction Channel Data

Interactions now include `channel`, a partial channel object guaranteed to contain `id` and `type`. Start using this field to identify the interaction source channel; `channel_id` may be deprecated later. See the [interaction documentation](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object). February 24, 2023

### Add Auto Moderation custom_message Action Metadata Field

Added `custom_message` [action metadata](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-action-object-action-metadata) for the `BLOCK_MESSAGE` [action type](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-action-object-action-types)). Each Auto Moderation rule can now define a custom string shown to members when the rule blocks their message. Use it to explain the block and help members understand server rules and guidelines. February 10, 2023

### Update to Locked Threads

### Upcoming Changes

At the time of this notice, Discord threads, including forum posts, could be archived alone or locked together with archived status. Starting on **March 6, 2023**, threads can be locked *without* being archived, changing the meaning of the [`locked` field](https://docs.discord.com/developers/resources/channel#thread-metadata-object-thread-metadata-structure). Today, `locked` means a user without [`MANAGE_THREADS` (`1 << 34`) permission](https://docs.discord.com/developers/topics/permissions#permissions-bitwise-permission-flags) cannot reopen the thread, but it does not limit activity inside active, non-archived threads. After this change, users and bot users without `MANAGE_THREADS` face more restrictions in locked threads: they cannot create or update messages, or update properties such as title or tags. Some actions, including deleting messages and adding or removing reactions, are allowed in locked threads *only* when the thread is active, meaning un-archived. Users or bot users with `MANAGE_THREADS` can still modify the thread and messages. The change does not affect [`archived` field](https://docs.discord.com/developers/resources/channel#thread-metadata-object-thread-metadata-structure) meaning or behavior for threads that are both locked and archived.

### How do I prepare for this change?

Apps that work with threads, including forum posts, should inspect `locked` and/or `archived` to decide which actions are allowed. Be ready to handle errors when a thread is locked. February 08, 2023

### Increase Auto Moderation Keyword Limits

- Increased maximum number of rules with `KEYWORD` [trigger_type](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-trigger-types) per guild from 5 to 6
- Increased maximum length for each keyword in `keyword_filter` and `allow_list` [trigger_metadata](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-trigger-metadata) from 30 to 60.
 January 18, 2023

### Guild Audit Log Events
