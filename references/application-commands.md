# Application Commands (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Application commands are Discord-client-native entry points for interacting with apps. Discord exposes 3 command types across different UI surfaces: chat input, a message context menu opened from the top-right menu or right-click, and a user context menu opened by right-clicking a user. (Figure: Client interfaces showing the different types of application commands)

### Application Command Object

###### Application Command Naming

 Names for `CHAT_INPUT` commands and command options must match `^[-_'\p{L}\p{N}\p{sc=Deva}\p{sc=Thai}]{1,32}$` with the unicode flag enabled. If a letter has a lowercase form, the lowercase form is required. Characters without lowercase variants, and uncased letters, are allowed. `USER` and `MESSAGE` command names may use mixed case and spaces.

###### Application Command Structure

| Field | Type | Description | Valid Types |
| --- | --- | --- | --- |
| id | snowflake | Unique ID of command | all |
| type? | one of command types | Type of command, defaults to `1` | all |
| application_id | snowflake | ID of the parent application | all |
| guild_id? | snowflake | Command guild ID when the command is guild-scoped rather than global | all |
| name | string | Name of command, 1-32 characters | all |
| name_localizations? | ?dictionary keyed by available locales | Localization dictionary for `name`; values use the same restrictions as `name` | all |
| description | string | `CHAT_INPUT` command description, 1-100 characters. For `USER` and `MESSAGE`, this is an empty string | all |
| description_localizations? | ?dictionary keyed by available locales | Localization dictionary for `description`; values follow the same restrictions as `description` | all |
| options? * | array of command options | Command parameters, maximum 25 | CHAT_INPUT |
| default_member_permissions | ?string | Permission set encoded as a bit set | all |
| dm_permission? | boolean | **Deprecated (use `contexts` instead)**; for global commands only, states whether the command is available in DMs with the app. Commands are visible by default. | all |
| default_permission? | ?boolean | Deprecated soon and not recommended. Reports the command's default enabled state after the app joins a guild; the default value is `true` | all |
| nsfw? | boolean | Whether the command is age-restricted; defaults to `false` | all |
| integration_types? | list of integration types | Installation contexts where this globally-scoped command is available. Defaults to the app's configured contexts | all |
| contexts? | ?list of interaction context types | Interaction context or contexts where this globally-scoped command can run. | all |
| version | snowflake | Autoincrementing version identifier updated during substantial record changes | all |
| handler? | one of command handler types | Determines whether the app's interactions handler handles the interaction or Discord handles it | PRIMARY_ENTRY_POINT |

 * `options` is valid only on application commands of type `CHAT_INPUT`. * `handler` is valid only on `PRIMARY_ENTRY_POINT` application commands for applications with the `EMBEDDED` flag, meaning applications with an Activity. `default_permission` will soon be deprecated. To disable a command for everyone except admins by default, set `default_member_permissions` to `"0"`; to disable globally-scoped commands inside DMs with your app, use `contexts`.

###### Application Command Types

| Name | Type | Description |
| --- | --- | --- |
| CHAT_INPUT | 1 | Slash commands; text commands displayed when a user types `/` |
| USER | 2 | UI command shown when a user is right-clicked or tapped |
| MESSAGE | 3 | UI command shown when a message is right-clicked or tapped |
| PRIMARY_ENTRY_POINT | 4 | UI command representing the primary invocation path for an app's Activity |

###### Application Command Option Structure

 List required `options` before optional ones.
