# Account Linking From Discord, Account Linking On Consoles

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: Overview.

## Overview

### Overview

 If account linking is new to you, start with the [Account Linking overview](https://docs.discord.com/developers/platform/account-linking) for the concept, mechanics, and flow selection. Discord can improve player choice by showing prompts and buttons inside the Discord client that invite users to link accounts with your game. These account-linking entry points can raise link rates, which strengthens social features, engagement, and the relationship between your game and Discord users. This guide explains how to let Discord show those prompts to players.

### Prerequisites

 Before turning on these entry points, make sure you have:
- **Discord application setup**
  - Created a Discord application in the [Developer Portal](https://discord.com/developers/applications)
  - Configured [OAuth2 settings](https://discord.com/developers/applications/select/oauth2) with the correct redirect URLs

- **Implemented account linking in your game**
  - Downloaded Discord Social SDK and integrated it into the game
  - Confirmed SDK initialization works, including connection to Discord
  - Implemented the [Basic account linking flow](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord) in the game

- **Development environment ready**
  - Confirmed the game can authenticate users with Discord OAuth2

 If any prerequisite is still missing, Discord recommends completing the [Getting Started](https://docs.discord.com/developers/discord-social-sdk/getting-started) guide first.

---

### Entry Points & Account Linking Flows

### What are Entry Points?

 Entry points are prompts, buttons, or other UI elements that Discord places around the Discord client to encourage users to link accounts. They function like Ã¢â‚¬Å“Link your accountÃ¢â‚¬Â affordances in contexts where a user may encounter your game.

### Example of an Entry Point

 Discord decides which entry points to display automatically based on implemented features and whether the game is open at that moment. (Figure: Discord account-linking entry points) Current examples include: **Game Detection** Discord notices that the player is in your game and offers a link option if the account is not already linked. (Figure: game-detection account linking prompt) **Game Invites** An invite to your game can include a link option so players can prepare their account and join friends sooner. (Figure: account-linking entry points in Game Invites) **Claiming Game Shop Items** Redeeming Game Shop items requires account linking so rewards can be delivered in game. (Figure: Game Shop entry points for account linking)

### Account Linking Flows

 Discord currently offers two client-side account linking paths: connected game flow and web flow. After a user chooses an entry point in Discord, the client sends them through one available path. For user experience, Discord checks them in this order:
- [Connected game flow](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-from-discord#connected-game-flow)
- [Web flow](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-from-discord#web-flow)
 Choose the flow or flows that fit the game, then follow the relevant implementation steps. Supporting both can give players more account-linking options.

---

### Connected Game Flow

 Discord prefers the connected game flow when available because it gives the best user experience. This flow sends the user into the game client to start account linking.

### Prerequisites

- Use SDK version 1.6 or higher
- Implement [account linking](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord) in your game

### Implementing the Connected Game Flow

 After adopting SDK version 1.6 or higher, the Client object includes two methods: [`Client::RegisterAuthorizeRequestCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a5f34b873e127a446c9ab549e4588ccd7) and [`Client::RemoveAuthorizeRequestCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ab7e48864b0cedf3e8572a228ca401f2a). These methods enable and disable the connected game flow while the game is running. Before invoking [`Client::RegisterAuthorizeRequestCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a5f34b873e127a446c9ab549e4588ccd7), call [`Client::SetApplicationId`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ad452335c06b28be0406dab824acccc49) so the game identifies itself to the Discord client and requests are tied to the correct application. Calling [`Client::RegisterAuthorizeRequestCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a5f34b873e127a446c9ab549e4588ccd7) tells the Discord client that the game is running and can begin account linking. Do this as soon as the game can handle an account link, such as after the user signs in or reaches the main menu. When a user selects an account-linking entry point in Discord, the SDK invokes the callback. Treat that event like a click on an in-game account-linking button and begin authorization immediately. Because the user is probably focused on Discord rather than the game, bring attention back to the game client or otherwise notify them. Discord also tells the user to check the game. For the smoothest flow, pull focus back into the game. Calling [`Client::RemoveAuthorizeRequestCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ab7e48864b0cedf3e8572a228ca401f2a) tells Discord the game can no longer start account linking. Call it when the game enters a state where account linking would fail or disrupt play, such as during a match, after logout, or in a full-screen flow like a cutscene. That is sufficient to enable the connected game flow, and you can test or debug it with the [built-in developer tools](https://docs.discord.com/developers/discord-social-sdk/how-to/debug-log#client-tools) in the Discord client.

```
// If you haven't already
client.SetApplicationId("123456789");

client.RegisterAuthorizeRequestCallback([client, myGameAuthorizationHandler]() {
	// Pull focus into your game using native APIs
	// ...

	// Run your normal authorization flow
	myGameAuthorizationHandler.StartDiscordAccountLink();
});

```

### Connected Game Best Practices

 Selecting an account-link button in Discord is a high-intent action. Avoid extra upsell steps until after the account is linked, because added steps increase drop-off points. Show upsells afterward to reinforce new features or rewards unlocked by linking. Enable and disable the callback at the right times; launching account linking from Discord during a match can interrupt gameplay and break the intended experience. You do not need to check whether the user is already linked, because the Discord client hides the entry point for linked users. Relying on that behavior keeps the implementation simpler and reduces failure surface.

### Web Flow

 The web flow sends users to a webpage to begin account linking instead of using the connected game flow. This flow is currently limited to selected partners.

### Prerequisites

- Implement the [standard OAuth2 flow](https://docs.discord.com/developers/topics/oauth2) on your website
- In the [Developer Portal](https://discord.com/developers/applications), set **Connection Entrypoint URL** to the URL that begins the OAuth2 web flow

### Implementing the Web Flow

 Make sure the webpage satisfies these requirements:
- If the user is not signed in to the game account, prompt sign-in and then return them directly to authorization.
- Request the same scopes during authorization that the game requests.
- After authorization, save the token to the user's account the same way in-game authorization would, so it is ready on the next game launch.
- If the user is already in game, send the token update to the client and call [`Client::UpdateToken`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a606b32cef7796f7fb91c2497bc31afc4) so the user signs in immediately without restarting.

---

### Next Steps

 After account linking with Discord is implemented, you can add more social features to the game.

### Design: Signing In

Design guidance for account linking and user authentication

### Creating a Unified Friends List

Merge Discord friends and game friends into one list for simpler management.

### Setting Rich Presence

Show game status and details to Discord friends. Need help? Join the [Discord Developers Server](https://discord.gg/discord-developers) and ask in `#social-sdk-dev-help` for community support. If you find a Social SDK bug, report it at [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report)

---

### Change Log

| Date | Changes |
| --- | --- |
| December 16, 2025 | initial release |

## Overview

### Overview

 If account linking is new to you, start with the [Account Linking overview](https://docs.discord.com/developers/platform/account-linking) for the concept, mechanics, and flow selection. This guide explains OAuth2 device authorization for console users so they can log in to the game with Discord. It covers how to:
- Initiate OAuth2 device authorization
- Authenticate users with QR codes or manual entry
- Trade the device code for an access token
- Use the SDK's automatic token handling

### Prerequisites

 Before starting, verify that you have:
- Read the [Core Concepts](https://docs.discord.com/developers/discord-social-sdk/core-concepts) guide to understand:
  - OAuth2 authentication flow
  - Discord application setup
  - SDK initialization

- Prepared your development environment with:
  - Discord application created in the [Developer Portal](https://discord.com/developers/applications)
  - Discord Social SDK downloaded and configured
  - Basic SDK integration working, including initialization and connection

 Console games using the Discord Social SDK require middleware approval and approved-developer status for the target console. See [this article](https://support-dev.discord.com/hc/en-us/articles/30209074764183) for details. If prerequisites are incomplete, Discord recommends following the [Getting Started](https://docs.discord.com/developers/discord-social-sdk/getting-started) guide.

---

### OAuth2 Device Authorization Flow

 Console users cannot authenticate through a browser inside the console experience. Instead, the process is:
1. The game asks Discord for a device code
2. The user approves by scanning a QR code or by entering a user code at `discord.com/activate`.
3. The game polls the Discord API and exchanges the device code for an access token.
 The SDK can complete the process automatically, or the game can handle tokens itself. OAuth2 device authorization works only for verified user accounts. (Figure: Authorization screen using OpenAuthorizeDeviceScreen and GetTokenFromDevice)

---

### Automatic Token Handling

 This approach requires **Public Client** to be enabled for the app, which most games should avoid shipping with. [Learn more](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes#oauth2-client-types) The SDK performs the complete OAuth2 device authorization process with one call to [`Client::GetTokenFromDevice`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a4a9c666b2d30bae0a16f5afd7ccee60d), including polling for the access token. After obtaining the access token, authenticate with [`Client::UpdateToken`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a606b32cef7796f7fb91c2497bc31afc4) and call [`Client::Connect`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a873a844c7c4c72e9e693419bb3e290aa) to connect to Discord.

```
discordpp::DeviceAuthorizationArgs args{};
args.SetClientId(APPLICATION_ID);
args.SetScopes(discordpp::Client::GetDefaultPresenceScopes());

client->GetTokenFromDevice(args, [client](
  discordpp::ClientResult result,
  std::string accessToken,
  std::string refreshToken,
  discordpp::AuthorizationTokenType tokenType,
  int32_t expiresIn,
  std::string scope) {
    if (result.Successful()) {
      std::cout << "🔓 Access token received! Establishing connection...\n";
      client->UpdateToken(discordpp::AuthorizationTokenType::Bearer, accessToken, [client](discordpp::ClientResult result) {
        client->Connect();
      });
    } else {
      std::cerr << "❌ Failed to get token from device \n";
    }
  });

```

---

### Manual Flow for Console Authorization

 For manual console authorization, use these steps. The flow is the same OAuth2 device authorization process as the automatic method, but your code handles polling and token exchange.
1. [Request a device code from Discord](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-on-consoles#step-1-request-a-device-code-from-discord)
2. [Display the user verification information](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-on-consoles#step-2-display-authorize-screen-with-qr-code-and-user-code) or optionally open the authorization screen
3. [Poll for the userÃ¢â‚¬â„¢s authorization](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-on-consoles#step-3-poll-for-users-authorization)
4. [Exchange the device code for an access token](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-on-consoles#step-4-exchange-device-code-for-access-token)
5. [Handle the token response](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-on-consoles#step-5-handle-token-response) and optionally close the authorization screen

### Step 1: Request a Device Code from Discord

 The game needs to request both a device code and a user code from the Discord API.

```
import requests

API_ENDPOINT = "https://discord.com/api/v10"
CLIENT_ID = "YOUR_CLIENT_ID"
CLIENT_SECRET = "YOUR_CLIENT_SECRET"
SCOPE = "sdk.social_layer"

def authorize_device():
    data = {"scope": SCOPE}
    headers = {"Content-Type": "application/x-www-form-urlencoded"}

    response = requests.post(f"{API_ENDPOINT}/oauth2/device/authorize", data=data, headers=headers, auth=(CLIENT_ID, CLIENT_SECRET))
    response.raise_for_status()
    return response.json()

```

#### Example Response

```
{
  "device_code": "<device code>",
  "user_code": "<user code>",
  "verification_uri": "https://discord.com/activate",
  "verification_uri_complete": "https://discord.com/activate?user_code=<user code>",
  "expires_in": 300,
  "interval": 5
}

```

### Step 2: Display Authorize Screen with QR Code and User Code

 After the user code is requested, call [`Client::OpenAuthorizeDeviceScreen`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#af29a13acc992a75fc0870051ff68575b) to open an authorization screen.

```
client->OpenAuthorizeDeviceScreen(user_code);

```

 A manual-entry UI can show `verification_uri_complete`, or show `verification_uri` together with `user_code`, so the user can type the code. (Figure: Authorization screen using OpenAuthorizeDeviceScreen and GetTokenFromDevice) After approval in a browser or on a mobile device, the Step 1 `device_code` can be exchanged for an access token.

### Step 3: Poll for User’s Authorization

 While the user completes authorization, the game must poll Discord's Oauth2 token endpoint to trade the device code for a valid access token. Poll at the provided `interval` until the request succeeds or the code expires after `expires_in`; both values come from the Authorize response above. If expiration occurs, start a new authorization request or cancel the flow.

### Step 4: Exchange Device Code for Access Token

 After Discord authentication is complete, exchange the device code for an access token.

```
def exchange_device_code(device_code):
    data = {
        "grant_type": "urn:ietf:params:oauth:grant-type:device_code",
        "device_code": device_code
    }
    headers = {"Content-Type": "application/x-www-form-urlencoded"}

    response = requests.post(f"{API_ENDPOINT}/oauth2/token", data=data, headers=headers, auth=(CLIENT_ID, CLIENT_SECRET))
    response.raise_for_status()
    return response.json()

```

### Step 5: Handle Token Response

 After exchange succeeds, the API returns an access token that can be passed to [`Client::UpdateToken`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a606b32cef7796f7fb91c2497bc31afc4) and followed with [`Client::Connect`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a873a844c7c4c72e9e693419bb3e290aa) to start using Discord Social SDK features.

#### Example Response

```
{
  "access_token": "<access token>",
  "token_type": "Bearer",
  "expires_in": 604800,
  "refresh_token": "<refresh token>",
  "scope": "sdk.social_layer"
}

```

 When using the manual flow, call [`Client::CloseAuthorizeDeviceScreen`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a4fcdd697702d086a8170a2d60a69acb8) to close the authorization screen.

### Example for Manual Token Authorization

 The following Python example demonstrates a manual authorization flow.

```
import requests
import json
import time

API_ENDPOINT = 'https://discord.com/api/v10'
CLIENT_ID = 'YOUR_CLIENT_ID'
CLIENT_SECRET = 'YOUR_CLIENT_SECRET'
SCOPE = 'sdk.social_layer'

def authorize_device():
  data = {
    'scope': SCOPE
  }
  headers = {
    'Content-Type': 'application/x-www-form-urlencoded'
  }
  r = requests.post('%s/oauth2/device/authorize' % API_ENDPOINT, data=data, headers=headers, auth=(CLIENT_ID, CLIENT_SECRET))
  r.raise_for_status()
  return r.json()

def exchange_device_code(device_code):
  print("Attempting to exchange device code for access token...")
  data = {
      "grant_type": "urn:ietf:params:oauth:grant-type:device_code",
      "device_code": device_code
    }
  headers = {"Content-Type": "application/x-www-form-urlencoded"}

  r = requests.post('%s/oauth2/token' % API_ENDPOINT, data=data, headers=headers, auth=(CLIENT_ID, CLIENT_SECRET))
  r.raise_for_status()
  response = r.json()
  return r.json()

def poll_token_exchange(device_code, interval, expires_in):
  start_time = time.time()

  while time.time() - start_time < expires_in:
    try:
      print("Checking for authorization...")
      result = exchange_device_code(device_code)
      return result
    except requests.exceptions.HTTPError as e:
      if e.response.status_code == 400:
        # Authorization is pending, wait and try again
        print(f"Waiting {interval} seconds for user to authorize...")
        time.sleep(interval)
      else:
        # Some other error occurred
        raise e

  raise Exception(f"Authorization timed out after {expires_in} seconds")

# Step 1: Get device authorization details
device_auth = authorize_device()

# Step 2: Display authorization details to user
print("\nPlease visit:", device_auth["verification_uri_complete"])

# Step 3: Poll for token exchange
try:
    access_token = poll_token_exchange(
        device_auth["device_code"],
        device_auth["interval"],
        device_auth["expires_in"]
    )
    print("\nSuccess! Access Token:", access_token["access_token"])
except Exception as e:
    print("\nError:", str(e))

# Step 4: Use the access token with discordpp::Client::UpdateToken and discordpp::Client::Connect

```

---

### Working with Tokens

 After receiving access and refresh tokens for the player, set the SDK access token with [`Client::UpdateToken`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a606b32cef7796f7fb91c2497bc31afc4). The user is then authorized, and the game can call [`Client::Connect`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a873a844c7c4c72e9e693419bb3e290aa).

```
client->UpdateToken(discordpp::AuthorizationTokenType::Bearer, accessToken, [client](discordpp::ClientResult result) {
  client->Connect();
});

```

 Store the player's access and refresh tokens for later sessions. Remember that `access_token` values expire. Use the `refresh_token` to refresh the token; that process is covered in [Refreshing Access Tokens](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord#refreshing-access-tokens).

---

### Next Steps

 After console account linking with Discord is working, you can add more social features to the game.

### Creating a Unified Friends List

Merge Discord friends and game friends into one list for easier management.

### Setting Rich Presence

Show game status and information to Discord friends.

### Designing for Consoles

Use Discord design guidelines when adding Discord features to a console game. Need help? Join the [Discord Developers Server](https://discord.gg/discord-developers) and ask in `#social-sdk-dev-help` for community support. If you find a Social SDK bug, report it at [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report)

---

### Change Log

| Date | Changes |
| --- | --- |
| March 17, 2025 | Initial release |
