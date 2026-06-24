# Social Settings, Status Rich Presence, Unified Friends List

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Covers Evolving with users and Core statuses, plus game-specific presentation. Also includes Figma File - The Social SDK: Friend List Starter Pack.

## Evolving with users

### Evolving with users

 Discord uses player feedback to add settings that let people adjust their social experience. These controls will keep evolving alongside the SDK and Discord client. Discord Social SDK settings are global, so they affect every game that integrates the Discord Social SDK. Players manage them in the Discord client. (Figure: Evolving with users)

### Main entrypoint

 Games must provide an entrypoint that [sends players to the appropriate Discord client settings page](https://docs.discord.com/developers/discord-social-sdk/development-guides/sending-direct-messages#ingame-direct-message-settings). Show it only to players with a connected Discord account. As with authentication, the player is redirected to the Discord client to adjust social settings; if the client is missing, the browser is used instead. (Figure: Main entrypoint)

### Content guidelines

 When referring to these settings in player-facing UI, use the official title "Discord Social Experience." A description is optional, but if the game's settings screen has room, describe what players will find after navigating to Discord. (Figure: Content guidelines)

---

### Resources

- [Development Guide: Direct Messages](https://docs.discord.com/developers/discord-social-sdk/development-guides/sending-direct-messages)

### Change Log

| Date | Changes |
| --- | --- |
| May 08, 2025 | Initial Release |

## Core statuses, plus game-specific

This guide explains design recommendations for adding status and rich presence features to a game. These features help players understand activity in real time and can improve engagement. The guide covers how to:
- Add a status matrix for showing player states in a friends list.
- Use core style rules so status indicators remain consistent and understandable.
- Adapt the visual treatment to the game while preserving the required status-indicator elements.

### Core statuses, plus game-specific

 (Figure: Core statuses, plus game-specific)

### Status matrix

 (Figure: Status matrix)

### Side-by-side

 The **core statuses** (online, idle, DND, offline) always match between the game and the Discord client. Status icons can be customized, but custom icons appear only inside the game. (Figure: Side-by-side)

### Core style guidelines

 To make status meaning easy to read, preserve the shared **core colors, symbols, and information architecture** used by Discord-enabled games, while letting each game define its surrounding visual style. Yellow/green/red should remain recognizable as the status color set, and moon/circle/minus/offline should remain the status symbol set; **all other styling is the game's choice**. Avatars are optional. (Figure: Core style guidelines)

### Rich Presence

 The game developer is responsible for setting rich presence, which appears in both the Discord client and the game client. See [**Setting Rich Presence**](https://docs.discord.com/developers/discord-social-sdk/development-guides/setting-rich-presence) for Rich Presence details, including additional trait types that can be included. (Figure: Rich Presence)

---

### Resources

- [Development Guide: Setting Rich Presence](https://docs.discord.com/developers/discord-social-sdk/development-guides/setting-rich-presence)

### Change Log

| Date | Changes |
| --- | --- |
| March 17, 2025 | Initial Release |

## Figma File - The Social SDK: Friend List Starter Pack

This guide gives design recommendations for adding a unified friends list to a game. A unified friends list lets players view and interact with friends across games and platforms, improving the social experience. The guide covers how to:
- Use sections to group friends by availability and game state.
- Show the Discord badge where it communicates cross-platform messaging and invites.
- Choose which identity to display based on the player's relationship to the game and Discord.
- Keep naming familiar and consistent so players are less likely to be confused.
 These design guidelines help create an intuitive friends list that uses Discord social features effectively.

---

### Figma File - The Social SDK: Friend List Starter Pack

 Discord provides a Figma resource with a templated design system for building a game's friends list. Update components to match the game's visual language and preview those changes live. The file includes common use cases and an example from SUPERVIVE, a partnered game. Feedback is welcome in the [Discord Developers Server](https://discord.gg/discord-developers); create a post in `#social-sdk-dev-help` for requests or suggestions. [(Figure: starter pack)](https://www.figma.com/community/file/1512487996808869592/the-social-sdk-friend-list-starter-pack) [See Figma Resource->](https://www.figma.com/community/file/1512487996808869592/the-social-sdk-friend-list-starter-pack)

---

### Sectioning

 Order sections by decreasing availability so users can quickly find friends available to play. **Online - GameTitle** contains friends who are online in the same game and compatible for play. **Online - Elsewhere** contains online friends outside the same game, who can still receive messages or game invites. (Figure: Sectioning Friends List)

### Discord = Communication

 Show the Discord badge next to a player's name only when the player is **online elsewhere** and has **connected their Discord** account. Do not show the badge for provisional accounts. The Discord logo communicates **universal communication**: users should understand that they can message or invite that friend anywhere Discord is available, including phone, computer, and console. (Figure: Discord logo represents universal communication)

### Identities

 When a friend **owns the game**, show the in-game identity, such as the username, because it is the name players already recognize. If the friend does not own the game, show their **Discord Display Name**. If the friend neither owns the game nor has a linked Discord account, use their **Provisional identity**. (Figure: Displaying different relationships)

### Examples of friends lists

 A player's **console ID** effectively serves as their **game ID**, because consoles require it. That is likely the name set players recognize most readily. The examples on the right show how this hierarchy applies on consoles. (Figure: Examples of friends lists)

### How did we land here?

 This approach limits unexpected **name changes** in the game, which can confuse players and cause unstable UI widths. It also scales well and remains **inclusive** for provisional accounts without connected Discord accounts. (Figure: How did we land here?)

### Discord Display Names

 When mentioning a player's Discord Identity, use the **Discord Display Name** rather than the username. For text chat and friends lists, see [How to Manage Special Characters in Discord Display Names](https://docs.discord.com/developers/discord-social-sdk/how-to/handle-special-characters-display-names). (Figure: Discord display names)

### Scaling to Multiple Platforms

 For games with friend graphs from multiple platforms, merge them into **Online Elsewhere**. Put available players higher in the friends list regardless of platform. Do not organize friends lists primarily by company, because that weakens the value of surfacing online players first in favor of platform separation. (Figure: Scaling to Multiple Platforms)

---

### Resources

- [Development Guide: Creating a Unified Friends List](https://docs.discord.com/developers/discord-social-sdk/development-guides/creating-a-unified-friends-list)
- [Figma File - The Social SDK: Friend List Starter Pack ->](https://www.figma.com/community/file/1512487996808869592/the-social-sdk-friend-list-starter-pack)

### Change Log

| Date | Changes |
| --- | --- |
| June 25, 2025 | Added Figma UFL starter pack |
| March 17, 2025 | initial release |
