# Managing Game Invites

> Discord Developers reference. Original prose; identifiers preserved verbatim.

### Overview

 Game Invites let users ask other people to enter their game session or party. The feature is available in the Discord client and through the Social SDK. Game Invites depend completely on Rich Presence rather than operating as their own separate system. After Rich Presence includes party data, a join secret, and/or supported platforms, Discord turns invite behavior on automatically. This guide covers the Rich Presence setup that makes game invites available.

### Prerequisites

 Before starting, confirm you have the following:
- Finished the [Setting Rich Presence](https://docs.discord.com/developers/discord-social-sdk/development-guides/setting-rich-presence) guide
- Understand that invites require active Rich Presence with party data
- Configured the Discord Social SDK by following the [Getting Started guide](https://docs.discord.com/developers/discord-social-sdk/getting-started)
 To use this communication capability, include [`Client::GetDefaultCommunicationScopes`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a71499da752fbdc2d4326ae0fd36c0dd1) in your OAuth Scope configuration. The [OAuth Scopes Core Concepts Guide](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes) covers the available scopes.

---

### Configuring Rich Presence to Enable Game Invites

 Game invites, sometimes called activity invites, rely on rich presence. The [Setting Rich Presence](https://docs.discord.com/developers/discord-social-sdk/development-guides/setting-rich-presence) guide covers the basics; this section repeats the invite-specific pieces. Discord uses Rich Presence and Activity for the same user-state concept in several places: the SDK represents it with the [`Activity`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Activity.html#ae793d9adbe16fef402b859ba02bee682) class, and gateway payloads include it [in our gateway events](https://docs.discord.com/developers/events/gateway-events#activity-object). Do not confuse that with [Discord Activities](https://docs.discord.com/developers/activities/overview), which are embedded games that may display rich presence too.

### Setting Up Rich Presence

 This example sets rich presence in your game so game invites can use it.

```
// Create discordpp::Activity - This Rich Presence Activity is your invite configuration
discordpp::Activity activity;
activity.SetType(discordpp::ActivityTypes::Playing);

// Set the game state and details
activity.SetState("In Competitive Match");
activity.SetDetails("Valhalla");

// Update Rich Presence presence
client->UpdateRichPresence(activity, [](discordpp::ClientResult result) {
    if(result.Successful()) {
        std::cout << "🎮 Rich Presence updated successfully!\n";
        // Note: Invites are NOT yet enabled - we need party info and join secret
    } else {
        std::cerr << "❌ Rich Presence update failed";
    }
});

```

 Running the game makes the Discord client show "In Competitive Match" on "Valhalla". To send game invites, configure the rich presence [`Activity`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Activity.html#ae793d9adbe16fef402b859ba02bee682) with party data and a join secret, then add the remaining invite fields.

### Adding an Activity Party

```
// rest of the code

// Create discordpp::ActivityParty
discordpp::ActivityParty party;
party.SetId("party1234");
// current party size
party.SetCurrentSize(1);
// max party size
party.SetMaxSize(5);
// Set the party information in the Activity, to inform the invite about the party size and how many players can join
activity.SetParty(party);

// Update Rich Presence
// Still not enough for invites - we need the join secret!

```

 After this change, the Discord client shows "In Competitive Match" on "Valhalla" with party details. Add the join secret next so the game can send and receive game invites.

### Adding Join Secret & Supported Platforms

 Add a join secret to the rich presence activity as the final required piece. The `Join Secret` is an opaque value your game can use to [join a Discord lobby](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-lobbies), a game session, or another destination. Game invites let players send that secret to other players; your game decides how to interpret it. Set supported platforms too, so the Discord client can show the invite button that matches the user's platform.

```
// Create discordpp::Activity

// Create ActivitySecrets
discordpp::ActivitySecrets secrets;
secrets.SetJoin("joinsecret1234");    // Rich Presence secret will be in the invite payload
activity.SetSecrets(secrets);

// Set supported platforms that can join the game
// See discordpp::ActivityGamePlatforms for available platforms
activity.SetSupportedPlatforms(discordpp::ActivityGamePlatforms::Desktop);

// Update Rich Presence
// ✅ NOW invites are enabled through Rich Presence!

```

### Putting It All Together

```
// Create discordpp::Activity - This Rich Presence Activity is your invite configuration
discordpp::Activity activity;
activity.SetType(discordpp::ActivityTypes::Playing);

// Set the game state and details
activity.SetState("In Competitive Match");
activity.SetDetails("Valhalla");

// Set the party information
discordpp::ActivityParty party;
party.SetId("party1234");
party.SetCurrentSize(1);          // current party size
party.SetMaxSize(5);              // max party size
// Set the party information in the Activity, to inform the invite about the party size and how many players can join
activity.SetParty(party);

// Create ActivitySecrets
discordpp::ActivitySecrets secrets;
secrets.SetJoin("joinsecret1234");    // Rich Presence secret will be in the invite payload
activity.SetSecrets(secrets);

// Set supported platforms that can join the game
// See discordpp::ActivityGamePlatforms for available platforms
activity.SetSupportedPlatforms(discordpp::ActivityGamePlatforms::Desktop);

// Update Rich Presence presence
client->UpdateRichPresence(activity, [](discordpp::ClientResult result) {
    if(result.Successful()) {
        std::cout << "🎮 Rich Presence updated successfully!\n";
        // ✅ Rich Presence updated = Game invites now available!
    } else {
        std::cerr << "❌ Rich Presence update failed";
        // ❌ No Rich Presence = No invites possible
    }
});

```

 Your game now publishes rich presence data that supports game invites. Friends who view the rich presence in the Discord client will see an invite button.

---

### Registering a Launch Command

 Before sending a game invite, register how Discord should launch your game when a user opens an invite. Launch registration is local, so it only applies on the machine where the call runs. Register the launch behavior every time the SDK starts inside your game client; then Discord can start the game for that player when they open an invite and the game is not already running. Choose one of these registration paths based on your distribution model:
- Register a command that launches your game
- Register a Steam Game ID

### Registering a Launch Command

 [`Client::RegisterLaunchCommand`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a024d7222931fdcb7d09c2b107642ecab) registers the command Discord should run on this machine to start your game. Call it during SDK startup so Discord can launch the game when the user opens an invite.

```
client->RegisterLaunchCommand(YOUR_APP_ID, "yourgame://");

```

### Registering a Steam Game

 For Steam distribution, [`Client::RegisterLaunchSteamApplication`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a45b2c791c5b06f77d457dacb53dfba40) registers the Steam game ID on this machine. Like command registration, run it during SDK startup so Discord can start the game when the user opens an invite.

```
client->RegisterLaunchSteamApplication(YOUR_APP_ID, STEAM_GAME_ID);

```

---

### Sending Game Invites

 **Testing invites requires a second account.** A user cannot view their own Rich Presence Invite, and invites sent to yourself will not arrive. Use a separate test account to exercise the full invite path. Game invites can be sent in two ways:
1. Users can send game invites from the Discord client.
2. Your game can send game invites for a user through the SDK.

### Sending Game Invites in the Discord Client

 Users can send game invites from the Discord client. The [Game Invites help center article](https://support.discord.com/hc/en-us/articles/115001557452-Game-Invites) explains the client experience.

### Sending Game Invites in the SDK

 When a player's rich presence includes the required party data, join secret, and supported platforms, your game can send invites through the SDK with [`Client::SendActivityInvite`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#afc14e98fc070399895739da6d53efa60). [`Client::SendActivityInvite`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#afc14e98fc070399895739da6d53efa60) requires active Rich Presence with the correct configuration. The invite and join behavior works with the [default presence scopes](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes#default-presence-scopes), but the invite message, such as `"Join my game!"`, is a Discord message and requires communication scopes. If the sender only has presence scopes, the invite can still be sent and accepted, but the message body will be empty.

```
uint64_t targetUserId = 1111785262289277050;
std::string inviteMessage = "Join my game!";
client->SendActivityInvite(targetUserId, inviteMessage, [](discordpp::ClientResult result) {
  if(result.Successful()) {
    std::cout << "Activity Invite sent to user" << std::endl;
  } else {
    std::cerr << "Failed - check if Rich Presence has party, secret, and platforms set" << std::endl;
  }
});

```

 (Figure: Example of a sent game invite in the Discord client)

---

### Receiving Game Invites

 Game invites can be received in two ways:
1. Users can receive game invites in the Discord client.
2. Your game can receive game invites for a user through the SDK.

### Receiving Game Invites in the Discord Client

 Users receive game invites in their DMs. The [Game Invites help center article](https://support.discord.com/hc/en-us/articles/115001557452-Game-Invites) describes the Discord client behavior.

### Receiving Game Invites in the SDK

 Use [`Client::SetActivityInviteCreatedCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a3b4e37a222a8662506d763514774bedc) to listen for new invites, and call [`Client::AcceptActivityInvite`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ad12cf35065e4d2b303ee470af7c6ef37) to accept one. The callback passed to [`Client::AcceptActivityInvite`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ad12cf35065e4d2b303ee470af7c6ef37) receives the join secret that was set in Rich Presence.

```
client->SetActivityInviteCreatedCallback([&client](discordpp::ActivityInvite invite) {
  std::cout << "Activity Invite received from user: " << invite.SenderId() << std::endl;
  if(auto message = client->GetMessageHandle(invite.MessageId())){
    std::cout << "Invite Message: " << message->Content() << std::endl;
  }
  client->AcceptActivityInvite(invite, [](discordpp::ClientResult result, std::string joinSecret) {
    if(result.Successful()) {
      std::cout << "Activity Invite accepted successfully!\n";
      // joinSecret comes from the sender's Rich Presence configuration
      // Use the joinSecret to connect the two players in your game
    } else {
      std::cerr << "❌ Activity Invite accept failed";
    }
  });
});

```

---

### Accepting Game Invites

 Use [`Client::SetActivityJoinCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a587d1c6d0352eba397c888987aa58418) to detect invite acceptance from either the game or Discord. The callback provides the join secret, which your title can resolve into the correct session for both players.

```
// This fires when a user clicks "Join" on someone's Rich Presence
client->SetActivityJoinCallback([&client](std::string joinSecret) {
  // joinSecret is pulled from the host's Rich Presence ActivitySecrets
  // Use the joinSecret to connect the players in your game
});

```

---

### Using Game Invites with Lobbies

 You can pair game invites with [Lobbies](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-lobbies) so players can join a game session or party smoothly. When a player accepts an invite, use the join secret to connect both players in your game. One possible sequence is:
- When a user begins playing, create a lobby with a random secret string by using [`Client::CreateOrJoinLobby`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a8b4e195555ecaa89ccdfc0acd28d3512)
- The user publishes Rich Presence with the join secret set to the lobby secret and includes party size data
- Another user sees that Rich Presence in Discord and requests to join
- After the invite is accepted, the joining user gets the join secret and can pass it to CreateOrJoinLobby(joinSecret) to enter the lobby
- The original user's client notices the lobby membership change, then publishes a Rich Presence update with the new party size
 These examples use client-side lobby management, but the same pattern can be adapted for server-created lobbies.

### Game Invite with Lobby Example

 This code sample demonstrates one way to implement the sequence:

```
// User A
// 1. Create a lobby with secret
std::string lobbySecret = "foo"
uint64_t USER_B_ID = 01234567890;
client->CreateOrJoinLobby(lobbySecret, [&client](discordpp::ClientResult result, uint64_t lobbyId) {
  // 2. Update Rich Presence with a party and join secret to enable invites
  discordpp::Activity activity{};
  activity.SetType(discordpp::ActivityTypes::Playing);
  activity.SetState("In Lobby");

  // Rich Presence party configuration for how many players can join
  discordpp::ActivityParty party{};
  party.SetId("party1234");
  party.SetCurrentSize(1);
  party.SetMaxSize(4);
  activity.SetParty(party);

  // Rich Presence secret is what is shared via invites
  discordpp::ActivitySecrets secrets{};
  secrets.SetJoin(lobbySecret);  // This connects Rich Presence to your lobby
  activity.SetSecrets(secrets);

  // Don't forget to set this Rich Presence update, otherwise SendActivityInvite won't work!
  client->UpdateRichPresence(std::move(activity), [&client](discordpp::ClientResult result) {
    // 3. NOW we can send invites because Rich Presence is configured
    client->SendActivityInvite(USER_B_ID, "come play with me", [](discordpp::ClientResult result) {
        if(result.Successful()) {
            std::cout << "💌 Invite sent successfully!\n";
        } else {
            std::cerr << "❌ Invite failed - check Rich Presence configuration\n";
        }
    });
  });
});

// User B
// 4. Monitor for new invites
client->SetActivityInviteCreatedCallback([&client](discordpp::ActivityInvite invite) {
  std::cout << "💌 New invite received: " << invite.SenderId() << "\n";
  // 5. When an invite is received, ask the user if they want to accept it.

  // If they choose to do so then go ahead and invoke AcceptActivityInvite
  client->AcceptActivityInvite(invite, [&client](discordpp::ClientResult result, std::string joinSecret) {
    if (result.Successful()) {
      std::cout << "🎮 Invite accepted! Joining lobby...\n";
      // joinSecret came from User A's Rich Presence configuration

      // 6. Join the lobby using the joinSecret
      client->CreateOrJoinLobby(joinSecret, [=](discordpp::ClientResult result, uint64_t lobbyId) {
        // Successfully joined lobby!
        if (result.Successful()) {
          std::cout << "🎮 Lobby joined successfully! " << lobbyId << std::endl;
        } else {
          std::cerr << "❌ Lobby join failed\n";
        }
      });
    }
  });
});

```

### Lobby Join Request Example

 Users can request access to each other's parties too. This code sample demonstrates that path:

```
// User A
// 1. Create a lobby with secret
std::string lobbySecret = "foo";
uint64_t USER_A_ID = 286438705638408203;
client->CreateOrJoinLobby(lobbySecret, [&client](discordpp::ClientResult result, uint64_t lobbyId) {
  // 2. Update Rich Presence with a party and join secret to enable invites
  discordpp::Activity activity{};
  activity.SetType(discordpp::ActivityTypes::Playing);
  activity.SetState("In Lobby");

  // Rich Presence party configuration for how many players can join
  discordpp::ActivityParty party{};
  party.SetId("party1234");
  party.SetCurrentSize(1);
  party.SetMaxSize(4);
  activity.SetParty(party);

  // Rich Presence secret is what is shared via invites
  discordpp::ActivitySecrets secrets{};
  secrets.SetJoin(lobbySecret);
  activity.SetSecrets(secrets);

  // This Rich Presence update is what enables the "Ask to Join" button in Discord
  client->UpdateRichPresence(std::move(activity), [&client](discordpp::ClientResult result) {});
});

// User B
// 3. Request to join User A's party
client->SendActivityJoinRequest(USER_A_ID, [](discordpp::ClientResult result) {});

// User A
// 4. Monitor for new invites:
client->SetActivityInviteCreatedCallback([&client](discordpp::ActivityInvite invite) {
  // 5. The game can now show that User A has received a request to join their party
  // If User A is ok with that, they can reply back:
  // Note: invite.type will be ActivityActionTypes::JoinRequest in this example
  client->SendActivityJoinRequestReply(invite, [](discordpp::ClientResult result) {});
});

// User B
// 6. Same as before, user B can monitor for invites
client->SetActivityInviteCreatedCallback([&client](discordpp::ActivityInvite invite) {
  std::cout << "💌 New invite received: " << invite.SenderId() << "\n";
  // 7. When an invite is received, ask the user if they want to accept it.
  // If they choose to do so then go ahead and invoke AcceptActivityInvite
  client->AcceptActivityInvite(invite, [&client](discordpp::ClientResult result, std::string joinSecret) {
    if (result.Successful()) {
      std::cout << "🎮 Invite accepted! Joining lobby...\n";
      // joinSecret came from User A's Rich Presence
      // 5. Join the lobby using the joinSecret
      client->CreateOrJoinLobby(joinSecret, [=](discordpp::ClientResult result, uint64_t lobbyId) {
        // Successfully joined lobby!
        if (result.Successful()) {
          std::cout << "🎮 Lobby joined successfully! " << lobbyId << std::endl;
        } else {
          std::cerr << "❌ Lobby join failed\n";
        }
      });
    }
  });
});

```

---

### Supporting Mobile Game Invites

 When a player receives a game invite on mobile, Discord needs launch information for your game. Mobile launching uses deep linking so Discord can pass join data into the game.

### Setting Up Mobile Deep Links

1. Add your deep link URL in the Discord Developer Portal:

- Open your application's `General` tab
- Enter the game's URL scheme, such as `yourgame://`
- Discord appends `/_discord/join?secret=SECRETHERE` to that URL

1. Tell Discord which platforms are allowed to accept invites:

```
activity.SetSupportedPlatforms(
    ActivityGamePlatforms.Desktop |  // Enable PC/Mac invites
    ActivityGamePlatforms.IOS |      // Enable iOS invites
    ActivityGamePlatforms.Android    // Enable Android invites
);

```

### How Mobile Deep Links Work

1. The user receives an invite in Discord and accepts it
2. Discord starts your game through your URL scheme:

```
yourgame://_discord/join?secret=the_join_secret_you_set

```

3. Your game receives the URL and reads the join secret
4. Use the secret to connect the player to the session

---

### Setting a Cover Image for the Invite (Optional)

 You can define an invite cover image through the rich presence activity. That image appears as the invite banner and can make the invite more noticeable. Because the image is set through rich presence, you can vary it per activity, such as by game mode or map, instead of relying on one static invite image. If no activity image is set, the banner uses the "Rich Presence Invite Image" configured on the "Rich Presence" tab in the Developer Portal. An invite cover image is optional.

```
discordpp::ActivityAssets assets;
assets.SetInviteCoverImage("invite-cover-image"); // This example uses an asset key, but you can use an external URL asset for more dynamic cover images.
activity.SetAssets(assets);

```

---

### Next Steps

### Creating a Unified Friends List

Merge Discord friends and in-game friends into one manageable list.

### Managing Lobbies

Use shared lobbies to gather players with invites, text chat, and voice comms.

### Sending Direct Messages

Support private player-to-player messaging. Need help? Join the [Discord Developers Server](https://discord.gg/discord-developers) and ask questions in `#social-sdk-dev-help` for community support. To report a Social SDK bug, use [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report)

---

### Change Log

| Date | Changes |
| --- | --- |
| June 10, 2026 | State that launch command registration applies only to the local machine |
| June 8, 2026 | Recommend communication scopes; clarify empty invite message behavior |
| March 17, 2025 | initial release |
