# Application Commands (part 2)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

###### Example Slash Command

```
{
    "name": "blep",
    "type": 1,
    "description": "Send a random adorable animal photo",
    "options": [
        {
            "name": "animal",
            "description": "The type of animal",
            "type": 3,
            "required": true,
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
            "required": false
        }
    ]
}

```

 When a user invokes a slash command, Discord sends your application an interaction:

###### Example Interaction

Slash Command Interaction

Open the example payload for Slash Command interactions

```
{
    "type": 2,
    "token": "A_UNIQUE_TOKEN",
    "member": {
        "user": {
            "id": "53908232506183680",
            "username": "Mason",
            "avatar": "a_d5efa99b3eeaa7dd43acca82f5692432",
            "discriminator": "1337",
            "public_flags": 131141
        },
        "roles": ["539082325061836999"],
        "premium_since": null,
        "permissions": "2147483647",
        "pending": false,
        "nick": null,
        "mute": false,
        "joined_at": "2017-03-13T19:19:14.040000+00:00",
        "is_pending": false,
        "deaf": false
    },
    "id": "786008729715212338",
    "guild_id": "290926798626357999",
    "app_permissions": "442368",
    "guild_locale": "en-US",
    "locale": "en-US",
    "data": {
        "options": [{
            "type": 3,
            "name": "cardname",
            "value": "The Gitrog Monster"
        }],
        "type": 1,
        "name": "cardsearch",
        "id": "771825006014889984"
    },
    "channel_id": "645027906669510667"
}

```

### Subcommands and Subcommand Groups

 At present, the command explorer shows subcommands and subcommand groups at the top level. Discord may later display them as nested autocomplete choices. Use subcommands and groups when a command set needs clearer organization or more complex structure. **Subcommands** split a command by **declaring actions inside a command or group**. **Subcommand Groups** arrange your **subcommands** by **collecting related actions or resources under one command**. These patterns are guidance rather than enforced design rules; you can model subcommands and groups in whatever way fits your app. Adding subcommands or subcommand groups makes the base command itself unavailable. For example, `/permissions` is not a valid command invocation once `/permissions add | remove` exists as subcommands or subcommand groups. Nesting is limited to one level inside a group: a top-level command can contain subcommand groups, and each group can contain subcommands. **No other nesting pattern is supported.** The following visuals illustrate this:

```
VALID

command
|
|__ subcommand
|
|__ subcommand

----

VALID

command
|
|__ subcommand-group
    |
    |__ subcommand
|
|__ subcommand-group
    |
    |__ subcommand

----

VALID

command
|
|__ subcommand-group
    |
    |__ subcommand
|
|__ subcommand

-------

INVALID

command
|
|__ subcommand-group
    |
    |__ subcommand-group
|
|__ subcommand-group
    |
    |__ subcommand-group

----

INVALID

command
|
|__ subcommand
    |
    |__ subcommand-group
|
|__ subcommand
    |
    |__ subcommand-group

```

### Example Walkthrough

 Consider this scenario: you operate a moderation bot and want a `/permissions` command that can perform these actions:
- Retrieve guild permissions for a user or a role
- Retrieve permissions for a user or a role in one specific channel
- Modify guild permissions for a user or a role
- Modify permissions for a user or a role in one specific channel
 Begin with the top-level definition for `/permissions`:

```
{
    "name": "permissions",
    "description": "Get or edit permissions for a user or a role",
    "options": []
}

```

 (Figure: A command with no arguments. It says /permissions) The command is now named `permissions`. Because it needs to operate on both users and roles, subcommand groups let one command cover both resources instead of requiring two separate commands. In this design, the groups represent the resource being acted on: `user` or `role`.

```
{
    "name": "permissions",
    "description": "Get or edit permissions for a user or a role",
    "options": [
        {
            "name": "user",
            "description": "Get or edit permissions for a user",
            "type": 2 // 2 is type SUB_COMMAND_GROUP
        },
        {
            "name": "role",
            "description": "Get or edit permissions for a role",
            "type": 2
        }
    ]
}

```

 A command in this state **will not appear** in the command explorer, because the groups behave like command folders and both folders are still empty. Continue by adding the `get` and `edit` actions. Since `user` and `role` now act as the folders, define `get` and `edit` subcommands inside each subcommand group:

