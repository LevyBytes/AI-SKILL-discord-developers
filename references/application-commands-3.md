# Application Commands (part 3)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

```
{
  "name": "birthday",
  "type": 1,
  "description": "Wish a friend a happy birthday",
  "name_localized": "生日",
  "description_localized": "祝你朋友生日快乐",
  "options": [
    {
      "name": "age",
      "type": 4,
      "description": "Your friend's age",
      "name_localized": "岁数",
      "description_localized": "你朋友的岁数",
    }
  ]
}

```

 When a requester's locale has no entry in a localization dictionary, that field's matching `name_localization` or `description_localization` is omitted as well. Discord resolves the locale from `X-Discord-Locale`, then falls back to `Accept-Language`, and finally uses the user's configured locale.

### Age-Restricted Commands

 Commands that expose age-restricted material should set the [`nsfw` field](https://docs.discord.com/developers/interactions/application-commands#application-command-object-application-command-structure) to `true` during create or update. An age-restricted command is only visible and usable for eligible users and channel contexts. Apps with [discovery enabled](https://support-dev.discord.com/hc/en-us/articles/9489299950487), which is required for App Directory listing, must not include age-restricted commands or content.

### Using Age-Restricted Commands

 A user can run an age-restricted command only when they are at least 18 years old and invoke it from one of these places:
- an [age-restricted channel](https://support.discord.com/hc/articles/115000084051-Age-Restricted-Channels-and-Content), or
- a DM with the app, after turning on [enabling age-restricted commands](https://support.discord.com/hc/en-us/articles/10123937946007) in User Settings.
 The Help Center article on [enabling age-restricted commands](https://support.discord.com/hc/en-us/articles/10123937946007) covers access and usage details.

### Endpoints

 Authorization for every endpoint accepts either the application's [bot token](https://docs.discord.com/developers/reference#authentication) or a [client credentials token](https://docs.discord.com/developers/topics/oauth2#client-credentials-grant).

### Get Global Application Commands

  When localization is enabled, this endpoint's response objects can include [additional fields if localization is active](https://docs.discord.com/developers/interactions/application-commands#retrieving-localized-commands). Use it to list every global command for the application; the response is an array of [application command](https://docs.discord.com/developers/interactions/application-commands#application-command-object) objects.

###### Query String Params

| Field | Type | Description |
| --- | --- | --- |
| with_localizations? | boolean | Controls whether response objects include complete localization dictionaries (`name_localizations` and `description_localizations`) rather than `name_localized` and `description_localized`. Defaults to `false`. |

### Create Global Application Command

  Creating an application command with a name already used by that application replaces the prior command. This endpoint creates a global command and returns `201` when no matching name existed, or `200` when it overwrites the existing command. In both cases, the body contains an [application command](https://docs.discord.com/developers/interactions/application-commands#application-command-object) object.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name | string | Name of command, 1-32 characters |
| name_localizations? | ?dictionary keyed by available locales | Localization entries for `name`; each value must satisfy the restrictions that apply to `name`. |
| description? | string | 1-100 character description for `CHAT_INPUT` commands |
| description_localizations? | ?dictionary keyed by available locales | Localization entries for `description`; each value must satisfy the restrictions that apply to `description`. |
| options? | array of application command option | Command parameters, limited to 25, and valid only on `CHAT_INPUT` commands. |
| default_member_permissions? | ?string | Permissions encoded as a bit set. |
| dm_permission? | ?boolean | **Deprecated (use `contexts` instead)**; controls DM availability with the app for globally-scoped commands only. Commands are visible by default. |
| default_permission? | boolean | Superseded by `default_member_permissions` and planned for future deprecation. Specifies whether the command starts enabled by default when the app is installed in a guild. Default is `true`. |
| integration_types? | list of integration types | Installation context(s) that expose the command. |
| contexts? | list of interaction context types | Interaction context(s) allowed to invoke the command. |
| type? | one of application command type | Command type; uses `1` when omitted. |
| nsfw? | boolean | Whether this command is marked age-restricted. |
| handler? | one of entry point command handler types | Selects whether Discord or the app's interactions handler processes the interaction. Applies only to `PRIMARY_ENTRY_POINT` commands. |

### Get Global Application Command

  Retrieve one global command for the application. The response body is an [application command](https://docs.discord.com/developers/interactions/application-commands#application-command-object) object.

### Edit Global Application Command

  Every parameter for this endpoint is optional. It edits a global command and returns `200` with an [application command](https://docs.discord.com/developers/interactions/application-commands#application-command-object) object. Supplying a field fully replaces the command's current value for that field.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name? | string | Name of command, 1-32 characters |
| name_localizations? | ?dictionary keyed by available locales | Localization entries for `name`; each value must satisfy the restrictions that apply to `name`. |
| description? | string | 1-100 character description |
| description_localizations? | ?dictionary keyed by available locales | Localization entries for `description`; each value must satisfy the restrictions that apply to `description`. |
| options? | array of application command option | Command parameters, limited to 25, and valid only on `CHAT_INPUT` commands. |
| default_member_permissions? | ?string | Permissions encoded as a bit set. |
| dm_permission? | ?boolean | **Deprecated (use `contexts` instead)**; controls DM availability with the app for globally-scoped commands only. Commands are visible by default. |
| default_permission? | boolean | Superseded by `default_member_permissions` and planned for future deprecation. Specifies whether the command starts enabled by default when the app is installed in a guild. Default is `true`. |
| integration_types? | list of integration types | Installation context(s) that expose the command. |
| contexts? | list of interaction context types | Interaction context(s) allowed to invoke the command. |
| nsfw? | boolean | Whether this command is marked age-restricted. |
| handler? | one of entry point command handler types | Selects whether Discord or the app's interactions handler processes the interaction. Applies only to `PRIMARY_ENTRY_POINT` commands. |

### Delete Global Application Command

  Remove a global command. A successful deletion returns `204 No Content`.

### Bulk Overwrite Global Application Commands

  Submit a list of application commands to replace the application's current global command list. The endpoint returns `200` with a list of [application command](https://docs.discord.com/developers/interactions/application-commands#application-command-object) objects. Commands not already present consume daily application command creation quota. The replacement covers **all** command types: slash commands, user commands, and message commands.

### Get Guild Application Commands

  When localization is enabled, this endpoint's response objects can include [additional fields if localization is active](https://docs.discord.com/developers/interactions/application-commands#retrieving-localized-commands). Use it to list every guild command for the application in a specific guild; the response is an array of [application command](https://docs.discord.com/developers/interactions/application-commands#application-command-object) objects.

###### Query String Params

| Field | Type | Description |
| --- | --- | --- |
| with_localizations? | boolean | Controls whether response objects include complete localization dictionaries (`name_localizations` and `description_localizations`) rather than `name_localized` and `description_localized`. Defaults to `false`. |

### Create Guild Application Command

  Creating a guild command with a name already used by the application replaces the older command. This endpoint creates a guild command, and new guild commands are usable in that guild immediately. It returns `201` when no same-name command existed, or `200` when an existing command is overwritten. Both responses contain an [application command](https://docs.discord.com/developers/interactions/application-commands#application-command-object) object.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name | string | Name of command, 1-32 characters |
| name_localizations? | ?dictionary keyed by available locales | Localization entries for `name`; each value must satisfy the restrictions that apply to `name`. |
| description? | string | 1-100 character description for `CHAT_INPUT` commands |
| description_localizations? | ?dictionary keyed by available locales | Localization entries for `description`; each value must satisfy the restrictions that apply to `description`. |
| options? | array of application command option | Command parameters, limited to 25, and valid only on `CHAT_INPUT` commands. |
| default_member_permissions? | ?string | Permissions encoded as a bit set. |
| default_permission? | boolean | Superseded by `default_member_permissions` and planned for future deprecation. Specifies whether the command starts enabled by default when the app is installed in a guild. Default is `true`. |
| type? | one of application command type | Command type; uses `1` when omitted. |
| nsfw? | boolean | Whether this command is marked age-restricted. |

### Get Guild Application Command

  Retrieve one guild command for the application. The response body is an [application command](https://docs.discord.com/developers/interactions/application-commands#application-command-object) object.

### Edit Guild Application Command

  Every parameter for this endpoint is optional. It edits a guild command, with changes becoming available immediately, and returns `200` plus an [application command](https://docs.discord.com/developers/interactions/application-commands#application-command-object) object. Any supplied field fully replaces that field's existing value.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name? | string | Name of command, 1-32 characters |
| name_localizations? | ?dictionary keyed by available locales | Localization entries for `name`; each value must satisfy the restrictions that apply to `name`. |
| description? | string | 1-100 character description |
| description_localizations? | ?dictionary keyed by available locales | Localization entries for `description`; each value must satisfy the restrictions that apply to `description`. |
| options? | array of application command option | Command parameters, limited to 25, and valid only on `CHAT_INPUT` commands. |
| default_member_permissions? | ?string | Permissions encoded as a bit set. |
| default_permission? | boolean | Superseded by `default_member_permissions` and planned for future deprecation. Specifies whether the command starts enabled by default when the app is installed in a guild. Default is `true`. |
| nsfw? | boolean | Whether this command is marked age-restricted. |

### Delete Guild Application Command

  Remove a guild command. A successful deletion returns `204 No Content`.

### Bulk Overwrite Guild Application Commands

  Submit a list of application commands to replace the application's command list for the selected guild. The endpoint returns `200` and a list of [application command](https://docs.discord.com/developers/interactions/application-commands#application-command-object) objects. This replacement covers **all** command types: slash commands, user commands, and message commands.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| id? | snowflake | Command ID, when available. |
| name | string | Name of command, 1-32 characters |
| name_localizations? | ?dictionary keyed by available locales | Localization entries for `name`; each value must satisfy the restrictions that apply to `name`. |
| description | string | 1-100 character description |
| description_localizations? | ?dictionary keyed by available locales | Localization entries for `description`; each value must satisfy the restrictions that apply to `description`. |
| options? | array of application command option | Parameters for the command |
| default_member_permissions? | ?string | Permissions encoded as a bit set. |
| dm_permission? | ?boolean | **Deprecated (use `contexts` instead)**; controls DM availability with the app for globally-scoped commands only. Commands are visible by default. |
| default_permission? | boolean | Superseded by `default_member_permissions` and planned for future deprecation. Specifies whether the command starts enabled by default when the app is installed in a guild. Default is `true`. |
| integration_types | list of integration types | Installation context(s) that expose the command; default is `GUILD_INSTALL` (`[0]`). |
| contexts | list of interaction context types | Interaction context(s) where the command is usable; defaults to every context `[0,1,2]`. |
| type? | one of application command type | Command type; uses `1` when omitted. |

### Get Guild Application Command Permissions

  Retrieve permissions for every application command in a guild. The response is an array of [guild application command permissions](https://docs.discord.com/developers/interactions/application-commands#application-command-permissions-object-guild-application-command-permissions-structure) objects.

### Get Application Command Permissions

  Retrieve permissions for one application command in a guild. The response is a [guild application command permissions](https://docs.discord.com/developers/interactions/application-commands#application-command-permissions-object-guild-application-command-permissions-structure) object.

### Edit Application Command Permissions

  This endpoint replaces the command's current permissions in that guild. It edits permissions for one application command in a guild and returns a [guild application command permissions](https://docs.discord.com/developers/interactions/application-commands#application-command-permissions-object-guild-application-command-permissions-structure) object. The change fires an [Application Command Permissions Update](https://docs.discord.com/developers/events/gateway-events#application-command-permissions-update) Gateway event. A command can have up to 100 permission overwrites. Authenticate with a Bearer token that can manage the guild and its roles. See [application command permissions](https://docs.discord.com/developers/interactions/application-commands#permissions) for more detail. If a command is renamed or deleted, all permissions for that command are permanently removed.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| permissions | array of application command permissions | Command permissions within the guild. |

### Batch Edit Application Command Permissions

  This endpoint is unavailable after [updates to command permissions (Permissions v2)](https://docs.discord.com/developers/change-log#updated-command-permissions). Use [edit each application command permissions](https://docs.discord.com/developers/interactions/application-commands#edit-application-command-permissions) instead, while accounting for possible [rate limits](https://docs.discord.com/developers/topics/rate-limits).
