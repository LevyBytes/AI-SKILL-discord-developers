# Oauth2 (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

OAuth2 lets application developers create apps that use Discord API authentication and data. Discord supports several OAuth2 authentication patterns: authorization code grant, implicit grant, client credentials, plus Discord-specific variants for Bots and Webhooks.

### Shared Resources

 Start OAuth2 setup by [registering a developer application](https://discord.com/developers/applications), then collect the client ID and client secret. Most OAuth2 implementations should use a library for the chosen programming language. If you implement OAuth2 directly, refer to [RFC 6749](https://tools.ietf.org/html/rfc6749). After the Discord application exists, keep `client_id` and `client_secret` available, then choose the OAuth2 flow that matches your use case.

###### OAuth2 URLs

| URL | Description |
| --- | --- |
| https://discord.com/oauth2/authorize | Base authorization URL |
| https://discord.com/api/oauth2/token | Token URL |
| https://discord.com/api/oauth2/token/revoke | Token Revocation URL |

 Following the relevant RFCs, the token URL and token revocation URL accept **only** `application/x-www-form-urlencoded` as the content type. JSON bodies are rejected with an error.

###### OAuth2 Scopes

 This table lists the OAuth2 scopes supported by Discord. Some scopes need Discord approval before use. Requesting an approval-required scope from a user without approval can produce errors or undocumented OAuth2 behavior. For scopes restricted to approved partners, contact your Discord account representative to discuss whether access fits your needs.
| Name | Description |
| --- | --- |
| activities.read | permits the app to fetch a user's "Now Playing/Recently Played" data; not currently available for apps |
| activities.write | lets the app modify a user's current activity; unavailable to apps at this time (NOT REQUIRED FOR GAMESDK ACTIVITY MANAGER) |
| applications.builds.read | permits the app to read build data for the user's applications |
| applications.builds.upload | permits the app to upload or update builds for the user's applications; approved partners only |
| applications.commands | permits the app to add commands to a guild; included by default with `bot` |
| applications.commands.update | permits the app to update its commands with a Bearer token; client credentials grant only |
| applications.commands.permissions.update | permits the app to update guild command permissions where the user has permission |
| applications.entitlements | permits the app to read entitlements for the user's applications |
| applications.store.update | permits the app to read and modify store data for the user's applications, including SKUs, store listings, achievements, and similar data |
| bot | for oauth2 bots, adds the bot to the user's selected guild by default |
| connections | permits `/users/@me/connections` to return connected third-party accounts |
| dm_channels.read | permits the app to view details about the user's DMs and group DMs; approved partners only |
| email | enables `/users/@me` to return an `email` |
| gdm.join | permits the app to add users to a group dm |
| guilds | permits `/users/@me/guilds` to return basic details for all guilds belonging to a user |
| guilds.join | permits `/guilds/{guild.id}/members/{user.id}` for adding users to a guild |
| guilds.members.read | permits `/users/@me/guilds/{guild.id}/member` to return the user's member data in a guild |
| identify | allows `/users/@me` without `email` |
| identify.premium | permits the app to read a user's Nitro subscription type from `premium_type` on the User object; approved partners only |
| messages.read | for local rpc server api access, permits reading messages from every client channel instead of only app-created channels or guilds |
| relationships.read | Grants access to the user's Discord Friends list, pending requests, and blocked users. This is a Social SDK scope; request access here. The Social SDK Terms, including Section 5(a)(ii), govern the data obtained through it |
| role_connections.write | permits the app to update a user's app connection and metadata |
| rpc | for local rpc server access, permits control of the user's local Discord client; approved partners only |
| rpc.activities.write | for local rpc server access, permits updating a user's activity; approved partners only |
| rpc.notifications.read | for local rpc server access, permits receiving notifications pushed to the user; approved partners only |
| rpc.voice.read | for local rpc server access, permits reading a user's voice settings and listening for voice events; approved partners only |
| rpc.voice.write | for local rpc server access, permits changing a user's voice settings; approved partners only |
| voice | permits the app to connect to voice for the user and view all voice members; approved partners only |
| webhook.incoming | creates a webhook returned in the oauth token response for authorization code grants |

 To add a user to a guild, the bot must already be a member of that guild. `role_connections.write` is not compatible with the [Implicit grant type](https://docs.discord.com/developers/topics/oauth2#implicit-grant).

### State and Security

 Before comparing OAuth2 grant semantics, address security through the `state` parameter. [Cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery), or CSRF, and [Clickjacking](https://en.wikipedia.org/wiki/Clickjacking) are risks OAuth implementers need to mitigate. The usual mitigation is `state`. The client sends `state` with the authorization request, and Discord returns it with the response. Its value should bind the request to the user's authenticated session, such as a hash of the session cookie or another nonce associated with that user's session. When the user begins authorization, generate a request-specific `state` value and store it somewhere accessible only to that client and user, such as storage protected by the [same-origin policy](https://en.wikipedia.org/wiki/Same-origin_policy). After redirect, compare the returned `state` to the stored value. A match indicates a valid authorization request. Treat a missing `state` or mismatched `state` as failed validation. A mismatch may mean interception or false authorization against another user's resources, so deny the request. Discord does not require `state`, but supports it and strongly recommends using it to protect applications and data.

### Authorization Code Grant

 The authorization code grant is the OAuth2 flow many developers consider the standard path: obtain an authorization code, then exchange it for the user's access token. The authorization server mediates between the client and resource owner, so the resource owner's credentials are not shared directly with the client. Requests to OAuth2 endpoints must use either HTTP Basic authentication or include `client_id` and `client_secret` in the form data body.

###### Authorization URL Example

```
https://discord.com/oauth2/authorize?response_type=code&client_id=157730590492196864&scope=identify%20guilds.join&state=15773059ghq9183habn&redirect_uri=https%3A%2F%2Fnicememe.website&prompt=consent&integration_type=0

```

 `client_id` is the application's `client_id`. `scope` is a list of [OAuth2 scopes](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-scopes) separated by URL-encoded spaces (`%20`). `redirect_uri` is the registered redirect URL for the application, URL-encoded. `state` is the unique value described in [State and Security](https://docs.discord.com/developers/topics/oauth2#state-and-security). Opening this URL prompts the user to authorize the requested scopes. If accepted, Discord redirects to `redirect_uri` with a querystring `code`. If `state` was supplied, it is returned too and should be validated then. `prompt` controls behavior when authorization already exists. With `consent`, a user who already authorized the requested scopes is asked to approve again. With `none`, Discord skips the authorization screen and redirects to the redirect URI without asking again. Passthrough scopes such as `bot` and `webhook.incoming` always require authorization. `integration_type` specifies the [installation context](https://docs.discord.com/developers/resources/application#installation-context). The installation context decides where the application is installed and matters only when `scope` includes `applications.commands`. Value 0 (GUILD_INSTALL) authorizes installation to a server, while value 1 (USER_INSTALL) authorizes installation to a user. The Developer Portal must configure the application to support the requested `integration_type`.

###### Redirect URL Example

```
https://nicememe.website/?code=NhhvTDYsFcdgNLnnLijcl7Ku7bEEeee&state=15773059ghq9183habn

```

 Exchange `code` for the user's access token by sending a `POST` request to the [token URL](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-urls) with these parameters:
- `grant_type` - set this to `authorization_code`
- `code` - the code taken from the querystring
- `redirect_uri` - the `redirect_uri` tied to this authorization, usually the one used in the authorization URL

###### Access Token Exchange Example

```
import requests

API_ENDPOINT = 'https://discord.com/api/v10'
CLIENT_ID = '332269999912132097'
CLIENT_SECRET = '937it3ow87i4ery69876wqire'
REDIRECT_URI = 'https://nicememe.website'

def exchange_code(code):
  data = {
    'grant_type': 'authorization_code',
    'code': code,
    'redirect_uri': REDIRECT_URI
  }
  headers = {
    'Content-Type': 'application/x-www-form-urlencoded'
  }
  r = requests.post('%s/oauth2/token' % API_ENDPOINT, data=data, headers=headers, auth=(CLIENT_ID, CLIENT_SECRET))
  r.raise_for_status()
  return r.json()

```

 In response, you will receive:

###### Access Token Response

```
{
  "access_token": "6qrZcUqja7812RVdnEKjpzOL4CvHBFG",
  "token_type": "Bearer",
  "expires_in": 604800,
  "refresh_token": "D43f5y0ahjqew82jZ4NViEr2YafMKhue",
  "scope": "identify"
}

```

 With the user's access token, the application can call selected API routes on that user's behalf, limited to the requested scopes. `expires_in` gives the number of seconds until the access token expires, so the app can refresh before expiration. To refresh it, send another `POST` request to the [token URL](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-urls) with these parameters:
- `grant_type` - set this to `refresh_token`
- `refresh_token` - the user's refresh token

###### Refresh Token Exchange Example

```
import requests

API_ENDPOINT = 'https://discord.com/api/v10'
CLIENT_ID = '332269999912132097'
CLIENT_SECRET = '937it3ow87i4ery69876wqire'

def refresh_token(refresh_token):
  data = {
    'grant_type': 'refresh_token',
    'refresh_token': refresh_token
  }
  headers = {
    'Content-Type': 'application/x-www-form-urlencoded'
  }
  r = requests.post('%s/oauth2/token' % API_ENDPOINT, data=data, headers=headers, auth=(CLIENT_ID, CLIENT_SECRET))
  r.raise_for_status()
  return r.json()

```

 Boom; fresh [access token response](https://docs.discord.com/developers/topics/oauth2#authorization-code-grant-access-token-response)!

###### Token Revocation Example

 Revoke an access token or refresh token by sending a `POST` request to the [token revocation URL](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-urls) with these parameters:
- `token` - the access token or refresh token being revoked
- `token_type_hint` *(optional)* - indicates the kind of `token`; either `access_token` or `refresh_token`
 Revoking a token revokes every active access token and refresh token associated with that authorization, no matter what `token` or `token_type_hint` values you send.

```
import requests

API_ENDPOINT = 'https://discord.com/api/v10'
CLIENT_ID = '332269999912132097'
CLIENT_SECRET = '937it3ow87i4ery69876wqire'

def revoke_access_token(access_token):
  data = {
    'token': access_token,
    'token_type_hint': 'access_token'
  }
  headers = {
    'Content-Type': 'application/x-www-form-urlencoded'
  }
  requests.post('%s/oauth2/token/revoke' % API_ENDPOINT, data=data, headers=headers, auth=(CLIENT_ID, CLIENT_SECRET))

```

 The related tokens are now revoked.

### Implicit Grant

 The implicit OAuth2 grant is a simpler flow intended for browser-based clients. Instead of returning an authorization code that the client exchanges for an access token, Discord issues the access token directly to the client. The URL uses this form:

###### Authorization URL Example

```
https://discord.com/oauth2/authorize?response_type=token&client_id=290926444748734499&state=15773059ghq9183habn&scope=identify

```

 After redirect, the redirect URI contains additional **URI fragments**: `access_token`, `token_type`, `expires_in`, `scope`, and [`state`](https://docs.discord.com/developers/topics/oauth2#state-and-security) if provided. **They are not querystring parameters.** Pay attention to the "#" character:

###### Redirect URL Example

```
https://findingfakeurlsisprettyhard.tv/#access_token=RTfP0OK99U3kbRtHOoKLmJbOn45PjL&token_type=Bearer&expires_in=604800&scope=identify&state=15773059ghq9183habn

```

 The implicit grant has tradeoffs. It is faster and simpler to implement, but the token is returned in the URI fragment rather than exchanged through a secure HTTP body, which can expose it to unauthorized parties. **No refresh token is returned, so the user must explicitly authorize again after the token expires.**

### Client Credentials Grant

 The client credential flow gives bot developers a quick path to bearer tokens for testing. Send a `POST` request to the [token URL](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-urls) using grant type `client_credentials` and Basic authentication, with the client id as the username and client secret as the password. Discord returns an access token for the bot owner. Because of that, treat `client_id` and `client_secret` as sensitive and keep them out of source code. You may provide scopes through the `scope` parameter, a space-separated list of [OAuth2 scopes](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-scopes). Team applications are restricted to `identify` and `applications.commands.update`, because teams are not tied to one specific user.

###### Client Credentials Token Request Example

```
import requests

API_ENDPOINT = 'https://discord.com/api/v10'
CLIENT_ID = '332269999912132097'
CLIENT_SECRET = '937it3ow87i4ery69876wqire'

def get_token():
  data = {
    'grant_type': 'client_credentials',
    'scope': 'identify connections'
  }
  headers = {
    'Content-Type': 'application/x-www-form-urlencoded'
  }
  r = requests.post('%s/oauth2/token' % API_ENDPOINT, data=data, headers=headers, auth=(CLIENT_ID, CLIENT_SECRET))
  r.raise_for_status()
  return r.json()

```

 The response includes an access token and does not include a refresh token:

###### Client Credentials Access Token Response

```
{
  "access_token": "6qrZcUqja7812RVdnEKjpzOL4CvHBFG",
  "token_type": "Bearer",
  "expires_in": 604800,
  "scope": "identify connections"
}

```

### Bot Users

 Discord's API includes bot users, a user type dedicated to automation. Every app automatically receives a bot user, authenticated with the bot token from the [app's settings](https://discord.com/developers/applications). Unlike ordinary OAuth2, bot users can access most API routes without bearer tokens and can connect to the [Real Time Gateway](https://docs.discord.com/developers/events/gateway).

### Bot vs User Accounts

 Developers must follow the [terms of service](https://support-dev.discord.com/hc/articles/8562894815383-Discord-Developer-Terms-of-Service), including the restriction against automating normal user accounts, often called "self-bots", outside the OAuth2/bot API. Bot users differ from regular Discord users in several ways:
1. Bots join guilds through the OAuth2 API and cannot accept ordinary invites.
2. Bots cannot have friends, be added to Group DMs, or join Group DMs.
3. [Verified bots](https://support-dev.discord.com/hc/en-us/articles/23926564536471-How-Do-I-Get-My-App-Verified) are not capped at a maximum guild count.
4. Bots use their own separate [rate limits](https://docs.discord.com/developers/topics/rate-limits#rate-limits).

### Bot Authorization Flow

 Bot authorization is a callback-less, server-less OAuth2 variant that lets users add bots to guilds with little setup. The generated URL resembles the full-stack version:

###### Bot Auth Parameters

| name | description |
| --- | --- |
| client_id | your app’s client id |
| scope? | must include `bot` for the bot flow |
| permissions? | the permissions you’re requesting |
| guild_id? | preselects a guild in the user's guild dropdown |
| disable_guild_select? | `true` or `false`; prevents the user from changing the guild dropdown |
| integration_type? | authorization installation context |

###### URL Example

```
https://discord.com/oauth2/authorize?client_id=157730590492196864&scope=bot&permissions=1

```

 For bots, set `scope` to `bot`. The `permissions` parameter is also available; it is an integer derived from the bot's [permission calculations](https://docs.discord.com/developers/topics/permissions#permissions-bitwise-permission-flags). Notice that `response_type` and `redirect_uri` are absent, because bot authorization does not retrieve the user's access token. When a user opens this page, Discord prompts them to add the bot to a guild where they have the needed permissions. If they accept, the bot is added. If you already know the ID of the target guild, include it as `guild_id=GUILD_ID`; when the page opens, Discord preselects that guild if the user can add the bot there. Combine this with `disable_guild_select=true` to prevent the user from choosing another guild. If the URL supplies only `client_id`, and optionally `permissions`, `guild_id`, or `disable_guild_select`, authorization uses the [default install settings](https://docs.discord.com/developers/resources/application#configuring-an-install-link-and-default-install-settings) from the developer portal. Supplying `scope`, `integration_type`, or `redirect_uri` overrides that default behavior. If the bot should remain private, leave `Public Bot` unchecked in the application's settings; then only you can add it to guilds. If it is public, anyone with the bot URL can add it to guilds where they have sufficient permissions.

### Advanced Bot Authorization

 Developers can expand bot authorization. Requesting scopes beyond `bot` and `applications.commands` continues into a full [authorization code grant flow](https://docs.discord.com/developers/topics/oauth2#authorization-code-grant), allowing the app to request the user's access token. If any scope outside `bot` and `applications.commands` is requested, `response_type` is required again. Unless `redirect_uri` is specified, Discord automatically redirects to the first URI registered for the application. The redirect containing the access code also includes `guild_id` and `permissions` querystring parameters. Treat `guild_id` only as a hint about the bot's relationship with a guild. To verify that relationship, consider requiring the Oauth2 code grant in the bot settings. Enabling it requires anyone adding the bot to a server to complete the full OAuth2 [authorization code grant flow](https://docs.discord.com/developers/topics/oauth2#authorization-code-grant). When you retrieve the user's access token, the response also includes details about the guild where the bot was added:

###### Extended Bot Authorization Access Token Example

```
{
  "token_type": "Bearer",
  "guild": {
    "mfa_level": 0,
    "emojis": [],
    "application_id": null,
    "name": "SomeTest",
    "roles": [
      {
        "hoist": false,
        "name": "@everyone",
        "mentionable": false,
        "color": 0,
        "position": 0,
        "id": "290926798626357250",
        "managed": false,
        "permissions": 49794241,
        "permissions_new": "49794241"
      }
    ],
    "afk_timeout": 300,
    "system_channel_id": null,
    "widget_channel_id": null,
    "region": "us-east",
    "default_message_notifications": 1,
    "explicit_content_filter": 0,
    "splash": null,
    "features": [],
    "afk_channel_id": null,
    "widget_enabled": false,
    "verification_level": 0,
    "owner_id": "53908232999183680",
    "id": "2909267986347357250",
    "icon": null,
    "description": null,
    "public_updates_channel_id": null,
    "safety_alerts_channel_id": null,
    "rules_channel_id": null,
    "max_members": 100000,
    "vanity_url_code": null,
    "premium_subscription_count": 0,
    "premium_tier": 0,
    "preferred_locale": "en-US",
    "system_channel_flags": 0,
    "banner": null,
    "max_presences": null,
    "discovery_splash": null,
    "max_video_channel_users": 25
  },
  "access_token": "zMndOe7jFLXGawdlxMOdNvXjjOce5X",
  "scope": "bot",
  "expires_in": 604800,
  "refresh_token": "mgp8qnvBwJcmadwgCYKyYD5CAzGAX4"
}

```

### Two-Factor Authentication Requirement

 For bots with [elevated permissions](https://docs.discord.com/developers/topics/permissions#permissions-bitwise-permission-flags), meaning permissions marked with `*`, Discord requires two-factor authentication on the owner's account when the bot is added to guilds with server-wide 2FA enabled.

### Webhooks

 Discord's webhook flow is a specialized form of an [authorization code](https://docs.discord.com/developers/topics/oauth2#authorization-code-grant) flow. For this case, set the `scope` querystring parameter to `webhook.incoming`:

###### URL Example

```
https://discord.com/oauth2/authorize?response_type=code&client_id=157730590492196864&scope=webhook.incoming&state=15773059ghq9183habn&redirect_uri=https%3A%2F%2Fnicememe.website

```
