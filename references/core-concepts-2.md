# Core Features, Integration Overview, Oauth2 Scopes, Platform Compatibility ...

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Covers Account Linking, Next Steps, and Default Presence Scopes. Also covers Game Engine and Language plus Release Schedule.

## Account Linking

The Discord Social SDK provides features for adding social interaction to games. Developers can use them to make player experiences more connected and engaging.

### Account Linking

 Account linking lets a game sign users in with Discord credentials and then access social capabilities such as friends, chat, and presence. The flow is based on OAuth2 authentication. For a deeper explanation of account linking behavior, purpose, and implementation, see the [Account Linking](https://docs.discord.com/developers/platform/account-linking) platform overview.
| Development Guides |
| --- |
| Account Linking from Your Game |
| Account Linking on Mobile |
| Account Linking on Consoles |
| Account Linking through Discord Entry Points |

| Design Guidelines |
| --- |
| Signing in with Discord |
| Consoles |

### Provisional Accounts

 Provisional accounts allow players to use social features without linking a Discord account, which helps keep gameplay consistent for all players.
| Development Guides |
| --- |
| Using Provisional Accounts |

| Design Guidelines |
| --- |
| Provisional Accounts |

### Friend System & Relationships

 The SDK represents friend and relationship data in two categories:
- Discord Friends: Persist across games.
- Game-Specific Friends: Scoped only to the current game.

| Development Guides |
| --- |
| Creating a Unified Friends List |

| Design Guidelines |
| --- |
| Unified Friends List |
| Game Friends |

### Presence & Rich Presence

 Presence describes online status, while Rich Presence adds game-specific activity details:
- Shows whether a user is online, idle, or offline.
- Shows specific game details, such as current level or play time.
- Lets users send game invites from Discord and from inside the game.

| Development Guides |
| --- |
| Setting Rich Presence |

| Design Guidelines |
| --- |
| Status & Rich Presence |

---

### Next Steps

 Review implementation steps, set up authentication, and continue into development guides:

### Communication Features

Learn about messaging, voice chat, and lobbies inside games.

### Integration Overview

Review the integration overview and the implementation sequence.

### Development Guides

Use step-by-step guides for each social feature implementation.

---

### Change Log

| Date | Changes |
| --- | --- |
| July 21, 2025 | initial release |

## Next Steps

Most Discord Social SDK integrations follow this general sequence across platforms:
1. Import the SDK
2. Initialize the SDK:
  - Instantiate a [`Client`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a91716140c699d8ef0bdf6bfd7ee0ae13).
  - Register event listeners for SDK callbacks and events

3. Authenticate users with flexible account choices
  - Link an existing Discord account using OAuth through [`Client::Authorize`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ace94a58e27545a933d79db32b387a468).
  - Use [`Client::GetProvisionalToken`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a8003130b6c46e54ac68442483bf0480c) to create and manage provisional accounts for users who do not have, or do not want to use, a Discord account.

4. Implement social features
  - Build a [unified friends list](https://docs.discord.com/developers/discord-social-sdk/development-guides/creating-a-unified-friends-list) and manage [relationships](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-relationships).
  - Publish game activity updates with [rich presence](https://docs.discord.com/developers/discord-social-sdk/development-guides/setting-rich-presence).
  - Configure [lobbies](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-lobbies) for multiplayer interactions and [game invites](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-game-invites).
  - Manage [direct message](https://docs.discord.com/developers/discord-social-sdk/development-guides/sending-direct-messages), [linked channels](https://docs.discord.com/developers/discord-social-sdk/development-guides/creating-a-unified-friends-list), and [voice communication](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-voice-chat).

5. Handle events & API calls:
  - Subscribe to friend list changes, presence changes, and chat messages.
  - Call Discord's APIs to update statuses, send messages, and manage connections.

 This page is a conceptual overview. When you are ready to build, [follow our step-by-step guide](https://docs.discord.com/developers/discord-social-sdk/getting-started) to configure the SDK for your game engine.

---

### Next Steps

 Review platform support, communication features, and integration setup:

### Platform Compatibility

Review supported platforms and SDK download details.

### Communication Features

Learn about messaging, voice chat, and lobbies inside games.

### Getting Started

Select a platform and follow the setup guide.

---

### Change Log

| Date | Changes |
| --- | --- |
| July 21, 2025 | initial release |

## Default Presence Scopes

This page describes OAuth2 scopes used by the Discord Social SDK. For the broader Discord API OAuth2 scope list, including bots and webhooks, see the [OAuth2 topic guide](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-scopes). OAuth2 scopes control what access your app has to a user's Discord account. Configure the scopes for your integration with [`AuthorizationArgs::SetScopes`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1AuthorizationArgs.html#aa3714d11a196e0d71c8c1cf38c506d92) on [`AuthorizationArgs`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1AuthorizationArgs.html#adb47ac55258db29d4cb8a2c506093eed), which is passed to [`Client::Authorize`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ace94a58e27545a933d79db32b387a468) during Social SDK authentication.

### Default Presence Scopes

 For rich presence, friends list, and similar baseline features, the Social SDK needs at least these scopes:
- `openid`
- `sdk.social_layer_presence`
 The default presence features include:
- [Account Linking](https://docs.discord.com/developers/discord-social-sdk/core-concepts/core-features#account-linking)
- [Provisional Accounts](https://docs.discord.com/developers/discord-social-sdk/core-concepts/core-features#provisional-accounts)
- [Friend System & Relationships](https://docs.discord.com/developers/discord-social-sdk/core-concepts/core-features#friend-system-relationships)
- [Presence & Rich Presence](https://docs.discord.com/developers/discord-social-sdk/core-concepts/core-features#presence-rich-presence)
 The Social SDK includes [`Client::GetDefaultPresenceScopes`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a7648bd1d2f7d9a86ebd0edb8bef12b5c), which returns `openid sdk.social_layer_presence`. Use it when setting up OAuth2 for integrations that need only the functionality listed above. Default presence scopes alone do not allow a game to use limited-access communication features.

### Default Communication Scopes

 Communication features are available with limited access. They **require** `sdk.social_layer`, a scope that includes `sdk.social_layer_presence` and also permits the app to use those limited features for the user.
- `openid`
- `sdk.social_layer`
 These communication features include:
- [Messaging & Communication](https://docs.discord.com/developers/discord-social-sdk/core-concepts/communication-features#messaging-communication)
- [Lobbies & In-Game Chat](https://docs.discord.com/developers/discord-social-sdk/core-concepts/communication-features#lobbies-ingame-chat)
- [Linked Channels](https://docs.discord.com/developers/discord-social-sdk/core-concepts/communication-features#linked-channels)
 The Social SDK includes [`Client::GetDefaultCommunicationScopes`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a71499da752fbdc2d4326ae0fd36c0dd1), which returns `openid sdk.social_layer`. Use it for OAuth2 setup when the integration needs both default functionality and limited communication features. For details, see [Core Concepts: Communication Features](https://docs.discord.com/developers/discord-social-sdk/core-concepts/communication-features). If your game needs additional scopes, append only the scopes required for the game to function. Asking for unnecessary scopes can reduce user trust and may cause users to avoid linking Discord.

### OAuth2 Client Types

 OAuth2 clients are either **Confidential** or **Public**. Most games should avoid shipping with **Public Client** enabled. Some Social SDK methods require a Discord application configured as a **Public Client**, but server-side alternatives exist for **Confidential Client** integrations.
- Production applications are generally better served by confidential clients with properly protected secrets.
- Public clients cannot securely store client secrets.
- Have the security team review this setting and the authentication flows before release.
 [Learn more about OAuth2 client types](https://oauth.net/2/client-types)

---

### Next Steps

 Begin integration work, add user authentication, and review UI guidance:

### Getting Started

Select a platform and follow the setup guide.

### Account Linking

Add Discord OAuth2 authentication for users.

### Design Guidelines

Learn how to design game UI around social feature integration.

---

### Change Log

| Date | Changes |
| --- | --- |
| July 21, 2025 | initial release |

## Game Engine and Language

SDK download instructions for each platform are available in the [Getting Started](https://docs.discord.com/developers/discord-social-sdk/getting-started) guide.

---

### Game Engine and Language

 The Discord Social SDK works with these game engines and language environments:
| Integration | Supported Version |
| --- | --- |
| Standalone C++ | C++20+ |
| Unity | 2021.3+ |
| Unreal Engine | 5.5+ |

 **PlayStation 4 / PS5 exception:** Discord Social SDK integrations on PlayStation 4 still support C++17. The same C++17 allowance applies to PlayStation 5 projects using a PS5 SDK older than 13.000. Every other platform requires C++20. Before Social SDK release 1.9, C++17 was the minimum supported version.

---

### Platforms

 The Discord Social SDK can be used on these platforms:
| Platform | Supported Versions | Support Level | Standalone C++ | Unreal Engine | Unity |
| --- | --- | --- | --- | --- | --- |
| **Desktop** |  |  |  |  |  |
| Windows (x64) | 10+ | Generally Available | ✅ | ✅ | ✅ |
| Windows (ARM64) | 11+ | Generally Available | ✅ | ❌ | ❌ |
| macOS (x64) | 10.5+ | Generally Available | ✅ | ❌ | ✅ |
| macOS (ARM64) | 11+ | Generally Available | ✅ | ❌ | ✅ |
| Linux | *glibc 2.31+ | Experimental | ✅ | ✅ | ✅ |
| **Mobile** |  |  |  |  |  |
| Android | 7.0+ | Generally Available | ✅ | ✅ | ✅ |
| iOS | 15.1+ | Generally Available | ✅ | ✅ | ✅ |
| **Console** |  |  |  |  |  |
| Xbox One | GDK 250400-260400 | Experimental | ✅ | ✅ | ❌ |
| Xbox Series X|S | GDK 250400-260400 | Generally Available | ✅ | ✅ | ✅ |
| PlayStation 4 | OS 11.500-12.500 | Experimental | ✅ | ✅ | ❌ |
| PlayStation 5 | OS 11.000-13.000 | Generally Available | ✅ | ✅ | ✅ |

 * Discord cannot test every Linux distribution, but most distributions using glibc 2.31, such as Ubuntu 20.04, or newer should work. Each Social SDK release covers roughly an 18-month window of Xbox and PlayStation OS SDK versions. See [Console OS SDK Support](https://docs.discord.com/developers/discord-social-sdk/core-concepts/release-cadence-and-support#console-os-sdk-support) for how that window advances with each release. Console games require middleware approval and approved-developer status for the target console before using the Discord Social SDK. See [this article](https://support-dev.discord.com/hc/en-us/articles/30209074764183) for more information. Exception: Prohibited Off-Platform Interactions by Sony
 For integrations that run on Sony Platforms, such as PlayStation 5...

1. **Do not allow access to Account Linking** entry points.
2. **Do not allow access to Linked Channels**.
3. **Limit messaging and Invites to other players currently playing** the game.
4. **Do not display off-platform presence and logos** (including Discord).

---

### Next Steps

 Set up authentication, review features, and read the release cadence guidance:

### Release Cadence & Support

Release timing, supported-version window, and end-of-life policy.

### Core Features

Explore social capabilities such as account linking, friends, and rich presence.

### Getting Started

Setup guidance and download links by platform.

---

### Change Log

| Date | Changes |
| --- | --- |
| May 26, 2026 | Added console minimum versions and linked Release Cadence & Support |
| May 20, 2026 | Added the Game Engine and Language section. |
| July 21, 2025 | initial release |

## Release Schedule

This page explains which Discord Social SDK versions receive updates, how releases are scheduled, and how end-of-life works. For platform support, see [Platform Compatibility](https://docs.discord.com/developers/discord-social-sdk/core-concepts/platform-compatibility). These changes begin with the Discord Social SDK **1.10 release**, shipping in **July 2026**. Starting with 1.10:
- The supported range will cover **1.6 through 1.10**. Releases **1.5 and earlier** will move to end-of-life.
- New Xbox and PlayStation OS SDK support will be delivered only through new Social SDK releases. Older Social SDK versions will not receive that support through patches. Each Social SDK release includes roughly the latest 18 months of console OS SDK versions.
**Already-shipped Social SDK releases (1.6-1.9) are unchanged** - each keeps the console OS SDK support it originally shipped with. The new support model applies from 1.10 onward. Discord announced the change ahead of the 1.10 release to give developers on older versions upgrade time.

### Release Schedule

 Discord ships **three minor releases** of the Discord Social SDK each year: **April**, **July**, and **November**. That schedule lines up with console OS SDK release windows so each Social SDK release can include the newest supported console SDK versions. Patch releases may be published between minor releases when needed; see What's Covered below.

### Supported Version Window

 Discord provides **hotfix patches** for the **five most recent minor versions** of the Discord Social SDK. Those patches can include bug fixes and security fixes. Older releases become End-of-Life (EOL) and no longer receive updates. To get a patch fix when on an EOL version, upgrade to a supported release first. Console OS SDK update behavior is described in Console OS SDK Support below.

### Supported Versions

 After the 1.10 release ships in July 2026, the support window will be:
| Version | Status |
| --- | --- |
| 1.10 | Supported — current |
| 1.9 | Supported |
| 1.8 | Supported |
| 1.7 | Supported |
| 1.6 | Supported |
| 1.5 and earlier | End-of-Life |

### Console OS SDK Support

 New Xbox and PlayStation OS SDK support is added only in **new minor Social SDK releases**, which occur three times per year. Discord does not backport that platform-SDK support to older Social SDK versions as patches. Each Social SDK release packages roughly the last **18 months** of console OS SDK versions, starting from the Xbox GDK or PlayStation OS current about 18 months before the release and ending with the latest available at release. Console OS SDK versions older than that window are removed from support with each new Social SDK release. Practically, this means:
- Upgrade to the latest Social SDK release when you need the newest Xbox or PlayStation platform SDK support.
- Social SDK releases that already shipped keep supporting the console OS SDK versions they originally included; Discord does not add or remove console OS SDK support from already-shipped versions.
 The current Social SDK release's minimum Xbox and PlayStation platform SDK versions are listed in [Platform Compatibility](https://docs.discord.com/developers/discord-social-sdk/core-concepts/platform-compatibility).

### End-of-Life Process

 When a new minor Social SDK version is released, the oldest release in the supported window exits support and becomes End-of-Life:
1. It is removed from the supported version table above
2. No additional hotfix patch builds are released for that version
3. Developers using that version should move to the newest supported version
 EOL transitions are announced in the [Change Log](https://docs.discord.com/developers/change-log) entry for the relevant minor release.

### What’s Covered

 For a minor version inside the supported Social SDK window, hotfix patches may include:
- Stability and bug-fix hotfix patches
- Security hotfix patches

### What’s Not Covered

 Older minor versions of the Social SDK do **not** receive:
- New feature development
- Backports of features added in a later minor version
- Backports that add support for newer console OS SDK versions

---

### Next Steps

### Platform Compatibility

Review supported platforms and minimum supported versions.

### Change Log

Read release notes and EOL announcements.

### Getting Started

Download the newest supported SDK version.

---

### Change Log

| Date | Changes |
| --- | --- |
| May 26, 2026 | initial release |
