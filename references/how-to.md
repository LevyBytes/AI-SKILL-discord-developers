# Debug Log, Handle Rate Limits, Handle Special Characters Display Names

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: Debug & Log; Overview; Use a font family that supports Unicode.

## Debug & Log

These how-to pages give practical patterns for adding Discord Social SDK functionality to a game.

### Debug & Log

Use the SDK debugging and logging features when investigating integration problems.

### Use with Discord APIs

Call Discord HTTP APIs from inside your game.

### Integrate Moderation

Set up and operate content moderation for games that use the Discord Social SDK.

### Market Your Integration

Recommendations for telling players about a Discord Social SDK integration.

### Handle Special Characters in Display Names

Account for Unicode in Discord Display Names when rendering game chat or friends lists.

### Voice Muting Based on Player Blocks

Mute lobby voice according to Discord block state or signals from another platform.

### Handle Rate Limits

Use ClientResult to recognize rate-limit failures and respond correctly.

---

### Change Log

| Date | Changes |
| --- | --- |
| July 23, 2025 | migrated several docs |
| June 17, 2025 | added How To category |

## Overview

### Overview

 Use this guide to set up debugging symbols and logging so Discord Social SDK integration work is easier to diagnose.

### Debugging

 Debugging symbols are available from [https://storage.googleapis.com/discord-public-symbols](https://storage.googleapis.com/discord-public-symbols). In Visual Studio, add that address under `Tools > Options > Debugging > Symbols` as a pdb location. The link is not browsable as a file listing, but individual symbol files are served beneath the domain and Visual Studio can use it as a symbol server.

### Logging

 Read SDK log output through [`Client::AddLogCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#af78996cff24a40f5dc7066beed16692c). If you configure [`Client::SetLogDir`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a23bd5802dfa3072201ea864ee839c001), the SDK writes logs into that directory. Production builds should generally use `ERROR (4)` or `WARN (3)`. During development, `INFO (2)` and `VERBOSE (1)` provide more detailed diagnostics.

```
client->AddLogCallback([](auto message, auto severity) {
  std::cout << "[" << EnumToString(severity) << "] " << message << std::endl;
}, discordpp::LoggingSeverity::Info);

```

### Voice Logging

 Voice has a separate log stream for the voice subsystem and the WebRTC layer. You can capture it in two ways:
- [`Client::SetVoiceLogDir`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a48c6b7e8bbc2b632a935acafc6a5f7a7) - stores voice logs on disk under the directory you provide.
- [`Client::AddVoiceLogCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ac4f9b459f853ef7270d1bfc12509ede5) - sends voice log entries to an application callback.
 You may use both APIs together. Call [`Client::SetVoiceLogDir`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a48c6b7e8bbc2b632a935acafc6a5f7a7) immediately after constructing Client and before anything that starts the voice subsystem, including [`Client::AddVoiceLogCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ac4f9b459f853ef7270d1bfc12509ede5). After voice initialization, setting the directory will not change logging behavior.

```
// Must be called right after constructing the client, before anything else
client->SetVoiceLogDir("/path/to/log/dir", discordpp::LoggingSeverity::Info);

// Can be called any time after construction
client->AddVoiceLogCallback([](auto message, auto severity) {
  std::cout << "[Voice][" << EnumToString(severity) << "] " << message << std::endl;
}, discordpp::LoggingSeverity::Info);

```

 Voice and WebRTC logs are produced separately from the primary SDK logs exposed through [`Client::AddLogCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#af78996cff24a40f5dc7066beed16692c). When troubleshooting voice or call-quality behavior, turn on voice logging as well as the main SDK callback.

### Audio Logging

 For acoustic echo cancellation (AEC) investigations, call [`Client::SetAecDump`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a3a05b2cafaa546d915a5249c63f4059f). It records diagnostic input and output waveform data, which helps analyze echo, feedback, and other audio-processing issues. [`Client::SetAecDump`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a3a05b2cafaa546d915a5249c63f4059f) toggles AEC diagnostic capture. When enabled, those waveform files are written to the same log directory configured with [`Client::SetLogDir`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a23bd5802dfa3072201ea864ee839c001).

```
// Enable AEC diagnostic recording
client->SetAecDump(true);

// ... perform voice chat operations to reproduce the issue ...

// Disable AEC diagnostic recording when done
client->SetAecDump(false);

```

 **When to use:**
- Players report echo or feedback in voice chat
- Audio quality problems appear connected to echo cancellation
- You are evaluating AEC behavior across different environments
- Debugging audio processing pipeline issues
 AEC dumps store raw waveform data and can grow quickly. Turn the capture off after you have enough material to inspect.

### Client Tools

 The Discord client includes developer tooling for testing Discord Social SDK features and diagnosing issues. Open `Settings > Advanced`, enable Application Test Mode, enter your Application ID for this guide, leave the other modal options alone, and click Activate. After activation, a wrench icon appears in the upper-right corner of the client; click it to open the tools.

### Account Linking

 This tab reports the state of each on-platform account-linking flow and gives you a quick way to start linking without hunting through client entry points. For flow details, see the [Account Linking from Discord guide](https://docs.discord.com/developers/discord-social-sdk/development-guides/account-linking-from-discord).

---

### Next Steps

### Getting Started Guide

Start here for initial Discord Social SDK setup.

### Core Concepts

Review the main concepts behind the Discord Social SDK.

### Account Linking with Discord

Use Discord account linking for users. Need help? Join the [Discord Developers Server](https://discord.gg/discord-developers) and ask in `#social-sdk-dev-help` for community support. Report Social SDK bugs at [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report).

---

### Change Log

| Date | Changes |
| --- | --- |
| March 17, 2025 | initial release |
| November 17, 2025 | add client tools |
| May 4, 2026 | add voice logging section |

## Overview

### Overview

 This guide shows how to recognize and handle rate-limit failures from Discord Social SDK requests. When the SDK rate limits an operation, the callback gets a failed [`ClientResult`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1ClientResult.html#a685015ca8d29a50d47fd1ed5d469ac2e), [`ClientResult::Retryable`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1ClientResult.html#a0d220638f4a36c0b8b731f601e0ac02d) is `true`, and [`ClientResult::RetryAfter`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1ClientResult.html#ac739adca52b90d6b4cbed5753c30fd65) tells you how many seconds to wait.

---

### Checking the Result

 Callbacks for SDK operations that perform network requests receive a [`ClientResult`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1ClientResult.html#a685015ca8d29a50d47fd1ed5d469ac2e). Check [`ClientResult::Successful`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1ClientResult.html#aef3b1aca3cd156daf488ca13ae87313b) before anything else. If the call failed, use [`ClientResult::Retryable`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1ClientResult.html#a0d220638f4a36c0b8b731f601e0ac02d) to decide whether another attempt is appropriate:

```
client->SendLobbyMessage(lobbyId, message, [](discordpp::ClientResult result, uint64_t messageId) {
  if (result.Successful()) {
    return;
  }

  if (result.Retryable()) {
    float delaySeconds = result.RetryAfter();
    // Schedule a retry after delaySeconds — see below
  } else {
    std::cerr << "Unrecoverable error: " << result.ToString() << "\n";
  }
});

```

 [`ClientResult::Retryable`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1ClientResult.html#a0d220638f4a36c0b8b731f601e0ac02d) is `true` for HTTP 429 rate limits and temporary network failures. It is `false` for validation problems, permission failures, and other errors that cannot be recovered by retrying.

---

### Implementing Retry Logic

 Treat [`ClientResult::RetryAfter`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1ClientResult.html#ac739adca52b90d6b4cbed5753c30fd65) as the earliest allowed retry time. Retrying sooner will keep hitting the limit, so wait for the full `RetryAfter` period. A basic helper can schedule the retry with your engine timer:

```
void SendMessageWithRetry(
    std::shared_ptr<discordpp::Client> client,
    uint64_t lobbyId,
    std::string message,
    int attemptsLeft)
{
  client->SendLobbyMessage(lobbyId, message,
    [client, lobbyId, message, attemptsLeft](discordpp::ClientResult result, uint64_t messageId) {
      if (result.Successful()) {
        std::cout << "Message sent\n";
        return;
      }

      if (result.Retryable() && attemptsLeft > 1) {
        float delay = result.RetryAfter();
        ScheduleCallback(delay, [client, lobbyId, message, attemptsLeft]() {
          SendMessageWithRetry(client, lobbyId, message, attemptsLeft - 1);
        });
      } else {
        std::cerr << "Failed after retries: " << result.ToString() << "\n";
      }
    });
}

```

 `ScheduleCallback` stands in for the delayed execution facility your engine uses, such as a task queue, coroutine sleep, or tick-based callback.

---

### Rate Limit Reference

 For production limit increases and communication-feature rate-limit details, see [Communication Features](https://docs.discord.com/developers/discord-social-sdk/core-concepts/communication-features). For the broader Discord rate-limit model, see [Discord's rate limiting documentation](https://docs.discord.com/developers/topics/rate-limits).

---

### Next Steps

### Managing Lobbies

Build and operate game lobbies that include text chat and voice chat.

### Sending Direct Messages

Let the game send Discord users direct messages.

### Debug & Log

Use debugging and logging features while diagnosing issues. Need help? Join the [Discord Developers Server](https://discord.gg/discord-developers) and ask in `#social-sdk-dev-help` for community support. Report Social SDK bugs at [https://dis.gd/social-sdk-bug-report](https://dis.gd/social-sdk-bug-report).

---

### Change Log

| Date | Changes |
| --- | --- |
| April 27, 2026 | Initial release |

## Use a font family that supports Unicode

The Social SDK [recommends](https://discord.com/developers/docs/discord-social-sdk/development-guides/creating-a-unified-friends-list#step-1-fetch-relationships) showing an account's [Display Name](https://discord.com/developers/docs/slayer-sdk/classdiscordpp_1_1UserHandle.html#af6447fa2011bfa4fcd7e55bc56847f5c) in the Unified Friends List. Display Names may contain many Unicode characters, and a game font might not include every glyph. You can handle that in several ways:

### Use a font family that supports Unicode

 Fonts such as [Noto](https://notofonts.github.io/) target broad Unicode coverage. If the selected game font cannot draw a Display Name, render with a fallback font; in severe cases, choose a broadly covered font for the Friends List. **Tradeoffs:**
- No single family is guaranteed to cover every character Discord allows.
- A Unicode-capable font may conflict with the visual style of your game.
- Engine handling can be incomplete, particularly for UTF surrogate-pair characters.

### Map the Unicode characters to ASCII

 A tool such as [Unidecode](https://metacpan.org/pod/Text::Unidecode), available in many languages, can transliterate many Unicode characters to ASCII. When a Display Name contains unsupported glyphs, you can pass it through such a library before display. **Tradeoffs:**
- Transliteration can be wrong or surprising. A perfect Unicode-to-ASCII conversion does not exist for every case, so libraries may produce an inaccurate, inappropriate, or offensive display name.
- Coverage is not complete. Some Unicode characters have no ASCII equivalent; for example, consider what should happen to a Display Name like `ʕ•͡-•ʔ`.
- Many transliteration libraries assume English output.
- Users may object if their display names are altered without notice or consent.

### Use the Discord Username as a Fallback

 If a Display Name cannot be rendered at all, use the user's [`Username`](https://discord.com/developers/docs/slayer-sdk/classdiscordpp_1_1UserHandle.html#a0eda41fe18b50bce373fb5a1b88cb411) as a fallback. **Tradeoffs:**
- This can reduce immersion when a friend expected their in-game identity to use their Display Name.
- Some users treat Username as more private and may not expect it to appear to others, for example on a visible friends list during a stream.

---

### Change Log

| Date | Changes |
| --- | --- |
| June 17, 2025 | special characters how-to added |
