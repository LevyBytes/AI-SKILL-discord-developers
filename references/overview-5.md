# Interactions — Overview

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Commands, message components, and similar interactive features let users invoke an app directly inside Discord. When someone uses one of those features, the app receives an [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object). This overview explains the main interaction categories and the preparation required before an app can receive them. For detailed interaction handling, see [Receiving and Responding](https://docs.discord.com/developers/interactions/receiving-and-responding).

---

### Types of Interactions

 Apps can use several interaction types, choosing the ones that fit the Discord experience being built.

### Commands

 **[Application commands](https://docs.discord.com/developers/interactions/application-commands)** let users call an app through native Discord UI, usually for core app capabilities. (Figure: Command launcher in the Desktop client) When defining a command, the app chooses a command type; that type controls where Discord shows it and what metadata arrives when a user invokes it. Application commands include these types:
- **Slash commands** are the usual command style, opened by typing `/` in the chat input or using the command picker.
- **Message commands** apply to a message or its content. Users open them from a message's context menu, such as the three-dot menu at the top-right or right-click menu, then the "Apps" section.
- **User commands** apply to a Discord user. Users access them by right-clicking a user profile and opening the "Apps" section.
- **Entry Point commands** are the main launch path for [Activities](https://docs.discord.com/developers/activities/overview) from the App Launcher.
 For command creation and command-interaction handling, use the [Application Commands](https://docs.discord.com/developers/interactions/application-commands) documentation.

### Message Components

 **[Message components](https://docs.discord.com/developers/components/reference)** are interactive controls that can be attached to messages sent by the app in Discord. (Figure: Button message components in a message) The primary interactive message components are:
- [Buttons](https://docs.discord.com/developers/components/reference#button) are clickable controls with configurable styles, text, and emoji.
- [Static select menus](https://docs.discord.com/developers/components/reference#string-select) open a list of select options defined by the developer, including custom labels and descriptions.
- [Auto-populated select menus](https://docs.discord.com/developers/components/reference#string-select) are four select component variants filled from contextual Discord resources, such as users or channels in a server.
 The [Message Components](https://docs.discord.com/developers/components/reference) documentation lists every message component and explains how to send components and receive their interactions.

### Modals

 **[Modals](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-response-object-modal)** are pop-up interfaces shown to a single user for collecting form-style input. An app can open a modal only as a response to a command or message component invoked by a user. (Figure: Modals in the Discord client) Modals may contain only one interactive component type: [text inputs](https://docs.discord.com/developers/components/reference#text-input), which support single-line or multi-line form fields. See [Receiving and Responding](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-response-object-modal) for modal creation and usage details.

---

### Preparing for Interactions

 When someone uses the app, interactions can be delivered to the app in one of two mutually exclusive modes:
- WebSocket-based Gateway connection
- HTTP via outgoing webhooks
 By default, interactions arrive over a Gateway connection. To use HTTP-based delivery instead, add an **Interactions Endpoint URL** in the app settings. Implementation details for handling interactions are in [Receiving and Responding](https://docs.discord.com/developers/interactions/receiving-and-responding).

### Configuring an Interactions Endpoint URL

 An **Interactions Endpoint URL** is a public app endpoint where Discord posts HTTP interactions. Apps that receive interactions through the [Gateway](https://docs.discord.com/developers/events/gateway) do not need to set an Interactions Endpoint URL.

#### Setting Up an Endpoint

 Before saving an Interactions Endpoint URL, make sure the endpoint already supports two required behaviors:
1. Acknowledging `PING` requests from Discord
2. Validate the security request headers (`X-Signature-Ed25519` and `X-Signature-Timestamp`)
 If either requirement is missing, Discord will not validate the Interactions Endpoint URL. The following sections explain `PING` acknowledgements and security-header validation.

#### Acknowledging PING requests

 When you add an Interactions Endpoint URL, Discord sends a `POST` request containing a `PING` payload with `type: 1`. The app must acknowledge it with a `200` response containing a `PONG` payload, also with `type: 1`. For interaction response details, see the [Receiving and Responding documentation](https://docs.discord.com/developers/interactions/receiving-and-responding). Responses to `PING`s must include a valid `Content-Type`; see [here](https://docs.discord.com/developers/reference#http-api) for more information.

Responding to PING Requests

Code example for acknowledging `PING` interactions

A correct `PING` acknowledgement returns a `200` response and a body with `type: 1`:

```
@app.route('/', methods=['POST'])
def my_command():
    if request.json["type"] == 1:
        return jsonify({
            "type": 1
        })

```

###### Validating Security Request Headers

 Public unauthenticated endpoints need strict security handling. Before an app can receive HTTP interactions, it **must** implement the required validation steps. Every interaction request includes these headers:
- `X-Signature-Ed25519` containing the signature
- `X-Signature-Timestamp` containing the timestamp
 Use an appropriate security library and **validate the request every time an [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object) is received**. When signature validation fails, respond with a `401` error code.

Validating Security Headers

Code example for validating security request headers

The examples below demonstrate validation for interaction request headers.**JavaScript**

```
const nacl = require("tweetnacl");

// Your public key can be found on your application in the Developer Portal
const PUBLIC_KEY = "APPLICATION_PUBLIC_KEY";

const signature = req.get("X-Signature-Ed25519");
const timestamp = req.get("X-Signature-Timestamp");
const body = req.rawBody; // rawBody is expected to be a string, not raw bytes

const isVerified = nacl.sign.detached.verify(
    Buffer.from(timestamp + body),
    Buffer.from(signature, "hex"),
    Buffer.from(PUBLIC_KEY, "hex")
);

if (!isVerified) {
    return res.status(401).end("invalid request signature");
}

```

**Python**

```
from nacl.signing import VerifyKey
from nacl.exceptions import BadSignatureError

# Your public key can be found on your application in the Developer Portal
PUBLIC_KEY = 'APPLICATION_PUBLIC_KEY'

verify_key = VerifyKey(bytes.fromhex(PUBLIC_KEY))

signature = request.headers["X-Signature-Ed25519"]
timestamp = request.headers["X-Signature-Timestamp"]
body = request.data.decode("utf-8")

try:
    verify_key.verify(f'{timestamp}{body}'.encode(), bytes.fromhex(signature))
except BadSignatureError:
    abort(401, 'invalid request signature')

```

 Discord checks signature-header validation both when the endpoint is saved and through ongoing automated security probes, including deliberately invalid signatures. If validation fails, Discord removes the interactions URL and notifies you by email and System DM. The [Community Resources](https://docs.discord.com/developers/developer-tools/community-resources#interactions) page and its libraries are strongly recommended; many provide typings for Interactions data models and decorators for frameworks such as Flask and Express that simplify validation.

#### Adding an Interactions Endpoint URL

 Once a public endpoint is ready to serve as the app's Interactions Endpoint URL, add it in the [app's settings](https://discord.com/developers/applications). On [**General Information**](https://discord.com/developers/applications/select/information), find **Interactive Endpoint URL**. Paste the public URL that acknowledges `PING` messages and validates the required signature headers.

---

### Handling Interactions

 After the app is ready for interactions, continue with [Receiving and Responding](https://docs.discord.com/developers/interactions/receiving-and-responding) for the technical details of processing interaction requests.
