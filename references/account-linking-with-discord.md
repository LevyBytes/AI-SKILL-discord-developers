# Account Linking With Discord

> Discord Developers reference. Original prose; identifiers preserved verbatim.

### Overview

 If account linking is new to you, start with the [Account Linking overview](https://docs.discord.com/developers/platform/account-linking) for the core model, behavior, and flow choices. This guide shows how to authenticate users through their existing Discord accounts with OAuth2 so they can sign in cleanly and use Discord features.

### Flexible Account Options

 Players without Discord accounts can still use game features if you create a [provisional account](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts) through the SDK. The recommended sequence is to create the provisional account first and link Discord later; see Recommended Integration Path below.

### Prerequisites

 Before starting, confirm you have the following:
- Read the [Core Concepts](https://docs.discord.com/developers/discord-social-sdk/core-concepts) guide so these topics are familiar:
  - OAuth2 authentication flow
  - Discord application setup
  - SDK initialization

- Prepared your development environment with:
  - A Discord application in the [Developer Portal](https://discord.com/developers/applications)
  - The Discord Social SDK downloaded and configured
  - A working basic SDK integration, including initialization and connection

 If these prerequisites are not complete, follow the [Getting Started](https://docs.discord.com/developers/discord-social-sdk/getting-started) guide first. This feature needs the **Default Presence Scopes** (`openid` and `sdk.social_layer_presence`). Configure your OAuth2 flow with [`Client::GetDefaultPresenceScopes`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a7648bd1d2f7d9a86ebd0edb8bef12b5c), and use the [OAuth2 Scopes guide](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes) for the full scope list.

---

### Our Authentication Flow

 OAuth2 is the standard way to let users sign in with Discord. The flow uses these steps:
1. [Request authorization](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord#step-1-request-authorization): Your game asks Discord to authenticate the user.
2. [User Approval](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord#step-2-user-approval): The user grants your application the requested access.
3. [Receive Authorization Code](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord#step-3-receiving-the-authorization-code): Discord sends the user back to your app with an authorization code after approval.
4. [Exchange for Tokens](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord#step-4-exchanging-the-authorization-code-for-an-access-token): Trade the authorization code for:
  - Access Token, valid for ~7 days
  - Refresh Token, used to request a new access token

 The OAuth2 flow requires the user's account to be verified

### OAuth2 using the Discord Social SDK

- If the Discord client supports the [overlay](https://support.discord.com/hc/en-us/articles/217659737-Game-Overlay-101) on Windows, the OAuth2 login modal appears inside the game instead of opening a browser.
- The SDK handles redirects automatically, which reduces the authentication plumbing you need to write.
- Built-in protections include measures such as CSRF defense, but your app should still follow security best practices.

---

### Requesting Access Tokens

### Step 0: Configure OAuth2 Redirects

 OAuth2 works only when your app has the correct redirect URIs registered in the [**Discord Developer Portal**](https://discord.com/developers/applications/select/oauth2).
| Platform | Redirect URI |
| --- | --- |
| **Desktop** | `http://127.0.0.1/callback` |
| **Mobile** | `discord-APP_ID:/authorize/callback` *(substitute your Discord application ID for `APP_ID`)* |

### Step 1: Request Authorization

 The SDK includes helpers for OAuth2 login. Call [`Client::Authorize`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ace94a58e27545a933d79db32b387a468) to begin authorization and let the user approve access.

#### Authorization Scopes

 Scopes are a required argument to [`Client::Authorize`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ace94a58e27545a933d79db32b387a468); they define the permissions your game requests from the user. The Discord Social SDK provides two helper methods for the common cases:
| Helper Method | Scopes Requested | Features Enabled |
| --- | --- | --- |
| `Client::GetDefaultPresenceScopes` | `openid sdk.social_layer_presence` | Account linking, friends list, rich presence |
| `Client::GetDefaultCommunicationScopes` | `openid sdk.social_layer` | Everything above, plus lobbies, voice chat, direct messaging, and linked channels |

 Use [`Client::GetDefaultPresenceScopes`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a7648bd1d2f7d9a86ebd0edb8bef12b5c) as the starting point unless communication features are required. You can request additional scopes later as the integration grows. The [OAuth2 Scopes guide](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes) has the complete details.

#### Authorization Code Verifier

 If [Step 4](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord#step-4-exchanging-the-authorization-code-for-an-access-token) uses [`Client::GetToken`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#aaee636f91fb1ea3465157c20313b702c), your requests need a code challenge and a code verifier. The SDK can create both values for you with [`Client::CreateAuthorizationCodeVerifier`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#aba6adc1f978e7bf4c5433c560e1ad704), so you do not need to implement that cryptographic setup yourself.

```
// Create a code verifier and challenge if using GetToken
auto codeVerifier = client->CreateAuthorizationCodeVerifier();
discordpp::AuthorizationArgs args{};
args.SetClientId(YOUR_DISCORD_APPLICATION_ID);
args.SetScopes(discordpp::Client::GetDefaultPresenceScopes());
args.SetCodeChallenge(codeVerifier.Challenge());

client->Authorize(args, [client, codeVerifier](discordpp::ClientResult result, std::string code, std::string redirectUri) {
  if (!result.Successful()) {
    std::cerr << "❌ Authorization Error: " << result.Error() << std::endl;
  } else {
    std::cout << "✅ Authorization successful! Next step: exchange code for an access token \n";
  }
});

```

### Step 2: User Approval

 After [`Client::Authorize`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ace94a58e27545a933d79db32b387a468) is called, the SDK shows the approval prompt in a browser window, the Discord client, or an in-game overlay.

### Step 3: Receiving the Authorization Code

 After the user approves the Step 2 request, Discord redirects back to your app with an authorization code that can be exchanged for an access token.

### Step 4: Exchanging the Authorization Code for an Access Token

#### Server-to-Server Get Token Exchange

 If your application has a backend server and `Public Client` is not enabled, exchange the authorization code for an access token yourself through the Discord API.

```
import requests

API_ENDPOINT = 'https://discord.com/api/v10'
CLIENT_ID = 'YOUR_CLIENT_ID'
CLIENT_SECRET = 'YOUR_CLIENT_SECRET'

def exchange_code(code, redirect_uri, code_verifier):
    data = {
        'grant_type': 'authorization_code',
        'code': code,
        'redirect_uri': redirect_uri,
        'code_verifier': code_verifier
    }
    headers = {'Content-Type': 'application/x-www-form-urlencoded'}
    r = requests.post(f'{API_ENDPOINT}/oauth2/token', data=data, headers=headers, auth=(CLIENT_ID, CLIENT_SECRET))
    r.raise_for_status()
    return r.json()

```

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

#### Token Exchange for Public Clients

 This path requires **Public Client** to be enabled for your app, which most games should avoid shipping with. [Learn more](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes#oauth2-client-types) If there is no backend server, enable `Public Client` in the Discord Developer Portal and use [`Client::GetToken`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#aaee636f91fb1ea3465157c20313b702c) to exchange the authorization code automatically. Provide the code verifier that produced the Step 1 code challenge.

```
client->GetToken(YOUR_DISCORD_APPLICATION_ID, code, codeVerifier.Verifier(), redirectUri,
  [client](discordpp::ClientResult result,
    std::string accessToken,
    std::string refreshToken,
    discordpp::AuthorizationTokenType tokenType,
    int32_t expiresIn,
    std::string scope) {
    std::cout << "🔓 Access token received! Establishing connection...\n";
    // Next step: Update the token in the client and connect to Discord
  });

```

---

### Working with Tokens

 After you receive an access token, pass it into the SDK with [`Client::UpdateToken`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a606b32cef7796f7fb91c2497bc31afc4). The user is authorized at that point. Store both the player's access token and refresh tokens, because `access_token` values expire and the `refresh_token` is needed to obtain a fresh access token.

```
client->UpdateToken(discordpp::AuthorizationTokenType::Bearer, ACCESS_TOKEN_VALUE, [client](discordpp::ClientResult result) {
  client->Connect();
);

```

---

### Refreshing Access Tokens

 Access tokens expire after 7 days, so use refresh tokens to obtain replacements.

### Server-to-Server Token Refresh

 When authentication is handled by your server, refresh the token with an API request.

```
import requests

API_ENDPOINT = 'https://discord.com/api/v10'
CLIENT_ID = 'YOUR_CLIENT_ID'
CLIENT_SECRET = 'YOUR_CLIENT_SECRET'

def refresh_token(refresh_token):
    data = {
        'grant_type': 'refresh_token',
        'refresh_token': refresh_token
    }
    headers = {'Content-Type': 'application/x-www-form-urlencoded'}
    r = requests.post(f'{API_ENDPOINT}/oauth2/token', data=data, headers=headers, auth=(CLIENT_ID, CLIENT_SECRET))
    r.raise_for_status()
    return r.json()

```

### Refreshing Access Tokens for Public Clients

 This path requires **Public Client** to be enabled for your app, which most games should avoid shipping with. [Learn more](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes#oauth2-client-types) For public clients, the simplest refresh path is the SDK method [`Client::RefreshToken`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a187af0f99f94b3b9a4ad4302f6a443e7).

```
client->RefreshToken(
      YOUR_DISCORD_APPLICATION_ID, GetRefreshToken(),
      [client](discordpp::ClientResult result, std::string accessToken,
               std::string refreshToken,
               discordpp::AuthorizationTokenType tokenType, int32_t expiresIn,
               std::string scope) {
        if (!result.Successful()) {
          std::cout << "❌ Error refreshing token: " << result.Error()
                    << std::endl;
          return;
        }

        // Update token and connect
        UpdateToken(client, refreshToken, accessToken);
      });

```

### When Refresh Fails

 A saved refresh token can become unusable between sessions for several reasons:
- The user removed your application from *User Settings -> Authorized Apps*.
- Your app called the unmerge endpoint or the token revocation endpoint.
- The [user's Discord account was banned](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#ban-driven-unmerge).
- The access and refresh tokens expired or became invalid for a reason unrelated to revocation, unmerge, or ban.
 All of those situations produce the same `/oauth2/token` response when `grant_type=refresh_token` is used:
- **HTTP status:** `400`
- **Body:** `{ "error": "invalid_grant", "error_description": "Invalid \"refresh_token\" in request" }`
 This endpoint has no error code that identifies a ban or unlink. `invalid_grant` only means the refresh token is no longer valid on the server; it does not reveal which cause applies.

#### Recovering from `invalid_grant`

 Recovery depends on whether the user still has a linked Discord account. Because `invalid_grant` merges several causes into one error, recover by attempting the provisional account fallback first and then handling error `530010`:
1. Delete the user's stored Discord tokens and start the [provisional account flow](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#implementing-provisional-accounts).
2. **If the provisional flow succeeds**, the linked account had already been reverted to a provisional account by revocation, unmerge, or ban. The player can remain in the game even though the Discord link is gone.
3. **If the provisional flow returns error code `530010`** - `User account is non-provisional and should be authed through OAuth2` - the tokens expired or became invalid while the user **still has a linked Discord account**. The token-exchange endpoint intentionally rejects the provisional path for linked users. Ask the player to complete OAuth2 authorization again so new tokens can be issued.
 To receive proactive notice when a user's authorization is revoked by unlink, token revocation, or Discord account ban, integrate the [`APPLICATION_DEAUTHORIZED`](https://docs.discord.com/developers/events/webhook-events#application-deauthorized) webhook event. That webhook lets you react immediately instead of waiting for the next refresh failure.

---

### Revoking Access Tokens

 A user's app authorization may be revoked by your backend, by the user in Discord's UI, or by Discord when an account is banned. No matter which path causes the revocation:
- The user's access and refresh tokens become invalid immediately. Later refresh attempts on `/oauth2/token` return `HTTP 400` with `error: "invalid_grant"`; see When Refresh Fails.
- Discord sends an [`APPLICATION_DEAUTHORIZED`](https://docs.discord.com/developers/events/webhook-events#application-deauthorized) webhook to your app.
 In apps that use the Discord Social SDK, each revocation route in this section acts as an [unmerge](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#unmerging-provisional-accounts): the merged Discord account becomes a new provisional account that retains the original `external_auth_token`. This is true for `/oauth2/token/revoke`, `Client::RevokeToken`, user removal through Discord's UI, and bans. Only [ban-driven unmerges](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#ban-driven-unmerge) create a *restricted* provisional account; the other paths create an unrestricted provisional account that can be merged again. Your own code will know when it initiates revocation. For revocations outside your code, covered in Out-of-Band Revocation below, the webhook is the only proactive signal. Revoking any valid access or refresh token immediately invalidates every access and refresh token your application has for that user.

### Server-to-Server Token Revocation

 Applications with backend servers can revoke tokens by sending an API request to Discord's token revocation endpoint.

```
import requests

API_ENDPOINT = 'https://discord.com/api/v10'
CLIENT_ID = 'YOUR_CLIENT_ID'
CLIENT_SECRET = 'YOUR_CLIENT_SECRET'

def revoke_token(access_or_refresh_token):
    data = {'token': access_or_refresh_token}
    headers = {'Content-Type': 'application/x-www-form-urlencoded'}
    r = requests.post(f'{API_ENDPOINT}/oauth2/token/revoke', data=data, headers=headers, auth=(CLIENT_ID, CLIENT_SECRET))
    r.raise_for_status()

```

### Revoking Access Tokens for Public Clients

 This path requires **Public Client** to be enabled for your app, which most games should avoid shipping with. [Learn more](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes#oauth2-client-types) For public clients, the simplest revocation path is the SDK method `Client::RevokeToken`. It invalidates all access and refresh tokens for the user so they cannot be used again.

```
client->RevokeToken(YOUR_DISCORD_APPLICATION_ID,
                    accessToken, // Can also use refresh token
                    [](const discordpp::ClientResult &result) {
                      if (!result.Successful()) {
                        std::cout
                            << "? Error revoking token: " << result.Error()
                            << std::endl;
                        return;
                      }

                      std::cout
                          << "? Token successfully revoked! User logged out."
                          << std::endl;
                      // Handle successful logout (clear stored tokens,
                      // redirect to login, etc.)
                    });

```

### Out-of-Band Revocation

 Two situations remove the user's Discord link without your code calling an unmerge or revoke endpoint:
- The user removes your app from *User Settings -> Authorized Apps* in Discord, which is a [user-initiated unmerge](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#out-of-band-unmerge).
- The user's Discord account is banned, causing a [ban-driven unmerge](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#ban-driven-unmerge) and an added cross-platform-restricted state on the new provisional account.
 Subscribe to [`APPLICATION_DEAUTHORIZED`](https://docs.discord.com/developers/events/webhook-events#application-deauthorized) so your app is notified when either event occurs. If the webhook is missed, the next token operation fails, and your game should retry through the [provisional account flow](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#implementing-provisional-accounts).

---

### Recommended Integration Path

 The recommended Discord Social SDK integration starts with your game's primary authentication system, not Discord. That primary identity creates a provisional account first; the player can link Discord to it later. This protects game access and data if the player's Discord account has a problem, including a temporary or permanent ban. To follow this path:
1. Create the user through a [non-Discord authentication provider](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#configuring-your-identity-provider), then create a provisional account attached to that identity.
2. When the user later authenticates with Discord to link accounts, have your game backend perform the [merge their provisional account with their Discord Account](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#merging-provisional-accounts).
3. During account merging, Discord stores the provisional account's `externalAuthToken` against the Discord account. If the Discord account is banned, that `externalAuthToken` is attached to the replacement provisional account along with the original Discord account's in-game friends, and remains usable through the authentication provider that created the account.
4. Your backend should keep its own record of the user's `externalAuthToken` after the merge, because it is required to [authenticate via a provisional account](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#implementing-provisional-accounts) if Discord authentication fails because of a ban or another issue.
  Using Discord as the primary or only authentication method can cause players to permanently lose access to in-game data if their Discord account is banned, because there is no migration path to a provisional account tied to an external authentication provider.

---

### Next Steps

 After account linking with Discord is working, you can add more social features to your game.

### Design: Signing In

Design guidance for account linking and user authentication

### Creating a Unified Friends List

Merge Discord friends and in-game friends into one manageable list.

### Setting Rich Presence

Show game status and details to Discord friends. Need help? Join the [Discord Developers Server](https://discord.gg/discord-developers) and ask questions in `#social-sdk-dev-help` for community support. To report a Social SDK bug, use [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report)

---

### Change Log

| Date | Changes |
| --- | --- |
| June 9, 2026 | Clarify `invalid_grant` recovery and the `530010` error |
| May 22, 2026 | Add the out-of-band revocation behavior and the recommended integration path |
| March 17, 2025 | initial release |
