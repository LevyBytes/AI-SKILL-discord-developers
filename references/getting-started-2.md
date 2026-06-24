# Getting Started (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

[Discord apps](https://docs.discord.com/developers/quick-start/overview-of-apps) let you extend Discord with APIs and interactive features. This JavaScript tutorial builds a first Discord app that uses slash commands, sends messages, and handles component interactions. If your goal is an iframe-based game or social experience, use the tutorial for [building an Activity](https://docs.discord.com/developers/activities/building-an-activity). The example app is a rock-paper-scissors variant with 7 choices instead of 3. The guide is written for beginners but expects basic [JavaScript](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/JavaScript_basics) knowledge.

What we'll be building

The completed app works like this: (Figure: Demo of example app) The user flow is:
1. User A starts a new game and selects an object with the app's `/challenge` slash command
2. The app posts a channel message containing a button for other users to accept the challenge
3. User B clicks the **Accept** button
4. User B receives an ephemeral message and chooses an object
5. The app posts the game result back to the original channel so everyone can see it

Resources used in this guide

Tools and technologies used in this tutorial

- **[GitHub repository](https://github.com/discord/discord-example-app)** containing the guide code plus extra feature-specific examples.
- **[discord-interactions](https://github.com/discord/discord-interactions-js)**, a library with Discord app types and helper functions.
- **[Express](https://expressjs.com)**, the JavaScript web framework used here to create a server that receives Discord requests.
- **[ngrok](https://ngrok.com/)**, a tunneling tool that exposes the local server through a public URL for Discord requests.

---

### Step 0: Project Setup

 Start by preparing the local environment and retrieving code from the [sample app repository](https://github.com/discord/discord-example-app). This guide develops locally with help from [ngrok](https://ngrok.com/), though any preferred development environment can work. Install [NodeJS](https://nodejs.org/en/download/) first if it is not already installed. Then open a command line and clone the project:

```
git clone https://github.com/discord/discord-example-app.git

```

 Next, change into the directory and install the project's dependencies:

```
# navigate to directory
cd discord-example-app

# install dependencies
npm install

```

Project structure

Project layout for the sample app used by this tutorial

```
├── examples    -> short, feature-specific sample apps
├──── app.js  -> finished app.js code
├──── button.js
├──── command.js
├──── modal.js
├──── selectMenu.js
├── .env        -> your credentials and IDs
├── app.js      -> main entrypoint for app
├── commands.js -> slash command payloads + helpers
├── game.js     -> logic specific to Rock, Paper, Scissors
├── utils.js    -> utility functions and constants
├── package.json
├── README.md
└── .gitignore

```

 After setup, open the project in your preferred code editor and continue with Discord app configuration.

### Step 1: Creating an app

 Create an app in the developer portal if needed: [Create App](https://discord.com/developers/applications?new_application=true). Enter an app name and press **Create**. The new app opens on the **General Information** settings page, where you can edit metadata such as description and icon. The page also shows **Application ID** and **Interactions Endpoint URL**, both used later in the guide.

### Fetching your credentials

 The app needs several sensitive credentials, including its token and ID. The token authorizes API requests and carries app permissions, so treat it as *highly* sensitive: never share it or commit it to version control. In the project folder, rename `.env.sample` to `.env`; the `.env` file stores the app credentials. Add these three values from the app settings:
- On the [**General Information** page](https://discord.com/developers/applications/select/information), copy **Application ID**. In `.env`, replace `<YOUR_APP_ID>` with that value.
- On the [**General Information** page](https://discord.com/developers/applications/select/information), copy **Public Key**, which verifies that HTTP requests came from Discord. In `.env`, replace `<YOUR_PUBLIC_KEY>` with the copied value.
- On the [**Bot** page](https://discord.com/developers/applications/select/bot), under **Token**, click **Reset Token** to create a new bot token. In `.env`, replace `<YOUR_BOT_TOKEN>` with it.
 The token is hidden after generation unless you regenerate it, so store it securely, such as in a password manager. With credentials ready, configure the bot user and installation settings.

### Configuring your bot

 New apps include a bot user by default. A bot user lets the app appear and act like a server member when [installed to a server](https://docs.discord.com/developers/quick-start/overview-of-apps#where-are-apps-installed). In the left sidebar of the app settings, open the [**Bot** page](https://discord.com/developers/applications/select/bot), the same place where the token was created. That page also controls options such as [privileged intents](https://docs.discord.com/developers/events/gateway#privileged-intents) and whether other users can install the app.

What are intents?

Introductory notes on standard and privileged intents

Intents control which events Discord sends to the app over a [Gateway API connection](https://docs.discord.com/developers/events/gateway). For example, reacting to users adding reactions to messages can use the `GUILD_MESSAGE_REACTIONS` (`1 << 10`) intent. Some intents are [privileged](https://docs.discord.com/developers/events/gateway#privileged-intents), giving access to sensitive data such as message contents. You can toggle privileged intents on the **Bot** page, but they need approval before you [verify your app](https://support-dev.discord.com/hc/en-us/articles/23926564536471-How-Do-I-Get-My-App-Verified). Standard, non-privileged intents require no extra setup. The [Gateway documentation](https://docs.discord.com/developers/events/gateway#gateway-intents) has more information and the full list of intents with their events. This tutorial does not require more configuration here, though future app features may.

### Choosing installation contexts

 Next, choose where the app can be installed in Discord. That behavior is controlled by the [installation contexts](https://docs.discord.com/developers/resources/application#installation-context) the app supports.

What are installation contexts?

Summary of app installation locations

**Installation contexts** define whether an app can be installed to servers, users, or both. The supported contexts are selected in the app settings.
- Apps installed in a **[server context](https://docs.discord.com/developers/resources/application#server-context)** (server-installed apps) require authorization from a server member with the `MANAGE_GUILD` permission and are visible to everyone in that server.
- Apps installed in a **[user context](https://docs.discord.com/developers/resources/application#user-context)** (user-installed apps) appear just for the person who authorized them, so no server permission check is required. A user-context app appears across that user's servers, DMs, and GDMs, but it can only use commands.
 Open [**Installation**](https://discord.com/developers/applications/select/installation) from the left sidebar. Under **Installation Contexts**, make sure **User Install** and **Guild Install** are selected. Some apps may support only one context, such as a moderation app that works only in server context, but the default recommendation is to support both. For more detail on user-installed apps, see the [user-installable app tutorial](https://docs.discord.com/developers/tutorials/developing-a-user-installable-app).

### Setting up an install link

 [Install links](https://docs.discord.com/developers/resources/application#install-links) give users a simple way to install the app in Discord. This guide uses the default [Discord Provided Link](https://docs.discord.com/developers/resources/application#discord-provided-link); the [Application documentation](https://docs.discord.com/developers/resources/application#types-of-install-links) covers other install link types. On the [**Installation** page](https://discord.com/developers/applications/select/installation), find **Install Link** and choose **Discord Provided Link** if it is not already selected. Selecting it reveals **Default Install Settings**, which are configured next.

### Adding scopes and bot permissions

 Installing users must grant apps approval before the app can take Discord actions, such as creating slash commands or reading a server member list. Add scopes and bot permissions before installing the app.

What are scopes and permissions?

Introduction to scopes and bot permissions

When you create an app, scopes and permissions define what it may access and do in Discord.
- [OAuth2 Scopes](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-scopes) define data access and actions granted on behalf of an installing or authenticating user.
- [Permissions](https://docs.discord.com/developers/topics/permissions#permission-overwrites) are granular bot-user permissions, like the permissions regular Discord users have. The installing user can approve them, and they can later be changed in server settings or with [permission overwrites](https://docs.discord.com/developers/topics/permissions#permission-overwrites). Because user-context apps can only answer commands, these permissions matter only for server-installed apps.
 On the [**Installation** page](https://discord.com/developers/applications/select/installation), use the **Default Install Settings** section:
- For **User Install**, add the `applications.commands` scope
- For **Guild Install**, add the `applications.commands` scope and the `bot` scope. Selecting `bot` opens a **Permissions** menu for choosing bot-user permissions. Select the permissions the app may need; for now, select `Send Messages`.
 (Figure: Default Install Settings) See the full list of [OAuth2 scopes](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-scopes), or read the [permissions](https://docs.discord.com/developers/topics/permissions) documentation.

### Installing your app

 During development, test with your own user account for user-installable apps and with an inactive test server for server-installable apps. If you need a server, you can [create one for free](https://support.discord.com/hc/en-us/articles/204849977-How-do-I-create-a-server-). After adding scopes, copy the URL from **Install Link**. Because this app supports both installation contexts, install it to a test server and to your user account so both [installation contexts](https://docs.discord.com/developers/resources/application#installation-context) can be tested.

###### Install to server

 To install the app to a test server, copy the default Install Link from the **Installation** page. Paste it into a browser, press enter, and choose **Add to server** in the prompt. Select the test server and finish the installation. The app should then appear in the member list.

###### Install to user account

 Next, install the app to your user account. Paste the same Install Link in the browser and press enter. Choose **Add to my apps** in the prompt, then finish the installation. After installation, you can open a DM with the app.

---

### Step 2: Running your app

 After the app is configured and installed to the test server and account, inspect the code. The sample uses [discord-interactions](https://github.com/discord/discord-interactions-js) for types and helper functions. Developers using other languages or libraries should consult [Community Resources](https://docs.discord.com/developers/developer-tools/community-resources).

### Installing slash commands

 Slash command installation in the sample uses [`node-fetch`](https://github.com/node-fetch/node-fetch). The installation implementation is in `utils.js` inside `DiscordRequest()`. The project includes a `register` script that installs commands from `ALL_COMMANDS`, defined at the bottom of `commands.js`. It bulk-overwrites global commands by calling the HTTP API's [`PUT /applications/<APP_ID>/commands`](https://docs.discord.com/developers/interactions/application-commands#bulk-overwrite-global-application-commands) endpoint. To change or add commands, use the command structure in the [commands documentation](https://docs.discord.com/developers/interactions/application-commands#application-command-object-application-command-structure). From the project folder, run:

```
npm run register

```

 Returning to the server should show the slash commands. Running them will not produce behavior yet because the app is not receiving or handling Discord requests.

What are Discord's APIs?

Overview of Discord's HTTP and Gateway APIs

Apps can combine these [APIs](https://docs.discord.com/developers/quick-start/overview-of-apps#what-apis-can-apps-use) to build Discord functionality:
- **[HTTP API](https://docs.discord.com/developers/reference#http-api)** is a REST-like API for general operations such as sending, updating, or fetching Discord resource data.
- **[Gateway API](https://docs.discord.com/developers/reference#gateway-websocket-api)** is a WebSocket API for maintaining state or listening to events in a Discord server. This guide does not use it, but the [Gateway documentation](https://docs.discord.com/developers/events/gateway) explains how to create a Gateway connection and what events are available.

---

### Step 3: Handling interactivity

 To receive slash command requests and other interaction requests, the app needs a public Discord-accessible URL. Configure that URL in the app settings as **Interaction Endpoint URL**.

### Set up a public endpoint

 Create a public endpoint by starting the app's [Express](https://expressjs.com/) server and exposing it with [ngrok](https://ngrok.com/). First, open the project folder and run the command to start the app:

```
npm run start

```

 The output should indicate that the app is running on port `3000`. The sample is already prepared to handle Discord interactions by validating security request headers and answering `PING` requests. This tutorial skips many implementation details; [Interactions Overview](https://docs.discord.com/developers/interactions/overview#preparing-for-interactions) explains how to prepare apps for interactions. By default, the server listens on port 3000, but you can choose a different port by setting `PORT` in `.env`. Next, start the ngrok tunnel. If ngrok is not installed, follow the [ngrok download page](https://ngrok.com/download). Then open a new terminal and create a public endpoint that forwards traffic to the Express server:

```
ngrok http 3000

```

 The connection output should look similar to this:

```
Tunnel Status                 online
Version                       2.0/2.0
Web Interface                 http://127.0.0.1:4040
Forwarding                    https://1234-someurl.ngrok.io -> localhost:3000

Connections                  ttl     opn     rt1     rt5     p50     p90
                              0       0       0.00    0.00    0.00    0.00

```

 Use the **Forwarding** URL as the public URL that Discord sends interaction requests to in the next step.

### Adding an interaction endpoint URL

 Open your [app's settings](https://discord.com/developers/applications), then on the [**General Information** page](https://discord.com/developers/applications/select/information), paste the ngrok forwarding URL under **Interaction Endpoint URL** and append `/interactions`. (Figure: Interactions Endpoint URL) Click **Save Changes** and confirm the endpoint verifies. If verification fails, check that ngrok and the app are using the same port and that the ngrok URL was copied correctly. The sample app handles verification in two ways:
- It uses `PUBLIC_KEY` and the [discord-interactions package](https://github.com/discord/discord-interactions-js#usage) with a wrapper function imported from `utils.js`, matching [Express's `verify` interface](http://expressjs.com/en/5x/api.html#express.json). This runs for every incoming request.
- It answers incoming `PING` requests.
 The interactions documentation explains more about preparing an app to receive interactions: [the interactions documentation](https://docs.discord.com/developers/interactions/overview#preparing-for-interactions).

### Handling slash command requests

 After the endpoint is verified, open `app.js` in the project and find the block that handles the `/test` command:

```
// "test" command
if (name === 'test') {
  // Send a message into the channel where command was triggered from
  return res.send({
    type: InteractionResponseType.CHANNEL_MESSAGE_WITH_SOURCE,
    data: {
      flags: InteractionResponseFlags.IS_COMPONENTS_V2,
      components: [
        {
          type: MessageComponentTypes.TEXT_DISPLAY,
          // Fetches a random emoji to send from a helper function
          content: `hello world ${getRandomEmoji()}`
        }
      ]
    },
  });
}

```

 That code responds to the interaction by posting a message in the originating channel, DM, or Group DM. The documentation lists response types such as modals in the [interaction response object](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-response-object-interaction-callback-type). `InteractionResponseType.CHANNEL_MESSAGE_WITH_SOURCE` is a constant [exported from `discord-interactions`](https://github.com/discord/discord-interactions-js/blob/main/src/index.ts#L33). Go to the server and test the `/test` slash command. Triggering it should make the app send a message containing "hello world" followed by a random emoji. Next, add a command that uses slash command options, buttons, and select menus for the rock-paper-scissors game.

---

### Step 4: Adding message components

 The `/challenge` command starts the rock-paper-scissors-style game. When triggered, the app posts message components to the channel, guiding users through the game.

### Adding a command with options

 The `/challenge` command is named `CHALLENGE_COMMAND` in `commands.js` and contains an array of `options`. In the sample, those options are objects for selectable rock-paper-scissors choices, generated from the keys of `RPSChoices` in `game.js`. The [documentation](https://docs.discord.com/developers/interactions/application-commands#application-command-object-application-command-option-structure) covers command options and their structure. This guide does not focus on `game.js`, but you can inspect it and adjust commands or options.

Handling the command interaction

Code for handling the challenge command and returning a message with a button

To handle the `/challenge` command, place this code after the `if name === “test”` if block:

```
// "challenge" command
if (name === 'challenge' && id) {
  // Interaction context
  const context = req.body.context;
  // User ID is in user field for (G)DMs, and member for servers
  const userId = context === 0 ? req.body.member.user.id : req.body.user.id;
  // User's object choice
  const objectName = req.body.data.options[0].value;

  // Create active game using message ID as the game ID
  activeGames[id] = {
    id: userId,
    objectName,
  };

  return res.send({
    type: InteractionResponseType.CHANNEL_MESSAGE_WITH_SOURCE,
    data: {
      flags: InteractionResponseFlags.IS_COMPONENTS_V2,
      components: [
        {
          type: MessageComponentTypes.TEXT_DISPLAY,
          // Fetches a random emoji to send from a helper function
          content: `Rock papers scissors challenge from <@${userId}>`,
        },
        {
          type: MessageComponentTypes.ACTION_ROW,
          components: [
            {
              type: MessageComponentTypes.BUTTON,
              // Append the game ID to use later on
              custom_id: `accept_button_${req.body.id}`,
              label: 'Accept',
              style: ButtonStyleTypes.PRIMARY,
            },
          ],
        },
      ],
    },
  });
}

```
