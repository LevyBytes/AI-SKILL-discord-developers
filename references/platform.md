# Account Linking, Activities, App Monetization, Bots ...

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Scope in this file: linking player accounts, Discord multiplayer features, monetization surfaces, bot entry points, game claiming, and community-server setup.

## What is Account Linking?

(Figure: Account Linking Hero Image) *Already know the basics? Go straight to the implementation guides or the [Social SDK docs](https://docs.discord.com/developers/discord-social-sdk/overview).*

### What is Account Linking?

 Account linking associates a player's Discord account with their in-game account. After the connection is made, the game can surface Discord friends, invites, rich presence, and voice without leaving the game. The connection remains available between sessions, so the player signs in once and only has to link again if your account linking [scopes](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes) are changed. Players who are not ready to connect Discord, or who do not yet have a Discord account, can still use social systems through [provisional accounts](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts). If they connect Discord later, their friends, DM history, and lobby memberships are carried over to the Discord account automatically. Account linking is a key requirement for [Social Commerce](https://docs.discord.com/developers/social-commerce/overview) in Discord communities because linked players can redeem in-game items bought or gifted through Discord Game Shops.

### Why Account Linking Matters

### For Players

 For players, account linking turns Discord into an in-game social layer:
- **Seamless social experiences:** Use Discord friends and game invites from inside the game.
- **Rich presence integration:** Share detailed status and activity information with friends.
- **Social Commerce:** Redeem in-game items bought or gifted through Discord Game Shops.

### For Developers

 For developers, linked players show stronger game value:
- Players who connect in-game accounts to Discord through Social SDK **play more often** - median game launch days are up 25%*
- Players who connect in-game accounts to Discord through the Social SDK **play longer** - median session length is up 16%*
- Players who connect in-game accounts to Discord through the Social SDK **retain better** - median D28 retention reaches 34%*
 Those results come from causal data for games that adopted account linking. **Discord Internal Data, 2025*

### How Account Linking Works

### Understanding the OAuth2 Flow

 Account linking uses OAuth2, the standard authorization protocol. A player link follows this sequence:
1. **Your game requests that Discord begin the OAuth flow.**
2. **Discord presents an OAuth modal where the player approves your game and the requested OAuth scopes.**
3. **The player approves** (authorizes).
4. **Discord returns a temporary access token** that proves the player granted authorization.
5. **Your game uses that token** to call Discord social features for the player.
 You do not have to build OAuth2 yourself. The Discord Social SDK manages authorization, tokens, and API calls for the game. Web Flow uses standard OAuth2, so most web frameworks already have suitable support. The access token is scoped, which limits it to the Discord capabilities your game requested. For common setups, the SDK exposes `GetDefaultPresenceScopes` to cover friends plus presence and `GetDefaultCommunicationScopes` to cover voice, lobbies, and messaging. Read the [OAuth2 Scopes for Social SDK page](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes) for the complete scope details. (Figure: Account Linking in Action)

### Two Ways to Link

 Account linking can be implemented through two flows:
|  | **Game Flow** | **Web Flow** |
| --- | --- | --- |
| **What it is** | The player authorizes from your game through the Discord client, overlay, or browser | The player authorizes on a web page from any device |
| **Requires** | Discord Social SDK | Standard OAuth2 implementation |
| **Platforms** | Desktop, Mobile, Console | All platforms that support web |
| **Player experience** | Keeps the player in game context; the Discord app or overlay opens, with browser fallback | Opens a browser / web page |

 Game Flow and Web Flow both link the player's Discord account to your game. If they request the same OAuth2 scopes, the player receives the same features no matter which flow was used. Offering both gives players more ways to connect. Game Flow requires the player to be running your game on the same device. Web Flow is more portable and works from web-based touchpoints such as an Official Game Community, Game Shop, or external URL. A player linked through one method is already linked for the other, so you can add the second flow later without asking players to connect again. Web Flow is not an embedded browser inside your game; it is a separate web OAuth2 flow, usually reached from Discord entry points or outside URLs.

### Implementing Account Linking

### Linking from Your Game

 Use the Discord Social SDK when players should link accounts from inside the game. Pick the platform guide that matches your target:

### Desktop (Windows, Mac, Linux)

Guide for adding Social SDK account linking to a desktop game.

### Mobile (iOS, Android)

Guide for adding Social SDK account linking to a mobile game.

### Console (PlayStation, Xbox)

Guide for adding Social SDK account linking to a console game.

### Linking from the Web

 Use a standard OAuth2 flow when players should link accounts from a web page on any device. The Social SDK is not required, which fits teams that already operate web infrastructure. Web Flow is the main route for [Social Commerce](https://docs.discord.com/developers/social-commerce/overview) features such as Game Shops and Game Stat Widgets.

### OAuth2 Documentation

Guide for building the standard OAuth2 web account-linking flow.

### Linking from Discord Entry Points

 Discord entry points are client-side buttons and prompts that invite players to link their game account even when the game is not open. Links started from Discord behave the same as links started from the game. (Figure: Account Linking from Discord Entry Points) When a player selects an entry point, Discord chooses the flow this way:
1. **Game Flow** runs when the game is open and the developer has called `RegisterAuthorizeRequestCallback` in the Social SDK
2. **Web Flow** runs when the Developer Portal has a **Connection Entrypoint URL** configured
 Current Discord entry points include: **Game Detection** Discord notices that the player is playing your game and can offer linking if the account is not linked yet. (Figure: Discord entry points for account linking) **Game Invites** Players receiving an invite can be prompted to link so they can get into the game with friends faster. (Figure: Game Invite entry points for account linking) **Claiming Game Shop Items** Players must link accounts to redeem Game Shop items and receive their in-game rewards. (Figure: Game Shop entry points for account linking) Web Flow support from official Discord entry points is currently limited to select partners. Contact Discord if you want it enabled for your game.

### Account Linking from Discord Entry Points

Guide for enabling Discord entry points that let players link inside the Discord client.

### Provisional Accounts

 Players without Discord, or players who postpone linking, can still participate through provisional accounts. Those temporary accounts support friendships, lobby access, messages, and voice without requiring a Discord account first. After the player connects Discord, the existing friends, DM history, and lobby memberships are migrated automatically. Read the [Provisional Accounts guide](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts) for details.

### Next Steps

### Social Layer for Games

Explore the Discord-backed social features that account linking makes available to your game.

### Design Guidelines

Best practices for shaping the account-linking experience inside your game.

### Getting Started

Step-by-step integration guides for the Discord Social SDK in C++, Unity, and Unreal Engine.

## Multiplayer Experiences in Discord

*Already know this area? Jump to the [Activities docs](https://docs.discord.com/developers/activities/overview) or [Building Your First Activity](https://docs.discord.com/developers/activities/building-an-activity).*

### Multiplayer Experiences in Discord

 Activities are multiplayer games and social experiences that run directly inside Discord. Users can launch them from channels, DMs, or the App Launcher without opening a separate window or installing a separate download. Because friends can start together from an existing voice channel, Activities work well for party games, watch-together sessions, collaboration tools, and similar shared experiences. (Figure: Discord Activities)

### Embedded App SDK for Activities

 Technically, an Activity is an iframe that talks to Discord through the [Embedded App SDK](https://docs.discord.com/developers/developer-tools/embedded-app-sdk). The SDK exposes methods your Activity can use to interact with the Discord client.

### Next Steps

### Activities Overview

Complete guide to Activity behavior, launch paths, and the Embedded App SDK.

### How Activities Work

Detailed guide to the Activity lifecycle, iframe runtime, and client messaging.

### Build Your First Activity

Walkthrough for creating, configuring, and deploying a first Activity.

## Monetization Overview

(Figure: Monetizing Your Discord App Hero Image) App Monetization supports [Bots](https://docs.discord.com/developers/bots/overview) and [Activities](https://docs.discord.com/developers/activities/overview). For game integrations, see [Social Commerce & Game Shops](https://docs.discord.com/developers/social-commerce/overview) for Discord game monetization. Premium Apps lets you sell app features inside Discord with Discord payment and checkout handling. Users can subscribe or buy without leaving Discord, keeping the purchase native and reducing friction. Discord manages billing, fraud checks, and receipts; your app grants access to the premium feature. You can monetize in two ways:
- **Recurring monthly subscriptions** offered to individuals or entire servers
- **One-Time purchases** for durable upgrades or consumable items
 Both monetization models use the same [SKU](https://docs.discord.com/developers/resources/sku) and [Entitlement](https://docs.discord.com/developers/resources/entitlement) APIs, so the app checks entitlement status before enabling a feature. Monetized apps gain more App Directory presence, including a store page where users can browse and purchase offerings before installing the app.

### Monetization Overview

Complete guide to SKUs, Entitlements, Subscriptions, and the App Directory store page.

### Enabling Monetization

Setup walkthrough for enabling payments and creating a first SKU.

## Bots Overview

*Already know bots? Jump to the [Bots & Companion Apps docs](https://docs.discord.com/developers/bots/overview) or [Getting Started guide](https://docs.discord.com/developers/quick-start/getting-started).* Bots are the most common Discord app type. They show up in servers as bot users with an `APP` tag and can receive events, answer slash commands, moderate, send messages, and run many other workflows through code that uses the Discord API. A bot can connect to Discord through a persistent [Gateway WebSocket](https://docs.discord.com/developers/events/gateway) for live events, an [HTTP interactions endpoint](https://docs.discord.com/developers/interactions/overview) for slash commands and UI components without a persistent socket, or both. Typical bot uses include moderation, server utilities, games, external-service integrations, and automation. If the only requirement is posting messages into a channel, a [webhook](https://docs.discord.com/developers/platform/webhooks) may be simpler than a full bot. Webhooks can send rich embedded messages, but they cannot listen for events or answer interactions the way bots can.

### Bots Overview

Explains what bots are, how they operate, and when to choose one.

### Getting Started

Build and run a first Discord bot from start to finish.

### Bot Dev Guides

Guides covering interactions, components, monetization, and related bot topics.

## What is Game Claiming?

(Figure: A rendition of the benefits that come with the verified checkmark)

### What is Game Claiming?

 Game claiming associates the game's Discord community server and profile with the verified developer identity for that title. Players get an official destination for the community, while you control the game's presentation inside the Discord app.

### Benefits of Claiming Your Game

### Game Profile Customization

 Discord creates a [game profile](https://docs.discord.com/developers/platform/game-profiles) automatically for each game it detects. After claiming the game, you manage profile details and visuals across four areas:
- **Title:** description, genres, platforms
- **Visual Assets:** cover, icon, banner, screenshots
- **Links:** official website, social media
- **Company:** name, publisher or developer role
 You make these changes in the [Developer Portal](https://discord.com/developers/applications), which lets you keep the game's identity aligned with live-service updates or seasonal content.

### Verified Checkmark

 After Discord confirms the claim, the Discord server receives a green verified badge on its server profile, identifying it as the verified community for the game on Discord. The badge appears in Server Discovery and is limited to servers tied to claimed games. [Learn more about verified servers](https://discord.com/community).

---

### Before You Begin

 Confirm these requirements before starting:
- Your game needs a Steam store page. Other storefronts are not supported yet.
- The Steam store page needs a public Discord invite link. During claiming, Discord gives you a `proof` value that must be appended to that link before submission.
- Games primarily delivering sexual content, and Mods of other games, cannot be claimed.
- Your Discord application must belong to a [development team](https://docs.discord.com/developers/topics/teams#teams), not an individual account.
- The server owner has to be on your Team. Discord checks ownership by emailing a verification code to that owner.
- Discord must already have an official record for the game. Automatic recognition depends on enough playerbase and play time among Discord users over a sustained period.
 To determine whether Discord already recognizes the game, look it up in the Game Identity dropdown during Step 2. If the title is absent, Discord has not yet created an official record for it. Eligibility can appear later as playerbase and playtime grow. Implementing [Rich Presence](https://docs.discord.com/developers/platform/rich-presence) through the Discord Social SDK is separate from automatic game detection; the systems are independent, and Rich Presence alone does not make a game claimable. Private beta and limited-release games are not supported at this time. A game must be publicly available and playable to qualify for claiming.

---

### Claiming Your Game

### Step 1: Open Game Identity in the Developer Portal

 Open [Game Identity in the Developer Portal](https://discord.com/developers/applications/select/game-identity) and choose your application. The claiming UI appears automatically. Existing SDK partners should claim the game to the application already used for the SDK integration.

### Step 2: Find and Select Your Game

 Use the Game Identity interface to search for your game by name, then select it to start the claim. If the game is not listed, Discord does not have an official record for it yet, so it cannot be claimed now. As playerbase and playtime increase, the game may become eligible for automatic recognition.

### Step 3: Update Your Steam Invite Link

 After you choose the game, the claim UI shows a `proof` query parameter. Copy it and add it to the Discord invite link on the Steam store page.
1. Copy the `proof` value displayed in the Developer Portal.
2. Edit the Discord invite link on the Steam store page so it includes the value, such as `discord.gg/yourcode?proof=<string>`.
 After the link is changed, Discord checks the `proof` value before letting you proceed.

### Step 4: Verify Server Ownership

 Discord verifies control of the server linked from the Steam store page by sending a **verification code to the server owner's email address**.
1. Open the email inbox for the Discord account that owns the server.
2. Enter the code when the portal asks for it.

### Step 5: Provide Your Details

 The process asks for these details:
- Your full legal name (required)
- A professional contact email (required)
- Your role at the company (optional)
- Your company or game website (optional)
 Discord uses this information according to the [Developer Terms of Service](https://support-dev.discord.com/hc/en-us/articles/8562894815383-Discord-Developer-Terms-of-Service).

### Step 6: Submit for Review

 After every step is complete, submit the claim. Discord will tell you whether it succeeded or contact you if more information is required.

---

### If Your Claim is Successful

 A successful claim sends an email notification and updates the **Game Identity** section in your **Developer Portal**. You can then start editing the game's profile.

### If Your Claim is Rejected

 If the claim is rejected, continue through the [Developer Support](https://dis.gd/developer-support/game-development) ticket and address the concerns raised there.

---

### Current Limitations

 Game Claiming currently has these limits:
- **No mobile support:** The game must be playable on non-mobile platforms such as PC, Mac, or consoles. Mobile-only games cannot be claimed right now.
- **Steam only:** Claiming currently works only for games with a Steam store page. Other platform support is not available yet.
- **Released only:** The game must be listed on Steam and publicly playable; private betas are excluded.
 Discord is expanding coverage so additional games can be claimed over time. If you think your game should appear but cannot find it, [contact us](https://dis.gd/developer-support/game-development).

---

### Next Steps

### Discord Social SDK

Add social features inside the game with Discord Social SDK: friends, voice, lobbies, and more.

### Game Profiles

Learn where Discord displays a game's profile and how that profile is presented across the platform.

## Enabling Community Server Features

Community Servers are Discord servers with features designed for building a public audience:
- [Structured Community Onboarding](https://support.discord.com/hc/en-us/articles/10394859532823-Community-Onboarding-Examples)
- [Announcement Channels](https://support.discord.com/hc/en-us/articles/360032008192-Announcement-Channel-FAQ)
- [AutoMod](https://support.discord.com/hc/en-us/articles/4421269296535-AutoMod-FAQ)
- [Server Discovery](https://support.discord.com/hc/en-us/articles/360023968311-Server-Discovery)
- [Server Insights](https://support.discord.com/hc/en-us/articles/360032807371-Server-Insights-FAQ)
 Game developers often use a Community Server as the official player hub for updates, support, and keeping the community connected between sessions.

### Enabling Community Server Features

 Any server can be converted into a Community Server by following [Enabling Your Community Server](https://support.discord.com/hc/en-us/articles/360047132851-Enabling-Your-Community-Server). After Community features are enabled, you can build out the server with the listed features.

### Official Game Communities

 After the Community Server is ready, you can make it the Official Game Community by [Claiming Your Game](https://docs.discord.com/developers/platform/claim-your-game). This unlocks features such as a custom vanity URL, a verified badge, and linking between the game and server so players can find it from the store page and Discord profile.

### Next Steps

### Enabling Community Server Features

Step-by-step instructions for turning on Community features for a Discord server.

### Create a Community For Your Game

Create a community server, set up roles and channels, and bring in players from the game.

### Community Management Guides

Guides for creating and growing a Discord community.
