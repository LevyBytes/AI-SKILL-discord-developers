# Account Linking On Mobile

> Discord Developers reference. Original prose; identifiers preserved verbatim.

### Overview

 If account linking is new to you, read the [Account Linking overview](https://docs.discord.com/developers/platform/account-linking) for the concept, behavior, and flow choices. This guide covers OAuth2 authentication for mobile users on iOS and Android so they can link Discord accounts to your game. Mobile authentication relies on **deep linking**: when the Discord mobile app is installed, the user is sent to Discord for authorization and then returned to your game through a custom URL scheme. As of Discord Social SDK 1.5, native deep-link authentication support makes mobile account linking much simpler.

### Key Differences from Desktop Authentication

 Mobile platforms differ from desktop in several important ways:
- **Custom URL schemes** (deep links) are required instead of HTTP redirects
- **PKCE (Proof Key for Code Exchange)** is **mandatory** for every mobile app that uses deep links, whether the client is public or confidential
- **Platform-specific configuration** is required, including Info.plist for iOS and AndroidManifest.xml for Android

### Prerequisites

 Before starting, confirm you have the following:
- Read the [Core Concepts](https://docs.discord.com/developers/discord-social-sdk/core-concepts) guide so these topics are familiar:
  - OAuth2 authentication flow
  - Discord application setup
  - SDK initialization

- Prepared your development environment with:
  - A Discord application in the [Developer Portal](https://discord.com/developers/applications)
  - Discord Social SDK version 1.5 or higher downloaded and configured
  - A [Basic account linking flow](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord) already working on desktop

 If these prerequisites are not complete, follow the [Getting Started](https://docs.discord.com/developers/discord-social-sdk/getting-started) guide first.

---

### Configure OAuth2 Redirect URI

 OAuth2 on mobile requires the correct redirect URI to be registered for your app in the [**Discord Developer Portal**](https://discord.com/developers/applications/select/oauth2).
| Platform | Redirect URI |
| --- | --- |
| **Mobile (iOS & Android)** | `discord-YOUR_APP_ID:/authorize/callback` *(substitute your Discord application ID for `YOUR_APP_ID`)* |

 Use your application ID in the redirect URI format `discord-YOUR_APP_ID:/authorize/callback`. Do not use `http://127.0.0.1/callback` on mobile.

---

### Unity Setup

 Use the [Unity Getting Started guide](https://docs.discord.com/developers/discord-social-sdk/getting-started/using-unity) for the general Unity setup, then add these mobile-specific configuration steps:

### iOS Configuration

#### 1. Configure URL Scheme in Unity Project Settings

 To support [`Client::Authorize`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ace94a58e27545a933d79db32b387a468), configure a callback URL scheme:
1. Open Project Settings: `Edit -> Project Settings...`
2. Navigate to the `Player` section
3. Select the `iOS` tab
4. Under `Other Settings`, find `Supported URL Schemes`
5. Add `discord-YOUR_APP_ID` to the list; for example, if your application ID is `123456`, add `discord-123456`

#### 2. Set Microphone Usage Description

 On the same Player settings page:
1. Set `Microphone Usage Description` to a valid permission description
2. iOS displays this string when it asks for microphone permission
3. Required for voice support

#### 3. Configure Info.plist for Deep Linking

 Native authentication requires your `Info.plist` to include the `discord` scheme:

```
<key>LSApplicationQueriesSchemes</key>
<array>
  <string>discord</string>
</array>

```

 This lets your app detect whether the Discord mobile app is installed and open it through a deep link for authentication.

#### 4. Enable Background Voice Support (Optional)

 To keep voice support working while your game is backgrounded:
1. Update `Info.plist` with the needed background modes
2. The [Unity sample project](https://discord.com/developers/applications/select/social-sdk/downloads) includes a build postprocessor you can copy into your project at `Assets/Scripts/Editor/VoicePostBuildProcessor.cs`
 For background voice, choose *Audio, AirPlay, and Picture in Picture* background mode rather than Voice over IP. See [Configuring your app for Media Playback](https://developer.apple.com/documentation/avfoundation/configuring-your-app-for-media-playback?language=objc) for details.

### Android Configuration

#### 1. Configure Custom URL Scheme

 [`Client::Authorize`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ace94a58e27545a933d79db32b387a468) requires an activity with a custom URL scheme in your application manifest:
1. The [Unity sample project](https://discord.com/developers/applications/select/social-sdk/downloads) provides an example build processor at `Assets/Scripts/Editor/AndroidPostBuildProcessor.cs`
2. You can instead add an intent filter manually to `AndroidManifest.xml`:

```
<activity android:name="com.discord.socialsdk.AuthenticationActivity"
android:exported="true">
  <intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="discord-YOUR_APP_ID" />
  </intent-filter>
</activity>

```

 Replace `YOUR_APP_ID` with the real Discord application ID, such as `discord-1234567890123456789`.

#### 2. Add androidx.browser Dependency

 Authorization needs `androidx.browser` as a Gradle dependency:
- If your project uses **Google External Dependency Manager**, the Unity plugin includes a suitable dependencies XML file
- If not, add the dependency yourself in `build.gradle`:

```
dependencies {
    implementation 'androidx.browser:browser:1.8.0'
    // Your other dependencies...
}

```

#### 3. Android Permissions

 The Android SDK requests these permissions:

```
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_MICROPHONE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_MEDIA_PLAYBACK" />
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />

```

 If your application does not use voice features, you may remove every permission except `INTERNET` by using the `tools:node="remove"` attribute [in your AndroidManifest.xml](https://developer.android.com/build/manage-manifests#node_markers).

---

### Unreal Engine Setup

 Use the [Unreal Engine Getting Started guide](https://docs.discord.com/developers/discord-social-sdk/getting-started/using-unreal-engine) for general setup. For mobile, the only extra project setting required is your Discord Application ID.

### Configure Discord Application ID

 To turn on mobile authentication in Unreal Engine:
1. Open **Project Settings** (`Edit -> Project Settings...`)
2. Search for `discord` in the search bar
3. Under **Discord Social SDK**, enter your **Discord Application ID**
 (Figure: Unreal Engine Discord Application ID Setting) This setting automatically prepares `discord-YOUR_APP_ID://` as the custom URL scheme used by iOS and Android deep links.

---

### C++ Standalone Setup

 In a standard iOS or Android project, you can integrate the Discord Social SDK as a C++ library. Use the setup path for the target platform:

### iOS Setup

#### 1. Create and Configure Xcode Project

1. Create an Objective-C iOS project in Xcode
2. Add `discord_partner_sdk.xcframework` to the project

#### 2. Configure Build Settings

1. Add the xcframework to `Build Phases -> Link Binary with Libraries` if needed
2. In the `General` tab, under `Frameworks, Libraries and Embedded Content`:
  - Set `discord_partner_sdk.xcframework` to **Embed & Sign**

#### 3. Configure Background Audio (Optional)

 To keep voice connected while the app is backgrounded:
1. Configure background audio modes in `Info.plist` through the `Signing & Capabilities` tab
2. Select *Audio, AirPlay, and Picture in Picture*, **not** Voice over IP
3. See [Configuring your app for Media Playback](https://developer.apple.com/documentation/avfoundation/configuring-your-app-for-media-playback?language=objc) for detailed instructions

#### 4. Register URL Scheme

 To support [`Client::Authorize`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ace94a58e27545a933d79db32b387a468), register the proper URL scheme in `Info.plist`:

```
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>discord-YOUR_APP_ID</string>
        </array>
    </dict>
</array>
<key>LSApplicationQueriesSchemes</key>
<array>
    <string>discord</string>
</array>

```

 Replace `YOUR_APP_ID` with the application ID from the Discord Developer Portal; for example, if the app ID is `123456`, register `discord-123456`.

#### 5. Include SDK Headers

 In one C++ or Objective-C++ (`.mm`) source file:

```
#define DISCORDPP_IMPLEMENTATION // Define this in exactly ONE file
#include <discord_partner_sdk/discordpp.h>

```

 `DISCORDPP_IMPLEMENTATION` must be defined in **exactly one file** in your codebase so the required implementation code is expanded once.

#### 6. Run Callbacks in Game Loop

 Call this from your main game loop:

```
discordpp::RunCallbacks();

```

### Android Setup

#### 1. Create Android Project

 Create an Android game project from the **Game Activity (C++)** template.

#### 2. Add SDK as Dependency

 Add `discord_partner_sdk.aar` as a Gradle project dependency:
1. Place the AAR in a directory such as `app/libs`
2. Add it to dependencies in `app/build.gradle`:

```
dependencies {
    implementation files("libs/discord_partner_sdk.aar")
}

```

#### 3. Enable Prefab

 Make sure Prefab is enabled in the Gradle build. See [Native Dependencies in AARs](https://developer.android.com/build/native-dependencies#native-dependencies-aars) for details.

#### 4. Configure CMake

 In your `CMakeLists.txt`:

```
find_package(discord_partner_sdk REQUIRED CONFIG)
target_link_libraries(your_target discord_partner_sdk::discord_partner_sdk)

```

#### 5. Include SDK Headers

 In your C++ source:

```
#define DISCORDPP_IMPLEMENTATION // Define this in exactly ONE file
#include "discordpp.h"

```

 `DISCORDPP_IMPLEMENTATION` must be defined in **exactly one file** in your codebase.

#### 6. Initialize SDK in Activity

 In your main activity's `onCreate` method (Java/Kotlin):

```
com.discord.socialsdk.DiscordSocialSdkInit.setEngineActivity(this);

```

#### 7. Run Callbacks in Game Loop

 In your main C++ loop:

```
discordpp::RunCallbacks();

```

#### 8. Configure Authorization Support

 To support [`Client::Authorize`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ace94a58e27545a933d79db32b387a468):
1. Add the `androidx.browser` dependency, version 1.8 or later, to `build.gradle`:

```
dependencies {
    implementation 'androidx.browser:browser:1.8.0'
    // Your other dependencies...
}

```

1. Add the matching `AndroidManifest.xml` activity registration:

```
<activity android:name="com.discord.socialsdk.AuthenticationActivity"
android:exported="true">
  <intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="discord-YOUR_APP_ID" />
  </intent-filter>
</activity>

```

 Replace `YOUR_APP_ID` with the real Discord application ID, such as `discord-1234567890123456789`.

---

### Understanding PKCE for Mobile

### What is PKCE?

 PKCE (Proof Key for Code Exchange, pronounced "pixie") is an OAuth2 security extension that helps prevent authorization code interception. It requires the client and server to prove they are participating in the same authentication exchange.

### Why is PKCE Required on Mobile?

 **Critical Requirement**: Every mobile application that uses custom URL schemes (deep links) must use PKCE, whether it uses a public client or a confidential client with server-side token exchange. Custom URL schemes do not provide the same security guarantees as HTTPS redirects, so PKCE is needed to reduce interception risk.

### How PKCE Works

1. **Client generates a code verifier**: A cryptographically random string
2. **Client creates a code challenge**: A hashed form of the verifier
3. **Client sends code challenge** with the authorization request
4. **Server stores the code challenge** with the authorization code
5. **Client provides the code verifier** during the authorization-code-to-token exchange
6. **Server verifies** that the verifier matches the stored challenge
 The Discord Social SDK generates the code verifier and challenge for you through [`Client::CreateAuthorizationCodeVerifier`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#aba6adc1f978e7bf4c5433c560e1ad704).

---

### Authentication Flow for Public Clients

 This path requires **Public Client** to be enabled for your app, which most games should avoid shipping with. [Learn more](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes#oauth2-client-types) If the app has no backend server and `Public Client` is enabled in the Discord Developer Portal, the SDK can handle the full authentication sequence, including PKCE.

### Step 1: Create Code Verifier and Challenge

 Call [`Client::CreateAuthorizationCodeVerifier`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#aba6adc1f978e7bf4c5433c560e1ad704) to create the PKCE values:

```
// Generate code verifier and challenge
auto codeVerifier = client->CreateAuthorizationCodeVerifier();

```

### Step 2: Request Authorization

 Call [`Client::Authorize`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ace94a58e27545a933d79db32b387a468) with the code challenge:

```
discordpp::AuthorizationArgs args{};
args.SetClientId(YOUR_DISCORD_APPLICATION_ID);
args.SetScopes(discordpp::Client::GetDefaultPresenceScopes());
args.SetCodeChallenge(codeVerifier.Challenge());

client->Authorize(args, [client, codeVerifier](
    discordpp::ClientResult result,
    std::string code,
    std::string redirectUri) {
  if (!result.Successful()) {
    std::cerr << "❌ Authorization Error: " << result.Error() << std::endl;
  } else {
    std::cout << "✅ Authorization successful! Exchanging code for token...\n";
    // Proceed to Step 3
  }
});

```

### Step 3: User Approval

 After [`Client::Authorize`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ace94a58e27545a933d79db32b387a468) runs, the SDK will:
- Open the Discord mobile app through a deep link if it is installed
- Open a browser if Discord is not installed
- Show the authorization screen to the user

### Step 4: Exchange Authorization Code for Token

 After the user approves and your app receives the authorization code, exchange that code for an access token with [`Client::GetToken`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#aaee636f91fb1ea3465157c20313b702c) and the code verifier:

```
client->GetToken(
    YOUR_DISCORD_APPLICATION_ID,
    code,
    codeVerifier.Verifier(),  // Critical: Pass the verifier
    redirectUri,
    [client](discordpp::ClientResult result,
        std::string accessToken,
        std::string refreshToken,
        discordpp::AuthorizationTokenType tokenType,
        int32_t expiresIn,
        std::string scope) {
      if (!result.Successful()) {
        std::cerr << "❌ Error getting token: " << result.Error() << std::endl;
        return;
      }

      std::cout << "🔓 Access token received! Establishing connection...\n";

      // Update token and connect
      client->UpdateToken(tokenType, accessToken, [client](discordpp::ClientResult result) {
        client->Connect();
      });
    });

```

---

### Authentication Flow for Confidential Clients

 If your application uses a backend server and a confidential client with client secret, mobile still requires PKCE, but your server performs the token exchange.

### Client-Side Implementation

#### Step 1: Create Code Verifier and Challenge

 Generate the PKCE values on the client:

```
// Generate code verifier and challenge
auto codeVerifier = client->CreateAuthorizationCodeVerifier();

```

#### Step 2: Request Authorization

 Include the code challenge in the authorization request:

```
discordpp::AuthorizationArgs args{};
args.SetClientId(YOUR_DISCORD_APPLICATION_ID);
args.SetScopes(discordpp::Client::GetDefaultPresenceScopes());
args.SetCodeChallenge(codeVerifier.Challenge());

client->Authorize(args, [client, codeVerifier](
    discordpp::ClientResult result,
    std::string code,
    std::string redirectUri) {
  if (!result.Successful()) {
    std::cerr << "❌ Authorization Error: " << result.Error() << std::endl;
  } else {
    std::cout << "✅ Authorization successful!\n";

    // Send BOTH the authorization code AND the code verifier to your server
    SendToServer(code, redirectUri, codeVerifier.Verifier());
  }
});

```

 **Critical**: Send the **code verifier** (not the challenge) to your server with the authorization code. The server needs the verifier to finish the token exchange.

### Server-Side Implementation

 Your server must send the `code_verifier` parameter when it exchanges the authorization code for an access token:

```
import requests

API_ENDPOINT = 'https://discord.com/api/v10'
CLIENT_ID = 'YOUR_CLIENT_ID'
CLIENT_SECRET = 'YOUR_CLIENT_SECRET'

def exchange_code(code, redirect_uri, code_verifier):
    """
    Exchange authorization code for access token with PKCE verification.

    IMPORTANT: The code_verifier parameter is REQUIRED for mobile apps,
    even when using confidential clients with client secrets.
    """
    data = {
        'grant_type': 'authorization_code',
        'code': code,
        'redirect_uri': redirect_uri,
        'code_verifier': code_verifier,  # Required for mobile deep links
    }
    headers = {'Content-Type': 'application/x-www-form-urlencoded'}

    r = requests.post(
        f'{API_ENDPOINT}/oauth2/token',
        data=data,
        headers=headers,
        auth=(CLIENT_ID, CLIENT_SECRET)
    )
    r.raise_for_status()
    return r.json()

```

#### Example Response

```
{
  "access_token": "<access token>",
  "token_type": "Bearer",
  "expires_in": 604800,
  "refresh_token": "<refresh token>",
  "scope": "sdk.social_layer"
}

```

#### Client-Side: Update Token

 After your server returns the access token, pass it into the SDK:

```
// Receive access token from your server
void OnTokenReceived(const std::string& accessToken) {
  client->UpdateToken(
      discordpp::AuthorizationTokenType::Bearer,
      accessToken,
      [client](discordpp::ClientResult result) {
    if (result.Successful()) {
      client->Connect();
    }
  });
}

```

---

### Working with Tokens

 Store the player's access and refresh tokens for later sessions. Because `access_token`s generally expire after 7 days, use the `refresh_token` to refresh the player's access token; the process is covered in [Refreshing Access Tokens](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-with-discord#refreshing-access-tokens).

---

### Troubleshooting

### Common Issues

#### iOS: Authorization Opens Browser Instead of Discord App

 **Problem**: Authorization opens a browser instead of the Discord mobile app. **Solutions**:
1. Confirm that `LSApplicationQueriesSchemes` includes `discord` in Info.plist
2. Confirm that the Discord mobile app is installed on the device
3. Verify that the redirect URI is configured as `discord-YOUR_APP_ID:/authorize/callback`

#### Android: Deep Link Not Working

 **Problem**: The app does not receive the callback after authorization. **Solutions**:
1. Confirm that AndroidManifest.xml contains the correct intent filter
2. Confirm that `android:scheme` matches the redirect URI: `discord-YOUR_APP_ID`
3. If you need to process multiple intents, confirm that the activity launch mode is `singleTask` or `singleTop`

#### Token Exchange Fails with “invalid_grant”

 **Problem**: Server-side token exchange returns `invalid_grant`. **Solutions**:
1. **Most common**: The token exchange request is missing the `code_verifier` parameter
2. Confirm that you are sending the verifier from `codeVerifier.Verifier()`, not the challenge
3. Confirm that the code has not expired; codes are short-lived
4. Confirm that the redirect URI exactly matches the one used in the authorization request

---

### Next Steps

 After mobile account linking is working, you can add more social features to your game.

### Design: Signing In

Design guidance for account linking and user authentication

### Creating a Unified Friends List

Merge Discord friends and in-game friends into one manageable list.

### Setting Rich Presence

Show game status and details to Discord friends. Need help? Join the [Discord Developers Server](https://discord.gg/discord-developers) and ask questions in `#social-sdk-dev-help` for community support. To report a Social SDK bug, use [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report)

---

### Change Log

| Date | Changes |
| --- | --- |
| January 26, 2026 | Initial release |
