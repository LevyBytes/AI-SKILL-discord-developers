# Using Provisional Accounts (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

### Overview

 Provisional accounts let players use Social SDK features inside a game without linking a Discord account, giving every player a consistent gameplay experience. With provisional accounts, players can:
- Add friends and talk with other players
- Join lobby voice chats in the game
- Send direct messages to other players
- Show up in friends lists and game lobbies
 The same experience works whether players already have Discord accounts or not. This guide covers how to:
1. Configure provisional accounts for your game
2. Create and manage provisional accounts
3. Manage account merging when users upgrade to full Discord

### Prerequisites

 Before starting, confirm you have:
- Familiarity with SDK behavior from the [Getting Started Guide](https://docs.discord.com/developers/discord-social-sdk/getting-started)
- An external authentication provider configured for your game

---

### What Are Provisional Accounts?

 Provisional accounts are temporary Discord accounts that:
- Work only with your game
- Can later be upgraded into full Discord accounts
- Persist between game sessions
- Rely on your game's authentication system
 Provisional accounts let players use Discord chat and voice features and interact with game friends without creating a full Discord account. They function as placeholder Discord accounts owned and managed by your game. When an existing Discord user adds a provisional account as a game friend, that provisional account appears in the user's friend list, enabling direct messages plus lobby text and voice interactions.

### How It Works

1. Your game authenticates players through its existing system (Steam, Epic, etc.)
2. Discord creates temporary accounts tied to those identities
3. Players begin using Discord features right away
4. Players may upgrade to full Discord accounts later
5. Friends and chat history carry over during upgrade

### Benefits

- Instant Access: Social features are available immediately
- Seamless Experience: Player experience stays consistent
- Easy Upgrade Path: Full Discord account conversion is straightforward
- Data Persistence: Friends and history remain available
- Cross-Platform: Supported platforms all work

---

### Getting Set Up

### Choosing an Authentication Method

 Discord provides several authentication methods; choose based on how your game and account system are built:
1. Use the [Bot Token Endpoint](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#server-authentication-with-bot-token-endpoint) when your game account system uniquely identifies users. This is the recommended approach when available.
2. Use [Server Authentication with External Credentials Exchange](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#server-authentication-with-external-credentials-exchange) when you have an OIDC provider or no account system.
3. Use the [Client Side Token Exchange Method](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#authentication-for-public-clients) when using a Public Client.
 When using (2) or (3), configure your identity provider before creating provisional accounts.

### Configuring Your Identity Provider

 The bot token endpoint does not require Identity Provider configuration. Open the Discord app for your game in the [Developer Portal](https://discord.com/developers/applications). In the sidebar's `Discord Social SDK` section, open [External Auth](https://discord.com/developers/applications/select/social-sdk/external-auth-providers). Select `Add Auth Provider`, choose the provider type such as Steam or OIDC, and enter the required provider details. Supported provider types are:
- OpenID Connect (OIDC)
- Steam Session Tickets
- Epic Online Services (EOS)
- Unity
- Apple
- PlayStation Network (PSN)
 For OIDC configuration, OIDC Integration Requirements lists the issuer URL, discovery document, and ID token requirements. Discord represents providers with these types:

#### External Auth Types

| Type | Description |
| --- | --- |
| OIDC | OpenID Connect ID token |
| STEAM_SESSION_TICKET | Steam auth ticket for web, generated with discord as the identity. |
| EPIC_ONLINE_SERVICES_ACCESS_TOKEN | Epic Online Services access token; supports EOS Auth access tokens. |
| EPIC_ONLINE_SERVICES_ID_TOKEN | Epic Online Services ID token; supports EOS Auth + Connect ID tokens. |
| UNITY_SERVICES_ID_TOKEN | Unity Services authentication ID token |
| APPLE_ID_TOKEN | Apple sign-in ID token. |
| PLAYSTATION_NETWORK_ID_TOKEN | PlayStation Network account authentication ID token |
| DISCORD_BOT_ISSUED_ACCESS_TOKEN | Access token for a user authenticated with the Bot Token Endpoint. |

---

### Implementing Provisional Accounts

 Creating a Provisional Account and requesting its Access Token are one operation. You provide external authentication that uniquely identifies the user, and Discord locates the user linked to that identifier.
- If the identity has no associated account, Discord creates a provisional account and returns a new access token for that user.
- If the identity already has a provisional account, Discord returns an access token.
- If the identity already maps to a *full Discord account*, the request is aborted; see [Error Handling](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#error-handling).
 After authentication succeeds, use the access token the same way you would use a full Discord user's access token.

### Server Authentication with Bot Token Endpoint

 This preferred authentication method is usually the simplest option for provisional account integrations.

```
# filepath: your_game/server/auth.py
import requests
from models import GameAccount

def get_provisional_token(game_account: GameAccount):
  response = requests.post(
    'https://discord.com/api/v10/partner-sdk/token/bot',
    headers={
      'Content-Type': 'application/json',
      'Authorization': 'Bot <BOT_TOKEN>' # your application's bot token
    },
    json={
      'external_user_id': game_account.id,       # your account system's unique id
      'preferred_global_name': game_account.display_name, # your account system's display name for the user
    }
  )
  return response.json()

```

#### Bot Token Endpoint Response

```
{
  "access_token": "<access token>",
  "id_token": "<id token>",
  "token_type": "Bearer",
  "expires_in": 604800,
  "scope": "sdk.social_layer"
}

```

### Server Authentication with External Credentials Exchange

```
# filepath: your_game/server/auth.py
import requests

def get_provisional_token(external_token: str):
  response = requests.post(
    'https://discord.com/api/v10/partner-sdk/token',
    json={
      'client_id': CLIENT_ID,
      'client_secret': CLIENT_SECRET,
      'external_auth_type': EXTERNAL_AUTH_TYPE,  # See External Auth Types above
      'external_auth_token': external_token
    }
  )
  return response.json()

```

#### External Credentials Exchange Response

```
{
  "access_token": "<access token>",
  "id_token": "<id token>",
  "token_type": "Bearer",
  "expires_in": 604800,
  "scope": "sdk.social_layer"
}

```

 OIDC responses may include `refresh_token`. The OAuth2 `refresh_token` grant is **deprecated** for it; re-authenticate with a fresh provider token instead. See Refreshing Provisional Account Tokens.

### Authentication for Public Clients

 This method requires **Public Client** to be enabled for the app. Most games should not ship with this enabled. [Learn more](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes#oauth2-client-types) If `Public Client` is enabled on the Discord app, the following code can authenticate players with the external provider.

```
// filepath: your_game/auth_manager.cpp
void AuthenticateUser(std::shared_ptr<discordpp::Client> client) {
    // Get your external auth token (Steam, OIDC, etc.)
    std::string externalToken = GetExternalAuthToken();

    // Get provisional token from Discord
    client->GetProvisionalToken(DISCORD_APPLICATION_ID,
        discordpp::AuthenticationExternalAuthType::OIDC,
        externalToken,
        [client](discordpp::ClientResult result, std::string accessToken, std::string refreshToken, discordpp::AuthorizationTokenType tokenType, int32_t expiresIn, std::string scope) {
        if (result.Successful()) {
            std::cout << "🔓 Provisional token received! Establishing connection...\n";
            client->UpdateToken(discordpp::AuthorizationTokenType::Bearer, accessToken, [client](discordpp::ClientResult result) {
                client->Connect();
            });
        } else {
            std::cerr << "❌ Provisional token request failed: " << result.Error() << std::endl;
        }
    });
}

```

### OIDC Integration Requirements

 When OpenID Connect (OIDC) is the identity provider, Discord validates configuration and tokens against these requirements. They apply when saving OIDC configuration in the [Developer Portal](https://discord.com/developers/applications/select/social-sdk/external-auth-providers) and during runtime token exchange.

#### Issuer URL Requirements

 The issuer URL configured in the Developer Portal must satisfy every requirement below:
| Requirement | Details |
| --- | --- |
| HTTPS scheme | Use `https://` because HTTP is not allowed. |
| No query parameters | Do not include a `?` character in the URL. |
| No fragment | Do not include a `#` character in the URL. |
| No embedded credentials | Do not include a username or password in the URL. |
| Public hostname | The hostname needs at least two segments, such as `example.com`; bare hostnames like `localhost` are not allowed. |
| No private or reserved TLDs | The hostname cannot end with `.local`, `.arpa`, `.internal`, or `.localhost`. |
| No IP addresses | The hostname cannot be a bare IP address, such as `192.168.1.1`. |

 Non-standard ports such as `:8080` are allowed.

#### OIDC Discovery Document Requirements

 Discord loads OIDC configuration from `{issuer_url}/.well-known/openid-configuration` according to [RFC 8414](https://www.rfc-editor.org/rfc/rfc8414). This endpoint must:
- Be accessible over HTTPS
- **Not require HTTP redirects**; Discord does not follow redirects for this document or the JWKS endpoint
- Return a valid JSON object rather than an array
 The discovery document must contain these fields:
| Field | Type | Requirement |
| --- | --- | --- |
| `issuer` | HTTPS URL | Must exactly equal the issuer URL used to request the document. |
| `jwks_uri` | HTTPS URL | URI for the JWKS signing key endpoint. |
| `id_token_signing_alg_values_supported` | Array of strings | Must list at least one supported algorithm. |

 `authorization_endpoint` and `token_endpoint` may be present, but Discord does not use them.

#### Supported Signing Algorithms

 ID tokens must be signed with an asymmetric algorithm. Discord supports:
| Algorithm family | Algorithms |
| --- | --- |
| RSA | `RS256`, `RS384`, `RS512` |
| ECDSA | `ES256`, `ES384`, `ES512` |
| RSA-PSS | `PS256`, `PS384`, `PS512` |

 Symmetric (HMAC) algorithms such as `HS256` are unsupported. Unsupported entries in `id_token_signing_alg_values_supported` are ignored.

#### ID Token Requirements

 The OIDC ID token supplied as `external_auth_token` must meet every requirement below:
| Requirement | Details |
| --- | --- |
| `kid` header | The JWT header needs a `kid` (Key ID) field matching a key in the JWKS. |
| Signing algorithm | Use one of the supported algorithms from the discovery document. |
| `iss` claim | Match the configured issuer URL exactly. |
| `sub` claim | Required; the unique user identifier from the identity provider. |
| `aud` claim | Include the client ID configured for this application in the Developer Portal. |
| `exp` claim | Required; the token must not be expired. |
| `iat` claim | Required; the token must have been issued within the past **7 days**. |

 If present, the optional `preferred_username` claim sets the provisional account display name and must be 1-32 characters.

### Provisional Account Access Tokens

 These methods generate a Discord access token. You submit the user's identity, and Discord creates a new Discord account tied to it. The identity can come from Steam/Epic services or your own identity system. Each method returns an access token that expires in 7 days.

#### Refreshing Provisional Account Tokens

 Use [`Client::SetTokenExpirationCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#aab5bfc76809ea22e79f2f7a067ac4519) to receive a callback when the current token is near expiration or expired, letting the game refresh without disrupting the user. When the token expires, call the original token acquisition method again, then pass the new token to [`Client::UpdateToken`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a606b32cef7796f7fb91c2497bc31afc4). The SDK still receives updates such as lobby messages, and voice calls stay active, after token expiration. New actions, such as sending messages or adding friends, fail until a fresh token is passed to [`Client::UpdateToken`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a606b32cef7796f7fb91c2497bc31afc4).

```
// Register a callback to handle token expiration
client->SetTokenExpirationCallback([client](discordpp::AuthorizationTokenType tokenType) {
    // Re-acquire a new token using the same method you used originally.
    // For example, if you used GetProvisionalToken:
    std::string externalToken = GetExternalAuthToken(); // get a fresh token from your identity provider
    client->GetProvisionalToken(DISCORD_APPLICATION_ID,
        discordpp::AuthenticationExternalAuthType::OIDC,
        externalToken,
        [client](discordpp::ClientResult result, std::string accessToken, std::string refreshToken,
                 discordpp::AuthorizationTokenType tokenType, int32_t expiresIn, std::string scope) {
            if (result.Successful()) {
                // Pass the new access token to UpdateToken — no reconnect needed
                client->UpdateToken(discordpp::AuthorizationTokenType::Bearer, accessToken, [](discordpp::ClientResult result) {
                    if (result.Successful()) {
                        std::cout << "✅ Token refreshed successfully\n";
                    }
                });
            } else {
                std::cerr << "❌ Failed to refresh provisional token: " << result.Error() << std::endl;
            }
        });
});

```

 For Server Authentication with OIDC, the response returns `refresh_token`, but the OAuth2 `refresh_token` grant is **deprecated**. Re-authenticate with a fresh provider token.

#### Provisional Account Access Token Storage

 Prefer not to store provisional tokens; call this function when the game launches and when tokens approach expiration. If storing them, keep provisional account tokens clearly separate from full Discord account tokens.

### Error Handling

 Common server token exchange error codes and fixes:
| Code | Meaning | Solution |
| --- | --- | --- |
| 530000 | Application not configured | Ask Discord support to enable provisional accounts for the application. |
| 530001 | Expired ID token | Request a fresh token from the identity provider. |
| 530004 | Token too old | Request a fresh token; tokens older than 1 week are rejected. |
| 530006 | Username generation failed | Retry the operation (temporary error) |
| 530007 | Invalid client secret | Check or regenerate the client secret in the Developer Portal. |
| 530010 | User account non-provisional | The user is already linked to a Discord account; use the standard OAuth2 flow. |

 OIDC integrations may return more specific errors:
| Code | Meaning | Solution |
| --- | --- | --- |
| 530002 | Invalid issuer | Check that the `iss` claim in the ID token exactly matches the issuer URL in the OIDC configuration. |
| 530003 | Invalid audience | Check that the `aud` claim in the ID token includes the client ID from the OIDC configuration. |
| 530008 | OIDC configuration not found | Confirm the issuer URL is correct, reachable over HTTPS, and serves a valid discovery document without HTTP redirects. |
| 530009 | OIDC JWKS not found | Confirm the JWKS endpoint is reachable over HTTPS without HTTP redirects. |
| 530020 | Invalid OIDC JWT token | Confirm the ID token is signed correctly and uses a supported algorithm. |
| 530027 | Missing `kid` header | Make sure the ID token JWT header includes `kid` (Key ID) identifying the signing key. |

---

### Setting Display Names

 With these credentials, Discord creates a limited account for the game and attempts to set its username using these rules:
- Bot issued tokens use the supplied `preferred_global_name`.
- For OIDC, the provisional account display name comes from the `preferred_username` claim when the ID token includes it. This optional field should be between 1 and 32 characters. Without it, the display name defaults to the unique username Discord generates at creation.
- [Steam session tickets](https://partner.steamgames.com/doc/features/auth) use the display name from the user's Steam account.
- [EOS Auth](https://dev.epicgames.com/docs/epic-account-services/auth/auth-interface) Access Tokens or ID Tokens use the user's Epic account name. EOS Connect ID Tokens do not expose a username, so the game must set the display name with [`Client::UpdateProvisionalAccountDisplayName`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a7485979ab2d4c533b75f8efd5e50bc60).
- [Unity Services ID Tokens](https://services.docs.unity.com/docs/client-auth/) use the display name from the user's Unity Player Account.
 To set a provisional account display name directly, call [`Client::UpdateProvisionalAccountDisplayName`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a7485979ab2d4c533b75f8efd5e50bc60).

```
client->UpdateProvisionalAccountDisplayName("CoolPlayer123", [](discordpp::ClientResult result) {
    if (result.Successful()) {
      std::cout << "✅ Display name updated\n";
    }
  }
);

```

---

### Merging Provisional Accounts

 When a player converts a provisional account to a full Discord account, Discord starts a special version of the [access token request flow](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord#requesting-access-tokens) that includes the provisional user's external identity.
- If you have a backend, use [Merging Provisional Accounts for Servers](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#merging-provisional-accounts-for-servers)
- If you do not have a backend, use [Merging Provisional Accounts for Public Clients](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#merging-provisional-accounts-for-public-clients)
 The merge operation has a strict per-user limit, unlike most API rate limits, because account merging is expected to be rare. Normally a player links an account only once. For merge integration testing, add QA users to the application's [App Testers](https://discord.com/developers/applications/select/testers) list to avoid testing rate limits.

### Merging Provisional Accounts for Servers

 To merge a provisional account, extend the standard [OAuth2 token exchange](https://docs.discord.com/developers/topics/oauth2#authorization-code-grant-access-token-exchange-example) by posting to `/oauth2/token` with `external_auth_type` and `external_auth_token`. Those fields let Discord find the matching provisional identity and attach it to the full Discord account associated with the authorization code or device code. The External Auth Types table lists supported `external_auth_type` values. `external_auth_token` is the credential used when creating the provisional account, such as an OIDC identity token, Steam session ticket, or EOS access token. If the Bot Token Endpoint created the provisional account, set `external_auth_type` to `DISCORD_BOT_ISSUED_ACCESS_TOKEN` and set `external_auth_token` to `external_user_id` (your game account ID).

#### Desktop & Mobile

###### Request Body Parameters

| Parameter | Description |
| --- | --- |
| `grant_type` | Must be `authorization_code`. This standard OAuth2 authorization code grant exchanges the authorization code from `Client::Authorize` for an access token. |
| `code` | Authorization code returned to the server after the user completes `Client::Authorize`. |
| `redirect_uri` | Redirect URI from the original authorization request. It must match exactly. |
| `external_auth_type` | External identity provider type. See External Auth Types. |
| `external_auth_token` | External identity token. For `OIDC`, provide the OIDC identity token. For `DISCORD_BOT_ISSUED_ACCESS_TOKEN`, provide the `external_user_id` (game account ID) that created the provisional account. |

```
import requests

API_ENDPOINT = 'https://discord.com/api/v10'
CLIENT_ID = '332269999912132097'
CLIENT_SECRET = '937it3ow87i4ery69876wqire'
# See External Auth Types above for all supported values
EXTERNAL_AUTH_TYPE = 'DISCORD_BOT_ISSUED_ACCESS_TOKEN'

def exchange_code_with_merge(code, redirect_uri, external_auth_token):
  data = {
    'grant_type': 'authorization_code',
    'code': code,
    'redirect_uri': redirect_uri,
    'external_auth_type': EXTERNAL_AUTH_TYPE,
    'external_auth_token': external_auth_token
  }
  headers = {
    'Content-Type': 'application/x-www-form-urlencoded'
  }
  r = requests.post('%s/oauth2/token' % API_ENDPOINT, data=data, headers=headers, auth=(CLIENT_ID, CLIENT_SECRET))
  r.raise_for_status()
  return r.json()

```

#### Console

###### Request Body Parameters

| Parameter | Description |
| --- | --- |
| `grant_type` | Must be `urn:ietf:params:oauth:grant-type:device_code`. This RFC 8628 device authorization grant is used for consoles and devices without a browser. |
| `device_code` | Device code from the device authorization flow. See Account Linking on Consoles. |
| `external_auth_type` | External identity provider type. See External Auth Types. |
| `external_auth_token` | External identity token. For `OIDC`, provide the OIDC identity token. For `DISCORD_BOT_ISSUED_ACCESS_TOKEN`, provide the `external_user_id` (game account ID) that created the provisional account. |