```
{
    "name": "permissions",
    "description": "Get or edit permissions for a user or a role",
    "options": [
        {
            "name": "user",
            "description": "Get or edit permissions for a user",
            "type": 2, // 2 is type SUB_COMMAND_GROUP
            "options": [
                {
                    "name": "get",
                    "description": "Get permissions for a user",
                    "type": 1 // 1 is type SUB_COMMAND
                },
                {
                    "name": "edit",
                    "description": "Edit permissions for a user",
                    "type": 1
                }
            ]
        },
        {
            "name": "role",
            "description": "Get or edit permissions for a role",
            "type": 2,
            "options": [
                {
                    "name": "get",
                    "description": "Get permissions for a role",
                    "type": 1
                },
                {
                    "name": "edit",
                    "description": "Edit permissions for a role",
                    "type": 1
                }
            ]
        }
    ]
}

```

 (Figure: A command with grouped subcommands. It says /permissions user get) Next, add arguments. Choosing `user` should allow user selection, and choosing `role` should allow role selection. The command also needs to distinguish guild-wide permissions from channel-specific permissions, which can be represented with optional arguments:

```
{
    "name": "permissions",
    "description": "Get or edit permissions for a user or a role",
    "options": [
        {
            "name": "user",
            "description": "Get or edit permissions for a user",
            "type": 2, // 2 is type SUB_COMMAND_GROUP
            "options": [
                {
                    "name": "get",
                    "description": "Get permissions for a user",
                    "type": 1, // 1 is type SUB_COMMAND
                    "options": [
                        {
                            "name": "user",
                            "description": "The user to get",
                            "type": 6, // 6 is type USER
                            "required": true
                        },
                        {
                            "name": "channel",
                            "description": "The channel permissions to get. If omitted, the guild permissions will be returned",
                            "type": 7, // 7 is type CHANNEL
                            "required": false
                        }
                    ]
                },
                {
                    "name": "edit",
                    "description": "Edit permissions for a user",
                    "type": 1,
                    "options": [
                        {
                            "name": "user",
                            "description": "The user to edit",
                            "type": 6,
                            "required": true
                        },
                        {
                            "name": "channel",
                            "description": "The channel permissions to edit. If omitted, the guild permissions will be edited",
                            "type": 7,
                            "required": false
                        }
                    ]
                }
            ]
        },
        {
            "name": "role",
            "description": "Get or edit permissions for a role",
            "type": 2,
            "options": [
                {
                    "name": "get",
                    "description": "Get permissions for a role",
                    "type": 1,
                    "options": [
                        {
                            "name": "role",
                            "description": "The role to get",
                            "type": 8, // 8 is type ROLE
                            "required": true
                        },
                        {
                            "name": "channel",
                            "description": "The channel permissions to get. If omitted, the guild permissions will be returned",
                            "type": 7,
                            "required": false
                        }
                    ]
                },
                {
                    "name": "edit",
                    "description": "Edit permissions for a role",
                    "type": 1,
                    "options": [
                        {
                            "name": "role",
                            "description": "The role to edit",
                            "type": 8,
                            "required": true
                        },
                        {
                            "name": "channel",
                            "description": "The channel permissions to edit. If omitted, the guild permissions will be edited",
                            "type": 7,
                            "required": false
                        }
                    ]
                }
            ]
        }
    ]
}

```

 That completes the model. The JSON is dense, but the result is one command that narrows first by action, then by resource, then by optional arguments. Here is the assembled command. (Figure: A command with grouped subcommands and parameters. It says /permissions user get with arguments for a user and a channel.)

### User Commands

 User commands are application commands shown from a user's context menu, reached by right click or tap. They are useful for quick app actions that operate on users. They accept no arguments, and the interaction response includes the user that was clicked or tapped. The invoking user must be allowed to send text messages in the channel where the user command is invoked; otherwise Discord returns a 'Permission Denied' interaction error. Do not include the `description` field when creating user commands. For data model compatibility, fetched commands return `description` as an **empty string** rather than `null`.

###### Example User Command

```
{
    "name": "View Stats",
    "type": 2
}

```

 (Figure: An example user command. The context menu has an Apps section open to a High Five command) When a user command is invoked, your app receives an interaction:

###### Example Interaction

User Command Interaction

Open the example payload for User Command interactions

