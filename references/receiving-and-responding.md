# Receiving And Responding (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

An **[Interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object)** is the payload sent to your application when someone invokes an application command or uses a message component. For [Slash Commands](https://docs.discord.com/developers/interactions/application-commands#slash-commands), it carries the submitted option values. For [User Commands](https://docs.discord.com/developers/interactions/application-commands#user-commands) and [Message Commands](https://docs.discord.com/developers/interactions/application-commands#message-commands), it includes the resolved user or message that was targeted. For [Message Components](https://docs.discord.com/developers/components/reference), it identifies the component that was activated. The payload also supplies trigger context such as `guild_id`, `channel`, `member`, and related fields; the full field set is described in the data models below.

### Interaction Object

###### Interaction Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | ID of the interaction |
| application_id | snowflake | Application ID associated with this interaction |
| type | interaction type | Type of interaction |
| data?* | interaction data | Interaction data payload |
| guild? | partial guild object | Guild where this interaction originated |
| guild_id? | snowflake | Guild where this interaction originated |
| channel? | partial channel object | Channel where this interaction originated |
| channel_id? | snowflake | Channel where this interaction originated |
| member?** | guild member object | Guild member information for the user who invoked it, permissions included |
| user? | user object | User object for the invoker when the interaction happens in a DM |
| token | string | Token used to continue responding to this interaction |
| version | integer | Read-only property, always `1` |
| message? | message object | For components, and modals opened from components, the attached message |
| app_permissions*** | string | Bitfield containing the app permissions in the location that produced the interaction |
| locale?**** | string | Locale selected by the user who invoked the interaction |
| guild_locale? | string | Preferred locale for the guild, when the interaction is from a guild |
| entitlements | array of entitlement objects | For monetized apps, entitlements owned by the invoking user that represent premium SKU access |
| authorizing_integration_owners | dictionary keyed by application integration types | Maps authorized installation contexts for this interaction to the relevant user or guild IDs. See Authorizing Integration Owners Object for details |
| context? | interaction context type | Context from which the interaction was triggered |
| attachment_size_limit | integer | Attachment size limit in bytes |

 * Present for application command, message component, and modal submit interaction types; optional so later interaction types can omit it ** `member` appears for guild invocations, while `user` appears for DM invocations *** In (G)DMs with other users, `app_permissions` includes `ATTACH_FILES | EMBED_LINKS | MENTION_EVERYONE`; DMs with the app's bot user also include `USE_EXTERNAL_EMOJIS` **** Present on every interaction type except PING

###### Interaction Type

| Name | Value |
| --- | --- |
| PING | 1 |
| APPLICATION_COMMAND | 2 |
| MESSAGE_COMPONENT | 3 |
| APPLICATION_COMMAND_AUTOCOMPLETE | 4 |
| MODAL_SUBMIT | 5 |

###### Interaction Context Types

 Discord context where an interaction is allowed or where it was initiated. For application-command usage, see the [commands context documentation](https://docs.discord.com/developers/interactions/application-commands#interaction-contexts).
| Name | Type | Description |
| --- | --- | --- |
| GUILD | 0 | Interaction is available in servers |
| BOT_DM | 1 | Interaction is available in DMs with the app's bot user |
| PRIVATE_CHANNEL | 2 | Interaction is available in Group DMs and DMs that are not with the app's bot user |

###### Authorizing Integration Owners Object

 The `authorizing_integration_owners` field identifies the authorizing user or guild for the installation or installations tied to the interaction. For user-installed apps, it can distinguish the user who authorized the app from the user who triggered the interaction, such as with a message component. A key appears only when all of these conditions hold:
- The app is authorized for the [installation context](https://docs.discord.com/developers/resources/application#application-object-application-integration-types) represented by that key (`GUILD_INSTALL` or `USER_INSTALL`)
- The source [interaction context](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-context-types) (`GUILD`, `BOT_DM`, or `PRIVATE_CHANNEL`) supports the interaction for the installation context represented by that key
- For command invocations, the command also supports that installation context through [`integration_types`](https://docs.discord.com/developers/interactions/application-commands#contexts)
 Values in `authorizing_integration_owners` are determined by the key:
- For `GUILD_INSTALL` (`"0"`), the value changes based on where the interaction came from:
  - If the interaction is from a server, the value is that guild ID
  - If the interaction is from a DM with the app's bot user, the value is `"0"`

- For `USER_INSTALL` (`"1"`), the value is the authorizing user's ID

###### Interaction Data

 The `data` field is present for every [interaction type](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-type) except `PING`, but its shape depends on the type. The tables below describe each interaction type's inner `data` payload.
| Interaction Type | Interaction Data |
| --- | --- |
| PING (`1`) | N / A |
| APPLICATION_COMMAND (`2`) | Application Command Data Structure |
| MESSAGE_COMPONENT (`3`) | Message Component Data Structure |
| APPLICATION_COMMAND_AUTOCOMPLETE (`4`) | Application Command Data Structure |
| MODAL_SUBMIT (`5`) | Modal Submit Data Structure |

###### Application Command Data Structure

 Used with `APPLICATION_COMMAND` and `APPLICATION_COMMAND_AUTOCOMPLETE` interactions.
| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | `ID` of the invoked command |
| name | string | `name` of the invoked command |
| type | integer | `type` of the invoked command |
| resolved? | resolved data | Converted users + roles + channels + attachments |
| options?* | array of application command interaction data option | Params + values from the user |
| guild_id? | snowflake | Guild ID where the command is registered |
| target_id? | snowflake | ID of the user or message selected by a user or message command |

 * This [can be partial](https://docs.discord.com/developers/interactions/application-commands#autocomplete) for `APPLICATION_COMMAND_AUTOCOMPLETE` responses

###### Message Component Data Structure

| Field | Type | Description |
| --- | --- | --- |
| custom_id | string | `custom_id` of the component |
| component_type | integer | type of the component |
| values?* | array of select option values | Values chosen by the user in a select menu component |
| resolved? | resolved data | Resolved entities from selected options |

 * Always included for select menu components

###### Modal Submit Data Structure

| Field | Type | Description |
| --- | --- | --- |
| custom_id | string | Modal `custom_id` supplied by the developer |
| components | array of component interaction response | Values submitted by the user |
| resolved? | resolved data | Resolved entities from selected options |

###### Component Interaction Response Structures

 Structures used when responding to modal interactions and message component interactions.
| Component |
| --- |
| String Select |
| Text Input |
| User Select |
| Role Select |
| Mentionable Select |
| Channel Select |
| Text Display |
| Label |
| File Upload |
| Radio Group |
| Checkbox Group |
| Checkbox |

###### Resolved Data Structure

 When Member data is present, the matching User data is included as well.
| Field | Type | Description |
| --- | --- | --- |
| users? | Map from Snowflakes to user objects | IDs and User objects |
| members?* | Map from Snowflakes to partial member objects | IDs and partial Member objects |
| roles? | Map from Snowflakes to role objects | IDs and Role objects |
| channels?** | Map from Snowflakes to partial channel objects | IDs and partial Channel objects |
| messages? | Map from Snowflakes to partial message objects | IDs and partial Message objects |
| attachments? | Map from Snowflakes to attachment objects | IDs and attachment objects |

 * Partial `Member` objects omit `user`, `deaf`, and `mute` ** Partial `Channel` objects contain only `id`, `name`, `type`, `permissions`, `last_message_id`, `last_pin_timestamp`, `nsfw`, `parent_id`, `guild_id`, `flags`, `rate_limit_per_user`, `topic`, and `position`. Threads also include `thread_metadata`.

###### Application Command Interaction Data Option Structure

 Each option has a name. An option is either a parameter with an input value, which sets `value`, or a subcommand/subcommand group, which contains a top-level key plus an `options` array. `value` and `options` do not appear together.
| Field | Type | Description |
| --- | --- | --- |
| name | string | Name of the parameter |
| type | integer | Application command option type value |
| value? | string, integer, double, or boolean | Option value supplied by user input |
| options? | array of application command interaction data option | Included when this option represents a group or subcommand |
| focused? | boolean | `true` when this option is currently focused for autocomplete |

### Message Interaction Object

 This appears on a [message object](https://docs.discord.com/developers/resources/message#message-object) when the message responds to an Interaction that did not already have a message. Responses to [Message Components](https://docs.discord.com/developers/components/reference) do not include it, because components always belong to existing messages and instead use a [message reference](https://docs.discord.com/developers/resources/message#message-reference-structure) object.

###### Message Interaction Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | ID of the interaction |
| type | interaction type | Type of interaction |
| name | string | Application command name, including any subcommands and subcommand groups |
| user | user object | User who invoked the interaction |
| member? | partial member object | Guild member that invoked the interaction |

### Receiving an Interaction

 When someone interacts with your app, the app receives an **[Interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object)**. There are two possible delivery methods:
- Through the [Interaction Create](https://docs.discord.com/developers/events/gateway-events#interaction-create) gateway event
- Via outgoing webhook
 These delivery methods are **mutually exclusive**; an app can receive Interactions by only one of them. Connected clients can handle the `INTERACTION_CREATE` [Gateway Event](https://docs.discord.com/developers/events/gateway-events#interaction-create), while the webhook path described below works without a connected client. To receive interactions as HTTP outgoing webhooks, configure an Interactions Endpoint URL for the app. See [Preparing for Interactions](https://docs.discord.com/developers/interactions/overview#preparing-for-interactions) in Interactions Overview for setup details.

### Interaction Metadata

 An [Interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object) carries handling metadata plus interaction-type-specific `data`. Samples for each type are on their own pages:
- [Slash Commands](https://docs.discord.com/developers/interactions/application-commands#slash-commands-example-interaction)
- [User Commands](https://docs.discord.com/developers/interactions/application-commands#user-commands-example-interaction)
- [Message Commands](https://docs.discord.com/developers/interactions/application-commands#message-commands-example-interaction)
- [Message Components](https://docs.discord.com/developers/components/using-message-components)
- [Modal Components](https://docs.discord.com/developers/components/using-modal-components)
 All fields are described in the [data models](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object). After collecting user-provided data, the next step is sending a response.

### Responding to an Interaction

 Receiving and responding to Interactions both use webhooks internally, so responding works much like sending a webhook request. Interaction responses follow the same header rules as ordinary HTTP API requests; see [here](https://docs.discord.com/developers/reference#http-api). You can respond to an interaction in several ways:

### Interaction Response Object

###### Interaction Response Structure

| Field | Type | Description |
| --- | --- | --- |
| type | interaction callback type | Type of response |
| data? | interaction callback data | An optional response message |

###### Interaction Callback Type

| Name | Value | Description |
| --- | --- | --- |
| PONG | 1 | ACK a `Ping` |
| CHANNEL_MESSAGE_WITH_SOURCE | 4 | Reply to an interaction with a message |
| DEFERRED_CHANNEL_MESSAGE_WITH_SOURCE | 5 | ACK the interaction, show a loading state, and edit the response later |
| DEFERRED_UPDATE_MESSAGE* | 6 | For components, ACK the interaction and later edit the original message without showing a loading state |
| UPDATE_MESSAGE* | 7 | For components, update the message where the component was attached |
| APPLICATION_COMMAND_AUTOCOMPLETE_RESULT | 8 | Return suggested choices for an autocomplete interaction |
| MODAL** | 9 | Open a popup modal as the interaction response |
| PREMIUM_REQUIRED | 10 | **Deprecated**; return an upgrade button, available only to monetization-enabled apps |
| LAUNCH_ACTIVITY | 12 | Start the app's associated Activity; available only for apps with Activities enabled |

 * Valid only for [component-based](https://docs.discord.com/developers/components/reference) interactions ** Unavailable for `MODAL_SUBMIT` and `PING` interactions.

###### Interaction Callback Data Structure

###### Messages

 Some message fields are not supported at this time.
| Field | Type | Description |
| --- | --- | --- |
| tts? | boolean | Whether the response is TTS |
| content? | string | Message content |
| embeds? | array of embeds | Supports up to 10 embeds |
| allowed_mentions? | allowed mentions | Allowed mentions object |
| flags? * | integer | Message flags as a combined bitfield; only `SUPPRESS_EMBEDS`, `EPHEMERAL`, `IS_COMPONENTS_V2`, `IS_VOICE_MESSAGE`, and `SUPPRESS_NOTIFICATIONS` may be set |
| components? | array of components | Message components |
| attachments? ** | array of partial attachment objects | Attachment objects that include filename and description |
| poll? | poll request object | Details about the poll |

 * For a callback of [type](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-response-object-interaction-callback-type) `DEFERRED_CHANNEL_MESSAGE_WITH_SOURCE`, the only permitted [message flag](https://docs.discord.com/developers/resources/message#message-object-message-flags) is `EPHEMERAL`. To create a component-based message with `IS_COMPONENTS_V2`, use the [edit original response](https://docs.discord.com/developers/interactions/receiving-and-responding#edit-original-interaction-response) endpoint instead. ** See [Uploading Files](https://docs.discord.com/developers/reference#uploading-files) for file details.

###### Autocomplete

| Field | Type | Description |
| --- | --- | --- |
| choices | array of choices | autocomplete choices (max of 25 choices) |

###### Modal

| Field | Type | Description |
| --- | --- | --- |
| custom_id | string | Developer-specified modal identifier, 1-100 characters |
| title | string | Popup modal title, maximum 45 characters |
| components | array of components | Modal components, from 1 through 5 inclusive |

 When application responses contain user data, use [`allowed_mentions`](https://docs.discord.com/developers/resources/message#allowed-mentions-object) to control which mentions in `content` create pings.

### Interaction Callback

 To answer a received interaction, send a `POST` request to `/interactions/<interaction_id>/<interaction_token>/callback`. `interaction_id` is the received payload's unique ID for that Interaction, and `interaction_token` is its unique token. If the Interaction arrives through the gateway, the response **must be sent over HTTP**; Interaction responses **are not gateway commands**. **When this request is sent for an HTTP-delivered interaction, answer the original HTTP request with status 202 and an empty body.**

```
import requests

url = "https://discord.com/api/v10/interactions/<interaction_id>/<interaction_token>/callback"

json = {
    "type": 4,
    "data": {
        "content": "Congrats on sending your command!"
    }
}
r = requests.post(url, json=json)

```

 Interaction `tokens` remain usable for **15 minutes** and can create followup messages, but the initial response **must be sent within 3 seconds of receiving the event**. Missing that 3 second window invalidates the token.

Inline HTTP Response Behavior

For HTTP-delivered interactions, the server may answer the incoming `POST` directly. Return a `200` status code when successful and include `type` plus `data`, forming an [Interaction Response](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-response-object) object:

```
@app.route('/', methods=['POST'])
def my_command():
    if request.json["type"] == 1:
        return jsonify({
            "type": 1
        })

    else:
        return jsonify({
            "type": 4,
            "data": {
                "tts": False,
                "content": "Congrats on sending your command!",
                "embeds": [],
                "allowed_mentions": { "parse": [] }
            }
        })

```

###### Interaction Callback Response Object

| Field | Type | Description |
| --- | --- | --- |
| interaction | interaction callback object | Interaction object linked to the interaction response. |
| resource? | interaction resource object | Resource created by the interaction response. |

###### Interaction Callback Object

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | ID of the interaction |
| type | integer | Interaction type |
| activity_instance_id? | string | Activity instance ID when an Activity was launched or joined |
| response_message_id? | snowflake | Message ID created by the interaction |
| response_message_loading? | boolean | Whether the message is currently loading |
| response_message_ephemeral? | boolean | Whether the response message is ephemeral |

###### Interaction Callback Resource Object

| Field | Type | Description |
| --- | --- | --- |
| type | integer | Interaction callback type |
| activity_instance?* | Activity instance resource | Activity launched by this interaction. |
| message?** | message object | Message created by the interaction. |

 * Present only when type is `LAUNCH_ACTIVITY`. ** Present only when type is `CHANNEL_MESSAGE_WITH_SOURCE` or `UPDATE_MESSAGE`.

###### Interaction Callback Activity Instance Resource

| Field | Type | Description |
| --- | --- | --- |
| id | string | Activity instance ID when an Activity was launched or joined. |

### Followup Messages

 After the initial interaction response, you may need to send followup messages or modify the original response. Whether delivery was through gateway or outgoing webhook, these endpoints handle edits and followups:
- [`PATCH /webhooks/<application_id>/<interaction_token>/messages/@original`](https://docs.discord.com/developers/interactions/receiving-and-responding#edit-original-interaction-response) edits the first response to an Interaction
- [`DELETE /webhooks/<application_id>/<interaction_token>/messages/@original`](https://docs.discord.com/developers/interactions/receiving-and-responding#delete-original-interaction-response) removes the first response to an Interaction
- [`POST /webhooks/<application_id>/<interaction_token>`](https://docs.discord.com/developers/interactions/receiving-and-responding#create-followup-message) creates an additional followup message
- [`PATCH /webhooks/<application_id>/<interaction_token>/messages/<message_id>`](https://docs.discord.com/developers/interactions/receiving-and-responding#edit-followup-message) edits a message sent with that `token`
 Interaction webhooks use the same rate-limit behavior as regular webhooks. Interaction tokens remain valid for **15 minutes**, so responses may be made during that window.

### Endpoints

 The endpoints listed here are excluded from the application's [Global Rate Limit](https://docs.discord.com/developers/topics/rate-limits#global-rate-limit).

### Create Interaction Response

 POST/interactions/[{interaction.id}](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object)/[{interaction.token}](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object)/callback Creates a response for an Interaction. The body is an [interaction response](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-response-object). It returns `204` by default; when `with_response` is `true`, it returns `200` with an [interaction callback response](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-callback-interaction-callback-response-object) body. File attachments are supported in the same style as webhook endpoints. See [Uploading Files](https://docs.discord.com/developers/reference#uploading-files) for upload and `multipart/form-data` details.

###### Query String Params

| Field | Type | Description |
| --- | --- | --- |
| with_response? | boolean | Whether the response should include an interaction callback object |

### Get Original Interaction Response

 GET/webhooks/[{application.id}](https://docs.discord.com/developers/resources/application#application-object)/[{interaction.token}](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object)/messages/@original Returns the original Interaction response. It behaves like [Get Webhook Message](https://docs.discord.com/developers/resources/webhook#get-webhook-message).

### Edit Original Interaction Response

 PATCH/webhooks/[{application.id}](https://docs.discord.com/developers/resources/application#application-object)/[{interaction.token}](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object)/messages/@original Updates the original Interaction response. It behaves like [Edit Webhook Message](https://docs.discord.com/developers/resources/webhook#edit-webhook-message).

### Delete Original Interaction Response
