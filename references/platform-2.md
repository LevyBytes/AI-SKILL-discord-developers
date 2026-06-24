# Components, Discovery, Game Profiles, Interactions ...

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Scope in this file: components and modals, discovery surfaces, game profiles, command entry points, authentication choices, SDK selection, app capabilities, account linking, and incoming webhook usage.

## Message Components

(Figure: Discord Bots & Companion Apps Hero Image) Discord components let an app render rich interactive UI inside the Discord client without building a separate custom frontend.

### Message Components

 Message Components are layout and interaction elements attached to messages sent by an app. Discord groups them into three categories:
- **Layout components** for organizing content
- **Content components** that show text and media
- **Interactive components** such as buttons, select menus, and text inputs that users can operate
 Components are delivered with the message as structured data, then rendered natively by the Discord client.

### Modals

 Modals are form-style overlays an app can show after an interaction, such as a button click, slash command, or select menu choice. They gather freeform text input and work well for multi-field entry, confirmations, bug reports, and configuration paths that need more than one tap. Modals are not attached to messages like message components. They are sent as interaction responses, appear as focused client overlays, and produce a new interaction when the user submits the form.

### Further Reading

### Components Overview

Complete guide to component types and how components are included in messages.

### Using Message Components

Guide for sending Message Components, including code examples.

### Using Modal Components

Guide for creating and showing Modal Components, including code examples.

## App Directory

App Discovery gives users several paths to find and install an app across Discord.

### App Directory

 The [App Directory](https://discord.com/discovery/applications) is a searchable hub where users browse apps by name, category, or collection and open product pages with descriptions, screenshots, and links. To be listed there, an app must be verified and opted into discovery. See [Enabling Discovery](https://docs.discord.com/developers/discovery/enabling-discovery) for setup steps.

### App Launcher

 The [App Launcher](https://support-apps.discord.com/hc/en-us/articles/26592957841303-How-to-Discover-and-Add-Apps#h_01J8JE2VM7QEKH9XKNR696P56C) opens from the app shapes icon around Discord and highlights recently used apps plus curated collections.

### Organic Discovery & Sharing

 After an app becomes discoverable, Discord social activity can help users find it. When someone uses the app in a server, others in that server can see it, open its profile, and install it. Each Discord app also has shareable links to its profile or [store page](https://docs.discord.com/developers/monetization/managing-skus#linking-to-your-store), and those links can be posted in Discord messages or on external sites.

### Next Steps

### Discovery Overview

Complete guide to discovery surfaces, including the App Directory, App Launcher, and social discovery.

### Enabling Discovery

Enable discovery so the app can appear in the App Directory and App Launcher.

### Discovery Best Practices

Advice for helping an app stand out and attract more users through discovery.

## Where Game Profiles Appear

Discord creates profiles for detected games people play. A game profile is a dedicated page containing the game's description, screenshots, and social links, helping users learn about friends' games or find something new to play.

### Where Game Profiles Appear

 Discord shows game profiles across multiple platform surfaces so users can discover and learn about games.

### Rich Presence

 When someone is playing a game, the activity appears on their profile and in the member list. Selecting the activity and then the game's cover art opens the game profile page.

### Game Collections in User Profiles

 Users can add profile widgets that curate lists of games. Selecting a game's cover art from any widget opens that game's profile page.

### Activity

 The **Activity** tab on a user's profile lists recently played games. Selecting a game's cover art opens its profile page.

### Game Profile Page

 The full game profile page organizes information into four areas:
- **Title:** description, genres, platforms
- **Visual Assets:** cover, icon, banner, screenshots
- **Links:** official website, social media
- **Company:** name plus publisher or developer role

---

### Take Control of Your Game Profile

 Game developers and publishers can claim ownership of a Discord game profile. Claiming a game lets you:
- Update your game description
- Choose screenshots that best represent the game
- Keep publisher and developer names accurate
- Link players to the official Discord community and social channels
- Add an official game community checkmark to the Discord server
 Profile edits publish immediately without extra review, allowing updates for new content, patches, and seasonal events.

### Claim Your Game

Guide for claiming a game on Discord and editing the game's profile.

## Application Commands

Interactions power user-facing Discord app features. When a user runs a slash command, presses a button, chooses from a dropdown, or submits a modal, Discord sends the app an interaction payload that the app can answer directly. Apps receive interactions either over the Gateway or through HTTP requests to a configured endpoint. Both delivery models can support anything from a small command handler to a larger interactive system.

### Application Commands

 [Application Commands](https://docs.discord.com/developers/interactions/application-commands) are the main way users invoke an app in Discord. The four command types are:
- **Slash commands:** opened by typing `/` in chat input
- **Message commands:** opened from a message context menu
- **User commands:** opened from a user profile context menu
- **Entry Point commands:** launch [Activities](https://docs.discord.com/developers/platform/activities) from the App Launcher
 When a command is invoked, Discord sends an interaction payload that the app can answer with a message, modal, or deferred response for longer work.

### Message Components & Modals

 [Message components](https://docs.discord.com/developers/platform/components) also create interactions when users use them. A button click or select menu submission on an app message is delivered like a command interaction. [Modals](https://docs.discord.com/developers/platform/components#modals) follow the same model: the app responds to an interaction by opening a modal, and the submitted modal becomes another interaction for the app to handle.

### Gateway Interactions

 By default, apps receive interactions through the [Gateway](https://docs.discord.com/developers/events/gateway), the persistent WebSocket connection used for other real-time events. If the bot is already connected to listen for messages, member joins, or similar activity, interactions arrive over that same connection.

### HTTP Interactions

 Alternatively, set an **Interactions Endpoint URL** in the app settings to receive interactions as HTTP POST requests directly on your server. Discord delivers the requests, and the app must respond within a few seconds. This model fits apps that only answer commands and UI components, requires no persistent connection, and scales with ordinary web infrastructure. The endpoint must validate Discord request signatures and handle the initial `PING` handshake. See [Interactions Overview](https://docs.discord.com/developers/interactions/overview#preparing-for-interactions) for setup details.

### Further Reading

### Interactions Overview

Complete guide to interaction types, endpoint setup, and security checks.

### Application Commands

Create application commands of slash, message, user, and entry point types.

### Receiving & Responding

Technical reference for receiving interaction payloads and sending responses.

## Two Ways to Authenticate

Discord uses [OAuth2](https://docs.discord.com/developers/topics/oauth2) to authorize app access to users and servers. Any Discord app should understand OAuth2 and the permissions model.

### Two Ways to Authenticate

### Bot Token

 Creating a bot user in the Developer Portal generates a **bot token**. Code uses that token to authenticate as the bot user, which is a dedicated application account separate from a regular user. **Bot tokens:**
- Authenticate as the bot user rather than on behalf of a person
- Are used for Gateway connections and most REST API requests
- Carry the permissions granted when the bot was added to a server
- Must be protected like passwords and never exposed publicly

### OAuth2 User Token

 To act for a user, use OAuth2 to obtain a user access token. The user signs in with Discord and authorizes the app for chosen scopes. **User tokens:**
- Allow the app to read or write data for the user within the requested scopes
- Are scoped, limiting access to what the user approved
- Expire quickly and require refresh
- Must be protected like passwords and never exposed publicly

### Scopes

 Scopes describe what an app may do. They are requested during OAuth2 authorization and must be declared in the Developer Portal. Common scopes include:
| Scope | What It Grants |
| --- | --- |
| `bot` | Adds the bot to a guild |
| `identify` | Reads the user's basic profile, such as id, username, and avatar |
| `guilds` | Lists guilds the user belongs to |
| `guilds.join` | Adds the user to a guild |
| `email` | Reads the user's email address |
| `connections` | Views the user's linked accounts, such as Twitch, Steam, and others |
| `applications.commands` | Registers slash commands in a guild |

 See the [OAuth2 scopes reference](https://docs.discord.com/developers/topics/oauth2#shared-resources) for the complete list.

### Permissions

 Permissions define the actions a **bot** may perform at server scope or within individual channels. When a bot is installed through OAuth2, the server admin grants permissions that are stored as a bitfield. Permissions can be:
- **Guild-level:** Apply across the whole server
- **Channel-level:** Override behavior for selected channels
 Request only the permissions the app needs. Overly broad permissions reduce user trust and should be avoided under the [Developer Policy](https://support-dev.discord.com/hc/en-us/articles/8563934450327-Discord-Developer-Policy).

### Further Reading

### OAuth2 Reference

Complete OAuth2 flow documentation, scopes, and endpoint details.

### Permissions Reference

Complete permissions bitfield reference and explanation of overrides.

## Choosing an SDK

(Figure: Rich Presence) Rich Presence shows live, actionable game state on a Discord user's profile when activity sharing is enabled, such as the game being played, level, session duration, or an invitation for friends to join. It connects an app to Discord's social layer with a lightweight integration. The displayed data is controlled by the developer and can include custom text, timestamps, party size indicators, and uploaded art. With a 'Join' button, friends can jump from the profile card into that player's active game session. Native games use the Discord Social SDK, while Activities inside Discord use the Embedded App SDK.

### Choosing an SDK

 Rich Presence can be integrated through two SDK choices. Pick the SDK that matches the project:
- The **[Discord Social SDK](https://docs.discord.com/developers/discord-social-sdk/overview)** adds social features to a game or app, including friends lists, game invites, and more.
- The **[Embedded App SDK](https://docs.discord.com/developers/developer-tools/embedded-app-sdk)** is the right choice for building an [Activity](https://docs.discord.com/developers/activities/overview) in Discord.
 Because Rich Presence appears publicly on a Discord profile, development should use a test account that belongs only to private development server(s).

### Discord Social SDK

 The Discord Social SDK is for adding social features directly to a game. When a user plays, the game can show activity on the user's Discord profile and offer game invites so friends can join. For off-platform games, Rich Presence can describe the player's current game state, and a 'Join' button can let friends jump into the session and send game invites. [Learn how to implement Rich Presence with the Discord Social SDK](https://docs.discord.com/developers/discord-social-sdk/development-guides/setting-rich-presence)

### Embedded App SDK

 The Embedded App SDK supports Activities, which are multiplayer games and social experiences hosted in an iframe inside Discord. The SDK manages communication between Discord and the Activity and helps use platform features such as Rich Presence. After a user joins an Activity, Rich Presence can update dynamically with what the user is doing or playing and invite others to join. [Learn how to implement Rich Presence with the Embedded App SDK](https://docs.discord.com/developers/rich-presence/using-with-the-embedded-app-sdk)

### Start Building

### Using Rich Presence with the Embedded App SDK

Guide for using Rich Presence with the Embedded App SDK while building an Activity

### Using Rich Presence with the Discord Social SDK

Guide for using Rich Presence in the Discord Social SDK for a game

### Rich Presence Best Practices

Design guidance and launch checklist for a polished Rich Presence integration.

## What Apps Can Do

Discord apps can use APIs to interact with Discord servers (guilds) and channels: reading content, sending messages, managing members, creating channels, and more. Most actions require the bot to have the needed permissions in the target server or channel.

### What Apps Can Do

 Server and channel management can include the following capabilities. The [API Reference](https://docs.discord.com/developers/reference) lists the full endpoint and capability set.

### Guilds

- Read guild metadata: name, icon, member count, features
- List and manage roles, including creating, updating, deleting, and assigning them to members
- Kick and ban members
- Manage and audit server events
- List guild integrations and webhooks

### Channels

- List channels and read them across the server
- Create, update, and remove channels and threads
- Set channel permissions and position
- Read and send messages when message permissions allow it
- Manage pinned messages

### Members

- Retrieve member profiles and assigned roles
- Add roles to members or remove roles from members
- Modify member nicknames
- Manage member timeouts

### Key Permissions

 Most server and channel actions require explicit permissions. The [Permissions](https://docs.discord.com/developers/topics/permissions) reference lists all permissions. Common examples include:
| Permission | What It Enables |
| --- | --- |
| `MANAGE_GUILD` | Edit server settings, view audit log |
| `MANAGE_CHANNELS` | Create, edit, delete channels |
| `MANAGE_ROLES` | Create and manage roles below the bot's highest role |
| `KICK_MEMBERS` | Remove members from the server |
| `BAN_MEMBERS` | Permanently ban members |
| `MANAGE_MESSAGES` | Delete other users' messages and pin messages |
| `VIEW_CHANNEL` | View a channel and its message history |
| `SEND_MESSAGES` | Post messages in a channel |

 Channel-level overrides can change permission behavior, so a bot with server-wide access may still be blocked in a channel, or a channel may grant access not available server-wide.

### Further Reading

### Guild Resource

Complete API reference for guild objects and endpoints.

### Channel Resource

Complete API reference for channel endpoints, sending messages, and threads.

## Account Linking

*Already familiar? Go to the [Discord Social SDK docs](https://docs.discord.com/developers/discord-social-sdk/overview) or [Getting Started guides](https://docs.discord.com/developers/discord-social-sdk/getting-started).* Social Layer for Games is a collection of Discord-powered features that developers add to games with the [Discord Social SDK](https://docs.discord.com/developers/discord-social-sdk/overview). You can add selected features alongside an existing social system or build a broader Discord-powered social experience. Discord provides SDK paths for C++, Unity, and Unreal Engine projects targeting PC, mobile devices, or consoles.

### Account Linking

 Players can [link their Discord account](https://docs.discord.com/developers/platform/account-linking) to a game to access the full social experience. After linking, they can use Discord friends list, game invites, rich presence, and voice inside the game. The link persists across sessions, so players authenticate once instead of linking again. Account linking supports Game Flow from inside the game through the Social SDK and Web Flow through a normal OAuth2 page.

### Provisional Accounts

 Players who do not want to link Discord can still use SDK social features through [provisional accounts](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts). This lets the game offer a consistent social experience whether or not a player has or wants a Discord account.

### Friends List

 The SDK friends list works for players with linked Discord accounts and players using provisional accounts. Provisional-account players can manage in-game friends, while linked players also see Discord friends next to in-game friends.

### Rich Presence

 [Rich Presence](https://docs.discord.com/developers/platform/rich-presence) displays a player's in-game activity on their Discord profile when activity sharing is enabled. Friends can see the game, mode, map, and open party slots, then request to join with one click. This supports organic discovery and creates a direct path into the game.

### Game Invites

 Players can invite Discord friends into a game session with one click. The invite appears in Discord and deep-links back into the game, making parties and shared sessions easier to fill.

### Direct Messages

 Players can send direct messages to Discord friends from inside the game, keeping communication in the game context.

### Lobbies and Voice Chat

 The SDK offers lobby-based voice and text chat that the game can create and manage through code. Players can talk inside a game session, party, or any group the game defines while using Discord voice infrastructure without separately opening the Discord app.

### Further Reading

### Social SDK Overview

Complete introduction to the SDK, its feature set, and starting points.

### Core Concepts

Explains how the SDK works, its core features, and platform compatibility.

### Getting Started

Guides that walk through C++, Unity, and Unreal Engine setup step by step.

## Incoming Webhooks

Discord has two webhook types that move data in opposite directions: **incoming webhooks**, which post messages into Discord, and **webhook events**, which notify your server when app-related events happen in Discord.

### Incoming Webhooks

 Incoming webhooks are HTTP endpoints attached to a Discord channel. You send a POST payload to the URL, and Discord posts the message in that channel. They do not require a bot account or persistent connection. See the [Webhook Resource](https://docs.discord.com/developers/resources/webhook) for the full API reference. They fit one-way integrations where an external system needs to push information into Discord:
- CI/CD pipelines posting build results or deployment notices
- Monitoring systems sending alerts for errors or anomalies
- External apps posting updates (GitHub commits, Jira tickets, form submissions)
- Scheduled digests or reports generated by cron jobs
 If the app also needs to listen for events or answer users, [building a Discord bot](https://docs.discord.com/developers/platform/bots) is a better match.

### Webhook Events

 Webhook events send data the other way: Discord POSTs to a public server URL when selected app events occur. Instead of polling, the app receives pushed notifications. Subscribe to event types in the Developer Portal, and Discord sends signed HTTP POST requests to the endpoint. The endpoint must validate Discord request signatures and acknowledge events quickly. Subscribable events include:
- A user authorizes or de-authorizes the application
- Entitlement created when a user purchased something from the app
- Quest enrollment and completion events
 See [Webhook Events](https://docs.discord.com/developers/events/webhook-events) for the complete event list and setup instructions.

### Further Reading

### Webhook Resource Reference

Complete API reference for incoming webhook objects and endpoints.

### Webhook Events

Subscribable event types and endpoint setup for receiving them.
