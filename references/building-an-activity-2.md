# Building An Activity (part 2)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

```
import { DiscordSDK } from "@discord/embedded-app-sdk";

import rocketLogo from '/rocket.png';
import "./style.css";

// Will eventually store the authenticated user's access_token
let auth;

const discordSdk = new DiscordSDK(import.meta.env.VITE_DISCORD_CLIENT_ID);

setupDiscordSdk().then(() => {
  console.log("Discord SDK is authenticated");

  // We can now make API calls within the scopes we requested in setupDiscordSDK()
  // Note: the access_token returned is a sensitive secret and should be treated as such
});

async function setupDiscordSdk() {
  await discordSdk.ready();
  console.log("Discord SDK is ready");

  // Authorize with Discord Client
  const { code } = await discordSdk.commands.authorize({
    client_id: import.meta.env.VITE_DISCORD_CLIENT_ID,
    response_type: "code",
    state: "",
    prompt: "none",
    scope: [
      "identify",
      "guilds",
      "applications.commands"
    ],
  });

  // Retrieve an access_token from your activity's server
  const response = await fetch("/api/token", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      code,
    }),
  });
  const { access_token } = await response.json();

  // Authenticate with Discord client (using the access_token)
  auth = await discordSdk.commands.authenticate({
    access_token,
  });

  if (auth == null) {
    throw new Error("Authenticate command failed");
  }
}

document.querySelector('#app').innerHTML = `
  <div>
    <img src="${rocketLogo}" class="logo" alt="Discord" />
    <h1>Hello, World!</h1>
  </div>
`;

```

 After relaunching the app, Discord prompts for authorization with the `identify`, `guilds`, and `applications.commands` scopes. (Figure: Prompt to authorize Activity) **Safe storage of tokens**Access tokens and refresh tokens grant meaningful access, so protect them like passwords or other sensitive secrets. Store both token types securely and encrypted.

Step 5 Checkpoint

Before leaving Step 5, confirm that you have:
- Changed `client/main.js` so it calls the backend for user authorization and authentication support
- Verified that opening the Activity lets your app complete the authorization flow successfully

---

### Step 6: Use the SDK to fetch the channel

 With users authenticated, the app can use contextual Discord data inside the Activity. Start by using the SDK to read details for the channel where the Activity is running. Add an async function that calls the `commands.getChannel` SDK method.

Using the SDK to retrieve a channel

Add this function to the same `getting-started-activity/client/main.js` file:

```
async function appendVoiceChannelName() {
  const app = document.querySelector('#app');

  let activityChannelName = 'Unknown';

  // Requesting the channel in GDMs (when the guild ID is null) requires
  // the dm_channels.read scope which requires Discord approval.
  if (discordSdk.channelId != null && discordSdk.guildId != null) {
    // Over RPC collect info about the channel
    const channel = await discordSdk.commands.getChannel({channel_id: discordSdk.channelId});
    if (channel.name != null) {
      activityChannelName = channel.name;
    }
  }

  // Update the UI with the name of the current voice channel
  const textTagString = `Activity Channel: "${activityChannelName}"`;
  const textTag = document.createElement('p');
  textTag.textContent = textTagString;
  app.appendChild(textTag);
}

```

Next, change the callback following `setupDiscordSdk()` so it invokes the new function:

```
setupDiscordSdk().then(() => {
  console.log("Discord SDK is authenticated");

  appendVoiceChannelName();
});

```

 After closing and joining the Activity again, the current channel name should be visible. (Figure: Discord Activities)

Step 6 Checkpoint

Before finishing Step 6, verify that you have:
- Changed `client/main.js` so it gets the channel name through the SDK
- Called the new function from the `setupDiscordSdk()` callback

---

### Step 7: Use the API to fetch the guild

 Because the authorization request included the `identify` and `guilds` scopes, the earlier `access_token` can also be used to request those API resources. The next code block will:
1. Request [`GET /users/@me/guilds`](https://docs.discord.com/developers/resources/user#get-current-user-guilds) with `auth.access_token` to retrieve guilds for the authorizing user
2. Loop through the guilds and identify the one matching the `guildId` value from discordSdk
3. Build a new HTML image element from the guild avatar and attach it to the frontend
 This example uses a plain `fetch` request for the API call, but a JavaScript [community-built library](https://docs.discord.com/developers/developer-tools/community-resources) can be used instead.

Getting details for the current server

Add the following function in the same `client/main.js` file:

```
async function appendGuildAvatar() {
  const app = document.querySelector('#app');

  // 1. From the HTTP API fetch a list of all of the user's guilds
  const guilds = await fetch(`https://discord.com/api/v10/users/@me/guilds`, {
    headers: {
      // NOTE: we're using the access_token provided by the "authenticate" command
      Authorization: `Bearer ${auth.access_token}`,
      'Content-Type': 'application/json',
    },
  }).then((response) => response.json());

  // 2. Find the current guild's info, including it's "icon"
  const currentGuild = guilds.find((g) => g.id === discordSdk.guildId);

  // 3. Append to the UI an img tag with the related information
  if (currentGuild != null) {
    const guildImg = document.createElement('img');
    guildImg.setAttribute(
      'src',
      // More info on image formatting here: https://docs.discord.com/developers/reference#image-formatting
      `https://cdn.discordapp.com/icons/${currentGuild.id}/${currentGuild.icon}.webp?size=128`
    );
    guildImg.setAttribute('width', '128px');
    guildImg.setAttribute('height', '128px');
    guildImg.setAttribute('style', 'border-radius: 50%;');
    app.appendChild(guildImg);
  }
}

```

Then invoke the new function inside the callback for `setupDiscordSdk`:

```
setupDiscordSdk().then(() => {
  console.log("Discord SDK is authenticated");

  appendVoiceChannelName();
  appendGuildAvatar();
});

```

 When the Activity is relaunched, the Activity should render the current server's avatar. (Figure: Discord Activities)

Step 7 Checkpoint

At this stage, the Activity should be running. For Step 7, confirm that you have:
- Updated `client/main.js` so it fetches guild data through the [`GET /users/@me/guilds`](https://docs.discord.com/developers/resources/user#get-current-user-guilds) API endpoint
- Called the new function from the `setupDiscordSdk()` callback

---

### Next Steps

 You now have a first Activity built. This deliberately small example introduces communication between an Activity and Discord through the Embedded App SDK and APIs. Continue with the [Activities documentation](https://docs.discord.com/developers/activities/overview) and related resources.

### Development Guides

Use the Activities Development Guides for recommended implementation practices and design considerations.

### Sample Activity Projects

Experiment with the playground app to exercise the Embedded App SDK feature set, or review the other sample projects.

### Discord Developers

Use the community to ask API questions, join events from the Discord platform team, and talk with other Activities developers.