| Field | Type | Description | Valid Option Types |
| --- | --- | --- | --- |
| type | one of application command option type | Type of option | all |
| name * | string | 1-32 character name | all |
| name_localizations? | ?dictionary keyed by available locales | Localization dictionary for `name`; values use the same restrictions as `name` | all |
| description | string | 1-100 character description | all |
| description_localizations? | ?dictionary keyed by available locales | Localization dictionary for `description`; values use the same restrictions as `description` | all |
| required? | boolean | Whether the option is required; defaults to `false` | all but `SUB_COMMAND` and `SUB_COMMAND_GROUP` |
| choices? | array of application command option choice | Selectable user choices, maximum 25 | `STRING`, `INTEGER`, `NUMBER` |
| options? | array of application command option | For subcommand or subcommand group options, nested options hold the parameters or subcommands respectively; maximum 25 | `SUB_COMMAND` , `SUB_COMMAND_GROUP` |
| channel_types? | array of channel types | Restricts displayed channels to these channel types | `CHANNEL` |
| min_value? | integer for `INTEGER` options, double for `NUMBER` options | The minimum value permitted | `INTEGER` , `NUMBER` |
| max_value? | integer for `INTEGER` options, double for `NUMBER` options | The maximum value permitted | `INTEGER` , `NUMBER` |
| min_length? | integer | Minimum allowed string length; minimum `0`, maximum `6000` | `STRING` |
| max_length? | integer | Maximum allowed string length; minimum `1`, maximum `6000` | `STRING` |
| autocomplete? ** | boolean | Whether this option accepts autocomplete interactions | `STRING`, `INTEGER`, `NUMBER` |

 * `name` must be unique inside an array of [application command options](https://docs.discord.com/developers/interactions/application-commands#application-command-object-application-command-option-structure). ** `autocomplete` cannot be true when `choices` are present. Options with `autocomplete` are not limited to choices supplied by the application.

###### Application Command Option Type

| Name | Value | Note |
| --- | --- | --- |
| SUB_COMMAND | 1 |  |
| SUB_COMMAND_GROUP | 2 |  |
| STRING | 3 |  |
| INTEGER | 4 | Any integer between -2^53+1 and 2^53-1 |
| BOOLEAN | 5 |  |
| USER | 6 |  |
| CHANNEL | 7 | Includes all channel types + categories |
| ROLE | 8 |  |
| MENTIONABLE | 9 | Includes users and roles |
| NUMBER | 10 | Any double between -2^53 and 2^53 |
| ATTACHMENT | 11 | attachment object |

###### Application Command Option Choice Structure

 When an option defines `choices`, users may select only those values.
| Field | Type | Description |
| --- | --- | --- |
| name | string | 1-100 character choice name |
| name_localizations? | ?dictionary keyed by available locales | Localization dictionary for `name`; values use the same restrictions as `name` |
| value | string, integer, or double * | Choice value; for strings, maximum 100 characters |

 * The [option type](https://docs.discord.com/developers/interactions/application-commands#application-command-object-application-command-option-type) that owns the choice determines the type of `value`.

###### Entry Point Command Handler Types

| Name | Value | Note |
| --- | --- | --- |
| APP_HANDLER | 1 | The app processes the interaction with an interaction token |
| DISCORD_LAUNCH_ACTIVITY | 2 | Discord launches an Activity for the interaction and sends the follow-up message itself, without coordinating with the app |

 See [Entry Point handlers](https://docs.discord.com/developers/interactions/application-commands#entry-point-handlers) for details about Entry Point command handler types.

### Authorizing Your Application

 Application commands use the [interactions](https://docs.discord.com/developers/interactions/receiving-and-responding) model and do not require a bot user in the guild. To create guild commands, authorize the app with `applications.commands`. This scope can stand alone and is automatically included with `bot`. Requesting it uses a shortcut OAuth2 flow similar to adding a bot; you do not need to finish an exchange for a token. If commands do not need a bot user in the guild, **do not add the bot scope or a permission bitfield to the URL**.

### Registering a Command

 Commands are registered only through HTTP endpoints. They may be global or scoped to a guild. Global commands are available in every guild that adds the app, and an app's global commands are also available in DMs when the app has a bot sharing a mutual guild with the user. Guild commands apply only to the specified guild and do not appear in DMs. Command names are unique per application, per type, and per scope, meaning:
- An app **cannot** define two global `CHAT_INPUT` commands with the same name
- An app **cannot** define two guild `CHAT_INPUT` commands with the same name **in the same guild**
- An app **cannot** define two global `USER` commands with the same name
- An app **can** define both global and guild `CHAT_INPUT` commands with the same name
- An app **can** define a global `CHAT_INPUT` command and a global `USER` command with the same name
- An app **cannot** define a `PRIMARY_ENTRY_POINT` guild command
- Different apps **can** use the same command names
 This is not a complete list. As a rule, command names must be unique for one application by type and by global or guild scope. Apps may have the following command counts:
- 100 global `CHAT_INPUT` commands
- 15 global `USER` commands
- 15 global `MESSAGE` commands
- 1 global `PRIMARY_ENTRY_POINT` command
 For every command type except `PRIMARY_ENTRY_POINT`, the same number of guild-specific commands is allowed per guild. Creating application commands has a global rate limit of 200 creates per day, per guild.

### Making a Global Command

 Global commands are available in *all* guilds for the app. They also have read-repair behavior: after a global command is updated, if a user invokes an old version before their client has refreshed, Discord performs an internal version check, rejects that invocation, and causes the command to reload. To create a **global** command, send an HTTP POST like this:

```
import requests

url = "https://discord.com/api/v10/applications/<my_application_id>/commands"

# This is an example CHAT_INPUT or Slash Command, with a type of 1
json = {
    "name": "blep",
    "type": 1,
    "description": "Send a random adorable animal photo",
    "options": [
        {
            "name": "animal",
            "description": "The type of animal",
            "type": 3,
            "required": True,
            "choices": [
                {
                    "name": "Dog",
                    "value": "animal_dog"
                },
                {
                    "name": "Cat",
                    "value": "animal_cat"
                },
                {
                    "name": "Penguin",
                    "value": "animal_penguin"
                }
            ]
        },
        {
            "name": "only_smol",
            "description": "Whether to show only baby animals",
            "type": 5,
            "required": False
        }
    ]
}

# For authorization, you can use either your bot token
headers = {
    "Authorization": "Bot <my_bot_token>"
}

# or a client credentials token for your app with the applications.commands.update scope
headers = {
    "Authorization": "Bearer <my_credentials_token>"
}

r = requests.post(url, headers=headers, json=json)

```

### Making a Guild Command

 Guild commands are available only in the guild used during creation. They update **instantly**. Use guild commands for fast testing, and switch to global commands when ready for public use. To create a **guild** command, send a similar HTTP POST scoped with a specific `guild_id`:

```
import requests

url = "https://discord.com/api/v10/applications/<my_application_id>/guilds/<guild_id>/commands"

# This is an example USER command, with a type of 2
json = {
    "name": "High Five",
    "type": 2
}

# For authorization, you can use either your bot token
headers = {
    "Authorization": "Bot <my_bot_token>"
}

# or a client credentials token for your app with the applications.commands.update scope
headers = {
    "Authorization": "Bearer <my_credentials_token>"
}

r = requests.post(url, headers=headers, json=json)

```

### Updating and Deleting a Command

 Update and delete commands with `PATCH` and `DELETE` calls to the command endpoint. The endpoints are:
- `applications/<my_application_id>/commands/<command_id>` for global commands, or
- `applications/<my_application_id>/guilds/<guild_id>/commands/<command_id>` for guild commands
 Because command names are unique within type and scope, `POST` requests for new commands are treated as upserts. **Creating a command with a name already used by your application updates the existing command**. Endpoint and parameter details are in [the endpoints section](https://docs.discord.com/developers/interactions/application-commands#endpoints).

### Contexts

 Application commands have two context-related fields on the [application command object](https://docs.discord.com/developers/interactions/application-commands#application-command-object) that configure when and where the command is usable:
- `integration_types` defines the supported **[installation contexts](https://docs.discord.com/developers/interactions/application-commands#installation-context)** for the command
- `contexts` defines the **[interaction contexts](https://docs.discord.com/developers/interactions/application-commands#interaction-contexts)** where the command may be used
 The sections below describe both context types. Contexts are separate from [command permissions](https://docs.discord.com/developers/interactions/application-commands#permissions) for server-installed apps and do not affect those permissions.

### Installation Context

 The [installation context](https://docs.discord.com/developers/resources/application#installation-context) is where the app is installed: a server, a user, or both. If both are supported, some commands may be intended for only one context, such as a `/profile` command for user installs. Set a command's supported installation context or contexts with [`integration_types` field](https://docs.discord.com/developers/interactions/application-commands#application-command-object-application-command-structure) during create or update, as long as the included contexts are already [supported on the application-level](https://docs.discord.com/developers/resources/application#setting-supported-installation-contexts). A command's `integration_types` can influence which [interaction contexts](https://docs.discord.com/developers/interactions/application-commands#interaction-contexts) show that command.

### Interaction Contexts

 A command's interaction contexts determine where in the Discord client it can be invoked. Configure them with the [`contexts` field](https://docs.discord.com/developers/interactions/application-commands#application-command-object-application-command-structure) during create or update. The three [interaction context types](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-context-types) are `GUILD` (`0`), `BOT_DM` (`1`), and `PRIVATE_CHANNEL` (`2`). `PRIVATE_CHANNEL` only matters for commands installed to a user, where `integration_types` includes `USER_INSTALL`.

### Permissions

 Application command permissions let an app enable or disable commands for up to 100 users, roles, and channels in a guild. Users with sufficient permissions can also edit command permissions in the client. These permissions can only be updated with a [Bearer token](https://docs.discord.com/developers/topics/oauth2#client-credentials-grant); bot-token authentication returns an error. Fetch current permissions with [`GET /applications/{application.id}/guilds/{guild.id}/commands/{command.id}/permissions`](https://docs.discord.com/developers/interactions/application-commands#get-application-command-permissions), which returns a `permissions` array containing IDs and permission types. Update them with [`PUT /applications/{application.id}/guilds/{guild.id}/commands/{command.id}/permissions`](https://docs.discord.com/developers/interactions/application-commands#edit-application-command-permissions). Calling the endpoint requires a Bearer token authorized with [`applications.commands.permissions.update`](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-scopes) from a user with enough permissions. For **the authenticating user**, not the app or bot user, all of these must be true:
- Has [permission to Manage Guild and Manage Roles](https://docs.discord.com/developers/topics/permissions) in the guild where the command is edited
- Can run the command being edited
- Can manage the affected resources, meaning roles, users, and/or channels depending on the [permission types](https://docs.discord.com/developers/interactions/application-commands#application-command-permissions-object-application-command-permission-type)

### Syncing and Unsyncing Permissions

 Users can open the command permissions interface in the client through `Server Settings` > `Integrations`, then selecting `Manage` beside an installed app. The top of that interface allows permissions to be edited for a specific user, role, or channel. By default, those top-level permissions apply to all commands for the app. A permission can also be unsynced and customized for individual commands for finer control. When a command-specific permission no longer matches the top-level permission, users see "Not Synced".

### Application Command Permissions Object

###### Guild Application Command Permissions Structure

 Returned when retrieving permissions for an app's command or commands in a guild.
| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | Command ID, or application ID |
| application_id | snowflake | Application ID that owns the command |
| guild_id | snowflake | ID of the guild |
| permissions | array of application command permissions | Command permissions in the guild, maximum 100 |

 If `id` is the application ID rather than a command ID, the permissions apply to every command without explicit overwrites.

###### Application Command Permissions Structure

 Application command permissions can enable or disable commands for individual users, roles, or channels in a guild.
| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | Role, user, or channel ID; may also be a permission constant |
| type | application command permission type | role (`1`), user (`2`), or channel (`3`) |
| permission | boolean | `true` to allow, `false`, to disallow |

###### Application Command Permissions Constants

 The constants below may be used in the `id` field for command permissions payloads.
| Permission | Value | Type | Description |
| --- | --- | --- | --- |
| `@everyone` | `guild_id` | snowflake | All members in a guild |
| All Channels | `guild_id - 1` | snowflake | All channels in a guild |

###### Application Command Permission Type

| Name | Value |
| --- | --- |
| ROLE | 1 |
| USER | 2 |
| CHANNEL | 3 |

 Application command permissions support detailed command access for both guild and global commands of every type. With the [necessary permissions](https://docs.discord.com/developers/interactions/application-commands#permissions), guild members and apps may grant or block access for individual users and roles, or switch commands on and off for entire channels. As threads [inherit user and role permissions from the parent channel](https://docs.discord.com/developers/topics/threads#permissions), channel command permissions also apply to contained threads. Commands a member cannot use are hidden from the command picker. Members with Administrator permission can use every command.

###### Using Default Permissions

 Default permissions can be set while creating a command with `default_member_permissions` and `context`, without a Bearer token because the setting is part of command creation and does not target particular roles, users, or channels. `default_member_permissions` defines the permissions a user needs to use the command. The value for `default_member_permissions` is a string containing the bitwise OR of [permissions](https://docs.discord.com/developers/topics/permissions#permissions-bitwise-permission-flags). Setting it to `"0"` blocks everyone in a guild from using the command unless there is a specific overwrite or the user has admin permissions. For global commands, include `BOT_DM` (`1`) in `contexts` when setting [interaction contexts](https://docs.discord.com/developers/interactions/application-commands#interaction-contexts) to control whether the command runs in DMs with the app. Guild commands do not support `BOT_DM`.

###### Example of editing permissions

 For example, this command would be unavailable to everyone except admins in every guild by default:

```
{
    "name": "permissions_test",
    "description": "A test of default permissions",
    "type": 1,
    "default_member_permissions": "0"
}

```

 This version would allow only users with `MANAGE_GUILD` to use it:

```
permissions = str(1 << 5)

command = {
    "name": "permissions_test",
    "description": "A test of default permissions",
    "type": 1,
    "default_member_permissions": permissions
}

```

 This example disables a command in one specific channel:

```
A_SPECIFIC_CHANNEL = "<channel_id>"
url = "https://discord.com/api/v10/applications/<application_id>/guilds/<my_guild_id>/commands/<my_command_id>/permissions"

json = {
    "permissions": [
        {
            "id": A_SPECIFIC_CHANNEL,
            "type": 3,
            "permission": False
        }
    ]
}

headers = {
    "Authorization": "Bearer <my_bearer_token>"
}

r = requests.put(url, headers=headers, json=json)

```

### Slash Commands

 Slash commands, the `CHAT_INPUT` type, are application commands made from a name, description, and `options` block, similar to function arguments. The name and description help users discover the command, while `options` validate input during command composition. Slash commands can also use groups and subcommands for organization. Each command has an 8000 character maximum across combined name, description, and value properties for the command, its options including subcommands and groups, and choices. When [localization fields](https://docs.discord.com/developers/interactions/application-commands#localization) are present, the size calculation uses only the longest localization for each field, including the default value.
