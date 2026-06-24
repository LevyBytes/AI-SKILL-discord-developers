# Integrate Moderation

> Discord Developers reference. Original prose; identifiers preserved verbatim.

This guide explains how to add and operate content moderation for a game that uses Discord Social SDK.

### Overview

 Healthy social features require effective moderation. This guide covers how to:
- Better understand your moderation responsibilities
- Add server-side moderation for text by using Discord's moderation metadata API.
- Add client-side audio moderation with the Discord Social SDK.

### Prerequisites

 Before starting, confirm that you have:
- Basic knowledge of SDK behavior from the [Getting Started Guide](https://docs.discord.com/developers/discord-social-sdk/getting-started).
- Basic knowledge of the communication features in your game.
- Familiarity with [provisional accounts](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts).
- Reviewed the [Discord Social SDK Terms](https://support-dev.discord.com/hc/en-us/articles/30225844245271-Discord-Social-SDK-Terms).

### Your Moderation Responsibilities

### Moderation on Discord

 [Discord's Community Guidelines](https://discord.com/guidelines) and [Terms of Service](https://discord.com/terms) govern content rendered on Discord, including:
- Text messages, audio, and video transmitted inside Discord's platform.
- Text messages shown on Discord, such as DMs or Linked Channels, after players send them in your game, whether those players linked a Discord account or use a provisional account.
 Discord may respond to violating content on Discord with actions such as platform-wide account bans or restrictions. Enforcement against a Discord account does not alter the player's separate game account; however, a banned Discord account can no longer use SDK features that require account connection.

### Game Developer’s Responsibility

 Your own terms and policies govern in-game content. You are responsible for:
- Compliance with the [Discord Social SDK Terms](https://support-dev.discord.com/hc/en-us/articles/30225844245271-Discord-Social-SDK-Terms).
- Defining and enforcing content policies specific to your game.
- Moderating messages or audio that occur inside the game.
- Providing suitable reporting and moderation UIs, including a way for players to report issues or policy violations and a process for reviewing those reports and acting on them.
 You are also responsible for third-party moderation tools or services used by the game. That includes complying with applicable terms and laws and obtaining any required player consents for processing data through those moderation services.

### Server-Side Chat Moderation

 The moderation metadata API lets a backend evaluate messages and write application-scoped `moderation_metadata` onto them. Discord stores that metadata and sends it to active game sessions through realtime `GAME_DIRECT_MESSAGE_UPDATE` or `LOBBY_MESSAGE_UPDATE` [Webhook Events](https://docs.discord.com/developers/events/webhook-events), so polling is unnecessary. Other applications cannot see the metadata, and it is excluded from other API endpoints and Gateway events.

### How It Works

1. A webhook event tells your backend that a message was created or changed.
2. The moderation service reviews the message and uses the Discord API to update `moderation_metadata`, specifying the client action such as hide, blur, or replace.
3. Discord saves the metadata and sends an update event to relevant game clients through the Social SDK.
4. The game client receives the update through [`Client::SetMessageUpdatedCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#aa01cf3c15403f29780dabfcfaf3b1dcd), reads [`MessageHandle::ModerationMetadata`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1MessageHandle.html#afb9ae126f6b1f0de006f2be8e3688205) from the relevant [`MessageHandle`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1MessageHandle.html#ae25595b43bc74b0c4c92c5165d16382f) in that callback, and renders the message based on the metadata.
5. When message content changes, Discord removes existing moderation metadata before sending the next update, signaling that the edited content must be moderated again.
  The sequence diagram uses direct messages, but lobby messages follow the same pattern. The sections below identify the corresponding Social SDK methods, Webhook Event types, and API paths.

### Webhook Events

 First, [subscribe to the following events](https://docs.discord.com/developers/events/webhook-events#subscribing-to-events) on the application's webhook so messages arrive for moderation:
| Event | Description |
| --- | --- |
| `GAME_DIRECT_MESSAGE_CREATE` | Emitted when a DM is created during a Social SDK session. |
| `GAME_DIRECT_MESSAGE_UPDATE` | Emitted when a DM changes because its content was edited. |
| `LOBBY_MESSAGE_CREATE` | Emitted when a lobby receives a new message. |
| `LOBBY_MESSAGE_UPDATE` | Emitted when an existing lobby message is edited. |

 Use the [Webhook Events reference](https://docs.discord.com/developers/events/webhook-events) for complete event schemas. Metadata-only changes do **not** emit webhook events; active game sessions receive them only through realtime SDK callbacks.

### Applying Moderation Decisions

 After the moderation backend receives and evaluates a message, write the decision to `moderation_metadata` on that message so the game client knows how to display the content.

#### DM Messages

 `user_id_1` and `user_id_2` identify the DM participants, and either order is valid.

```
import requests

API_ENDPOINT = 'https://discord.com/api/v10'
BOT_TOKEN = 'YOUR_BOT_TOKEN'

def apply_dm_moderation(user_id_1, user_id_2, message_id, metadata):
  r = requests.put(
    f'{API_ENDPOINT}/partner-sdk/dms/{user_id_1}/{user_id_2}/messages/{message_id}/moderation-metadata',
    headers={
      'Authorization': f'Bot {BOT_TOKEN}',
      'Content-Type': 'application/json',
    },
    json=metadata
  )
  r.raise_for_status()

# Instruct the client to hide this message — moderation service flagged it as toxic
metadata = {
  'action': 'hide',   # client will not render this message
  'reason': 'toxicity',  # logged by the client for reporting purposes
}
apply_dm_moderation(user_id_1, user_id_2, message_id, metadata)

# Alternatively, instruct the client to show this message — moderation service approved it
metadata = {
  'action': 'show',  # client will render the message normally
}
apply_dm_moderation(user_id_1, user_id_2, message_id, metadata)

```

#### Lobby Messages

```
import requests

API_ENDPOINT = 'https://discord.com/api/v10'
BOT_TOKEN = 'YOUR_BOT_TOKEN'

def apply_lobby_moderation(lobby_id, message_id, metadata):
  r = requests.put(
    f'{API_ENDPOINT}/lobbies/{lobby_id}/messages/{message_id}/moderation-metadata',
    headers={
      'Authorization': f'Bot {BOT_TOKEN}',
      'Content-Type': 'application/json',
    },
    json=metadata
  )
  r.raise_for_status()

# Instruct the client to replace the message content with a policy reminder
metadata = {
  'action': 'replace',  # client will display replacement text instead of the original
  'replacement': 'Be kind to others!',  # the text the client will render in place of the message
}
apply_lobby_moderation(lobby_id, message_id, metadata)

```

 Both endpoints succeed with `HTTP 204: No Content`.

### Moderation Metadata Fields

 The metadata body is an unrestricted key-value map. Choose keys understood by your client. Common patterns include:
| Key | Example values | Purpose |
| --- | --- | --- |
| `action` | `hide`, `show`, `blur`, `replace` | Rendering instruction for the client. |
| `reason` | `toxicity`, `spam` | Reason for the moderation result, useful for logs. |
| `replacement` | any string | Replacement text to show instead of the original message. |
| `severity` | `low`, `medium`, `high` | Optional severity classification |

 **Limits:** Each message can have up to 5 keys; keys may be no more than 1024 characters; values may be no more than 2000 characters, matching the maximum Discord message length and allowing modified message content. Moderation metadata is scoped to the application, so only your application can read it, even when the message is visible in Discord.

### Handling Moderation Metadata on the Client

 Use [`Client::SetMessageUpdatedCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#aa01cf3c15403f29780dabfcfaf3b1dcd) to receive metadata changes. Read the backend-written key-value map through [`MessageHandle::ModerationMetadata`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1MessageHandle.html#afb9ae126f6b1f0de006f2be8e3688205).

```
client->SetMessageUpdatedCallback([&client](uint64_t messageId) {
    if (auto message = client->GetMessageHandle(messageId)) {
        auto metadata = message->ModerationMetadata();
        if (metadata.empty()) {
            // No moderation decision yet — render as pending, normally, or hide completely,
            // depending on your moderation requirements
            renderPendingModeration(message);
            return;
        }
        auto action = metadata.count("action") ? metadata.at("action") : "show";
        if (action == "hide") {
            hideMessage(messageId);
        } else if (action == "show") {
            renderMessage(message);
        } else if (action == "replace") {
            renderMessage(messageId, metadata.at("replacement"));
        } else if (action == "blur") {
            renderBlurred(message);
        } else {
            std::cerr << "Unknown moderation action: " << action << "\n";
            renderMessage(message);  // fall back to rendering normally
        }
    }
});

```

 While moderation is pending, consider withholding new messages or showing a "pending moderation" state so unreviewed content is not briefly displayed.

### Handling Content Edits

 When a message is edited, Discord clears its stored `moderation_metadata` automatically and sends a new update notification. [`Client::SetMessageUpdatedCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#aa01cf3c15403f29780dabfcfaf3b1dcd) runs with an empty metadata map for that edited message, and the backend receives a `GAME_DIRECT_MESSAGE_UPDATE` or `LOBBY_MESSAGE_UPDATE` webhook event for the edit. Use that event to start moderation for the new content. During that re-moderation window, avoid rendering the updated message or show it as "pending moderation" so unmoderated content does not flash on screen.

### Handling Users with Banned Discord Accounts

### Discord Platform Bans

 If Discord bans a player's Discord account, temporarily or permanently, the player can no longer authenticate to the app through Discord, the current [`Client`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a91716140c699d8ef0bdf6bfd7ee0ae13) disconnects, OAuth2 tokens are invalidated, and an [`APPLICATION_DEAUTHORIZED`](https://docs.discord.com/developers/events/webhook-events#application-deauthorized) webhook is sent. Mechanically, the ban is a Discord-initiated [unmerge](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#unmerging-provisional-accounts): OAuth2 tokens are removed, and a new cross-platform-restricted provisional account replaces the linked Discord identity. The full lifecycle is documented in [Ban-Driven Unmerge](https://docs.discord.com/developers/discord-social-sdk/development-guides/using-provisional-accounts#ban-driven-unmerge) in the Provisional Accounts guide. Auth signals and integration architecture are covered in [Out-of-Band Revocation](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord#out-of-band-revocation) in the Account Linking guide.

### Discord Server Bans

 To connect in-game moderation rules with a Discord server you control, such as an official community server, retrieve Discord Server ban data through the REST APIs. For details on listing all guild bans or reading the ban for one guild user, see [`{guild.id}/guilds/{guild.id}/bans`](https://docs.discord.com/developers/resources/guild#get-guild-bans) and [`/guilds/{guild.id}/bans/{user.id}`](https://docs.discord.com/developers/resources/guild#get-guild-ban).

### Voice Chat Moderation

 Discord Social SDK exposes audio streams for in-game voice calls, enabling voice-chat moderation. Call data is available through [`Client::StartCallWithAudioCallbacks`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#abcaa891769f9e912bfa0e06ff7221b05) and can be forwarded to your audio moderation system.

```
// Example: Capturing local voice chat audio for asynchronous moderation.

// Callback for local users' audio with moderation
auto capturedCallback = [](int16_t const* data,
                     uint64_t samplesPerChannel, int32_t sampleRate,
                     uint64_t channels) {
    // Call the moderation function
    moderateCapturedVoice(data, samplesPerChannel);
};

// Start the call with our moderation callback
auto call = client->StartCallWithAudioCallbacks(lobbyId, receivedCallback, capturedCallback);

```

---

### Next Steps

### Sending Direct Messages

Enable private messaging between players.

### Managing Voice Chat

Add in-game voice communication.

### Linked Channels

Link game lobbies with Discord text channels. Need help? Join the [Discord Developers Server](https://discord.gg/discord-developers) and ask in `#social-sdk-dev-help` for community support. Bugs found while using the Social SDK should be reported at [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report).

---

### Change Log

| Date | Changes |
| --- | --- |
| May 22, 2026 | Move platform-ban lifecycle information into the Account Linking guide. |
| Feb 20, 2026 | Swapped client-side message moderation for server-side message moderation. |
| May 22, 2025 | initial release |
