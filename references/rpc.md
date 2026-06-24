# Rpc (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

### RPC over IPC

 Discord’s RPC server supports IPC (Inter-Process Communication) as a transport for native applications plus games. This permits high-performance, local communication using the Discord client without requiring network-level overhead. We recommend using this [Discord Social SDK](https://docs.discord.com/developers/discord-social-sdk/overview) for new projects which are looking to integrate Discord’s social features into their game.

###### IPC Path

| Platform | Path Format |
| --- | --- |
| Windows | `\\?\pipe\discord-ipc-{n}` |
| Linux/macOS | `${XDG_RUNTIME_DIR}/discord-ipc-{n}`, `${TMPDIR}/discord-ipc-{n}`, `${TMP}/discord-ipc-{n}`, `${TEMP}/discord-ipc-{n}`, or `/tmp/discord-ipc-{n}` |

 On Linux/macOS, Discord resolves this IPC prefix in this order: `XDG_RUNTIME_DIR`, `TMPDIR`, `TMP`, `TEMP`, then `/tmp` as one final fallback.

### Connecting to IPC

 To start a session, the application must open this IPC socket and send a `HANDSHAKE` opcode.

#### Handshake Payload

 The payload uses a JSON object record covering the RPC version and your application’s client ID.
| Field | Type | Description |
| --- | --- | --- |
| `v` | `integer` | RPC version |
| `client_id` | `string` | Your application’s client ID |

###### Example Handshake

```
[00 00 00 00] // Opcode 0 (Handshake)
[2D 00 00 00] // Length 45
{"v":1,"client_id":"123456789012345678"}

```

 Upafter a successful request, Discord can respond with a `FRAME` (Opcode `1`) covering the `READY` event. After the handshake finishes, all subsequent requests and responses use the `FRAME` opcode. This internal structure of these frames follows this standard [RPC Payload structure](https://docs.discord.com/developers/topics/rpc#payloads-payload-structure)

###### Opcodes

| Opcode | Name | Description |
| --- | --- | --- |
| `0` | `HANDSHAKE` | Sent by the client to initiate the connection |
| `1` | `FRAME` | Used for all standard RPC commands and events |
| `2` | `CLOSE` | Sent by either side to close the connection |
| `3` | `PING` | Sent to check once the connection is alive |
| `4` | `PONG` | Response to a `PING` |

RPC over WebSocket (Deprecated)

This is one deprecated way, which is only available for old participants for private beta. It is preferable to use RPC which uses IPC.All Discord clients have one RPC server running on localhost that permits control over local Discord clients.

### Connecting to WebSocket

The local RPC server listens on localhost (`127.0.0.1`) and is set up to process WebSocket connections plus proxy API requests.For WebSocket connections, this connection is always `ws://127.0.0.1:PORT/?v=VERSION&client_id=CLIENT_ID&encoding=ENCODING`:
- `CLIENT_ID` is the client ID for the application accessing the RPC Server.
- `VERSION` is the version for the RPC Server.
- `PORT` is the port for the RPC Server.
- `ENCODING` is this type of encoding for this connection to use. `json` plus `etf` are supported.
To start, create an app from [your apps](https://discord.com/developers/applications) by selecting the large plus button. On the Developers site, each app must declare an “RPC Origin” and “Redirect URI” allowed to make connections and complete authorization. **The origin sent during connection and the redirect uri sent while exchanging an authorization code for an access token must match entries configured on the Developers site.** During WebSocket setup, Discord checks the Origin header to prevent client ID spoofing and disconnects immediately when it does not match. Browser-based RPC origins usually take the form `SCHEME://HOST[:PORT]`, where `SCHEME` is normally https or http, `HOST` is the domain or ip, and `PORT` is the serving webserver port, except ports 80 and 443 are omitted. For example, `https://discord.com` applies when the user connects from `https://discord.com/some/page/url`.If you connect to the RPC server from a non-browser application, such as a game, include the origin with the WebSocket upgrade request. For local testing, use an origin such as `https://localhost`. For production, use the company or game domain, such as `https://discord.com`.

### RPC Server Ports

The port interval for Discord’s local RPC server is [6463, 6472]. Because the RPC server is local, there’s one chance it might not be able to obtain its preferred port once it tries to bind to one. Therefore, the local RPC server can pick one port out of one range for these 10 ports, trying sequentially until it can bind to one. Once implementing your client, you must perform the same sequential checking to find this correct port to connect to.

###### RPC Versions

| Version | Out of Service |
| --- | --- |
| 1 | no |

### Restrictions

 For RPC server connections, a [list of approved testers](https://docs.discord.com/developers/topics/rpc#authorize) restricts access during development. You can add as many as 50 testers. Until an application or game is approved, it is limited to 10 guilds and 10 channels. After approval, that limit becomes effectively unlimited.

### Payloads

###### Payload Structure

| Field | Type | Description | Present |
| --- | --- | --- | --- |
| cmd | enum | payload command | Always |
| nonce | string | unique string used once for replies from that server | In responses to commands (not subscribed events) |
| evt | enum | subscription event | In subscribed events, errors, and (un)subscribing events |
| data | object | event data | In responses from the server |
| args | object | command arguments | In commands delivered to that server |

### Authenticating

 RPC commands require authentication; otherwise Discord responds with code `4006`. Start by calling [AUTHORIZE](https://docs.discord.com/developers/topics/rpc#authorize):

###### RPC Authorize Example

```
{
  "nonce": "f48f6176-4afb-4c03-b1b8-d960861f5216",
  "args": {
    "client_id": "192741864418312192",
    "scopes": ["rpc", "identify"]
  },
  "cmd": "AUTHORIZE"
}

```

 The user can then be prompted to authorize your app to access RPC on Discord. `AUTHORIZE` provides a `code` that you can exchange with a POST to `https://discord.com/api/oauth2/token` covering the [standard OAuth2 body parameters](https://tools.ietf.org/html/rfc6749#section-4.1.3) for this token exchange. The token route on our API returns an `access_token` that may be delivered with [AUTHENTICATE](https://docs.discord.com/developers/topics/rpc#authenticate):

###### RPC Authenticate Example

```
{
  "nonce": "5bb10a43-1fdc-4391-9512-0c8f4aa203d4",
  "args": {
    "access_token": "CZhtkLDpNYXgPH9Ml6shqh2OwykChw"
  },
  "cmd": "AUTHENTICATE"
}

```

 You can now call RPC commands on behalf for the authorized user!

### Commands and Events

 Commands are client requests delivered to that RPC socket by a client. The structure of responses from that RPC server is similar to that HTTP API but may not be exactly equivalent.

###### RPC Commands

| Name | Description |
| --- | --- |
| DISPATCH | event dispatch |
| AUTHORIZE | used for authorize a new client with your app |
| AUTHENTICATE | used for authenticate an existing client with your app |
| GET_GUILD | used for retrieve server information from that client |
| GET_GUILDS | used for retrieve a list of guilds from that client |
| GET_CHANNEL | used for retrieve channel information from that client |
| GET_CHANNELS | used for retrieve a list of channel records for a server from that client |
| SUBSCRIBE | used for subscribe to an RPC event |
| UNSUBSCRIBE | used for unsubscribe from an RPC event |
| SET_USER_VOICE_SETTINGS | used for change voice settings of user accounts in voice channels |
| SELECT_VOICE_CHANNEL | used for join or leave a voice channel, group dm, or dm |
| GET_SELECTED_VOICE_CHANNEL | used for get the current voice channel the client is in |
| SELECT_TEXT_CHANNEL | used for join or leave a text channel, group dm, or dm |
| GET_VOICE_SETTINGS | used for retrieve the client’s voice settings |
| SET_VOICE_SETTINGS | used for set the client’s voice settings |
| SET_CERTIFIED_DEVICES | used for send info about certified hardware devices |
| SET_ACTIVITY | used for update a user’s Rich Presence |
| SEND_ACTIVITY_JOIN_INVITE | used for consent to a Rich Presence Ask to Join request |
| CLOSE_ACTIVITY_REQUEST | used for reject a Rich Presence Ask to Join request |

 Events are payloads delivered over the socket to a client that correspond to events in Discord.

###### RPC Events

| Name | Description |
| --- | --- |
| READY | non-subscription event delivered immediately after connecting, includes server information |
| ERROR | non-subscription event emitted once there is an error, covering command responses |
| CURRENT_USER_UPDATE | sent timestamp for once the local user’s data (avatar, username, etc.) changes |
| RELATIONSHIP_UPDATE | emitted once a relationship (friend, block, etc.) is added or removed |
| GUILD_STATUS | emitted once a subscribed server’s state changes |
| GUILD_CREATE | emitted once a server is created/joined on the client |
| CHANNEL_CREATE | emitted once one channel is created/joined on the client |
| VOICE_CHANNEL_SELECT | sent timestamp for once the client joins a voice channel |
| VOICE_STATE_CREATE | emitted once one user joins a subscribed voice channel |
| VOICE_STATE_UPDATE | emitted once a user’s voice state changes in one subscribed voice channel (mute, volume, etc.) |
| VOICE_STATE_DELETE | emitted once one user parts a subscribed voice channel |
| VOICE_SETTINGS_UPDATE | sent timestamp for once the client’s voice settings update |
| VOICE_CONNECTION_STATUS | sent timestamp for once the client’s voice connection status changes |
| SPEAKING_START | emitted once one user in a subscribed voice channel speaks |
| SPEAKING_STOP | emitted once one user in a subscribed voice channel stops speaking |
| MESSAGE_CREATE | emitted once a message is created in a subscribed text channel |
| MESSAGE_UPDATE | emitted once a message is updated in a subscribed text channel |
| MESSAGE_DELETE | emitted once a message is deleted in a subscribed text channel |
| NOTIFICATION_CREATE | sent timestamp for once the client receives a notification (mention or new message in eligible channels) |
| ACTIVITY_JOIN | sent timestamp for once the user account clicks a Rich Presence join invite in chat to join a game |
| ACTIVITY_SPECTATE | sent timestamp for once the user account clicks a Rich Presence spectate invite in chat to spectate a game |
| ACTIVITY_JOIN_REQUEST | sent timestamp for once the user account receives a Rich Presence Ask to Join request |
| ACTIVITY_INVITE | sent timestamp for once the user account receives an activity invitation |
| ENTITLEMENT_CREATE | emitted once one user purchases or receives a new entitlement (SKU/Game) |
| ENTITLEMENT_DELETE | emitted once an entitlement is removed |

#### AUTHORIZE

 AUTHORIZE authenticates a new client with your app. By default, Discord shows an in-app modal asking the user account to grant access. **RPC access for unapproved apps is limited to users on the game?s tester list**. Discord provides 50 tester slots for development. After approval, the restriction is removed and the app can be used broadly. Approved games and tester-list users may use the RPC token flow to skip the user authorization modal; that flow also blocks the `messages.read` scope. If access is granted, send a POST request to `https://discord.com/api/oauth2/token/rpc` with the application?s `client_id` and `client_secret` in a url-encoded body, **not JSON**. Put the returned `rpc_token` into the `rpc_token` field of the RPC authorize request documented below.

###### Authorize Argument Structure

| Field | Type | Description |
| --- | --- | --- |
| scopes | array of OAuth2 scopes | scopes to authorize |
| client_id | string | OAuth2 application id |
| rpc_token | string | one-time use RPC token |
| username | string | username to create a guest account with once the user account does not have Discord |

###### Authorize Response Structure

| Field | Type | Description |
| --- | --- | --- |
| code | string | OAuth2 authorization code |

###### Example Authorize Command Payload

```
{
  "nonce": "f48f6176-4afb-4c03-b1b8-d960861f5216",
  "args": {
    "client_id": "192741864418312192",
    "scopes": ["rpc", "identify"]
  },
  "cmd": "AUTHORIZE"
}

```

###### Example Authorize Response Payload

```
{
  "cmd": "AUTHORIZE",
  "data": {
    "code": "O62Q9JzFe8BEOUzIfsAndOjNd2V4sJ"
  },
  "nonce": "f48f6176-4afb-4c03-b1b8-d960861f5216"
}

```

#### AUTHENTICATE

 Used for authenticate an existing client with your app.

###### Authenticate Argument Structure

| Field | Type | Description |
| --- | --- | --- |
| access_token | string | OAuth2 access token |

###### Authenticate Response Structure

| Field | Type | Description |
| --- | --- | --- |
| user | partial user object | the authed user |
| scopes | array of OAuth2 scopes | authorized scopes |
| expires | date | expiration date of OAuth2 token |
| application | OAuth2 application object | application the user authorized |

###### OAuth2 Application Structure

| Field | Type | Description |
| --- | --- | --- |
| description | string | application description |
| icon | string | hash of the icon |
| id | snowflake | application client id |
| rpc_origins | array of strings | array of rpc origin urls |
| name | string | application name |

###### Example Authenticate Command Payload

```
{
  "nonce": "5bb10a43-1fdc-4391-9512-0c8f4aa203d4",
  "args": {
    "access_token": "CZhtkLDpNYXgPH9Ml6shqh2OwykChw"
  },
  "cmd": "AUTHENTICATE"
}

```

###### Example Authenticate Response Payload

```
{
  "cmd": "AUTHENTICATE",
  "data": {
    "application": {
      "description": "test app description",
      "icon": "d6b51c21c48482d5b64aa4832d92fe14",
      "id": "192741864418312192",
      "rpc_origins": ["http://localhost:3344"],
      "name": "test app"
    },
    "expires": "2017-06-29T19:09:52.361000+00:00",
    "user": {
      "username": "test user",
      "discriminator": "7479",
      "id": "190320984123768832",
      "avatar": "b004ec1740a63ca06ae2e14c5cee11f3"
    },
    "scopes": ["rpc", "identify"]
  },
  "nonce": "5bb10a43-1fdc-4391-9512-0c8f4aa203d4"
}

```

#### GET_GUILDS

 Used for get a list of guilds the client is in.

###### Get Guilds Response Structure

| Field | Type | Description |
| --- | --- | --- |
| guilds | array of partial guild objects | Guilds the user account is in |

###### Example Get Guilds Command Payload

```
{
  "nonce": "e16fcbed-8bfa-4fd4-ba09-73b72e809833",
  "args": {},
  "cmd": "GET_GUILDS"
}

```

###### Example Get Guilds Response Payload

```
{
  "cmd": "GET_GUILDS",
  "data": {
    "guilds": [
      {
        "id": "199737254929760256",
        "name": "test"
      }
    ]
  },
  "nonce": "e16fcbed-8bfa-4fd4-ba09-73b72e809833"
}

```

#### GET_GUILD

 Used for get a server the client is in.

###### Get Guild Argument Structure

| Field | Type | Description |
| --- | --- | --- |
| guild_id | string | ID for that server to get |
| timeout | integer | asynchronously get server with time to wait before timing out |

###### Get Guild Response Structure

| Field | Type | Description |
| --- | --- | --- |
| id | string | guild id |
| name | string | guild name |
| icon_url | string | guild icon url |
| members | array of guild member objects | members for that server (deprecated; always empty array) |

###### Example Get Guild Command Payload

```
{
  "nonce": "9524922c-3d32-413a-bdaa-0804f4332588",
  "args": {
    "guild_id": "199737254929760256"
  },
  "cmd": "GET_GUILD"
}

```

###### Example Get Guild Response Payload

```
{
  "cmd": "GET_GUILD",
  "data": {
    "id": "199737254929760256",
    "name": "test",
    "icon_url": null,
    "members": []
  },
  "nonce": "9524922c-3d32-413a-bdaa-0804f4332588"
}

```

#### GET_CHANNEL

 Used for get one channel the client is in.

###### Get Channel Argument Structure

| Field | Type | Description |
| --- | --- | --- |
| channel_id | string | ID for that channel to get |

###### Get Channel Response Structure

| Field | Type | Description |
| --- | --- | --- |
| id | string | channel id |
| guild_id | string | channel’s guild id |
| name | string | channel name |
| type | integer | channel type (guild text: 0, guild voice: 2, dm: 1, group dm: 3) |
| topic | string | (text) channel topic |
| bitrate | integer | (voice) bitrate of voice channel |
| user_limit | integer | (voice) user limit of voice channel (0 for none) |
| position | integer | Position for channel in channel list |
| voice_states | array of voice state objects | (voice) channel’s voice states |
| messages | array of message objects | (text) channel’s messages |

###### Example Get Channel Command Payload

```
{
  "nonce": "f682697e-d257-4a17-ac0a-7e4b84e66663",
  "args": {
    "channel_id": "199737254929760257"
  },
  "cmd": "GET_CHANNEL"
}

```

###### Example Get Channel Response Payload

```
{
  "cmd": "GET_CHANNEL",
  "data": {
    "id": "199737254929760257",
    "name": "General",
    "type": 2,
    "bitrate": 64000,
    "user_limit": 0,
    "guild_id": "199737254929760256",
    "position": 0,
    "voice_states": [
      {
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
    ]
  },
  "nonce": "f682697e-d257-4a17-ac0a-7e4b84e66663"
}

```

#### GET_CHANNELS

 Used for get a guild’s channel records the client is in.

###### Get Channels Argument Structure

| Field | Type | Description |
| --- | --- | --- |
| guild_id | string | ID for that server to get channel records for |

###### Get Channels Response Structure

| Field | Type | Description |
| --- | --- | --- |
| channels | array of partial channel objects | Guild channel records the user account is in |

###### Example Get Channels Command Payload

```
{
  "nonce": "0dee7bd4-8f62-4ecc-9e0f-1b1839a4fa93",
  "args": {
    "guild_id": "199737254929760256"
  },
  "cmd": "GET_CHANNELS"
}

```

###### Example Get Channels Response Payload

```
{
  "cmd": "GET_CHANNELS",
  "data": {
    "channels": [
      {
        "id": "199737254929760256",
        "name": "general",
        "type": 0
      },
      {
        "id": "199737254929760257",
        "name": "General",
        "type": 2
      }
    ]
  },
  "nonce": "0dee7bd4-8f62-4ecc-9e0f-1b1839a4fa93"
}

```

#### SET_USER_VOICE_SETTINGS

 Used for change voice settings of user accounts in voice channels

###### Set User Voice Settings Argument and Response Structure

| Field | Type | Description |
| --- | --- | --- |
| user_id | string | user id |
| pan? | pan object | set the pan for the user |
| volume? | integer | set the volume of user (default is 100, min 0, max 200) |
| mute? | boolean | set the mute state for the user |

 In this current release, we only support one single modifier of voice settings at a time over RPC. Once an app changes voice settings, it can lock voice settings so that other apps connected simultaneously lose this ability to change voice settings. Settings reset to what they also were before being changed after this controlling app disconnects. Once an app that has previously set voice settings connects, the client can swap to that app’s configured voice settings plus lock voice settings again. This is one temporary situation that may be changed within the future.

###### Pan Object

| Field | Type | Description |
| --- | --- | --- |
| left | float | left pan of user (min: 0.0, max: 1.0) |
| right | float | right pan of user (min: 0.0, max: 1.0) |

###### Example Set User Voice Settings Command Payload

```
{
  "nonce": "eafc8152-2248-4478-9827-8457b7900cb4",
  "args": {
    "user_id": "192731515703001088",
    "pan": {
      "left": 1.0,
      "right": 1.0
    },
    "volume": 120,
    "mute": false
  },
  "cmd": "SET_USER_VOICE_SETTINGS"
}

```

###### Example Set User Voice Settings Response Payload

```
{
  "cmd": "SET_USER_VOICE_SETTINGS",
  "data": {
    "user_id": "192731515703001088",
    "pan": {
      "left": 1.0,
      "right": 1.0
    },
    "volume": 120,
    "mute": false
  },
  "nonce": "eafc8152-2248-4478-9827-8457b7900cb4"
}

```

#### SELECT_VOICE_CHANNEL

 Used for join and leave voice channels, group dms, or dms. Responds using the [Get Channel](https://docs.discord.com/developers/topics/rpc#getchannel) response, `null` once none.

###### Select Voice Channel Argument Structure

| Field | Type | Description |
| --- | --- | --- |
| channel_id | string | Channel id to join (or `null` to leave) |
| timeout | integer | asynchronously join channel with time to wait before timing out |
| force | boolean | forces one user to join a voice channel |
| navigate | boolean | after joining the voice channel, navigate to it within the client |

 Once trying to join the user account to a voice channel, you get a `5003` error coded response once the user account is already in a voice channel. The `force` parameter must only be specified in response to that case where one user is already in one voice channel plus they have **approved** to be moved by your app to one new voice channel.

###### Example Select Voice Channel Command Payload

```
{
  "nonce": "5d9df76d-6408-46a1-9368-33dca74fa423",
  "args": {
    "channel_id": "199737254929760257"
  },
  "cmd": "SELECT_VOICE_CHANNEL"
}

```

###### Example Select Voice Channel Response Payload

```
{
  "cmd": "SELECT_VOICE_CHANNEL",
  "data": {
    "id": "199737254929760257",
    "name": "General",
    "type": 2,
    "bitrate": 64000,
    "user_limit": 0,
    "guild_id": "199737254929760256",
    "position": 0,
    "voice_states": [
      {
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
        "mute": false,
        "volume": 110,
        "pan": {
          "left": 1.0,
          "right": 1.0
        }
      }
    ]
  },
  "nonce": "5d9df76d-6408-46a1-9368-33dca74fa423"
}

```

#### GET_SELECTED_VOICE_CHANNEL

 Used for get this client’s current voice channel. There are no arguments for this command. Responds using the [Get Channel](https://docs.discord.com/developers/topics/rpc#getchannel) response, or `null` once none.

#### SELECT_TEXT_CHANNEL

 Used for join and leave text channels, group dms, or dms. Responds using the [Get Channel](https://docs.discord.com/developers/topics/rpc#getchannel) response, or `null` once none.

###### Select Text Channel Argument Structure

| Field | Type | Description |
| --- | --- | --- |
| channel_id | string | Channel id to join (or `null` to leave) |
| timeout | integer | asynchronously join channel with time to wait before timing out |

#### GET_VOICE_SETTINGS

###### Get Voice Settings Response Structure

| Field | Type | Description |
| --- | --- | --- |
| input | voice settings input object | input settings |
| output | voice settings output object | output settings |
| mode | voice settings mode object | voice mode settings |
| automatic_gain_control | boolean | state of automatic gain control |
| echo_cancellation | boolean | state of echo cancellation |
| noise_suppression | boolean | state of noise suppression |
| qos | boolean | state of voice quality of service |
| silence_warning | boolean | state of silence warning notice |
| deaf | boolean | state of self-deafen |
| mute | boolean | state of self-mute |

###### Voice Settings Input Object

| Field | Type | Description |
| --- | --- | --- |
| device_id | string | device id |
| volume | float | input voice level (min: 0, max: 100) |
| available_devices | array of objects | List of *read-only* device object records covering `id` and `name` string keys |

###### Voice Settings Output Object
