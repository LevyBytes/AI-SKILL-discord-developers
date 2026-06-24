# Using Unity (part 2)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

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

        if (status == Client.Status.Ready)
        {
            ClientReady();
        }
    }

    private void ClientReady()
    {
        Debug.Log($"Friend Count: {client.GetRelationships().Count()}");
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

---

### Step 10: Set Rich Presence

 Add Rich Presence so Discord can display what the player is doing in your game to friends. Update `ClientReady` with this code:

```
private void ClientReady()
{
    Debug.Log($"Friend Count: {client.GetRelationships().Count()}");

    Activity activity = new Activity();
    activity.SetType(ActivityTypes.Playing);
    activity.SetState("In Competitive Match");
    activity.SetDetails("Rank: Diamond II");
    client.UpdateRichPresence(activity, (ClientResult result) => {
        if (result.Successful()) {
            Debug.Log("Rich presence updated!");
        } else {
            Debug.LogError("Failed to update rich presence");
        }
    });
}

```

### What This Code Does

1. Builds an [`Activity`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Activity.html#ae793d9adbe16fef402b859ba02bee682) object that describes the player's current game state
2. Sets basic information like:

- The activity type (Playing)
- Current state (“In Competitive Match”)
- Additional details (“Rank: Diamond II”)

1. Sends the rich presence update to Discord

### Testing It Out

 Run the scene and click the button. After OAuth completes, the status should change to `Ready`; the console reports whether rich presence succeeded, and the result can be confirmed on your Discord profile.

### Troubleshooting

 If presence does not appear:
- Confirm the client is connected and status is `Ready`
- Read the console for error messages
- Confirm the activity settings are valid
- Confirm your Discord status is not invisible

---

The full DiscordManager.cs

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

        if (status == Client.Status.Ready)
        {
            ClientReady();
        }
    }

    private void ClientReady()
    {
            Debug.Log($"Friend Count: {client.GetRelationships().Count()}");

            Activity activity = new Activity();
            activity.SetType(ActivityTypes.Playing);
            activity.SetState("In Competitive Match");
            activity.SetDetails("Rank: Diamond II");
            client.UpdateRichPresence(activity, (ClientResult result) => {
                if (result.Successful()) {
                    Debug.Log("Rich presence updated!");
                } else {
                    Debug.LogError("Failed to update rich presence");
                }
            });
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

---

### Conclusion

 The Discord Social SDK is now integrated into Unity. Review the completed work:

### What You’ve Built

- Set up a Discord application with OAuth2
- Added SDK logging and status monitoring
- ✅ Implemented user authentication flow
- ✅ Retrieved Discord relationships data
- ✅ Added Rich Presence support

### Key Concepts Learned

- Initializing and configuring the Discord SDK
- Managing authentication and connections
- Using Discord social features
- Handling asynchronous callbacks
- Monitoring SDK status and events

---

### Next Steps

### Social SDK Unity sample

 The [in depth sample for using the Social SDK in Unity](https://github.com/discord/social-sdk-unity-sample) shows a best-practice Unity integration. It includes prefabs with code and UI that can be dropped into a game to add Discord social features quickly. (Figure: Video showing off the Unity sample with the Discord Social SDK)

### Creating a Unified Friends List

Build one friends list that merges Discord relationships with game-specific friendships

### Setting Rich Presence

Extend your game's rich presence with richer details and game invite support

### Managing Game Invites

Let players invite friends into a game session or party. For help, join the [Discord Developers Server](https://discord.gg/discord-developers) and ask in `#social-sdk-dev-help`. Report Social SDK bugs at [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report).

---

### Change Log

| Date | Changes |
| --- | --- |
| March 17, 2025 | Initial release |
