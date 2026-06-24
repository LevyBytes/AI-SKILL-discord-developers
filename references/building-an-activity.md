# Building An Activity (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

[Activities](https://docs.discord.com/developers/activities/overview) are web-based games and apps that run inside Discord. They are embedded as iframes in the Discord client and can start from the App Launcher or as responses to interactions. If Activities are new to you, read the [Activities Overview](https://docs.discord.com/developers/activities/overview) for background and more advanced [sample projects](https://docs.discord.com/developers/activities/overview#sample-projects).

### Introduction

 This guide builds a Discord app with a simple Activity that authenticates users and retrieves data through the API. It assumes familiarity with [JavaScript](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/JavaScript_basics), [async functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function), and frontend frameworks such as [React](https://react.dev/) and [Vue](https://vuejs.org/). If you are still learning programming fundamentals, free resources include [The Odin Project](https://www.theodinproject.com/paths/foundations/courses/foundations), [Codecademy](https://www.codecademy.com/learn/paths/web-development), and [Khan Academy](https://www.khanacademy.org/computing/computer-programming/programming).

What we'll be building

(Figure: Building Your First Activity Tutorial)

Resources used in this guide

- **[discord/getting-started-activity](https://github.com/discord/getting-started-activity)**, the starter project template
- **[@discord/embedded-app-sdk](https://github.com/discord/embedded-app-sdk)**, the SDK that lets an Activity communicate with Discord
- **[Node.js](https://nodejs.org)**, latest version
- **[Express](https://expressjs.com)**, the JavaScript web framework used here for authentication handling and app serving
- **[Vite](https://vite.dev/)**, a modern JavaScript build tool that simplifies serving the app
- **[cloudflared](https://github.com/cloudflare/cloudflared?tab=readme-ov-file#installing-cloudflared)**, used to expose the local development server to the internet

---

### Step 0: Enable Developer Mode

 Enable Developer Mode on your Discord account before starting if it is not already on. Developer Mode lets you run in-development Activities and exposes resource IDs, such as users, channels, and servers, which helps during testing. To enable it:
1. Open **User Settings** in the Discord client. On Desktop, click the cogwheel near the bottom-left beside your username.
2. Select the **Advanced** tab in the left sidebar and turn on `Developer Mode`.

### Step 1: Setting up the project

 Before creating the app, set up the project code from [`discord/getting-started-activity`](https://github.com/discord/getting-started-activity). Open a terminal and clone the repository:

```
git clone git@github.com:discord/getting-started-activity.git

```

 The cloned sample project has two main parts:
- `client` contains the sample Activity frontend, built with vanilla JavaScript and wired to [Vite](https://vitejs.dev/) for local development.
- `server` contains a backend written with vanilla JavaScript, Node.js, and Express. When building your own Activity, you can choose a different backend.

Project structure

Project layout for the sample app used in this guide

```
├── client
├────── main.js       -> your Activity frontend
├────── index.html
├────── package.json
├────── rocket.png
├────── vite.config.js
├── server
├────── package.json
├────── server.js     -> your Activity backend
└── .env              -> your credentials, IDs and secrets

```

### Install project dependencies

 Before creating the Discord app, install the frontend dependencies. Move into the project's `client` directory, where the sample Activity frontend code is located:

```
cd getting-started-activity/client

```

 Then install the dependencies and start the sample Activity frontend:

```
# install project dependencies
npm install

# start frontend
npm run dev

```

 Opening [http://localhost:5173/](http://localhost:5173/) should show a vanilla JS frontend template running through [Vite](https://vitejs.dev/). It is minimal now, but the next steps connect it to backend services, make it launchable in Discord, and fill it with data from Discord APIs.

Step 1 Checkpoint

By the end of Step 1, confirm that you have:
- A basic understanding of Discord [Activities](https://docs.discord.com/developers/activities/overview)
- Developer Mode enabled for your Discord account
- The [sample project](https://github.com/discord/getting-started-activity) cloned into your development environment
- Frontend dependencies installed in the `client` folder

---

### Step 2: Creating an app

 With the project ready, create the app and configure its Activity. If you do not already have an app, create one in the developer portal. Enter a name, choose a development team, and press **Create**. **Development Team Access** Non-distributed Activities can only be launched by you or members of the developer team. If others are collaborating during development, create a [developer team](https://discord.com/developers/teams) and set that team as the owner when creating the app. After creation, the app opens on the **General Information** settings page, where you can update details such as description and icon.

### Choose installation contexts

 Discord apps can be installed to different **[installation contexts](https://docs.discord.com/developers/resources/application#installation-context)**: servers, user accounts, or both. The recommended default is to support both contexts so the installer can choose during installation. You can change supported contexts in the app settings.

Why do installation contexts matter?

Overview of supported app installation locations

Installation contexts control where the app can be installed. They affect who manages the installation, where commands are visible, and what data is returned for interactions.
- Apps installed in a **[server context](https://docs.discord.com/developers/resources/application#server-context)** (server-installed apps) require approval from a member who has `MANAGE_GUILD`, and every member of that server can see them.
- Apps installed in a **[user context](https://docs.discord.com/developers/resources/application#user-context)** (user-installed apps) belong to the person who authorized the app, so no server-specific permission is needed. A user-context app appears across that user's servers, DMs, and GDMs, but it is limited to commands.
Read more about installation contexts in the [Application documentation](https://docs.discord.com/developers/resources/application#installation-context) and the [Developing a User-Installable App tutorial](https://docs.discord.com/developers/tutorials/developing-a-user-installable-app). Open [**Installation**](https://discord.com/developers/applications/select/installation) from the left sidebar. Under **Installation Contexts**, make sure **User Install** and **Guild Install** are selected. This lets users launch the Activity from Discord servers, DMs, and Group DMs.

### Add a Redirect URI

 Add a Redirect URI, the location a user is normally sent to after authorizing your app through the standard OAuth flow. The Redirect URI is required, but the Embedded App SDK sends users back to the Activity automatically when the RPC [`authorize` command](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#authorize) runs. The [OAuth2 documentation](https://docs.discord.com/developers/topics/oauth2) explains the OAuth flow and redirect URIs. Because this guide authorizes only inside an Activity, use `https://127.0.0.1` as a placeholder and let the Embedded App SDK handle the return path. In the app settings, open [**OAuth2**](https://discord.com/developers/applications/select/oauth2). Under **Redirects**, enter `https://127.0.0.1` then click **Save Changes**. (Figure: Redirect URI in Activity Settings)

### Fetch Your OAuth2 Credentials

 To access user information such as username, or server information such as a server avatar, the app must request OAuth **scopes**. The sample app requests three scopes: `identify` for basic user information, `guilds` for basic information about servers the user is in, and `applications.commands` to install [commands](https://docs.discord.com/developers/interactions/overview#commands). These are requested later, and the [OAuth2 documentation](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-scopes) lists all available scopes. Later scope requests need the app's OAuth2 identifiers. For now, copy those identifiers into the project environment file. The project root includes `example.env`; from the root, copy it to a new `.env` file:

```
cp example.env .env

```

 **Secure Your Secrets** `DISCORD_CLIENT_SECRET` and `DISCORD_BOT_TOKEN` are *highly* sensitive. Never share either secret or commit it to version control. In the app settings, open [**OAuth2**](https://discord.com/developers/applications/select/oauth2):
1. **Client ID**: Copy Client ID into `.env` as **`VITE_CLIENT_ID`**. This public ID identifies the app to Discord and is usually the same as the App ID.
2. **Client Secret**: Copy Client Secret into `.env` as **`DISCORD_CLIENT_SECRET`**. This private identifier lets the app grant an OAuth2 `access_token`; do not share it or commit it to version control.
 **Why is there a VITE_ prefix before our Client ID?** Adding `VITE_` before the `CLIENT_ID` environment variable exposes it to client-side code. This keeps only intentionally public browser variables available while other environment variables stay private. The [Vite documentation](https://vitejs.dev/guide/env-and-mode) has the details.

Step 2 Checkpoint

By the end of Step 2, verify that you have:
- Configured a placeholder Redirect URI
- Added the app's Client ID and Client Secret to the project's `.env` file.

### Step 3: Setting Up the Embedded App SDK

 After setting up the project and app, install and configure the [Embedded App SDK](https://docs.discord.com/developers/developer-tools/embedded-app-sdk), which the rest of the guide uses heavily. The Embedded App SDK is Discord's first-party bridge between the Activity and the Discord client. It provides [commands](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#sdk-commands) for client interactions, such as fetching channel information, and [events](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#sdk-events) for reacting to user actions or state changes, such as speaking status. The Embedded App SDK command and event set is a subset of the [RPC API](https://docs.discord.com/developers/topics/rpc), so RPC docs help explain the lower-level Activity behavior.

### Install the SDK

 From the project's earlier `client` directory (`getting-started-activity/client`), install the Embedded App SDK [via NPM](https://www.npmjs.com/package/@discord/embedded-app-sdk):

```
npm install @discord/embedded-app-sdk

```

 The install adds `@discord/embedded-app-sdk` to `getting-started-activity/client/package.json` and places the SDK in `node_modules`.

### Import the SDK in your Project

 After installation, import the SDK into the client code and instantiate it so the app can begin the handshake with the Discord client. Instantiation uses the environment variables created in Step 2. The code also waits for the [`ready` event](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#ready) with async/await so the app can log or run other behavior after the handshake succeeds.

Add SDK initialization to frontend

Code for adding the Embedded App SDK

In `getting-started-activity/client/main.js`, import and instantiate the SDK:

```
// Import the SDK
import { DiscordSDK } from "@discord/embedded-app-sdk";

import "./style.css";
import rocketLogo from '/rocket.png';

// Instantiate the SDK
const discordSdk = new DiscordSDK(import.meta.env.VITE_DISCORD_CLIENT_ID);

setupDiscordSdk().then(() => {
  console.log("Discord SDK is ready");
});

async function setupDiscordSdk() {
  await discordSdk.ready();
}

document.querySelector('#app').innerHTML = `
  <div>
    <img src="${rocketLogo}" class="logo" alt="Discord" />
    <h1>Hello, World!</h1>
  </div>
`;

```

 **Time to leave your browser behind** After adding the SDK, the app will no longer be viewable in a normal web browser. The next step runs the Activity inside Discord.

Step 3 Checkpoint

By the end of Step 3, verify that you have:
- Installed the Embedded App SDK in the project
- Imported the SDK in the project's `client/main.js` file

---

### Step 4: Running your app in Discord

 Next, check that everything is connected, enable activities in the dev portal, and launch the Activity in Discord.

### Run your app

 Restart the sample app first. Open a terminal, go to the project's `client` folder, and start the client-side app:

```
cd client
npm run dev

```

 The app should start and print output resembling this:

```
VITE v5.0.12  ready in 100 ms

➜  Local:   http://localhost:5173/
➜  Network: use --host to expose
➜  press h + enter to show help

```

 Use the Local URL as the public URL in the next step.

### Set up a public endpoint

 Next, create the public endpoint that serves the Activity frontend. Use a reverse-proxy tunnel. This guide uses [`cloudflared`](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/), but [ngrok](https://ngrok.com/docs) or another reverse proxy can be used instead. While the app is still running, open another terminal and start a tunnel to the previous port, `5173` in this example:

```
cloudflared tunnel --url http://localhost:5173

```

 Running `cloudflared` generates a public URL and prints output similar to this:

```
Your quick Tunnel has been created! Visit it at (it may take some time to be reachable):
https://funky-jogging-bunny.trycloudflare.com

```

 Copy the URL from that output; it must be added to the app settings.

### Set up your Activity URL Mapping

 Activities run in a sandbox and use the Discord proxy, so you need a public URL mapping to serve the application and make external requests from the Activity. For local development, use the public endpoint created earlier. In the app settings, open the [**URL Mappings** page](https://discord.com/developers/applications/select/embedded/url-mappings) under **Activities** in the left sidebar. Enter the URL generated by `cloudflared` in the prior step. (Figure: Configuring your URL Mapping)
| PREFIX | TARGET |
| --- | --- |
| `/` | `funky-jogging-bunny.trycloudflare.com` |

 Read the [development guide](https://docs.discord.com/developers/activities/development-guides/local-development#url-mapping) for URL Mapping details.

### Enable Activities

 Enable Activities for the app next. In the left sidebar under **Activities**, open [**Settings**](https://discord.com/developers/applications/select/embedded/settings). Find the first checkbox, `Enable Activities`, and turn it on. (Figure: Enabling Activities in Settings)

#### Default Entry Point Command

 Enabling Activities automatically creates a [default Entry Point command](https://docs.discord.com/developers/interactions/application-commands#default-entry-point-command) named **Launch**. This [Entry Point command](https://docs.discord.com/developers/interactions/application-commands#entry-point-commands) is the main way users start the Activity in Discord. By default, using it opens the Activity for the user and posts a message in the launch channel. If your app should handle those interactions itself, update the [`handler` field](https://docs.discord.com/developers/interactions/application-commands#entry-point-handlers) or create a custom command. See the Entry Point command [documentation](https://docs.discord.com/developers/interactions/application-commands#entry-point-commands) and [development guide](https://docs.discord.com/developers/activities/development-guides/user-actions#setting-up-an-entry-point-command) for more detail.

### Running your Activity in Discord

 With Discord pointing to the local app, launch the Activity in Discord. Go to a Discord test server and open the App Launcher from a voice or text channel; the in-development Activity should appear there. If it is missing, search for the app name. Clicking the app launches the local app inside Discord. (Figure: Running your activity) **Customizing your Activity** To add images for the Activity, see the metadata guide [here](https://docs.discord.com/developers/activities/development-guides/assets-and-metadata#setting-up-activity-metadata). The Activity is now launching, but Discord functionality still needs to be wired in.

Step 4 Checkpoint

By the end of Step 4, verify that you have:
- Set up a public endpoint
- Added an Activity URL Mapping in the app settings
- Enabled Activities for your app
- Launched the Activity successfully inside Discord

---

### Step 5: Authorizing & authenticating users

 To authenticate users playing the Activity, finish the server-side app and connect it to the client-side app. This example uses `express`, but any backend language or framework can serve the same role.

OAuth2 Flow Diagram

This diagram shows the typical pattern for granting a user an OAuth2 access_token: (Figure: Flow diagram for Oauth2) The tutorial implements this pattern, and the sample project includes more examples:
- [Back-end code](https://github.com/discord/embedded-app-sdk-examples/blob/main/discord-activity-starter/packages/server/src/app.ts)
- [Front-end code](https://github.com/discord/embedded-app-sdk-examples/blob/main/discord-activity-starter/packages/client/src/main.ts)

```
# move into our server directory
cd server

# install dependencies
npm install

```

 The server code is not edited here. It contains one `POST` route for `/api/token`, which lets the server perform the OAuth2 flow securely.

getting-started-activity/server/server.js

The file `getting-started-activity/server/server.js` should already contain this code:

```
import express from "express";
import dotenv from "dotenv";
import fetch from "node-fetch";
dotenv.config({ path: "../.env" });

const app = express();
const port = 3001;

// Allow express to parse JSON bodies
app.use(express.json());

app.post("/api/token", async (req, res) => {

  // Exchange the code for an access_token
  const response = await fetch(`https://discord.com/api/oauth2/token`, {
    method: "POST",
    headers: {
      "Content-Type": "application/x-www-form-urlencoded",
    },
    body: new URLSearchParams({
      client_id: process.env.VITE_DISCORD_CLIENT_ID,
      client_secret: process.env.DISCORD_CLIENT_SECRET,
      grant_type: "authorization_code",
      code: req.body.code,
    }),
  });

  // Retrieve the access_token from the response
  const { access_token } = await response.json();

  // Return the access_token to our client as { access_token: "..."}
  res.send({access_token});
});

app.listen(port, () => {
  console.log(`Server listening at http://localhost:${port}`);
});

```

 Now start the project's backend server:

```
npm run dev

```

 The output should resemble this:

```
> server@1.0.0 dev
> node server.js

Server listening at http://localhost:3001

```

 Run the server app and client-side app in separate terminal windows. The other [sample projects](https://docs.discord.com/developers/activities/overview#sample-projects) show alternate setups.

### Calling external resources from your activity

 Before calling the backend Activity server, account for the Discord proxy and avoid Content Security Policy (CSP) problems. The guides for [Constructing a Full URL](https://docs.discord.com/developers/activities/development-guides/networking#construct-a-full-url) and [Using External Resources](https://docs.discord.com/developers/activities/development-guides/networking#using-external-resources) explain the topic.

### Calling your backend server from your client

 The client now needs to call the server so the OAuth process can start and produce an access token. **What is vite.config.js?** Vite needs a proxy from `/api/*` to the Express backend on port 3001 so the frontend can call it. The [Vite](https://vitejs.dev/) docs explain the tool in more depth.

Calling the backend server

Code for authorizing and authenticating

Copy this code into `getting-started-activity/client/main.js`:
