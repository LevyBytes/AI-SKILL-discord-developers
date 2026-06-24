# Voice Muting For Blocked Players

> Discord Developers reference. Original prose; identifiers preserved verbatim.

This guide describes how to mute lobby voice participants when a block relationship exists, whether the signal comes from Discord relationships or another system such as a game service or platform backend.

### Overview

 This guide will help you:
- Know that blocking someone does not, by itself, mute their voice audio
- Build a per-player mute list from Discord block relationships on the client, or from your game server on the server side
- Use one shared mute-application path after either mute-list source has populated the data
- Meet the bi-directional mute requirement, so if A blocks B, audio is muted both ways

### Prerequisites

 Before starting, confirm these pieces are in place:
- You have finished the [Getting Started Guide](https://docs.discord.com/developers/discord-social-sdk/getting-started)
- You have a functioning lobby from [Managing Lobbies](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-lobbies)
- You understand the flow in [Managing Voice Chat](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-voice-chat)
- You understand the relationship APIs covered in [Managing Relationships](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-relationships)

---

### Why Voice Muting Must Be Handled Explicitly

### Discord Blocking Does Not Silence Voice

 Calling [`Client::BlockUser`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#add4a917c8382e411d5a55737c9edc8ad) stops friend requests and messages from that user, but lobby voice remains audible. To silence a blocked participant, the game must explicitly invoke [`Call::SetLocalMute`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Call.html#aaf8e7728b15da5d1be8d8b4258225171).

### Block Signals May Come From Outside Discord

 A game can learn about blocks from systems other than Discord, such as game-level or platform-level block lists. When that happens, the client has no Discord relationship record to query. Send the needed block data to clients by another route, for example through lobby member metadata.

### Bi-Directional Muting

 [`Call::SetLocalMute`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Call.html#aaf8e7728b15da5d1be8d8b4258225171) affects only the local listener. If A calls it, A no longer hears B, but B can still hear A. Full muting requires both clients to call `SetLocalMute` for the other participant. Each player may know about different block signals, so the guide uses lobby member metadata as the shared visibility layer: every player writes their own `mute_list`, all lobby members can inspect it, and a client can mute another participant when that participant has listed them, even when the original block data came from elsewhere.

---

### Step 1: Populate the Mute List

 Both methods store a `mute_list` entry in each lobby member's metadata. Its value is a comma-separated set of Discord user IDs that the player should not hear in voice. The distinction between the methods is how those IDs are collected.

### Client-Side

 Choose this path when the client can directly access the block source, such as Discord relationships, an in-game friend/block list, or another local data source. On lobby join, derive the mute list from the available client-side information and write those IDs into the player's own member metadata with [`Client::CreateOrJoinLobbyWithMetadata`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a5c84fa76c73cf3c0bfd68794ca5595c1). Other lobby members can then see that list and apply reciprocal muting. The sample uses Discord block relationships; replace the `myMuteList` construction when your game uses a different signal.

```
const auto lobbySecret = "my-lobby-secret";

// Example: build the mute list from Discord block relationships.
// Replace this with your own logic if using a different data source.
std::string myMuteList;
for (const auto& rel : client->GetRelationships()) {
    if (rel.DiscordRelationshipType() == discordpp::RelationshipType::Blocked) {
        if (!myMuteList.empty()) myMuteList += ",";
        myMuteList += std::to_string(rel.Id());
    }
}

// Join the lobby with your mute list in your member metadata
client->CreateOrJoinLobbyWithMetadata(
  lobbySecret,
  {},  // no lobby-level metadata needed
  {{"mute_list", myMuteList}},
  [](const discordpp::ClientResult &result, uint64_t lobbyId) {
      if (!result.Successful()) {
          std::cerr << "Failed to join lobby\n";
      }
  }
);

```

 Member metadata is capped at 1,000 characters total. A Discord snowflake ID plus comma separator is about 19 characters, so roughly 50 blocked users fit comfortably. If that is not enough, use the Server Side approach.

### Server-Side

 Choose this path when the block source is external, when the server must enforce bi-directional muting without trusting clients to publish metadata, or when very large block lists need to be filtered to the players currently in the lobby. The server can reduce each player's block list to only members in the session and write that result into metadata as the lobby is created. Because the server knows the joining set, it only checks block relationships among those participants, not every block the player has. Step 2's client code covers the reverse direction by checking whether another lobby member listed the local player.

```
import requests

API_ENDPOINT = 'https://discord.com/api/v10'
BOT_TOKEN = 'YOUR_BOT_TOKEN'

def create_lobby_with_mute_metadata(session_members, block_relationships):
    """
    session_members: list of Discord user ID strings for this lobby
    block_relationships: dict mapping user ID to list of user IDs they have blocked

    Each player's mute_list contains only the session members they have blocked.
    The client handles the reverse direction by checking whether others have listed them.
    """
    mute_map = {user_id: set() for user_id in session_members}

    # Only include blocks where both users are in this session
    for blocker_id, blocked_ids in block_relationships.items():
        if blocker_id in mute_map:
            for blocked_id in blocked_ids:
                if blocked_id in mute_map:
                    mute_map[blocker_id].add(blocked_id)

    members = []
    for user_id in session_members:
        mute_list = mute_map.get(user_id, set())
        members.append({
            "id": user_id,
            "metadata": {"mute_list": ",".join(mute_list)} if mute_list else None,
        })

    response = requests.post(
        f'{API_ENDPOINT}/lobbies',
        headers={
            'Authorization': f'Bot {BOT_TOKEN}',
            'Content-Type': 'application/json',
        },
        json={"members": members},
    )
    response.raise_for_status()
    return response.json()

# Example: Player A has blocked Player B.
# Only A's mute_list contains B. B's mute_list is empty (B has not blocked anyone).
# The client-side code in Step 2 handles the reverse: B will detect A has listed them and mute A.
lobby = create_lobby_with_mute_metadata(
    session_members=[
        "111111111111111111",  # Player A
        "222222222222222222",  # Player B
        "333333333333333333",  # Player C (no blocks)
    ],
    block_relationships={
        "111111111111111111": ["222222222222222222"],  # A has blocked B
    },
)
print(f"Lobby created: {lobby['id']}")

```

 When players enter after lobby creation, update metadata with the blocklists that apply to them. Use [`POST /lobbies/{lobby.id}/members/bulk`](https://docs.discord.com/developers/resources/lobby#bulk-update-lobby-members) for up to 25 member additions or updates at once, or [`PUT /lobbies/{lobby.id}/members/{user.id}`](https://docs.discord.com/developers/resources/lobby#add-a-member-to-a-lobby) for one member.

---

### Step 2: Apply the Mute List

 After either method has written `mute_list` metadata, the same client routine reads it and applies the needed mutes.

### On Lobby Join

 Once the lobby is joined and the voice call has started, call `ApplyMuteChecks` for each other member in the lobby.

```
const auto currentUser = client->GetCurrentUserV2();
if (!currentUser) return;
const auto myUserId = currentUser->Id();
const auto lobby = client->GetLobbyHandle(lobbyId);
auto call = client->StartCall(lobbyId);

if (lobby && call) {
    for (auto memberId : lobby->LobbyMemberIds()) {
        if (memberId != myUserId) {
            ApplyMuteChecks(call, *lobby, myUserId, memberId);
        }
    }
}

// Mutes lobbyUserId locally if either myUserId or lobbyUserId has listed the other in their mute_list.
void ApplyMuteChecks(discordpp::Call& call, const discordpp::LobbyHandle& lobby,
                     const uint64_t myUserId, const uint64_t lobbyUserId) {

  auto isListed = [&](const uint64_t ownerId, const uint64_t searchId) -> bool {
    const auto member = lobby.GetLobbyMemberHandle(ownerId);
    if (!member) return false;
    auto metadata = member->Metadata();
    const auto it = metadata.find("mute_list");
    if (it == metadata.end()) return false;
    std::stringstream ss(it->second);
    std::string idStr;
    while (std::getline(ss, idStr, ',')) {
      if (std::stoull(idStr) == searchId) return true;
    }
    return false;
  };

  if (isListed(myUserId, lobbyUserId) || isListed(lobbyUserId, myUserId)) {
    call.SetLocalMute(lobbyUserId, true);
  }
}

```

### Handling Participants Who Join Later

 Register [`Call::SetParticipantChangedCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Call.html#acb20d338a04abec2369217f41c22c0e5) so mute checks run for participants who enter during the session. The callback uses `added = true` for joins and `added = false` for leaves.

```
call.SetParticipantChangedCallback(
    [client, lobbyId, myUserId, call](uint64_t userId, const bool added) mutable {
      if (!added) return;
      auto lobby = client->GetLobbyHandle(lobbyId);
      if (!lobby) return;
      ApplyMuteChecks(call, *lobby, myUserId, userId);
    }
);

```

---

### Best Practices

- **Consider filtering at matchmaking time.** The best user experience is to keep blocked players out of the same lobby. Voice muting covers audio, but the players may still appear to each other in game UI.
- **Update mutes when relationships change.** If a player blocks someone while a session is active, immediately call [`Call::SetLocalMute`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Call.html#aaf8e7728b15da5d1be8d8b4258225171) and refresh the `mute_list` member metadata by invoking [`Client::CreateOrJoinLobbyWithMetadata`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a5c84fa76c73cf3c0bfd68794ca5595c1) again with the updated list. In the server-side model, update that member through [`PUT /lobbies/{lobby.id}/members/{user.id}`](https://docs.discord.com/developers/resources/lobby#add-a-member-to-a-lobby).
- **Keep metadata compact.** Lobby member metadata is limited to 1,000 characters, so comma-separated ID strings are preferable to JSON objects.

---

### Next Steps

### Managing Voice Chat

Add lobby voice communication inside your game.

### Managing Lobbies

Create game lobbies and manage them for matchmaking flows.

### Managing Relationships

Work with Discord relationship data for friends and blocked users. Need help? Join the [Discord Developers Server](https://discord.gg/discord-developers) and ask in `#social-sdk-dev-help`. Bugs found while using the Social SDK should be reported at [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report).

---

### Change Log

| Date | Changes |
| --- | --- |
| April 22, 2026 | Initial release |

[How To Handle Special Characters in Display NamesPrevious](https://docs.discord.com/developers/discord-social-sdk/how-to/handle-special-characters-display-names)⌘I
