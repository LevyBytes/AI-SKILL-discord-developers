# Using C++ (part 2)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Read the user's Discord relationships, or friends list, and show the count so the game can start using Discord data. Inside `client->SetStatusChangedCallback()`, after the `status == discordpp::Client::Status::Ready` branch, add this code to print how many Discord friends are available:

```
// Access initial relationships data
std::cout << "👥 Friends Count: " << client->GetRelationships().size() << std::endl;

```

### What This Code Does

1. [`Client::GetRelationships`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ad481849835cd570f0e03adafcf90125d) returns the current friend list immediately

### Example Output

```
👥 Friends Count: 42

```

### Testing It Out

1. Run your program
2. Wait until the initial friend count is printed
 Relationship data can back friend lists, activity feeds, and multiplayer invites.

### Troubleshooting

 If relationship data is missing:
- Confirm the OAuth2 scopes grant relationships access
- Confirm the client is connected with `Status::Ready`
- Confirm the account has Discord friends
- Inspect the logging callback for errors
 Next, add Rich Presence so Discord can show game activity.

main.cpp with friends count

```
#define DISCORDPP_IMPLEMENTATION
#include "discordpp.h"
#include <iostream>
#include <thread>
#include <atomic>
#include <string>
#include <functional>
#include <csignal>

// Replace with your Discord Application ID
const uint64_t APPLICATION_ID = 1349146942634065960;

// Create a flag to stop the application
std::atomic<bool> running = true;

// Signal handler to stop the application
void signalHandler(int signum) {
    running.store(false);
}

int main() {
    std::signal(SIGINT, signalHandler);
    std::cout << "🚀 Initializing Discord SDK...\n";

    // Create our Discord Client
    auto client = std::make_shared<discordpp::Client>();

    // Set up logging callback
    client->AddLogCallback([](auto message, auto severity) {
      std::cout << "[" << EnumToString(severity) << "] " << message << std::endl;
    }, discordpp::LoggingSeverity::Info);

    // Set up status callback to monitor client connection
    client->SetStatusChangedCallback([client](discordpp::Client::Status status, discordpp::Client::Error error, int32_t errorDetail) {
      std::cout << "🔄 Status changed: " << discordpp::Client::StatusToString(status) << std::endl;

      if (status == discordpp::Client::Status::Ready) {
        std::cout << "✅ Client is ready! You can now call SDK functions.\n";

        // Access initial relationships data
        std::cout << "👥 Friends Count: " << client->GetRelationships().size() << std::endl;

      } else if (error != discordpp::Client::Error::None) {
        std::cerr << "❌ Connection Error: " << discordpp::Client::ErrorToString(error) << " - Details: " << errorDetail << std::endl;
      }
    });

    // Generate OAuth2 code verifier for authentication
    auto codeVerifier = client->CreateAuthorizationCodeVerifier();

    // Set up authentication arguments
    discordpp::AuthorizationArgs args{};
    args.SetClientId(APPLICATION_ID);
    args.SetScopes(discordpp::Client::GetDefaultPresenceScopes());
    args.SetCodeChallenge(codeVerifier.Challenge());

    // Begin authentication process
    client->Authorize(args, [client, codeVerifier](auto result, auto code, auto redirectUri) {
      if (!result.Successful()) {
        std::cerr << "❌ Authentication Error: " << result.Error() << std::endl;
        return;
      } else {
        std::cout << "✅ Authorization successful! Getting access token...\n";

        // Exchange auth code for access token
        client->GetToken(APPLICATION_ID, code, codeVerifier.Verifier(), redirectUri,
          [client](discordpp::ClientResult result,
          std::string accessToken,
          std::string refreshToken,
          discordpp::AuthorizationTokenType tokenType,
          int32_t expiresIn,
          std::string scope) {
            std::cout << "🔓 Access token received! Establishing connection...\n";
            // Next Step: Update the token and connect
            client->UpdateToken(discordpp::AuthorizationTokenType::Bearer,  accessToken, [client](discordpp::ClientResult result) {
              if(result.Successful()) {
                std::cout << "🔑 Token updated, connecting to Discord...\n";
                client->Connect();
              }
            });
        });
      }
    });

    // Keep application running to allow SDK to receive events and callbacks
    while (running) {
      discordpp::RunCallbacks();
      std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }

    return 0;
}

```

---

### Step 11: Set Rich Presence

 Add Rich Presence so Discord can show the player's game activity in the Discord friends list.

### Add Rich Presence Code

 After the `client->GetRelationships()` call, add this code to set your rich presence:

```
// Configure rich presence details
discordpp::Activity activity;
activity.SetType(discordpp::ActivityTypes::Playing);
activity.SetState("In Competitive Match");
activity.SetDetails("Rank: Diamond II");

// Update rich presence
client->UpdateRichPresence(activity, [](discordpp::ClientResult result) {
  if(result.Successful()) {
    std::cout << "🎮 Rich Presence updated successfully!\n";
  } else {
    std::cerr << "❌ Rich Presence update failed";
  }
});

```

### What This Code Does

