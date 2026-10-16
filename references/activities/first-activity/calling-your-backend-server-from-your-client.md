# Calling your backend server from your client
[Source](project-and-authentication-index.md) / [Continued source contract](calling-your-backend-server-from-your-client.md) / [Previous source section](calling-external-resources-from-your-activity.md)

<a id="source-calling-your-backend-server-from-your-client"></a>

### Calling your backend server from your client

 The client now needs to call the server so the OAuth process can start and produce an access token. **What is vite.config.js?** Vite needs a proxy from `/api/*` to the Express backend on port 3001 so the frontend can call it. The [Vite](https://vitejs.dev/) docs explain the tool in more depth.

Calling the backend server

Code for authorizing and authenticating

Copy this code into `getting-started-activity/client/main.js`:

<a id="source-building-an-activity-part-2"></a>

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
