# Rpc (part 2)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

| Field | Type | Description |
| --- | --- | --- |
| device_id | string | device id |
| volume | float | output voice level (min: 0, max: 200) |
| available_devices | array of objects | array of *read-only* device objects; each object has `id` and `name` string keys |

###### Voice Settings Mode Object

| Field | Type | Description |
| --- | --- | --- |
| type | string | voice settings mode type; allowed values are `PUSH_TO_TALK` and `VOICE_ACTIVITY` |
| auto_threshold | boolean | whether voice activity sets the threshold automatically |
| threshold | float | voice activity threshold, in dB (min: -100, max: 0) |
| shortcut | shortcut key combo object | shortcut key combos for PTT |
| delay | float | PTT release delay, in ms (min: 0, max: 2000) |

###### Shortcut Key Combo Object

| Field | Type | Description |
| --- | --- | --- |
| type | integer | see key types |
| code | integer | key code |
| name | string | key name |

###### Key Types

| Type | Id |
| --- | --- |
| KEYBOARD_KEY | 0 |
| MOUSE_BUTTON | 1 |
| KEYBOARD_MODIFIER_KEY | 2 |
| GAMEPAD_BUTTON | 3 |

###### Example Get Voice Settings Response Payload

```
{
  "cmd": "GET_VOICE_SETTINGS",
  "data": {
    "input": {
      "available_devices": [
        {
          "id": "default",
          "name": "Default"
        },
        {
          "id": "Built-in Microphone",
          "name": "Built-in Microphone"
        }
      ],
      "device_id": "default",
      "volume": 49.803921580314636
    },
    "output": {
      "available_devices": [
        {
          "id": "default",
          "name": "Default"
        },
        {
          "id": "Built-in Output",
          "name": "Built-in Output"
        }
      ],
      "device_id": "default",
      "volume": 93.00000071525574
    },
    "mode": {
      "type": "VOICE_ACTIVITY",
      "auto_threshold": true,
      "threshold": -46.92622950819673,
      "shortcut": [{ "type": 0, "code": 12, "name": "i" }],
      "delay": 98.36065573770492
    },
    "automatic_gain_control": false,
    "echo_cancellation": false,
    "noise_suppression": false,
    "qos": false,
    "silence_warning": false,
    "deaf": false,
    "mute": false
  },
  "nonce": "fa07c532-bb03-4f75-8b9a-397f5109afb6"
}

```

#### SET_VOICE_SETTINGS

 RPC currently allows only one app at a time to modify voice settings. When an app updates voice settings, Discord locks those settings, preventing any other simultaneously connected app from changing them. After the controlling app disconnects, the settings return to their previous values. If an app that already supplied voice settings connects again, the client reapplies that app's settings and locks voice settings again. This limitation is temporary and is planned to change. For voice settings updates, every field is optional, and Discord updates only the fields that are included.

###### Set Voice Settings Argument and Response Structure

| Field | Type | Description |
| --- | --- | --- |
| input | voice settings input object | input settings |
| output | voice settings output object | output settings |
| mode | voice settings mode object | voice mode settings |
| automatic_gain_control | boolean | state of automatic gain control |
| echo_cancellation | boolean | state of echo cancellation |
| noise_suppression | boolean | state of noise suppression |
| qos | boolean | voice quality of service state |
| silence_warning | boolean | state of silence warning notice |
| deaf | boolean | state of self-deafen |
| mute | boolean | state of self-mute |

###### Example Set Voice Settings Command Payload

```
{
  "nonce": "3d64ed55-ef6e-4bd5-99c9-677533babc22",
  "args": {
    "input": {
      "volume": 90.5
    }
  },
  "cmd": "SET_VOICE_SETTINGS"
}

```

###### Example Set Voice Settings Response Payload

```
{
  "cmd": "SET_VOICE_SETTINGS",
  "data": {
    "input": {
      "available_devices": [
        {
          "id": "default",
          "name": "Default"
        },
        {
          "id": "Built-in Microphone",
          "name": "Built-in Microphone"
        }
      ],
      "device_id": "default",
      "volume": 90.5
    },
    "output": {
      "available_devices": [
        {
          "id": "default",
          "name": "Default"
        },
        {
          "id": "Built-in Output",
          "name": "Built-in Output"
        }
      ],
      "device_id": "default",
      "volume": 93.00000071525574
    },
    "mode": {
      "type": "VOICE_ACTIVITY",
      "auto_threshold": true,
      "threshold": -46.92622950819673,
      "shortcut": [{ "type": 0, "code": 12, "name": "i" }],
      "delay": 98.36065573770492
    },
    "automatic_gain_control": false,
    "echo_cancellation": false,
    "noise_suppression": false,
    "qos": false,
    "silence_warning": false,
    "deaf": false,
    "mute": false
  },
  "nonce": "3d64ed55-ef6e-4bd5-99c9-677533babc22"
}

```

