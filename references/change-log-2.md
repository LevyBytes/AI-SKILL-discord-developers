# Change Log (part 2)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

### Server-Side Message Moderation

Discord released and documented two API endpoints for changing application-scoped moderation metadata on game direct messages and lobby messages. Together with the [previously announced](https://docs.discord.com/developers/change-log#discord-social-sdk-release-1813395) [`MessageHandle::ModerationMetadata`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1MessageHandle.html#afb9ae126f6b1f0de006f2be8e3688205), plus game direct message and lobby [Webhook Events](https://docs.discord.com/developers/events/webhook-events#event-types), these endpoints support server-side moderation for lobby and game direct message traffic. A game backend can review messages, attach moderation metadata, and have the Discord Social SDK stream those results to clients in real time so in-game rendering can react to moderation decisions. The updated [How To Integrate Moderation](https://docs.discord.com/developers/discord-social-sdk/how-to/integrate-moderation) guide walks through the complete server-side moderation path. February 19, 2026Discord Social SDK

### Discord Social SDK Release 1.8.14026

Discord Social SDK has a new release with these updates:

### Stabilization

- Fixed an intermittent crash that could happen when linking a Discord account to an Application during a Voice Call
 February 18, 2026Discord Social SDKDocs

### Publisher Level Account Linking

Discord published documentation for **Publisher Level Account Linking**, which lets game publishers that operate multiple titles provide one authorization flow across their portfolio. **Publisher Level Account Linking** requires Discord approval. To ask about access, work with your Discord account representative or [contact us](https://discord.com/developers/contact-us). Players authenticate once through a publisher application rather than repeating the process per game, giving players a simpler sign-in path and publishers centralized account management. The guide covers these capabilities:
- Creating parent-child application hierarchies that connect publisher applications with game applications
- Trading parent access tokens for child access tokens
- Using child tokens with the Discord Social SDK
- Understanding token lifecycles and revocation
Start with the [Publisher Level Account Linking guide](https://docs.discord.com/developers/discord-social-sdk/development-guides/publisher-level-account-linking) to learn the setup. February 12, 2026Discord Social SDK

### Discord Social SDK Release 1.8.13870

Discord Social SDK has a new release with these updates:

### Playstation 5

- Added OS 12.000 support

### iOS

- Added code stripping for iOS builds to reduce size
 February 12, 2026HTTP API

### Community Invite Guide

Discord published a guide for the new [invite endpoints](https://docs.discord.com/developers/change-log#new-invite-endpoints) and related features. It shows how to create role-granting invites through the UI and API, and how to limit invite access with `target_users` through the API. The guide focuses on community-server use cases and explains how to create and send invites that assign roles or restrict acceptance to selected users. [Check out the new guide here!](https://docs.discord.com/developers/tutorials/using-community-invites) February 12, 2026InteractionsComponents

### Radio Groups, Checkbox Groups, and Checkboxes in Modals

Discord introduced three modal components: **Radio Groups**, **Checkbox Groups**, and **Checkboxes**. They add modal inputs for single-choice answers, multi-option selection, and yes/no toggles.

#### The New Components

- [**Radio Group**](https://docs.discord.com/developers/components/reference#radio-group): Presents a list where the user chooses exactly one option. The `required` field can make the selection optional.
- [**Checkbox Group**](https://docs.discord.com/developers/components/reference#checkbox-group): Lets users select multiple options while `min_values` and `max_values` define the allowed range.
- [**Checkbox**](https://docs.discord.com/developers/components/reference#checkbox): Provides a boolean yes/no control.
All three components must be nested inside a [**Label**](https://docs.discord.com/developers/components/reference#label) component and can be used only in modals.

#### Developer Resources

- [Using Modal Components](https://docs.discord.com/developers/components/using-modal-components) - Guide for creating modals
- [Component Reference](https://docs.discord.com/developers/components/reference) - Full documentation for available components
 February 10, 2026Docs

### Next Generation Docs Project

Discord launched the Next Gen Docs project, an effort to make its developer documentation easier to navigate, more complete, and more useful for developers.

### Our Goal

Discord API has grown from its bot-platform roots into a broader developer platform that now includes:
- **Discord Apps and Bots** - The base of the ecosystem, adding custom apps, commands, integrations, and new Discord capabilities
- **Discord Activities** - Real-time games and social experiences users can open inside Discord
- **Social SDK for Games** - Discord-backed voice, chat, rich presence, and social graph features for games
Discord wants the documentation to keep pace with that platform growth. The project is meant to improve how developers learn and build on Discord in 2026 and later, not merely refresh existing pages.

### Our Next Gen Docs Journey

**Progress so far:**
- Moved the docs to [Mintlify](https://mintlify.com) for a modern documentation site
- Kept the existing content and familiar design during migration, with plans to refine both over time
- Shifted effort away from maintaining a custom docs platform and toward content quality and developer experience
- Gained new capabilities:
  - **PR Previews** - Preview docs changes locally and in pull requests before publication
  - **More Components and Docs Features** - Access to Mintlify documentation components, with more capabilities expected over time
  - **AI features & improved search** - Send the docs to an LLM of your choice or access them through MCP, with more details coming soon

The migration is complete. It required a full repository reorganization and many formatting changes, so existing PRs may face difficult merge conflicts. Discord plans to spend the next few weeks reducing the PR backlog. The team reviewed the migrated docs for accuracy; report anything incorrect by opening an issue or sending feedback [here](https://github.com/discord/discord-api-docs/discussions/categories/next-gen-docs-feedback). Feedback will be monitored so problems can be fixed quickly.

### Why This Matters

Good documentation needs accurate information and presentation that helps developers make progress. Over the coming months, Discord plans documentation changes that:
- Helps new developers reach a working result faster
- Keeps guidance useful as projects become more complex
- Tracks platform changes as Discord evolves quickly
- Supports developers from beginner through expert levels

### Contributing to Our Vision

Discord wants the developer community involved in the documentation changes and welcomes:
- **Feedback** about what works and what needs improvement
- **Content contributions** through our existing PR process
- **Bug reports** when documentation and reality do not match
Discord created a Github Discussion topic for [Next Gen Docs Feedback](https://github.com/discord/discord-api-docs/discussions/categories/next-gen-docs-feedback). Read the [Contributing Guidelines](https://github.com/discord/discord-api-docs/blob/main/CONTRIBUTING.md) to learn how to participate. February 05, 2026HTTP APIBreaking Change

### Community Invites Update

Discord changed the community invite endpoints in two ways because of a security concern:
- [Get Target Users](https://docs.discord.com/developers/resources/invite#get-target-users) now returns a standardized CSV file whose header is `user_id`, followed by one user ID per line. If your implementation depended on the submitted header or ignored the returned header, update it to expect only `user_id` as the CSV header.
- [Get Channel Invites](https://docs.discord.com/developers/resources/channel#get-channel-invites) now returns a partial [roles](https://docs.discord.com/developers/topics/permissions#role-object) object instead of the complete role object. This breaking change means the response includes only `id`, `name`, `position`, `color`, `colors`, `icon`, and `unicode_emoji`.
 January 21, 2026ActivitiesEmbedded App SDK

### Relationships.read scope

Discord opened the `relationships.read` scope for Activities under the [Social SDK terms](https://support-dev.discord.com/hc/en-us/articles/30225844245271-Discord-Social-SDK-Terms). To use the scope, accept the Social SDK terms for the app in [Social SDK settings](https://discord.com/developers/applications/select/social-sdk/getting-started); Discord approval is no longer required. With this scope, `getRelationships()` in the embedded app SDK returns a player's relationships. The Embedded App SDK is available on [npm](https://www.npmjs.com/package/@discord/embedded-app-sdk) and [GitHub](https://github.com/discord/embedded-app-sdk). Use the [installation guide and reference](https://docs.discord.com/developers/developer-tools/embedded-app-sdk) to get started. January 13, 2026HTTP API

### New Invite Endpoints

Discord added endpoints and behavior that let invites grant roles and restrict acceptance to specified users. Communities can use them for finer access control or to assign special roles when members join. [Create Channel Invite](https://docs.discord.com/developers/resources/channel#create-channel-invite) now accepts `target_users_file` and `role_ids` parameters.
- `target_users_file`: CSV file containing the user IDs allowed to accept the invite
- `role_ids`: Role IDs for roles assigned when users accept the invite
Invite endpoints:
- [Get Target Users](https://docs.discord.com/developers/resources/invite#get-target-users): Retrieves users who may view and accept an invite
- [Update Target Users](https://docs.discord.com/developers/resources/invite#update-target-users): Changes the users who may view and accept an invite
- [Get Target Users Job Status](https://docs.discord.com/developers/resources/invite#get-target-users-job-status): Reports the processing job status for target users uploaded from a file
 December 17, 2025Discord Social SDK

### Discord Social SDK Release 1.8.13395

Discord Social SDK has a new release with these updates:

### Moderation Metadata Support

- [Coming Soon] Messages will add [`MessageHandle::ModerationMetadata`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1MessageHandle.html#afb9ae126f6b1f0de006f2be8e3688205), custom string key-value pairs for carrying moderation information between a game backend and client.
  - This SDK version contains support for that feature, but the feature becomes usable only after a later Discord API change.

### WebSocket Reporting

- SDK logs now include an HTTP status code when a WebSocket connection fails.

### PlayStation 5

- Improved timing-metric accuracy for monitoring connection and voice-call health.
 December 16, 2025Discord Social SDK

### Discord Social SDK Release 1.7.13357

Discord Social SDK has a new release with these updates:
- Fixed a Steam Deck crash that occurred when calling RegisterLaunchCommand
- Fixed an iOS crash when Deeplink Authentication fails.
 December 16, 2025Discord Social SDK

### New Social SDK Guide: Account Linking from Discord

Discord added a guide for enabling account-linking entry points inside the Discord client. With Social SDK 1.6+, Discord can show 'Link your account' prompts and buttons in the client, helping players connect game accounts and improving linking rates and social engagement. The guide covers two implementation paths:
- **Connected Game Flow (recommended)**: Uses new callback methods so clicks on Discord entry points start account linking directly in your game
- **Web Flow**: Sends players through a web OAuth flow, currently available only to select partners
[Read the full guide](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-from-discord) December 10, 2025Discord Social SDK

### Discord Social SDK Release 1.6.13305

Discord Social SDK has a new release with these updates:
- Fixed a Steam Deck crash that occurred when calling RegisterLaunchCommand
- Fixed an iOS crash when Deeplink Authentication fails.
 December 09, 2025HTTP API

### Get Guild Role Member Counts Endpoint

Apps can use the [Get Guild Role Member Counts](https://docs.discord.com/developers/resources/guild#get-guild-role-member-counts) endpoint to retrieve how many members have each role. December 04, 2025Discord Social SDK

### Discord Social SDK Release 1.7.13152

Discord Social SDK has a new release with these updates:

### macOS

- Added macOS 10.15 build support. Intel Mac builds now require at least 10.15, while Apple Silicon builds require at least 11.0.
 November 24, 2025Breaking Change

### Permission Changes Going into Effect February 2026 for PIN_MESSAGES, BYPASS_SLOWMODE, CREATE_GUILD_EXPRESSIONS, and CREATE_EVENTS

Discord announced permission changes that take effect in February 2026. The permissions below were introduced without breaking existing behavior, but the breaking portion starts on February 23, 2026, when older permissions stop granting these abilities. The changes split several actions away from broader permissions so bots and users can be controlled more precisely.

### What’s Changing?

**1. Pin Messages Permission**
- The `PIN_MESSAGES` permission (`1 << 51`) was separated from `MANAGE_MESSAGES` on August 20, 2025.
- See the [change log](https://docs.discord.com/developers/change-log#pin-permission-split).
- Beginning February 23, 2026, pinning messages requires `PIN_MESSAGES`; `MANAGE_MESSAGES` by itself will not be enough.
**2. Bypass Slowmode Permission**
- The `BYPASS_SLOWMODE` permission (`1 << 52`) is being separated from `MANAGE_MESSAGES`, `MANAGE_CHANNEL`, and `MANAGE_THREADS`.
- See the [change log](https://docs.discord.com/developers/change-log#new-bypassslowmode-permission-permission-split).
- Note: This mostly impacts users because bots are not subject to slowmode restrictions.
- Beginning February 23, 2026, users need `BYPASS_SLOWMODE` to be exempt from slowmode restrictions.
**3. Create Expressions Permission**
- The `CREATE_GUILD_EXPRESSIONS` permission (`1 << 43`) was created in July 2023 and separated from `MANAGE_GUILD_EXPRESSIONS` for users in December 2023. See the [change log](https://docs.discord.com/developers/change-log#clarification-on-permission-splits-for-expressions-and-events).
- Bots now have access to the `CREATE_GUILD_EXPRESSIONS` permission.
- See the [change log](https://docs.discord.com/developers/change-log#guild-expressions-and-events-permissions-now-available-to-developers).
- Beginning February 23, 2026, bots need `CREATE_GUILD_EXPRESSIONS` to create custom emoji and stickers; `MANAGE_GUILD_EXPRESSIONS` by itself will not be enough.
**4. Create Events Permission**
- The `CREATE_EVENTS` permission (`1 << 44`) was created in July 2023 and separated from `MANAGE_EVENTS` for users in December 2023. See the [change log](https://docs.discord.com/developers/change-log#clarification-on-permission-splits-for-expressions-and-events).
- Bots now have access to the `CREATE_EVENTS` permission.
- See the [change log](https://docs.discord.com/developers/change-log#guild-expressions-and-events-permissions-now-available-to-developers).
- Beginning February 23, 2026, bots need `CREATE_EVENTS` to create scheduled events; `MANAGE_EVENTS` by itself will not be enough.

### What Do You Need to Do?

Review and update bot permission requests before February 23, 2026 if your bot does any of the following:
- **Message pinning:** Request `Pin Messages`
- **Custom emoji or sticker creation:** Request `Create Expressions`
- **Scheduled event creation:** Request `Create Events`
The changes give server administrators tighter control over bot and user capabilities, improving security and permission management. November 24, 2025

### New BYPASS_SLOWMODE Permission & Permission Split

Discord introduced `BYPASS_SLOWMODE`, a permission that lets selected roles or users bypass channel slowmode restrictions.
- The `BYPASS_SLOWMODE` permission (`1 << 52`) is being separated from `MANAGE_MESSAGES`, `MANAGE_CHANNEL`, and `MANAGE_THREADS`.
- Note: This mostly impacts users because bots are not subject to slowmode restrictions.
- Beginning February 23, 2026, users need `BYPASS_SLOWMODE` to avoid slowmode restrictions.
 November 20, 2025Breaking Change

### Guild Expressions and Events Permissions now available to developers

Discord introduced permission splits in 2023 for guild expressions, which include custom emoji and stickers, and for scheduled events. Those splits gave server administrators more precise control over who can create community content. See the [change log](https://docs.discord.com/developers/change-log#clarification-on-permission-splits-for-expressions-and-events). These permissions are now available to bot developers and will be required for selected actions starting **February 23, 2026**. **Create Guild Expressions Permission**
- The `CREATE_GUILD_EXPRESSIONS` permission (`1 << 43`) was created in July 2023 and separated from `MANAGE_GUILD_EXPRESSIONS` for users in December 2023.
- Bots now have access to the `CREATE_GUILD_EXPRESSIONS` permission.
- Beginning February 23, 2026, bots need `CREATE_GUILD_EXPRESSIONS` to create custom emoji and stickers; `MANAGE_GUILD_EXPRESSIONS` by itself will not be enough.
**Create Events Permission**
- The `CREATE_EVENTS` permission (`1 << 44`) was created in July 2023 and separated from `MANAGE_EVENTS` for users in December 2023.
- Bots now have access to the `CREATE_EVENTS` permission.
- Beginning February 23, 2026, bots need `CREATE_EVENTS` to create scheduled events; `MANAGE_EVENTS` by itself will not be enough.
 November 20, 2025Discord Social SDK

### Discord Social SDK Release 1.7.13024

Discord Social SDK has a new release with these updates:

### Windows

- Fixed default audio device selection so it prefers the system default audio device instead of the default communications device again.

### Xbox

- Added build support for GDK 240605.
 November 12, 2025Discord Social SDK

### Discord Social SDK Release 1.7

Discord Social SDK has a new release with these updates:

### Authentication

- Added Sign in with Apple and PlayStation Network support as external identity providers.

### Linked Channels

- [`Client::GetGuildChannels`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#adba1e5a83c219a9c4f6dab1657778017) results now match the channel order shown in the Discord client. [`GuildChannel`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1GuildChannel.html#ac83eedfa1d9434dfa245ae39bb3a0915) also gained properties for type, position, and parent category ID.

### PlayStation 5

- Added Low Energy Mode support. SDK thread affinity can be controlled explicitly through the `cpuAffinityMask` property on [`ClientCreateOptions`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1ClientCreateOptions.html#ae655ad66ba64f443496c158307cc77b4).
- Fixed a crash path involving destruction and recreation of [`Client`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a91716140c699d8ef0bdf6bfd7ee0ae13).

### Windows

- Fixed a Windows issue that disabled local crash dumps in processes loading the Social SDK.

### Voice

- On Windows, the Social SDK now uses Discord's latest audio subsystem, aligning it more closely with the desktop client. This improves device compatibility and fixes long-running problems such as Bluetooth devices switching profiles on startup.
- Desktop clients now use WebRTC m130 internally, upgraded from m116.
- Improved and fixed bugs in the DAVE E2EE protocol implementation.
 November 05, 2025Discord Social SDK

### Discord Social SDK Release 1.6.12767

Discord Social SDK has a new release with this update: fixed a Playstation crash that could occur when creating the Discord client, destroying it, and creating another client. October 16, 2025Discord Social SDK

### Discord Social SDK Release 1.5.12211

Discord Social SDK has a new release with this update:

### Bug Fix

- Fixed an iOS crash when calling [`Client::GetUserGuilds`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#aac1ec02df6074ed9213ce230e6a42fe1). The crash first appeared in 1.5.10839
 October 15, 2025Discord Social SDK

### Discord Social SDK Release 1.6.12170

Discord Social SDK has a new release with these updates:

### New Features

This small v1.6 update adds several partner-requested features that fit into the release.
- Game invites shown in Discord can now display custom banner art, supplied in one of two ways:
  - Provide an image URL or `asset key` through `activity.assets.inviteCoverImage` when calling `Client::UpdateRichPresence`. This also lets each invite use its own image.
  - Upload cover image art in the `Rich Presence` tab of the Developer Portal for your Application. This image is used as the fallback when `activity.assets.inviteCoverImage` is not specified.

- The displayed name for your Application in Discord Rich Presence can now be customized by setting `activity.name` when calling [`Client::UpdateRichPresence`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#af0a85e30f2b3d8a0b502fd23744ee58e)
- Added Microsoft Xbox GDK version 2025.04 support
- Fixed an iOS crash when calling [`Client::GetUserGuilds`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#aac1ec02df6074ed9213ce230e6a42fe1)
 October 15, 2025InteractionsComponents

### Introducing the File Upload component in Modals
