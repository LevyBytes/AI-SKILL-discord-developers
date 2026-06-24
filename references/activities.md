# How Activities Work, Overview

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: Launching Activities; Build Your First Discord Activity.

## Launching Activities

Activities are web apps loaded in an iframe inside Discord across desktop, mobile, and web clients. Secure communication between the Activity and Discord uses the [`postMessage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) protocol. The [Embedded App SDK](https://github.com/discord/embedded-app-sdk) handles that `postMessage` work for the app. For command availability and usage details, see the [SDK Reference](https://docs.discord.com/developers/developer-tools/embedded-app-sdk). The [Sample Projects](https://docs.discord.com/developers/activities/overview#sample-projects) show practical implementations.

### Launching Activities

 After Activities are enabled in the Application's [Activity's settings](https://discord.com/developers/applications/select/embedded/settings), the app can start Activities in either of these ways:
1. A user runs the app's Entry Point command from the App Launcher
2. The app responds to an interaction using the `LAUNCH_ACTIVITY` callback type
 The following sections describe both options in more depth.

### Entry Point Command

 The usual launch path for Activities is a user's [Entry Point command](https://docs.discord.com/developers/interactions/application-commands#entry-point-commands) in the App Launcher. Enabling Activities creates a [default Entry Point command](https://docs.discord.com/developers/interactions/application-commands#default-entry-point-command) named "Launch". By default, Discord opens the Activity automatically when a user runs the Entry Point command. The [development guide](https://docs.discord.com/developers/activities/development-guides/user-actions#setting-up-an-entry-point-command) covers Entry Point command setup.

### Interaction Response

 An Activity may also open from [command](https://docs.discord.com/developers/interactions/overview#commands), [message component](https://docs.discord.com/developers/interactions/overview#message-components), or [modal submission](https://docs.discord.com/developers/interactions/overview#modals) interactions. To launch one, respond to the interaction with callback type `LAUNCH_ACTIVITY` (type `12`) as described in [responding to the interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#responding-to-an-interaction).

---

### Designed for Single-Page Apps (SPAs)

 The SDK is designed for a single-page application. If a project uses a framework or architecture that does not naturally match a single-page app, place that framework inside the Activity's top-level single-page application and pass messages as needed. The [Nested Messages App](https://docs.discord.com/developers/activities/overview#sample-projects) sample demonstrates this pattern.

### Activity Lifecycle

1. **Initialization:** When Discord loads the iframe, its URL includes unique query parameters. The application can identify those parameters with the Discord SDK.
2. **Handshake Process:** Creating the SDK instance starts a handshake with the Discord client. After connection, the iframe receives `[FRAME, {evt: 'READY', ...}]`. The SDK instance's `ready()` method resolves after the connection succeeds.
3. **Authorization and Authentication:** After the `READY` payload arrives, the application should run authorization and authentication to obtain the required permissions or scopes. This is necessary before using features or scopes such as `rpc.activities.write`.
4. **Interacting with Discord Client:** Once authenticated, the application may subscribe to events and send commands to the Discord client. Commands or event subscriptions outside the granted scope return errors. Adding scopes can cause an OAuth modal to ask the user to confirm permission again.
5. **Disconnection and Errors:** A `[CLOSE, {message: string, code: number}]` message means the connection process encountered an error or must be restarted.
6. **Sending Errors or Close Requests:** To report an error or ask the Discord client to close, send `[CLOSE, {message?: string, code: number}]`. Codes other than CLOSE_NORMAL show the message to the user; CLOSE_NORMAL closes silently.

### Sample Code and Activity Lifecycle Diagram

 The following minimal example initializes the SDK. For more complete applications, see the [Sample Projects](https://docs.discord.com/developers/activities/overview#sample-projects).

```
import {DiscordSDK} from '@discord/embedded-app-sdk';
const discordSdk = new DiscordSDK(YOUR_OAUTH2_CLIENT_ID);

async function setup() {
  // Wait for READY payload from the discord client
  await discordSdk.ready();

  // Pop open the OAuth permission modal and request for access to scopes listed in scope array below
  const {code} = await discordSdk.commands.authorize({
    client_id: YOUR_OAUTH2_CLIENT_ID,
    response_type: 'code',
    state: '',
    prompt: 'none',
    scope: ['identify'],
  });

  // Retrieve an access_token from your application's server
  const response = await fetch('/.proxy/api/token', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      code,
    }),
  });
  const {access_token} = await response.json();

  // Authenticate with Discord client (using the access_token)
  auth = await discordSdk.commands.authenticate({
    access_token,
  });
}

```

 This diagram shows the message flow between the application and Discord for the sample above. (Figure: Diagram of how Activities communicate with Discord)

## Build Your First Discord Activity

(Figure: Building Discord Activities) Activities are iframe-hosted web apps that communicate with Discord clients through the [Embedded App SDK](https://docs.discord.com/developers/activities/overview#embedded-app-sdk). For a higher-level introduction, read the [Activities](https://docs.discord.com/developers/platform/activities) platform page. This section explains Activity construction. To see production examples, review the [developer case studies](https://discord.com/build#case-studies), or use the resources below to start building.

### Build Your First Discord Activity

Use the guide to create your first Activity with the Embedded App SDK.

### Development Patterns

Review common development practices and patterns that simplify Activity work.

### How Activities Work

Study the Activity lifecycle and how Activities execute inside Discord clients.

---

### Developing Activities

 An Activity, whether it is a multiplayer game, social experience, or another idea, is built as a web app running in an iframe inside Discord on desktop, mobile, and web. Start with [Building an Activity using the Embedded App SDK](https://docs.discord.com/developers/activities/building-an-activity), or browse the [sample projects](https://docs.discord.com/developers/activities/overview#sample-projects). The sections below introduce the Embedded App SDK; for runtime mechanics, see [How Activities Work](https://docs.discord.com/developers/activities/how-activities-work).

### Embedded App SDK

 The [Embedded App SDK](https://docs.discord.com/developers/developer-tools/embedded-app-sdk) manages communication between Discord and the app. It helps with common Activity needs such as connected-user management, mobile-client support, and debugging. The SDK exposes events and commands for Discord-to-Activity communication, described below.

#### Commands

 The SDK provides commands for interacting with a Discord client when the app has the required scopes. Use them to authorize and authenticate users, retrieve Activity information, or update Activity or authenticated-user data. The [Embedded App SDK documentation](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#sdk-commands) lists every command and explains each one.

#### Events

 The SDK also supports subscribing and unsubscribing to events from the connected client, when the granted scopes allow it. These events help with initial connection handling, errors, connected-user updates, and Activity-instance changes. The [Embedded App SDK documentation](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#sdk-events) lists all events and their details.

---

### Sample Projects

 Use the sample projects to start quickly and learn common implementation patterns. Community-maintained samples are also listed on [GitHub](https://github.com/discord/embedded-app-sdk-examples?tab=readme-ov-file#community-curated-examples), including framework-specific examples.

### Discord Activity Starter

This TypeScript starter template is suited for quickly bringing up a new Activity.

### Embedded App SDK Playground

This reference example covers the Embedded App SDK commands and events available to an app.

### Nested Framework App

This reference example shows an Activity that nests another framework, such as a game engine.

[How Activities WorkNext](https://docs.discord.com/developers/activities/how-activities-work)I
