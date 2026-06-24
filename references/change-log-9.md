# Change Log (part 9)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

### Upcoming Permissions Change to Webhook Routes

On August 8th, 2022 Discord will start requiring `VIEW_CHANNEL (1 << 10)` on webhook routes that already require `MANAGE_WEBHOOKS (1 << 29)`, matching the documented permission model. Most applications should not be affected, but affected apps should update permissions for these routes:
- [`GET /webhooks/{webhook.id}`](https://docs.discord.com/developers/resources/webhook#get-webhook)
- [`DELETE /webhooks/{webhook.id}`](https://docs.discord.com/developers/resources/webhook#delete-webhook)
- [`PATCH /webhooks/{webhook.id}`](https://docs.discord.com/developers/resources/webhook#modify-webhook)
- [`GET /channels/{channel.id}/webhooks`](https://docs.discord.com/developers/resources/webhook#get-channel-webhooks)
- [`POST /channels/{channel.id}/webhooks`](https://docs.discord.com/developers/resources/webhook#create-webhook)
 July 01, 2022Breaking Changes

### Add Subcommand Groups and Subcommands to Message Interaction Objects

This is a breaking change, but most apps depend on interaction responses (`INTERACTION_CREATE`) rather than message interaction objects (`MESSAGE_CREATE`). [Interaction responses](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-data) do not change.

#### Upcoming Changes

Beginning **July 18, 2022**, the `name` field on [message interaction objects](https://docs.discord.com/developers/interactions/receiving-and-responding#message-interaction-object) will include subcommands and subcommand groups along with the top-level command. Future apps should avoid depending on this `name` field. The `name` value is built from command levels, when present, separated by spaces: `<command name> <subcommand group name> <subcommand name>`. The `name` field appears only on messages that respond to an interaction without an existing message, so message component interaction objects do not include it.

#### Updating your app

Most apps use interaction responses rather than message interaction objects. Depending on `name` in message interaction objects is not recommended; apps that do so should handle subcommands and subcommand groups. For example, suppose an app has `/role` with subcommands `add` and `remove`. Today, the original interaction payload has `name` set to `role`, and a fetched response message's message interaction object also has `name` set to `role`. After the change, the original interaction payload still uses `role`, but the fetched response message's message interaction object may use `role add` or `role remove`. July 01, 2022

### Min and Max Length for Command Options

Application [command options](https://docs.discord.com/developers/interactions/application-commands#application-command-object-application-command-option-structure) of type `STRING` now support optional `min_length` and `max_length` fields for controlling user-entered text length. `min_length` must be greater than or equal to `0`, and `max_length` must be greater than or equal to `1`. June 29, 2022

### Calculated Permissions in Interaction Payloads

Interaction payloads now include `app_permissions`, containing the computed [permissions](https://docs.discord.com/developers/topics/permissions#permissions-bitwise-permission-flags) for a bot or app in the interaction's context, including channel overwrites. Like other permission values, `app_permissions` is a string containing a bitwise OR-ed permission set. See the [interactions documentation](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object). Apps without a bot user, or without the `bot` scope, receive the permissions for `@everyone`, limited to permissions usable in interaction responses: `ATTACH_FILES`, `EMBED_LINKS`, `MENTION_EVERYONE`, and `USE_EXTERNAL_EMOJIS`. June 29, 2022Breaking Changes

### Changes to Bot Permissions for Interactions and Webhooks

#### Upcoming Changes

Only `MENTION_EVERYONE`, `SEND_TTS_MESSAGES`, and `USE_EXTERNAL_EMOJIS` are affected. An older version of this changelog said `ATTACH_FILES` and `EMBED_LINKS` would be affected, but they are not. Starting **August 3, 2022**, calculation of some bot `MENTION_EVERYONE`, `SEND_TTS_MESSAGES`, and `USE_EXTERNAL_EMOJIS` [permissions](https://docs.discord.com/developers/topics/permissions) changes in two places:
- When **responding to an [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding)**, such as application commands or message components
- When **executing a [webhook](https://docs.discord.com/developers/resources/webhook) that the bot created**
In those cases, a bot's `MENTION_EVERYONE`, `SEND_TTS_MESSAGES`, and `USE_EXTERNAL_EMOJIS` permissions will be calculated from permissions granted to the bot, *including* [overwrites](https://docs.discord.com/developers/topics/permissions#permission-overwrites). Previously, these cases used only permissions granted to `@everyone`. This applies only to bots. Apps without a bot user, or without the `bot` scope, still depend on `@everyone`.

#### Updating Your App

If a bot needs `MENTION_EVERYONE`, `SEND_TTS_MESSAGES`, or `USE_EXTERNAL_EMOJIS` while replying to interactions or executing a webhook, make sure the bot was installed with those permissions or explicitly granted them. Overwrites can still change permissions after installation. For interactions, inspect [`app_permissions` field](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure) before replying to determine the app or bot's `app_permissions` context. June 21, 2022Breaking Changes

### Message Content in Auto Moderation events

In API v10, receiving non-empty `content` and `matched_content` values in [`AUTO_MODERATION_ACTION_EXECUTION`](https://docs.discord.com/developers/events/gateway-events#auto-moderation-action-execution) gateway events requires the `MESSAGE_CONTENT` (`1 << 15`) intent. This matches intended message-content behavior across the API. June 17, 2022

### Updated Connection Property Field Names

The `$` prefix on [identify connection properties](https://docs.discord.com/developers/events/gateway-events#identify-identify-connection-properties) is deprecated. Use `os`, `browser`, and `device` instead. If `$`-prefixed names are passed, they resolve to the new names. API v11 removes support for `$os`, `$browser`, and `$device`. June 16, 2022

### Auto Moderation

Discord added [Auto Moderation feature](https://docs.discord.com/developers/resources/auto-moderation), allowing guilds to moderate message content for keywords, harmful links, and unwanted spam. The change includes:
- New endpoints for [creating](https://docs.discord.com/developers/resources/auto-moderation#create-auto-moderation-rule), [updating](https://docs.discord.com/developers/resources/auto-moderation#modify-auto-moderation-rule), and [deleting](https://docs.discord.com/developers/resources/auto-moderation#delete-auto-moderation-rule) Auto Moderation rules
- New gateway events for Auto Moderation rules being [created](https://docs.discord.com/developers/events/gateway-events#auto-moderation-rule-create) (`AUTO_MODERATION_RULE_CREATE`), [updated](https://docs.discord.com/developers/events/gateway-events#auto-moderation-rule-update) (`AUTO_MODERATION_RULE_UPDATE `), and [deleted](https://docs.discord.com/developers/events/gateway-events#auto-moderation-rule-delete) (`AUTO_MODERATION_RULE_DELETE `). These require the `AUTO_MODERATION_CONFIGURATION` (`1 << 20`) intent
- New gateway event emitted when an [action is executed](https://docs.discord.com/developers/events/gateway-events#auto-moderation-action-execution) (`AUTO_MODERATION_ACTION_EXECUTION`). This requires the `AUTO_MODERATION_EXECUTION` (`1 << 21`) intent
- New [audit log entries](https://docs.discord.com/developers/resources/audit-log#audit-log-entry-object-audit-log-events) for rule creation (`AUTO_MODERATION_RULE_CREATE`), update (`AUTO_MODERATION_RULE_UPDATE`), deletion (`AUTO_MODERATION_RULE_DELETE`), and Auto Moderation action execution (`AUTO_MODERATION_BLOCK_MESSAGE`)
 April 27, 2022Breaking Changes

### Updated Command Permissions

Application command permissions now provide more granular command access control in Discord. The main changes are summarized below; see [the updated documentation](https://docs.discord.com/developers/interactions/application-commands#permissions) for full details.

#### Breaking changes

- Editing command permissions now requires Bearer tokens. These tokens are based on the permissions of the user who authenticated through [OAuth](https://docs.discord.com/developers/topics/oauth2), and that user needs permission to manage both the guild and its roles.
- [`applications.commands.permissions.update`](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-scopes) scope is now required to edit command permissions.
- The batch editing endpoint ([`PUT /applications/{application.id}/guilds/{guild.id}/commands/permissions`](https://docs.discord.com/developers/interactions/application-commands#batch-edit-application-command-permissions)) has been disabled.

#### Other changes

- Added a [`CHANNEL` command permission type](https://docs.discord.com/developers/interactions/application-commands#application-command-permissions-object-application-command-permission-type)
- Increase permission limit from `10` to `100`
- Added a [constant (`guild_id - 1`)](https://docs.discord.com/developers/interactions/application-commands#application-command-permissions-object-application-command-permissions-constants) representing all channels in command permissions
- Added `default_member_permissions`, a string containing a bitwise OR-ed [permissions](https://docs.discord.com/developers/topics/permissions#permissions-bitwise-permission-flags) set. It replaces `default_permission`, which will soon be deprecated.
- Added `dm_permission`, a boolean that controls whether a global application command is available in DMs. If omitted, the global command is visible in DMs.
- Added `APPLICATION_COMMAND_PERMISSIONS_UPDATE` [gateway](https://docs.discord.com/developers/events/gateway-events#application-command-permissions-update) event and `APPLICATION_COMMAND_PERMISSION_UPDATE` [audit log](https://docs.discord.com/developers/resources/audit-log) event.
 April 06, 2022

### Forum Channels

Discord added `GUILD_FORUM` (15), a new unreleased channel type. A `GUILD_FORUM` channel is similar to `GUILD_TEXT` from the API perspective, except it only permits thread creation and does not accept direct messages. See the [forums topic](https://docs.discord.com/developers/topics/threads#forums) for more information. March 31, 2022

### Guild Bans Pagination

`GET /guilds/{guild.id}/bans` now requires pagination to improve reliability and stability. See the [endpoint docs](https://docs.discord.com/developers/resources/guild#get-guild-bans) for details. February 14, 2022

### API v10

- API v8 is now deprecated.
- `GET /channels/{channel.id}/threads/active` is decommissioned and replaced by [`GET /guilds/{guild.id}/threads/active`](https://docs.discord.com/developers/resources/guild#list-active-guild-threads).
- In v10 and later, message-related dispatch content fields require the message content intent (`1 << 15`). See [Gateway Intents documentation](https://docs.discord.com/developers/events/gateway#gateway-intents).
- Apps must provide administrative-action audit log reasons with the `X-Audit-Log-Reason` header instead of the `reason` parameter on all endpoints. See [Audit Logs documentation](https://docs.discord.com/developers/resources/audit-log).
- Message routes, such as [`POST /channels/{channel.id}/messages`](https://docs.discord.com/developers/resources/message#create-message), now use `embeds`, an array of embed objects, instead of `embed`.
- The `summary` field for [applications](https://docs.discord.com/developers/resources/application) returns an empty string in every API version.
- For [Achievements](https://github.com/discord/discord-api-docs/blob/legacy-gamesdk/docs/game_sdk/Achievements.md#achievement-struct), `name` and `description` are now strings, and localization data uses `name_localizations` and `description_localizations` dictionaries. This aligns localization with [Application Commands](https://docs.discord.com/developers/interactions/application-commands#localization). See [Achievements documentation](https://github.com/discord/discord-api-docs/blob/legacy-gamesdk/docs/game_sdk/Achievements.md#achievement-struct).
- When [`PATCH`ing messages with new attachments](https://docs.discord.com/developers/reference#editing-message-attachments), existing attachments must be included. Any attachment omitted from the request is removed and replaced by the provided list.
- Requests to v10 and higher are no longer supported on `discordapp.com`; `cdn.discordapp.com` is not affected.

#### Upcoming changes

- API v6 and v7 will be decommissioned **in early 2023**
- `MESSAGE_CONTENT` becomes a privileged intent **on August 31, 2022** for verified bots that are in 75+ servers. See [the FAQ](https://support-dev.discord.com/hc/en-us/articles/4404772028055-Message-Content-Privileged-Intent-FAQ) or [the guide](https://docs.discord.com/developers/tutorials/upgrading-to-application-commands) for upgrade details.
- Applications will lose the `summary` field in v11, the next API version
 February 08, 2022

### Interaction Modals and Application Command Attachment Option Type

Interaction modals are available, letting applications request more detailed input from users. See [the modal docs](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-response-object-modal). Application Commands can now include an attachment option type; see [the option type table](https://docs.discord.com/developers/interactions/application-commands#application-command-object-application-command-option-type). December 20, 2021

### Guild Member Timeouts

Added documentation for the recently released guild member timeout feature. November 23, 2021

### Guild Scheduled Events

- Added official support for `guild_scheduled_events` on the `Guild` resource sent with `GUILD_CREATE` event

#### Nov 18, 2021

- Breaking change to the return type for `GET /guilds/{guild.id}/scheduled-events/{guild_scheduled_event.id}/users`
- Added `with_user_count` query param for `GET /guilds/{guild.id}/scheduled-events/{guild_scheduled_event.id}`
- Response for `GET /guilds/{guild.id}/scheduled-events/{guild_scheduled_event.id}` now returns an extra `creator` field by default
- Added more detail and clarification for Guild Scheduled Events.
- Documented `before` and `after` query params for `GET /guilds/{guild.id}/scheduled-events/{guild_scheduled_event.id}/users`

#### Nov 15, 2021

Added documentation for the recently released Guild Scheduled Events feature. October 27, 2021

### Application Command Autocomplete Interactions

Autocomplete interactions are available, allowing application commands to provide server-completed options. See [the autocomplete interaction docs](https://docs.discord.com/developers/interactions/application-commands#autocomplete). September 16, 2021

### Updated Thread Permissions

Thread permissions were simplified and updated:
- "Use Public Threads" became "Create Public Threads", allowing users to create public threads and announcement threads in a channel even without permission to send messages there.
- "Use Private Threads" became "Create Private Threads", allowing users to create private threads in a channel even without permission to send messages there.
A new permission was added:
- "Send Messages in Threads" lets users send messages in a thread. "Send Messages" has no effect inside threads: users **must** have "Send Messages in Threads" to send a thread message. This supports configurations where a user can participate in a thread but not send messages in the parent channel, such as a thread on an announcement post.
 August 10, 2021

### User and Message Commands

[User commands](https://docs.discord.com/developers/interactions/application-commands#user-commands) and [message commands](https://docs.discord.com/developers/interactions/application-commands#message-commands) are live. They appear on context menus for users and messages, with more planned. Context menu commands are application commands. The "Slash Commands" documentation page was renamed to "Application Commands" and separated by command type. June 30, 2021

### Select Menu Components

Select Menus are part of the components API. They let apps present one or many UI-based choices to users. Select menus work like other [message components](https://docs.discord.com/developers/components/overview); see the [documentation](https://docs.discord.com/developers/components/reference#string-select). June 10, 2021

### Support for Multiple Embeds in Message Routes

Message routes now accept an embeds array in addition to the old embed field. Bots can send up to 10 embeds per message, matching webhook behavior. The old embed field is deprecated; a later API version removes it after May 26, 2021.

### Buttons and Message Components

Message components are available with two initial components: layout-based `ActionRow` and buttons. Apps can include buttons on bot messages and interaction responses. See [Learn more about message components](https://docs.discord.com/developers/components/overview). This adds these fields and response types:
- Optional `components` field added to the [message object](https://docs.discord.com/developers/resources/message#message-object)
- New response types `6` and `7` added for [interaction responses](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-response-object-interaction-callback-type), valid only for component-based interactions
 April 28, 2021

### API v9

API v9 is available. It includes support for [threads](https://docs.discord.com/developers/topics/threads), an upcoming feature. Older API versions will not receive Gateway Events for threads, so updating is important. The [migration guide](https://docs.discord.com/developers/topics/threads) is available to simplify the upgrade. This documentation is being released early so bots have at least two months to upgrade before threads launch. API v9 also removes `/channels/:id/messages/:id/suppress-embeds`. April 05, 2021

### Application Command Permissions

Commands now support [command permissions](https://docs.discord.com/developers/interactions/application-commands#permissions). A guild or global command can be enabled or disabled for a specific user or role in a guild. At this stage, users can still see commands they cannot use. New routes support this:
- [`GET Guild Application Command Permissions`](https://docs.discord.com/developers/interactions/application-commands#get-guild-application-command-permissions)
- [`GET Application Command Permissions`](https://docs.discord.com/developers/interactions/application-commands#get-application-command-permissions)
- [`PUT Application Command Permissions`](https://docs.discord.com/developers/interactions/application-commands#batch-edit-application-command-permissions)
The [ApplicationCommand](https://docs.discord.com/developers/interactions/application-commands#application-command-object-application-command-structure) model gained `default_permission`. It can disable commands for everyone in a guild by default when commands should be opt-in. March 15, 2021

### Large Bot Sharding Lowered to 150,000 Guilds

Bots in many guilds were reported to hit more session-start errors, including gateway connection timeouts. To improve reliability, the large bot sharding threshold was lowered to 150,000 guilds. March 05, 2021

### Changes to Slash Command Response Types and Flags

Interaction response types changed to better support application command designs:
- Type `2` `Acknowledge` has been deprecated
- Type `3` `ChannelMessage` has been deprecated
- Type `5` `AcknowledgeWithSource` was renamed to `DeferredChannelMessageWithSource`
These deprecated types will be removed and become breaking on **April 9, 2021**. `flags` is now documented on [InteractionApplicationCommandCallbackData](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-response-object-interaction-callback-data-structure). Setting `flags` to `64` makes the interaction response ephemeral. February 09, 2021

### Slash Commands in DMs

Slash Commands now work in DMs with bots. To support DM command context, some [Interaction object](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure) fields became optional. The newly optional fields do not change Slash Command behavior in guilds. January 22, 2021

### Change to Permission Checking when Creating Channels

The guild channel creation endpoint now validates permission overwrites against the bot's guild permissions. Permission overwrites in the request body require the bot to have the permissions being applied. Setting `MANAGE_ROLES` in channel overwrites is allowed only for guild administrators or users with `MANAGE_ROLES` as a channel permission overwrite. December 15, 2020

### Slash Commands and Interactions

Slash Commands are available. See [Slash Commands](https://docs.discord.com/developers/interactions/application-commands) for the detailed documentation. Slash Commands add these webhook features:
- Webhooks can update previously sent messages from the same webhook with [Edit Webhook Message](https://docs.discord.com/developers/resources/webhook#edit-webhook-message) and [Delete Webhook Message](https://docs.discord.com/developers/resources/webhook#delete-webhook-message)
This PR documents the `application` field on the `READY` gateway event. It is a partial [application object](https://docs.discord.com/developers/resources/application#application-object) containing `id` and `flags`. November 16, 2020
