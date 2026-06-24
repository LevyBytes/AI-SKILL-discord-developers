# Creating A Unified Friends List

> Discord Developers reference. Original prose; identifiers preserved verbatim.

### Overview

 A unified friends list presents Discord relationships and game-specific relationships together. This guide covers how to:
- Fetch all relationship data
- Filter and organize relationships
- Display online status
- Handle different relationship types

### Prerequisites

 Before starting, confirm that you have:
- Configured the Discord Social SDK by following the [Getting Started Guide](https://docs.discord.com/developers/discord-social-sdk/getting-started)
- Authenticated users with [Development Guide: Account Linking](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord)
- Learned the relationship model from [Development Guide: Managing Relationships in Your Game](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-relationships)
 This feature needs the **Default Presence Scopes** (`openid` and `sdk.social_layer_presence`). When configuring OAuth2, use [`Client::GetDefaultPresenceScopes`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a7648bd1d2f7d9a86ebd0edb8bef12b5c). The [OAuth2 Scopes guide](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes) describes the full scope set.

### Implementing a Unified Friends List

 A Discord friend list is built from Relationships and Users. Use the Relationships API to identify the people connected to the user, then use the Users API to fetch rendering details such as online state.

### Relationships

 [Relationships](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-relationships) represent Discord friends, friend requests, and related states. The Client loads every relationship for the current user when it connects. Each relationship includes a target user id and a type, such as `Friend`, `PendingOutgoing`, or `Blocked`. To let users manage relationships from your game, provide controls to accept or reject friend requests, block users, and handle pending requests. See [Development Guide: Managing Relationships in Your Game](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-relationships) for implementation details.

### Users

 Users are the Discord accounts referenced by relationships. The SDK can fetch a user by ID, and the user object includes fields such as username, display name, avatar, and other profile data.

### User Status & Rich Presence

 Presence records whether a user is online and which activities they are doing, such as playing a game. The SDK exposes two status categories:
- Online Status: Online, Offline, Idle, etc.
- Rich Presence: Activities connected to the current game, or application in Discord terminology.
 The SDK shows only activities associated with the current game, so Rich Presence will not include other game activity even when it appears in the Discord client. See [Design Guidelines: Status & Rich Presence](https://docs.discord.com/developers/discord-social-sdk/design-guidelines/status-rich-presence) for presence display guidance. You can build a unified friends list in two ways:
1. Use the SDK Unified Friends List helper functions, which group and sort relationships and users automatically.
2. Retrieve relationships and users from the SDK directly, then sort them yourself.

### Approach 1: Using SDK Unified Friends List Helper Functions

 This recommended approach reduces the code you write and maintain while aligning the list with Discord's [recommended design guidelines](https://docs.discord.com/developers/discord-social-sdk/design-guidelines/unified-friends-list). The Discord Social SDK includes helper functions that group and sort friends according to those guidelines, avoiding manual relationship fetching and organization. The SDK places friends into the three groups defined by [`RelationshipGroupType`](https://discord.com/developers/docs/social-sdk/namespacediscordpp.html#a503ed2f7b0bfbd435321a0e8b1dfba35):
- `OnlinePlayingGame`: Friends who are online and playing your game
- `OnlineElsewhere`: Friends who are online but are not playing your game
- `Offline`: Friends who are offline

### Step 1: Display the Unified Friends List

 The [`Client::GetRelationshipsByGroup`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a9f7898d3f3d1ec92b06c662df70746d5) method returns pre-sorted relationships for a given group type, so you do not need to filter, categorize, or sort friends manually. The SDK decides each friend's group from online status and game activity, then sorts users inside the group; for example, users who have played your game appear first in `OnlineElsewhere`. Create a function that uses these helpers to render an organized friends list:

```
void DisplayUnifiedFriendsList(const std::shared_ptr<discordpp::Client> &client) {
    // Get friends playing the game
  const auto onlineInGame = client->GetRelationshipsByGroup(
        discordpp::RelationshipGroupType::OnlinePlayingGame
    );

    // Get friends online elsewhere
    const auto onlineElsewhere = client->GetRelationshipsByGroup(
        discordpp::RelationshipGroupType::OnlineElsewhere
    );

    // Get offline friends
    const auto offline = client->GetRelationshipsByGroup(
        discordpp::RelationshipGroupType::Offline
    );

    // Display "Online - GameTitle" Friends
    std::cout << "\n=== Online - GameTitle (" << onlineInGame.size() << ") ===\n";
    for (const auto& relationship : onlineInGame) {
        auto user = relationship.User();
        if (user) {
            std::string displayStr = "🟣 " + user->DisplayName();

            // Add Discord friend indicator
            if (relationship.DiscordRelationshipType() == discordpp::RelationshipType::Friend) {
                displayStr += " 👾";
            }

            // Add game friend indicator
            if (relationship.GameRelationshipType() == discordpp::RelationshipType::Friend) {
                displayStr += " 🎮";
            }

            std::cout << displayStr << "\n";
        }
    }

    // Display "Online - Elsewhere" Friends
    std::cout << "\n=== Online - Elsewhere (" << onlineElsewhere.size() << ") ===\n";
    for (const auto& relationship : onlineElsewhere) {
        auto user = relationship.User();
        if (user) {
            std::string displayStr = "🟢 " + user->DisplayName();

            // Add Discord friend indicator
            if (relationship.DiscordRelationshipType() == discordpp::RelationshipType::Friend) {
                displayStr += " 👾";
            }

            // Add game friend indicator
            if (relationship.GameRelationshipType() == discordpp::RelationshipType::Friend) {
                displayStr += " 🎮";
            }

            std::cout << displayStr << "\n";
        }
    }

    // Display "Offline" Friends
    std::cout << "\n=== Offline (" << offline.size() << ") ===\n";
    for (const auto& relationship : offline) {
        auto user = relationship.User();
        if (user) {
            std::string displayStr = "⚫ " + user->DisplayName();

            // Add Discord friend indicator
            if (relationship.DiscordRelationshipType() == discordpp::RelationshipType::Friend) {
                displayStr += " 👾";
            }

            // Add game friend indicator
            if (relationship.GameRelationshipType() == discordpp::RelationshipType::Friend) {
                displayStr += " 🎮";
            }

            std::cout << displayStr << "\n";
        }
    }
}

```

### Step 2: Set Up Automatic Updates

 Use [`Client::SetRelationshipGroupsUpdatedCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#af12441ef091298f968075b7190851098) to keep the friends list current. The callback runs whenever a change could affect grouping, such as a friend coming online or going offline, or a relationship update caused by accepting a friend request or blocking a user.

```
// Set up the unified friends list update callback
client->SetRelationshipGroupsUpdatedCallback([&client](const uint64_t userId) {
    std::cout << "👥 Friends list updated for user: " << userId << std::endl;
    DisplayUnifiedFriendsList(client);
});

```

---

### Approach 2: Manually Fetching Relationships and Users

 This section shows a manual approach that gives you more control over how your game presents the friends list.

### Step 1: Fetch Relationships

 Start by creating a function that uses [`Client::GetRelationships`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ad481849835cd570f0e03adafcf90125d) to query relationships and user information for the account:

```
void DisplayFriendsList(discordpp::Client& client) {
    std::vector<std::string> relationships{};
    for (auto& relationship: client->GetRelationships()) {
        auto user = relationship.User();
        if (!user) {
            continue;
        }

        std::string str{};
        // Identifying information about the user:
        str += " DiscordName: " + user->DisplayName();
        str += " DiscordId: " + std::to_string(user->Id());
        // Provisional users don't have a Discord icon shown next to them:
        str += " IsProvisional: " + std::to_string(user->IsProvisional());
        // Whether the relationship is for a friend, a friend request, or because the user is blocked:
        // For a friends list you'll want to filter out blocked users
        // And likely display friend requests in a different section
        str += " DiscordRelationshipType: " + std::string(discordpp::EnumToString(relationship.DiscordRelationshipType()));
        str += " GameRelationshipType: " + std::string(discordpp::EnumToString(relationship.GameRelationshipType()));
        // Whether the user is online/offline/etc:
        str += " IsOnlineAnywhere: " + std::to_string(user->Status() != discordpp::StatusType::Offline);
        str += " IsOnlineInGame: " + std::to_string(user->GameActivity() != std::nullopt);
        relationships.push_back(str);
    }

    std::sort(relationships.begin(), relationships.end());
    for (auto str : relationships) {
        printf("%s\n", str.c_str());
    }
}

```

 The `relationship.User()` function returns a [`UserHandle`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1UserHandle.html#a587bcc838e42dc5c56f840a350070707) object for a user known to the Discord Social SDK. Handle objects keep references to the SDK instance and underlying data, so existing handles reflect data updates without being recreated. Call this function once the client is ready by adding it to the ready callback:

```
// Set up status callback to monitor client connection
client->SetStatusChangedCallback([client](discordpp::Client::Status status, discordpp::Client::Error error, int32_t errorDetail) {
    std::cout << "🔄 Status changed: " << discordpp::Client::StatusToString(status) << std::endl;

    if (status == discordpp::Client::Status::Ready) {
        std::cout << "✅ Client is ready! You can now call SDK functions.\n";
        std::cout << "👥 Friends Count: " << client->GetRelationships().size() << std::endl;

        SetRichPresence(client);
        DisplayFriendsList(client);

    } else if (error != discordpp::Client::Error::None) {
        std::cerr << "❌ Connection Error: " << discordpp::Client::ErrorToString(error) << " - Details: " << errorDetail << std::endl;
    }
});

```

 This prints raw relationship data to the console. Use the data to filter, organize, and assemble a friends list that matches your game's design.

---

### Step 2: Organize Relationships

 The [Unified Friends List](https://docs.discord.com/developers/discord-social-sdk/design-guidelines/unified-friends-list) design guidelines split the player's friends into three sections: `Online - GameTitle`, `Online - Elsewhere`, and `Offline`. Because this sample is a text console application, it uses emojis for friend status, but your game can use its own visual language for status and presence. Update `DisplayFriendsList` so it creates those sections and categorizes friends by status:
- Create three vectors to hold friends for each category.
- Exclude pending friends and blocked users.
- Add indicators for Discord friends, game friends, and provisional users.
- Categorize friends from their game and presence status.
- Sort each category alphabetically.
- Render each category separately.
 This example is illustrative only. Use efficient data structures and cache data where suitable to avoid performance issues in your game.

```
void DisplayFriendsList(std::shared_ptr<discordpp::Client> client) {
    // Create vectors for each section
    std::vector<std::string> inGame;
    std::vector<std::string> online;
    std::vector<std::string> offline;

    for (auto& relationship : client->GetRelationships()) {
        auto user = relationship.User();
        if (!user) {
            continue;
        }

        // Filter out pending friends and blocked users
        // You can display friend requests and blocked users in a different view to allow players to manage them in your game
        if (relationship.DiscordRelationshipType() != discordpp::RelationshipType::Friend) {
            continue;
        }

        std::string str;
        str += user->DisplayName();

        // Add Discord friend indicator
        // In a real game, please use the official Discord logo available in our design guidelines
        if (relationship.DiscordRelationshipType() == discordpp::RelationshipType::Friend) {
            str += " 👾";
        }

        // Add game friend indicator
        if (relationship.GameRelationshipType() == discordpp::RelationshipType::Friend) {
            str += " 🎮";
        }

        // Add provisional indicator
        if (user->IsProvisional()) {
            str += " (Provisional)";
        }

        // Categorize based on status
        if (user->GameActivity()) {
            // in game
            inGame.push_back("🟣 " + str);
        } else if (user->Status() != discordpp::StatusType::Offline) {
            // online
            online.push_back("🟢 " + str);
        } else {
            // offline
            offline.push_back("⚫ " + str);
        }
    }

    // Sort each category
    std::sort(inGame.begin(), inGame.end());
    std::sort(online.begin(), online.end());
    std::sort(offline.begin(), offline.end());

    // Display "Online - GameTitle" Friends
    std::cout << "\n=== Online - GameTitle (" << inGame.size() << ") ===\n";
    for (const auto& str : inGame) {
        std::cout << str << "\n";
    }

    // Display "Online - Elsewhere" Friends
    std::cout << "\n=== Online - Elsewhere (" << online.size() << ") ===\n";
    for (const auto& str : online) {
        std::cout << str << "\n";
    }

    // Display "Offline" Friends
    std::cout << "\n=== Offline (" << offline.size() << ") ===\n";
    for (const auto& str : offline) {
        std::cout << str << "\n";
    }
}

```

 After building and running the application, the friends list should be split into `Online - GameTitle`, `Online - Elsewhere`, and `Offline`.

### Step 3: Monitor Changes to Users

 Use [`Client::SetUserUpdatedCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a3559f375165acedc6d6677ef599b3a4a) to monitor user changes. The callback runs when user information changes, including name updates, presence transitions such as online or offline, or starting your game.

```
client->SetUserUpdatedCallback([&client](uint64_t userId) {
    std::cout << "👤 User updated: " << userId << std::endl;
    DisplayFriendsList(*client);
});

```

 The friends list can refresh automatically when a friend's presence changes, such as going online, going offline, or starting your game. Because [`UserHandle`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1UserHandle.html#a587bcc838e42dc5c56f840a350070707) updates itself with the latest user information, it is usually enough for reading current user data. [`Client::SetUserUpdatedCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a3559f375165acedc6d6677ef599b3a4a) is useful when you need to re-sort the list or perform similar work.

---

### Step 4: Monitor Changes in Relationships

 Set up two callbacks for relationship updates. These examples rebuild the friends list whenever a relationship changes. For better performance, maintain a collection of [`UserHandle`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1UserHandle.html#a587bcc838e42dc5c56f840a350070707) objects and add or remove entries as relationships change.

#### Relationship Created Callback

 This can occur when a user sends or accepts a friend invite, or blocks another user.

```
client->SetRelationshipCreatedCallback([&client](uint64_t userId, bool isDiscordRelationshipUpdate) {
    std::optional<discordpp::UserHandle> user = client->GetUser(userId);
    // if the userid is valid (which it should be), we can display the user's display name
    if(user) {
        std::cout << "🤝 Relationship created: " << user->DisplayName() << std::endl;
        DisplayFriendsList(*client);
    }
});

```

#### Relationship Deleted Callback

 This can occur when a user rejects a friend request or removes a friend.

```
client->SetRelationshipDeletedCallback([&client](uint64_t userId, bool isDiscordRelationshipUpdate) {
    std::cout << "🔥 Relationship deleted: " << userId << std::endl;
    DisplayFriendsList(*client);
});

```

 The friends list can refresh automatically when relationships change, including adding a new friend, accepting a friend request, or blocking a user.

---

### Next Steps

 With a unified friends list in place, continue building social features such as relationship management and game invites. These guides cover the next areas:

### Designing a Unified Friends List

Best practices for designing a Unified Friends List.

### Managing Relationships

Let players manage friends, friend requests, and blocked users.

### Managing Game Invites

Let players invite friends into a game session or party. Need help? Join the [Discord Developers Server](https://discord.gg/discord-developers) and ask in `#social-sdk-dev-help` for community support. Report Social SDK bugs at [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report).

---

### Change Log

| Date | Changes |
| --- | --- |
| March 17, 2025 | Initial release |
| July 17, 2025 | Add UFL helper methods |
