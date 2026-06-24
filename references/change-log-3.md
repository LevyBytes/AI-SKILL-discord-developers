# Change Log (part 3)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Modals can now collect more than text. The new [**File Upload**](https://docs.discord.com/developers/components/reference#file-upload) component lets you set minimum and maximum accepted file counts between 0 and 10, and choose whether uploading files within that range is required before submission. All file types are accepted, and the maximum file size follows the user's upload limit for that channel.

#### The New Component:

- [**File Upload**](https://docs.discord.com/developers/components/reference#file-upload)

#### Developer Resources

- [Using Modal Components](https://docs.discord.com/developers/components/using-modal-components) - Learn how to create a modal.
- See the [Component Reference](https://docs.discord.com/developers/components/reference) for details on every available component.
 September 26, 2025Discord Social SDK

### Discord Social SDK Release 1.6

A Discord Social SDK release is available with these updates:

### ✨ New Features

- [`Client::GetGuildChannels`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#adba1e5a83c219a9c4f6dab1657778017) now returns channels sorted by their `position` field, matching the order used in the Discord client.
- Messages received through the Social SDK no longer create Discord client notifications for the same user, avoiding duplicate notifications on one machine.
**This release adds support features for upcoming Discord experiments that improve game integration with user profiles and authentication:**
- **Game Profile Integration**: New capabilities show game data on Discord user profiles. This includes `UserApplicationProfile` support and two Users class methods, `Client::GetUserApplicationProfiles` and `Client::GetUserApplicationProfilesNoLock`, which retrieve game identity data from external authentication providers.
- **In-Discord Authentication Flow**: Users can begin account linking from Discord instead of starting inside your game. Added [`Client::RegisterAuthorizeRequestCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a5f34b873e127a446c9ab549e4588ccd7) and [`Client::RemoveAuthorizeRequestCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ab7e48864b0cedf3e8572a228ca401f2a) to handle authentication requests launched from Discord entry points. These functions support upcoming Discord client experiments that will roll out gradually.

### ⚠️ Deprecations

This deprecation improves consistency across the SDK API surface and provides a safer implementation with fewer edge cases and less chance of accidental misuse.
- Deprecated [`Client::GetCurrentUser`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a7edea2a3dfe9ae560d5fa5ba8663b5cc) in favor of [`Client::GetCurrentUserV2`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ae52259570ba657252d91f5580636fe5d), which returns optional values rather than handles that may be invalid.

### 🚀 Performance Improvements

- [`Client::GetUserMessagesWithLimit`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a054a758a76c5873b38a4d79651a5f26c) now uses caching so unnecessary remote API calls can be avoided when enough messages already exist in the local cache.

### 🎤 Voice Communications Fixes and Improvements

This release fixes several critical voice communications bugs and improves overall reliability, noise suppression, echo suppression, and cancellation.
- Fixed an issue where voice calls sometimes entered the `Disconnected` state instead of reconnecting correctly after network interruption.
- Extended AGC2 (Automatic Gain Control 2) support to mobile platforms.

### 🐛 General Stability and Bug Fixes

Several critical bugs could cause crashes and panics. Upgrading to 1.6 is strongly recommended to avoid them in games.
- Fixed a critical bug where activity party privacy was not set correctly, causing "ask to join" to appear instead of "join" for public parties.
- Fixed a critical memory safety issue that prevented connection objects from deallocating during timer callbacks.
- Fixed C# marshaling alignment bugs and double-free crashes.
- Improved gateway resilience by falling back to generic URLs when zonal gateways fail.
- Fixed WebSocket write-after-close errors that prevented connection issues from being handled correctly.
To learn more about the Discord Social SDK, see the [Discord Social SDK Overview](https://discord.com/developers/docs/discord-social-sdk/overview). Questions can be asked in [#social-sdk-dev-help](https://discord.com/channels/613425648685547541/1350223314307776592). September 10, 2025InteractionsComponents

### Adding More Modal Components!

Modals now support more components. All select menus, including User, Role, Mentionable, and Channel, are fully supported in modals. To use a select menu in a modal, place it inside a [Label](https://docs.discord.com/developers/components/reference#label) component. The [Text Display](https://docs.discord.com/developers/components/reference#text-display) component with markdown support is also available as a top-level modal component.

#### Components Now Supported in Modals:

- [**User Select**](https://docs.discord.com/developers/components/reference#user-select)
- [**Role Select**](https://docs.discord.com/developers/components/reference#role-select)
- [**Mentionable Select**](https://docs.discord.com/developers/components/reference#mentionable-select)
- [**Channel Select**](https://docs.discord.com/developers/components/reference#channel-select)
- [**Text Display**](https://docs.discord.com/developers/components/reference#text-display)

#### Getting Started

- [Using Modal Components](https://docs.discord.com/developers/components/using-modal-components) - Learn how to create a modal.

#### Developer Resources

See the [Component Reference](https://docs.discord.com/developers/components/reference) for details on every available component. September 10, 2025HTTP API

### Banner, avatar, and bio can be set on modify current member

Starting September 10, 2025, bots can set the `banner`, `avatar`, and `bio` fields through the [modify current member](https://docs.discord.com/developers/resources/guild#modify-current-member) route. September 02, 2025Voice

### Deprecating Non-E2EE Voice Calls

Discord began work on end-to-end encryption more than two years ago to improve user privacy and security. With DAVE now available across all platforms, Discord is close to making every call fully end-to-end encrypted.

### Developer Impact

To meet long-term privacy goals, Discord will require E2EE after **March 1st, 2026** for Discord audio and video conversations in DMs, GDMs, voice channels, and Go Live streams. After that date, clients and applications without DAVE support will not be able to join Discord calls.

### Implementing E2EE Voice

Developers using Discord voice APIs can use [the updated voice documentation](https://docs.discord.com/developers/topics/voice-connections), implementation examples in the [open-source repository](https://github.com/discord/libdave), and [the protocol whitepaper](https://github.com/discord/dave-protocol). The [Discord Developers community](https://discord.gg/discord-developers) can help with integration questions. Discord aims to make the migration smooth while delivering DAVE privacy and security benefits to users. August 28, 2025Discord Social SDK

### Discord Social SDK Release 1.5.11353

A Discord Social SDK release is available with these updates:

### Unity Plugin

- Fixed a crash, mainly on low-end Android devices with armv7 architecture, in [`Client::CreateOrJoinLobbyWithMetadata`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a5c84fa76c73cf3c0bfd68794ca5595c1).
- Fixed a crash when large metadata payloads were passed to [`Client::CreateOrJoinLobbyWithMetadata`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a5c84fa76c73cf3c0bfd68794ca5595c1).
After enabling the Discord Social SDK for your application, download the Discord Social SDK binaries from the Developer Portal. To learn more, see the [Discord Social SDK Overview](https://docs.discord.com/developers/discord-social-sdk/overview). August 25, 2025InteractionsComponents

### Introducing New Modal Components!

New modal components are now available.

#### What’s New

A new top-level [Label](https://docs.discord.com/developers/components/reference#label) component for modals provides a `label`, `description`, and support for containing a Text Input or String Select. String Selects now work in modals.
- String Selects can be used in modals when placed inside a Label component.
- Text Inputs can also be placed inside a Label component.
- When a Text Input is inside a Label component, the Text Input `label` field is disallowed in favor of the Label component's `label`.
- ActionRow + TextInput is deprecated in favor of the new Label component for better accessibility.
- The `required` field is available on String Selects; it defaults to true in modals and is ignored in messages.
- The `disabled` field on String Selects is not currently allowed in modals and returns an error if used.
[modal interaction responses](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-component-interaction-response-structures) are now documented, along with resolved objects for interactive components in each component's Examples section.

#### New Layout Component

- [**Label**](https://docs.discord.com/developers/components/reference#label) - A top-level component for adding a title and description to modal components.

#### Updates to Modal Components

- [**Text Input**](https://docs.discord.com/developers/components/reference#text-input) - Text Input can now be used inside a [Label](https://docs.discord.com/developers/components/reference#label).
- [**String Select**](https://docs.discord.com/developers/components/reference#string-select) - String Selects work in modals when placed in a [Label](https://docs.discord.com/developers/components/reference#label).

#### Getting Started

- [Using Modal Components](https://docs.discord.com/developers/components/using-modal-components) - Learn how to create a modal.

#### Developer Resources

See the [Component Reference](https://docs.discord.com/developers/components/reference) for details on every available component. August 20, 2025Breaking ChangesHTTP API

### Pin Permission Split

[Pinning](https://docs.discord.com/developers/resources/message#pin-message) and [unpinning](https://docs.discord.com/developers/resources/message#unpin-message) messages now use their own [permission](https://docs.discord.com/developers/topics/permissions#permissions-bitwise-permission-flags). `PIN_MESSAGES` has been split from `MANAGE_MESSAGES` to give more precise control over who may pin messages in a channel. The change is effective immediately for both users and apps. Backwards compatibility remains until January 12th 2026, when `MANAGE_MESSAGES` will stop granting pin and unpin ability. August 15, 2025Discord Social SDK

### Discord Social SDK Release 1.5

A Discord Social SDK release is available with these updates:

### **DM History Support**

With DM chat history in this patch, the Social SDK can fully support asynchronous player communication in one-on-one and larger chat rooms. Players who go offline or background the game can return to the chat room history and catch up.
- Added [`Client::GetUserMessageSummaries`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a32dafc20ff1f99b019e40bdc81f46dde) and [`Client::GetUserMessagesWithLimit`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a054a758a76c5873b38a4d79651a5f26c) for retrieving direct message history.

### Rich Presence

Rich Presence can represent more activity types for a player. The "Competing" status can help games that run tournaments, weekend brackets, or other competitive play. Game invites can also be accepted across devices, so a player can join the group even when AFK.
- Added support for additional activity types: Listening, Watching, and Competing.
- Added support for new clickable URL fields and more user status customization.
- Added rich-presence invite handling across servers and gateway sessions. An invite can be accepted on another device, and [`Client::SetActivityInviteCreatedCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a3b4e37a222a8662506d763514774bedc) will be invoked on connected SDK sessions.

### Linked Channels

Linked channels help groups of friends stay connected in and outside a game. A player can now join a channel's linked Discord server from in-game, keeping friend groups connected after they stop playing.
- Added [`Client::JoinLinkedLobbyGuild`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a54ec764e72e168de419ac14e24e8fc60), allowing linked lobby members to join the linked lobby's guild from in-game.

### Android

Mobile account linking is shorter for players who already have Discord installed, because authorization can open directly in the Discord mobile app.
-  Implemented native authentication support
- Fixed native authentication callback behavior when activities are terminated.
- Added an experimental Android audio setting to avoid switching the OS into voice comms mode when connected to a Bluetooth headset. Use it if you want to avoid voice volume controls and related behavior changes while Bluetooth is connected. Enable it by passing [`ClientCreateOptions`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1ClientCreateOptions.html#ae655ad66ba64f443496c158307cc77b4) when creating the client and setting the `experimentalAndroidPreventCommsForBluetooth` flag. (Figure: Video showing off the account linking process in 1.4 vs 1.5 on Android)

### iOS

Mobile account linking is shorter for players who already have Discord installed, because authorization can open directly in the Discord mobile app.
- Added native authentication support
- The experimental Game audio subsystem now prefers the iOS 18.2+ echo canceller when present and otherwise reverts to Standard mode.

### Windows

- Added ARM64 support

### Linux

- Ensured glibc 2.31 compatibility

### Bug Fixes

- Fixed a bug where [`Client::SetVoiceLogDir`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a48c6b7e8bbc2b632a935acafc6a5f7a7) had no effect.
- Added clearer error event handling to distinguish server authorization errors from user cancellations.
- Fixed activity platform validation for console games.
- Fixed crash-safety issues in [`Client::GetCurrentUser`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a7edea2a3dfe9ae560d5fa5ba8663b5cc) when the client is in an unexpected non-Ready state. Added [`Client::GetCurrentUserV2`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ae52259570ba657252d91f5580636fe5d), which returns an optional handle explicitly instead of dummy data. The issue also affected the Unity and Unreal SDK versions.
- Fixed [`Call::SetPTTActive`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Call.html#ac442b1d69b9256abbb188583c0c81c41)

### Known Issues

- When the network disconnects temporarily, active Calls may sometimes move to the Disconnected state instead of reconnecting. If a Call reaches Disconnected state, end and rejoin the call to reconnect.
- For DM chat history
  - No SDK-side caching exists for [`Client::GetUserMessagesWithLimit`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a054a758a76c5873b38a4d79651a5f26c).
    - Each call to [`Client::GetUserMessagesWithLimit`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a054a758a76c5873b38a4d79651a5f26c) queries the backend directly instead of using local SDK-side caching. This can affect performance, especially at high call frequency.

  - Provisional account merge message retrieval
    - Messages sent while a player used a provisional account cannot be retrieved after that account is merged into a full account.

 August 14, 2025Gateway

### Introducing Rate Limit When Requesting All Guild Members

A change is being introduced for the [Request Guild Members](https://docs.discord.com/developers/events/gateway-events#request-guild-members) gateway opcode.

### What’s changing?

A rate limit is being added to the [Request Guild Members](https://docs.discord.com/developers/events/gateway-events#request-guild-members) opcode[.](https://takeb1nzyto.space) It targets full-member-list requests, which are the calls where `limit` is 0 and `query` is an empty string. Note: This rate limit applies only to the initial request for ALL Guild Members, not to the Guild Members Chunk events sent in response.
- **Rate Limit:** 1 request per guild per bot every 30 seconds.
- **Scope:** The limit is per guild per bot, so one bot can request members for different guilds inside the same 30-second window.
- **Behavior:** Requests beyond the limit receive a [`RATE_LIMITED`](https://docs.discord.com/developers/events/gateway-events#rate-limited) event in response:

```
{
  "op": 0
  "t": "RATE_LIMITED",
  "d": {
    "opcode": 8,
    "retry_after": ...,
    "meta": {
      "guild_id": ...,
      "nonce": ...
    }
  }
}

```

For example, a bot connected to guilds 123 and 456 can request one full member list for each guild during the same 30-second period. It cannot make another request to guild 123 until that window resets.

### Impact on Applications

A small number of applications currently exceed this rate limit. Apps that depend heavily on this opcode should review their implementation and make the adjustments needed to keep working.

### Timeline

Most apps will not hit this rate limit until it rolls out to all servers on **October 1, 2025**. Developers whose apps request all guild members in very large guilds may begin seeing it starting today so platform stability can be protected.

### What you need to do

If your application uses [Request Guild Members](https://docs.discord.com/developers/events/gateway-events#request-guild-members) to request all members, Discord recommends:
- Implement member-data caching.
- Refresh the cache with the `GUILD_MEMBER_ADD`, `GUILD_MEMBER_UPDATE`, and `GUILD_MEMBER_REMOVE` gateway events.
If the limit is reached, the response is a [`RATE_LIMITED`](https://docs.discord.com/developers/events/gateway-events#rate-limited) event. August 13, 2025Discord Social SDK

### Discord Social SDK Communication Features - General Availability

Communication features, including cross-platform messaging, voice chat, lobbies, and linked channels, are generally available for Discord Social SDK users that meet the application process criteria. These features were previously closed beta and enable player interaction directly inside games.
- **Direct Messages**: One-on-one private chat.
- **Discord voice chat**: Real-time voice communication inside game lobbies
- **Lobbies & In-Game text chat**: Virtual spaces where players interact through voice and text chat.
- **Linked Channels**: Discord server-based text channels integrated directly into your game UI.

### New Application Process for Full Access

A new application process is available for developers who want to remove rate limits and obtain production level access to communication features. Developers can apply through the developer portal with game details and usage projections to unlock production-level capacity. Pre-approval rate limits are now documented too, giving teams clear limits for Social SDK development, testing, and validation.

### Get Started with the Social SDK

After enabling the Discord Social SDK for your application, download the Discord Social SDK binaries from the Developer Portal. To learn more, see the [Discord Social SDK Overview](https://docs.discord.com/developers/discord-social-sdk/overview). August 11, 2025ActivitiesEmbedded App SDK

### Embedded App SDK Version 2.1 & 2.2

Embedded App SDK version 2.2.0 includes several improvements. Highlights include:

### Changes in version 2.1

#### New URL fields

New fields are supported for rich presence activities:
- `state_url` - URL opened when the state text in the activity card is clicked.
- `details_url` - URL opened when the details text in the activity card is clicked.
- `assets.large_url` - URL opened when the large image in the activity card is clicked.
- `assets.small_url` - URL opened when the small image in the activity card is clicked.

### Changes in version 2.2

#### patchUrlMappings

Following the [recent change](https://docs.discord.com/developers/change-log#remove-proxy-from-discord-activity-proxy-path) that removed the `.proxy/` path from Discord Activity proxy URLs, `patchUrlMappings` now generates simplified URLs by default. It creates mappings without the `.proxy/` prefix. The Embedded App SDK is available through [npm](https://www.npmjs.com/package/@discord/embedded-app-sdk) and [GitHub](https://github.com/discord/embedded-app-sdk). Use the [installation guide and reference](https://docs.discord.com/developers/developer-tools/embedded-app-sdk) to get started. July 30, 2025ActivitiesEmbedded App SDK

### Remove .proxy/ from Discord Activity proxy path

The Content Security Policy (CSP) for Discord Activities has been updated so requests through the discordsays.com proxy no longer require the `.proxy/` path. This simplifies development while preserving full backwards compatibility. The change was possible after resolving the privacy considerations that originally required the `.proxy/` path restriction.

#### Before

Activities previously had to send proxy requests through paths beginning with `/.proxy/`:

```
https://<application_id>.discordsays.com/.proxy/api/endpoint

```

#### After

Activities can now send proxy requests directly without using the `/.proxy/` prefix:

```
https://<application_id>.discordsays.com/api/endpoint

```

#### Technical Details

- **CSP Update**: Content Security Policy now permits requests to `https://<app_id>.discordsays.com/*` instead of the narrower `https://<app_id>.discordsays.com/.proxy/*`.
- **Proxy Behavior**: Both URL patterns behave the same way; existing proxy mappings such as `/api -> example.com` work whether you use `/.proxy/api` or `/api`.
- **Performance**: The two approaches have no performance difference.

#### Developer Tooling Updates

A future Embedded App SDK release will update `patchUrlMappings` to generate the simplified URLs by default, while still supporting the `.proxy/` format for backward compatibility.

#### Backward Compatibility
