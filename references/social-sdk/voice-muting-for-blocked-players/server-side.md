# Server side
Context: Discord developer reference: Server side
[Source](voice-muting-for-blocked-players-index.md) / [Previous source section](client-side.md) / [Following source section](apply-the-mute-list.md)
<a id="source-server-side"></a>

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

