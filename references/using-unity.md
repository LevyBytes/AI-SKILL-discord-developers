# Using Unity (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

### Overview

 This guide explains how to add the Discord Social SDK to a Unity project. When finished, the project will be able to:
- Authenticate users with Discord
- Configure logging and monitor SDK status
- Initialize the SDK and connect it
- Read how many Discord friends the player has
- Publish the player's rich presence for your game

### Prerequisites

 Before starting, ensure you have:
- Unity 2021.3 or later
 The sections below go through each step in detail.

---

### Step 1: Create a Discord Developer Team

 Start by creating a developer team in the Discord Developer Portal. Discord uses that team to administer your applications and SDK integrations. If a team already exists for this work, continue with that one.
1. On the [Discord Developer Portal](https://discord.com/developers/teams), create a developer team.
 After the team exists, invite collaborators so they can help with the integration.

---

### Step 2: Create a Discord Application

1. In the [Discord Developer Portal](https://discord.com/developers/applications), create an application and place it under your team.
2. In the [OAuth2 tab](https://discord.com/developers/applications/select/oauth2), add a redirect URL:
  - Desktop apps can use `http://127.0.0.1/callback` (you may change it later).
  - For more advanced redirect URI setup, refer to [`discordpp::Client::Authorize`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ace94a58e27545a933d79db32b387a468).

3. Turn on `Public Client` in the [OAuth2 tab](https://discord.com/developers/applications/select/oauth2).
 This walkthrough turns on **Public Client** so the SDK can be tried quickly. Most shipped games should not remain public clients; review this setting with your team before release. [Learn more about public clients](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes#oauth2-client-types).

---

### Step 3: Enable Discord Social SDK for Your App

 After the Discord application exists, enable Discord Social SDK on that application.
1. Open the [Discord Developer Portal](https://discord.com/developers/applications) and choose the application created in Step 2.
2. In the app sidebar, under `Discord Social SDK`, select [Getting Started](https://discord.com/developers/applications/select/social-sdk/getting-started).
3. Complete the form with information about your game.
4. Select `Submit`; the Social SDK becomes enabled for the application.
5. After enablement, Social SDK binaries are available from [Downloads](https://discord.com/developers/applications/select/social-sdk/downloads).

---

### Step 4: Download the Social SDK for Unity

1. In the sidebar's Discord Social SDK section, open [Downloads](https://discord.com/developers/applications/select/social-sdk/downloads).
2. Choose the newest version in the version menu and download the Unity SDK.
 That page also provides a Unity sample project. This guide does not cover it, but it is useful to explore after completing these steps.

---

### Step 5: Project Setup

 Prepare the Unity project by adding the Social SDK package, then create the objects and scripts needed to call it.
1. In Unity Hub, create a new 2D project using Unity version 2021.3 or newer
2. Either:
  1. Extract the zip into Unity's `Packages` folder, or
  2. Extract the zip elsewhere and use [Install Package from Disk](https://docs.unity3d.com/Manual/upm-ui-local.html). Keep that folder in a stable location, because the Unity project loads the package from there.

3. Add a `Scripts` folder to the project and create `DiscordManager.cs` inside it
4. Put this code in `DiscordManager.cs`:

```
using UnityEngine;
using UnityEngine.UI;
using Discord.Sdk;
using System.Linq;

public class DiscordManager : MonoBehaviour
{
    [SerializeField]
    private ulong clientId; // Set this in the Unity Inspector from the dev portal

    [SerializeField]
    private Button loginButton;

    [SerializeField]
    private Text statusText;

    private Client client;
    private string codeVerifier;
}

```

1. Create an empty scene object named **DiscordManager** with **GameObject > Create Empty**, then attach `DiscordManager.cs` to it
2. Add a scene button with **GameObject > UI > Legacy > Button**
3. Add scene text with **GameObject > UI > Legacy > Text**
4. Move the button and text to visible positions on screen
5. In the inspector for **DiscordManager**, assign the button and text references
6. Run it!
 That is enough for the initial setup. Nothing visible should happen yet; if problems appear, use the troubleshooting notes before continuing.

### Troubleshooting

- Confirm that Unity added the Social SDK package successfully

Mac libdiscord_partner_sdk.dylib Not Opened

On Mac, Apple may show a Not Opened warning for libdiscord_partner_sdk.dylib because the library cannot be verified. Choose **Done**. Open **System Settings > Privacy & Security**, go to **Security**, and find the blocked-library message for libdiscord_partner_sdk.dylib. Choose **Open Anyway**, run again, then choose **Open Anyway** in the next popup so the library loads. (Figure: Error)(Figure: Settings)(Figure: Open)

---

### Step 6: Setting Up SDK Event Handling

 Add event handlers so the Unity project can observe the Discord connection. Two callbacks are important:
- A logging callback that exposes SDK activity
- A status callback that reports when Discord features are ready to use
 Begin by adding this code to `DiscordManager.cs`:

```
void Start()
{
    client = new Client();

    // Modifying LoggingSeverity will show you more or less logging information
    client.AddLogCallback(OnLog, LoggingSeverity.Error);
    client.SetStatusChangedCallback(OnStatusChanged);

    // Make sure the button has a listener
    if (loginButton != null)
    {
        //loginButton.onClick.AddListener(StartOAuthFlow);
    }
    else
    {
        Debug.LogError("Login button reference is missing, connect it in the inspector!");
    }

    // Set initial status text
    if (statusText != null)
    {
        statusText.text = "Ready to login";
    }
    else
    {
        Debug.LogError("Status text reference is missing, connect it in the inspector!");
    }
}

private void OnLog(string message, LoggingSeverity severity)
{
    Debug.Log($"Log: {severity} - {message}");
}

private void OnStatusChanged(Client.Status status, Client.Error error, int errorCode)
{
    Debug.Log($"Status changed: {status}");
    statusText.text = status.ToString();
    if(error != Client.Error.None)
    {
        Debug.LogError($"Error: {error}, code: {errorCode}");
    }
}

```

 This wires API status updates into the in-game text. Copy your Client ID from the developer portal [OAuth2 tab](https://discord.com/developers/applications/select/oauth2) and place it in ClientID on **DiscordManager** in the inspector.

### What These Callbacks Do

- The **logging callback** exposes behind-the-scenes SDK behavior and is valuable for debugging
- The **status callback** reports when the client is connected and Discord features can be used
 The Unity plugin runs SDK callbacks inside Unity automatically, so you do not need [`RunCallbacks`](https://discord.com/developers/docs/social-sdk/namespacediscordpp.html#ab5dd8cf274f581ee1885de5816be3c29) as shown in the [C++ guide](https://docs.discord.com/developers/discord-social-sdk/getting-started/using-c++). Most Discord functionality is unavailable until status becomes `Ready`. The status callback tells you when that point is reached. Authentication with Discord is required before reaching `Ready`, so handle that next.

---

### Step 7: Account Linking with Discord

 This step adds OAuth2 authentication for account linking with Discord. The flow will:
1. Launch Discord or open a browser window for Discord login
2. Get an authorization code
3. Trade that code for an access token
4. Connect to Discord

### Add the Authentication Code

 **Choosing your OAuth2 scopes:** This walkthrough uses `Client.GetDefaultPresenceScopes()` and requests `openid` plus `sdk.social_layer_presence`. These scopes cover account linking, friends list access, and rich presence. Games that need lobbies, voice chat, or direct messaging should use `Client.GetDefaultCommunicationScopes()` instead. The [OAuth2 Scopes guide](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes) has the complete breakdown. Start by adding this code to `DiscordManager.cs`:

```
private void StartOAuthFlow() {
    var authorizationVerifier = client.CreateAuthorizationCodeVerifier();
    codeVerifier = authorizationVerifier.Verifier();

    var args = new AuthorizationArgs();
    args.SetClientId(clientId);
    args.SetScopes(Client.GetDefaultPresenceScopes());
    args.SetCodeChallenge(authorizationVerifier.Challenge());
    client.Authorize(args, OnAuthorizeResult);
}

private void OnAuthorizeResult(ClientResult result, string code, string redirectUri) {
    Debug.Log($"Authorization result: [{result.Error()}] [{code}] [{redirectUri}]");
    if (!result.Successful()) {
        return;
    }
    GetTokenFromCode(code, redirectUri);
}

private void GetTokenFromCode(string code, string redirectUri) {
    client.GetToken(clientId,
                    code,
                    codeVerifier,
                    redirectUri,
                    (result, token, refreshToken, tokenType, expiresIn, scope) => {});
}

```

 After that, uncomment `loginButton.onClick.AddListener(StartOAuthFlow);` in `Start()`

```
if (loginButton != null)
{
    loginButton.onClick.AddListener(StartOAuthFlow);
}

```

### What’s Happening Here?

1. The code creates an OAuth2 PKCE code verifier
2. It prepares authorization arguments with the app ID and requested scopes
3. [`Client::Authorize`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ace94a58e27545a933d79db32b387a468) starts authentication and opens a browser
4. After authorization succeeds, the code is exchanged for an access token
 Never expose access tokens in logs or store them carelessly. Treat them as sensitive credentials.

### Testing It Out

 Press play, then click the button to start OAuth. The browser should open so you can log in and authorize the game. Console logs should appear, but the status will not change yet.

### Troubleshooting

- If the button appears inactive, confirm `loginButton.onClick.AddListener(StartOAuthFlow);` is uncommented
- Verify that `ClientId` is correct
- Confirm the redirect URL exists in the Discord Developer Portal
- Review the console for exact error messages

DiscordManager.cs at this stage

```
using UnityEngine;
using UnityEngine.UI;
using Discord.Sdk;
using System.Linq;

public class DiscordManager : MonoBehaviour
{
    [SerializeField]
    private ulong clientId; // Set this in the Unity Inspector from the dev portal

    [SerializeField]
    private Button loginButton;

    [SerializeField]
    private Text statusText;

    private Client client;
    private string codeVerifier;

        void Start()
    {
        client = new Client();

        // Modifying LoggingSeverity will show you more or less logging information
        client.AddLogCallback(OnLog, LoggingSeverity.Error);
        client.SetStatusChangedCallback(OnStatusChanged);

        // Make sure the button has a listener
        if (loginButton != null)
        {
            loginButton.onClick.AddListener(StartOAuthFlow);
        }
        else
        {
            Debug.LogError("Login button reference is missing, connect it in the inspector!");
        }

        // Set initial status text
        if (statusText != null)
        {
            statusText.text = "Ready to login";
        }
        else
        {
            Debug.LogError("Status text reference is missing, connect it in the inspector!");
        }
    }

    private void OnLog(string message, LoggingSeverity severity)
    {
        Debug.Log($"Log: {severity} - {message}");
    }

    private void OnStatusChanged(Client.Status status, Client.Error error, int errorCode)
    {
        Debug.Log($"Status changed: {status}");
        statusText.text = status.ToString();
        if(error != Client.Error.None)
        {
            Debug.LogError($"Error: {error}, code: {errorCode}");
        }
    }

    private void StartOAuthFlow() {
        var authorizationVerifier = client.CreateAuthorizationCodeVerifier();
        codeVerifier = authorizationVerifier.Verifier();

        var args = new AuthorizationArgs();
        args.SetClientId(clientId);
        args.SetScopes(Client.GetDefaultPresenceScopes());
        args.SetCodeChallenge(authorizationVerifier.Challenge());
        client.Authorize(args, OnAuthorizeResult);
    }

    private void OnAuthorizeResult(ClientResult result, string code, string redirectUri) {
        Debug.Log($"Authorization result: [{result.Error()}] [{code}] [{redirectUri}]");
        if (!result.Successful()) {
            return;
        }
        GetTokenFromCode(code, redirectUri);
    }

    private void GetTokenFromCode(string code, string redirectUri) {
        client.GetToken(clientId,
                        code,
                        codeVerifier,
                        redirectUri,
                        (result, token, refreshToken, tokenType, expiresIn, scope) => {});
    }
}

```

---

### Step 8: Connect the SDK to Discord

 With an access token available, connect the SDK to Discord. Start by adding this code to `DiscordManager.cs`:

```
private void OnReceivedToken(string token) {
    Debug.Log("Token received: " + token);
    client.UpdateToken(AuthorizationTokenType.Bearer, token, (ClientResult result) => { client.Connect(); });
}

private void OnRetrieveTokenFailed() { statusText.text = "Failed to retrieve token"; }

```

 Then change `GetTokenFromCode` so it calls these functions when finished:

```
private void GetTokenFromCode(string code, string redirectUri) {
    client.GetToken(clientId,
                    code,
                    codeVerifier,
                    redirectUri,
                    (result, token, refreshToken, tokenType, expiresIn, scope) => {
                        if (token != "") {
                            OnReceivedToken(token);
                        } else {
                            OnRetrieveTokenFailed();
                        }
                    });
}

```

### What’s Happening Here?

1. [`Client::UpdateToken`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a606b32cef7796f7fb91c2497bc31afc4) gives the SDK the access token to use for Discord API calls
2. In the update-token callback, call [`Client::Connect`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a873a844c7c4c72e9e693419bb3e290aa)
3. The SDK starts connecting asynchronously
4. The status text reports when the SDK reaches readiness

### Testing the Connection

 Run the scene again, then use the button for the login and authorization step. The status text should move through the OAuth flow and finish at `Ready`.

### Troubleshooting

 If status never reaches `Ready`:
- Confirm the access token is valid
- Ensure you have internet connectivity
- Check the console for SDK error output
- Make sure the inspector value for `ClientID` is correct

DiscordManager.cs at this stage

```
using UnityEngine;
using UnityEngine.UI;
using Discord.Sdk;
using System.Linq;

public class DiscordManager : MonoBehaviour
{
    [SerializeField]
    private ulong clientId; // Set this in the Unity Inspector from the dev portal

    [SerializeField]
    private Button loginButton;

    [SerializeField]
    private Text statusText;

    private Client client;
    private string codeVerifier;

        void Start()
    {
        client = new Client();

        // Modifying LoggingSeverity will show you more or less logging information
        client.AddLogCallback(OnLog, LoggingSeverity.Error);
        client.SetStatusChangedCallback(OnStatusChanged);

        // Make sure the button has a listener
        if (loginButton != null)
        {
            loginButton.onClick.AddListener(StartOAuthFlow);
        }
        else
        {
            Debug.LogError("Login button reference is missing, connect it in the inspector!");
        }

        // Set initial status text
        if (statusText != null)
        {
            statusText.text = "Ready to login";
        }
        else
        {
            Debug.LogError("Status text reference is missing, connect it in the inspector!");
        }
    }

    private void OnLog(string message, LoggingSeverity severity)
    {
        Debug.Log($"Log: {severity} - {message}");
    }

    private void OnStatusChanged(Client.Status status, Client.Error error, int errorCode)
    {
        Debug.Log($"Status changed: {status}");
        statusText.text = status.ToString();
        if(error != Client.Error.None)
        {
            Debug.LogError($"Error: {error}, code: {errorCode}");
        }
    }

    private void StartOAuthFlow() {
        var authorizationVerifier = client.CreateAuthorizationCodeVerifier();
        codeVerifier = authorizationVerifier.Verifier();

        var args = new AuthorizationArgs();
        args.SetClientId(clientId);
        args.SetScopes(Client.GetDefaultPresenceScopes());
        args.SetCodeChallenge(authorizationVerifier.Challenge());
        client.Authorize(args, OnAuthorizeResult);
    }

    private void OnAuthorizeResult(ClientResult result, string code, string redirectUri) {
        Debug.Log($"Authorization result: [{result.Error()}] [{code}] [{redirectUri}]");
        if (!result.Successful()) {
            return;
        }
        GetTokenFromCode(code, redirectUri);
    }

    private void GetTokenFromCode(string code, string redirectUri) {
        client.GetToken(clientId,
                        code,
                        codeVerifier,
                        redirectUri,
                        (result, token, refreshToken, tokenType, expiresIn, scope) => {
                            if (token != "") {
                                OnReceivedToken(token);
                            } else {
                                OnRetrieveTokenFailed();
                            }
                        });
    }

    private void OnReceivedToken(string token) {
        Debug.Log("Token received: " + token);
        client.UpdateToken(AuthorizationTokenType.Bearer, token, (ClientResult result) => { client.Connect(); });
    }

    private void OnRetrieveTokenFailed() { statusText.text = "Failed to retrieve token"; }
}

```

 With the client ready, you can begin adding Discord social features.

---

### Step 9: Access Discord Relationships

 Read the user's Discord relationships, also called the friends list, and print the count. This shows how to retrieve Discord data for use in the game. Add this code to `DiscordManager.cs`:

```
private void ClientReady()
{
    Debug.Log($"Friend Count: {client.GetRelationships().Count()}");
}

```

 Call it once the client is usable, which you can detect inside `OnStatusChanged`:

```
private void OnStatusChanged(Client.Status status, Client.Error error, int errorCode)
{
    Debug.Log($"Status changed: {status}");
    statusText.text = status.ToString();
    if(error != Client.Error.None)
    {
        Debug.LogError($"Error: {error}, code: {errorCode}");
    }

    if (status == Client.Status.Ready)
    {
        ClientReady();
    }
}

```

### What This Code Does

 When client status is `Ready,`, `ClientReady` runs and calls [`Client::GetRelationships`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ad481849835cd570f0e03adafcf90125d). That returns the player's friends list, and the sample logs the friend count to the console.

### Testing It Out

 Press play and click the button. After OAuth completes and status becomes `Ready`, check the console for the output.

### Example Output

```
Friend Count: 42

```

### Troubleshooting

- Confirm the OAuth2 scopes include relationship access
- Make sure the client is connected, with status `Ready`
- Confirm the Discord account has friends
- Inspect the logging callback for errors
 Next, add Rich Presence so Discord can display your game's activity.

DiscordManager.cs at this stage