#### SUBSCRIBE

 Subscribes to events. Set payload `evt` to the event name and payload `args` to the arguments required for that event.

###### Subscribe Response Structure

| Field | Type | Description |
| --- | --- | --- |
| evt | string | event name now subscribed to |

###### Example Subscribe Command Payload

```
{
  "nonce": "be9a6de3-31d0-4767-a8e9-4818c5690015",
  "args": {
    "guild_id": "199737254929760256"
  },
  "evt": "GUILD_STATUS",
  "cmd": "SUBSCRIBE"
}

```

###### Example Subscribe Response Payload

```
{
  "cmd": "SUBSCRIBE",
  "data": {
    "evt": "GUILD_STATUS"
  },
  "nonce": "be9a6de3-31d0-4767-a8e9-4818c5690015"
}

```

#### UNSUBSCRIBE

 Unsubscribes from events. Set payload `evt` to the event that was previously subscribed to, and set `args` to the arguments used for that subscription.

###### Unsubscribe Response Structure

| Field | Type | Description |
| --- | --- | --- |
| evt | string | event name now unsubscribed from |

###### Example Unsubscribe Command Payload

```
{
  "nonce": "647d814a-4cf8-4fbb-948f-898aad24f55b",
  "args": {
    "guild_id": "199737254929760256"
  },
  "evt": "GUILD_STATUS",
  "cmd": "UNSUBSCRIBE"
}

```

###### Example Unsubscribe Response Payload

```
{
  "cmd": "UNSUBSCRIBE",
  "data": {
    "evt": "GUILD_STATUS"
  },
  "nonce": "647d814a-4cf8-4fbb-948f-898aad24f55b"
}

```

#### SET_CERTIFIED_DEVICES

 Lets hardware manufacturers report the current state of certified devices connected to Discord.

###### Set Certified Devices Argument Structure

| Field | Type | Description |
| --- | --- | --- |
| devices | array of certified device objects | list of the manufacturer's devices, ordered by priority |

###### Device Object

| Field | Type | Description |
| --- | --- | --- |
| type | device type | the type of device |
| id | string | the device’s Windows UUID |
| vendor | vendor object | the hardware vendor |
| model | model object | the model of the product |
| related | array of strings | UUIDs of related devices |
| echo_cancellation?* | boolean | whether native echo cancellation is enabled for the device |
| noise_suppression?* | boolean | whether native noise suppression is enabled for the device |
| automatic_gain_control?* | boolean | whether native automatic gain control is enabled for the device |
| hardware_mute?* | boolean | whether the device is muted in hardware |

 *These fields apply only to `AUDIO_INPUT` device types

###### Vendor Object

| Field | Type | Description |
| --- | --- | --- |
| name | string | name of the vendor |
| url | string | url for the vendor |

###### Model Object

| Field | Type | Description |
| --- | --- | --- |
| name | string | name of the model |
| url | string | url for the model |

###### Device Type

| Type | Value |
| --- | --- |
| AUDIO_INPUT | ”audioinput” |
| AUDIO_OUTPUT | ”audiooutput” |
| VIDEO_INPUT | ”videoinput” |

###### Example Set Certified Devices Command Payload

```
{
  "nonce": "9b4e9711-97f3-4f35-b047-32c82a51978e",
  "cmd": "SET_CERTIFIED_DEVICES",
  "args": {
    "devices": [
      {
        "type": "audioinput",
        "id": "aafc2003-da0e-42a3-b982-6a17a2812510",
        "vendor": {
          "name": "SteelSeries",
          "url": "https://steelseries.com"
        },
        "model": {
          "name": "Arctis 7",
          "url": "https://steelseries.com/gaming-headsets/arctis-7"
        },
        "related": ["aafc2003-da0e-42a3-b982-6a17a2819999"],
        "echo_cancellation": true,
        "noise_suppression": true,
        "automatic_gain_control": true,
        "hardware_mute": false
      }
    ]
  }
}

```

