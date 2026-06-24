# Branding Guidelines, Chat History, Connection Points, Consoles ...

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Overview of Discord branding, style guidance, sign-in, console connection, online-elsewhere badges, user needs, linked game chats, principle 1 about player value, unlinked Discord accounts, and the full connection flow.

## Discord Logo

### Discord Logo

 Get the Discord word-mark assets from [discord.com/branding](https://discord.com/branding). Do not modify, reshape, distort, recolor, or rearrange the Discord logo. (Figure: Discord Logo)

### Buttons

 This integration uses four button style groups: Blurple, Light, Dark, and Flexible.

#### Flexible

 Flexible buttons are designed so games can adapt them to their own visual style (examples). This flexibility covers the text font before the logo and the button colors. The Discord logo itself, including its colors, must remain unchanged. Use this set for in-game connection locations.

#### Blurple / Light / Dark

 The Blurple, Light, and Dark button sets use colors approved by Discord Design Systems for contrast across different background treatments. (Figure: Brand Buttons)

### Spacing

 Leave enough space between the Discord logo, the button text, and the button's left and right edges. With Flexible buttons, align the chosen typeface baseline and x-height to the Discord logo. (Figure: Brand Spacing)

### Resources

- [Discord branding guidelines](https://discord.com/branding)

### Change Log

| Date | Changes |
| --- | --- |
| March 17, 2025 | Initial Release |

## Style recommendations

### Style recommendations

 Chat history is a frequently requested **optional** capability that lets players resume conversations after returning to a lobby or linked channel. Discord recommends showing **timestamps** so players understand when messages were sent and read. (Figure: Badge when online elsewhere)

### Display logic

 When one player sends messages from a game lobby or linked channel, preserve the chat history even if the other player is using Discord. When players return to the lobby or open the linked channel, surface **10 to 15** recent messages for useful context. This is not a strict cap; older history can be exposed through scrolling or a load-more pattern depending on the interface. (Figure: Message styling)

---

### Resources

- [Development Guide: Creating and Managing Lobbies](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-lobbies)

### Change Log

| Date | Changes |
| --- | --- |
| July 02, 2025 | initial release |

## Signing in

This guide describes recommended design practices for key connection points in a game. These points are places where players can log in to Discord and improve the surrounding game experience.

### Signing in

 When a game includes account management, this connection point is **required**; otherwise it is optional. Present Discord sign-in as the **primary option** among **external identity-providers** because it provides deeper user benefits than ordinary OAuth login. Use the [**Blurple Button**](https://docs.discord.com/developers/discord-social-sdk/design-guidelines/branding-guidelines#buttons) style for the sign-in connection point. (Figure: Signing In)

### Friends list

 Until a player connects a Discord account, show a **persistent call-to-action** in the friends list. Choose the preferred *button-styling* for this friends-list connection point from the Brand guidelines section. After account connection succeeds, remove that connection point from view. (Figure: Friends List)

### Content Guidelines

 Use the strings shown here in their matching contexts. Connection-point copy should stay consistent across games using the Discord Social SDK so users build recognition, trust, and understanding around the action. (Figure: Content Guidelines)

---

### Change Log

| Date | Changes |
| --- | --- |
| March 17, 2025 | initial release |

## Connect to Discord on console

Console games using the Discord Social SDK require middleware approval and approved-developer status for the target console. See [this article](https://support-dev.discord.com/hc/en-us/articles/30209074764183) for details. Exception: Prohibited Off-Platform Interactions by Sony
 For integrations shown on Sony Platforms, such as PlayStation 5...

1. Block **Account Linking** entry points.
2. Block access to **Linked Channels**.
3. Allow messaging and **Invites** only with other players who are currently in the game.
4. Hide **off-platform presence and logos** (including Discord). Players should get a coherent Discord social experience across PC, mobile, and console. Although console largely follows the same experience as other platforms, the following slides call out console-navigation adjustments. Console players **will need a secondary device** to access the full Discord Social SDK benefits. (Figure: Connect on console)

### Connect to Discord on console

 On console, the broad connection flow matches other devices. After a player selects an in-game CTA to connect to Discord, the game shows sign-in options. Users should use a phone or computer rather than a controller-driven console browser, which creates a poor sign-in experience. A player without a Discord account can create one on the web. (Figure: Connect on console) Scanning the QR code with a mobile camera lets users skip the device-code screen and move to game authorization. If scanning does not detect the Discord app, the user is asked to sign in and connect through a web browser. Users who skip QR scanning should go to discord.com/activate and enter the 8-digit code. (Figure: phone connection step)

### Chatting on console

 Games that support console game chat can make that chat available. Unsupported rich media from Discord, such as attachments, polls, and voice messages, should not show a clickable link icon. As with authorization, do not push players toward Discord in the console web browser; use secondary devices for that content. (Figure: chat on console)

### Friends list on console

 Use the existing [friends list guidelines](https://docs.discord.com/developers/discord-social-sdk/design-guidelines/unified-friends-list) from the playbook. The console friends list does not otherwise differ materially from other devices. Because consoles do not support hover, provide a console-suitable way to reveal secondary friend details such as alternate identities. (Figure: chat on console)

---

### Resources

- [Development Guide: Account Linking on Consoles](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-on-consoles)

### Change Log

| Date | Changes |
| --- | --- |
| March 17, 2025 | initial release |

## Badge when online elsewhere

### Badge when online elsewhere

 Direct-message badging follows the same rules described in [Unified Friends List](https://docs.discord.com/developers/discord-social-sdk/design-guidelines/unified-friends-list). When a user is **online elsewhere** and not in the game client, show a **Discord badge** beside their username. Once they are online in the game-client, remove that Discord badge. (Figure: Badge when online elsewhere)

### Message styling

 Color the Discord logo to match the player's username color. The mark should read as part of the user's identity rather than part of the message body. (Figure: Message styling)

### Unsupported rich media

 Unsupported rich media should use an **external link icon** whether or not the user is signed in to Discord. If the content cannot be clicked, do not display the icon. (Figure: Unsupported rich media)

---

### Resources

- [Development Guide: Direct Messages](https://docs.discord.com/developers/discord-social-sdk/development-guides/sending-direct-messages)

### Change Log

| Date | Changes |
| --- | --- |
| March 17, 2025 | initial release |

## User needs

Any in-game imagery used here is fictional and intended only for concept illustration.

### User needs

 Players have asked for clearer separation between relationships created in a game and friends they interact with on Discord outside one game or across multiple games. A person you party with is not always a "friend." Users want more control over how in-game relationships extend outside the game client through communication, presence, and identity. (Figure: User needs)

### Game Friends — A new tier

 Discord is adding a friendship tier named **Game Friends**. Game Friends create an in-game relationship with another player, preserving in-game benefits while limiting how far that relationship extends outside the game. (Figure: Game Friends)

### Game vs Discord Friends

### Communication

 A Game Friend may DM Discord-connected friends outside the game while those friends are offline; the messages arrive in the Discord client. Discord also gives players a setting to block DMs from Game Friends.

### Presence

 A Game Friend **cannot see your rich presence** outside the game where the friendship began, except when both users have connected Discord accounts and share a server. (Figure: Game vs Discord Friends)

### Equally-weighted

 When a player adds someone, show both friend types and list **Game Friends** first because it is the more local and limited tier. Do not preselect either friendship type; aside from the ordering, give both options equal weight. (Figure: Equally-weighted friend options)

### Adding friends

 For a user who is not yet a friend, offer both choices: add as a Game Friend or as a Discord Friend. If the user is already a Game Friend, offer the additional action to add them on Discord. (Figure: Adding friends)

### Removing friends

 When removing either a Game Friend or Discord Friend, present the normal â€œRemove Friendâ€ action. The SDK removes the friend relationship wherever it applies. (Figure: Removing friends)

### Style guidelines

 When removing either a Game Friend or Discord Friend, present the normal â€œRemove Friendâ€ action. The SDK removes the friend relationship wherever it applies. (Figure: Removing friends)

### UX, not UI

 The â€œAdd Friendâ€ menu-item visuals are not prescribed. As long as both choices are weighted equally, use UI that fits the game. (Figure: UX, not UI)

### Setting on Discord

 This mock shows the matching DM state inside Discord's UI. (Figure: Removing friends)

---

### Resources

- [Development Guide: Managing Relationships](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-relationships)

### Change Log

| Date | Changes |
| --- | --- |
| March 17, 2025 | initial release |

## Link in-game chats to Discord

### Link in-game chats to Discord

 Let players keep conversations going after they leave the game. An in-game chat can now be linked to a Discord text-channel. (Figure: Linking a channel via the game)

### Style guidelines

#### Feature Name & Language

 The official feature-name shown across the Discord client is â€œLinked Channels.â€ Use the language below when referring to the feature so players understand it consistently across clients.
- “Linked to {Discord/GameTitle}”
- “Link a {groupChat} to Discord”
- “Edit/Remove Channel Link”

#### Entrypoints

 Use the string guidance above: â€œLink {groupChat} to Discord.â€ For the UI, choose a treatment that fits the game. (Figure: Linked channel style guidelines)

### Selecting a channel

 Provide a channel-selection flow where players choose which text-channel to link with their game-chat. Only one text-channel may be linked at once. The player also needs permission to manage the channel. Include these elements in the selection UI:

#### UI Elements

- Explain what a channel link changes, including **the users who can read and send messages in the channel** after the link is created
- A list of **servers** with nested **text-channels** and appropriate icons, such as a lock for private channels
- **Search bar** for channel-names
 (Figure: Selecting a channel)

### Setup warning

 Players may link Discord channels with **limited access**. That lets players from the game-chat read and send messages in the linked channel even when they lack Discord permissions. In effect, linking a game-channel bypasses the existing Discord permissions for that channel. Show a **warning** before a user links to a limited-access channel. Discord strongly recommends using the copy shown in the right image. (Figure: Setup warning)

### Success state & Entrypoint

 After a channel link is created, show a **success state** that identifies the Discord text-channel that was linked. Add a **persistent entrypoint** in the game UI as a separate step. That entrypoint should surface the Discord server and channel name, and it becomes the place to manage the linked channel in the next slide. [You can download the official "Linked Channels" channel-icon in our GitHub repository.](https://github.com/discord/discord-api-docs/blob/main/resources/discord-social-sdk) (Figure: Success state & Entrypoint)

### Removing Channel Link

 Players can open the relevant Discord channel settings from the "**Edit settings**" menu item; pair that action with an "open in new window" icon. A channel link can be removed from the game client or from Discord. When the player selects "Remove channel-link," show a **confirmation** dialog before completing the action. (Figure: Removing Channel Link)

---

### Resources

- [Development Guide: Linked Channels](https://docs.discord.com/developers/discord-social-sdk/development-guides/linked-channels)
- [Design Assets: Linked Channel Icon](https://github.com/discord/discord-api-docs/blob/main/resources/discord-social-sdk)

### Change Log

| Date | Changes |
| --- | --- |
| March 17, 2025 | initial release |

## 1. Add Value for the Player

Any in-game imagery used here is fictional and intended only for concept illustration.

### 1. Add Value for the Player

 The integration should add value to the player's game experience. Branded connection points need to explain that value in the surrounding context. (Figure: Add Value for the Player)

### 2. Foster trust with Discord’s brand

 Use Discord visual-styling to build trust through recognizable branding. As more players connect, the game's social graph becomes stronger. (Figure: Foster trust with Discord's brand)

### 3. Promote Understanding through consistency

 Keep branding and user-flows consistent across games so players learn what to expect. The priority is UX rather than matching a particular UI. (Figure: Promote Understanding through consistency)

### 4. Prioritize the player’s immersion

 Protect player immersion. Games do not need to copy Discord's UI; the visual-design should feel as integrated into the game as the SDKs do. (Figure: Prioritize the player's immersion)

### 5. Uphold data privacy, transparency, and control

 Each platform controls and is responsible for its own platform data. Players should be told where their data appears. (Figure: Uphold data privacy, transparency, and control)

---

### Change Log

| Date | Changes |
| --- | --- |
| March 17, 2025 | initial release |

## For those who don’t connect a Discord account

### For those who don’t connect a Discord account

 Some users will not connect a Discord account to the game. That can force developers to maintain **two different friend systems** by account type. Discord Social SDK provides provisional accounts so game-devs can create lightweight, **limited Discord account** records for unlinked users and use the same APIs whether or not the user has connected Discord. (Figure: Badge when online elsewhere)

### What is a provisional account?

 From the player's view, a provisional account is a **standard account** that has not been connected to Discord yet. It supports expected communication features for a game user. The extra provisional-account complexity that game-developers must handle should not be visible to players. (Figure: What is a provisional account?)

### How provisional accounts look within the game

 In-game, provisional accounts **won't have the Discord badge** next to their username in the friends list or other username contexts because they have not connected Discord. This state is visible only to players with connected Discord accounts who have provisional-account friends from other Discord Social SDK games. (Figure: How provisional accounts look within the game)

### How provisional accounts look within Discord

 Inside Discord, provisional accounts use **branded avatars** and **text-hints** to distinguish them from normal Discord users. Since provisional accounts have limited capabilities, such as no voice or video call support, they must be visually separate from standard Discord users. (Figure: provisional accounts in Discord)

---

### Resources

- [Development Guide: Using Provisional Accounts](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts)

### Change Log

| Date | Changes |
| --- | --- |
| March 17, 2025 | initial release |

## Overall flow

Any in-game imagery used here is fictional and intended only for concept illustration.

### Overall flow

 Every sign-in or log-in flow starts when the player interacts with a [**Discord-styled button**](https://docs.discord.com/developers/discord-social-sdk/design-guidelines/branding-guidelines#buttons). Discord Social SDK supports journeys for users who already **have a Discord account** and users who **do not**. It also supports both users with Discord installed and users without it through the browser. A player may also continue **without** connecting Discord and proceed into the game. (Figure: Overall Flow)

### Connecting your Discord account via Browser (gif)

 (Figure: Connecting your Discord account via Browser)

### Overlay Authorization

 Discord's [Overlay](https://support.discord.com/hc/en-us/articles/217659737-Game-Overlay-101) can support authorization without leaving the game. It is available for games that support this feature. (Figure: Overlay Authorization)

### Before the user connects

 Discord recommends a **one-time user education** modal based on developer and player feedback. The modal should include key value props from Discord Social SDK and appear contextually **in the game**.

#### Suggested Moments

- Trigger on game-start the first time after players receive the Discord Social SDK integration
- Show when the player opens the friends list for the first time
- Show during authorization after the player taps the Discord connection point
 (Figure: Before the user connects)

### After the user connects

 After authorization completes, Discord recommends showing an in-game **success state** styled to match the game's aesthetic.

#### Content

 Include the following points in order using the game's writing-style. Remove any point for features the game does not use.
- Confirm that the Discord account connection succeeded
- Explain that the player can chat in the game and on Discord
- Mention that Discord may need time to show game friends
 (Figure: After the user connects)

### Error State

 If authorization fails, show the user an **error state** that also matches the game's aesthetic. Include these points in order using the game's writing-style:
- State that the Discord account could not be connected
- Provide a call-to-action that returns to Discord and opens the authorization modal again
 (Figure: Error State)

---

### Resources

- [Development Guide: Account Linking with Discord](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord)

### Change Log

| Date | Changes |
| --- | --- |
| March 17, 2025 | initial release |
