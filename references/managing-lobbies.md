# Managing Lobbies

> Discord Developers reference. Original prose; identifiers preserved verbatim.

This feature is available, but rate limits apply. To request higher limits for a production game, follow [Communication Features: Applying for Increased Rate Limits for Production Releases](https://docs.discord.com/developers/discord-social-sdk/core-concepts/communication-features#applying-for-increased-rate-limits-for-production-releases).

### Overview

 Lobbies group users so they can use text and voice communication together. This guide explains how to:
- Create and manage lobbies
- Handle lobby membership
- Send messages to lobbies

### Prerequisites

 Before continuing, confirm that you have:
- Installed and configured the Discord Social SDK using the [Getting Started guide](https://docs.discord.com/developers/discord-social-sdk/getting-started)
 This communication feature requires [`Client::GetDefaultCommunicationScopes`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a71499da752fbdc2d4326ae0fd36c0dd1) in the OAuth Scope configuration. See the [OAuth Scopes Core Concepts Guide](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes) for the related details.

---

### Lobby Features

 A lobby is a user group with shared text and voice communication. A user can belong to several lobbies at the same time. Both the lobby and each user in it can have metadata attached as an arbitrary JSON blob.

### Lobby Members

 A lobby can contain up to 1,000 members, and a user can be in up to 100 lobbies for one game. Contact Discord if the game has a use case requiring more than 1,000 members in one lobby.

### Text Chat

 Each lobby includes a text chat channel. Any lobby member can send messages, and those messages go to all other members.

### Voice Chat

 Lobbies can host voice calls. Even though a lobby can have 1,000 members, voice calls should not be started at that size. Discord recommends roughly 25 members or fewer for voice calls; contact Developer Support if the game needs calls larger than 25 members.

---

### Managing Lobbies

 Lobbies can be managed in two ways:
1. From the server through the Discord API.
2. From the client with [`Client::CreateOrJoinLobby`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a8b4e195555ecaa89ccdfc0acd28d3512).

### Server-Side Lobby Management

 Use the Discord HTTP API to create, update, delete, and manage membership for lobbies. The [Lobby](https://docs.discord.com/developers/resources/lobby) API resource lists the endpoints, and [Use with Discord APIs](https://docs.discord.com/developers/discord-social-sdk/how-to/use-with-discord-apis) explains request authentication. SDK clients cannot use [`Client::CreateOrJoinLobby`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a8b4e195555ecaa89ccdfc0acd28d3512), [`Client::CreateOrJoinLobbyWithMetadata`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a5c84fa76c73cf3c0bfd68794ca5595c1), or [`Client::LeaveLobby`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a8c78f797240b35d721383461a2e62926) for lobbies created through the API.

### Client-Side Lobby Management

 The SDK client can create and join lobbies by using a secret associated with the lobby. Share that secret with the users who should be able to join. If no lobby exists for the secret, one is created when the client joins.
- The main SDK functions are [`Client::CreateOrJoinLobby`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a8b4e195555ecaa89ccdfc0acd28d3512) and [`Client::LeaveLobby`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a8c78f797240b35d721383461a2e62926).
- Lobby secrets are unique per game, meaning per application. For a new lobby after a match ends, use a different secret. Calling [`Client::LeaveLobby`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a8c78f797240b35d721383461a2e62926) and then [`Client::CreateOrJoinLobby`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a8b4e195555ecaa89ccdfc0acd28d3512) with the same secret will add the user back to that same lobby.
- If the user is already in the lobby, calling [`Client::CreateOrJoinLobby`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a8b4e195555ecaa89ccdfc0acd28d3512) updates that user's metadata when metadata is supplied.
- Discord Rich Presence can sync the same secret. With that flow, clients may ask to join another user's activity. After approval, the SDK receives the secret, allowing the game to join the related lobby if desired. See the [Game Invites for Lobbies](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-lobbies#creating-lobby-invites) example below.
- To track metadata for a lobby or its members, call [`Client::CreateOrJoinLobbyWithMetadata`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a5c84fa76c73cf3c0bfd68794ca5595c1). It accepts a JSON object that is stored on the lobby and lobby members. Retrieve that metadata through the discordpp::Client::GetLobbyHandle function.

```
// Create or join a lobby from the client
client->CreateOrJoinLobby("your-unique-lobby-secret",[client](discordpp::ClientResult result, uint64_t lobbyId) {
  if(result.Successful()) {
    std::cout << "🎮 Lobby created or joined successfully! Lobby Id: " << lobbyId << std::endl;
  } else {
    std::cerr << "❌ Lobby creation/join failed\n";
  }
});

```

#### Leaving a Lobby

 Remove a user from a lobby with [`Client::LeaveLobby`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a8c78f797240b35d721383461a2e62926). Only lobbies created by [`Client::CreateOrJoinLobby`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a8b4e195555ecaa89ccdfc0acd28d3512) can be left using [`Client::LeaveLobby`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a8c78f797240b35d721383461a2e62926).

```
uint64_t lobbyId = 01234567890;

// Leaving a lobby from the client
client->LeaveLobby(lobbyId, [&](discordpp::ClientResult result) {
  if(result.Successful()) {
    std::cout << "🎮 Left lobby successfully! Lobby Id: " << lobbyId << std::endl;
  } else {
    std::cerr << "❌ Leaving lobby failed\n";
  }
}

```

### Lobby Lifecycle

 Lobbies are designed to be temporary and should be removed when the game, match, area, or world is no longer needed. Configure a "max idle time" so Discord deletes a lobby after no users have been connected for longer than that period. If any user remains connected, the lobby is not deleted and the timer resets. The default is 5 minutes, which should fit most cases. Longer-lived cases such as "world chat" may need more time. The expected maximum "idle time" is 7 days, meaning deletion occurs only after a full week with no connections. This gives lobbies some persistence when needed, though rare cases may still require rebuilding a lobby after everyone stays offline for a long time. SDK-created lobbies using [`Client::CreateOrJoinLobby`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a8b4e195555ecaa89ccdfc0acd28d3512) have another limit: the "secret" used for the call expires after 30 days. The lobby can still exist, but new users cannot join it with that secret after expiration.

---

### Sending Messages to a Lobby

 After a lobby exists, its members can send messages with [`Client::SendLobbyMessage`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a779e0483f51dc99f0db3dd761d22ab6f). A client may message any lobby it belongs to, regardless of whether the lobby was joined through client-side or server-side creation.

```
uint64_t lobbyId = 01234567890;

client->SendLobbyMessage(lobbyId, "Hello", [](discordpp::ClientResult result, uint64_t messageId) {
  if(result.Successful()) {
    std::cout << "📨 Message sent successfully!\n";
  } else {
    std::cerr << "❌ Message sending failed\n";
  }
});

```

---

### Receiving Lobby Messages

 Receive lobby messages by registering [`Client::SetMessageCreatedCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a28325a8e8c688a84ac851da4bc86e148). The callback fires for each new lobby message or DM. Use `messageId` to obtain the [`MessageHandle`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1MessageHandle.html#ae25595b43bc74b0c4c92c5165d16382f), then fetch the [`ChannelHandle`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1ChannelHandle.html#ac32096b2ef15c5c220e9b7b92253cc46) to identify where the message was sent.

```
client->SetMessageCreatedCallback([&client](uint64_t messageId) {
  discordpp::MessageHandle message = client->GetMessageHandle(messageId);
  std::cout << "📨 New message received: " << message->Content() << "\n";
});

```

---

### Getting Lobby Chat History

 Use [`Client::GetLobbyMessagesWithLimit`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a0586192e85caf548b8b321f1cb21301f) to fetch earlier messages for a lobby. This supports showing prior chat when a user joins or needs conversation history. The function takes a lobby ID and a limit specifying how many recent messages to retrieve. **Important limitations:**
- Retrieval is limited to 200 messages and 72 hours of history
- A user can retrieve only messages from lobbies they currently belong to

```
const uint64_t lobbyId = 01234567890;
const uint32_t messageLimit = 50; // Number of recent messages to retrieve (max 200)

client->GetLobbyMessagesWithLimit(
  lobbyId, messageLimit,
  [](const discordpp::ClientResult &result, const std::vector<discordpp::MessageHandle> &messages) {
    if (result.Successful()) {
      std::cout << "? Retrieved " << messages.size()
                << " messages from lobby chat history!\n";

      // Process the messages (they are returned in chronological order)
      for (const auto &message : messages) {
        std::cout << "Message: " << message.Content() << std::endl;
      }
    } else {
      std::cerr << "? Failed to retrieve lobby chat history\n";
    }
  });

```

 Results are returned as [`MessageHandle`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1MessageHandle.html#ae25595b43bc74b0c4c92c5165d16382f) objects in chronological order, oldest to newest. Each [`MessageHandle`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1MessageHandle.html#ae25595b43bc74b0c4c92c5165d16382f) includes content, author information, and timestamp. This is useful for:
- Showing recent chat when a user enters a lobby
- Implementing chat history scrollback features
- Preserving conversation context across game sessions

---

### Linking a Channel to Lobby

 Linked Channels can connect a lobby to a Discord text channel, allowing Discord users to chat with lobby members even when they are outside the game. See [Linked Channels](https://docs.discord.com/developers/discord-social-sdk/development-guides/linked-channels) for setup details.

---

### Managing Voice Chat

 See [Managing Voice Chat](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-voice-chat) for guidance on starting a voice call in a lobby.

---

### Creating Lobby Invites

 Games can combine lobbies with game invites so users can invite friends into an existing lobby. Try the [Using Game Invites with Lobbies](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-game-invites#using-game-invites-with-lobbies) example in the game.

---

### Development Rate Limits

 Apps that have not received [increased rate limits for production releases](https://docs.discord.com/developers/discord-social-sdk/core-concepts/communication-features#applying-for-increased-rate-limits-for-production-releases) must follow these application-wide limits:
| Operation | Limit |
| --- | --- |
| `Client::CreateOrJoinLobby` | 100 per 2 hours |
| `Client::LinkChannelToLobby` | 20 per 2 hours |
| `Client::UnlinkChannelFromLobby` | 20 per 2 hours |
| `Client::SendLobbyMessage` | 100 per 2 hours |

 These limits apply per application rather than per user. They are intended to allow development, testing, and small demos while maintaining service stability. For handling and retrying SDK requests after rate limits, see [Handle Rate Limits](https://docs.discord.com/developers/discord-social-sdk/how-to/handle-rate-limits).

---

### Next Steps

 After the game can create and manage lobbies, add related features:

### Managing Voice Chat

Add voice communication for lobby members.

### Linked Channels

Link lobbies with Discord text channels.

### Voice Muting Based on Player Blocks

Mute lobby voice participants locally based on player block relationships. Need help? Join the [Discord Developers Server](https://discord.gg/discord-developers) and ask in `#social-sdk-dev-help`. Bugs in the Social SDK can be reported at [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report).

---

### Change Log

| Date | Changes |
| --- | --- |
| June 30, 2025 | Add communications scope warning |
| March 17, 2025 | initial release |
