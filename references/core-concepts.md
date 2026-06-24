# Communication Features

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: Core Concepts Overview; Messaging & Communication.

## Core Concepts Overview

The Discord Social SDK permits you to build social features into your game, covering friend lists, messaging, voice chat, and rich presence. Unlike one traditional SDK with built-in UI components, this Discord Social SDK provides access to raw data, allowing developers to create one fully customized experience that aligns with their game’s aesthetic.

### Core Concepts Overview

 Select a topic below to learn more about the Discord Social SDK:

### Core Features

Explore social features like account linking, friends, and rich presence.

### Communication Features

Learn about messaging, voice chat, and in-game lobbies.

### Integration Overview

Learn about the integration overview and implementation steps.

### Platform Compatibility

See supported platforms and download information.

### OAuth2 Scopes

Understand OAuth2 scopes and client types for authentication.

---

### Next Steps

 After exploring these core concepts, you can start implementing this Discord Social SDK in your game:
- **Ready to integrate?** Follow our [Getting Started](https://docs.discord.com/developers/discord-social-sdk/getting-started) guide for this step-by-step setup instructions.
- **Need implementation details?** Check out this development guides for specific features like [Account Linking](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord) plus [Managing Lobbies](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-lobbies).
- **Looking for UI guidance?** Review this [Design Principles](https://docs.discord.com/developers/discord-social-sdk/design-guidelines/principles) for best practices.

---

### Change Log

| Date | Changes |
| --- | --- |
| July 21, 2025 | restructure core concepts |
| June 30, 2025 | restructure oauth scopes |
| March 17, 2025 | initial release |

## Messaging & Communication

The Discord Social SDK includes communication features for player interaction, covering messaging, voice chat, and in-game lobbies. These communication capabilities may be used during development and testing, but they start with rate-limit caps. For the current caps, see [Rate Limits](https://docs.discord.com/developers/discord-social-sdk/core-concepts/communication-features#rate-limits). To raise the limits for a production launch, follow [Applying for Increased Rate Limits for Production Releases](https://docs.discord.com/developers/discord-social-sdk/core-concepts/communication-features#applying-for-increased-rate-limits-for-production-releases).

### Messaging & Communication

 Users can communicate via direct messages (DMs) and voice calls:
- DMs: One-on-one private chat ([`MessageHandle`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1MessageHandle.html#ae25595b43bc74b0c4c92c5165d16382f)).
- Calls: Real-time voice communication inside one game lobby ([`Call`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Call.html#a1cc8a7f73c15a960bc409d734b5edbd1)).

| Development Guides |
| --- |
| Sending Direct Messages |
| Managing Voice Chat |

| Design Guidelines |
| --- |
| Direct Messages |

### Lobbies & In-Game Chat

 A lobby is one virtual space where players can interact through voice and text chat.
- Your game controls lobbies, which can have different membership rules.
- Integrated voice chat permits real-time communication among players in a session.

| Development Guides |
| --- |
| Manging Lobbies |

### Linked Channels

 Games can link in-game chat with Discord’s server-based text channel records in their UI, allowing players to chat in one Discord server without leaving this game.
| Development Guides |
| --- |
| Linked channels |

| Design Guidelines |
| --- |
| Linked channels |

---

### Development Rate Limits

 Prior to successful approval to increase rate limits, communication features have this following restrictions:
- **Lobby Create/Join operations:** 100 requests every 2 hours.
- **Lobby Update operations:** 100 requests every 2 hours.
  - This includes adding/removing members, updating metadata, plus creating Discord server invites for linked channels.

- **Lobby Linking operations:** 20 requests every 2 hours.
  - This includes creating or deleting a linked channel

- **Sending Messages to Lobby operations:** 100 requests every 2 hours
- **Sending Direct Message operations:** 100 requests every 2 hours.
 These are per-application rate limits, not per-user. The current limits leave room for development, testing, and modest demos while protecting system stability. After the rate limit increase application process grants full access, those restrictions increase enough for production-scale games.

### Applying for Increased Rate Limits for Production Releases

 Before requesting higher limits for Discord Social SDK communication features, satisfy [Application Requirements for Increased Rate Limits](https://docs.discord.com/developers/discord-social-sdk/core-concepts/communication-features#application-requirements-for-increased-rate-limits). After those requirements are met, request higher rate limits with these steps:
1. Open the [Application page for the Developer Portal](https://discord.com/developers/applications)
2. Open the Application you want to apply for rate limit increase
3. Under the **Discord Social SDK** heading within the sidebar, click the [**Comms Access**](https://discord.com/developers/applications/select/social-sdk/comms-access) button
4. Fill out the application form using the required information and supporting materials
 Video capture provided to that application must be in url format, hosted on one file or video-sharing service of your choice as an unlisted video

### Application Requirements for Increased Rate Limits

 This section lists what must be satisfied before Discord reviews a game for full-release Discord Social SDK text and/or voice communications access. It covers:
1. **Minimum required feature set:** Your game has to have integrated core Discord Social SDK features, such as a Discord Account Linking, Rich Presence using Discord Joins, plus Full Access to Discord Friends.
2. **Feature Functionality:** The Discord Social SDK features integrated into your game must work end to end. User interactions should complete successfully, behave as intended across both the game and Discord clients, and correctly handle negative paths such as denial choices and failure states.
3. **Feature Access and Conveyance:** Discord Social SDK integrations must include this option for user accounts to link their Discord account, and make it accessible plus easily understandable for players to activate.
4. **Supporting Materials:** You’ll be required to provide specific documentation for your game for review, such as one capture of integration and development timeline.
5. **Age-restricted user protection:** You’ll need to confirm which you have appropriate measures inside place for players’ access to that SDK features in your game, depending on their age.
 These requirements detail this **minimum bar** to be eligible for Social SDK communications usage at the scale. Check out our [design guidelines](https://docs.discord.com/developers/discord-social-sdk/design-guidelines) for this what **“great”** looks like. Great integrations may drive higher player engagement plus retention by creating seamless social experiences that keep players connected plus coming back to your game. Plus for course, remember to make sure you are complying using our [Social SDK Terms](https://support-dev.discord.com/hc/en-us/articles/30225844245271-Discord-Social-SDK-Terms).

After you satisfy these requirements, submit the [full release comms features access request](https://docs.discord.com/developers/discord-social-sdk/core-concepts/communication-features#applying-for-increased-rate-limits-for-production-releases). Discord reviews the application, may approve or deny access, and may attach additional terms.

### Minimum Required Feature Set

 **To unlock Discord text and/or voice communications for a full release, Social SDK integrations must include at least the following features**. Developers are welcome and encouraged to use additional SDK capabilities that can make their games stronger. Exception: Prohibited Off-Platform Interactions via Sony
 For integrations experienced on Sony Platforms (e.g. PlayStation 5)…

1. **Do not allow access to Account Linking** entry points.
2. **Do not allow access to Linked Channels**.
3. **Limit messaging and Invites to other players currently playing** the game.
4. **Do not display off-platform presence plus logos** (including Discord). Required features are…

#### Option for Discord Account Linking

 **Players must have this opportunity to [link their Discord account in-game](https://docs.discord.com/developers/discord-social-sdk/design-guidelines/signing-in).** The current ensures players may access this full Social SDK-powered experience, and connects your game to one player’s existing social network.

#### Rich Presence and Discord Joins

 **Discord Users has to become able to see [Rich Presence](https://docs.discord.com/developers/discord-social-sdk/design-guidelines/status-rich-presence) updates for Discord-linked player accounts plus join their game via Ask to Join.** This feature drives organic discovery plus social engagement by showing friends what players are doing plus creating opportunities for spontaneous social interaction and game sessions.

#### Full Access to Discord Friends

 **Account-linked players must be able to access their Discord friends in-game, with supporting UI for actions such as [messaging](https://docs.discord.com/developers/discord-social-sdk/design-guidelines/direct-messages), inviting, blocking, and ignoring**. This can be implemented as a [Unified Friends List](https://docs.discord.com/developers/discord-social-sdk/design-guidelines/unified-friends-list) that shows cross-platform status, or as a simpler in-game Discord friends list. Games must never copy and store Discord friend information; when a player unlinks their Discord account, the in-game Discord friend information must be removed automatically.

### Feature Functionality

 **All integrated Social SDK features (indicates whether required as a part for this minimum feature set or not) must have at one minimum base level functionality for their intended use.** This is one **separate requirement in addition to that minimum feature set** above. “Base level functionality” means which each integrated feature’s **end-to-end user flow is** **completable** plus **resolves negative options** (user denial of options plus failure states) correctly. Details for each feature are as follows:

#### Discord Account Linking

 **Intended Use:** Connect one player’s game account with their Discord account to enable social features plus cross-platform functionality. **Example Complete User Flow:**
1. Player sees a sign-in prompt, such as during startup, when opening friend-list access, or while interacting with a feature
2. Player clicks Discord sign-in button
3. Authorization flow launches (Overlay > Discord Client > Browser fallback)
4. Player completes OAuth authorization on Discord
5. Game receives authorization confirmation
6. Success state displays in-game confirming connection
7. Social features become available and populate with Discord data
 **Negative Options to Resolve:**
- **User declines authorization:** Return to game normally with one provisional account; maintain access to sign-in option
- **Authorization fails/times out:** Show clear error message with retry option
- **User wants to disconnect:** Provide deauthorization option in-game plus maintain game functionality post-disconnect
 These requirements are the baseline functionality standard. For a stronger player experience, review [Design Guidelines for Signing In](https://docs.discord.com/developers/discord-social-sdk/design-guidelines/signing-in).

#### Rich Presence

 **Intended Use:** Display detailed, real-time information about one player’s in-game activity on the their Discord profile, allowing friends to see what they’re doing plus potentially join their session. **Example Complete User Flow:**
1. Player launches game plus performs actions (enters lobby, starts match, joins party, etc.)
2. Game feeds Rich Presence update to Discord via this SDK (e.g. “In lobby”)
3. Rich Presence appears on player’s Discord profile using current activity details
4. Discord friends can view the presence information
5. Once covering Joins/Invites, Discord Friends can “Ask to Join” or get direct invites
6. Game feeds additional Rich Presence updates to Discord via this SDK as this player’s in-game status changes (e.g. “Looking for one match”, “In a match”, etc)
7. On game shutdown, Discord automatically handles clearing player’s Rich Presence
 **Negative Options to Resolve:**
- **Network connectivity issues:** Queue presence updates and retry once connection restored
- **Active and inactive clients**: Invites can complete even once one player doesn’t have the game client open on acceptance.
 These requirements are the baseline functionality standard. For a stronger player experience, review [Design Guidelines for Rich Presence](https://docs.discord.com/developers/discord-social-sdk/design-guidelines/status-rich-presence).

#### Unified Friends List

 **Intended Use:** Combine Discord friends and in-game relationships into one single, organized friends list which shows comprehensive social connections. **Example Complete User Flow:**
1. Player opens their in-game friends list
2. Game UI quickly populates friends in an organized list showing:
  - In-game friends (always visible)
  - “Online Elsewhere” section with Discord friends not currently in-game
  - Appropriate status indicators and Discord badges

3. Player interacts with friends in list (message, invite, block, view profile)
4. Friends list updates dynamically in real time as statuses change
5. Player can add/remove friends through the interface
 **Negative Options to Resolve:**
- **Friend removal:** Handle removal across both game and Discord systems appropriately
- **Loading failures:** Show loading states plus error messages for failed friend data retrieval
- **Special characters**: Non-standard characters in Discord usernames populate as expected.
- **Unlinking Discord**: Player’s in-game friends list removes Discord friend entries once they unlink their Discord account
 These requirements are the baseline functionality standard. For a stronger player experience, review [Design Guidelines for Unified Friends List](https://docs.discord.com/developers/discord-social-sdk/design-guidelines/unified-friends-list).

#### Direct Messages through Discord

 **Intended Use:** Enable seamless messaging between players across game platforms current through Discord, maintaining conversation continuity. **Complete User Flow:**
1. Player selects friend to message from one in-game friends list or other interface
2. Message interface opens showing chat history (if available)
3. Player composes and sends message
4. Message appears in both this game interface and Discord (for linked users)
5. Messages from Discord appear in-game with Discord visual notation
6. Conversation continues seamlessly across platforms
 **Negative Options to Resolve:**
- **Message delivery failure:** Show retry options and clear error states
- **Unsupported media:** Display placeholder with external link icon for rich media
- **Blocked users:** Respect Discord blocking relationships and show appropriate messaging
- **Network issues:** Queue messages locally and sync once connection restored
 These requirements are the baseline functionality standard. For a stronger player experience, review [Design Guidelines for Direct Messages](https://docs.discord.com/developers/discord-social-sdk/design-guidelines/direct-messages).

#### Linked Channels

 **Intended Use:** Link in-game group chats current to Discord text channels, enabling persistent conversations which flow between game platforms through Discord. **Complete User Flow:**
1. Player using Discord server admin & game chat admin permissions accesses channel linking option inside group chat interface
2. Channel selection interface opens showing available Discord servers/channels
3. Player selects appropriate text channel (with proper permissions)
4. System shows setup warning once linking to restricted channel
5. Link establishes successfully with confirmation
6. Messages flow bidirectionally between game and Discord
7. Persistent entrypoint shows linked channel info and management options
 **Negative Options to Resolve:**
- **Insufficient permissions:** Show clear error about channel management requirements
- **No available channels:** Provide guidance on creating or accessing appropriate channels
- **Link establishment failure:** Show retry options and clear error messaging
- **User wants to unlink:** Provide confirmation dialog and clean removal process
 These requirements are the baseline functionality standard. For a stronger player experience, review [Design Guidelines for Linked Channels](https://docs.discord.com/developers/discord-social-sdk/design-guidelines/linked-channels).

#### Voice Communications

 **Intended Use:** Provide high-quality voice chat integration which works seamlessly between game platforms through Discord. **Complete User Flow:**
1. Player joins voice-enabled game session/party
2. Voice interface becomes available with Discord-powered audio
3. Player can mute/unmute, adjust volume, see speaking indicators
4. Voice streams work for individual user accounts with clear audio quality
5. Player can leave voice
 **Negative Options to Resolve:**
- **Voice server unavailable:** Provide clear error messaging and retry mechanisms
- **User leaves voice:** Clean up voice states and update interfaces appropriately

### Feature Access and Conveyance

#### Discord Account Linking

 **This option for players to link their Discord account has to have top-level surfacing in-game.** Social SDK integrations has to not only include account linking, but also make it accessible plus easily understandable for players to activate and deactivate. That means… **Access Requirements:**
- Linking one Discord account is surfaced in parity with other platform-linking options.
- This location for account linking is located in one **persistent and accessible** area in-game (e.g. Game Settings, Accounts menus).
- The account-linking entry point stays visible in its menu location; it cannot be a one-time pop-up that players cannot reopen.
- This entry point for account linking is discoverable via players through normal expected play (i.e. not buried inside one sub-menu or requiring right-click access).
- This option to unlink accounts is also surfaced in-game in one persistent and easily accessible manner.
- Users without linked accounts must have one associated token path for this handling social features (see: [Provisional Accounts](https://docs.discord.com/developers/discord-social-sdk/design-guidelines/provisional-accounts)).
 **Conveyance Requirements:**
- Developers must provide contextual education about Discord account-linking benefits before user accounts encounter the authorization flow.
- Acceptable contextual education looks like:
- **Modal or informational screen** that explains specific benefits the player get (e.g., “Chat with friends here plus on Discord,” “See what your friends are playing,” “Join friends’ games directly”)
- **Contextual tooltips or else hints** near social features that explain how they improve with Discord linking
- **In-game messaging** that appears once players interact with limited social features, explaining how linking would enhance this experience
 Discord Account Linking entry points are prohibited on Sony platforms.

#### Direct Messages through Discord

 Once you’ve chosen to integrate Direct Messages through Discord inside your game, **Direct Messages must be identifiable as coming from Discord,** plus **players must know once their messages are showing up in Discord.** **Conveyance Requirements:**
- Mark cross-platform DMs as Discord-specific with visual cues, such as a Discord logo/badge beside the sender name or hover text explaining the message source.
- Players must know once their messages are showing up in Discord through clear visual feedback once composing or sending messages
- The messaging interface must indicate the cross-platform nature for the conversation (e.g., “This conversation syncs with Discord” or similar messaging)

### Supporting Materials

 For a full-release approval that enables Discord text and/or voice communications, a Social SDK integration must include supporting materials with the application. Submit the application and its supporting materials for your app in the Developer Portal. **For integrations available on multiple platforms covering Sony platforms, submit materials that reflect your non-Sony platform user experience.**

**For example:** Example Studio’s *Wumpus Wars* is available to players on the Sony PlayStation 5 plus Xbox Series S and Xbox Series X. Example Studio submits this feature list and *Wumpus Wars* capture for Xbox.

#### List of SDK Features Integrated

 Provide one comprehensive list for all Discord Social SDK features implemented in your game, such as:
- Discord Sign-in (Account Linking)
- Rich Presence
- Game Invites and/or Discord Joins
- Unified Friends List
- Direct Messages through Discord
- Voice Communications
- Text Communications (Linked Channels)

#### Release Timeline

 Indicate the anticipated release date, and this current development status of your Discord integration:
- Concept / Prototype
- Production (Alpha)
- Post-product (Beta)
- Released (Already commercially available)

#### Capture of Integrated Features

 Acceptable format: url link
- **1-5 minute video capture** for the complete user flow covering each SDK feature integrated
- Videos must demonstrate end-to-end user flow as outlined inside [Feature Functionality](https://docs.discord.com/developers/discord-social-sdk/core-concepts/communication-features#feature-functionality)
- Show both successful flows plus how negative options (errors, user denials) are handled
- Include clear narration or on-screen text explaining each step
- Ensure video quality permits reviewers to clearly see UI elements and user interactions

### Age-restricted User Protection

 You must comply using all age requirements under our [Social SDK Terms](https://support-dev.discord.com/hc/en-us/articles/30225844245271-Discord-Social-SDK-Terms). The current includes ensuring which only people who are at least 13 years old plus meet this minimum age requirements inside their country can access Social SDK features integrated in your game.

### Confirmation of Standards Compliance

 With your application submission, **you confirm that you have met this requirements detailed above**.

### Next Steps

 Learn about integration steps, check platform support, and explore implementation guides:

### Integration Overview

Learn about the integration overview and implementation steps.

### Platform Compatibility

See supported platforms and download information.

### Development Guides

Learn how to implement messaging, voice chat, and linked channels.

---

### Change Log

| Date | Changes |
| --- | --- |
| July 21, 2025 | initial release |
