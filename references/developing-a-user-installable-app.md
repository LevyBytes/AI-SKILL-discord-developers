# Developing A User Installable App

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Discord apps can be installed to servers, users, or both. This guide builds a simple game integration app that supports installation by both Discord users and servers. The tutorial centers on [installation contexts](https://docs.discord.com/developers/resources/application#installation-context) while creating a game integration with wiki lookup, user-specific bookmarking, and a server leaderboard. The app defines four commands (`/link`, `/profile`, `/leaderboard`, and `/wiki`) that run in different installation and interaction contexts, which the tutorial explains as it progresses.

Resources used in this guide

Tools and technologies used in this guide

- **[GitHub repository](https://github.com/discord/user-install-example)** containing the guide's sample code.
- **[discord-interactions](https://github.com/discord/discord-interactions-js)**, a library with types and helper functions for Discord apps.
- **[Express](https://expressjs.com/)**, the JavaScript web framework used to create the server that receives Discord requests.
- **[ngrok](https://ngrok.com/)**, a tunneling tool that exposes the local server at a public URL for Discord requests.

---

### Step 0: Project Setup

 Start by getting the project code from the [sample app repository](https://github.com/discord/user-install-example).

Project structure

Project structure overview for the sample app used in this tutorial

```
├── .env.sample -> sample .env file
├── app.js      -> main entrypoint for the app
├── commands.js -> slash command payloads + helpers
├── game.js     -> logic specific to the fake game
├── utils.js    -> utility functions and enums
├── package.json
├── README.md
└── .gitignore

```

 This tutorial uses [ngrok](https://ngrok.com/) with local development, though you can use another development setup. If [NodeJS](https://nodejs.org/en/download/) is not installed, install it first. Then clone the project code from the command line:

```
git clone https://github.com/discord/user-install-example.git

```

 Next, move into the directory and install the project's dependencies:

```
# navigate to directory
cd user-install-example

# install dependencies
npm install

```

 After setup, open the project in your preferred code editor and continue with the Discord app configuration.

### Step 1: Creating an App

 Create an app in the developer portal if you do not already have one: [Create App](https://discord.com/developers/applications?new_application=true) Enter the app name and press **Create**. After creation, the app opens on the **General Information** settings page, where you can edit basic details such as description and icon.

### Fetching app credentials

 From the app settings, collect the sensitive values needed by the sample, including the token and ID. The token authorizes API requests and carries the app's permissions, so treat it as highly sensitive: never share it or commit it to version control. In the project folder, rename `.env.sample` to `.env`. The `.env` file stores the app credentials. Add these three values to `.env`:
- On the [**General Information** page](https://discord.com/developers/applications/select/information), copy **Application ID**. In `.env`, replace `<YOUR_APP_ID>` with that ID.
- On the [**General Information** page](https://discord.com/developers/applications/select/information), copy **Public Key**, which verifies that HTTP requests came from Discord. In `.env`, replace `<YOUR_PUBLIC_KEY>` with that value.
- On the [**Bot** page](https://discord.com/developers/applications/select/bot), find **Token** and click "Reset Token" to generate a new bot token. In `.env`, replace `<YOUR_BOT_TOKEN>` with the new token.
 With credentials in place, configure the app for multiple installation contexts.

### Add Guild Members intent

 The sample app builds a fake game leaderboard by fetching server members. Fetching members requires the [privileged intent](https://docs.discord.com/developers/events/gateway#privileged-intents) for server members. Open the **Bot** page, find **Privileged Gateway Intents**, and turn on "Server Member Intent".

### Choosing Supported Installation Contexts

 An app's **[installation context](https://docs.discord.com/developers/resources/application#installation-context)** defines whether it is installed to a server, to a user, or to both. This sample supports both contexts, which is a useful default for many apps, though some apps fit only one context. In the app settings, open the [**Installation** page](https://discord.com/developers/applications/select/installation) from the sidebar. Under **Installation Contexts**, enable **User Install** and **Guild Install**, then press **Save Changes**.

### Configuring Default Install Settings

 Default install settings define the default [scopes](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-scopes) and [bot user permissions](https://docs.discord.com/developers/topics/permissions) for each supported installation context. User-context installs currently support only the `applications.commands` scope in default install settings; that scope lets the app install [commands](https://docs.discord.com/developers/interactions/application-commands).

###### Update Install Link

 Before adding default install settings, set the app's [install link](https://docs.discord.com/developers/resources/application#install-links) to Discord Provided Link. In **Install Link**, choose `Discord Provided Link` from the dropdown if it is not already selected. After that selection, **Default Install Settings** appears.

###### Adding Default Install Settings

 In **Default Install Settings**:
- For **User Install**, add the `applications.commands` scope
- For **Guild Install**, add the `applications.commands` scope and the `bot` scope. Selecting `bot` reveals a **Permissions** menu for bot user permissions. Choose the permissions your app needs; for the sample, select `Send Messages`.
 Bot user permissions work much like permissions for other Discord users. The [Permissions](https://docs.discord.com/developers/topics/permissions#permissions-bitwise-permission-flags) page lists available permissions and explains them. After selecting the app's scopes and permissions, click **Save Changes**. (Figure: Installation settings in App Settings)

### Installing your app

 Install the app to both a test server and your user account so both installation contexts can be tested.

###### Install to server

 To install the app to a test server, copy the default Install Link from the **Installation** page. Paste it into your browser, press enter, and choose "Add to server" in the prompt. Select the test server and finish the installation flow. After installation, the app should appear in the server member list.

###### Install to user account

 Next, install the app to your user account. Paste the same Install Link into your browser and press enter. This time, choose "Add to my apps" in the prompt and complete installation. After it is installed, you can open a DM with the app.

### Step 2: Setting Up Commands

 Next, register the [application commands](https://docs.discord.com/developers/interactions/application-commands) for the app. Before changing code, understand command contexts:

Contexts for application commands

Short introduction to command contexts

Commands have two context fields that can be set when a command is created or updated. These fields limit the supported install methods and Discord surfaces for that command:
- **`integration_types`** controls which **[installation contexts](https://docs.discord.com/developers/interactions/application-commands#installation-context)** support a command: user, guild, or both. For example, the `/link` and `/profile` commands created in this guide are available only when the app is installed to a user.
- **`contexts`** sets the **[interaction contexts](https://docs.discord.com/developers/interactions/application-commands#interaction-contexts)** where a command can be used, such as a guild channel, a DM with the bot user, or other DMs and GDMs. For example, the `/leaderboard` command created here is available only from a guild channel.
The [contexts](https://docs.discord.com/developers/interactions/application-commands#contexts) documentation has deeper command context details. In this tutorial, the sample app makes those concepts concrete by using them.

### Commands in the sample project

 The sample app defines four commands with different contexts, summarized in the table below:
| Name | Description | Installation Contexts (`integration_types`) | Interaction Contexts (`contexts`) |
| --- | --- | --- | --- |
| `/leaderboard` | Show the game leaderboard for the current server | `GUILD_INSTALL` | `GUILD` |
| `/wiki` | Look up details about game items and characters | `GUILD_INSTALL`, `USER_INSTALL` | `GUILD`, `BOT_DM`, `PRIVATE_CHANNEL` |
| `/profile` | Show your game inventory and progress | `USER_INSTALL` | `GUILD`, `BOT_DM`, `PRIVATE_CHANNEL` |
| `/link` | Connect your game account to Discord | `USER_INSTALL` | `BOT_DM` |

 A command's supported installation contexts affect which interaction contexts it may use. `PRIVATE_CHANNEL` can be included in `contexts` only when `USER_INSTALL` is included in that command's `integration_types`. See the [documentation](https://docs.discord.com/developers/interactions/application-commands#contexts) for details. The command payloads live in `commands.js` in the project folder, where you can change values or inspect the `integration_types` and `contexts` fields for each command in the table.

### Registering the commands

 Register the app's commands so they appear in Discord. From the project folder, run:

```
npm run register

```

 The register command calls the [Create Global Application Command](https://docs.discord.com/developers/interactions/application-commands#create-global-application-command) endpoint for each payload in `commands.js`. After the commands are created, open Discord and check the surfaces where they were made available:
- In channels for the guild where the app was installed, `/leaderboard`, `/wiki`, and `/profile` should be available
- In **channels within any of your guilds**, you should see `/wiki` and `/profile`
- In **your app's DM**, you should see `/wiki`, `/profile`, and `link`
- In **DMs or GDMs with other users**, you should see `/wiki` and `/profile`
 The commands will error until the interaction handler is configured.

### Step 3: Handling Interactivity

 To receive and process interactive requests, configure an **Interactions Endpoint URL**, a public URL where Discord sends the app's interactions.

### Set up a public endpoint

 Create a public endpoint by starting the app's [Express](https://expressjs.com/) server, then exposing it with [ngrok](https://ngrok.com/). First, go to the project folder and run this command to start the app:

```
npm run start

```

 The output should show that the app is running on port 3000, with Discord interaction handling already wired for security header verification and `PING` responses. This tutorial skips many implementation details; the [Interactions Overview](https://docs.discord.com/developers/interactions/overview#preparing-for-interactions) explains how to prepare apps for interactions. By default, the server listens on port 3000, but you can set a `PORT` variable in `.env` to change it. Next, start the ngrok tunnel. If ngrok is not installed locally, follow the [ngrok download page](https://ngrok.com/download). After installation, open a new terminal and create a public endpoint that forwards to the Express server:

```
ngrok http 3000

```

 The ngrok output includes a **Forwarding** URL. Use that public URL as the Interactions Endpoint URL in the next step.

### Configuring an interaction endpoint URL

 Configure the app's **Interaction Endpoint URL** in the [app's settings](https://discord.com/developers/applications). On the [**General Information** page](https://discord.com/developers/applications/select/information), under **Interaction Endpoint URL**, paste the ngrok URL and append `/interactions`, for example `https://84c5df474.ngrok-free.dev/interactions` (Discord should verify the endpoint if everything is running). Click **Save Changes**. If verification fails, confirm that ngrok and the app use the same port and that the ngrok URL was copied correctly.

### Understanding metadata for interactions

 After the Interactions Endpoint URL is configured, the app's commands should run. Open the app's DM and run `/profile`; the app should return a sample game profile. The command line logs incoming Discord requests, so you can inspect the request body for the command invocation.

Sample interaction payload

The payload below is shortened for readability, but the interaction request body should look similar to this:

```
{
  "app_permissions": "442368",
  "application_id": "234248956100616262",
  "authorizing_integration_owners": { "1": "1090372582781497424" },
  "channel": {
    // Partial channel object corresponding to channel_id
  },
  "channel_id": "1234563982236504123",
  "context": 1,
  "data": { "id": "1234358421659193405", "name": "link", "type": 1 },
  "entitlements": [],
  "id": "1234968734674853908",
  "locale": "en-US",
  "token": "a really long interactions token that your app can use to respond to the interactions",
  "type": 2,
  "user": {
    // Partial user object
  },
  "version": 1
}

```

 The [`data` object](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-data) identifies which command ran. This tutorial focuses on metadata tied to installation and interaction contexts. When building an app that supports multiple interaction contexts, watch these metadata fields:

###### `context`

 `context` identifies the [interaction context](https://docs.discord.com/developers/interactions/application-commands#interaction-contexts) where the command was invoked. In the sample, the command came from the app's DM, so `context` is `1` (`BOT_DM`). `BOT_DM` means only the DM with your bot user. The same command in a DM with another user or in a group DM uses `PRIVATE_CHANNEL` (`2`).

###### `authorizing_integration_owners`

 `authorizing_integration_owners` contains IDs related to the installation context or contexts for the interaction. Object keys represent the relevant installation contexts (`GUILD_INSTALL`/`"0"` and/or `USER_INSTALL`/`"1"`). Values depend on the key, but for `USER_INSTALL`, the value is always the ID of the user who authorized the app. `authorizing_integration_owners` is not the same as the user who triggered the interaction; the triggering user appears in the `user` object. Knowing the authorization owner helps when handling message component interactions for user-installed apps, discussed in [message component interactions](https://docs.discord.com/developers/tutorials/developing-a-user-installable-app#using-metadata-for-message-component-interactions). Technical details are in [Authorizing Integration Owners](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-authorizing-integration-owners-object).

###### `app_permissions`

 `app_permissions` is the bitwise permission set the app has where the interaction was triggered. The value changes between app DMs, servers, and G(DM)s with other users. The sample payload uses `"442368"`. These permissions can guide how the app should [respond to the interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#responding-to-an-interaction). For example, the sample app responds ephemerally for the `/profile` command when it is invoked from a server.

### Using metadata for command interactions

 The sample app's `/profile` command responds ephemerally when invoked from a server, so only the invoking user sees it. When invoked in a DM with the bot user, it sends a non-ephemeral response. The `app.js` handler for `/profile` contains this logic:

Using context for the profile command

The logic below handles `/profile`. When the interaction context is a server, the response is ephemeral and includes a button the user can use to share the profile.

```
// "profile" command
if (name === 'profile') {
  const profile = getFakeProfile(0);
  const profileEmbed = createPlayerEmbed(profile);

  // Use interaction context that the interaction was triggered from
  const interactionContext = req.body.context;

  // Construct `data` for our interaction response. The profile embed will be included regardless of interaction context
  let profilePayloadData = {
    embeds: [profileEmbed],
  };

  // If profile isn't run in a DM with the app, we'll make the response ephemeral and add a share button
  if (interactionContext !== 1) {
    // Make message ephemeral
    profilePayloadData['flags'] = 64;
    // Add button to components
    profilePayloadData['components'] = [
      {
        type: 1,
        components: [
          {
            type: 2,
            label: 'Share Profile',
            custom_id: 'share_profile',
            style: 2,
          },
        ],
      },
    ];
  }

  // Send response
  return res.send({
    type: InteractionResponseType.CHANNEL_MESSAGE_WITH_SOURCE,
    data: profilePayloadData,
  });
}

```

 In the sample app, the request body's `context` value is assigned to `interactionContext`. The handler uses that context to choose the response behavior. If the command ran in a guild or in a G(DM) other than the app bot user's DM, `flags` is set to `64` for an ephemeral response, and a button component is added so the user can share the profile.

### Using metadata for message component interactions

 Message component interactions can be triggered by any user who can see the component, regardless of installation context. Because the triggering user may be anyone with visibility, metadata helps identify the relevant installation context. Suppose you add a `/game` command for the `USER_INSTALL` installation context that lets a user post a message in the current guild or group DM asking who wants to join a game match. (Figure: Sample game command response) When another user clicks the button, the app needs to track two users:
1. User B, who clicked the "Join" button, so the app can record interest in joining the match
2. User A, who ran the `/game` command, so the app can tell them who wants to join the match
 Two additional fields are useful in this scenario:

###### `interaction_metadata`

 Messages created as interaction responses include an [`interaction_metadata` object](https://docs.discord.com/developers/resources/message#message-interaction-metadata-object) with metadata from the interaction.

###### `authorizing_integration_owners`

 `authorizing_integration_owners` matters most for message component interactions on user-installed apps. It can distinguish the user who installed the app from the user who triggered the interaction, because messages created by interaction responses or follow-up messages may be visible to users who have not installed the app to their own account.

### Next Steps

 At this point, the app supports both installation contexts and demonstrates the basics of using metadata for context-sensitive behavior. From here, continue into the documentation for details or extend the sample app with more complex features.

### Interactions documentation

Use the Interactions documentation for deeper coverage of command and message-component receive/respond flows

### Github repository

GitHub repository with sample project
