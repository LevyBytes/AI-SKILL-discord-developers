# Configuring App Metadata For Linked Roles

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Linked roles in Discord require a user to connect an external service and satisfy configured conditions. The condition may be simply connecting that service, or it can be more specific, such as a verified account, required stats, or a follower threshold. Apps may publish their own [role connection metadata](https://docs.discord.com/developers/resources/application-role-connection-metadata), which server admins use when setting up linked roles for servers where the app is installed. The app also needs an [OAuth2 flow](https://docs.discord.com/developers/topics/oauth2) so users can authenticate and approve the `role_connections.write` scope. This tutorial builds a JavaScript Discord app that supports linked roles. The complete sample appears in the [`linked-roles-sample` GitHub repo](https://github.com/discord/linked-roles-sample).

---

### Creating an app

 Start by creating an app in the [developer dashboard](https://discord.com/developers/applications). If an app already exists, continue at [Running your app](https://docs.discord.com/developers/tutorials/configuring-app-metadata-for-linked-roles#running-your-app). The basic creation flow is listed below; the [Getting Started guide](https://docs.discord.com/developers/quick-start/getting-started) provides a fuller walkthrough.
- Open the [developer dashboard](https://discord.com/developers/applications)
- Select **New Application** in the upper-right corner, choose a name, and create the app
- Open the [**Bot** tab](https://discord.com/developers/applications/select/bot) from the left sidebar. On that page, choose **Reset Token** and save the token in a secure place, such as a password manager
 Bot tokens authorize API calls and represent the bot's permissions, so they are sensitive secrets. Do not share the token or commit it to version control.

### Adding scopes

 An installing user must approve the app before it can act within Discord. Add the scopes that the installation flow should request before installing the app.
- In the left sidebar, open [OAuth2](https://discord.com/developers/applications/select/oauth2/url-generator), then choose `URL generator`
- Check the `bot` scope
- After choosing the scope, the page shows a **Generated URL** that can install the app
 For more detail, review all [OAuth2 scopes](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-scopes) or the documentation on [user permissions](https://docs.discord.com/developers/topics/permissions).

### Installing your app

 Copy the **Generated URL** and open it in a browser. Complete the installation flow, choosing a server where you have room to develop and test. Once installation finishes, visit that server to confirm the app joined.

### Running your app

 The example app's full source is available in the [GitHub repository](https://github.com/discord/linked-roles-sample).

### Remix the project

 This guide uses Glitch so you can clone and edit the app directly in a browser. The README also includes local-development instructions with ngrok for developers who prefer that setup. Glitch is convenient for testing, but [it has technical limitations](https://help.glitch.com/kb/article/17-technical-restrictions/), so use another host for production. Begin by [remixing or cloning the Glitch project](https://glitch.com/edit/#!/remix/linked-role-discord-bot). After remixing, Glitch creates a new project with its own unique name, similar to the screenshot. (Figure: Glitch Remix)

#### Project structure

 The project files appear on the left side. The structure is roughly:

```
├── assets     -> images used in this tutorial
├── src
├──├── config.js  -> Parsing of local configuration
├──├── discord.js -> Discord specific auth & API wrapper
├──├── register.js -> Tool to register the metadata schema
├──├── server.js  -> Main entry point for the application
├──├── storage.js -> Provider for storing OAuth2 tokens
├── .env -> your credentials and IDs
├── .gitignore
├── package.json
└── README.md

```

### Configure your app

 `server.js` already contains starter code, but requests need the app token and IDs. Store these credentials in `.env`. Do not commit credentials or secrets; the starter project's `.gitignore` already includes `.env`. First copy the bot user's token from earlier into `DISCORD_TOKEN` in `.env`. Then open the [app settings in the developer portal](https://discord.com/developers/applications), go to [OAuth2 -> General](https://discord.com/developers/applications/select/oauth2), and copy the application's Client ID and Client Secret into `DISCORD_CLIENT_ID` and `DISCORD_CLIENT_SECRET` in `.env`. (Figure: Configure OAuth2) Next configure the Redirect URL for the OAuth2 flow. In Glitch, press `Share` and copy the public live app URL. (Figure: Glitch Share) Return to [OAuth2 -> General](https://discord.com/developers/applications/select/oauth2) in the Discord developer portal and add a redirect using the Glitch URL plus `/discord-oauth-callback`. Copy that redirect URL into `DISCORD_REDIRECT_URI` in `.env`. Then open the [General Information tab](https://discord.com/developers/applications/select/information), find `Linked Roles Verification Url`, and save the Glitch base URL plus `/linked-role`. For the screenshot project, the verification URL would be `https://adjoining-crawling-yamamomo.glitch.me/linked-role` . (Figure: Verify endpoint) Finally, return to Glitch, open the `Terminal` tab, and generate a unique cookie secret with these commands:

```
$ node
crypto.randomUUID()

```

 Copy the generated UUID and set it as `COOKIE_SECRET` in `.env`. The `.env` file should resemble this:

```
DISCORD_CLIENT_ID: <your OAuth2 client Id>
DISCORD_CLIENT_SECRET: <your OAuth2 client secret>
DISCORD_TOKEN: <your bot token>
DISCORD_REDIRECT_URI: https://<your-project-name>.glitch.me/discord-oauth-callback
COOKIE_SECRET: <random generated UUID>

```

### Registering your metadata schema

 As a one-time setup action, register the metadata fields that admins may use for linked roles tied to the app. Configure the app's connection metadata by calling [PUT /users/@me/applications/<application_id>/role-connection](https://docs.discord.com/developers/resources/application-role-connection-metadata#update-application-role-connection-metadata-records) with [application connection role metadata](https://docs.discord.com/developers/resources/application-role-connection-metadata#application-role-connection-metadata-object). In the sample app, [`src/register.js`](https://github.com/discord/linked-roles-sample/blob/main/src/register.js) performs this registration and can be invoked from the command line. In Glitch, open the **terminal** tab and run:

```
$ node src/register.js

```

 (Figure: Register Metadata Schema)

### Trying it out

 After building the app, test it from both viewpoints: the server owner configuring the linked role and the user trying to receive it.

### Creating the linked role

 To test the app, use a server where your account has admin permissions and add a linked role there. Open **Server Settings**, choose **Roles**, then click `Create Role`. Name and save the role, then open `Links`. Click `Add requirement`; your bot should appear in the Apps list. Select it to open a setup screen where the role criteria can be configured. (Figure: Verification Setup)

### Acquiring the role

 To claim the role, open the server menu from the upper-left server name and choose `Linked Roles`. Select your role, then start the account-connection flow. The OAuth flow requests `role_connections.write`, which the app needs to update the user's role connection details. (Figure: Connect accounts) Click the linked role criteria to reach the Discord OAuth2 consent page. Choose `Authorize`, then return to Discord. (Figure: Consent Dialog) After returning, the account should have the linked role. (Figure: Connected) Finally, create a private channel and add that linked role.

### Tips & Tricks

### Token storage

 This app depends mainly on Discord's [OAuth2](https://docs.discord.com/developers/topics/oauth2) support for access tokens. User-based authentication in this pattern stores refresh tokens and uses them to request access tokens. The sample's [`src/storage.js`](https://github.com/discord/linked-roles-sample/blob/main/src/storage.js) keeps tokens in memory, but production systems should store them persistently in a database.

### Advanced examples

 For a fuller Fitbit API example, see [https://github.com/JustinBeckwith/fitbit-discord-bot/](https://github.com/JustinBeckwith/fitbit-discord-bot/).
