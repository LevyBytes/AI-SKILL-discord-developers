# Setting Rich Presence

> Discord Developers reference. Original prose; identifiers preserved verbatim.

### Overview

 Rich Presence shows detailed game activity for a player. Discord can display that state on the user profile and friends list, and Game Invites can use it so friends can join.

### Prerequisites

 Before starting, verify you have:
- [Set up the Discord Social SDK](https://docs.discord.com/developers/discord-social-sdk/getting-started)
- A valid client instance connected to Discord
 This feature needs the **Default Presence Scopes** (`openid` and `sdk.social_layer_presence`). Configure OAuth2 with [`Client::GetDefaultPresenceScopes`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a7648bd1d2f7d9a86ebd0edb8bef12b5c). The [OAuth2 Scopes guide](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes) covers the complete scope set.

---

### Understanding Rich Presence

 Rich Presence publishes detailed information about a player's in-game actions. Discord may show that information in places such as:
- User profiles
- Friend lists
- Server member lists
 First, distinguish the Discord terms. Rich Presence, also called “Activity”, represents a user's current activity and is modeled by [`Activity`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Activity.html#ae793d9adbe16fef402b859ba02bee682) in the SDK and [in our gateway events](https://docs.discord.com/developers/events/gateway-events#activity-object). Do not confuse this with [Discord Activities](https://docs.discord.com/developers/activities/overview), which are embedded games that can also publish rich presence. Each [`Activity`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Activity.html#ae793d9adbe16fef402b859ba02bee682) has fields for:
| Field | Description | Purpose |
| --- | --- | --- |
| `name` | Game or app name | Shown on the user's profile |
| `type` | Activity type | The player action category, such as “Playing”, “Watching”, or “Listening” |
| `details` | What the player is doing | Primary activity text, such as “Playing Capture the Flag” |
| `state` | Their current status | Secondary status, such as “In Queue”, “In Match”, or “In a group” |
| `party` | Party information | Party count and maximum size, for example “2 of 4” |
| `timestamps` | Activity duration | Shows elapsed or remaining time |
| `assets` | Custom artwork | Images such as map thumbnails, game art, or character icons |
| `secrets` | Join/spectate tokens | Enable Game Invite functionality |
| `supportedPlatforms` | Platform flags | Control where join buttons appear |

 The examples below show how to set these fields. Games should set `type` to `ActivityTypes::Playing`, even though multiple [`ActivityTypes`](https://discord.com/developers/docs/social-sdk/namespacediscordpp.html#a6c76a8cbbc9270f025fd6854d5558660) exist. The SDK ties the activity to your game, so fields such as `name` default to the game name. To override that displayed name, see [Setting the Application Name](https://docs.discord.com/developers/discord-social-sdk/development-guides/setting-rich-presence#setting-the-application-name).

### Customizing Rich Presence

 Discord renders Rich Presence using three major pieces:

```
Playing "Your Game Name"          <- Line 1: Game name (automatic)
Capture the Flag | 2 - 1          <- Line 2: Details field
In a group (2 of 3)               <- Line 3: State + Party info

```

 Lines 2 and 3 are configurable; the mapping is:
- Line 1, `Playing "game name"`, comes from the Discord name of your game or application.
- Line 2, `Capture the flag | 2 - 1`, comes from the activity `details` field and should usually describe the current player action. Dynamic values, such as match score, can go here.
- Line 3, `In a group (2 of 3)`, describes the player's *party*. A “Party” is a player group sharing a context such as a lobby, server, or team. The `state` field provides `In a group`; the activity party field provides `(2 of 3)` and represents current and maximum party size.
 This diagram maps the visual fields: (Figure: Graphical representation of the legend for rich presence details) For design advice, see the [Rich Presence best practices guide](https://docs.discord.com/developers/rich-presence/best-practices).

---

### Setting an Invite Image

 The Rich Presence invite image appears on invites for third-party games or apps integrated with the Discord Social SDK. After you upload the invite image, the application settings show a preview on the right under “IRL Invite Image Example”. (Figure: Rich Presence invite image in app settings)

---

### Uploading Assets

 Most Rich Presence integrations need custom app artwork. Discord strongly recommends 1024 x 1024 images for Rich Presence assets. In the app settings, choose Rich Presence in the left sidebar and use the [Art Assets page](https://discord.com/developers/applications/select/rich-presence/assets) to upload either of two asset types. (Figure: Rich Presence invite image in app settings) An app can store up to 300 custom assets for later Rich Presence use. These assets can be any art that helps others understand what the user is doing in your Activity or 3rd party game. If 300 assets is not enough, or your images live elsewhere, you can [specify an external URL](https://docs.discord.com/developers/events/gateway-events#activity-object-activity-asset-image) when it has the required dimensions and size. External URLs, unlike uploaded assets, may use GIF, animated WebP, and AVIF. For asset selection guidance, see the [Rich Presence best practices guide](https://docs.discord.com/developers/rich-presence/best-practices#have-interesting-expressive-art). Uploaded Rich Presence asset keys are **automatically converted to lowercase**. The app settings show this after save, and code should reference the lowercase key. After upload, use the asset key in code when [Setting Assets in Rich Presence](https://docs.discord.com/developers/discord-social-sdk/development-guides/setting-rich-presence#setting-assets). (Figure: Rich Presence assets in app settings)

---

### Setting Details and State

 This minimal example sets activity details and state:

```
// Create a new activity
discordpp::Activity activity;
activity.SetType(discordpp::ActivityTypes::Playing);
activity.SetDetails("Battle Creek");
activity.SetState("In Competitive Match");

// Update the presence
client->UpdateRichPresence(activity, [](discordpp::ClientResult result) {
  if (result.Successful()) {
    std::cout << "✅ Rich presence updated!\n";
  }
});

```

---

### Setting the Application Name

 By default, Rich Presence uses the Discord-registered application name on the top line, for example `Playing "My Game: Chapter Two"`. To change the displayed name, set the activity `name` with [`Activity::SetName`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Activity.html#a59f9a63a8b105946d0c9838f3e643ae2) before calling [`Client::UpdateRichPresence`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#af0a85e30f2b3d8a0b502fd23744ee58e):

```
// Override the displayed application name (top line of Rich Presence)
activity.SetName("My Game: Chapter Two");

```

 Discord allows only a limited character set for registered application names. [`Activity::SetName`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Activity.html#a59f9a63a8b105946d0c9838f3e643ae2) lets you display text with more flexibility, including characters that cannot appear in the registered app name.

---

### Setting Timestamps

 Rich Presence can include timestamps for elapsed time in the current activity or remaining time in a timed match. Example:

```
// Setting Timestamps
discordpp::ActivityTimestamps timestamps;
timestamps.SetStart(time(nullptr));
// timestamps.SetEnd(time(nullptr) + 3600);
activity.SetTimestamps(timestamps);

```

---

### Setting Assets

 After assets are uploaded, reference them by **asset key** in code to display custom Rich Presence art. This example uses keys `map-mainframe`, `tank-avatar`, and `invite-cover-image`:

```
// Setting Activity Assets
discordpp::ActivityAssets assets;
assets.SetLargeImage("map-mainframe");
assets.SetLargeText("Mainframe");
assets.SetSmallImage("tank-avatar");
assets.SetSmallText("Tank");
assets.SetInviteCoverImage("invite-cover-image"); // Used for Game Invites
activity.SetAssets(assets);

```

 If you need more than 300 custom assets or prefer externally hosted images, you can [specify an external URL](https://docs.discord.com/developers/events/gateway-events#activity-object-activity-asset-image) if it meets the required dimensions and size. External URLs also allow GIF, animated WebP, and AVIF, unlike uploaded assets.

---

### Setting Field URLs

 Rich Presence supports URLs on `details`, `state`, `assets.large_image`, and `assets.small_image`. When set, the matching text or image becomes a clickable link.

```
activity.SetState("Playing on Mainframe");
activity.SetStateUrl("https://example.com/maps/mainframe");
activity.SetDetails("Rank #1337 in global leaderboard");
activity.SetDetailsUrl("https://example.com/leaderboard/global");

discordpp::ActivityAssets assets;
assets.SetLargeImage("map-mainframe");
assets.SetLargeText("Mainframe");
assets.SetLargeUrl("https://example.com/maps/mainframe");
assets.SetSmallImage("tank-avatar");
assets.SetSmallText("Tank");
assets.SetSmallUrl("https://example.com/classes/tank");

activity.SetAssets(assets);

```

---

### Setting Buttons

 A player's Rich Presence can include up to two custom buttons. Each button has a label plus a URL, so viewers can click through to a store page, website, community server, or another call to action.

```
discordpp::ActivityButton buyButton;
buyButton.SetLabel("Buy On Steam!");
buyButton.SetUrl("https://store.steampowered.com/app/AppID/GameName/");

discordpp::ActivityButton communityButton;
communityButton.SetLabel("Join the Community!");
communityButton.SetUrl("https://discord.gg/your-discord-url-or-id");

activity.AddButton(buyButton);
activity.AddButton(communityButton);

```

 Buttons appear only for **other users**. You cannot see buttons on your own Rich Presence, so test with another account or ask someone else to view the profile.

---

### Configuring Status Text

 Rich Presence normally uses the game name as the user's status text. Set a status display type to override that default.

```
// uses the game's name in the status text (default)
activity.SetStatusDisplayType(discordpp::StatusDisplayTypes::Name);

// uses the activity's state field in the status text
activity.SetStatusDisplayType(discordpp::StatusDisplayTypes::State);

// uses the activity's details field in the status text
activity.SetStatusDisplayType(discordpp::StatusDisplayTypes::Details);

```

---

### Setting Party and Join Secret

 Rich Presence can also carry party information and a join secret for Game Invites. See the [Game Invites guide](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-game-invites) for details.

```

// Setting Party Details
discordpp::ActivityParty party;
party.SetId("party1234");
party.SetCurrentSize(1);
party.SetMaxSize(5);
activity.SetParty(party);

// Setting Join Secret details
discordpp::ActivitySecrets secrets;
secrets.SetJoin("your-join-secret");
activity.SetSecrets(secrets);

```

---

### Setting Supported Platforms

 Set supported platforms in Rich Presence to control where Discord shows join buttons. To show the join button only on desktop, configure supported platforms like this:

```
activity.SetSupportedPlatforms(discordpp::ActivityGamePlatforms::Desktop);

```

 The `ActivityGamePlatforms` enum lists every supported platform.

### Rich Presence Without Authentication

 RPC (Remote Procedure Call) Rich Presence works only when the Discord desktop client is running; it does not work with mobile, console, or web clients. Unlike many Discord Social SDK features, **Rich Presence can be set without authentication**. Instead of authenticating through [`Client::Connect`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a873a844c7c4c72e9e693419bb3e290aa), communicate directly with the running desktop client over RPC.

### Requirements

- The user must have the Discord desktop client running
- Your app must be registered on Discord and have a valid Application ID
 This direct mode is simpler when you only need basic presence while the Discord desktop client is open.

### Setting Up Direct Rich Presence

 For unauthenticated Rich Presence, do the following:
1. Set the application ID with [`Client::SetApplicationId`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ad452335c06b28be0406dab824acccc49)
2. Configure your activity details
3. Call [`Client::UpdateRichPresence`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#af0a85e30f2b3d8a0b502fd23744ee58e)

```
auto client = std::make_shared<discordpp::Client>();

// Set the application ID (no Connect() call needed)
client->SetApplicationId(APPLICATION_ID);

// Configure rich presence details
discordpp::Activity activity;
activity.SetType(discordpp::ActivityTypes::Playing);
activity.SetState("In Competitive Match");
activity.SetDetails("Rank: Diamond II");

// Update rich presence
client->UpdateRichPresence(
    activity, [](const discordpp::ClientResult &result) {
      if (result.Successful()) {
        std::cout << "🎮 Rich Presence updated successfully!\n";
      } else {
        std::cerr << "❌ Rich Presence update failed";
      }
    });

```

---

### Next Steps

 After configuring Rich Presence, consider these related guides:

### Managing Game Invites

Let players invite friends into a session or party.

### Managing Lobbies

Use lobbies to group players with invites, text chat, and voice communication.

### Design Guidelines for Rich Presence

Rich Presence UI/UX recommendations. Need help? Join the [Discord Developers Server](https://discord.gg/discord-developers) and ask in `#social-sdk-dev-help` for community support. Report Social SDK bugs at [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report).

---

### Change Log

| Date | Changes |
| --- | --- |
| June 5, 2026 | Added the Setting the Application Name section |
| March 31, 2026 | Added Setting Buttons section |
| March 17, 2025 | Initial release |
