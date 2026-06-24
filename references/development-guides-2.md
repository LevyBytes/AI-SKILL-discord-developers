# Local Development, Mobile

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: Run Your Application Locally; Supported Platforms: Web, iOS, Android.

## Run Your Application Locally

### Run Your Application Locally

 You can open an application from a localhost port or another unique URL. For web and desktop Discord, that URL must be reachable over HTTPS; mobile does not require HTTPS. With this approach, traffic from the app bypasses Discord's proxy, so requests must use complete URLs rather than ["mapped"](https://docs.discord.com/developers/activities/development-guides/local-development#url-mapping) URLs. To test a local web server, use [Launching your Application from the Discord Client](https://docs.discord.com/developers/activities/development-guides/local-development#launch-your-application-from-the-discord-client) and set Application URL Override to the web server address for the application.

### Running Your Application Through A Network Tunnel

 Local testing works, but Discord recommends testing through the Discord proxy so URL behavior matches production before release. A network tunnel such as [cloudflared](https://github.com/cloudflare/cloudflared#installing-cloudflared) can expose a local server while still going through the proxy. A common setup gives every developer a separate development-only application. To run a local environment through Discord's proxy, complete these steps:
1. In the Discord Developer portal, create a new application.
2. Enable Activities for your app.
3. Configure the application's [URL mapping](https://docs.discord.com/developers/activities/development-guides/local-development#url-mapping).
4. Start the local web server.
5. Install and start a tunneling tool such as [cloudflared](https://github.com/cloudflare/cloudflared#installing-cloudflared), pointing it at the local web server.
 The local web server may use HTTP while the tunnel presents HTTPS. With cloudflared, run the command below and substitute your server's port for `3000`.

```
cloudflared tunnel --url http://localhost:3000

```

 After the command starts, cloudflared prints the public tunnel URL.

```
Your quick Tunnel has been created! Visit it at (it may take some time to be reachable):
https://funky-jogging-bunny.trycloudflare.com

```

 In the Discord Developer Portal, change the Application URL mapping for the `/` url to `funky-jogging-bunny.trycloudflare.com`, matching the tunnel address, then save. (Figure: Configuring your URL Mapping) A domain you do not control, such as one from ngrok's free tier, can later be claimed by someone else and used to serve a harmful replacement site. Treat that as a risk, and when you must use a third-party temporary domain, clear or reset the URL mapping after the tunnel session. Then follow [Launching your Application from the Discord Client](https://docs.discord.com/developers/activities/development-guides/local-development#launch-your-application-from-the-discord-client). Leave Application URL Override disabled.

### Running Your Application In Production

 Production setup follows nearly the same sequence:
1. Create a new application if one does not already exist.
2. Enable Activities for your app.
3. Configure the application's [URL Mapping](https://docs.discord.com/developers/activities/development-guides/local-development#url-mapping). Set the URL for the application's html at the `/` route.
4. Use [Launching your Application from the Discord Client](https://docs.discord.com/developers/activities/development-guides/local-development#launch-your-application-from-the-discord-client). Keep Application URL Override turned off.
 The application is now using the configuration intended for the fully published version. (Figure: application-test-mode-prod)

---

### Launch your application from the Discord Client

 The Developer Activity Shelf lets you view and start activities that you own or that belong to teams you are on. An activity appears for a platform only when that platform, such as web, ios, or android, is enabled in [Settings/Supported Platforms](https://discord.com/developers/applications/select/embedded/settings) in the developer portal. To view the app from inside Discord in the Activity Shelf:

#### Web

1. Open ⚙️ User Settings > App Settings > Advanced, then enable `Developer Mode`.
2. Exit settings, then join a voice channel.
3. Open the Activity shelf by selecting the Rocket Button from the RTC Panel or Center Control Tray. The shelf should list the applications you can access in the developer portal. Note: only applications marked as "Embedded" appear in the shelf.
4. Select an activity to start it.

#### Mobile

1. From User Profile, choose Appearance and switch `Developer Mode` to On.
2. Enter a voice channel
3. Select an activity to start it.

---

### URL Mapping

 Discord sandboxes Activities through a Discord proxy. The proxy helps conceal user IP addresses and application IP addresses, and it blocks URLs that are known to be malicious. Application owners can allow proxy access to outside endpoints by defining mappings. Because the application is sandboxed, direct requests to external URLs are blocked. For example, to reach `https://some-api.com`. from the application, create a URL mapping with `PREFIX` set to `/api` and `TARGET` set to `some-api.com`. Requests made to `/api` inside the application are then forwarded through Discord's proxy to `some-api.com`.

#### How to set a URL Mapping

 To create or edit URL mappings, open [Activities -> URL Mappings](https://discord.com/developers/applications/select/embedded/url-mappings), then enter the needed prefix and target for each mapping. (Figure: Configuring your URL Mapping)

#### Prefix/Target formatting rules

- URL mappings support protocols such as https, wss, ftp, and others, so the URL target must omit the protocol. For a URL target, use `your-url.com` instead of `https://your-url.com`, and omit `https://` .
- Parameter matching can help route external domain URLs. For an external URL set with multiple subdomains, such as `foo.google.com` and `bar.google.com`, you could define this mapping:
| PREFIX | TARGET |
| --- | --- |
| `/google/{subdomain}` | `{subdomain}.google.com` |

- Targets must resolve to directories. A target that points at a file, such as `example.com/index.html`, is not supported and can behave unexpectedly.
- URL globbing means mappings that share the same starting path must be ordered with the shortest prefix last. For example, if both `/foo` and `/foo/bar` exist, place `/foo/bar` before `/foo`; otherwise `/foo` will catch the traffic and `/foo/bar` will not be reached.

| ✅ DO | ❌ DON’T |
| --- | --- |
| Requests mapped correctly | Requests for /foo/bar are routed incorrectly to `foo.com`. |
| (Figure: url-mapping-do.png) | (Figure: url-mapping-dont.png) |

#### Exceptions

 The sandbox described above is implemented with a [Content Security Policy (CSP)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP). Discord defines several CSP exceptions, so application clients can request those URLs directly without using proxy mappings. Notable exceptions include:
- `https://discord.com/api/`
- `https://canary.discord.com/api/`
- `https://ptb.discord.com/api/`
- `https://cdn.discordapp.com/attachments/`
- `https://cdn.discordapp.com/avatars/`
- `https://cdn.discordapp.com/icons/`
- `https://media.discordapp.net/attachments/`
- `https://media.discordapp.net/avatars/`
- `https://media.discordapp.net/icons/`

---

### Logging

 By default, the SDK forwards console `log`, `warn`, `error`, `info`, and `debug` events emitted by the app to the Discord application.

#### Viewing Logs on Desktop

 On desktop, logs can be inspected in the console tab of browser Developer Tools. For more details, see [Troubleshooting Console Log Errors](https://support.discord.com/hc/en-us/articles/115001239472-Troubleshooting-Console-Log-Errors). The Public Test Build (PTB) Discord client can also inspect logs through `View -> Developer -> Toggle Developer Tools`; download it from [https://discord.com/downloads](https://discord.com/downloads).

#### Viewing Logs on Mobile

 On mobile, logs are available from the `Debug Logs` item in User Settings. That item appears only after `Developer Mode` is enabled.
1. Use the bottom navigation to tap your avatar, then tap the gear icon to open `User Settings`.
2. Tap `Appearance`.
3. Turn the `Developer Mode` toggle ON.
4. `Debug Logs` appears in the `DEV ONLY` section.

#### Filtering for Application Logs

 In Debug Logs, search for application log entries with these keywords:
- `RpcApplicationLogger`
- Your Application ID
 Log entries use this format: `[RpcApplicationLogger] <application-id> - message`. The beginning of Debug Logs contains Discord application startup data, not application logs. Scroll farther down to find the logs from your application. (Figure: debug-logs-filtering)

#### Sharing Application Logs from Mobile

 When `Developer Mode` is active, mobile application logs can be shared from a Voice Channel.
1. In the voice channel, swipe up from the bottom to expand voice controls, then tap `Share Application Logs`.
2. A native share sheet opens, letting you save the logs to a file or send them in a message.

#### Disabling Logging

 To stop logs from being forwarded to the browser, disable logging with the optional configuration object.

```
import {DiscordSDK} from '@discord/embedded-app-sdk';
const discordSdk = new DiscordSDK(clientId, {
  disableConsoleLogOverride: true,
});

```

#### Forwarding Log Messages

 Use the SDK command `captureLog` to send selected log messages, as in the following example.

```
import {DiscordSDK} from '@discord/embedded-app-sdk';
const discordSdk = new DiscordSDK(clientId);
await discordSdk.ready();
discordSdk.commands.captureLog({
  level: 'log',
  message: 'This is my log message!',
});

```

## Supported Platforms: Web, iOS, Android

### Supported Platforms: Web, iOS, Android

 Activities launch on web/desktop by default. To change support for Web/iOS/Android:
- Open the [developer portal](https://discord.com/developers/applications).
- Select your application
- In the developer portal sidebar, choose [Activities -> Settings](https://discord.com/developers/applications/select/embedded/settings).
- Select the relevant platform checkboxes in the developer portal, then save.
 (Figure: supported-platforms)

---

### Mobile Safe Areas

 For example, safe area insets can be defined in CSS like this:

```
:root {
  --sait: var(--discord-safe-area-inset-top, env(safe-area-inset-top));
  --saib: var(--discord-safe-area-inset-bottom, env(safe-area-inset-bottom));
  --sail: var(--discord-safe-area-inset-left, env(safe-area-inset-left));
  --sair: var(--discord-safe-area-inset-right, env(safe-area-inset-right));
}

```

 This uses the `--discord-safe-area-inset-*` variable first and falls back to env values for iOS and for local development outside Discord. Reference the values this way:

```
body {
  padding-left: var(--sail);
  padding-right: var(--sair);
  padding-top: var(--sait);
  padding-bottom: var(--saib);
}

```

---

### Mobile Thermal States

 To improve the user experience, applications may need to react to thermal state changes using guidance from [thermal states surfaced by mobile devices](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/power_efficiency_guidelines_osx/RespondToThermalStateChanges.html). Discord's Embedded App SDK provides a shared layer over [Apple's thermal state APIs](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/power_efficiency_guidelines_osx/RespondToThermalStateChanges.html) and [Android's thermal state APIs](https://source.android.com/docs/core/power/thermal-mitigation#thermal-api). The mapping below shows how Discord's abstraction corresponds to Apple and Android thermal states.

```
enum ThermalState {
  NOMINAL = 0, // maps to "nominal" on iOS and "none" on Android
  FAIR = 1, // maps to "fair" on iOS and "light" / "moderate" on Android
  SERIOUS = 2, // maps to "serious" on iOS and "severe" on Android
  CRITICAL = 3, // maps to "critical" on iOS and "critical" / "emergency" / "shutdown" on Android
}

```

 Developers can subscribe to thermal state updates through the Embedded App SDK.

```
const handleThermalStateUpdate = (update: {thermal_state: number}) => {
  switch (thermalState) {
      case Common.ThermalStateTypeObject.NOMINAL:
        ...
      case Common.ThermalStateTypeObject.FAIR:
        ...
      case Common.ThermalStateTypeObject.SERIOUS:
        ...
      case Common.ThermalStateTypeObject.CRITICAL:
        ...
      default:
        ...
    }
}

discordSdk.subscribe('THERMAL_STATE_UPDATE', handleThermalStateUpdate);

```

 When an event subscription is created, Discord publishes the current thermal state and then publishes later changes as they occur. Android thermal state updates are available only on Android 10 and newer.
