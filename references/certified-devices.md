# Certified Devices

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Discord lets hardware vendors report extra details about certified devices connected to a user's computer. The feature is not for advertising that a player's PUBG Chicken Dinner came from the TotallyRealHardware RGB Mouse and Keyboard Set Extraordinaire(TM), but it can let your hardware provide a better Discord experience.

### How’s it work?

 I’m glad you asked!
1. [Create an application](https://discord.com/developers/applications) for the hardware vendor, then keep the Client ID.
2. Communicate with Discord through a single HTTP or WebSocket call.
3. Whenever device state changes, send Discord a [`SET_CERTIFIED_DEVICES`](https://docs.discord.com/developers/topics/rpc#setcertifieddevices) WebSocket payload, or make the matching HTTP POST.
 That is the whole flow: provide current information for connected devices, and Discord applies it so people using the hardware get the right experience. Discord audio settings also show a `CERTIFIED` badge for the device. (Figure: An example of how a certified device may be shown for an example audio input and output device)

### Connecting

###### Query String Params

| Name | Value | Required |
| --- | --- | --- |
| v | `1` | All |
| client_id | your app’s client id | All |
| encoding | `json` | WebSocket |

 Send device event updates to these URIs:

###### HTTP

```
http://127.0.0.1:PORT/rpc?v=1&client_id=YOUR_CLIENT_ID

```

###### WebSocket

```
ws://127.0.0.1:PORT?v=1&client_id=YOUR_CLIENT_ID&encoding=json

```

 `PORT` covers ports `6463` through `6473`. Try the ports in that range until the request returns a success status or the socket connects, then reuse the successful port for the session. Keep Discord synchronized with the hardware by sending an update whenever hardware mute changes, or when a voice feature such as echo cancellation is turned on or off by the user. These updates let Discord defer when your hardware is handling optimization, or assist when it is not. Every update must include the complete `devices` array, ordered by your priority preference. For example, place a headset first if Discord should attempt to choose it as the default.

### Getting Device UUID

 Each device in a `SET_CERTIFIED_DEVICES` payload has an `id` field. On Windows, set `id` to the device UUID from the native Windows API; the value resembles `{0.0.1.00000000}.{6cff2b76-44a8-46b9-b528-262ad8609d9f}`. On macOS, use the device name as the `id`.

###### Microphone Id Example

```
id = waveInMessage((HWAVEIN)IntToPtr(index),
                      DRV_QUERYFUNCTIONINSTANCEID,
                      (DWORD_PTR)pstrEndpointId,
                      cbEndpointId);

```

###### Speaker Id Example

```
id = waveOutMessage((HWAVEIN)IntToPtr(index),
                      DRV_QUERYFUNCTIONINSTANCEID,
                      (DWORD_PTR)pstrEndpointId,
                      cbEndpointId);

```

### HTTP Example

###### HTTP Request Example

```
curl -X POST -H 'Content-Type: application/json' -d '
{
  "nonce": "9b4e9711-97f3-4f35-b047-32c82a51978e",
  "cmd": "SET_CERTIFIED_DEVICES",
  "args": {
    "devices": [
      {
        "type": "audioinput",
        "id": "{0.0.1.00000000}.{6cff2b76-44a8-46b9-b528-262ad8609d9f}",
        "vendor": {
          "name": "SteelSeries",
          "url": "https://steelseries.com"
        },
        "model": {
          "name": "Arctis 7",
          "url": "https://steelseries.com/gaming-headsets/arctis-7"
        },
        "related": ["{0.0.1.00000000}.{6cff2b76-44a8-46b9-b528-262ad8609d9f}"],
        "echo_cancellation": true,
        "noise_suppression": true,
        "automatic_gain_control": true,
        "hardware_mute": false
      }
    ]
  }
}
' http://127.0.0.1:PORT/rpc?v=1&client_id=YOUR_CLIENT_ID

```

 The socket returns `200 OK` with this JSON body.

###### HTTP Response Example

```
{
  "cmd": "SET_CERTIFIED_DEVICES",
  "data": null,
  "evt": null,
  "nonce": "9b4e9711-97f3-4f35-b047-32c82a51978e"
}

```

### WebSocket Example

###### RPC Command Example

```
{
  "nonce": "9b4e9711-97f3-4f35-b047-32c82a51978e",
  "cmd": "SET_CERTIFIED_DEVICES",
  "args": {
    "devices": [
      {
        "type": "audioinput",
        "id": "{0.0.1.00000000}.{6cff2b76-44a8-46b9-b528-262ad8609d9f}",
        "vendor": {
          "name": "SteelSeries",
          "url": "https://steelseries.com"
        },
        "model": {
          "name": "Arctis 7",
          "url": "https://steelseries.com/gaming-headsets/arctis-7"
        },
        "related": ["{0.0.1.00000000}.{6cff2b76-44a8-46b9-b528-262ad8609d9f}"],
        "echo_cancellation": true,
        "noise_suppression": true,
        "automatic_gain_control": true,
        "hardware_mute": false
      }
    ]
  }
}

```

###### RPC Response Example

```
{
  "nonce": "9b4e9711-97f3-4f35-b047-32c82a51978e",
  "cmd": "SET_CERTIFIED_DEVICES",
  "data": null,
  "evt": null
}

```

### Models

###### Device Object

| Field | Type | Description |
| --- | --- | --- |
| type | device type | the type of device |
| id | string | the device’s Windows UUID |
| vendor | vendor object | the hardware vendor |
| model | model object | the model of the product |
| related | array of strings | UUIDs of related devices |
| echo_cancellation?* | boolean | whether native echo cancellation is active on the device |
| noise_suppression?* | boolean | whether the device has native noise suppression enabled |
| automatic_gain_control?* | boolean | whether native automatic gain control is active for the device |
| hardware_mute?* | boolean | whether the hardware mute state is enabled for the device |

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

[RPCPrevious](https://docs.discord.com/developers/topics/rpc)⌘I
