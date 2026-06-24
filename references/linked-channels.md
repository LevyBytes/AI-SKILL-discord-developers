# Linked Channels

> Discord Developers reference. Original prose; identifiers preserved verbatim.

This feature is offered under [limited access](https://docs.discord.com/developers/discord-social-sdk/core-concepts/communication-features#limited-access). To request full access for closed beta features or talk with Discord about your game, submit [this form](https://discord.com/developers/social-sdk-closed-beta-access-request-form).

### Overview

 Linked Channels connect game lobbies to Discord text channels so conversation can flow between the game and a Discord server. After linking:
- Messages sent in game are posted to the Discord channel
- Messages from Discord are shown inside the game
- Discord displays messages from the game with the game name and icon

### Prerequisites

 Before adding Linked Channels, confirm these pieces are in place:
- [Set up the Discord Social SDK](https://docs.discord.com/developers/discord-social-sdk/getting-started)
- Familiarity with [Creating and Managing Lobbies](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-lobbies)
 This communication feature requires [`Client::GetDefaultCommunicationScopes`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a71499da752fbdc2d4326ae0fd36c0dd1) in your OAuth Scope configuration. The [OAuth Scopes Core Concepts Guide](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes) has the broader scope details.

---

### Linked Channel Requirements

### Channel Requirements

 A Discord channel can be linked to a game lobby only when it satisfies these requirements:

#### Channel Type and Settings

| Requirement | Description |
| --- | --- |
| Channel Type | Must be a server text channel in Discord |
| Age Restriction | Must not be marked age-restricted |
| Link Status | Must not already be connected to a different lobby |

#### Required Permissions

 The channel-linking user needs these Discord permissions:
| Permission | Description |
| --- | --- |
| Manage Channels | Ability to edit channel settings |
| View Channel | Has permission to view and read the channel |
| Send Messages | Has permission to send messages in the channel |

### Private Channels

 Discord lets a user link any server channel they can access, including channels that are private from other server members. For example, an admin could link a private admins chat channel. The game cannot determine which Discord users should be allowed into that channel, and every lobby member can read and reply to linked-channel messages. If private channels are allowed, clearly warn users that the linked channel will be visible to the full lobby. Discord provides the `isViewableAndWriteableByAllMembers` boolean to help identify public versus restricted channels; use it to block private channel links or to decide when to show a warning.

### User Requirements

#### Linking A Lobby

 A lobby member can link a channel only when their `CanLinkLobby` flag is set. This flag starts disabled and must be granted through the [Lobby API](https://docs.discord.com/developers/resources/lobby) for users who should have elevated lobby permissions. Discord recommends enabling it only for the lobby equivalent of an administrator or owner, so channel configuration is limited to those users.

#### Unlinking a Lobby

 To remove a channel link from a lobby, the lobby member only needs `CanLinkLobby`. No Discord-side channel permissions are required for unlinking.

### Lobby Requirements

 Use channel linking only for **persistent** lobbies. Temporary match-chat lobbies are poor candidates, and client-created lobbies that use secrets cannot use channel linking.

---

### Creating a Linked Channel

 Linking is completed inside the game; the player is not sent to Discord to configure it. At a high level, fetch the servers, also called guilds, and channels available to the user, present UI for selecting a channel, then save that choice back to Discord.

### Fetch Available Servers (Guilds) and Channels

 The SDK includes helper calls for populating guild and channel pickers. [`Client::GetUserGuilds`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#aac1ec02df6074ed9213ce230e6a42fe1) retrieves the user's Discord servers through the API and returns `GuildMinimal` entries with guild ID and name on success. [`Client::GetGuildChannels`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#adba1e5a83c219a9c4f6dab1657778017) takes a guildId and returns the `GuildChannel` objects the user can access in that guild:

```
struct GuildChannel {
    struct LinkedLobby {
        uint64_t applicationId;
        uint64_t lobbyId;
    };
    uint64_t id;
    std::string name;
    bool isLinkable;
    bool isViewableAndWriteableByAllMembers;
    std::optional<LinkedLobby> linkedLobby;
};

```

- `isLinkable` says whether that guild channel can be linked by the user. It accounts for channel validity, including text-channel status, age restriction, and existing lobby links, as well as the user's permissions. See the **Edit Lobby/Channel Link** endpoint for the complete requirements.
- `isViewableAndWriteableByAllMembers` reports whether Discord read/write access is restricted for any member or role, which can indicate a private channel. Discord enforces those permissions only inside the Discord client. In game, any lobby member can read and write through the link, even if they could not see the channel in Discord. **Warn the linking player that channel contents may be exposed to in-game players who lack Discord access.**

### Saving a channel link selection

 After the user picks a channel, call [`Client::LinkChannelToLobby`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a3114d58d50d4d2cb5752d95e121315d4) to create or update the lobby link. Use [`Client::UnlinkChannelFromLobby`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a28f78a6fe46eb11eb54ee9b53fa94ffe) to clear it. The lobby, channel, and user must satisfy [Linked Channel Requirements](https://docs.discord.com/developers/discord-social-sdk/development-guides/linked-channels#channel-requirements).

### Try It Out

 To prototype Linked Channels before building a full player UI, use the Discord API with this flow:
1. Create a lobby with the **[Create Lobby](https://docs.discord.com/developers/resources/lobby#create-lobby)** endpoint.
2. Set the `CanLinkLobby` flag (`1 << 0`) for the lobby member by calling `/lobbies/<lobby_id>/members/<user_id>` or by including the member data in the [Create Lobby](https://docs.discord.com/developers/resources/lobby#create-lobby) request body.
3. Choose a Discord server text channel where the `lobby_member` has the required permissions, including read/write and manage channels, then record the channel id.
4. Call `/lobbies/<lobby_id>/channel-linking` with that channel id.
 The lobby and channel should now be connected; messages sent on either side should appear on the other side.

---

### Example Implementation

### Step 1: Set Up Message Handling

```
// filepath: your_game/channel_linking.cpp
void HandleDiscordMessage(const std::string& message) {
    std::cout << "📱 Game received: " << message << "\n";
    // Check if message is from the lobby
    // Display in your game UI
}

void SetupMessageHandlers(std::shared_ptr<discordpp::Client> client) {
    client->SetMessageCreatedCallback([](uint64_t messageId) {
        discordpp::MessageHandle message = client->GetMessageHandle(messageId);
        HandleDiscordMessage(message);
    });
}

```

### Step 2: Fetch Available Servers

```
void FetchDiscordServers(std::shared_ptr<discordpp::Client> client) {
    client->GetUserGuilds([](auto result, const auto& guilds) {
        if (!result.Successful()) {
            std::cout << "❌ Failed to fetch guilds\n";
            return;
        }

        for (const auto& guild : guilds) {
            std::cout << "📁 Guild: " << guild.name << "\n";
            // Store guild IDs for later use
            SaveGuildId(guild.id);
        }
    });
}

void FetchChannelsForGuild(
    std::shared_ptr<discordpp::Client> client,
    uint64_t guildId
) {
    client->GetGuildChannels(guildId, [](auto result, const auto& channels) {
        if (!result.Successful()) return;

        for (const auto& channel : channels) {
            if (channel.isLinkable) {
                std::cout << "  📝 Channel: " << channel.name;
                if (!channel.isViewableAndWriteableByAllMembers) {
                    std::cout << " (Private)";
                }
                std::cout << "\n";
            }
        }
    });
}

```

### Step 3: Link Channel to Lobby

```
void ShowPrivateChannelWarning() {
    std::cout << "⚠️ Warning: This is a private channel.\n";
    std::cout << "All lobby members will be able to see messages.\n";
    std::cout << "Do you want to continue? (y/n)\n";

    char response;
    std::cin >> response;
    return (response == 'y' || response == 'Y');
}

void LinkChannelToLobby(
    std::shared_ptr<discordpp::Client> client,
    uint64_t lobbyId,
    uint64_t channelId,
    bool isPrivate
) {
    if (isPrivate && !ShowPrivateChannelWarning()) {
        std::cout << "❌ Channel linking cancelled\n";
        return;
    }

    client->LinkChannelToLobby(
        lobbyId,
        channelId,
        [](auto result) {
            if (result.Successful()) {
                std::cout << "✅ Channel linked successfully!\n";
            } else {
                std::cout << "❌ Failed to link channel\n";
            }
        }
    );
}

```

### Step 4: Send and Receive Messages

```
void SendLobbyMessage(
    std::shared_ptr<discordpp::Client> client,
    uint64_t lobbyId,
    const std::string& message
) {
    client->SendLobbyMessage(lobbyId, message, [](auto result) {
        if (!result.Successful()) {
            std::cout << "❌ Failed to send message\n";
        }
    });
}

void UnlinkChannel(
    std::shared_ptr<discordpp::Client> client,
    uint64_t lobbyId
) {
    client->UnlinkChannelFromLobby(lobbyId, [](auto result) {
        if (result.Successful()) {
            std::cout << "✅ Channel unlinked\n";
        }
    });
}

```

### Step 5: Putting It All Together

 These functions can be combined like this:

```
// filepath: your_game/main.cpp
int main() {
    auto client = std::make_shared<discordpp::Client>();

    // Set up handlers
    SetupMessageHandlers(client);

    // When user wants to link a channel:
    FetchDiscordServers(client);

    // After user selects a guild:
    FetchChannelsForGuild(client, selectedGuildId);

    // After user selects a channel:
    LinkChannelToLobby(
        client,
        currentLobbyId,
        selectedChannelId,
        isPrivateChannel
    );

    // When sending a message:
    SendLobbyMessage(client, currentLobbyId, "Hello from the game!");

    // When done with the channel:
    UnlinkChannel(client, currentLobbyId);

    return 0;
}

```

### Joining Discord Servers via Linked Lobbies

 After a lobby is connected to a Discord channel, the game can let players join the related Discord server directly. [`Client::JoinLinkedLobbyGuild`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a54ec764e72e168de419ac14e24e8fc60) creates a single-use invite for the current user and, on supported platforms, opens Discord so the user can accept it. That removes the need to distribute invite links manually. A possible in-game flow is:
1. A Discord server admin links one server channel to the game lobby
2. Lobby players are shown a “Join Discord Server” action in game
3. Clicking it makes the SDK create a unique invite and open Discord
4. The player accepts the invite and joins the Discord server
 Only linked Discord accounts can use this server-join flow. For provisional accounts, prompt the player to link their Discord account first.

```
const uint64_t lobbyId = 1234567890;

// Invite the user to join the Discord guild associated with the linked lobby
client->JoinLinkedLobbyGuild(
    lobbyId,
    // This is triggered when the user is using a provisional account, since
    // the user needs a real Discord account to join the Discord server, so you don't need
    // to implement a provisional user check on implementation.
    [] {
        // Show a message in your UI explaining that they need to link their Discord account
        // and/or step them through the process
        std::cout << "📝 User needs to link their Discord account\n";
    },
    // Called after the invite generation attempt completes, providing either a
    // successful result with the invite URL or an error if the operation failed.
    [](const discordpp::ClientResult &result, const std::string& inviteUrl) {
        if(result.Successful()) {
            std::cout << "✅ Discord invite generated successfully!\n";

            // On console platforms, you'll need to display the invite URL
            // for users to manually navigate to
            #ifdef CONSOLE_PLATFORM
            std::cout << "Join the Discord server at: " << inviteUrl << "\n";
            // Display this URL in your game's UI for the player to use
            #endif
        } else {
            std::cerr << "❌ Failed to generate Discord invite\n";
        }
    }
);

```

### Platform Considerations

- **Desktop**: The SDK opens either the Discord client or a web browser with the invite
- **Console**: Console platforms cannot open Discord directly, so show the invite URL in the game UI for use on another device
 Server admins cannot limit this join path to a subset of linked-lobby players. Any player in the linked lobby can create a server invite, independent of lobby permissions.
Tell players to link channels only in servers they are comfortable letting the lobby join, or provide game-side controls that disable this feature for selected lobbies.

---

### Next Steps

### Creating a Unified Friends List

Merge Discord friends and game friends into one manageable list.

### Setting Rich Presence

Show Discord friends what a player is doing in the game.

### Managing Game Invites

Let players invite friends into a game session or party. Need help? Join the [Discord Developers Server](https://discord.gg/discord-developers) and ask in `#social-sdk-dev-help` for community support. Report Social SDK bugs at [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report).

---

### Change Log

| Date | Changes |
| --- | --- |
| June 30, 2025 | Add communications scope warning |
| March 17, 2025 | initial release |
