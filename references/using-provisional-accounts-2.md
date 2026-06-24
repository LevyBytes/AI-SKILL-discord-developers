# Using Provisional Accounts (part 2)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

```
import requests

API_ENDPOINT = 'https://discord.com/api/v10'
CLIENT_ID = '332269999912132097'
CLIENT_SECRET = '937it3ow87i4ery69876wqire'
# See External Auth Types above for all supported values
EXTERNAL_AUTH_TYPE = 'DISCORD_BOT_ISSUED_ACCESS_TOKEN'

def exchange_device_code_with_merge(device_code, external_auth_token):
  data = {
    'grant_type': 'urn:ietf:params:oauth:grant-type:device_code',
    'device_code': device_code,
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

#### Merge Request Response

```
{
  "access_token": "<access token>",
  "token_type": "Bearer",
  "expires_in": 604800,
  "refresh_token": "<refresh token>",
  "scope": "sdk.social_layer"
}

```

### Merging Provisional Accounts for Public Clients

 This approach requires **Public Client** on the app, a setting most shipped games should avoid unless the team intentionally accepts it. [Learn more](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes#oauth2-client-types). For clients without a backend, use [`Client::GetTokenFromProvisionalMerge`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a41062b7dafa331ddd2320daf1b4b273b) on Desktop & Mobile or [`Client::GetTokenFromDeviceProvisionalMerge`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#afd2207590ae7d6f60ee7bbb4fc7c21c8) on Console; those methods cover the process. First, enable Public Client in the Discord application's OAuth2 tab in the Discord developer portal. Then call [`Client::GetTokenFromProvisionalMerge`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a41062b7dafa331ddd2320daf1b4b273b) or [`Client::GetTokenFromDeviceProvisionalMerge`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#afd2207590ae7d6f60ee7bbb4fc7c21c8) from the client. Use this function together with [`Client::Authorize`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ace94a58e27545a933d79db32b387a468) when a provisional-account user wants to connect an existing Discord account or upgrade the provisional account into a full Discord account. During this account merging flow, provisional-account data is migrated into the Discord account: relationships, DMs, and lobby memberships move over. The provisional account is deleted after merge completion. If the user later unlinks, Discord creates a new provisional account with a new unique ID. The merge starts like standard login: call [`Client::Authorize`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ace94a58e27545a933d79db32b387a468) to receive an authorization code. Instead of `GetToken`, call the merge token function and include the provisional user's identity. Discord uses that identity to locate both the provisional account and the new Discord account, then merges data as needed. Callback behavior is covered by [`Client::GetToken`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#aaee636f91fb1ea3465157c20313b702c). The callback runs when token exchange finishes, but account merging is asynchronous and may still be in progress.

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
    std::cout << "✅ Authorization successful! Next step: GetTokenFromProvisionalMerge \n";

    // Retrieve your external auth token
    std::string externalAuthToken = GetExternalAuthToken();

    client->GetTokenFromProvisionalMerge(YOUR_DISCORD_APPLICATION_ID, code, codeVerifier, redirectUri, discordpp::AuthenticationExternalAuthType::OIDC, externalAuthToken,[client](
      discordpp::ClientResult result,
      std::string accessToken,
      std::string refreshToken,
      discordpp::AuthorizationTokenType tokenType,
      int32_t expiresIn,
      std::string scope) {
        if (result.Successful()) {
          std::cout << "🔓 Token received! Establishing connection...\n";
          client->UpdateToken(discordpp::AuthorizationTokenType::Bearer, accessToken, [client](discordpp::ClientResult result) {
            client->Connect();
          });
        } else {
          std::cerr << "❌ Token request failed: " << result.Error() << std::endl;
        }
    });

  }
});

```

### Data Migration During Merging

 When a provisional account is merged into a Discord account, Discord automatically moves this data:
- **Friends**: In-game friendships and Discord friendships created through the provisional account
- **Lobby Memberships**: Current and previous lobby participation
- **DM Messages**: Direct messages, including history
 This migration preserves the social graph a user built while they were using the provisional account.

### Merge Request Failures

 A merge attempt can return a merge-specific error code:
| Code | HTTP Status | Meaning | Solution |
| --- | --- | --- | --- |
| 530014 | 400 | Invalid merge source | The source account is not a provisional account. |
| 530016 | 400 | Invalid merge destination | The destination account is provisional |
| 530017 | 400 | Merge source user banned | The provisional account selected for merge is banned from the platform. |
| 530023 | 400 | Too many application identities | The user already has an external identity linked for this application. |
| - | 423 | Resource locked | Transient error, wait and retry |

 Error `530017` usually appears after a link had previously succeeded. If the linked Discord account is banned, Discord severs that identity into a new provisional account in a **restricted** state. That restricted provisional account cannot merge into another Discord account until a temporary ban expires or the ban is removed. Ban-Driven Unmerge describes the complete lifecycle.

