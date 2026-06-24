# Change Log (part 8)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Discord added the [`GUILD_AUDIT_LOG_ENTRY_CREATE`](https://docs.discord.com/developers/events/gateway-events#guild-audit-log-entry-create) gateway event so applications can respond to guild moderation actions. Receiving it requires `VIEW_AUDIT_LOG`, and gateway connections must request the [`GUILD_MODERATION` intent](https://docs.discord.com/developers/events/gateway#gateway-intents). January 09, 2023Breaking Changes

### Thread Member Details and Pagination

The [thread member object](https://docs.discord.com/developers/resources/channel#thread-member-object) gained a `member` field. That `member` value contains a [guild member object](https://docs.discord.com/developers/resources/guild#guild-member-object) and is included in returned thread member objects when `with_member` is `true` for [List Thread Members](https://docs.discord.com/developers/resources/channel#list-thread-members) (`GET /channels/<channel_id>/thread-members`) and [Get Thread Member](https://docs.discord.com/developers/resources/channel#get-thread-member) (`GET /channels/<channel_id>/thread-members/<user_id>`). Passing `with_member` as `true` also turns on pagination for [List Thread Members](https://docs.discord.com/developers/resources/channel#list-thread-members). With pagination, use `after` and `limit` to request later members and control result size. The default `limit` is 100.

#### Upcoming Changes

Beginning with API v11, [List Thread Members](https://docs.discord.com/developers/resources/channel#list-thread-members) (`GET /channels/<channel_id>/thread-members`) will always return paginated results, whether or not `with_member` is provided. December 13, 2022

### Add Default Layout setting for Forum channels

`default_forum_layout` is an optional [channel object](https://docs.discord.com/developers/resources/channel) field that controls the default post layout for a [forum channel](https://docs.discord.com/developers/topics/threads#forums). 1 (`LIST_VIEW`) displays posts as a chronological list, while 2 (`GALLERY_VIEW`) displays them as tiles. If unset, `default_forum_layout` is `0`. Changing `default_forum_layout` requires `MANAGE_CHANNELS`. December 12, 2022

### Add Application Connections Metadata and Linked Roles

Discord introduced [linked roles](https://discord.com/blog/connected-accounts-functionality-boost-linked-roles) and lets developers configure linked roles for their own applications. This includes:
- New [`role_connections_verification_url`](https://docs.discord.com/developers/resources/application#application-object), configurable in the developer portal so the application can appear as a linked-role verification option.
- [Application metadata](https://docs.discord.com/developers/resources/application-role-connection-metadata#application-role-connection-metadata-object) for expressing more precise linked-role requirements.
- New endpoints to [retrieve](https://docs.discord.com/developers/resources/application-role-connection-metadata#get-application-role-connection-metadata-records) (`GET /applications/<application.id>/role-connections/metadata`) and [update](https://docs.discord.com/developers/resources/application-role-connection-metadata#update-application-role-connection-metadata-records) (`PUT /applications/<application.id>/role-connections/metadata`) application connection metadata.
- New [`role_connections.write`](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-scopes) OAuth2 scope, required for the requests below.
- Endpoints to [retrieve](https://docs.discord.com/developers/resources/user#get-current-user-application-role-connection) (`GET /users/@me/applications/{application.id}/role-connection`) and [update](https://docs.discord.com/developers/resources/user#update-current-user-application-role-connection) (`PUT /users/@me/applications/{application.id}/role-connection`) a user's role connections; both return an [application role connection](https://docs.discord.com/developers/resources/user#application-role-connection-object) object.
For a quick setup path for linked roles, see the [tutorial](https://docs.discord.com/developers/tutorials/configuring-app-metadata-for-linked-roles). November 22, 2022

### Add Auto Moderation Allow List for Keyword Rules and Increase Max Keyword Rules Per Guild Limit

- Auto Moderation rules whose [trigger_type](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-trigger-types) is `KEYWORD` now accept `allow_list` inside [trigger_metadata](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-trigger-metadata). Message content matching an `allow_list` keyword is ignored by that Auto Moderation `KEYWORD` rule. Each `allow_list` keyword may be a multi-word phrase and may use [wildcard symbols](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-keyword-matching-strategies).
- The per-guild maximum for rules using `KEYWORD` [trigger_type](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-trigger-types) increased from 3 to 5
- Each regex pattern in `regex_patterns` under [trigger_metadata](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-trigger-metadata) may now be up to 260 characters instead of 75.
 November 17, 2022Breaking Changes

### Upcoming Application Command Permission Changes

Discord is revising [application commands](https://docs.discord.com/developers/interactions/application-commands) permissions based on feedback, with the goal of simplifying management and making command permissions behave more like other Discord permission systems. Server admins can opt in to the permissions changes described here per server **starting on December 16, 2022**. The change will not reach all servers **until late January or early February**. Existing behavior is described in [the application commands documentation](https://docs.discord.com/developers/interactions/application-commands#permissions) and [the changelog for the previous permissions update](https://docs.discord.com/developers/change-log#updated-command-permissions). The changes mainly affect how Discord clients apply configured permissions, so most apps need no action. Apps that use the [Update Permissions endpoint](https://docs.discord.com/developers/interactions/application-commands#edit-application-command-permissions) (`PUT /applications/<application_id>/guilds/<guild_id>/commands/<command_id>/permissions`) should review the details carefully.

#### Types of command permission configurations

The following concepts remain the same, but they provide useful background. Discord clients decide whether a user can see or invoke a command using three permission configuration types:
- **Command-level permissions** are configured by an admin for one specific *command* in a server and affect only that command.
- **App-level permissions** are configured by an admin for one *app* in a server and affect all commands from that app.
- **`default_member_permissions`** are set by the app during command create or update. `default_member_permissions` apply to that command in *all* servers unless an override exists. See [the documentation](https://docs.discord.com/developers/interactions/application-commands#application-command-permissions-object-using-default-permissions) for `default_member_permissions` details.
These permission configuration concepts are unchanged, which leads to the question of what is changing.

### What’s changing?

Two command-permission changes are being introduced:
1. Discord clients will use new logic when applying permission configurations to a user in a specific context
2. A new `APPLICATION_COMMAND_PERMISSIONS_V2` guild feature flag indicates whether a guild is on the old or upcoming permission logic.
Both changes are described below.

#### 1. How permission configurations are applied in Discord

##### Current behavior:

Under the current system, configuration types are **mutually-exclusive**: only one permission configuration type determines whether a user may invoke a command. The hierarchy is command-level permissions first when present, then app-level permissions when present, and finally `default_member_permissions` if neither of the first two exists. This means:
- Any configured command-level permissions cause app-level permissions and `default_member_permissions` to be ignored for that command.
- Any configured app-level permissions cause `default_member_permissions` to be ignored for every command from that app.
This behavior can accidentally escalate permissions and make moderators recreate app-level settings when they only want a small command-level adjustment.

##### Upcoming behavior:

The new model removes mutual exclusion, allowing multiple permission configuration types to combine when deciding command access. **`default_member_permissions` remains a developer-provided default for command creation or update.** **App-level permission configurations become the base configuration.** They define who may use the app and where, and they combine with `default_member_permissions`; a user granted app-level access still remains limited by each command's `default_member_permissions` by default. This prevents granting `/ban`, which requires `BAN_MEMBERS`, to `@BotMemers` simply because they have app access. **Command-level permission configurations become app-level overrides.** They override app-level settings and restrictions from `default_member_permissions`, so an admin can explicitly allow a user to use a command even if the app-level configuration denies them or they lack the command's `default_member_permissions`. If no command-level configuration exists for the context, Discord falls back to the app-level configuration.

##### Flowchart for command permissions logic

The simplified flowchart below shows how the Discord client applies permissions after the new behavior takes effect. (Figure: Flowchart with an overview of the new permissions configurations logic)

#### 2. `APPLICATION_COMMAND_PERMISSIONS_V2` Guild Feature

Discord added [`APPLICATION_COMMAND_PERMISSIONS_V2` feature flag](https://docs.discord.com/developers/resources/guild#guild-object-guild-features) to show whether a server is using **the current permissions logic**.
- If the flag is present, the server still uses the old command permissions behavior.
- If the flag is absent, the server has moved from the old command permissions behavior to the new one.

### Am I affected?

Only apps that call [`PUT /applications/<application_id>/guilds/<guild_id>/commands/<command_id>/permissions`](https://docs.discord.com/developers/interactions/application-commands#edit-application-command-permissions) are affected. This limited endpoint manages application command permissions for admins and requires the `applications.commands.permissions.update` scope. **Apps that do not use this endpoint have nothing to prepare for these changes.** Apps that do use it should follow the preparation guidance below.

### How do I prepare for the changes?

Prepare for the change in two steps. **1. Use the `APPLICATION_COMMAND_PERMISSIONS_V2` flag** to detect which permission logic a server uses. During migration, code may need separate paths depending on whether the flag exists.

```
if 'APPLICATION_COMMAND_PERMISSIONS_V2' in guild.features:
     # Use current behaviors when interacting with endpoint
else:
     # Use new permissions behaviors when interacting with endpoint

```

If guild features are not already available from Gateway events, fetch them with [`GET /guilds/<guild_id>` endpoint](https://docs.discord.com/developers/resources/guild#get-guild). **2. Modify the behavior based on your use case**. Once the server's permission behavior is known, adjust how your app handles that server. Review what admins expect when your app updates command permissions. For dashboards, compare the dashboard logic and configured permissions against the new behavior. For documentation that explains command-permission behavior, remap those explanations to the new model. If the correct mapping is unclear, ask admin users whether the new logic matches their expectations.

#### What happens if I don’t update my app?

Affected apps that do not update may configure permission behavior that differs from both developer intent and user expectations.

#### How long do I have to update my app?

The `APPLICATION_COMMAND_PERMISSIONS_V2` flag is already live and should appear in guild feature flags. New permission behavior rolls out **on December 16, 2022**, when admins start seeing a banner that lets them optionally migrate their server. In **late January or early February**, all servers move to the new behavior. Discord will publish another changelog then, after which old-behavior logic can be removed. November 09, 2022Breaking Changes

### GameSDK Feature Deprecation

Discord is deprecating the following GameSDK features **starting today** and will decommission them on **Tuesday, May 2, 2023**, to focus on more relevant gaming features and experiences:
- [Achievements](https://github.com/discord/discord-api-docs/blob/legacy-gamesdk/docs/game_sdk/Achievements.md)
- [Applications](https://github.com/discord/discord-api-docs/blob/legacy-gamesdk/docs/game_sdk/Applications.md)
- [Voice](https://github.com/discord/discord-api-docs/blob/legacy-gamesdk/docs/game_sdk/Discord_Voice.md)
- [Images](https://github.com/discord/discord-api-docs/blob/legacy-gamesdk/docs/game_sdk/Images.md)
- [Lobbies](https://github.com/discord/discord-api-docs/blob/legacy-gamesdk/docs/game_sdk/Lobbies.md)
- [Networking](https://github.com/discord/discord-api-docs/blob/legacy-gamesdk/docs/game_sdk/Networking.md)
- [Relationships](https://github.com/discord/discord-api-docs/blob/legacy-gamesdk/docs/game_sdk/Relationships.md)
- [Storage](https://github.com/discord/discord-api-docs/blob/legacy-gamesdk/docs/game_sdk/Storage.md)
- [Store](https://github.com/discord/discord-api-docs/blob/legacy-gamesdk/docs/game_sdk/Store.mdmd)
The deprecation window runs until **Tuesday May 2, 2023**. After that date, these parts will be decommissioned and stop working. Other GameSDK features remain supported. Developers can still use the GameSDK for Rich Presence, relationships, entitlements, basic user details, and the overlay. November 04, 2022

### Add Auto Moderation Regex Support

Auto Moderation rules using [trigger_type](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-trigger-types) `KEYWORD` now support `regex_patterns` in [trigger_metadata](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-trigger-types). Regex patterns can represent many keywords in one expression. Only Rust flavored regex is supported; tools such as [Rustexp](https://rustexp.lpil.uk/) can test it. October 20, 2022

### Delete Ephemeral Messages

Ephemeral interaction responses and followups can now be deleted with a valid interaction token through these endpoints:
- [`DELETE /webhooks/<application_id>/<interaction_token>/messages/@original`](https://docs.discord.com/developers/interactions/receiving-and-responding#delete-original-interaction-response)
- [`DELETE /webhooks/<application_id>/<interaction_token>/messages/<message_id>`](https://docs.discord.com/developers/interactions/receiving-and-responding#delete-followup-message)
Interaction tokens remain valid for up to 15 minutes after the interaction. See the [interaction documentation](https://docs.discord.com/developers/interactions/receiving-and-responding) for details. October 13, 2022

### New Select Menu Components

Discord added four select menu [component types](https://docs.discord.com/developers/components/reference#component-object-component-types) to make common Discord resources easier to present in selects:
- User select (type `5`)
- Role select (type `6`)
- Mentionable select for users *and* roles (type `7`)
- Channel select (type `8`)
The new select menu components follow the existing string select menu shape, except they omit `options`; channel select menus may also include `channel_types`. The [select menu interaction](https://docs.discord.com/developers/components/reference#string-select-string-select-interaction-response-structure) payload for the new components includes a [`resolved` field](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-resolved-data-structure). See the updated [select menu documentation](https://docs.discord.com/developers/components/reference#component-object-component-types) and the [select menu interaction](https://docs.discord.com/developers/components/reference#string-select-string-select-interaction-response-structure) details for more information. September 22, 2022

### Default Sort Order for Forum Channels

`default_sort_order` is an optional [channel object](https://docs.discord.com/developers/resources/channel) field that controls the default sorting of threads in a [forum channel](https://docs.discord.com/developers/topics/threads#forums). Changing `default_sort_order` requires `MANAGE_CHANNELS`. If unset, `default_sort_order` is `null`. September 21, 2022

### Auto Moderation Spam and Mention Spam Trigger Types

Auto Moderation gained two new [trigger types](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-trigger-types):
- `MENTION_SPAM` blocks messages that mention more than a configured number of unique guild members or roles. Apps set that number up to 50 using `mention_total_limit` in the [trigger metadata object](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-trigger-metadata) when creating or updating an Auto Moderation rule.
- `SPAM` blocks messages and links classified as spam.
See the [Auto Moderation documentation](https://docs.discord.com/developers/resources/auto-moderation) for more information. September 14, 2022

### Forum Channels Release

Forum channels ([`GUILD_FORUM` or `15`](https://docs.discord.com/developers/resources/channel#channel-object-channel-types)) are available to all community servers. `GUILD_FORUM` is a channel type that supports only threads, displayed differently than text (`GUILD_TEXT`) channels. See the [forums topic](https://docs.discord.com/developers/topics/threads#forums) for APIs and technical details, and the [Forums FAQ](https://support.discord.com/hc/en-us/articles/6208479917079-Forum-Channels-FAQ#h_01G69FJQWTWN88HFEHK7Z6X79N) for feature details. September 01, 2022Breaking Changes

### Message Content is a Privileged Intent

As of this change, [message content](https://docs.discord.com/developers/events/gateway#message-content-intent) is privileged for all verified apps and apps eligible for verification. The reason and application process are described in the [Help Center FAQ](https://support-dev.discord.com/hc/articles/4404772028055-Message-Content-Privileged-Intent-FAQ). Apps without the message content intent enabled in Developer Portal settings receive empty values for API fields that expose message content, including `content`, `embeds`, `attachments`, and `components`. These limits do not apply to messages sent by the bot or app, DMs it receives, or messages that mention it.

#### If your app is verified

Verified apps and verification-eligible apps need approval for the message content intent to receive message content. A verified app that lacks approval, or does not account for the restrictions, will break for users.

##### Temporary Message Content Intent

Verified apps and apps submitted for verification can temporarily opt in to a grace period that allows continued message-content access until October 1. Opting in prevents the app from joining additional servers until it opts out. See the [Help Center article](https://support-dev.discord.com/hc/en-us/articles/8561391080471) for details.

#### If your app is unverified

Unverified apps still need to enable the intent in the app's Developer Portal settings. Existing unverified apps automatically have message content intent toggled on, while new unverified apps must enable it manually in the Developer Portal. August 22, 2022

### Slash Command Mentions

[Slash Command mentions](https://docs.discord.com/developers/reference#message-formatting) are rolling out across Discord clients this week; on Android, they are limited to the [React Native client](https://discord.com/blog/android-react-native-framework-update). Clicking a Slash Command mention fills the user's message input with the command. Slash Command mentions use `</NAME:COMMAND_ID>`, with `</NAME SUBCOMMAND:ID>` and `</NAME SUBCOMMAND_GROUP SUBCOMMAND:ID>` available for subcommands and subcommand groups. August 09, 2022

### Session-specific Gateway Resume URLs

Starting **September 12, 2022**, apps that do not use the new `resume_gateway_url` field to resume gateway sessions will be disconnected much sooner than normal. The [Ready](https://docs.discord.com/developers/events/gateway-events#ready) gateway event now includes `resume_gateway_url` for session-specific gateway reconnects. Use that value when resuming after a disconnect. Previously, `wss://gateway.discord.gg` was used for both connecting and resuming; now it should be used only for the initial connection. For now, `resume_gateway_url` always returns `wss://gateway.discord.gg` to allow migration time, but the `resume_gateway_url` value will later change to session-specific URLs. July 13, 2022
