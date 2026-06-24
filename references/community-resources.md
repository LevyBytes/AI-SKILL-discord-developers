# Community Resources

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Discord community members publish tools and resources that make app development and maintenance easier. The ecosystem includes permissions calculators, embed preview tools, and broad API libraries that can reduce the effort required to build against Discord.

### Discord Developers

 The [Official Discord Developers server](https://discord.gg/discord-developers) is a community-driven support space run by developers. Use it for help building on Discord and for official updates from Discord's developer team.

### Libraries

 Discord does not provide official SDKs. The table below is a non-exhaustive set of third-party libraries with valid rate limit handling, recent maintenance, and sizable active-bot communities.

###### Discord Libraries

| Name | Language |
| --- | --- |
| Concord | C |
| Discord.Net | C# |
| DSharpPlus | C# |
| D++ | C++ |
| discljord | Clojure |
| DiscordGo | Go |
| Discord4J | Java |
| JDA | Java |
| discord.js | JavaScript |
| Eris | JavaScript |
| Oceanic | JavaScript |
| Discordia | Lua |
| DiscordPHP | PHP |
| discord.py | Python |
| disnake | Python |
| hikari | Python |
| interactions.py | Python |
| nextcord | Python |
| pycord | Python |
| discordrb | Ruby |
| Serenity | Rust |

### Interactions

 [Interactions](https://docs.discord.com/developers/interactions/receiving-and-responding) are a modern way to build Discord bots. The open-source libraries below help with the required security and authentication validation for Interactions delivered through outgoing webhooks, and they also include some Interactions data model types.
- C#
  - [Discord.Net.Rest](https://github.com/discord-net/Discord.Net)
  - [DSharpPlus.Http.AspNetCore](https://github.com/DSharpPlus/DSharpPlus)

- Clojure
  - [ring-discord-auth](https://github.com/JohnnyJayJay/ring-discord-auth)

- Dart
  - [nyxx_interactions](https://github.com/l7ssha/Nyxx)

- Go
  - [tempest](https://github.com/amatsagu/tempest)

- Javascript
  - [discord-interactions-js](https://github.com/discord/discord-interactions-js)
  - [discord-slash-commands](https://github.com/MeguminSama/discord-slash-commands) and its [Deno fork](https://deno.land/x/discord_slash_commands)
  - [slash-create](https://github.com/Snazzah/slash-create)

- Python
  - [discord-interactions-python](https://github.com/discord/discord-interactions-python)
  - [discord-interactions.py](https://github.com/LiBa001/discord-interactions.py)
  - [dispike](https://github.com/ms7m/dispike)
  - [flask-discord-interactions](https://github.com/breqdev/flask-discord-interactions)

- PHP
  - [discord-interactions-php](https://github.com/discord/discord-interactions-php)

- Other
  - [caddy-discord-interactions-verifier](https://github.com/CarsonHoffman/caddy-discord-interactions-verifier)
  - [BotForge’s Application Commands Builder & Previewer](https://tools.botforge.org/appbuilder)
  - [Bsati’s Slash Command Builder](https://bsati.github.io/dc-app-command-builder/)

### OpenAPI Specification

 The OpenAPI spec is public-preview material, so **breaking changes can still occur**. Through that preview, the [Discord HTTP API specification](https://github.com/discord/discord-api-spec) publishes the HTTP API as a standard [OpenAPI 3.1 spec](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md).

### Permission Calculators

 Discord [Permissions](https://docs.discord.com/developers/topics/permissions) can be difficult to calculate correctly. If you are building a bot for other users and need help computing permissions or generating an [authorization URL](https://docs.discord.com/developers/topics/oauth2#bot-authorization-flow), these calculators can help:
- [BotForge’s Permissions Calculator](https://tools.botforge.org/permissions)
- [FiniteReality’s Permissions Calculator](https://finitereality.github.io/permissions-calculator/?v=0)
- [abalabahaha’s Permissions Calculator](https://discordapi.com/permissions.html#0)

### Intent Calculators

 [Gateway Intents](https://docs.discord.com/developers/events/gateway#gateway-intents) can be hard to reason about at first. If you are unsure what belongs in the [Identify payload](https://docs.discord.com/developers/events/gateway-events#identify), these tools may help:
- [ziad87’s Intent Calculator](https://ziad87.net/intents/)
- [Larko’s Intent Calculator](https://discord-intents-calculator.vercel.app/)

### Embed Visualizers

 Webhooks and embeds are easier to work with when you can preview them. These tools let you test how embeds will render inside Discord:
- [JohnyTheCarrot’s Embed Previewer](https://github.com/JohnyTheCarrot/discord-embed-previewer) (Browser Extension)
- [AshMW’s Embed Previewer](https://embedl.ink) (Embed HTML Generation)

### API Types

 Projects that call the Discord API may benefit from an API types module, which provides type inspection and completion for Discord API structures.
| Name | Language |
| --- | --- |
| dasgo | Go |
| discord-api-types | JavaScript |

### Game SDK Tools

 Discord Game SDK's lobby and networking layer is similar to layers in other game platforms, including Valve's Steamworks SDK. The open source library below offers a shared interface for those overlapping features and can simplify multi-platform development. Note: this library is tailored for Unity3D development.
- [HouraiNetworking](https://github.com/HouraiTeahouse/HouraiNetworking)