###### Example Set Certified Devices Response Payload

```
{
  "nonce": "9b4e9711-97f3-4f35-b047-32c82a51978e",
  "cmd": "SET_CERTIFIED_DEVICES",
  "data": null,
  "evt": null
}

```

#### SET_ACTIVITY

 Updates a user's Rich Presence.

###### Set Activity Argument Structure

 When using `SET_ACTIVITY`, the `activity` object can only use `type` values Playing (`0`), Listening (`2`), Watching (`3`), or Competing (`5`).
| Field | Type | Description |
| --- | --- | --- |
| pid | integer | the application’s process id |
| activity | activity object | rich presence to set for the user |

###### Example Set Activity Payload

```
{
  "cmd": "SET_ACTIVITY",
  "args": {
    "pid": 9999,
    "activity": {
      "state": "In a Group",
      "state_url": "https://example.com/groups/50335231-9d9d-4ebd-873b-984787ee4d1d",
      "details": "Competitive | In a Match",
      "details_url": "https://example.com/matches/42340203-2f25-4534-8ff6-2a6509e81207",
      "timestamps": {
        "start": time(nullptr),
        "end": time(nullptr) + (60 * 5 + 23)
      },
      "assets": {
        "large_image": "numbani_map",
        "large_text": "Numbani",
        "large_url": "https://example.wiki/maps/Numbani",
        "small_image": "pharah_profile",
        "small_text": "Pharah",
        "small_url": "https://example.wiki/characters/Pharah"
      },
      "party": {
        "id": GameEngine.GetPartyId(),
        "size": [3, 6]
      },
      "secrets": {
        "join": "025ed05c71f639de8bfaa0d679d7c94b2fdce12f",
        "spectate": "e7eb30d2ee025ed05c71ea495f770b76454ee4e0",
        "match": "4b2fdce12f639de8bfa7e3591b71a0d679d7c93f"
      },
      "instance": true
    }
  },
  "nonce": "647d814a-4cf8-4fbb-948f-898abd24f55b"
}

```

#### SEND_ACTIVITY_JOIN_INVITE

 Accepts an Ask to Join request.

###### Send Activity Join Invite Argument Structure

| Field | Type | Description |
| --- | --- | --- |
| user_id | snowflake | id of the user who sent the request |

###### Example Send Activity Join Invite Payload

```
{
  "nonce": "5dc0c062-98c6-47a0-8922-15aerg126",
  "cmd": "SEND_ACTIVITY_JOIN_INVITE",
  "args": {
    "user_id": "53908232506183680"
  }
}

```

#### CLOSE_ACTIVITY_REQUEST

 Rejects an Ask to Join request.

###### Close Activity Request Argument Structure

| Field | Type | Description |
| --- | --- | --- |
| user_id | snowflake | id of the user who sent the request |

###### Example Close Activity Request Payload

```
{
  "nonce": "5dc0c062-98c6-47a0-8922-15aerg126",
  "cmd": "CLOSE_ACTIVITY_REQUEST",
  "args": {
    "user_id": "53908232506183680"
  }
}

```

#### READY

###### Ready Dispatch Data Structure

| Field | Type | Description |
| --- | --- | --- |
| v | integer | RPC version |
| config | rpc server configuration object | server configuration |
| user | partial user object | user that the current connection is with |

###### RPC Server Configuration Object

| Field | Type | Description |
| --- | --- | --- |
| cdn_host | string | server’s cdn |
| api_endpoint | string | server’s api endpoint |
| environment | string | server’s environment |

###### Example Ready Dispatch Payload

```
{
  "cmd": "DISPATCH",
  "data": {
    "v": 1,
    "config": {
      "cdn_host": "cdn.discordapp.com",
      "api_endpoint": "//discord.com/api",
      "environment": "production"
    },
    "user": {
      "id": "53908232506183680",
      "username": "Mason",
      "discriminator": "1337",
      "avatar": null
    }
  },
  "evt": "READY"
}

```

#### ERROR

###### Error Data Structure

| Field | Type | Description |
| --- | --- | --- |
| code | integer | RPC Error Code |
| message | string | Error description |

###### Example Error Payload

```
{
  "cmd": "AUTHORIZE",
  "data": {
    "code": 4007,
    "message": "No client id provided"
  },
  "evt": "ERROR",
  "nonce": "5102b6f0-c769-4f37-8cca-25fb0ab22628"
}

```