---

### Unmerging Provisional Accounts

 A Discord account and provisional account link can be broken in four ways:
1. The user unmerges from the Discord client
2. A developer calls the Discord API unmerge endpoint
3. A developer uses the SDK helper for public clients
4. If Discord bans the user's Discord account, Discord automatically breaks the link; see Ban-Driven Unmerge below
 Unmerge completion invalidates every access token and refresh token for that user, so none of them can be reused. Connected game sessions are disconnected. The `external_auth_token` from the original provisional account setup remains valid. After unmerge finishes, reuse that same token to fetch the new provisional token for the newly created provisional account.

### Unmerging Provisional Accounts for Servers

 A developer can unmerge a user's account by requesting the Discord API unmerge endpoint.

```
import requests

API_ENDPOINT = 'https://discord.com/api/v10'
CLIENT_ID = '332269999912132097'
CLIENT_SECRET = '937it3ow87i4ery69876wqire'
EXTERNAL_AUTH_TYPE = 'OIDC'

def unmerge_provisional_account(external_auth_token):
  data = {
    'client_id': CLIENT_ID,
    'client_secret': CLIENT_SECRET,
    'external_auth_type': EXTERNAL_AUTH_TYPE,
    'external_auth_token': external_auth_token
  }
  r = requests.post('%s/partner-sdk/provisional-accounts/unmerge' % API_ENDPOINT, json=data, headers=headers)
  r.raise_for_status()

```

 When a server backend exists, prefer the server-to-server unmerge endpoint over the SDK helper method so the backend keeps stronger security and control of the unmerge flow.

