# Managing Relationships

> Discord Developers reference. Original prose; identifiers preserved verbatim.

### Overview

 The Discord Social SDK includes relationship-management features for players in your game. This guide covers how to:
- Send and accept friend requests
- Handle different types of relationships
- Block and unblock users
- Handle Discord-wide friendships and game-specific friendships

### Prerequisites

 Before starting, confirm you have:
- Finished the [Getting Started](https://docs.discord.com/developers/discord-social-sdk/getting-started) guide
- Finished the [Creating a Unified Friends List](https://docs.discord.com/developers/discord-social-sdk/development-guides/creating-a-unified-friends-list) guide
 This feature depends on the **Default Presence Scopes** (`openid` and `sdk.social_layer_presence`). Configure your OAuth2 flow with [`Client::GetDefaultPresenceScopes`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a7648bd1d2f7d9a86ebd0edb8bef12b5c). For the complete scope list, see the [OAuth2 Scopes guide](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes).

---

### Understanding Relationship Types

 In the SDK, the Relationship entity represents the connection between two users. It covers more than friendship: the same model is used for sent requests, received requests, and blocked users. Although the SDK can modify a user's relationships, do so only after explicit user consent. Do not send or accept friend requests automatically. Call relationship-management APIs only from user-driven actions, such as pressing a "Send Friend Request" button.

### Relationship Types

 Some users want friendships that follow them across Discord and every game. When they try a new game, those friends are already visible. Other users prefer to share access only inside the current game. To support both cases, the Discord Social SDK models two relationship categories:
- **Discord relationships**: These persist across games and in the Discord client. Each user can see the other's online state even when they are not in the same game. Discord Relationships are equivalent to becoming friends in the Discord client.
- **Game relationships**: These apply only to one game and do not transfer to other games. The users can see each other online only while they are playing a game where they are friends. Game friends can DM each other, and those DMs appear in Discord, unless they disable that behavior and keep conversations limited to the game. Users can find that control by opening User Settings, going to "Content & Social", and using the "Connected Games" tab near the top.
 Use [`RelationshipHandle`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1RelationshipHandle.html#a7da36b15ad0b7d38ba658a622e9ded77) to inspect what kind of friendship exists between the player and another user. It exposes two fields:
- [`RelationshipHandle::DiscordRelationshipType`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1RelationshipHandle.html#a5fecfb79a4a2b6f3dc5f73b09d0c3881) for the **Discord friendship**
- [`RelationshipHandle::GameRelationshipType`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1RelationshipHandle.html#aa60146eb72ede07e3e615565f61f97eb) for the **game friendship**
 Both friend types matter because two users may begin as game friends and later choose to "upgrade" to full Discord friends. In that situation, [`RelationshipHandle::DiscordRelationshipType`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1RelationshipHandle.html#a5fecfb79a4a2b6f3dc5f73b09d0c3881) becomes `RelationshipType::PendingIncoming` or `RelationshipType::PendingOutgoing`, depending on whether the current user received or sent the request, while [`RelationshipHandle::GameRelationshipType`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1RelationshipHandle.html#aa60146eb72ede07e3e615565f61f97eb) remains `RelationshipType::Friend`. The API can represent users who are both kinds of friends, but you do not need to force every Discord friend to also be a game friend, or every game friend to also be a Discord friend. When adding friends, let users choose the relationship type and make the difference clear. See the design guidelines for more.

### Discord Friend Relationships

- Persist across Discord and every game
- Limited to 1,000 friends
- Online status visible everywhere
- Full Discord chat functionality

### Game Friend Relationships

- Only exist within your game
- No current friend limit
- Online presence is visible only inside the game

---

### Relationship Actions

 After you have [created a unified friends list](https://docs.discord.com/developers/discord-social-sdk/development-guides/creating-a-unified-friends-list), your game can manage relationships between players. Common actions include:

### Sending Game Friend Requests

 Sends a game friend request to the target user, or accepts one when the target already sent a pending request. You may address the target by Discord unique username or user ID. After sending, both users get a game relationship. The current user's [`RelationshipHandle::GameRelationshipType`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1RelationshipHandle.html#aa60146eb72ede07e3e615565f61f97eb) becomes `RelationshipType::PendingOutgoing`, while the target user's value becomes `RelationshipType::PendingIncoming`. If the current user already has an incoming game friend request from that target, meaning [`RelationshipHandle::GameRelationshipType`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1RelationshipHandle.html#aa60146eb72ede07e3e615565f61f97eb) is `RelationshipType::PendingIncoming`, the action makes the users game friends.

```
client->SendGameFriendRequest("username", [](discordpp::ClientResult result) {
  if (result.Successful()) {
    std::cout << "🎮 Game friend request sent successfully!\n";
  }
});

client->SendGameFriendRequestById(123456789, [](discordpp::ClientResult result) {
  if (result.Successful()) {
    std::cout << "🎮 Game friend request sent successfully!\n";
  }
});

```

### Sending Discord Friend Requests

 Sends a Discord friend request to the target user, or accepts one when the target already sent a pending request. You may address the target by Discord unique username or user ID. After sending, both users get a Discord relationship. The current user's [`RelationshipHandle::DiscordRelationshipType`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1RelationshipHandle.html#a5fecfb79a4a2b6f3dc5f73b09d0c3881) becomes `RelationshipType::PendingOutgoing`, while the target user's value becomes `RelationshipType::PendingIncoming`. If the current user already has an incoming Discord friend request from that target, meaning [`RelationshipHandle::DiscordRelationshipType`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1RelationshipHandle.html#a5fecfb79a4a2b6f3dc5f73b09d0c3881) is `RelationshipType::PendingIncoming`, the action makes the users Discord friends.

```
client->SendDiscordFriendRequest("username", [](discordpp::ClientResult result) {
  if (result.Successful()) {
    std::cout << "🎮 Discord friend request sent successfully!\n";
  }
});

client->SendDiscordFriendRequestById(123456789, [](discordpp::ClientResult result) {
  if (result.Successful()) {
    std::cout << "🎮 Discord friend request sent successfully!\n";
  }
});

```

### Accept Incoming Friend Requests

 Let players accept incoming friend requests represented by `RelationshipType::PendingIncoming`.

```
// RelationshipHandle::DiscordRelationshipType == RelationshipType::PendingIncoming
client->AcceptDiscordFriendRequest(userId, [](discordpp::ClientResult result) {
  if (result.Successful()) {
    std::cout << "🎮 Discord friend request accepted!\n";
  }
});

// RelationshipHandle::GameRelationshipType == RelationshipType::PendingIncoming
client->AcceptGameFriendRequest(userId, [](discordpp::ClientResult result) {
  if (result.Successful()) {
    std::cout << "🎮 Game friend request accepted!\n";
  }
});

```

### Reject Incoming Friend Requests

 Let players reject incoming friend requests represented by `RelationshipType::PendingIncoming`.

```
// Reject Incoming Friend Requests
// RelationshipHandle::DiscordRelationshipType == RelationshipType::PendingIncoming
client->RejectDiscordFriendRequest(userId,[](discordpp::ClientResult result) {
  if (result.Successful()) {
    std::cout << "🎮 Discord friend request rejected!\n";
  }
});

// RelationshipHandle::GameRelationshipType == RelationshipType::PendingIncoming
client->RejectGameFriendRequest(userId, [](discordpp::ClientResult result) {
  if (result.Successful()) {
    std::cout << "🎮 Game friend request rejected!\n";
  }
});

```

### Cancel Outgoing Friend Requests

 Let players cancel outgoing friend requests represented by `RelationshipType::PendingOutgoing`.

```
// Cancel Outgoing Friend Requests
// RelationshipHandle::DiscordRelationshipType == RelationshipType::PendingIncoming
client->CancelDiscordFriendRequest(userId, [](discordpp::ClientResult result) {
  if (result.Successful()) {
    std::cout << "🎮 Discord friend request canceled!\n";
  }
});

// RelationshipHandle::GameRelationshipType == RelationshipType::PendingOutgoing
client->CancelGameFriendRequest(userId, [](discordpp::ClientResult result) {
  if (result.Successful()) {
    std::cout << "🎮 Game friend request canceled!\n";
  }
});

```

### Managing Existing Relationships

 Let players remove relationships with other users. Removal affects both users, ending their ability to see each other's online status or send messages.

```
// Removes any friendship between the current user and the target user.
// This function will remove BOTH any Discord friendship and any game friendship between the users.
client->RemoveDiscordAndGameFriend(userId, [](discordpp::ClientResult result) {
  if (result.Successful()) {
    std::cout << "🎮 Discord and Game friendships removed!\n";
  }
});

// Removes any game friendship between the current user and the target user.
client->RemoveGameFriend(userId, [](discordpp::ClientResult result) {
  if (result.Successful()) {
    std::cout << "🎮 Game friendship removed!\n";
  }
});

```

### Blocking Users

 Let players block another user so that user can no longer send friend invites, activity invites, or messages to them. Blocking also removes any existing relationship between the users. It is global across Discord and games: blocking in one game or in Discord blocks the user everywhere.

```
client->BlockUser(userId, [](discordpp::ClientResult result) {
  if (result.Successful()) {
    std::cout << "🎮 User blocked successfully!\n";
  }
});

```

### Unblocking Users

 Let players unblock a user who is currently blocked. Unblocking does not recreate any relationship that existed before the block.

```
client->UnblockUser(userId, [](discordpp::ClientResult result) {
  if (result.Successful()) {
    std::cout << "🎮 User unblocked successfully!\n";
  }
});

```

---

### Next Steps

 Continue with these Discord Social SDK guides:

### Creating a Unified Friends List

Merge Discord friends and game friends into one easier-to-manage list.

### Managing Game Invites

Let players invite friends into their game session or party.

### Managing Lobbies

Gather players in a shared lobby that supports invites, text chat, and voice comms. Need help? Join the [Discord Developers Server](https://discord.gg/discord-developers) and ask in `#social-sdk-dev-help` for community support. To report a Social SDK bug, use [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report).

---

### Change Log

| Date | Changes |
| --- | --- |
| March 17, 2025 | Initial release |
