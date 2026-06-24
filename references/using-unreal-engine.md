# Using Unreal Engine (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

### Overview

 This walkthrough shows how to add the Discord Social SDK to an Unreal Engine project. After completing it, the project can:
- Authenticate users with Discord
- Configure logging and status checks
- Initialize the SDK and connect it
- Fetch the player's Discord friend count
 The first Unreal plugin release is named `DiscordPartnerSDK`; a later release will rename it to `DiscordSocialSDK`.

### Prerequisites

 This guide builds a small Unreal console application. Confirm these prerequisites first:
- Windows
- Visual Studio 2022
- Unreal Engine 5.5
 The next sections cover each step in order.

---

### Step 1: Create a Discord Developer Team

 Start by creating a developer team in the Discord Developer Portal. That team owns the Discord applications and SDK integrations for the project. If a team already exists, skip this setup step.
1. Create the developer team in the [Discord Developer Portal](https://discord.com/developers/teams).
 After the team exists, invite teammates so they can help with the integration.

---

### Step 2: Create a Discord Application

1. Create an application in the [Discord Developer Portal](https://discord.com/developers/applications), then assign it to the team.
2. In the [OAuth2 tab](https://discord.com/developers/applications/select/oauth2), add this redirect URL:
  - Desktop apps can use `http://127.0.0.1/callback` ; you can change it later.
  - For advanced redirect URI setup, see [`discordpp::Client::Authorize`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ace94a58e27545a933d79db32b387a468).

3. Turn on `Public Client` in the [OAuth2 tab](https://discord.com/developers/applications/select/oauth2).
 This walkthrough enables **Public Client** so the SDK can be tried quickly. Most shipped games should not use public-client mode without review. Have the team evaluate this setting before release. [Learn more about public clients](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes#oauth2-client-types).

---

### Step 3: Enable Discord Social SDK for Your App

 After the Discord application exists, enable Discord Social SDK access for it.
1. In the [Discord Developer Portal](https://discord.com/developers/applications), choose the application created in Step 2.
2. In the app sidebar, under `Discord Social SDK`, open [Getting Started](https://discord.com/developers/applications/select/social-sdk/getting-started).
3. Complete the form with information about the game.
4. Click `Submit` to enable the Social SDK for the application.
5. After enablement, Social SDK binaries are available from [Downloads](https://discord.com/developers/applications/select/social-sdk/downloads).

---

### Step 4: Download the Social SDK for Unreal Engine

1. In the sidebar's Discord Social SDK section, open [Downloads](https://discord.com/developers/applications/select/social-sdk/downloads).
2. Choose the newest version from the dropdown and download the Unreal Engine SDK package.
 The page also offers an Unreal Engine sample project. This guide does not cover that sample, but it can be explored after the walkthrough.

---

### Step 5: Project Setup

### Create a new Unreal Engine project

 Create a Third Person Unreal project using C++. Name it `DiscordSocialUnreal`, because the samples below assume that project name. The code samples also refer to `DiscordSocialUnreal`. If you choose another name, update the code accordingly. Wait for shader compilation to finish; it can take some time.

### Add the Discord SDK to the project

 Find the Unreal project in Windows file explorer. At the project root, create `Plugins`, giving `DiscordSocialUnreal/Plugins`. Extract the Discord SDK archive into that Plugins directory so it contains `DiscordPartnerSDK` at `DiscordSocialUnreal/Plugins/DiscordPartnerSDK`. Verify the `DiscordPartnerSDK` folder name because future versions may change it, and the code below must use the matching name. Reload Unreal Editor and open the project; on launch it should ask whether to recompile the `DiscordPartnerSDK` plugin. From Unreal Editor, open Visual Studio through `Tools -> Open Visual Studio`. In `/Source/DiscordSocialUnreal/`, open `DiscordSocialUnreal.Build.cs` and add this line after the `PublicDependencyModuleNames` array:

```
PrivateDependencyModuleNames.AddRange(new string[] { "DiscordPartnerSDK" });

```

 Return to Unreal Editor and use `Tools -> Refresh Visual Studio Project` to refresh the Visual Studio project. Compile in Unreal Editor with `Ctrl-Alt-F11`. A successful compile shows a "Live coding succeeded" modal.

---

### Step 6: Initialize the SDK

 Next, add callbacks for SDK logging, SDK status, and OAuth2 authentication so Discord account linking works. The flow will:
1. Open Discord login in the Discord app or a browser
2. Get an authorization code
3. Trade that code for an access token
4. Connect to Discord
 **Choosing your OAuth2 scopes:** This walkthrough uses `UDiscordClient::GetDefaultPresenceScopes()`, which asks for `openid` and `sdk.social_layer_presence`. Those scopes support core behavior such as account linking, friends list access, and rich presence. If the game also needs lobbies, voice chat, or direct messages, switch to `UDiscordClient::GetDefaultCommunicationScopes()`. The [OAuth2 Scopes guide](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes) explains the full scope set. Open `DiscordSocialUnrealCharacter.h` in Visual Studio to begin integrating the Social SDK.

### DiscordLocalPlayerSubsystem

 Unreal Engine integration needs `DiscordLocalPlayerSubsystem`, which owns the client lifecycle and runs the [`RunCallbacks`](https://discord.com/developers/docs/social-sdk/namespacediscordpp.html#ab5dd8cf274f581ee1885de5816be3c29) event loop. Add this code to `DiscordSocialUnrealCharacter.h`:

```
#include "DiscordLocalPlayerSubsystem.h"

```

 Add these functions to the public section of the DiscordSocialUnrealCharacter class:

##### **DiscordSocialUnrealCharacter.h**

```
UFUNCTION(Exec)
void DiscordConnect();

UFUNCTION()
void OnStatusChanged(EDiscordClientStatus Status, EDiscordClientError Error, int32 ErrorDetail);

UPROPERTY()
UDiscordAuthorizationCodeVerifier* CodeVerifier;

UPROPERTY()
UDiscordLocalPlayerSubsystem* Discord;

```

 Add these functions to the protected section of the DiscordSocialUnrealCharacter class:

##### **DiscordSocialUnrealCharacter.h**

```
void BeginPlay();
void OnLogMessage(FString Message, EDiscordLoggingSeverity Severity);
void OnAuthorizeCompleted(UDiscordClientResult* Result, FString Code, FString RedirectUri);
void OnTokenExchange(UDiscordClientResult* Result, FString AccessToken, FString RefreshToken, EDiscordAuthorizationTokenType TokenType, int32 ExpiresIn, FString Scope);
void OnTokenUpdated(UDiscordClientResult* Result);

```

 Implement the functions in `DiscordSocialUnrealCharacter.cpp`. Add this code to `DiscordSocialUnrealCharacter.cpp`:

##### **DiscordSocialUnrealCharacter.cpp**

```
#define APPLICATION_ID 1111111111111111111

void ADiscordSocialUnrealCharacter::BeginPlay() {
  auto PlayerController = Cast<APlayerController>(GetController());
  auto LocalPlayer = PlayerController->GetLocalPlayer();
  Discord = ULocalPlayer::GetSubsystem<UDiscordLocalPlayerSubsystem>(LocalPlayer);
  auto LogCallback = FDiscordClientLogCallback::CreateUObject(this, &ADiscordSocialUnrealCharacter::OnLogMessage);
  FScriptDelegate StatusChanged;
  StatusChanged.BindUFunction(this, "OnStatusChanged");
  Discord->Client->AddLogCallback(LogCallback, EDiscordLoggingSeverity::Info);
  Discord->OnStatusChanged.Add(StatusChanged);
}

void ADiscordSocialUnrealCharacter::DiscordConnect() {
  CodeVerifier = Discord->Client->CreateAuthorizationCodeVerifier();
  auto AuthArgs = NewObject<UDiscordAuthorizationArgs>();
  AuthArgs->Init();
  AuthArgs->SetClientId(APPLICATION_ID);
  AuthArgs->SetScopes(UDiscordClient::GetDefaultPresenceScopes());
  AuthArgs->SetCodeChallenge(CodeVerifier->Challenge());
  Discord->Client->Authorize(AuthArgs, FDiscordClientAuthorizationCallback::CreateUObject(this, &ADiscordSocialUnrealCharacter::OnAuthorizeCompleted));
}

void ADiscordSocialUnrealCharacter::OnLogMessage(FString Message, EDiscordLoggingSeverity Severity) {
  UE_LOG(LogTemplateCharacter, Log, TEXT("[%s] %s"), *UEnum::GetValueAsString(Severity), *Message);
}

void ADiscordSocialUnrealCharacter::OnStatusChanged(EDiscordClientStatus Status, EDiscordClientError Error, int32 ErrorDetail) {
  UE_LOG(LogTemplateCharacter, Log, TEXT("Connection status: %s"), *UEnum::GetValueAsString(Status));
  if (Status == EDiscordClientStatus::Ready) {
    UE_LOG(LogTemplateCharacter, Log, TEXT("Connected to Discord! Ready to go! You can now start using Discord features."));
  }
  else if (Error != EDiscordClientError::None) {
    UE_LOG(LogTemplateCharacter, Log, TEXT("Connection error: %s (Detail: %d)"), *UEnum::GetValueAsString(Error), ErrorDetail);
  }
}

void ADiscordSocialUnrealCharacter::OnAuthorizeCompleted(UDiscordClientResult* Result, FString Code, FString RedirectUri) {
  if (!Result->Successful()) {
    UE_LOG( LogTemplateCharacter, Error, TEXT("Discord authorization failed: %s"), *Result->Error());
    return;
  }

  UE_LOG(LogTemplateCharacter, Log, TEXT("Authorization successful! Getting access token..."));
  Discord->Client->GetToken(APPLICATION_ID, Code, CodeVerifier->Verifier(), RedirectUri, FDiscordClientTokenExchangeCallback::CreateUObject(this, &ADiscordSocialUnrealCharacter::OnTokenExchange));
}

void ADiscordSocialUnrealCharacter::OnTokenExchange(UDiscordClientResult* Result, FString AccessToken, FString RefreshToken, EDiscordAuthorizationTokenType TokenType, int32 ExpiresIn, FString Scope) {
  if (!Result->Successful()) {
    UE_LOG(LogTemplateCharacter, Error, TEXT("Discord token exchange failed: %s"), *Result->Error());
  }

  Discord->Client->UpdateToken(TokenType, AccessToken, FDiscordClientUpdateTokenCallback::CreateUObject(this, &ADiscordSocialUnrealCharacter::OnTokenUpdated));
}

void ADiscordSocialUnrealCharacter::OnTokenUpdated(UDiscordClientResult* Result) {
  Discord->Client->Connect();
}

```

### What’s Happening Here?

1. Create an OAuth2 PKCE code verifier
2. Configure authorization arguments with the app ID and requested scopes
3. Begin authorization with [`Client::Authorize`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ace94a58e27545a933d79db32b387a468), which opens a browser
4. After authorization succeeds, exchange the code for an access token
5. Call [`Client::UpdateToken`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a606b32cef7796f7fb91c2497bc31afc4) so the SDK uses the access token for Discord API calls
6. After token update, call [`Client::Connect`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a873a844c7c4c72e9e693419bb3e290aa); the SDK then connects asynchronously

### Testing It Out

 Save the Visual Studio changes and switch back to Unreal Editor. Compile with `Ctrl-Alt-F11`, enter play mode, and use the console at the bottom of Unreal Editor. Run `DiscordConnect` to start account linking.

```
DiscordConnect

```

 If Discord is running locally, an authorization prompt should appear. Click "Authorize". When authorization succeeds, the Unreal Editor console should show the success log emitted by the sample code. Do not call most Discord features until the client status is `Ready`; use the status callback as the readiness signal.

### Troubleshooting

- Confirm that `APPLICATION_ID` is correct
- Make sure the redirect URL exists in the Discord Developer Portal
- Review the output log for specific errors
- Verify that the access token is valid
- Ensure you have internet connectivity

Mac libdiscord_partner_sdk.dylib Not Opened

On Mac, Apple may block "libdiscord_partner_sdk.dylib" with a Not Opened warning because it cannot verify the library. If that appears, press **Done** in the popup. Open **System Settings > Privacy & Security**, then scroll to **Security**. macOS should say "libdiscord_partner_sdk.dylib" was blocked to protect the Mac; choose **Open Anyway** and run again. On the next popup, select **Open Anyway** so the library can be used. After the client reaches ready state, implement Discord social features.

### Step 7: Access Discord Relationships

 Read the user's Discord relationships, or friends list, and show the count. This demonstrates how game code can access Discord data. Add this code inside the `Ready` section of `OnStatusChanged`:

```
if (Status == EDiscordClientStatus::Ready) {
  UE_LOG(LogTemplateCharacter, Log, TEXT("Connected to Discord! Ready to go! You can now start using Discord features."));
  UE_LOG(LogTemplateCharacter, Log, TEXT("👥 Friends Count: %d"), Discord->Client->GetRelationships().Num());
}

```

### What This Code Does

 When client status becomes `Ready,`, call [`Client::GetRelationships`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ad481849835cd570f0e03adafcf90125d) to receive all of the player's friends, then log the friend count to the output log.

### Testing It Out

 Save in Visual Studio and return to Unreal Editor. Compile with `Ctrl-Alt-F11`, enter play mode, open the console at the bottom of Unreal Editor, and run `DiscordConnect`.

### Example Output

```
👥 Friends Count: 42

```

### Troubleshooting

- Confirm that OAuth2 scopes include relationship access
- Confirm the client is connected and status is `Ready`
- Make sure the Discord account has friends
- Check the output log for errors
 Next, add Rich Presence so Discord can show the game's activity.

### Step 8: Set Rich Presence

 Use Rich Presence to display the game's activity on Discord. Other players can then see in their Discord friends list what someone is doing in the game. Add this code inside the `Ready` section of `OnStatusChanged`:

```
if (Status == EDiscordClientStatus::Ready) {
    UE_LOG(LogTemplateCharacter, Log, TEXT("Connected to Discord! Ready to go! You can now start using Discord features."));
    UE_LOG(LogTemplateCharacter, Log, TEXT("👥 Friends Count: %d"), Discord->Client->GetRelationships().Num());

    UE_LOG(LogTemplateCharacter, Log, TEXT("Connected to Discord! Ready to go! You can now start using Discord features."));
    UDiscordActivity* Activity = NewObject<UDiscordActivity>();
    Activity->Init();
    Activity->SetType(EDiscordActivityTypes::Playing);
    Activity->SetState("In competitive match");
    Activity->SetDetails("Rank: Diamond II");
	Discord->Client->UpdateRichPresence(Activity, FDiscordClientUpdateRichPresenceCallback::CreateUObject(this, &ADiscordSocialUnrealCharacter::OnRichPresenceUpdated));
}

```

 Then add this new method:

```
void ADiscordSocialUnrealCharacter::OnRichPresenceUpdated(UDiscordClientResult* Result) {
  if (Result->Successful()) {
    UE_LOG(LogTemplateCharacter, Log, TEXT("Rich Presence updated successfully!"));
  } else {
    UE_LOG(LogTemplateCharacter, Error, TEXT("Rich Presence update failed"));
  }
}

```

### What This Code Does

1. Build an [`Activity`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Activity.html#ae793d9adbe16fef402b859ba02bee682) object representing the player's activity
2. Sets basic information like:

- The activity type (Playing)
- Current state (“In Competitive Match”)
- Additional details (“Rank: Diamond II”)

1. Update rich presence on Discord

### Testing It Out

 Save in Visual Studio and return to Unreal Editor. Compile with `Ctrl-Alt-F11`, enter play mode, open the console at the bottom of Unreal Editor, and run `DiscordConnect.` After OAuth completes, the output log reports whether rich presence was set successfully; check the Discord profile to view it.

### Troubleshooting

 If the presence does not appear:
- Confirm the client is connected and status is `Ready`
- Inspect the output log for error messages
- Verify that the activity settings are valid
- Make sure the account is not invisible on Discord

---

The full DiscordSocialUnreal.Build.cs

```
// Copyright Epic Games, Inc. All Rights Reserved.

using UnrealBuildTool;

public class DiscordSocialUnreal : ModuleRules
{
	public DiscordSocialUnreal(ReadOnlyTargetRules Target) : base(Target)
	{
		PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;

		PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "EnhancedInput" });
		PrivateDependencyModuleNames.AddRange(new string[] { "DiscordPartnerSDK" });
    }
}

```

The full DiscordSocialUnrealCharacter.cpp