1. Builds an [`Activity`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Activity.html#ae793d9adbe16fef402b859ba02bee682) object describing what the player is doing
2. Sets basic information like:

- The activity type (Playing)
- Current state (“In Competitive Match”)
- Additional details (“Rank: Diamond II”)

1. Sends the rich presence update to Discord

### Testing It Out

1. Run your program
2. Watch the console for `Rich Presence updated successfully!`
3. Check your Discord profile; it should show:

- “Playing [Your Game]”
- “In Competitive Match”
- “Rank: Diamond II”

### Troubleshooting

 If presence does not appear:
- Confirm the client is connected ([`Client::Status::Ready`])
- Inspect the callback for error messages
- Confirm the activity settings are valid
- Confirm your Discord status is not invisible

main.cpp complete

```
#define DISCORDPP_IMPLEMENTATION
#include "discordpp.h"
#include <iostream>
#include <thread>
#include <atomic>
#include <string>
#include <functional>
#include <csignal>

// Replace with your Discord Application ID
const uint64_t APPLICATION_ID = 1349146942634065960;

// Create a flag to stop the application
std::atomic<bool> running = true;

// Signal handler to stop the application
void signalHandler(int signum) {
    running.store(false);
}

int main() {
    std::signal(SIGINT, signalHandler);
    std::cout << "🚀 Initializing Discord SDK...\n";

    // Create our Discord Client
    auto client = std::make_shared<discordpp::Client>();

    // Set up logging callback
    client->AddLogCallback([](auto message, auto severity) {
      std::cout << "[" << EnumToString(severity) << "] " << message << std::endl;
    }, discordpp::LoggingSeverity::Info);

    // Set up status callback to monitor client connection
    client->SetStatusChangedCallback([client](discordpp::Client::Status status, discordpp::Client::Error error, int32_t errorDetail) {
      std::cout << "🔄 Status changed: " << discordpp::Client::StatusToString(status) << std::endl;

      if (status == discordpp::Client::Status::Ready) {
        std::cout << "✅ Client is ready! You can now call SDK functions.\n";

        // Access initial relationships data
        std::cout << "👥 Friends Count: " << client->GetRelationships().size() << std::endl;

        // Configure rich presence details
        discordpp::Activity activity;
        activity.SetType(discordpp::ActivityTypes::Playing);
        activity.SetState("In Competitive Match");
        activity.SetDetails("Rank: Diamond II");

        // Update rich presence
        client->UpdateRichPresence(activity, [](discordpp::ClientResult result) {
          if(result.Successful()) {
            std::cout << "🎮 Rich Presence updated successfully!\n";
          } else {
            std::cerr << "❌ Rich Presence update failed";
          }
        });

      } else if (error != discordpp::Client::Error::None) {
        std::cerr << "❌ Connection Error: " << discordpp::Client::ErrorToString(error) << " - Details: " << errorDetail << std::endl;
      }
    });

    // Generate OAuth2 code verifier for authentication
    auto codeVerifier = client->CreateAuthorizationCodeVerifier();

    // Set up authentication arguments
    discordpp::AuthorizationArgs args{};
    args.SetClientId(APPLICATION_ID);
    args.SetScopes(discordpp::Client::GetDefaultPresenceScopes());
    args.SetCodeChallenge(codeVerifier.Challenge());

    // Begin authentication process
    client->Authorize(args, [client, codeVerifier](auto result, auto code, auto redirectUri) {
      if (!result.Successful()) {
        std::cerr << "❌ Authentication Error: " << result.Error() << std::endl;
        return;
      } else {
        std::cout << "✅ Authorization successful! Getting access token...\n";

        // Exchange auth code for access token
        client->GetToken(APPLICATION_ID, code, codeVerifier.Verifier(), redirectUri,
          [client](discordpp::ClientResult result,
          std::string accessToken,
          std::string refreshToken,
          discordpp::AuthorizationTokenType tokenType,
          int32_t expiresIn,
          std::string scope) {
            std::cout << "🔓 Access token received! Establishing connection...\n";
            // Next Step: Update the token and connect
            client->UpdateToken(discordpp::AuthorizationTokenType::Bearer,  accessToken, [client](discordpp::ClientResult result) {
              if(result.Successful()) {
                std::cout << "🔑 Token updated, connecting to Discord...\n";
                client->Connect();
              }
            });
        });
      }
    });

    // Keep application running to allow SDK to receive events and callbacks
    while (running) {
      discordpp::RunCallbacks();
      std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }

    return 0;
}

```

---

### Conclusion

 The Discord Social SDK is now integrated into your C++ application. Review the completed work:

### What You’ve Built

- Set up a Discord application with OAuth2
- Added SDK logging and status monitoring
- ✅ Implemented user authentication flow
- ✅ Retrieved Discord relationships data
- ✅ Added Rich Presence support

### Key Concepts Learned

- Initializing and configuring the Discord SDK
- Managing authentication and connections
- Using Discord social features
- Handling asynchronous callbacks
- Monitoring SDK status and events

---

### Next Steps

 The Discord Social SDK is set up in C++ and authenticated with Discord. You can now add more social features to your project with the SDK.

### Creating a Unified Friends List

Build one friends list that merges Discord relationships with game-specific friendships

### Setting Rich Presence

Extend your game's rich presence with richer details and game invite support

### Managing Game Invites

Let players invite friends into a game session or party. For help, join the [Discord Developers Server](https://discord.gg/discord-developers) and ask in `#social-sdk-dev-help`. Report Social SDK bugs at [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report).

---

### Change Log

| Date | Changes |
| --- | --- |
| May 20, 2026 | The 1.9 release changed the required C++ standard to C++20. |
| March 17, 2025 | initial release |
