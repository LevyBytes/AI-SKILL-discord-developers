# Overview Of Apps

> Discord Developers reference. Original prose; identifiers preserved verbatim.

A **Discord application** is the primary record for an integration on the Discord platform. Bots, Activities, and Social SDK integrations all use an application created in the [Discord Developer Portal](https://discord.com/developers/applications). The application stores credentials, OAuth2 configuration, bot settings, and metadata. Whether the project is a small command-response bot or a game integration with rich presence and voice chat, the first step is creating an application.

### The Three Types of Discord Apps

### Bots

 Bots are automated accounts controlled by your application code. They show up in servers with an `APP` tag and can handle events, slash commands, and user actions. Choose bots when you need to:
- Provide server commands, moderation features, or utility tools
- Process events such as messages, member joins, or reactions as they happen
- Create interactive flows that use buttons, menus, and modals
- Publish automatic updates from outside services
 Bots may be **guild-installed**, which adds them to one server, or **user-installed**, which makes them available wherever that user can use them without server-level permissions.

### Activities

 Activities are web apps embedded inside Discord channels. They use the [Embedded App SDK](https://docs.discord.com/developers/developer-tools/embedded-app-sdk), and users can launch them in a channel or a bot can launch them with the `LAUNCH_ACTIVITY` interaction callback. Pick Activities when you want to:
- Create a game or other interactive experience that runs within Discord
- Create collaborative tools (watch parties, drawing apps, trivia games)
- Leverage Discord’s existing social context: voice, friends, servers

### Social Layer for Games

 The [Discord Social SDK](https://docs.discord.com/developers/discord-social-sdk/overview) adds Discord-based social capabilities to a game on PC, mobile, and console. You can adopt specific features beside your own social systems, or build a fuller experience around Discord. Use the Social SDK when you want to:
- Add rich presence, friends lists, and voice chat to your game
- Use Discord account linking for cross-platform social features
- Allow in-game player interaction through Discord

### Where Apps Install

| App Type | Install Target | What It Unlocks |
| --- | --- | --- |
| Bot (guild-installed) | A server | Access to channels, members, and events in a server |
| Bot (user-installed) | A user | Usable in DMs, group DMs, and servers |
| Activity | An application | Launchable from any voice channel |
| Social SDK | Your game | Game-client social functionality |

### What APIs Can Apps Use?

 Discord's developer platform exposes several APIs, and an app can use the ones that match its features and the Discord surfaces it needs. This section gives a short map of the main APIs; detailed behavior and usage rules are covered in the [API reference](https://docs.discord.com/developers/reference).

### HTTP API

 The **HTTP API** is a REST API for reading or changing core Discord resources such as [channels](https://docs.discord.com/developers/resources/channel), [servers (or guilds)](https://docs.discord.com/developers/resources/guild), [users](https://docs.discord.com/developers/resources/user), and [messages](https://docs.discord.com/developers/resources/message#message-object). Use the HTTP API to:
- Retrieve information about a resource
- Create, update, or delete a resource
 See the [API reference](https://docs.discord.com/developers/reference#http-api) for HTTP API usage details.

### Gateway API

 The **Gateway API** is a WebSocket connection linking your app to Discord. It is commonly used for a live event stream from Discord, but traffic can move both ways, so the same connection can also send events from your app to Discord. Use the Gateway API to:
- Receive Discord events in real time, including messages, member joins, reactions, voice state changes, and more
- Set your bot's presence and status
- Control voice state and join voice channels
- Complete the connection handshake and maintain the Gateway connection lifecycle
 See the [API reference](https://docs.discord.com/developers/reference#gateway-websocket-api) for Gateway API usage details.

### Start Building

### Bots & Companion Apps

Create a bot that handles server events, commands, and user interactions in Discord.

### Social Layer for Games

Use the Discord Social SDK to add Discord-based social capabilities to a game.

### Activities

Create an embedded web app or game that runs in Discord voice channels.