#### GUILD_STATUS

###### Guild Status Argument Structure

| Field | Type | Description |
| --- | --- | --- |
| guild_id | string | guild id to subscribe to for updates |

###### Guild Status Dispatch Data Structure

| Field | Type | Description |
| --- | --- | --- |
| guild | partial guild object | guild with requested id |
| online | integer | online user count for the guild (deprecated; always 0) |

###### Example Guild Status Dispatch Payload

```
{
  "cmd": "DISPATCH",
  "data": {
    "guild": {
      "id": "199737254929760256",
      "name": "test",
      "icon_url": null
    },
    "online": 0
  },
  "evt": "GUILD_STATUS"
}

```

#### GUILD_CREATE

 No arguments

###### Guild Create Dispatch Data Structure

| Field | Type | Description |
| --- | --- | --- |
| id | string | guild id |
| name | string | name of the guild |

###### Example Guild Create Dispatch Payload

```
{
  "cmd": "DISPATCH",
  "data": {
    "id": "199737254929767562",
    "name": "Test Server"
  },
  "evt": "GUILD_CREATE"
}

```

#### CHANNEL_CREATE

 No arguments

###### Channel Create Dispatch Data Structure

| Field | Type | Description |
| --- | --- | --- |
| id | string | channel id |
| name | string | name of the channel |
| type | integer | channel type (guild text: 0, guild voice: 2, dm: 1, group dm: 3) |

###### Example Channel Create Dispatch Payload

```
{
  "cmd": "DISPATCH",
  "data": {
    "id": "199737254929760257",
    "name": "General",
    "type": 0
  },
  "evt": "CHANNEL_CREATE"
}

```

#### VOICE_CHANNEL_SELECT

 No arguments

###### Voice Channel Select Dispatch Data Structure

| Field | Type | Description |
| --- | --- | --- |
| channel_id | string | id of channel (`null` if none) |
| guild_id | string | id of guild (`null` if none) |

###### Example Voice Channel Select Dispatch Payload

```
{
  "cmd": "DISPATCH",
  "data": {
    "channel_id": "199737254929760257",
    "guild_id": "199737254929760256"
  },
  "evt": "VOICE_CHANNEL_SELECT"
}

```

#### VOICE_SETTINGS_UPDATE

