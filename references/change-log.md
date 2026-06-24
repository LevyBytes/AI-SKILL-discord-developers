# Change Log (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

FiltersJune 16, 2026HTTP API

### Documentation Fix: Subscription Status Values

The documentation had the integer values for `INACTIVE` and `ENDING` subscription statuses reversed. The correct mapping is `INACTIVE` = `1` and `ENDING` = `2`, matching the [Subscription Statuses](https://docs.discord.com/developers/resources/subscription#subscription-statuses) reference. Both the reference and the [Implementing App Subscriptions](https://docs.discord.com/developers/monetization/implementing-app-subscriptions) guide, which previously showed the swapped lifecycle values, have been corrected. The API has always returned these values; only the documentation was wrong. If your app branches on numeric `status`, verify that `1` is handled as `INACTIVE` and `2` as `ENDING`. The issue was found while comparing the documentation with [our OpenAPI spec](https://github.com/discord/discord-api-spec) in [this commit](https://github.com/discord/discord-api-spec/commit/cdbd53a2ac5144536bfee2a34e6d0de13e08eec0). June 10, 2026HTTP APIGateway

### Changes to Privileged Intent Access for Discord Apps

(Figure: Privileged Intents Requirements) Discord is [announcing changes to how Discord Apps access Privileged Intent](https://support-dev.discord.com/hc/articles/40281523410967), including a user-based review threshold and a yearly process for reapplying to keep access.

>  If your app is accessible to fewer than 10,000 users, these changes do not impact your app.

### What’s Changing

#### 1. The review threshold is now based on user count, not server count

Previously, apps in fewer than 100 servers could enable Privileged Intents in the Developer Portal, while apps in 100+ servers had to apply. The threshold now uses the number of users your app can access across every server it is in. Apps below 10,000 reachable users may still turn on Privileged Intents in the Developer Portal. **At 10,000 reachable users, Privileged Intent access requires an application review**.

#### 2. Apps must now reapply annually for continued access

Apps that already received Privileged Intent access through review must reapply once per year in the Developer Portal to confirm continued access.

#### 3. Apps can keep growing during Privileged Intent review

With the user-based threshold, apps may continue joining servers and reaching more users while their Privileged Intent submission is being reviewed.

#### 4. App Verification and Privileged Intent review are now separate

[App Verification](https://support-dev.discord.com/hc/en-us/articles/23926564536471-How-Do-I-Get-My-App-Verified) and Privileged Intent review used to be handled together. This change separates App Verification from the Privileged Intent review process. (Figure: Privileged Intents Requirements)

### What This Means For Your App

Read the [Privileged Intents Change announcement article](https://support-dev.discord.com/hc/articles/40281523410967) in the developer help center for details on how the change affects Discord Apps.

### Updated Documentation & Resources

- Updated [Privileged Intents documentation](https://docs.discord.com/developers/events/gateway#privileged-intents) with the new threshold and review process.
- Added the [Getting Started with Privileged Intents Review](https://docs.discord.com/developers/gateway/getting-started-with-privileged-intent-review) guide.
- Added the [You Might Not Need a Privileged Intent](https://docs.discord.com/developers/gateway/you-might-not-need-a-privileged-intent) guide to help determine whether a privileged intent is needed or whether another approach fits the use case.
 June 3, 2026Discord Social SDK

### Discord Social SDK Release 1.9.16441

A Discord Social SDK release is available with these updates:

### Playstation 5

- Fixed a WebSocket callback use-after-free crash on Playstation 5.
 June 3, 2026Discord Social SDKHTTP API

### New Lobby HTTP API Endpoints

The Discord [Lobby resource](https://docs.discord.com/developers/resources/lobby) reference now includes several public endpoints:
- **[Create or Join Lobby](https://docs.discord.com/developers/resources/lobby#create-or-join-lobby)** (`PUT /lobbies`) - create a lobby for your application or join an existing lobby by `secret` in one request.
- **[Send Lobby Message](https://docs.discord.com/developers/resources/lobby#send-lobby-message)** (`POST /lobbies/{lobby.id}/messages`) - send a lobby message on behalf of an authenticated member.
- **[Get Lobby Messages](https://docs.discord.com/developers/resources/lobby#get-lobby-messages)** (`GET /lobbies/{lobby.id}/messages`) - retrieve up to 200 recent messages from a lobby where the caller is a member.
- **[Create Lobby Channel Invite for Self](https://docs.discord.com/developers/resources/lobby#create-lobby-channel-invite-for-self)** and **[for User](https://docs.discord.com/developers/resources/lobby#create-lobby-channel-invite-for-user)** - create a single-use, one-hour guild invite for a lobby's linked channel.
 May 26, 2026Discord Social SDK

### Discord Social SDK Release Cadence & Support

A new [Release Cadence & Support](https://docs.discord.com/developers/discord-social-sdk/core-concepts/release-cadence-and-support) page for the Discord Social SDK is available. It covers:
- The new three-per-year minor release cadence in April, July, and November.
- A support window for the five most recent minor versions, with hotfix patches for bug fixes and security fixes on older supported minor releases.
- How new Xbox and PlayStation OS SDK supports are delivered only in new Social SDK releases; each release bundles roughly the last 18 months of console OS SDKs.
- The end-of-life process for older minor releases.

### What this means for you

- **Over a year and a half of support per minor release.** With three Social SDK minor releases per year and a 5-version support window, every minor release you ship gets more than 18 months of patch coverage.
- **A predictable release calendar.** Releases arrive in April, July, and November, making Social SDK upgrades and console certification windows easier to plan.
- **More bandwidth for new features and stability.** A defined support window focuses engineering effort on current minor releases and on keeping supported versions robust and reliable.

### What changes with 1.10

These changes begin with the 1.10 release, shipping in **July 2026**:
- The supported version window will include versions **1.6 through 1.10**. Releases at **1.5 and earlier** move to end-of-life status and stop receiving patch updates.
- **Console OS SDK additions are delivered only through new Social SDK releases**. Older Social SDK versions will not receive new Xbox or PlayStation support through patches. Each Social SDK release bundles roughly the last 18 months of console OS SDK versions.
**Already-shipped Social SDK releases (1.6-1.9) are unchanged** - they retain the console OS SDK support they originally shipped with. The new model starts at 1.10. This is being shared before the 1.10 release so developers on older versions have time to upgrade. May 22, 2026Discord Social SDK

### Account Linking Failure Modes

Social SDK documentation now explains what happens when the Discord-to-game account link is broken, whether through a Discord account ban, explicit unmerge, or OAuth2 token revocation. Highlights:
- **[Ban-Driven Unmerge](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#ban-driven-unmerge)** explains the lifecycle after a Discord account ban. Discord deletes the tokens immediately, creates a cross-platform-restricted provisional account tied to the original external identity, and returns [`530017`](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#merge-request-failures) for merge attempts until the ban ends, or permanently for permanent bans.
- **[Out-of-Band Revocation](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord#out-of-band-revocation)** covers the auth-side signals: the `APPLICATION_DEAUTHORIZED` webhook and `invalid_grant` during refresh. These appear when a user removes your app from Discord or when that Discord account is banned.
- **[When Refresh Fails](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord#when-refresh-fails)** documents the `/oauth2/token` response returned after the underlying token is deleted and recommends treating `invalid_grant` like the webhook.
 May 12, 2026Discord Social SDK

### Discord Social SDK Release 1.9.15780

A Discord Social SDK release is available with these updates:

### Xbox

- Added Xbox GDK 260400 support

### Noise Suppression

- Integrated Krisp noise suppression on Android, iOS, macOS, and Windows.
 May 7, 2026HTTP API

### New `flags_new` Field on Application Object

The [Application object](https://docs.discord.com/developers/resources/application#application-object) now has a `flags_new` field, a string-serialized integer containing the full set of [application flag bits](https://docs.discord.com/developers/resources/application#application-object-application-flags), including new flag bits beyond bit 30. The existing `flags` field remains serialized as a 32-bit integer. Existing integrations that read `flags` for bits at or below `2^30` are unaffected, and request payloads containing flag values should continue to use `flags`. This follows the [`permissions_new` / `allow_new` / `deny_new`](https://docs.discord.com/developers/topics/permissions) pattern previously added for permission bitfields. May 5, 2026HTTP API

### User Premium Type Field Now Requires A Scope

The `premium_type` field on the [User object](https://docs.discord.com/developers/resources/user#user-object-user-structure) returns `0` for applications without approval for the `identify.premium` scope. To receive Nitro subscription level, request `identify.premium` during OAuth authorization. Without that scope, the field returns `0` regardless of the user's real subscription status. The `identify.premium` scope is available only to approved partners. April 24, 2026Discord Social SDKHTTP API

### Voice Muting Based on Player Blocks

A new how-to guide explains how to implement bidirectional voice muting in lobby voice calls based on block relationships. It describes two ways to populate per-member mute lists: client-side using client-accessible block signals such as Discord relationships or in-game block lists, and server-side through lobby member metadata set at lobby creation. It also gives one shared client-side approach that applies mutes both ways. The [`POST /lobbies/{lobby.id}/members/bulk`](https://docs.discord.com/developers/resources/lobby#bulk-update-lobby-members) endpoint is now documented too; it can add, update, or remove up to 25 lobby members in one request. [Voice Muting Based on Player Blocks](https://docs.discord.com/developers/discord-social-sdk/how-to/voice-muting-for-blocked-players) April 23, 2026Discord Social SDK

### Discord Social SDK Release 1.9.15332

A Discord Social SDK release is available with these updates:

### Playstation 5

- Added OS 13.000 support

### Activity Invites

- Accepting an activity invite over RPC now uses the invite's `application_id` when present. This fixes cases where sibling-application invites were sometimes attributed to the wrong app, and only affects teams using Publisher Level Account Linking for one auth flow across all games in their portfolio.
 April 23, 2026Discord Social SDK

### Proximity Voice Chat Guide for Game Developers

A new guide explains how to add proximity voice chat to a multiplayer game with the Discord Social SDK and Unity. It walks through the general architecture, and the concepts can be adapted to other games or engines. [Add proximity voice chat to your game](https://docs.discord.com/developers/game-development/how-to-add-proximity-voice-chat-to-your-game) April 22, 2026Discord Social SDK

### Security Updates for Provisional Account OIDC Integrations

Discord has strengthened validation for OIDC issuer URLs and OIDC configuration fetching for [provisional accounts](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts). The checks now include stricter host validation and reject HTTP redirects while loading configuration or JWKS data. **Existing provisional account OIDC integrations are not affected** and keep working without changes. If you edit OIDC configuration in the [Developer Portal](https://discord.com/developers/applications/select/social-sdk/external-auth-providers), the updated configuration is validated against the new requirements. New [OIDC Integration Requirements](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#oidc-integration-requirements) documentation lists the issuer URL constraints, discovery-document rules, supported signing algorithms, and ID token claim requirements. Review those requirements and verify the configuration in a test application before changing a production integration. April 20, 2026Documentation

### Voice Channel Status and Start Time Documentation

Voice channel status has existed for some time; the endpoint, permission, Gateway events, and audit log events are now officially documented so apps can integrate with it. The [Set Voice Channel Status](https://docs.discord.com/developers/resources/channel#set-voice-channel-status) endpoint (`PUT /channels/{channel.id}/voice-status`) lets apps set a short string, up to 500 characters, that describes what is happening in a voice channel.

### Permission

Setting a channel's status requires the `SET_VOICE_CHANNEL_STATUS` permission (`1 << 48`), now included in the [permissions](https://docs.discord.com/developers/topics/permissions) table. `MANAGE_CHANNELS` is also required when the bot user is outside that voice channel.

### Gateway Events

Voice channel status and voice session start time are temporary values and are not stored on the [channel](https://docs.discord.com/developers/resources/channel#channel-object) object. Apps read them by sending the [Request Channel Info](https://docs.discord.com/developers/events/gateway-events#request-channel-info) Gateway command, opcode `43`. Discord responds with a [Channel Info](https://docs.discord.com/developers/events/gateway-events#channel-info) event for guild channels that carries requested fields such as `status` and `voice_start_time`. Field changes are reported through two receive events:
- [Voice Channel Status Update](https://docs.discord.com/developers/events/gateway-events#voice-channel-status-update) - sent when a channel's status changes.
- [Voice Channel Start Time Update](https://docs.discord.com/developers/events/gateway-events#voice-channel-start-time-update) - sent when a voice session's start time changes.

### Audit Log

Two [audit log events](https://docs.discord.com/developers/resources/audit-log#audit-log-events) record status changes: `VOICE_CHANNEL_STATUS_UPDATE` (`192`) and `VOICE_CHANNEL_STATUS_DELETE` (`193`). On `VOICE_CHANNEL_STATUS_UPDATE` entries, the new status is carried in a `status` field inside [Optional Audit Entry Info](https://docs.discord.com/developers/resources/audit-log#audit-log-entry-object-optional-audit-entry-info). April 16, 2026Discord Social SDK

### Deprecation: OAuth2 Refresh Token Grant for OIDC Provisional Accounts

For [OIDC server-side authentication](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#server-authentication-with-external-credentials-exchange) with provisional accounts, a `refresh_token` is returned, but using it through the OAuth2 `refresh_token` grant is deprecated. This aligns OIDC with other provisional account authentication methods: when a token expires, re-authenticate with a fresh provider token and pass the new access token to [`Client::UpdateToken`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a606b32cef7796f7fb91c2497bc31afc4). See [Refreshing Provisional Account Tokens](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#refreshing-provisional-account-tokens) for details. April 14, 2026HTTP API

### Message Forwarding Requires Message Content Access

Starting today, applications must be able to read a message's content before forwarding it. When an app attempts to forward unreadable content, Discord now returns `160014` and explains that the content cannot be forwarded. Most applications are unaffected; recently affected developers were notified directly by system DM. April 6, 2026Discord Social SDK

### Discord Social SDK Release 1.8.14985

A Discord Social SDK release is available with these updates:

### Xbox

- Added Xbox GDK 251002 support
 April 3, 2026RPC

### RPC over IPC Documentation

Documentation is now available for Discord's RPC over IPC (Inter-Process Communication) transport. IPC lets native applications communicate with a local Discord client using high-performance local transport without network overhead. The [RPC documentation](https://docs.discord.com/developers/topics/rpc) covers:
- IPC socket paths for Windows and Linux/macOS.
- Handshake flow and opcodes
- IPC-specific events: `READY`, `ERROR`, and `GUILD_STATUS`.
For games integrating Discord social features, Discord recommends the [Discord Social SDK](https://docs.discord.com/developers/discord-social-sdk/overview) instead of direct RPC communication. March 27, 2026Discord Social SDK

### Discord Social SDK Release 1.8.14856

A Discord Social SDK release is available with these updates:

### iOS Unity & Unreal

- Unity and Unreal packages now include Debug plugins with Info and Verbose logging support.

### Party Privacy

- Fixed a bug where party privacy setting changes did not propagate when calling [`Client::UpdateRichPresence`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#af0a85e30f2b3d8a0b502fd23744ee58e), so changing a party from private to public failed to update the join button in Discord.
 March 19, 2026HTTP API

### Search Guild Messages

Documentation for the [Search Guild Messages](https://docs.discord.com/developers/resources/message#search-guild-messages) endpoint has been added. Bots can use it to search messages within a guild with filters such as content, author, channel, mentions, attachments, embeds, and more. Results may be sorted by timestamp or relevance. The endpoint requires `READ_MESSAGE_HISTORY` permission and the `MESSAGE_CONTENT` [Privileged Intent](https://docs.discord.com/developers/events/gateway#privileged-intents). Error code `110000` was added to the [error codes](https://docs.discord.com/developers/topics/opcodes-and-status-codes#json) table for the index-not-ready response returned when a guild has not been indexed for search yet. March 15, 2026Discord Social SDK

### Discord Social SDK Release 1.8.14587

A Discord Social SDK release is available with these updates:

### Debug Support

- Added Debug library support for the following:
  - Unity plugin & sample
  - Unreal plugin & sample
  - Windows, Mac, Linux libraries
  - Android library
  - PS5 11.000 & 12.000 library
  - PS4 12.000 & 12.500 library
  - Scarlett & Xbox One 250400 GDK library

 March 9, 2026Docs

### Developer Portal & Docs Refresh

The Developer Portal and Developer Docs have been refreshed with better navigation, improved usability, and a redesigned theme. More developer experience updates are planned. Current changes include:

### Developer Portal

- **New [dev portal home page](https://discord.com/developers/home)** with quick access to apps, docs, and new developer videos.
- **New theme** with refreshed navigation and updated light/dark support.
- **New Localization support!** It uses the same language configured in the Discord client.

### Developer Docs

- **Reorganized structure** with new tabs for feature areas and new sidebars for easier navigation.
- **New guides and overviews** covering broader platform features and common use cases, with additional material planned.
- **AI & MCP support:** Docs now support AI development tools through Copy-as-Markdown, an LLMs.txt file, and an MCP server at `https://docs.discord.com/mcp`
- **Existing docs URLs** remain stable and were kept during the reorganization. Some titles and content changed or moved for clarity.

### What’s Next

More portal and docs improvements are in progress. Share issues or feedback in the [Next Gen Docs discussion thread](https://github.com/discord/discord-api-docs/discussions/categories/next-gen-docs-feedback). March 5, 2026Discord Social SDK

### Discord Social SDK Release 1.8.14437

A Discord Social SDK release is available with these updates:

### Security

- Bearer tokens are filtered from all log levels.
- Release builds emit log statements only at Error or Warning severity.
- Debug builds still log Info and Verbose severity levels.
- Added a DEBUG BUILD Warning at the top of the debug log stating that it should not be used in production.
 March 5, 2026Discord Social SDK

### Discord Social SDK Release 1.7.14433

A Discord Social SDK release is available with these updates:

### Security

- Bearer tokens are filtered from all log levels.
- Release builds emit log statements only at Error or Warning severity.
- Debug builds still log Info and Verbose severity levels.
- Added a DEBUG BUILD Warning at the top of the debug log stating that it should not be used in production.
 March 5, 2026Discord Social SDK

### Discord Social SDK Release 1.6.14448

A Discord Social SDK release is available with these updates:

### Security

- Bearer tokens are filtered from all log levels.
- Release builds emit log statements only at Error or Warning severity.
- Debug builds still log Info and Verbose severity levels.
- Added a DEBUG BUILD Warning at the top of the debug log stating that it should not be used in production.
 March 3, 2026Interactions

### Updates to Context Menu Commands

The UI for [User Commands](https://docs.discord.com/developers/interactions/application-commands#user-commands) and [Message Commands](https://docs.discord.com/developers/interactions/application-commands#message-commands) has been redesigned to make commands easier to find and to allow higher limits.

#### Refreshed UI

The UI update responds to feedback and removes the need to scroll through a long command list:
- Commands are grouped by application, adding hierarchy.
- Frequently used commands are promoted to the top for faster access.
- A new search bar helps users find the exact command they need.

#### A Higher Limit

After feedback that the 5 command limit was too restrictive, the new UI raises the limit to 15 per type, giving apps more room.

#### Developer Resources

See [Application Commands](https://docs.discord.com/developers/interactions/application-commands) for details on every command type. February 20, 2026Discord Social SDKHTTP APIDocs
