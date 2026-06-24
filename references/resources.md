# Application, Application Role Connection Metadata

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: Application Object; Application Role Connection Metadata Object.

## Application Object

[Applications](https://docs.discord.com/developers/quick-start/overview-of-apps), or apps, group Discord developer platform features and can be installed to Discord servers, user accounts, or both.

### Application Object

###### Application Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | ID of the app |
| name | string | Name of the app |
| icon | ?string | Icon hash of the app |
| description | string | Description of the app |
| rpc_origins? | array of strings | RPC origin URL list, present when RPC is enabled |
| bot_public | boolean | When `false`, guild installation is limited to the app owner |
| bot_require_code_grant | boolean | When `true`, the app's bot joins only after the full OAuth2 code grant flow completes |
| bot? | partial user object | Partial user object for the app's associated bot user |
| terms_of_service_url? | string | URL for the app's Terms of Service |
| privacy_policy_url? | string | URL for the app's Privacy Policy |
| owner? | partial user object | Partial user object for the app owner |
| verify_key | string | Hex-encoded key used to verify interactions and the GameSDK's GetTicket |
| team | ?team object | Team member list when the app is owned by a team |
| guild_id? | snowflake | Guild linked to the app, such as a developer support server |
| guild? | partial guild object | Partial object for the linked guild |
| primary_sku_id? | snowflake | For a game sold on Discord, this is the id of the created "Game SKU" when one exists |
| slug? | string | For a game sold on Discord, this is the URL slug for its store page |
| cover_image? | string | App’s default rich presence invite cover image hash |
| flags? | integer | App’s public flags |
| approximate_guild_count? | integer | Approximate number of guilds where the app has been added |
| approximate_user_install_count? | integer | Approximate number of users who installed the app by authorizing the `application.commands` scope |
| approximate_user_authorization_count? | integer | Approximate number of users with OAuth2 authorizations for the app |
| redirect_uris? | array of strings | Redirect URI array for the app |
| interactions_endpoint_url? | ?string | Interactions endpoint URL for the app |
| role_connections_verification_url? | ?string | Role connection verification URL for the app |
| event_webhooks_url? | ?string | Event webhooks URL where the app receives webhook events |
| event_webhooks_status? | application event webhook status | Webhook event status for the app: `1` (default) is disabled, `2` is enabled, and `3` is disabled by Discord |
| event_webhooks_types? | array of strings | Webhook event types subscribed to by the app |
| tags? | array of strings | Tags that describe the app's content and features. Max of 5 tags. |
| install_params? | install params object | Settings for the app's default in-app authorization link, when enabled |
| integration_types_config? | dictionary keyed by application integration types | Default scopes and permissions for each supported installation context. Each key maps to an integration type configuration object |
| custom_install_url? | string | Default custom authorization URL for the app, when enabled |

###### Example Application Object

```
{
  "bot_public": true,
  "bot_require_code_grant": false,
  "cover_image": "31deabb7e45b6c8ecfef77d2f99c81a5",
  "description": "Test",
  "flags": 8192,
  "flags_new": "8192",
  "guild_id": "290926798626357260",
  "icon": null,
  "id": "172150183260323840",
  "integration_types_config": {
    "0": {
      "oauth2_install_params": {
        "scopes": [
          "applications.commands",
          "bot"
        ],
        "permissions": "2048"
      }
    },
    "1": {
      "oauth2_install_params": {
        "scopes": [
          "applications.commands"
        ],
        "permissions": "0"
      }
    }
  },
  "name": "Baba O-Riley",
  "interactions_endpoint_url": null,
  "role_connections_verification_url": null,
  "event_webhooks_url": null,
  "event_webhooks_status": 1,
  "owner": {
    "avatar": null,
    "discriminator": "1738",
    "flags": 1024,
    "id": "172150183260323840",
    "username": "i own a bot"
  },
  "primary_sku_id": "172150183260323840",
  "slug": "test",
  "team": {
    "name": "A team",
    "icon": "dd9b7dcfdf5351b9c3de0fe167bacbe1",
    "id": "531992624043786253",
    "owner_user_id": "511972282709709995",
    "members": [
      {
        "membership_state": 2,
        "permissions": ["*"],
        "team_id": "531992624043786253",
        "user": {
          "avatar": "d9e261cd35999608eb7e3de1fae3688b",
          "discriminator": "0001",
          "id": "511972282709709995",
          "username": "Mr Owner"
        }
      }
    ]
  },
  "verify_key": "1e0a356058d627ca38a5c8c9648818061d49e49bd9da9e3ab17d98ad4d6bg2u8"
}

```

###### Application Integration Types

 Defines where an app can be installed, meaning its supported [installation contexts](https://docs.discord.com/developers/resources/application#installation-context).
| Type | ID | Description |
| --- | --- | --- |
| `GUILD_INSTALL` | 0 | App is installable to servers |
| `USER_INSTALL` | 1 | App is installable to users |

###### Application Integration Type Configuration Object

| Field | Type | Description |
| --- | --- | --- |
| oauth2_install_params? | install params object | Install params used by each installation context's default in-app authorization link |

###### Application Event Webhook Status

 Status value showing whether event webhooks are enabled or disabled for an application.
| Name | Value | Description |
| --- | --- | --- |
| `DISABLED` | `1` | Webhook events disabled by the developer |
| `ENABLED` | `2` | Webhook events enabled by the developer |
| `DISABLED_BY_DISCORD` | `3` | Webhook events disabled by Discord, commonly because of inactivity |

###### Application Flags

 The `flags` field is serialized as a number, but it will not exceed 31 bits. Any new flag bits beyond bit 30 appear only in `flags_new`, a string-serialized integer that contains the complete flag bit set. Existing integrations that read `flags` keep working. `flags_new` is only for response serialization; requests that accept flag values should continue using `flags`.
| Value | Name | Description |
| --- | --- | --- |
| `1 << 6` | APPLICATION_AUTO_MODERATION_RULE_CREATE_BADGE | Shows whether an app uses the Auto Moderation API |
| `1 << 12` | GATEWAY_PRESENCE | Intent required for bots in **100 or more servers** to receive `presence_update` events |
| `1 << 13` | GATEWAY_PRESENCE_LIMITED | Intent required for bots in fewer than 100 servers to receive `presence_update` events, configured on the **Bot** page in the app's settings |
| `1 << 14` | GATEWAY_GUILD_MEMBERS | Bots in **100 or more servers** need this intent for member events such as `guild_member_add`. See member-related events under `GUILD_MEMBERS` |
| `1 << 15` | GATEWAY_GUILD_MEMBERS_LIMITED | Intent required for bots in fewer than 100 servers to receive member-related events such as `guild_member_add`, configured on the **Bot** page in the app's settings. See member-related events under `GUILD_MEMBERS` |
| `1 << 16` | VERIFICATION_PENDING_GUILD_LIMIT | Shows unusual app growth that blocks verification |
| `1 << 17` | EMBEDDED | Shows whether an app is embedded inside the Discord client; this is currently not publicly available |
| `1 << 18` | GATEWAY_MESSAGE_CONTENT | Bots in **100 or more servers** need this intent before they receive message content |
| `1 << 19` | GATEWAY_MESSAGE_CONTENT_LIMITED | Intent required for bots in fewer than 100 servers to receive message content, configured on the **Bot** page in the app's settings |
| `1 << 23` | APPLICATION_COMMAND_BADGE | Shows whether an app has registered global application commands |

### Install Params Object

###### Install Params Structure

| Field | Type | Description |
| --- | --- | --- |
| scopes | array of strings | Scopes used when adding the application to the server |
| permissions | string | Permissions requested for the bot role |

### Installation Context

 An app's installation context determines whether it is installed to a server, to a user, or to both. That context controls how the app appears and what it can do in Discord. For example, a user-only app cannot act in a server, while a server-only app is not available inside a user's DMs.

#### Server Context

 Apps installed in a server context, or server-installed apps, require authorization from a server member who has [`MANAGE_GUILD`](https://docs.discord.com/developers/topics/permissions#permissions-bitwise-permission-flags). Everyone in the server can see a server-installed app, while controls such as [command permissions](https://docs.discord.com/developers/interactions/application-commands#permissions) determine who can use specific app features and where. During installation, the app receives selected [OAuth2 scopes](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-scopes) and [bot user permissions](https://docs.discord.com/developers/topics/permissions), which define its access to server resources and data.

#### User Context

 Apps installed in a user context, or user-installed apps, are visible *only* to the authorizing user and do not need server-specific permissions. A user-installed app can appear across that user's servers, DMs, and GDMs, but it must respect the user's permissions in the current surface. For example, if a user runs a command for a user-installed app in a server channel where the user cannot send messages, the app cannot [respond to an interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-response-object-interaction-callback-type) with a non-ephemeral message. The [interaction context](https://docs.discord.com/developers/interactions/application-commands#interaction-contexts) documentation explains how installation context affects interactions.

### Setting Supported Installation Contexts

 New apps support guild installation by default. You can change supported contexts in the [app's settings](https://discord.com/developers/applications). On the [**Installation** page](https://discord.com/developers/applications/select/installation), use **Installation Contexts** to select the contexts your app supports. When you add a new context, update existing [commands](https://docs.discord.com/developers/interactions/application-commands#contexts) if those commands should work there; the [Application Command](https://docs.discord.com/developers/interactions/application-commands#contexts) documentation has the details.

### Install Links

 Install links give users a direct way to add your app in Discord. When configured, an "Add App" button appears on the app profile and App Directory page and starts the app installation flow.

### Types of Install Links

 An app can use one of three install link settings: "Discord Provided Link", "Custom URL", or "None". Selecting "None" removes the "Add App" button and makes the app ineligible for the App Directory. Install links are separate from OAuth2 flows such as the [authorization code grant](https://docs.discord.com/developers/topics/oauth2#authorization-code-grant), which you may still need when requesting user-specific [scopes](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-scopes) such as `identify` or `role_connections.write`.

#### Discord Provided Link

 The default Discord Provided Link is a short link that sends users through installation using the app's [configured installation contexts](https://docs.discord.com/developers/resources/application#setting-supported-installation-contexts). If **User Install** and **Guild Install** are both enabled, the user chooses the install target. Discord Provided Links do not include scopes or bot user permissions in the URL. Example:

```
https://discord.com/oauth2/authorize?client_id=1234567895647001626

```

 These links prompt the user for the scopes and bot user permissions set in Default Install Settings. Discord Provided Links are limited to the `application.commands` and `bot` scopes

#### Custom URL

 A Custom URL gives you more control over where users go after clicking "Add App" on the app profile or App Directory page. Custom URLs are flexible, but they are commonly [OAuth2 `/authorize` URLs](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-urls) with explicit scopes, permissions, and an installation context (`integration_type`).

### Configuring an Install Link and Default Install Settings

 Configure the install link in the [app's settings](https://discord.com/developers/applications). On the [**Installation** page](https://discord.com/developers/applications/select/installation), open **Install Link** and select the install link type. Discord Provided Link is recommended for most apps. When "Discord Provided Link" is selected, the **Default Install Settings** section appears on the [**Installation** page](https://discord.com/developers/applications/select/installation).

### Get Current Application

  Returns the [application](https://docs.discord.com/developers/resources/application#application-object) object tied to the requesting bot user.

### Edit Current Application

  Updates properties on the app tied to the requesting bot user. Only supplied properties are changed. On success, returns the updated [application](https://docs.discord.com/developers/resources/application#application-object) object. Every parameter for this endpoint is optional.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| custom_install_url | string | Default custom authorization URL for the app, when enabled |
| description | string | Description of the app |
| role_connections_verification_url | string | Role connection verification URL for the app |
| install_params | install params object | Settings for the app's default in-app authorization link, when enabled |
| integration_types_config | dictionary keyed by application integration types | Default scopes and permissions for each supported installation context. Each key maps to an integration type configuration object |
| flags * | integer | App’s public flags |
| icon | ?image data | Icon for the app |
| cover_image | ?image data | Default rich presence invite cover image for the app |
| interactions_endpoint_url ** | string | Interactions endpoint URL for the app |
| tags | array of strings | Tags that describe the app's content and features, with a max of 20 characters per tag. Max of 5 tags. |
| event_webhooks_url | string | Event webhooks URL where the app receives webhook events |
| event_webhooks_status | application event webhook status | Webhook event setting for the app: `1` disables events and `2` enables them |
| event_webhooks_types | array of strings | Webhook event types to subscribe to |

 * The API can update only limited intent flags (`GATEWAY_PRESENCE_LIMITED`, `GATEWAY_GUILD_MEMBERS_LIMITED`, and `GATEWAY_MESSAGE_CONTENT_LIMITED`). ** Updating an Interactions endpoint URL through the API requires a URL that is valid under the [Receiving an Interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#receiving-an-interaction) rules.

### Get Application Activity Instance

  Returns a serialized activity instance when one exists. This can help with [preventing unwanted activity sessions](https://docs.discord.com/developers/activities/development-guides/multiplayer-experience#preventing-unwanted-activity-sessions).

###### Example Activity Instance

```
{
  "application_id": "1215413995645968394",
  "instance_id": "i-1276580072400224306-gc-912952092627435520-912954213460484116",
  "launch_id": "1276580072400224306",
  "location": {
    "id": "gc-912952092627435520-912954213460484116",
    "kind": "gc",
    "channel_id": "912954213460484116",
    "guild_id": "912952092627435520"
  },
  "users": ["205519959982473217"],
}

```

###### Activity Instance Object

| Field | Type | Description |
| --- | --- | --- |
| application_id | snowflake | Application ID |
| instance_id | string | Activity Instance ID |
| launch_id | snowflake | Unique identifier for the launch |
| location | Activity Location | Location where the instance is running |
| users | array of snowflakes, user IDs | IDs for Users currently connected to the instance |

###### Activity Location Object

 The Activity Location object describes where an activity instance is running.
| Field | Type | Description |
| --- | --- | --- |
| id | string | Unique identifier for the location |
| kind | Activity Location Kind Enum | Enum describing kind of location |
| channel_id | snowflake | ID of the Channel |
| guild_id? | ?snowflake | ID of the Guild |

###### Activity Location Kind Enum

| Enum | Description |
| --- | --- |
| ’gc’ | Location is a Guild Channel |
| ’pc’ | Location is a Private Channel, such as a DM or GDM |

## Application Role Connection Metadata Object

Application role connection metadata represents metadata for an [application](https://docs.discord.com/developers/resources/application). When a guild adds a bot whose [`role_connections_verification_url`](https://docs.discord.com/developers/resources/application#application-object) is configured in the developer portal, that application can appear as a verification method in the guild's role verification configuration. If the application has role connection metadata configured, that metadata appears when the application is added as a verification method for the role. When a user connects an account through the bot's [`role_connections_verification_url`](https://docs.discord.com/developers/resources/application#application-object), the bot can [update a user's role connection with metadata](https://docs.discord.com/developers/resources/user#update-current-user-application-role-connection) by using the OAuth2 `role_connections.write` scope.

### Application Role Connection Metadata Object

###### Application Role Connection Metadata Structure

| Field | Type | Description |
| --- | --- | --- |
| type | ApplicationRoleConnectionMetadataType | type of metadata value |
| key | string | dictionary key for the metadata field; allowed characters are `a-z`, `0-9`, or `_`, with 1-50 characters |
| name | string | metadata field name, with 1-100 characters |
| name_localizations? | dictionary keyed by available locales | translations of the name |
| description | string | metadata field description, with 1-200 characters |
| description_localizations? | dictionary keyed by available locales | translations of the description |

###### Application Role Connection Metadata Type

| Type | Value | Description |
| --- | --- | --- |
| INTEGER_LESS_THAN_OR_EQUAL | 1 | compares whether the metadata value (`integer`) is at most the guild's configured value (`integer`) |
| INTEGER_GREATER_THAN_OR_EQUAL | 2 | compares whether the metadata value (`integer`) is at least the guild's configured value (`integer`) |
| INTEGER_EQUAL | 3 | the metadata value (`integer`) equals the guild's configured value (`integer`) |
| INTEGER_NOT_EQUAL | 4 | the metadata value (`integer`) differs from the guild's configured value (`integer`) |
| DATETIME_LESS_THAN_OR_EQUAL | 5 | checks whether the metadata value (`ISO8601 string`) is no later than the guild's configured value (`integer`; `days before current date`) |
| DATETIME_GREATER_THAN_OR_EQUAL | 6 | checks whether the metadata value (`ISO8601 string`) is no earlier than the guild's configured value (`integer`; `days before current date`) |
| BOOLEAN_EQUAL | 7 | the metadata value (`integer`) equals the guild's configured value (`integer`; `1`) |
| BOOLEAN_NOT_EQUAL | 8 | the metadata value (`integer`) differs from the guild's configured value (`integer`; `1`) |

 Each metadata type defines a comparison operation that guilds can use when setting role requirements from metadata values stored by the bot. Bots store a `metadata value` per user, and guilds set the required `guild's configured value` in the guild role settings.

### Get Application Role Connection Metadata Records

 GET/applications/[{application.id}](https://docs.discord.com/developers/resources/application#application-object)/role-connections/metadata retrieves that application's [application role connection metadata](https://docs.discord.com/developers/resources/application-role-connection-metadata#application-role-connection-metadata-object) object list.

### Update Application Role Connection Metadata Records

 PUT/applications/[{application.id}](https://docs.discord.com/developers/resources/application#application-object)/role-connections/metadata replaces the configured [application role connection metadata](https://docs.discord.com/developers/resources/application-role-connection-metadata#application-role-connection-metadata-object) object list and returns the updated set. Each application supports up to 5 metadata records.
