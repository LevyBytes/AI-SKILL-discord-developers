# Bots — Overview

> Discord Developers reference. Original prose; identifiers preserved verbatim.

(Figure: Discord Bots & Companion Apps Hero Image) A bot is an application that uses the Discord API to join Discord. In servers, it is shown like a user account with an `APP` label, and it can react to member activity, handle events, and carry out automated actions. For a broader explanation of bot capabilities and appropriate use cases, see [Bots & Companion Apps](https://docs.discord.com/developers/platform/bots).

### Getting Started

 Building a bot requires these setup steps:
- **Create an application** in the Discord Developer Portal, which stores the bot token, OAuth2 credentials, and app settings.
- **Add a bot user** on the application's Bot tab so Discord issues the token your program authenticates with.
- **Invite the bot** into a server through an OAuth2 URL that includes the bot scope plus the permissions it needs.
- **Connect to the API** through the Gateway for live events, the HTTP API for REST calls, or a combination of the two.
 Most projects use an established community library for their programming language instead of calling the API primitives by hand. Discord lists official and community options on [Community Resources](https://docs.discord.com/developers/developer-tools/community-resources). To start with a complete first project, follow the [Getting Started guide](https://docs.discord.com/developers/quick-start/getting-started), which covers creating and running an initial bot.

### Building Your Bot

- **[Interactions](https://docs.discord.com/developers/interactions/overview)** - Slash commands, context menu commands, buttons, select menus, and modals that provide user-facing ways to work with the bot.
- **[Components](https://docs.discord.com/developers/components/overview)** - Message and modal building blocks for layout, content, and interactive controls.
- **[Monetization](https://docs.discord.com/developers/monetization/overview)** - Use Discord's payment flow to offer subscriptions and one-time purchases from the bot.
- **[Discovery](https://docs.discord.com/developers/discovery/overview)** - Make the app visible in the App Directory and App Launcher so people can discover and install it.
- **[Guides & Tutorials](https://docs.discord.com/developers/guides/bots)** - Hands-on material for common bot capabilities and implementation practices.

### Build Your First App

A walkthrough for creating, launching, and testing an initial Discord bot.

### Bot Development Guides

Browse practical bot guides covering common features and recommended practices.

[Building your first Discord BotNext](https://docs.discord.com/developers/quick-start/getting-started)⌘I
