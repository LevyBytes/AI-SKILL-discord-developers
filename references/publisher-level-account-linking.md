# Publisher Level Account Linking

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Publisher Level Account Linking is gated by Discord approval. To ask about access, work with your Discord account representative or [contact us](https://discord.com/developers/contact-us).

### Overview

 Publisher Level Account Linking lets a developer that operates several games use one authorization flow for the full portfolio. Players authorize through a publisher application rather than authorizing separately per game. The model uses a parent-child relationship where the publisher application is the parent and each game application is a child. After that relationship exists, the publisher application handles player authentication, while a game application is used to connect to and operate the Discord Social SDK. This guide explains how to:
- Configure publisher applications and game applications
- Trade parent tokens for child tokens
- Authenticate the Discord Social SDK with child tokens
- Understand token lifecycle and limitations

### Benefits

### For Publishers

- **Centralized Account Management**: Keep user authorization under the publisher application

### For Players

- **Simplified Experience**: Lower friction for players moving across games from one publisher
- **Richer Social Features**: Use improved friend and activity data across the publisher's catalog

---

### Is Publisher Level Account Linking Right for You?

 Publisher Level Account Linking fits publishers that have a centralized social layer across games. It is not appropriate for every publisher with more than one game.
| **Good Fit** | **Poor Fit** |
| --- | --- |
| Publishers whose games share both a centralized account system **and** a publisher-level social experience, such as a friends list that spans the portfolio. | Publishers whose games only share a login system and do not share cross-game social features. |

 Publisher Level Account Linking was built for cross-game social ecosystems. If a player connects Discord to a publisher account and that publisher already shares a social graph between titles, the authorization model is easy to understand: friends from one game can appear throughout the portfolio.

### Key Behaviors to Understand

 Before adopting Publisher Level Account Linking, be clear about these system-level constraints:

#### Authorization is all-or-nothing at the publisher level:

- After a player links Discord to the publisher application, the authorization covers **all** child game applications beneath that publisher
- Players cannot stay linked at the publisher level while disabling Discord for only one child game
- Revocation breaks the Discord connection between the publisher account and the Discord account, which impacts every game

#### Relationships exist at the publisher level, not per-game:

 Friend relationships are scoped to the publisher, not separately to each game. There are no isolated per-game friend lists. If your product requires different friend lists for different games, Publisher Level Account Linking is the wrong model. **This is a one-way architectural decision.** After child applications are attached to a parent publisher application, some data is deleted and the relationship cannot be undone. Review the migration carefully before changing existing applications.

---

### How It Works

 The Publisher Authentication model uses two application roles:
1. **Publisher application (parent)**: Runs OAuth2 account linking and owns user authorization
2. **Game application (child)**: Receives child access tokens from the parent and uses them for Social SDK calls
 The flow works as follows:
1. The user authorizes once through the publisher application with standard OAuth2
2. The developer exchanges the parent access token at `/partner-sdk/child-token` to receive a child access token
3. The child token initializes the Discord Social SDK for the selected game
4. Subsequent Social SDK operations run under that child token
 The table below separates responsibilities between the Publisher application and the game application:
| **Action** | **Publisher Application (Parent)** | **Game Application (Child)** |
| --- | --- | --- |
| Discord account authentication | ✅ | ❌ |
| Provisional account authentication | ✅ | ❌ |
| Creates parent access tokens and refresh tokens | ✅ | ❌ |
| Trades parent tokens for a child token | ✅ | ❌ |
| Used when connecting to the Social SDK | ❌ | ✅ |
| Runs within the context of a single game | ❌ | ✅ |

---

### Prerequisites

 Before starting, confirm that you have:
- A Discord application acting as the publisher (parent) with OAuth2 configured
- At least one Discord application acting as a game (child)
- Social SDK integration enabled on every relevant application
- A Discord contact path for setting up parent-child application relationships

---

### Application Configuration

### 1. Configure the Publisher Application

 Configure the publisher application by following the normal [OAuth2 guidelines](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord):
1. Open the [Developer Portal](https://discord.com/developers/applications)
2. Select your publisher application
3. Add OAuth2 redirect URIs in the OAuth2 settings
4. Enable the Social SDK integration
5. Record the Client ID and Client Secret

### 2. Configure Game Applications

 For each game application:
1. Create another application in the [Developer Portal](https://discord.com/developers/applications)
2. Enable Social SDK integration
3. **Do not configure OAuth2 redirect URIs** - child game applications do not use them
4. Record the application's Client ID
 Game applications do not need OAuth2 setup because their tokens come from the parent application's exchange flow.

### 3. Establish Parent-Child Relationships

 Discord must link your game applications to the publisher application. To request those child-to-parent links, provide Discord with the following:
- Parent application ID (publisher)
- Child application IDs (games)
 Discord will create the relationships internally so token exchange can work. After the parent-child relationship exists, publisher and game applications lose access to some actions. Migrating an existing application into either role can immediately disrupt certain flows, so discuss the migration with Discord before starting. Another option is to create new publisher and game applications specifically for Publisher Level Account Linking. When a child application is linked, the following child data is removed:
| **Data Type** | **Action** |
| --- | --- |
| OAuth2 tokens | Every token for the child app is deleted |
| Lobbies | Every lobby tied to the child app is deleted |
| User Application Identities | Deleted for all users |
| Provisional accounts | All provisional users are deleted |
| Game Relationships | Removed for every user linked to the app |

 The parent application is not changed; existing parent tokens remain valid because they still work with ordinary OAuth2 flows.

### 4. Match Confidentiality Settings

 Child applications must use the same confidentiality model as the parent application:
- If the parent is a **confidential client** with a client secret, every child must also be confidential
- If the parent is a **public client**, every child must also be public

---

### Implementing Token Exchange

 Publisher Level Account Linking works with standard OAuth2 for linked Discord users and with provisional accounts for users who do not have Discord accounts.

### Linked Discord Account Flow

### Provisional Account Flow

### Obtaining the Parent Access Token

 Before exchanging tokens, get the parent access token by authorizing with the **publisher application ID**. Either Discord OAuth2 authorization or provisional-account authorization can produce it. For publisher level account linking, use the publisher application ID for all authorization work. OAuth2 and provisional account requests made with child (game) application IDs fail. User authorization is only valid through the publisher application. Follow [Account Linking with Discord](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord) or [Using Provisional Accounts](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts) with your publisher application id; the result is a parent access token and refresh token.

### Exchanging Parent Token for Child Token

 After the OAuth2 flow returns a parent access token, use `/partner-sdk/child-token` to exchange it for a child token. **Important Token Usage Restrictions** Parent access tokens cannot connect to Discord through the Social SDK; **only** child access tokens can do that
 Child applications cannot run OAuth2 account linking flows; child access tokens are available **only** from the token exchange flow
 For direct Discord server API calls, send the child access token as Bearer authorization

#### Exchange for Child Token

```
import requests

API_ENDPOINT = 'https://discord.com/api/v10'
PARENT_ACCESS_TOKEN = 'parent_token_here'
CHILD_APPLICATION_ID = '987654321098765432'
PARENT_CLIENT_SECRET = 'your_parent_client_secret'

def exchange_for_child_token(parent_token, child_app_id, parent_client_secret):
    response = requests.post(
        f'{API_ENDPOINT}/partner-sdk/child-token',
        headers={
            'Content-Type': 'application/json'
        },
        json={
            'parent_access_token': parent_token,
            'child_application_id': child_app_id,
            'parent_client_secret': parent_client_secret
        }
    )
    response.raise_for_status()
    return response.json()

# Exchange the token
child_token_data = exchange_for_child_token(
    PARENT_ACCESS_TOKEN,
    CHILD_APPLICATION_ID,
    PARENT_CLIENT_SECRET
)

print(f"Child access token: {child_token_data['access_token']}")
print(f"Expires in: {child_token_data['expires_in']} seconds")

```

#### Token Exchange Response

```
{
  "access_token": "child_application_access_token",
  "token_type": "Bearer",
  "expires_in": 604800,
  "scope": "sdk.social_layer"
}

```

### Response Fields

| Field | Description |
| --- | --- |
| `access_token` | Child access token used by the Social SDK |
| `token_type` | Always “Bearer” |
| `expires_in` | Expiration duration in seconds, matching the parent token's expiry |
| `scope` | OAuth2 scopes for the token |

 The child token endpoint does not issue a refresh token. To get another child token, perform the exchange again with the parent token.

---

### Integrating with the Social SDK

 After your backend completes token exchange, pass the child token into the game client's Social SDK. This section describes how publisher authentication changes SDK configuration.

### Use the Publisher Application ID for Authorization

 When the game starts authorization, use the **publisher application ID** rather than the game application ID:

```
// Use the PUBLISHER_APPLICATION_ID for authorization
discordpp::AuthorizationArgs args{};
args.SetClientId(PUBLISHER_APPLICATION_ID);  // Parent app ID, not child
args.SetScopes(discordpp::Client::GetDefaultPresenceScopes());

client->Authorize(args, [](discordpp::ClientResult result, std::string code, std::string redirectUri) {
    if (result.Successful()) {
        std::cout << "✅ Authorization successful!" << std::endl;
        // Exchange code for parent token, then exchange parent token for child token
    }
});

```

### Connect the SDK with the Child Token

 After your backend returns the child token, authenticate the Social SDK with it:

```
#include <discord-social-sdk/discord.h>

void InitializeSDKWithChildToken(const std::string& childAccessToken) {
    // Create SDK client instance
    auto client = std::make_shared<discordpp::Client>();

    // Update the client with the child token
    client->UpdateToken(
        discordpp::AuthorizationTokenType::Bearer,
        childAccessToken,
        [client](discordpp::ClientResult result) {
            if (result.Successful()) {
                std::cout << "✅ Token updated successfully!" << std::endl;

                // Connect to Discord
                client->Connect();
            } else {
                std::cerr << "❌ Failed to update token: " << result.Error() << std::endl;
            }
        }
    );
}

```

 After connection, Social SDK calls operate as the game (child) application.

---

### Token Lifecycles

 Child game application tokens inherit their lifecycle from the parent publisher application tokens. Understanding refresh, expiration, and revocation behavior is necessary for a correct integration.

### Token Expiration

 Game token expiration follows the parent publisher token:
- Game tokens expire at the **same time** as the publisher token
- The game token `expires_in` value equals the publisher token's remaining lifetime
- Game tokens do not include refresh tokens

### Publisher Token Refresh

 When tokens for the publisher application are refreshed:
- Existing game tokens remain valid
- Existing game tokens keep the original publisher token expiration time
- To obtain a game token with the refreshed expiry, exchange the refreshed publisher token again

### Publisher Token Revoked

 When tokens for the publisher application are revoked:
- Every linked child game token becomes invalid immediately
- SDK sessions authenticated with game tokens are disconnected

---

### Revoking Access Tokens

 In a publisher-level account linking integration, token revocation depends on which application ID the token represents.

### Token Issuance vs. Revocation

 Game tokens are issued under publisher authority, but revocation uses different application identity rules:
- **Game tokens are issued by the publisher application** - The `/partner-sdk/child-token` endpoint relies on publisher application credentials when issuing game tokens
- **Revocation uses the token's associated application ID**:
  - **Publisher token revocation** -> Use the **parent publisher application ID**
  - **Game token revocation** -> Use the **child game application ID**

 When revoking a token, choose the application ID represented by that token, not the application ID that originally issued it.

### Revoking Publisher Tokens

 To revoke a publisher access token, send the publisher application ID and credentials:

```
import requests

API_ENDPOINT = 'https://discord.com/api/v10'
PARENT_CLIENT_ID = 'YOUR_PARENT_CLIENT_ID'
PARENT_CLIENT_SECRET = 'YOUR_PARENT_CLIENT_SECRET'

def revoke_parent_token(parent_token):
    data = {'token': parent_token}
    headers = {'Content-Type': 'application/x-www-form-urlencoded'}
    r = requests.post(
        f'{API_ENDPOINT}/oauth2/token/revoke',
        data=data,
        headers=headers,
        auth=(PARENT_CLIENT_ID, PARENT_CLIENT_SECRET)
    )
    r.raise_for_status()

```

 Revoking a publisher token invalidates all publisher access and refresh tokens for that user and also **immediately invalidates all associated game tokens**. Any SDK session using a game token is disconnected.

### Revoking Game Tokens

 To revoke a game access token, send the game application ID and credentials:

```
import requests

API_ENDPOINT = 'https://discord.com/api/v10'
CHILD_CLIENT_ID = 'YOUR_CHILD_CLIENT_ID'
CHILD_CLIENT_SECRET = 'YOUR_CHILD_CLIENT_SECRET'

def revoke_child_token(child_token):
    data = {'token': child_token}
    headers = {'Content-Type': 'application/x-www-form-urlencoded'}
    r = requests.post(
        f'{API_ENDPOINT}/oauth2/token/revoke',
        data=data,
        headers=headers,
        auth=(CHILD_CLIENT_ID, CHILD_CLIENT_SECRET)
    )
    r.raise_for_status()

```

 For token revocation API details and extra options, see [Revoking Access Tokens](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord#revoking-access-tokens) in the Account Linking guide.

---

### Alternatives for Publishers Without a Shared Social Graph

 If your studio has a central publisher account system but no social graph shared between games, you can instead build a central backend that manages distinct Discord application IDs per game under the standard authentication model. That still lets you reuse Social SDK integration code across titles while keeping each game's Discord connection independent. You could also provide publisher account settings where players link, unlink, and choose which Discord account is connected to each game. The cost is that every game needs its own authorization and the implementation is larger, but players get more precise control over which games connect to Discord, which may better match your studio or game requirements.

---

### Next Steps

 After configuring publisher level account linking, continue with other Discord Social SDK capabilities:

### Creating a Unified Friends List

Merge Discord friends and game friends into one easier-to-manage list.

### Setting Rich Presence

Show game state and activity information to friends on Discord.

### Managing Lobbies

Gather players in a shared lobby with invites, text chat, and voice communication. Need help? Join the [Discord Developers Server](https://discord.gg/discord-developers) and ask in `#social-sdk-dev-help` for community support. If you find a Social SDK bug, report it at [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report).

---

### Change Log

| Date | Changes |
| --- | --- |
| Febraury 18, 2026 | Initial release |
