# Making a global command
Context: Discord developer reference: Making a global command
[Source](command-definitions-and-registration-index.md) / [Previous source section](registering-a-command.md) / [Following source section](making-a-guild-command.md)
<a id="source-making-a-global-command"></a>

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