```
{
    "application_id": "775799577604522054",
    "channel_id": "772908445358620702",
    "data": {
        "id": "866818195033292850",
        "name": "context-menu-user-2",
        "resolved": {
            "members": {
                "809850198683418695": {
                    "avatar": null,
                    "is_pending": false,
                    "joined_at": "2021-02-12T18:25:07.972000+00:00",
                    "nick": null,
                    "pending": false,
                    "permissions": "246997699136",
                    "premium_since": null,
                    "roles": []
                }
            },
            "users": {
                "809850198683418695": {
                    "avatar": "afc428077119df8aabbbd84b0dc90c74",
                    "bot": true,
                    "discriminator": "7302",
                    "id": "809850198683418695",
                    "public_flags": 0,
                    "username": "VoltyDemo"
                }
            }
        },
        "target_id": "809850198683418695",
        "type": 2
    },
    "guild_id": "772904309264089089",
    "guild_locale": "en-US",
    "app_permissions": "442368",
    "id": "867794291820986368",
    "locale": "en-US",
    "member": {
        "avatar": null,
        "deaf": false,
        "is_pending": false,
        "joined_at": "2020-11-02T20:46:57.364000+00:00",
        "mute": false,
        "nick": null,
        "pending": false,
        "permissions": "274877906943",
        "premium_since": null,
        "roles": ["785609923542777878"],
        "user": {
            "avatar": "a_f03401914fb4f3caa9037578ab980920",
            "discriminator": "6538",
            "id": "167348773423415296",
            "public_flags": 1,
            "username": "ian"
        }
    },
    "token": "UNIQUE_TOKEN",
    "type": 2,
    "version": 1
}

```

### Message Commands

 Message commands are application commands available from a message context menu, reached by right click or tap. They expose quick app actions that target messages. They take no arguments, and the interaction response includes the message that was clicked or tapped. Do not provide a `description` field when creating message commands. To keep existing data models stable, fetched commands return `description` as an **empty string** rather than `null`.

###### Example Message Command

```
{
    "name": "Challenge",
    "type": 3
}

```

 (Figure: An example message command. The context menu has an Apps section open to a Bookmark command) When a message command is used, your application receives an interaction:

###### Example Interaction

Message Command Interaction

Open the example payload for Message Command interactions

```
{
    "application_id": "775799577604522054",
    "channel_id": "772908445358620702",
    "data": {
        "id": "866818195033292851",
        "name": "context-menu-message-2",
        "resolved": {
            "messages": {
                "867793854505943041": {
                    "attachments": [],
                    "author": {
                        "avatar": "a_f03401914fb4f3caa9037578ab980920",
                        "discriminator": "6538",
                        "id": "167348773423415296",
                        "public_flags": 1,
                        "username": "ian"
                    },
                    "channel_id": "772908445358620702",
                    "components": [],
                    "content": "some message",
                    "edited_timestamp": null,
                    "embeds": [],
                    "flags": 0,
                    "id": "867793854505943041",
                    "mention_everyone": false,
                    "mention_roles": [],
                    "mentions": [],
                    "pinned": false,
                    "timestamp": "2021-07-22T15:42:57.744000+00:00",
                    "tts": false,
                    "type": 0
                }
            }
        },
        "target_id": "867793854505943041",
        "type": 3
    },
    "guild_id": "772904309264089089",
    "guild_locale": "en-US",
    "app_permissions": "442368",
    "id": "867793873336926249",
    "locale": "en-US",
    "member": {
        "avatar": null,
        "deaf": false,
        "is_pending": false,
        "joined_at": "2020-11-02T20:46:57.364000+00:00",
        "mute": false,
        "nick": null,
        "pending": false,
        "permissions": "274877906943",
        "premium_since": null,
        "roles": ["785609923542777878"],
        "user": {
            "avatar": "a_f03401914fb4f3caa9037578ab980920",
            "discriminator": "6538",
            "id": "167348773423415296",
            "public_flags": 1,
            "username": "ian"
        }
    },
    "token": "UNIQUE_TOKEN",
    "type": 2,
    "version": 1
}

```

### Entry Point Commands

 An Entry Point command is the main way users open an app's [Activity](https://docs.discord.com/developers/activities/overview) through the [App Launcher](https://support.discord.com/hc/articles/21334461140375-Using-Apps-on-Discord#h_01HRQSA6C8TRHS722P1H3HW1TV). Users can see an Entry Point command only when the app has [Activities](https://docs.discord.com/developers/activities/overview) enabled. (Figure: Entry Point command in App Launcher)

###### Example Entry Point Command

```
{
    "name": "launch",
    "description": "Launch Racing with Friends",
    "type": 4,
    "handler": 2
}

```