###### Voice Settings Argument Structure

 Takes no arguments. Dispatches the [Get Voice Settings](https://docs.discord.com/developers/topics/rpc#getvoicesettings) response.

###### Example Voice Settings Dispatch Payload

```
{
  "cmd": "DISPATCH",
  "data": {
    "input": {
      "available_devices": [
        {
          "id": "default",
          "name": "Default"
        },
        {
          "id": "Built-in Microphone",
          "name": "Built-in Microphone"
        }
      ],
      "device_id": "default",
      "volume": 49.803921580314636
    },
    "output": {
      "available_devices": [
        {
          "id": "default",
          "name": "Default"
        },
        {
          "id": "Built-in Output",
          "name": "Built-in Output"
        }
      ],
      "device_id": "default",
      "volume": 93.00000071525574
    },
    "mode": {
      "type": "VOICE_ACTIVITY",
      "auto_threshold": true,
      "threshold": -46.92622950819673,
      "shortcut": [{ "type": 0, "code": 12, "name": "i" }],
      "delay": 98.36065573770492
    },
    "automatic_gain_control": false,
    "echo_cancellation": false,
    "noise_suppression": false,
    "qos": false,
    "silence_warning": false
  },
  "evt": "VOICE_SETTINGS_UPDATE"
}

```

#### VOICE_STATE_CREATE/VOICE_STATE_UPDATE/VOICE_STATE_DELETE

 Dispatches channel voice state objects

###### Voice State Argument Structure

| Field | Type | Description |
| --- | --- | --- |
| channel_id | string | channel id to subscribe to for updates |

###### Example Voice State Dispatch Payload

```
{
  "cmd": "DISPATCH",
  "evt": "VOICE_STATE_CREATE",
  "data": {
    "voice_state": {
      "mute": false,
      "deaf": false,
      "self_mute": false,
      "self_deaf": false,
      "suppress": false
    },
    "user": {
      "id": "190320984123768832",
      "username": "test 2",
      "discriminator": "7479",
      "avatar": "b004ec1740a63ca06ae2e14c5cee11f3",
      "bot": false
    },
    "nick": "test user 2",
    "volume": 110,
    "mute": false,
    "pan": {
      "left": 1.0,
      "right": 1.0
    }
  }
}

```

#### VOICE_CONNECTION_STATUS

 No arguments

###### Voice Connection Status Dispatch Data Structure

| Field | Type | Description |
| --- | --- | --- |
| state | string | voice connection state, chosen from the states listed below |
| hostname | string | hostname for the voice server currently connected |
| pings | array of integers | last 20 pings (in ms) |
| average_ping | integer | average ping (in ms) |
| last_ping | integer | last ping (in ms) |

###### Voice Connection States

| Field | Description |
| --- | --- |
| DISCONNECTED | TCP disconnected |
| AWAITING_ENDPOINT | Waiting for voice endpoint |
| AUTHENTICATING | TCP authenticating |
| CONNECTING | TCP connecting |
| CONNECTED | TCP connected |
| VOICE_DISCONNECTED | TCP connected, Voice disconnected |
| VOICE_CONNECTING | TCP connected, Voice connecting |
| VOICE_CONNECTED | TCP connected, Voice connected |
| NO_ROUTE | No route to host |
| ICE_CHECKING | WebRTC ice checking |

###### Example Voice Connection Status Dispatch Payload

```
{
  "cmd": "DISPATCH",
  "evt": "VOICE_CONNECTION_STATUS",
  "data": {
    "state": "VOICE_CONNECTED",
    "hostname": "some-server.discord.gg",
    "pings": [20, 13.37],
    "average_ping": 13.37,
    "last_ping": 20
  }
}

```

#### MESSAGE_CREATE/MESSAGE_UPDATE/MESSAGE_DELETE

 Dispatches message objects. For deletions, the message object contains only the id.

###### Message Argument Structure

| Field | Type | Description |
| --- | --- | --- |
| channel_id | string | channel id to subscribe to for updates |

###### Example Message Dispatch Payload

```
{
  "cmd": "DISPATCH",
  "data": {
    "channel_id": "199737254929760256",
    "message": {
      "id": "199743874640379904",
      "blocked": false,
      "content": "test",
      "content_parsed": [
        {
          "content": "test",
          "type": "text"
        }
      ],
      "author_color": "#ffffff",
      "edited_timestamp": null,
      "timestamp": "2016-07-05T04:30:50.776Z",
      "tts": false,
      "mentions": [],
      "mention_roles": [],
      "mention_everyone": false,
      "embeds": [],
      "attachments": [],
      "type": 0,
      "pinned": false,
      "author": {
        "id": "190320984123768832",
        "username": "test user 2",
        "discriminator": "7479",
        "avatar": "b004ec1740a63ca06ae2e14c5cee11f3",
        "bot": false
      }
    }
  },
  "evt": "MESSAGE_CREATE"
}

```

#### SPEAKING_START/SPEAKING_STOP

###### Speaking Argument Structure

| Field | Type | Description |
| --- | --- | --- |
| channel_id | string | channel id to subscribe to for updates |

###### Speaking Dispatch Data Structure

| Field | Type | Description |
| --- | --- | --- |
| user_id | string | id of the user who began or stopped speaking |

###### Example Speaking Dispatch Payload

```
{
  "cmd": "DISPATCH",
  "data": {
    "user_id": "190320984123768832"
  },
  "evt": "SPEAKING_STOP"
}

```

#### NOTIFICATION_CREATE

 Takes no arguments. Requires the `rpc.notifications.read` [OAuth2 scope](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-scopes).

###### Notification Create Dispatch Data Structure

| Field | Type | Description |
| --- | --- | --- |
| channel_id | string | id of the channel where the notification occurred |
| message | message object | message that generated this notification |
| icon_url | string | icon url of the notification |
| title | string | title of the notification |
| body | string | body of the notification |

###### Example Notification Create Dispatch Payload

```
{
  "cmd": "DISPATCH",
  "data": {
    "channel_id": "199737254929760256",
    "message": {
      "id": "199743874640379904",
      "blocked": false,
      "content": "test",
      "content_parsed": [
        {
          "content": "test",
          "type": "text"
        }
      ],
      "author_color": "#ffffff",
      "edited_timestamp": null,
      "timestamp": "2016-07-05T04:30:50.776Z",
      "tts": false,
      "mentions": [],
      "mention_roles": [],
      "mention_everyone": false,
      "embeds": [],
      "attachments": [],
      "type": 0,
      "pinned": false,
      "author": {
        "id": "190320984123768832",
        "username": "test user 2",
        "discriminator": "7479",
        "avatar": "b004ec1740a63ca06ae2e14c5cee11f3",
        "bot": false
      }
    },
    "icon_url": "https://cdn.discordapp.com/avatars/155607406007681024/8ab559b8286e48270c04471ae382cd9d.jpg",
    "title": "test_user (#general)",
    "body": "test message"
  },
  "evt": "NOTIFICATION_CREATE"
}

```

#### ACTIVITY_JOIN

 No arguments

###### Activity Join Dispatch Data Structure

| Field | Type | Description |
| --- | --- | --- |
| secret | string | the `join_secret` for the given invite |

###### Example Activity Join Dispatch Payload

```
{
  "cmd": "DISPATCH",
  "data": {
    "secret": "025ed05c71f639de8bfaa0d679d7c94b2fdce12f"
  },
  "evt": "ACTIVITY_JOIN"
}

```

#### ACTIVITY_SPECTATE

 No arguments

###### Activity Spectate Dispatch Data Structure

| Field | Type | Description |
| --- | --- | --- |
| secret | string | the `spectate_secret` for the given invite |

###### Example Activity Spectate Dispatch Payload

```
{
  "cmd": "DISPATCH",
  "data": {
    "secret": "e7eb30d2ee025ed05c71ea495f770b76454ee4e0"
  },
  "evt": "ACTIVITY_SPECTATE"
}

```

#### ACTIVITY_JOIN_REQUEST

 No arguments

###### Activity Join Request Data Structure

| Field | Type | Description |
| --- | --- | --- |
| user | partial user object | details about the user asking to join |

###### Example Activity Join Request Dispatch Payload

```
{
  "cmd": "DISPATCH",
  "data": {
    "user": {
      "id": "53908232506183680",
      "username": "Mason",
      "discriminator": "1337",
      "avatar": "a_bab14f271d565501444b2ca3be944b25"
    }
  },
  "evt": "ACTIVITY_JOIN_REQUEST"
}

```

#### ACTIVITY_INVITE

 No arguments

###### Activity Invite Dispatch Data Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | invite type; `1` for join |
| user | partial user object | user who sent the invite |
| activity | activity object | activity tied to the invite |
| channel_id | string | id of the channel where the invite was sent |
| message_id | string | id of the invite message |

###### Example Activity Invite Dispatch Payload

```
{
  "cmd": "DISPATCH",
  "data": {
    "type": 1,
    "user": {
      "id": "53908232506183680",
      "username": "Mason",
      "discriminator": "1337",
      "avatar": "a_bab14f271d565501444b2ca3be944b25"
    },
    "activity": {
      "application_id": "192741864418312192",
      "name": "My Game",
      "party": {
        "id": "party1234",
        "size": [2, 5]
      }
    },
    "channel_id": "199737254929760256",
    "message_id": "199743874640379904"
  },
  "evt": "ACTIVITY_INVITE"
}

```

#### CURRENT_USER_UPDATE

 Takes no arguments. Dispatches the current user's profile each time it changes, such as avatar or username updates.

###### Current User Update Dispatch Data Structure

| Field | Type | Description |
| --- | --- | --- |
| id | string | user’s id |
| username | string | user’s username |
| discriminator | string | user’s discriminator |
| global_name | string | user’s display name |
| avatar | string | user’s avatar hash |
| avatar_decoration_data | object | avatar decoration data, or `null` when absent |
| bot | boolean | whether this user account is a bot |
| flags | integer | public flags attached to the user's account |
| premium_type | integer | Nitro subscription type. Requires `identify.premium` scope. |

###### Example Current User Update Dispatch Payload

```
{
  "cmd": "DISPATCH",
  "data": {
    "id": "53908232506183680",
    "username": "Mason",
    "discriminator": "0",
    "global_name": "Mason",
    "avatar": "a_bab14f271d565501444b2ca3be944b25",
    "avatar_decoration_data": null,
    "bot": false,
    "flags": 64,
    "premium_type": 0
  },
  "evt": "CURRENT_USER_UPDATE"
}

```

#### RELATIONSHIP_UPDATE
