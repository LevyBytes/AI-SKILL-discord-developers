# Sending Direct Messages

> Discord Developers reference. Original prose; identifiers preserved verbatim.

This feature is available now, but rate limits apply. For higher production limits for a game, follow [Communication Features: Applying for Increased Rate Limits for Production Releases](https://docs.discord.com/developers/discord-social-sdk/core-concepts/communication-features#applying-for-increased-rate-limits-for-production-releases).

### Overview

 Direct Messages (DMs) provide private player-to-player communication. This guide explains how to:
- Send text messages between users
- Display messages inside the game
- Retrieve conversation history and summaries

### Prerequisites

 Before starting, confirm that you have:
- Configured the Discord Social SDK by following the [Getting Started guide](https://docs.discord.com/developers/discord-social-sdk/getting-started)
 This communication feature requires enabling [`Client::GetDefaultCommunicationScopes`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a71499da752fbdc2d4326ae0fd36c0dd1) in the OAuth Scope configuration. The [OAuth Scopes Core Concepts Guide](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes) has the related scope details.

### Types of Chat Messages

 The Discord Social SDK supports two chat forms:
- [Direct messages (DMs) between two users](https://docs.discord.com/developers/discord-social-sdk/development-guides/sending-direct-messages#sending-a-direct-message-to-a-user)
- [Chat messages within a lobby](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-lobbies#sending-messages-to-a-lobby)
 Messages arrive through these SDK channel types:
- DM
- Ephemeral DM
- Lobby
 Begin with sending a direct message to another user.

---

### Sending a Direct Message to a User

 The SDK can send a message as the user, but only in direct response to an action the user took. Do not send messages automatically. To send a direct message, provide the recipient's Discord ID and the message text.

```
std::string message = "ready to queue?";
uint64_t recipientId = 1234567890; // The recipient's Discord ID

client->SendUserMessage(recipientId, message, [](auto result, uint64_t messageId) {
  if (result.Successful()) {
    std::cout << "✅ Message sent successfully\n";
  } else {
    std::cout << "❌ Failed to send message: " << result.Error() << "\n";
  }
});

```

### Syncing Messages with Discord

 Some Social SDK messages from the game are also shown in Discord. That synchronization occurs for:
- 1 on 1 chat where at least one participant is a full Discord user
- Lobby chat when that lobby is [linked to a Discord channel](https://docs.discord.com/developers/discord-social-sdk/development-guides/linked-channels)
- Messages sent by users who are not banned on Discord.
 For messages between provisional accounts or users who are not friends, the channel ID and recipient ID are both set to the other user's ID. These messages are ephemeral and are not stored in a channel, so a [`ChannelHandle`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1ChannelHandle.html#ac32096b2ef15c5c220e9b7b92253cc46) cannot be resolved for them.

---

### Receiving and Rendering Messages

 Register [`Client::SetMessageCreatedCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a28325a8e8c688a84ac851da4bc86e148) to react when the SDK receives a new message. Inside the callback, read message content and sender ID from the [`MessageHandle`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1MessageHandle.html#ae25595b43bc74b0c4c92c5165d16382f). A [`MessageHandle`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1MessageHandle.html#ae25595b43bc74b0c4c92c5165d16382f) represents one message delivered to the SDK.

```
client->SetMessageCreatedCallback([&client](uint64_t messageId) {
  if (auto message = client->GetMessageHandle(messageId)) {
    std::cout << "New message from " << message->AuthorId() << ": " << message->Content() << "\n";
  }
});

```

 The SDK also exposes callbacks for message edits and deletions. Register them with [`Client::SetMessageUpdatedCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#aa01cf3c15403f29780dabfcfaf3b1dcd) and [`Client::SetMessageDeletedCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a2b6079eded10bea29abbb9695702637b).

### Suppressing Double Notifications

 If the Discord desktop app is open on the same computer as the game, Discord may play notifications even though the messages are visible in-game. Call [`Client::SetShowingChat`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#acdf400ecb926392d1a110da73152b594) whenever the chat UI is shown or hidden so duplicate notifications can be suppressed.

### Legal Disclosure Message Type

 For developer convenience, when a user sends their first in-game message, Discord shows a generated disclosure message in the Discord client. The SDK injects this "fake" chat entry with a simple English explanation for the user. Detect it with [`MessageHandle::DisclosureType`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1MessageHandle.html#abefb9be7836951a6acf78a4bb1676638). You can render it differently, replace the wording, localize it, style it to look more official, or omit it from your UI.

### Message History

 The SDK stores the 25 newest messages per channel in memory, including DMs. To load older DM content, use [Getting Direct Message History](https://docs.discord.com/developers/discord-social-sdk/development-guides/sending-direct-messages#getting-direct-message-history) to request more history from Discord servers. An existing [`MessageHandle`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1MessageHandle.html#ae25595b43bc74b0c4c92c5165d16382f) remains usable after the SDK drops the message from its recent-message cache, but new [`MessageHandle`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1MessageHandle.html#ae25595b43bc74b0c4c92c5165d16382f) objects for that discarded message cannot be created with [`Client::GetMessageHandle`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a7825220b28952a2156bd0e46db40ea5c).

### Working with Unrenderable Content

 Discord messages can include media and structures a game may not render, such as images, videos, embeds, polls, and other content. The game does not need to display those directly; instead, show a notice that additional content exists and provide a path to view it in Discord. [`MessageHandle::AdditionalContent`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1MessageHandle.html#af4497491a95fda65402b6acf7a8b42e5) includes metadata for that non-text content. Use it to render a placeholder, and link to Discord through [`Client::CanOpenMessageInDiscord`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ae2aac143a691091691c5cc75aa07dace) and [`Client::OpenMessageInDiscord`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a66b8f85b14403a5d5ea125f39aa6e1b1). More background is available in the [messages in the Discord API](https://docs.discord.com/developers/resources/message) documentation.

### Getting Direct Message History

 The SDK has two APIs for loading direct message history so a game can show prior conversations when a user opens a DM or views a message list.

### Retrieving All Conversation Summaries

 Call [`Client::GetUserMessageSummaries`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a32dafc20ff1f99b019e40bdc81f46dde) to list every user who has a DM conversation with the current user, including each conversation's latest message ID:

```
client->GetUserMessageSummaries([](const discordpp::ClientResult &result,
                                  const std::vector<discordpp::UserMessageSummary> &summaries) {
    if (result.Successful()) {
        std::cout << "📋 Retrieved " << summaries.size() << " conversations\n";

        for (const auto &summary : summaries) {
            std::cout << "User ID: " << summary.UserId()
                     << ", Last Message ID: " << summary.LastMessageId() << "\n";
        }
    } else {
        std::cerr << "❌ Failed to retrieve conversation summaries\n";
    }
});

```

 This is particularly useful for:
- Building a conversation list UI
- Identifying users with existing conversations
- Locating the newest activity for each conversation

### Retrieving Messages from a Specific Conversation

 Call [`Client::GetUserMessagesWithLimit`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a054a758a76c5873b38a4d79651a5f26c) to load history for one DM conversation:

```
const uint64_t recipientId = 1234567890; // The other user's Discord ID
const int32_t messageLimit = 50; // Number of recent messages to retrieve (max 200)

client->GetUserMessagesWithLimit(
    recipientId, messageLimit,
    [](const discordpp::ClientResult &result,
       const std::vector<discordpp::MessageHandle> &messages) {
        if (result.Successful()) {
            std::cout << "💬 Retrieved " << messages.size()
                     << " messages from conversation\n";

            // Messages are returned in reverse chronological order (newest first)
            for (const auto &message : messages) {
                std::cout << "Message: " << message.Content()
                         << " from " << message.AuthorId() << std::endl;
            }
        } else {
            std::cerr << "❌ Failed to retrieve message history\n";
        }
    }
);

```

 **Important limitations:**
- Retrieval is capped at 200 messages and 72 hours of history
- DM history is available only when both players have played the game
- If either participant has not played the game, no channel may exist between them and the request can return a 404 `discordpp::ErrorType::HTTPError` error
 Important behavior for [`Client::GetUserMessagesWithLimit`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a054a758a76c5873b38a4d79651a5f26c):
- Results are ordered reverse chronologically, with newest messages first
- The function uses the local cache first and performs an HTTP request only when necessary
- Use 0 or a negative `limit` value to request all available messages, still subject to the 200 message cap
 This functionality is useful for:
- Showing prior messages when a user opens a DM
- Implementing message scrollback features
- Preserving conversation context across game sessions
- Implementing a complete in-game messaging UI

### In-Game Direct Message Settings

 The Discord client includes settings that let users control who can DM them in-game through the Social SDK. (Figure: Discord content and social - Connected game settings) The Social SDK cannot change those settings directly. It can call [`Client::OpenConnectedGamesSettingsInDiscord`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a24f268f5eebe9919a3f774354eb577e0), which opens Connected Games settings in the Discord client for game-related DM controls. If the client is disconnected or the user is a provisional account, the call has no effect. On console platforms, it is always a no-op.

---

### Development Rate Limits

 Until an app is approved for [increased rate limits](https://docs.discord.com/developers/discord-social-sdk/core-concepts/communication-features#applying-for-increased-rate-limits-for-production-releases), direct messaging is limited to **100 per 2 hours** per application. These development limits provide room for testing and small demos while protecting service stability. For SDK retry guidance after rate limiting, see [Handle Rate Limits](https://docs.discord.com/developers/discord-social-sdk/how-to/handle-rate-limits).

---

### Next Steps

 After implementing message send and receive behavior, review these related Discord Social SDK capabilities:

### Setting Rich Presence

Show friends the user's in-game status and current activity.

### Managing Lobbies

Create shared lobbies with invites, text chat, and voice communication.

### Linked Channels

Bridge a Discord channel with in-game text chat. Need help? Join the [Discord Developers Server](https://discord.gg/discord-developers) and ask in `#social-sdk-dev-help`. Bugs in the Social SDK can be reported at [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report).

---

### Change Log

| Date | Changes |
| --- | --- |
| August 29, 2025 | Add DM history |
| June 30, 2025 | Add communications scope warning |
| May 06, 2025 | link to DM Settings |
| March 17, 2025 | initial release |
