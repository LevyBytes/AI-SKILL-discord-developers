# Managing Voice Chat

> Discord Developers reference. Original prose; identifiers preserved verbatim.

This capability is available now, but it is rate limited. If your game needs higher limits, use [Communication Features: Applying for Increased Rate Limits for Production Releases](https://docs.discord.com/developers/discord-social-sdk/core-concepts/communication-features#applying-for-increased-rate-limits-for-production-releases).

### Overview

 Voice calls are one of the Discord Social SDK's main communication tools, letting players talk in real time while they are together in lobbies. This guide covers how to:
- Open voice calls in lobbies and enter existing calls
- Manage mute, deafen, and volume behavior
- Handle audio data through custom callbacks
- Integrate with external audio systems
- Inspect call presence and each participant's voice state

### Prerequisites

 Before initiating a voice call, complete the required setup. For this communication feature, include [`Client::GetDefaultCommunicationScopes`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a71499da752fbdc2d4326ae0fd36c0dd1) in your OAuth Scope configuration. Additional context is available in the [OAuth Scopes Core Concepts Guide](https://docs.discord.com/developers/discord-social-sdk/core-concepts/oauth2-scopes).

### 1. Lobby Management

 **Voice calls need a live lobby that contains participants.** Required setup:
1. **Create or join a lobby** with the Discord Social SDK
2. **Place players in the lobby** - voice is available only to members of that lobby
 Essential Reading: For the full flow for creating lobbies, entering them, and administering lobby membership, refer to the [Managing Lobbies Guide](https://docs.discord.com/developers/discord-social-sdk/development-guides/managing-lobbies).

### 2. Lobby Size Limitations

 Discord lobbies can technically contain as many as 1,000 members, but **voice calls should use far smaller groups**. For the best performance and player experience, **keep voice calls at 25 members or fewer**.

---

### Starting and Joining Voice Calls

 With the Discord Social SDK, starting and joining use the same API style whether the call or lobby already exists or needs to be created.

### Creating/Joining a Lobby and Starting/Joining a Call

 The following end-to-end sample joins a lobby and begins a voice call:

```
// First, create or join a lobby using a shared secret
const std::string lobbySecret = "my-game-lobby-secret";

client->CreateOrJoinLobby(lobbySecret, [client](const discordpp::ClientResult& result, uint64_t lobbyId) {
    if (result.Successful()) {
        std::cout << "🎮 Successfully joined lobby!" << std::endl;

        // Now start or join the voice call in this lobby
        // StartCall returns a Call object but has no callback
        const auto call = client->StartCall(lobbyId);

        // StartCall returns null if user is already in this voice channel
        if (call) {
          std::cout << "🎤 Voice call operation initiated..." << std::endl;
        } else {
          std::cout << "ℹ️ Already in this voice channel" << std::endl;
        }

    } else {
        std::cerr << "❌ Failed to join lobby: " << result.Error() << std::endl;
    }
});

```

### How It Works

 [`Client::CreateOrJoinLobby`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a8b4e195555ecaa89ccdfc0acd28d3512) and [`Client::StartCall`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#aef4f25d761fe198fbe9bc721fc24d83f) both cover the create and join cases without separate caller logic:
- [`Client::CreateOrJoinLobby`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a8b4e195555ecaa89ccdfc0acd28d3512): When the supplied secret matches an existing lobby, the caller joins it; otherwise, the SDK creates a lobby using that secret.
- [`Client::StartCall`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#aef4f25d761fe198fbe9bc721fc24d83f): When the lobby already has an active voice call, the caller enters it; otherwise, a new call is opened.
 Your code does not need a preflight test for lobby existence or call activity. The SDK resolves both cases, which keeps the implementation shorter and less error-prone.

### Controlling Voice Features

 The Discord Social SDK includes voice controls for a single call as well as settings that apply across every active call.

### Global Voice Controls

 The following methods apply voice settings across all active calls through the [`Client`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a91716140c699d8ef0bdf6bfd7ee0ae13) object:
- [`Client::SetSelfMuteAll`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a9c6ef96590533d103a866cb8a99d2669) - Turns off your microphone in every current call
- [`Client::SetSelfDeafAll`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a59be56ae5752e9f2f0f299bc552282b2) - Deafens you in all active calls
- [`Client::SetInputVolume`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ad4358f5baffd9a5f2a6fa74d62459313) - Changes microphone volume
- [`Client::SetOutputVolume`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a61a9321a79479c8b1be1559e2bbdd934) - Changes speaker volume

### Per-Call Voice Controls

 These methods adjust voice behavior for one call through the [`Call`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Call.html#a1cc8a7f73c15a960bc409d734b5edbd1) object:
- [`Call::SetSelfMute`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Call.html#afa35a5d6a4564df97452df58bb74f617) - Disables your mic for this call, preventing other participants from hearing you
- [`Call::SetSelfDeaf`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Call.html#a07d67c210f2a4655c6f1d2899c6d32d6) - Silences this call for you and also stops other participants from hearing you
- [`Call::SetParticipantVolume`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Call.html#ad974fadbe89c453e4d8a3f9824e21ceb) - Changes volume for one selected participant

### Voice Activity Detection

 Call [`Call::SetVADThreshold`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Call.html#a7c3fd83c5dfe37d796e30c5e28c93b6e) to tune the sensitivity of voice activity detection, controlling when speech is considered active.

```
// Per-call controls (assuming you have a Call object)
uint64_t lobbyId = 123456789;  // Your lobby ID
auto call = client->GetCall(lobbyId);
if (call) {
    call.SetSelfMute(true);           // Mute in this call only
    call.SetSelfDeaf(false);          // Unmute audio in this call
    call.SetParticipantVolume(userId, 150.0f);  // Increase participant volume
    call.SetVADThreshold(false, -30.0f);       // Set custom voice detection threshold
}

// Global controls
client->SetSelfMuteAll(true);          // Mute across all calls
client->SetInputVolume(75.0f);         // Set microphone to 75%
client->SetOutputVolume(120.0f);       // Increase speaker volume to 120%

```

### Detecting No Audio Input

 The SDK can alert you when the microphone is not receiving audio, such as when the selected device is wrong, the OS has muted input, or the hardware is failing. Your game can then show a hint that the mic appears silent instead of leaving the player without feedback.
- [`Client::SetNoAudioInputThreshold`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ab33f5d70461ee7590b6f3cfccaeb6df4) - dBFS cutoff used to classify input as silent. The range is `[-100.0, 100.0]`; the default `-100.0` disables detection. A value such as `-60.0` enables it.
- [`Client::SetNoAudioInputCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a479e60724bf6b0b39b555c1ff8489b9e) - Invoked with `bool inputDetected` when mic input crosses the threshold, in either direction from silent to active or active to silent.

```
// Enable detection at -60 dBFS
client->SetNoAudioInputThreshold(-60.0f);

client->SetNoAudioInputCallback([](bool inputDetected) {
    if (!inputDetected) {
        // Show a UI hint: "Your mic appears silent — check your device settings."
    } else {
        // Mic is receiving audio again; clear the hint.
    }
});

```

### Noise Suppression & Cancellation

 The SDK has two levels of microphone processing: WebRTC-based defaults that stay enabled by default, plus optional Krisp noise cancellation for stronger filtering. Krisp can remove noise at higher quality, but it adds libraries and model assets that increase install size. For size-sensitive builds such as mobile, see Excluding Krisp to Reduce Installation Size to ship with only WebRTC defaults.

### Default Audio Processing (WebRTC)

 Every build includes these three WebRTC audio processors, and they start enabled by default:
- [`Client::SetNoiseSuppression`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ae3f6e33b956964525adfa4536bd1fe73) - Reduces constant background sounds such as fans, keyboard noise, and room tone. Enabled by default.
- [`Client::SetEchoCancellation`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a1def244b7ecd388902ba5256ce506ca3) - Filters speaker echo captured by the microphone. Enabled by default.
- [`Client::SetAutomaticGainControl`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a818ae7f46b5bd3873dcd51dd3d9fa64d) - Automatically evens out mic level for clearer, steadier input. Enabled by default.

```
// Toggle individual WebRTC processors from a voice settings UI
client->SetNoiseSuppression(true);
client->SetEchoCancellation(true);
client->SetAutomaticGainControl(true);

```

### Advanced Noise Cancellation (Krisp)

 [`Client::SetNoiseCancellation`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a54aad09e8e06dc327695209b733d3f4c) turns on Krisp, which cancels a broader set of background sounds, such as typing, traffic, and barking, than WebRTC noise suppression. It is off by default. Krisp and WebRTC noise suppression cannot run together; enabling [`Client::SetNoiseCancellation`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a54aad09e8e06dc327695209b733d3f4c) also disables [`Client::SetNoiseSuppression`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ae3f6e33b956964525adfa4536bd1fe73), so you do not need a separate disable call.

```
// Enable Krisp noise cancellation
client->SetNoiseCancellation(true);

```

### Excluding Krisp to Reduce Installation Size

 Krisp is distributed as extra libraries and model files next to the core SDK, which increases the installed footprint. If reducing size matters, for example on mobile, omit Krisp and keep WebRTC noise suppression. To exclude Krisp, remove all `.kef` and `.kw` files, plus any file or directory whose name includes `krisp`, such as `discord_krisp.dll`, `libdiscord_krisp.dylib`, `discord_partner_sdk_krisp.aar`, and `discord_partner_sdk_krisp.xcframework`.

### Advanced Audio Processing

### Manipulating Voice Data with Callbacks

 For custom audio processing, call [`Client::StartCallWithAudioCallbacks`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#abcaa891769f9e912bfa0e06ff7221b05) to receive raw audio data. That path supports live sample changes and handoff to outside audio systems.

### In-Place Audio Modification

 To alter incoming samples directly, such as lowering volume in place:

```
const auto call = client->StartCallWithAudioCallbacks(
    lobbyId,
    [](uint64_t userId, int16_t *data, const size_t samplesPerChannel,
       int sampleRate, const size_t channels,
       bool &outShouldMuteData) {
      // Dampen volume of incoming audio by modifying data's samples
      // in-place
      for (int i = 0; i < samplesPerChannel * channels; i++) {
        data[i] *= 0.5; // Reduce volume by 50%
      }
    },
    [](int16_t *data, uint64_t samplesPerChannel, int32_t sampleRate,
       uint64_t channels) {});

```

### External Audio Pipeline Integration

 To send audio through an external processor such as [FMOD](https://www.fmod.com/) or [Wwise](https://www.audiokinetic.com/en/wwise/overview/):

```
const auto call = client->StartCallWithAudioCallbacks(lobbyId,
    [](uint64_t userId, int16_t* data, size_t samplesPerChannel,
       int sampleRate, size_t channels, bool& outShouldMuteData) {
        // Prevent Discord from playing the audio directly
        outShouldMuteData = true;

        const int totalNumSamples = samplesPerChannel * channels;
        // Send audio data to your external audio system
        SendAudioToExternalAudioSystem(data, totalNumSamples);
    },
    [](int16_t *data, uint64_t samplesPerChannel, int32_t sampleRate,
       uint64_t channels) {});

```

### Key Audio Processing Points

1. **Direct Manipulation**: You may edit the `data` argument in `Client::UserAudioReceivedCallback` in place to change incoming samples
2. **External Processing**: Set `outShouldMuteData = true` when Discord should not play the audio itself, letting your own audio pipeline handle playback
3. **No Encoding Required**: Voice encode and decode work is handled by the SDK; your callback receives raw audio samples

### Ending Voice Calls

 When a voice call should be ended, choose one of two approaches:

### End a Call For a Specific Lobby

```
uint64_t lobbyId = 123456789;  // Your lobby ID
client->EndCall(lobbyId, []() {
    std::cout << "🔇 Call ended successfully" << std::endl;
});

```

### End All Calls

```
client->EndCalls([]() {
    std::cout << "🔇 All calls ended successfully" << std::endl;
});

```

### Checking Lobby Voice Call Status

 Your game may inspect lobby voice status to show UI state, track activity, or tell players who is in voice. The Discord Social SDK exposes multiple ways to read active call details and participant states.

### Checking if a Call is Active

 Call `LobbyHandle::GetCallInfoHandle()` to find out whether the lobby currently has an active voice call:

```
// Check if there's an active call in the lobby
const auto callInfoHandle = lobby->GetCallInfoHandle();
if (callInfoHandle) {
  // There's an active call - you can join it or get participant
  // info
  const auto participants = callInfoHandle->GetParticipants();
  std::cout << "Active call with " << participants.size()
            << " participants" << std::endl;
} else {
  // No active call in this lobby
  std::cout << "No active voice call in this lobby" << std::endl;
}

```

### Checking Individual Participant Status

 Use `VoiceStateHandle` to read voice state for individual participants in a call:

```
// Get voice state information for participants
const auto callInfo = lobby->GetCallInfoHandle();
if (callInfo) {
  const auto participants = callInfo->GetParticipants();

  for (const auto &participantId : participants) {
    const auto voiceState = callInfo->GetVoiceStateHandle(participantId);
    if (voiceState) {
      const bool isMuted = voiceState->SelfMute();
      const bool isDeafened = voiceState->SelfDeaf();

      std::cout << "Participant " << participantId
                << " - Muted: " << (isMuted ? "Yes" : "No")
                << ", Deafened: " << (isDeafened ? "Yes" : "No")
                << std::endl;
    }
  }
}

```

### Voice State Information Available

 [`VoiceStateHandle`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1VoiceStateHandle.html#aad2d4454b6677d82721128b0cd98a2d8) exposes these important participant details:
- [`VoiceStateHandle::SelfMute`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1VoiceStateHandle.html#a5476a6e8d5e9092a153b4646371a9f3f): Returns `true` when the participant muted their own microphone, so others cannot hear them
- [`VoiceStateHandle::SelfDeaf`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1VoiceStateHandle.html#a9fd4ac5fb813b926d1336fc65b440f42): Returns `true` when the participant deafened themselves, preventing them from hearing others and preventing others from hearing them
 These values are useful for:
- Showing voice status indicators in the UI
- Building voice features or investigating audio problems

---

### Diagnosing Audio Issues

 For reports of echo, feedback, or other audio quality issues, use the SDK's diagnostics for voice and audio capture. See these Debug & Log guide sections:
- [Voice Logging](https://docs.discord.com/developers/discord-social-sdk/how-to/debug-log#voice-logging) - Capture logs from voice internals and the underlying WebRTC layer.
- [Audio Logging](https://docs.discord.com/developers/discord-social-sdk/how-to/debug-log#audio-logging) - Save input and output waveforms to disk for later analysis.

---

### Next Steps

### Integrate Moderation

Add and operate content moderation in a game that uses the Discord Social SDK.

### Use with Discord APIs

Call Discord's HTTP APIs from inside your game.

### Voice Muting Based on Player Blocks

Mute lobby voice participants according to player block relationships. Need help? Join the [Discord Developers Server](https://discord.gg/discord-developers) and ask in `#social-sdk-dev-help` for community support. If you find a Social SDK bug, report it at [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report).

---

### Change Log

| Date | Changes |
| --- | --- |
| May 13, 2026 | Added no-audio-input detection plus noise suppression and cancellation controls |
| June 30, 2025 | Add communications scope warning |
| June 19, 2025 | released guide |
| March 17, 2025 | initial release |
