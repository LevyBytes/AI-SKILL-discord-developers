# Using C++ (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

### Overview

 This guide shows how to add the Discord Social SDK to a new standalone C++ project. By the end, the application can:
- Authenticate users with Discord
- Configure logging and monitor status
- Initialize the SDK and connect it
- Request how many Discord friends the player has
- Update the player's rich presence for your game

### Prerequisites

 The guide uses a simple C++ console application. Have these prerequisites ready:
- Basic knowledge of C++ and the build system used by your platform
- C++ compiler and build system with [C++20](https://en.cppreference.com/w/cpp/20) or greater support.
- Network access to Discord's API endpoints.
 The detailed walkthrough follows.

---

### Step 1: Create a Discord Developer Team

 Before integrating the SDK, create a developer team in the Discord Developer Portal. The team owns your Discord applications and SDK integrations. Skip this step if a team already exists.
1. Create a developer team in the [Discord Developer Portal](https://discord.com/developers/teams).
 Afterward, invite teammates so they can collaborate on the integration.

---

### Step 2: Create a Discord Application

1. Create an application in the [Discord Developer Portal](https://discord.com/developers/applications) and place it under your team.
2. Add a redirect URL on the [OAuth2 tab](https://discord.com/developers/applications/select/oauth2):
  - Desktop applications can use `http://127.0.0.1/callback` and you can change it later.
  - See [`discordpp::Client::Authorize`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ace94a58e27545a933d79db32b387a468) for details on configuring more advanced redirect URIs.

3. Turn on the `Public Client` toggle in the [OAuth2 tab](https://discord.com/developers/applications/select/oauth2).
 This guide uses **Public Client** so you can begin with the SDK quickly. Most games should not ship as a public client; review this setting with your team before release. [Learn more about public clients](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes#oauth2-client-types).

---

### Step 3: Enable Discord Social SDK for Your App

 After creating the Discord application, enable the Discord Social SDK on that app.
1. Open the [Discord Developer Portal](https://discord.com/developers/applications) and choose the application created in Step 2.
2. In the app sidebar, open [Getting Started](https://discord.com/developers/applications/select/social-sdk/getting-started) under `Discord Social SDK`.
3. Complete the form with information about your game.
4. Click `Submit` to enable the Social SDK for the application.
5. After enablement, Social SDK binaries appear under [Downloads](https://discord.com/developers/applications/select/social-sdk/downloads).

---

### Step 4: Download the Discord SDK for C++

1. In the sidebar's Discord Social SDK section, open [Downloads](https://discord.com/developers/applications/select/social-sdk/downloads).
2. Choose the latest version in the version dropdown, then download the SDK for C++.

### Runtime Dependencies

 When distributing a build with the Social SDK, include its runtime dependencies:
- **Windows:** Put `discord_partner_sdk.dll` in the executable directory.
- **Linux/macOS:** Ensure `libdiscord_partner_sdk.so/libdiscord_partner_sdk.dylib` can be found through `LD_LIBRARY_PATH` or placed next to the binary.

---

### Step 5: Project Setup

 Using the Discord Social SDK from C++ requires the following:
- Include `discordpp.h` from your C++ source.
- Link the correct SDK libraries for your platform in your build system:
  - **Windows:** `discord_partner_sdk.dll`
  - **Linux**: `libdiscord_partner_sdk.so`
  - **macOS:** `libdiscord_partner_sdk.dylib`

 The SDK download archive contains all of these files. **Troubleshooting Tip:** unresolved external symbols usually mean the SDK library is not linked correctly in your build. Start by creating the project folder and Social SDK dependency layout:

```
mkdir MyGame
cd MyGame
mkdir lib

```

 Unzip the Social SDK archive into `lib`. When finished, `lib` should contain a `discord_social_sdk` folder. Inside the `MyGame` directory, create `CMakeLists.txt`:

```
touch CMakeLists.txt

```

 Place the following contents in `CMakeLists.txt` to configure the Social SDK dependency for the current operating system:

```
cmake_minimum_required(VERSION 3.10)
project(DiscordSDKExample)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED True)

add_executable(DiscordSDKExample main.cpp)

# Define some handy Social SDK variables
set(DISCORD_SDK_ROOT "${CMAKE_SOURCE_DIR}/lib/discord_social_sdk")
set(DISCORD_SDK_LIB_DIR "${DISCORD_SDK_ROOT}/lib/release")
set(DISCORD_SDK_BIN_DIR "${DISCORD_SDK_ROOT}/bin/release")
set(DISCORD_SDK_INCLUDE_DIR "${DISCORD_SDK_ROOT}/include")

# Include for Social SDK headers
target_include_directories(DiscordSDKExample PRIVATE ${DISCORD_SDK_INCLUDE_DIR})

# Platform-specific Social SDK library paths
if(WIN32)
    set(DISCORD_LIB_PATH "${DISCORD_SDK_LIB_DIR}/discord_partner_sdk.lib")
    set(DISCORD_SHARED_LIB "${DISCORD_SDK_BIN_DIR}/discord_partner_sdk.dll")
elseif(APPLE)
    set(DISCORD_LIB_PATH "${DISCORD_SDK_LIB_DIR}/libdiscord_partner_sdk.dylib")
    set(DISCORD_SHARED_LIB "${DISCORD_SDK_LIB_DIR}/libdiscord_partner_sdk.dylib")
else() # Linux
    set(DISCORD_LIB_PATH "${DISCORD_SDK_LIB_DIR}/libdiscord_partner_sdk.so")
    set(DISCORD_SHARED_LIB "${DISCORD_SDK_LIB_DIR}/libdiscord_partner_sdk.so")
endif()

# Link the Social SDK library
target_link_libraries(DiscordSDKExample PRIVATE ${DISCORD_LIB_PATH})

# Set the runtime search path (RPATH) for Linux and macOS
if(UNIX)
    # Use RPATH when building
    set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
    # Set the RPATH to use the lib directory relative to the executable
    set(CMAKE_INSTALL_RPATH "$ORIGIN")
    if(APPLE)
        set(CMAKE_INSTALL_RPATH "@executable_path")
    endif()
endif()

# Copy Social SDK shared library to output directory, so it's available at runtime.
add_custom_command(TARGET DiscordSDKExample POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
    "${DISCORD_SHARED_LIB}"
    $<TARGET_FILE_DIR:DiscordSDKExample>
)

```

 Create a main.cpp file:

```
touch main.cpp

```

 Add this code to `main.cpp`:

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
const uint64_t APPLICATION_ID = 123456789012345678;

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

  // Keep application running to allow SDK to receive events and callbacks
  while (running) {
      std::this_thread::sleep_for(std::chrono::milliseconds(10));
  }

  return 0;
}

```

 Compile and run the project:

```
mkdir build && cd build
cmake ..
cmake --build .
./DiscordSDKExample

```

 At this stage the code has mostly created a client instance, so the console should print output like this:

```
🚀 Initializing Discord SDK...

```

### Troubleshooting

Mac `libdiscord_partner_sdk.dylib` blocked from opening

On Mac, Apple may block `libdiscord_partner_sdk.dylib` with a message saying it could not be verified. If that popup appears, press **Done**. Open **System Settings > Privacy & Security**, scroll to **Security**, and look for the notice that `libdiscord_partner_sdk.dylib` was blocked to protect your Mac. Press **Open Anyway** and run the app again. On the next popup, choose **Open Anyway** so the library can be used. (Figure: Error)(Figure: Settings)(Figure: Open)

---

### Step 6: Setting Up SDK Event Handling

 Next, add event handlers so the application can observe the Discord connection. The setup uses two key callbacks:
1. A logging callback for SDK activity
2. A status callback that indicates when Discord features are ready

### Adding Logging Support

 Start by adding logging so SDK behavior is visible:

```
client->AddLogCallback([](auto message, auto severity) {
  std::cout << "[" << EnumToString(severity) << "] " << message << std::endl;
}, discordpp::LoggingSeverity::Info);

```

### Monitoring Connection Status

 Then add a status callback so the program knows when Discord features can be used:

```
client->SetStatusChangedCallback([client](discordpp::Client::Status status, discordpp::Client::Error error, int32_t errorDetail) {
  std::cout << "🔄 Status changed: " << discordpp::Client::StatusToString(status) << std::endl;

  if (status == discordpp::Client::Status::Ready) {
    std::cout << "✅ Client is ready! You can now call SDK functions.\n";
  } else if (error != discordpp::Client::Error::None) {
    std::cerr << "❌ Connection Error: " << discordpp::Client::ErrorToString(error) << " - Details: " << errorDetail << std::endl;
  }
});

```

main.cpp with event handling

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
const uint64_t APPLICATION_ID = 123456789012345678;

// Create a flag to stop the application
std::atomic<bool> running = true;

// Signal handler to stop the application
void signalHandler(int signum) {
    running.store(false);
}

int main() {
    std::signal(SIGINT, signalHandler);
    std::cout << "🚀 Initializing Discord SDK...\n";

    // Create Discord Client
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
      } else if (error != discordpp::Client::Error::None) {
        std::cerr << "❌ Connection Error: " << discordpp::Client::ErrorToString(error) << " - Details: " << errorDetail << std::endl;
      }
    });

    // Keep application running to allow SDK to receive events and callbacks
    while (running) {
      std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }

    return 0;
}

```

 Most Discord features are unavailable until the status reaches [`Client::Status::Ready`]. The status callback tells you when that point is reached.

### What These Callbacks Do

- The **logging callback** exposes SDK activity that would otherwise be hidden
- The **status callback** reports when the client is connected and ready for Discord features
 At this point, the callbacks **won't get called** because client setup is incomplete. They will soon be used to print debug information and connection status. To reach [`Client::Status::Ready`], authenticate with Discord next.

---

### Step 7: Run Callbacks

 After registering SDK callbacks, run them from your program's event loop. Add something like this to the main loop or tick function. The example adds [`RunCallbacks`](https://discord.com/developers/docs/social-sdk/namespacediscordpp.html#ab5dd8cf274f581ee1885de5816be3c29) to the main loop:

```
 // Keep application running to allow SDK to receive events and callbacks
  while (running) {
    discordpp::RunCallbacks();
    std::this_thread::sleep_for(std::chrono::milliseconds(10));
  }

```

---

### Step 8: Account Linking with Discord

 This step adds OAuth2 authentication for account linking with Discord. The process will:
1. Open the Discord app or a browser for Discord login
2. Get an authorization code
3. Exchange the authorization code for an access token
4. Connect to Discord

### Add the Authentication Code

 After the status callback setup in `main.cpp`, add this authentication code. **Choosing your OAuth2 scopes:** The guide uses [`Client::GetDefaultPresenceScopes`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a7648bd1d2f7d9a86ebd0edb8bef12b5c), which requests `openid` and `sdk.social_layer_presence`. Those scopes enable account linking, friends list access, rich presence, and other core behavior. If the game also needs lobbies, voice chat, or direct messaging, use [`Client::GetDefaultCommunicationScopes`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a71499da752fbdc2d4326ae0fd36c0dd1). See the [OAuth2 Scopes guide](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes) for the complete breakdown.

```
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
    });
  }
});

```

### What’s Happening Here?

1. Create an OAuth2 PKCE code verifier
2. Configure authorization arguments with the app ID and required scopes
3. Call [`Client::Authorize`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ace94a58e27545a933d79db32b387a468) to begin the auth flow and open a browser
4. After authorization, exchange the code for an access token
 Never log access tokens or store them insecurely; treat them as sensitive credentials.

### Testing It Out

1. Build and run your program
2. A browser window should ask you to authorize the app
3. After approval, the console should report that the access token was received
 If problems occur:
- Double-check that APPLICATION_ID is correct
- Confirm that the redirect URL is registered in the Discord Developer Portal
- Review the console for concrete error messages

main.cpp with Account Linking

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
const uint64_t APPLICATION_ID = 123456789012345678;

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

### Step 9: Connect the SDK to Discord

 With an access token available, connect the SDK to Discord. This takes two steps:
1. Give the SDK the access token by using [`Client::UpdateToken`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a606b32cef7796f7fb91c2497bc31afc4).
2. Start the connection with [`Client::Connect`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a873a844c7c4c72e9e693419bb3e290aa).
 Find `// Next Step: Update the token and connect` inside `client->UpdateToken()`, then add the following code after it:

```
// Next Step: Update the token and connect
client->UpdateToken(discordpp::AuthorizationTokenType::Bearer,  accessToken, [client](discordpp::ClientResult result) {
  if(result.Successful()) {
    std::cout << "🔑 Token updated, connecting to Discord...\n";
    client->Connect();
  }
});

```

### What’s Happening Here?

1. `client->UpdateToken()` tells the SDK to use the access token for Discord API calls
2. After the token update completes, call `client->Connect()` in the callback
3. The SDK starts connecting asynchronously
4. The status callback from Step 6 reports when the client is ready
 Watch the console output; it should show status changes while the connection is established.

### Testing the Connection

1. Run your program
2. Watch for status messages in this order:
  - ”🔑 Token updated, connecting to Discord…”
  - Status changes should include Connecting, Connected, and Ready
  - The ready message should indicate that SDK functions can now be called

### Troubleshooting

 If Ready never appears:
- Check whether the access token is valid
- Ensure you have internet connectivity
- Inspect error messages reported by the status callback
- Verify that `APPLICATION_ID` is correct
 Once the client reaches a ready state, you can begin adding Discord social features.

main.cpp with SDK connection

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

### Step 10: Access Discord Relationships
