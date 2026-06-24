# Webhook Events

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Webhook events are one-way HTTP notifications sent to your app after something happens in Discord. They differ from events [sent over Gateway connections](https://docs.discord.com/developers/events/gateway) because webhook delivery is not realtime and ordering is not guaranteed. [Incoming webhooks](https://docs.discord.com/developers/resources/webhook) are initiated by outside services, while webhook events, or outgoing webhooks, come from Discord activity. Your app therefore needs a public HTTP URL for receiving events, described under [preparing for events](https://docs.discord.com/developers/events/webhook-events#preparing-for-events).

### Subscribing to Events

 To use webhook events, configure the receiving URL and choose which events the app should get. The steps below describe setup in the developer portal. API-based setup can use [Edit Current Application](https://docs.discord.com/developers/resources/application#edit-current-application). In your [app's settings](https://discord.com/developers/applications), open the [**Webhooks** page](https://discord.com/developers/applications/select/webhooks) from the left sidebar and complete these steps:
1. Under **Endpoint**, enter a public URL that can receive webhook events and acknowledge them. The [preparing for events](https://docs.discord.com/developers/events/webhook-events#preparing-for-events) section explains how to build a Webhook Events URL.
2. Turn on Events with the toggle in the **Events** section.
3. Choose the [webhook events](https://docs.discord.com/developers/events/webhook-events#event-types) the app should receive.
4. Click **Save Changes**.
 After the URL verifies successfully, Discord begins sending the selected events to the app.

### Preparing for Events

 Receiving webhook events requires setting the app's **Webhook Event URL** in the app settings.

### Configuring a Webhook Events URL

 A **Webhook Events URL** is the public endpoint where Discord sends HTTP events for your app. Apps that use [Gateway events](https://docs.discord.com/developers/events/gateway) do not need a Webhook Events URL.

#### Setting Up an Endpoint

 Before adding a Webhook Events URL to the app, prepare the endpoint for two requirements:
1. Acknowledging `PING` events from Discord
2. Check the security headers `X-Signature-Ed25519` and `X-Signature-Timestamp`
 If either requirement is missing, Discord will not validate the Webhook Events URL. The sections below cover acknowledging PING events and checking security headers.

###### Acknowledging PING requests

 When you add the Webhook Events URL, Discord sends a `POST` request containing a `PING` payload with `type: 0` to the endpoint. The app must acknowledge it by returning a `204` response and no body. Responses to `PING`s must include a valid `Content-Type`. See [here](https://docs.discord.com/developers/reference#http-api) for more information.

Responding to PING Requests

Example code for acknowledging PING events

A correct `PING` acknowledgement returns `204` with an empty body:

```
@app.route('/', methods=['POST'])
def my_command():
    if request.json["type"] == 0:
        return Response(status=204)

```

###### Validating Security Request Headers

 Before an app can receive HTTP events, it **must** satisfy several security requirements. Each webhook request includes these headers:
- `X-Signature-Ed25519` containing the signature
- `X-Signature-Timestamp` containing the timestamp
 Use a trusted security library to **validate the request each time you receive an event**. If signature validation fails, respond with a `401` error code. Security-header validation examples are available in the [Interactions documentation](https://docs.discord.com/developers/interactions/overview#setting-up-an-endpoint-validating-security-request-headers). Discord checks this when you save the endpoint and continues to run automated security probes, including requests with deliberately invalid signatures. Failed validation causes Discord to remove the Webhook Events URL and notify you by email and System DM. Discord recommends the [Community Resources](https://docs.discord.com/developers/developer-tools/community-resources) and the libraries listed there.

#### Adding an Webhook Events Endpoint URL

 After you have a public endpoint for the app's Event Webhooks URL, add it from your [app's settings](https://discord.com/developers/applications). On the [**Webhooks** page](https://discord.com/developers/applications/select/webhooks), find **Endpoint URL**. Paste a public URL that acknowledges `PING` messages and validates security signature headers. Once the Webhook Events URL is configured, use the same page to [enable and subscribe to events](https://docs.discord.com/developers/events/webhook-events#subscribing-to-events).

### Responding to Events

 When the Webhook Event URL receives an event, the app should acknowledge receipt by returning a `204` status code with no body **within 3 seconds**. If the app does not respond, Discord retries delivery several times with exponential backoff for up to 10 minutes. Repeated response failures cause Discord to stop sending webhook events and notify you by email.

### Webhook Event Payloads

 Webhook events use an outer payload that contains an inner `event` object.

### Payload Structure

 Outer webhook payload structure
| Field | Type | Description |
| --- | --- | --- |
| version | integer | Webhook event version scheme. Currently always `1` |
| application_id | snowflake | ID of your app |
| type | webhook type | Webhook type: `0` for PING or `1` for webhook events |
| event? | event body object | Event data payload |

#### Webhook Types

| Type | Value | Description |
| --- | --- | --- |
| PING | `0` | PING event used to confirm the Webhook Event URL is active |
| Event | `1` | Webhook event, with details stored in the event body object |

#### Event Body Object

 The event body carries top-level event information such as type and trigger time. The inner `data` object holds fields specific to the [event type](https://docs.discord.com/developers/events/webhook-events#event-types).
| Field | Type | Description |
| --- | --- | --- |
| type | string | Event type |
| timestamp | string | Timestamp for when the event occurred, in ISO8601 format |
| data? | object | Event data. Its structure is determined by the event type |

### Event Types

 The table lists the webhook event types an app can subscribe to. The "Value" column maps to the event body's `type` field value in the [event body object](https://docs.discord.com/developers/events/webhook-events#event-body-object).
| Name | Value | Description |
| --- | --- | --- |
| Application Authorized | `APPLICATION_AUTHORIZED` | Sent after a user authorizes an app for a server or account |
| Application Deauthorized | `APPLICATION_DEAUTHORIZED` | Sent after a user deauthorizes an app |
| Entitlement Create | `ENTITLEMENT_CREATE` | Entitlement was created |
| Entitlement Update | `ENTITLEMENT_UPDATE` | Entitlement was updated |
| Entitlement Delete | `ENTITLEMENT_DELETE` | Entitlement was deleted |
| Quest User Enrollment | `QUEST_USER_ENROLLMENT` | User joined a Quest (currently unavailable) |
| Lobby Message Create | `LOBBY_MESSAGE_CREATE` | Sent when a lobby message is created |
| Lobby Message Update | `LOBBY_MESSAGE_UPDATE` | Sent when a lobby message is updated |
| Lobby Message Delete | `LOBBY_MESSAGE_DELETE` | Sent when a lobby message is deleted |
| Game Direct Message Create | `GAME_DIRECT_MESSAGE_CREATE` | Reports creation of a game DM tied to an active Social SDK session |
| Game Direct Message Update | `GAME_DIRECT_MESSAGE_UPDATE` | Reports edits to a game DM tied to an active Social SDK session |
| Game Direct Message Delete | `GAME_DIRECT_MESSAGE_DELETE` | Reports removal of a game DM tied to an active Social SDK session |

#### Application Authorized

 `APPLICATION_AUTHORIZED` fires when the app is added to a server or to a user account.

###### Application Authorized Structure

| Field | Type | Description |
| --- | --- | --- |
| integration_type? | integer | Authorization installation context: guild (`0`) when installed to a server, or user (`1`) when installed to a user's account |
| user | user object | User who authorized the app |
| scopes | array of strings | Scopes authorized by the user |
| guild? | guild object | Server the app was authorized for, present when integration type is `0` |

###### Application Authorized Example

```
{
  "version": 1,
  "application_id": "1234560123453231555",
  "type": 1,
  "event": {
    "type": "APPLICATION_AUTHORIZED",
    "timestamp": "2024-10-18T14:42:53.064834",
    "data": {
      "integration_type": 1,
      "scopes": [
        "applications.commands"
      ],
      "user": {
        // user data
      }
    }
  }
}

```

#### Application Deauthorized

 `APPLICATION_DEAUTHORIZED` fires when a user deauthorizes the app.

###### Application Deauthorized Structure

| Field | Type | Description |
| --- | --- | --- |
| user | user object | User who deauthorized the app |

###### Application Deauthorized Example

```
{
  "version": 1,
  "application_id": "1234560123453231555",
  "type": 1,
  "event": {
    "type": "APPLICATION_DEAUTHORIZED",
    "timestamp": "2024-10-18T14:42:53.064834",
    "data": {
      "user": {
        // user data
      }
    }
  }
}

```

###### Discord Social SDK

 For [Discord Social SDK](https://docs.discord.com/developers/discord-social-sdk) apps, `APPLICATION_DEAUTHORIZED` is the required webhook for learning outside the game that a user's linked-account state changed. Listen for it so application state stays synchronized; if the webhook is missed, the [next token use will fail](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord#when-refresh-fails) and the game should return to the [provisional account flow](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#implementing-provisional-accounts). Each revocation that emits this event is mechanically an [unmerge](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#unmerging-provisional-accounts): the merged Discord account becomes a new provisional account, and the Discord user's OAuth2 tokens are invalidated. Triggers include:
- Token revocations through [`/oauth2/token/revoke`](https://docs.discord.com/developers/topics/oauth2#authorization-code-grant) or `Client::RevokeToken`.
- The user removing the app from *User Settings -> Authorized Apps* in Discord.
- The [Social SDK unmerge endpoints](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#unmerging-provisional-accounts).
- The user's Discord [account being banned](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#ban-driven-unmerge).

#### Entitlement Create

 `ENTITLEMENT_CREATE` fires when an [entitlement](https://docs.discord.com/developers/resources/entitlement) is created because a user purchased or otherwise received one of the app's SKUs. See the [Monetization documentation](https://docs.discord.com/developers/monetization/overview) for details.

###### Entitlement Create Structure

 The inner payload is an [entitlement](https://docs.discord.com/developers/resources/entitlement#entitlement-object) object.

###### Entitlement Create Example

```
{
  "version": 1,
  "application_id": "1234560123453231555",
  "type": 1,
  "event": {
    "type": "ENTITLEMENT_CREATE",
    "timestamp": "2024-10-18T18:41:21.109604",
    "data": {
      "application_id": "1234560123453231555",
      "consumed": false,
      "deleted": false,
      "gift_code_flags": 0,
      "id": "1234505980407808808",
      "promotion_id": null,
      "sku_id": "123489045643835123",
      "type": 4,
      "user_id": "111178765189277770"
    }
  }
}

```

#### Entitlement Update

 `ENTITLEMENT_UPDATE` fires when an [entitlement](https://docs.discord.com/developers/resources/entitlement) changes. See the [Monetization documentation](https://docs.discord.com/developers/monetization/overview) for details.

###### Entitlement Update Structure

 The inner payload is an [entitlement](https://docs.discord.com/developers/resources/entitlement#entitlement-object) object.

###### Entitlement Update Example

```
{
  "version": 1,
  "application_id": "1234560123453231555",
  "type": 1,
  "event": {
    "type": "ENTITLEMENT_UPDATE",
    "timestamp": "2024-10-18T18:41:21.109604",
    "data": {
      "application_id": "1234560123453231555",
      "consumed": false,
      "deleted": false,
      "gift_code_flags": 0,
      "id": "1234505980407808808",
      "promotion_id": null,
      "sku_id": "123489045643835123",
      "type": 4,
      "user_id": "111178765189277770"
    }
  }
}

```

#### Entitlement Delete

 `ENTITLEMENT_DELETE` fires when an [entitlement](https://docs.discord.com/developers/resources/entitlement) is deleted. See the [Monetization documentation](https://docs.discord.com/developers/monetization/overview) for details.

###### Entitlement Delete Structure

 The inner payload is an [entitlement](https://docs.discord.com/developers/resources/entitlement#entitlement-object) object.

###### Entitlement Delete Example

```
{
  "version": 1,
  "application_id": "1234560123453231555",
  "type": 1,
  "event": {
    "type": "ENTITLEMENT_DELETE",
    "timestamp": "2024-10-18T18:41:21.109604",
    "data": {
      "application_id": "1234560123453231555",
      "consumed": false,
      "deleted": true,
      "gift_code_flags": 0,
      "id": "1234505980407808808",
      "promotion_id": null,
      "sku_id": "123489045643835123",
      "type": 4,
      "user_id": "111178765189277770"
    }
  }
}

```

#### Quest User Enrollment

 Apps cannot receive this event at this time. It is documented because it appears on the Webhooks settings page. `QUEST_USER_ENROLLMENT` fires when a user is added to a Quest on Discord.

#### Lobby Message Create

 `LOBBY_MESSAGE_CREATE` fires when a lobby message is created.

###### Lobby Message Create Structure

 The inner payload is a [lobby message object](https://docs.discord.com/developers/events/webhook-events#lobby-message-object).

###### Lobby Message Create Example

```
{
  "version": 1,
  "application_id": "1234567765431056709",
  "type": 1,
  "event": {
    "type": "LOBBY_MESSAGE_CREATE",
    "timestamp": "2024-10-18T18:41:21.109604",
    "data": {
      "id": "1397729799727878254",
      "type": 0,
      "content": "welcome to the party!",
      "lobby_id": "1397729744753266719",
      "channel_id": "1397729744753266719",
      "author": {
        // user data
      },
      "flags": 65536,
      "application_id": "1234567765431056709"
    }
  }
}

```

#### Lobby Message Update

 `LOBBY_MESSAGE_UPDATE` fires when a lobby message is updated.

###### Lobby Message Update Structure

 The inner payload is a [lobby message object](https://docs.discord.com/developers/events/webhook-events#lobby-message-object) plus fields used for message updates.

###### Lobby Message Update Example

```
{
  "version": 1,
  "application_id": "1234567765431056709",
  "type": 1,
  "event": {
    "type": "LOBBY_MESSAGE_UPDATE",
    "timestamp": "2025-08-05T20:39:19.587872",
    "data": {
      "id": "1402390388030832792",
      "type": 0,
      "content": "noice",
      "lobby_id": "1402385687281537066",
      "channel_id": "1402389638311841883",
      "author": {
        // user data
      },
      "edited_timestamp": "2025-08-05T20:39:19.557970+00:00",
      "flags": 0,
      "timestamp": "2025-08-05T20:38:43.660000+00:00"
    }
  }
}

```

#### Lobby Message Delete

 `LOBBY_MESSAGE_DELETE` fires when a lobby message is deleted.

###### Lobby Message Delete Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | ID of the deleted message |
| lobby_id | snowflake | ID of the lobby where deletion occurred |

###### Lobby Message Delete Example

```
{
  "version": 1,
  "application_id": "1234567765431056709",
  "type": 1,
  "event": {
    "type": "LOBBY_MESSAGE_DELETE",
    "timestamp": "2025-08-05T21:44:09.412957",
    "data": {
      "id": "1402406637632884857",
      "lobby_id": "1402399883394285659"
    }
  }
}

```

#### Game Direct Message Create

 `GAME_DIRECT_MESSAGE_CREATE` fires after a game DM is created and at least one participant currently has an active Social SDK session.

###### Game Direct Message Create Structure

 The inner payload is a [message object](https://docs.discord.com/developers/events/webhook-events#message-object) or an [SDK DM message object](https://docs.discord.com/developers/events/webhook-events#sdk-dm-message-object).

###### Game Direct Message Create Example

```
{
  "version": 1,
  "application_id": "1234567765431056709",
  "type": 1,
  "event": {
    "type": "GAME_DIRECT_MESSAGE_CREATE",
    "timestamp": "2025-08-14T18:09:38.063234",
    "data": {
      "id": "1405614357781545021",
      "type": 0,
      "content": "get in friend, we're going raiding",
      "channel_id": "1405604229820715098",
      "author": {
        // user data
      },
      "timestamp": "2025-08-14T18:09:37.947000+00:00",
      "application_id": "1234567765431056709",
      "attachments": []
    }
  }
}

```

#### Game Direct Message Update

 `GAME_DIRECT_MESSAGE_UPDATE` fires after a game DM changes and at least one participant currently has an active Social SDK session.

###### Game Direct Message Update Structure

 The inner payload is a [message object](https://docs.discord.com/developers/events/webhook-events#message-object) or an [SDK DM message object](https://docs.discord.com/developers/events/webhook-events#sdk-dm-message-object).

###### Game Direct Message Update Example

```
{
  "version": 1,
  "application_id": "1234567765431056709",
  "type": 1,
  "event": {
    "type": "GAME_DIRECT_MESSAGE_UPDATE",
    "timestamp": "2025-08-14T16:44:31.847073",
    "data": {
      "id": "1405591838810706081",
      "content": "almost ready to queue?",
      "channel_id": "1404960877324533784",
      "author": {
        // user data
      },
      "recipient_id": "1404960877324533784"
    }
  }
}

```

#### Game Direct Message Delete

 `GAME_DIRECT_MESSAGE_DELETE` fires after a game DM is removed and at least one participant currently has an active Social SDK session.

###### Game Direct Message Delete Structure

 The inner payload is a [message object](https://docs.discord.com/developers/events/webhook-events#message-object) or an [SDK DM message object](https://docs.discord.com/developers/events/webhook-events#sdk-dm-message-object).

###### Game Direct Message Delete Example

```
{
  "version": 1,
  "application_id": "1234567765431056709",
  "type": 1,
  "event": {
    "type": "GAME_DIRECT_MESSAGE_DELETE",
    "timestamp": "2025-08-20T17:01:50.099204",
    "data": {
      "id": "1407771600643686503",
      "type": 0,
      "content": "cant make it in time",
      "channel_id": "1405604229820715098",
      "author": {
        // user data
      },
      "timestamp": "2025-08-20T17:01:44.725000+00:00",
      "flags": 0,
      "attachments": [],
      "components": []
    }
  }
}

```

### Social SDK Message Objects

 Discord Social SDK uses specialized message objects for lobby and direct-message events during active game sessions. These objects adapt the standard Discord message structure for in-game communication features.
- [Lobby messages](https://docs.discord.com/developers/events/webhook-events#lobby-message-object) include lobby-specific fields like `lobby_id`
- [Standard Discord messages](https://docs.discord.com/developers/events/webhook-events#message-object) can include extra fields in SDK contexts
- [SDK DM messages](https://docs.discord.com/developers/events/webhook-events#sdk-dm-message-object) handle communication between provisional accounts
 Webhook events under `LOBBY_MESSAGE_*` and `GAME_DIRECT_MESSAGE_*` use these objects according to the messaging context.

### Lobby Message Object

 Represents a message sent in a lobby or [Linked Channel](https://docs.discord.com/developers/discord-social-sdk/development-guides/linked-channels).

###### Lobby Message Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | ID of the message |
| type | integer | Type of message |
| content | string | Contents of the message |
| lobby_id | snowflake | ID of the lobby that received the message |
| channel_id | snowflake | ID of the channel where the message was sent |
| author | user object | Author of the message |
| metadata? | object | Extra message metadata as key-value pairs |
| flags | integer | Message flags represented as a bitfield |
| application_id? | snowflake | ID of the application, present only during active Social SDK sessions |

### Message Object

 Standard [Message Object](https://docs.discord.com/developers/resources/message#message-object) with extra fields.

###### Additional Fields

| Field | Type | Description |
| --- | --- | --- |
| lobby_id? | snowflake | ID of the lobby where the message was created, present only for Linked Channel messages |
| channel | channel object | Channel object with recipient information |

### SDK DM Message Object

 Represents an in-game-only message between provisional users.

###### SDK DM Message Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | ID of the message |
| type | integer | Type of message |
| content | string | Contents of the message |
| author | user object | Author of the message |
| flags | integer | Message flags represented as a bitfield |
| application_id | snowflake | ID of the application that created the message |
| channel | channel object | Channel object with recipient information |
| activity? | message activity object | Included with Rich Presence-related chat embeds |
| application? | partial application object | Included with Rich Presence-related chat embeds |

 When both direct-message participants are provisional accounts, the messages are "SDK DM messages" that only appear in-game and follow this specialized structure.