#### Unmerging with Bot Token Endpoint

 If authentication uses the [Bot Token Endpoint](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#server-authentication-with-bot-token-endpoint), accounts can be unmerged without an external auth token.

```
import requests

API_ENDPOINT = 'https://discord.com/api/v10'
BOT_TOKEN = 'YOUR_BOT_TOKEN'

def unmerge_provisional_account(external_user_id):
  data = {
    'external_user_id': external_user_id # identifier used in the /token/bot endpoint
  }
  headers = {
    'Content-Type': 'application/json',
    'Authorization': f'Bot {BOT_TOKEN}'
  }
  r = requests.post('%s/partner-sdk/provisional-accounts/unmerge/bot' % API_ENDPOINT, json=data, headers=headers)
  r.raise_for_status()

```

 This endpoint is also useful when the Discord Auth token was lost because of an error or data loss, and an unmerge is needed to move the user to a provisional account before linking Discord again.

### Unmerging Provisional Accounts for Public Clients

 This method requires **Public Client** on the app, which most shipped games should avoid. [Learn more](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes#oauth2-client-types). For the fastest public-client unmerge path, call [`Client::UnmergeIntoProvisionalAccount`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a2da21ae8a3015e0e5e42c1a7226b256f); it performs the full process. The method is intended for public clients with no backend server. **Important Notes:**
- This function is only for **public clients**, meaning applications without backend servers
- Enable "Public Client" in the Discord application's OAuth2 tab in the Discord developer portal
- After unmerge, call [`Client::GetProvisionalToken`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a8003130b6c46e54ac68442483bf0480c) to retrieve a fresh provisional token for the new provisional account

```
// unmerge a user account
void UnmergeUserAccount(const std::shared_ptr<discordpp::Client>& client) {
    // Get your external auth token (Steam, OIDC, etc.)
    std::string externalToken = GetExternalAuthToken();

    // Unmerge the Discord account from the external identity
    client->UnmergeIntoProvisionalAccount(
        YOUR_DISCORD_APPLICATION_ID,
        discordpp::AuthenticationExternalAuthType::OIDC, // or STEAM, EOS, etc.
        externalToken,
        [client, externalToken](const discordpp::ClientResult &result) {
            if (result.Successful()) {
                std::cout << "✅ Account unmerged successfully! Creating new provisional account...\n";

                // Now get a new provisional token for the unlinked identity
                client->GetProvisionalToken(
                    YOUR_DISCORD_APPLICATION_ID,
                    discordpp::AuthenticationExternalAuthType::OIDC,
                    externalToken,
                    [client](const discordpp::ClientResult &result,
                                 const std::string &accessToken,
                                                     const std::string& refreshToken,
                                                     discordpp::AuthorizationTokenType tokenType,
                                                     int32_t expiresIn,
                                                     const std::string& scopes) {
                        if (result.Successful()) {
                            std::cout << "🔓 New provisional account created! Establishing connection...\n";
                            client->UpdateToken(discordpp::AuthorizationTokenType::Bearer, accessToken,
                                [client](const discordpp::ClientResult &updateResult) {
                                    if (updateResult.Successful()) {
                                        client->Connect();
                                    } else {
                                        std::cerr << "❌ Failed to update token: " << updateResult.Error() << std::endl;
                                    }
                                }
                            );
                        } else {
                            std::cerr << "❌ Failed to create new provisional account: " << result.Error() << std::endl;
                        }
                    }
                );
            } else {
                std::cerr << "❌ Unmerge failed: " << result.Error() << std::endl;
            }
        }
    );
}

```

### Out-of-Band Unmerge

 The link may also break even when application code never calls an unmerge endpoint:
- **User-initiated**: the user removes the app from Discord `User Settings -> Authorized Apps`. This produces a normal unmerge, and the user may link again later.
- **Ban-driven**: if Discord bans the user's Discord account, Discord breaks the link automatically. Mechanically this is an unmerge, but it has additional lifecycle effects; see Ban-Driven Unmerge below.
 Both paths expose the same auth-side signals to the app: an `APPLICATION_DEAUTHORIZED` webhook fires and stored tokens become invalid. The Account Linking guide's [Out-of-Band Revocation](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord#out-of-band-revocation) section explains details and recovery. These flows require no app code changes, but an in-app unmerge option using one of the methods above usually gives users a better experience.

#### Ban-Driven Unmerge

 When Discord temporarily or permanently bans a player's Discord account, Discord automatically unmerges it. Two effects follow:
1. **OAuth2 tokens are deleted immediately**. This produces the same `APPLICATION_DEAUTHORIZED` webhook and `invalid_grant` on refresh that other out-of-band revocations produce. There is no grace period.
2. **A new provisional account is created for the same external identity.** Normal unmerge migration still applies, so username, friends list, lobbies, and related data are preserved and the player keeps the in-game social graph after the Discord identity is removed. The new provisional account starts in a **restricted** state.
 While that new provisional account remains restricted:
- It **cannot be merged** into another Discord account. A merge attempt through `/oauth2/token` with `external_auth_token` fails with error `530017` "Merge source user is banned".
- For a **temporary ban**, the restriction is removed automatically when the ban ends, after which the provisional account can merge again.
- For a **permanent ban**, the restriction remains indefinitely.
 No API reports whether a player's Discord account is currently banned. Use `APPLICATION_DEAUTHORIZED` and/or `invalid_grant` on refresh as the signal to fall back to the provisional account flow.

### Data Migration During Unmerging

 When a user unmerges, Discord creates a provisional account with a new user ID. This data moves to that new provisional account:
- **Username**: The global name is copied onto the new provisional account
- **In-game friends**: All are copied to the new provisional account
- **Discord friends who use this application**: Copied to the provisional account
- **Blockers**: Accounts that blocked the original Discord account remain preserved
- **Lobbies**: Active lobby memberships for the application are moved
 This data is **not** moved:
- **Discord friends who don't use this application**: Not transferred
- **DM message history**: Not moved into provisional accounts
 Provisional accounts can include Discord friends, but messaging those friends is allowed only while actively playing the game.

### Unmerge Request Failures

 An unmerge attempt can return an unmerge-specific error code:
| Code | HTTP Status | Meaning | Solution |
| --- | --- | --- | --- |
| 50229 | 400 | Invalid user type | User account is provisional and cannot be unmerged. |
| - | 404 | Unknown user | No user identity found for the provided external identity |

---

### Next Steps

 After provisional accounts are configured for the game, continue with other Discord Social SDK features:

### Design: Provisional Accounts

Design guidance for adding provisional accounts to a game.

### Creating a Unified Friends List

Merge Discord friends and game friends into one manageable list.

### Setting Rich Presence

Show game status and details to Discord friends. For help, join the [Discord Developers Server](https://discord.gg/discord-developers) and ask in `#social-sdk-dev-help`. Bugs found while using the Social SDK can be reported at [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report).

---

### Change Log

| Date | Changes |
| --- | --- |
| May 22, 2026 | Document ban-driven unmerge lifecycle |
| April 21, 2026 | Document OIDC integration requirements |
| April 14, 2026 | Clarify provisional account token refresh flow |
| April 13, 2026 | Clarify Merging Provisional Accounts for Servers |
| February 25, 2026 | Clarify unmerge behavior and data migration |
| March 17, 2025 | Initial release |