### Entry Point handlers

 When someone invokes an app's Entry Point command, [`handler`](https://docs.discord.com/developers/interactions/application-commands#application-command-object-application-command-structure) controls how Discord processes the interaction:
- With `APP_HANDLER` (`1`), the application must [respond to the interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#responding-to-an-interaction). It may launch the related Activity by returning the `LAUNCH_ACTIVITY` (type `12`) [interaction callback type](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-response-object-interaction-callback-type), or it may perform another action such as sending a channel follow-up message.
- With `DISCORD_LAUNCH_ACTIVITY` (`2`), Discord processes the interaction for you by opening the linked Activity and posting a message in the launch channel.

### Default Entry Point command

 Enabling Activities automatically creates an Entry Point command named "Launch" for the app, using `DISCORD_LAUNCH_ACTIVITY` (`2`) as the [Entry Point handler](https://docs.discord.com/developers/interactions/application-commands#entry-point-handlers). To inspect that generated command, including its ID, call [Get Global Application Commands](https://docs.discord.com/developers/interactions/application-commands##get-global-application-commands) and find the "Launch" command. The [Setting Up an Entry Point Command guide](https://docs.discord.com/developers/activities/development-guides/user-actions#setting-up-an-entry-point-command) explains how to update or replace the default Entry Point command.

### Autocomplete

 Autocomplete interactions let an application return option suggestions dynamically while a user types. An autocomplete interaction **may include partial data** for option values. Your app receives partial data for current user input when that input passes client-side validation; partial strings may arrive, while invalid numbers will not. The option currently being edited includes `focused: true`; options already completed by the user are also included, but without `focused`. Required options may be absent in this flow if the user has not filled them yet. This validation happens **only on the client side**.

```
{
  "type": 4,
  "data": {
    "id": "816437322781949972",
    "name": "airhorn",
    "type": 1,
    "version": "847194950382780532",
    "options": [
      {
        "type": 3,
        "name": "variant",
        "value": "data a user is typ",
        "focused": true
      }
    ]
  }
}

```

### Localization

 Application commands support localization, allowing command names and descriptions to vary based on the user's selected client language. Localization is optional. You can localize command, subcommand, and option names and descriptions, plus choice names, by sending the relevant `name_localizations` and `description_localizations` fields when creating or updating the application command. A command can be partially localized: it does not need every [available locale](https://docs.discord.com/developers/reference#locales), and each field can support a different locale set. If a field's localization dictionary lacks a user's locale, Discord shows the field's default value. You do not need to duplicate the default value across all locales; localized values identical to the default are ignored. Localized option names have an extra rule: each localized option name must be unique against the command's default option names and against other option names for that locale on the same command. Interaction payloads still use the default names for commands, subcommands, and options. To localize the interaction response, read the client's selected language from `locale` in the interaction payload. A localized application command can look like this:

```
{
  "name": "birthday",
  "type": 1,
  "description": "Wish a friend a happy birthday",
  "name_localizations": {
    "zh-CN": "生日",
    "el": "γενέθλια"
  },
  "description_localizations": {
    "zh-CN": "祝你朋友生日快乐"
  },
  "options": [
    {
      "name": "age",
      "type": 4,
      "description": "Your friend's age",
      "name_localizations": {
        "zh-CN": "岁数"
      },
      "description_localizations": {
        "zh-CN": "你朋友的岁数"
      }
    }
  ]
}

```

### Locale fallbacks

 Application commands include locale fallback behavior when a user's locale is missing from the localizations. If the fallback locale is not present either, Discord uses the default. Include the default value under its matching locale key to avoid an unexpected fallback. For example, if the default value is `en-US` but localizations omit `en-US`, a user with `en-US` selected may see the `en-GB` value when that key exists. In a command whose default name is "color", if only the `en-GB` localization "colour" is provided, users in `en-US` see "colour" because the `en-US` localization key is absent.
| Locale | Fallback |
| --- | --- |
| en-US | en-GB |
| en-GB | en-US |
| es-419 | es-ES |

### Retrieving localized commands

 Most endpoints that return application command objects include `name_localizations` and `description_localizations`, but some omit them unless requested. This applies to `GET` endpoints that list all guild or global commands for an application. Those responses instead include `name_localized` or `description_localized`, containing only the localization for the requester's locale. You can still retrieve the full dictionaries by passing the correct query argument. For instance, if a batch `GET` request uses locale `zh-CN` and includes the command above, the response object would be:
