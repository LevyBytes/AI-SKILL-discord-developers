# Change Log (part 5)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Starting today, app file upload limits are evaluated per attachment instead of per message. App uploads now match the behavior users get when they attach multiple files to one message.
- File size limits are now checked separately for each attachment
- Earlier behavior used the combined size of all attachments on a message
- App attachment handling now matches user attachment handling
Interaction payloads now include `attachment_size_limit`, which gives the maximum attachment size allowed. The value may rise above the default when the server's Boost tier or the invoking user's Nitro entitlement permits larger uploads. See [our documentation on file uploads](https://docs.discord.com/developers/reference#uploading-files). March 17, 2025 Discord Social SDK

### Introducing the Discord Social SDK

The Discord Social SDK can now add social systems to games, including friends lists, cross-platform messaging, voice, and related features for players with or without a Discord account. Discord Social SDK features are intended to improve connectivity and player engagement, including:
- Account Linking
- Provisional Accounts
- Rich Presence
- Deeplink Game Invites
Closed beta access is available for these in-game communication features:
- Cross-Platform Messaging
- Linked Channels
- Voice Chat
Developers can request expanded closed beta access for these features.

#### Discord Social SDK Developer Resources

Developer Portal resources are available for getting started with the Discord Social SDK:
- [Getting Started Guides](https://docs.discord.com/developers/discord-social-sdk/getting-started) cover C++, Unity and Unreal Engine.
- [Development Guides](https://docs.discord.com/developers/discord-social-sdk/development-guides) cover game social feature implementation.
- [Design Guidelines](https://docs.discord.com/developers/discord-social-sdk/design-guidelines) cover social feature design for games.
- [SDK Reference](http://discord.com/developers/docs/social-sdk/index.html) is available.
- Discord Social SDK binaries can be downloaded from the Developer Portal after the Discord Social SDK is enabled for the application.
Read the [Discord Social SDK Overview](https://docs.discord.com/developers/discord-social-sdk/overview) to learn about building with Discord Social SDK. December 16, 2024 HTTP API Interactions Breaking Changes

### Default File Upload Limit Change

On January 16, 2025, the default file upload limit changes from 25 MiB to 10 MiB. The 10 MiB limit already applies to users and bot users, and will extend to webhooks.
- The change takes effect on January 16, 2025.
- Webhooks and interaction responses will both use the 10 MiB limit.
See [our documentation on file uploads](https://docs.discord.com/developers/reference#uploading-files). December 12, 2024 Premium Apps

### Premium Apps: Multiple Subscription Tiers

Developers with monetization enabled can create and publish multiple subscription SKUs of the same type for one app. This supports different subscription tiers with separate benefits and prices, and users can move up or down between published subscription SKUs.

### What’s Changed

#### Developer Portal

- The `Monetization` tab now supports publishing multiple subscription SKUs of the same type for an app.

#### App’s Store Page

- When multiple subscription SKUs are published, users can upgrade or downgrade between the published subscription SKUs.

#### User App Subscription Settings

- When multiple subscription SKUs are published, user app subscription settings let users upgrade or downgrade between them.
- These settings are under `User Settings → Subscriptions → App Subscriptions`.

#### Subscription Object

- The [subscription object](https://docs.discord.com/developers/resources/subscription#subscription-object) response for `SUBSCRIPTION_UPDATE` events and API endpoints now includes `renewal_sku_ids`.
- `renewal_sku_ids` is a snowflake list identifying the SKU(s) the user will receive at renewal.

#### Updated Guide: Managing SKUs

- [Managing SKUs](https://docs.discord.com/developers/monetization/managing-skus#creating-a-sku) now covers creating and managing multiple subscription SKUs.

#### Updated Guide: Implementing App Subscriptions

- [Implementing App Subscriptions](https://docs.discord.com/developers/monetization/implementing-app-subscriptions#supporting-upgrades-and-downgrades) now covers upgrade and downgrade support across multiple subscription SKUs.
 November 05, 2024Premium Apps

### Entitlement Migration Completed

The [entitlement migration](https://docs.discord.com/developers/change-log#premium-apps-entitlement-migration-and-new-subscription-api), which began on **October 1, 2024**, was completed on **November 1, 2024**.

### What’s Changed

- Documentation now treats the new entitlement system as standard behavior.
- For subscription-related entitlements, `ENTITLEMENT_UPDATE` occurs only when the subscription ends.
- The [entitlement object](https://docs.discord.com/developers/resources/entitlement#entitlement-object) now sets `ends_at` when the subscription ends.
- To detect subscription cancellation, listen for `SUBSCRIPTION_UPDATE` events or call the [Subscription API](https://docs.discord.com/developers/resources/subscription) and read `status` plus `canceled_at`.
See the [migration guide](https://docs.discord.com/developers/change-log#updates-to-entitlement-migration-guide) for migration details. October 25, 2024 Events User Apps

### Webhook Events

After [configuring a webhook events URL](https://docs.discord.com/developers/events/webhook-events#configuring-a-webhook-events-url), apps can subscribe to a limited set of HTTP-based outgoing [webhook events](https://docs.discord.com/developers/events/webhook-events#event-types). The currently available events are 3: `APPLICATION_AUTHORIZED`, `ENTITLEMENT_CREATE`, and `QUEST_USER_ENROLLMENT`. Read the [webhook events](https://docs.discord.com/developers/events/webhook-events) docs for subscription and usage details. When building [user-installable apps](https://docs.discord.com/developers/resources/application#user-context), [Application Authorized](https://docs.discord.com/developers/events/webhook-events#application-authorized) is useful because [the Gateway](https://docs.discord.com/developers/events/gateway) does not emit it for installs to a user or server. For monetization, webhook events currently expose only `ENTITLEMENT_CREATE`; continue using the Gateway for [entitlement-related events](https://docs.discord.com/developers/events/gateway-events#entitlements) until more monetization events are added. October 07, 2024 Premium Apps

### Updates to Entitlement Migration Guide

The entitlement migration began on **October 1, 2024** and continues through 11:59PM PST on **November 1, 2024**. The earlier migration guide was revised with current developer impact details. Summary of changes:
- The migration runs through November 1, 2024 so October renewals can move into the new entitlement system on renewal.
- `ENTITLEMENT_UPDATE` events occur only when a subscription ends.
- In `ENTITLEMENT_UPDATE` events, `ends_at` gives the timestamp for **when the entitlement is no longer valid**.
- The [entitlement object](https://docs.discord.com/developers/resources/entitlement#entitlement-object) sets `ends_at` when the subscription ends.
- To detect cancellation timing, listen for `SUBSCRIPTION_UPDATE` events or query the [Subscription API](https://docs.discord.com/developers/resources/subscription).
Open the [updated migration guide](https://docs.discord.com/developers/change-log#premium-apps-entitlement-migration-and-new-subscription-api). The full diff is in [Entitlement Migration Guide Updates](https://github.com/discord/discord-api-docs/pull/7201). September 26, 2024 Activities Embedded App SDK Premium Apps

### Activities General Availability

After [the rollout of the App Launcher](https://discord.com/blog/discover-more-ways-to-play-with-apps-now-anywhere-on-discord), [Activities](https://docs.discord.com/developers/activities/overview) are generally available for developers. Along with API stability, apps with Activities can be [verified](https://support-dev.discord.com/hc/en-us/articles/23926564536471-How-Do-I-Get-My-App-Verified), made [discoverable](https://docs.discord.com/developers/discovery/enabling-discovery) in the App Directory, and set up to [implement monetization](https://docs.discord.com/developers/monetization/overview).

### Recent API Updates

The developer preview has been followed by several important API updates:
- Activities can enable and implement monetization features. [`getEntitlements`](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#getentitlements), [`getSkus`](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#getskus), and [`startPurchase`](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#startpurchase) are generally available in the Embedded App SDK.
- The new [Get Application Activity Instance](https://docs.discord.com/developers/resources/application#get-application-activity-instance) endpoint simplifies [managing Activity instances](https://docs.discord.com/developers/activities/development-guides/multiplayer-experience#activity-instance-management).
- Apps with Activities can create an [Entry Point command (type `4`)](https://docs.discord.com/developers/interactions/application-commands#entry-point-commands), the main App Launcher entry point for Activities. New apps that enable Activities receive a [default Entry Point command](https://docs.discord.com/developers/interactions/application-commands#default-entry-point-command). See the [original change log](https://docs.discord.com/developers/change-log#entry-point-commands) and [Entry Point command guide](https://docs.discord.com/developers/activities/development-guides/user-actions#setting-up-an-entry-point-command) for details.
- Interactions can launch Activities through the `LAUNCH_ACTIVITY` (type `12`) [interaction callback type](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-response-object-interaction-callback-type) for `APPLICATION_COMMAND`, `MESSAGE_COMPONENT`, and `MODAL_SUBMIT` [interaction types](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-type).
- Apps can be installed to users as well as servers. After [setting up your installation contexts](https://docs.discord.com/developers/resources/application#setting-supported-installation-contexts), request the `application.commands` scope during user authorization so the Activity is available across Discord servers, DMs, and Group DMs.
- August updates to the Content Security Policy (CSP) for Activities restrict requests to external resources while building Activities. See [the change log](https://docs.discord.com/developers/change-log#activities-proxy-csp-update) and [using external resources](https://docs.discord.com/developers/activities/development-guides/networking#using-external-resources).

### Documentation Updates

Documentation for Activities and the Embedded App SDK has been expanded to make builds easier:
- New [Monetization](https://docs.discord.com/developers/monetization/overview) SDK command documentation covers [`getEntitlements`](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#getentitlements), [`getSkus`](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#getskus), and [`startPurchase`](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#startpurchase).
- [Embedded App SDK Reference](https://docs.discord.com/developers/developer-tools/embedded-app-sdk) now includes signatures and arguments.
- Development guides for [Activity Instance Management](https://docs.discord.com/developers/activities/development-guides/multiplayer-experience#activity-instance-management) and [Activity Proxy Considerations](https://docs.discord.com/developers/activities/development-guides/networking#activity-proxy-considerations) were updated for external-resource usage.
- A new [In-App Purchases (IAP) for Activities](https://docs.discord.com/developers/monetization/implementing-iap-for-activities) guide is available.
- New [Verification and Discovery Surfaces](https://docs.discord.com/developers/discovery/overview) guides are available.
- A new [Using Rich Presence with the Embedded App SDK](https://docs.discord.com/developers/rich-presence/using-with-the-embedded-app-sdk) guide is available.
 September 20, 2024

### Soundboard API

[Soundboard](https://docs.discord.com/developers/resources/soundboard) is available in the API. Apps can [get](https://docs.discord.com/developers/resources/soundboard#list-default-soundboard-sounds), [modify](https://docs.discord.com/developers/resources/soundboard#modify-guild-soundboard-sound), and [send](https://docs.discord.com/developers/resources/soundboard#send-soundboard-sound) soundboard sounds in voice channels, and can listen for other users playing them. September 17, 2024 Voice

### Voice End-to-End Encryption (DAVE Protocol)

New [high-level documentation](https://docs.discord.com/developers/topics/voice-connections) explains the DAVE protocol for encrypted Discord voice/video sessions and the [new voice gateway opcodes](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) that implementations must handle.

### **Developer Impact**

Beginning in September 2024, Discord started rolling E2EE into DM calls, Group DM calls, voice-channel sessions, and Go Live streams. **Who this affects:** Libraries or apps that implement [Voice Connections](https://docs.discord.com/developers/topics/voice-connections). Immediate E2EE support is not required because calls automatically upgrade or downgrade according to the clients present in the call.

### **Implementing E2EE Voice**

The new voice documentation gives an overview of DAVE and the voice gateway opcodes that DAVE-capable clients need. The most complete DAVE protocol details are in the [Protocol Whitepaper](https://daveprotocol.com/). The open-source [libdave](https://github.com/discord/libdave) library can help implementation. The [Voice Gateway Opcodes section of the protocol whitepaper](https://daveprotocol.com/#voice-gateway-opcodes) defines the exact DAVE protocol opcode format.

### **Future Deprecation and Discontinuation of Non-E2EE Voice**

Discord plans to remove non-E2EE support for those DM, Group DM, voice-channel, and Go Live voice paths after a transition period. During 2025, official Discord clients are expected to support DAVE, after which eligible audio/video session types will require it. Once Discord finalizes the deprecation and discontinuation timeline, developers will have **at least six months** to implement before non-E2EE voice connections are sunset. Additional material on DAVE is available here:
- [Meet DAVE: Discord's New End-to-End Encryption for Audio & Video](https://discord.com/blog/meet-dave-e2ee-for-audio-video)
- [DAVE Protocol Whitepaper](https://daveprotocol.com/)
- [libdave open-source library on GitHub](https://github.com/discord/libdave)
 September 04, 2024Interactions

### Add Polls when Editing Deferred Interaction Responses

The [Edit Original Interaction Response](https://docs.discord.com/developers/interactions/receiving-and-responding#edit-original-interaction-response) endpoint can now add a poll while editing a deferred interaction response. August 28, 2024 Premium AppsBreaking Changes

### Premium Apps: Entitlement Migration and New Subscription API

An update to this Change Log entry was published on **October 7, 2024**. See the [new Change Log entry](https://docs.discord.com/developers/change-log#updates-to-entitlement-migration-guide) for current details. Discord is migrating entitlements to behavior where they do not end until explicitly canceled, which is a breaking change for subscription management. The new [Subscription API](https://docs.discord.com/developers/resources/subscription) and [Subscription Events](https://docs.discord.com/developers/events/gateway-events#subscriptions) support subscription-related handling. Rollout begins on October 1, 2024 for all existing applications with user and guild subscription SKU entitlements.

#### Entitlement Migration Details

- `ENTITLEMENT_CREATE` events now fire with `ends_at` set to null for ongoing subscriptions, marking an indefinite entitlement.
- `ENTITLEMENT_UPDATE` events fire only when a subscription ends, with `ends_at` carrying the end date.
- Discord-managed Subscription entitlements use `type` value `PURCHASE` (type `1`) instead of `APPLICATION_SUBSCRIPTION` (type `8`).

### Migration Plan & Guide:

As of **October 1, 2024**, existing entitlements that grant access to user-subscription and guild-subscription SKUs automatically move to the new system on their renewal date. This creates a month-long migration window for entitlements to move on renewal. Developers should update systems before rollout to handle `ENTITLEMENT_CREATE` and `ENTITLEMENT_UPDATE` as described in the migration guide and avoid service disruptions.

### Introducing a New Subscription API

Under the new entitlement behavior, subscription SKU entitlements no longer emit events at the start of each new billing period. Subscription lifecycle handling should move to the new [Subscription API](https://docs.discord.com/developers/monetization/implementing-app-subscriptions#using-the-subscription-api). Use the [Subscription resource](https://docs.discord.com/developers/resources/subscription) for calling the Subscription API and responding to Subscription events. For implementation details, read [Implementing App Subscriptions](https://docs.discord.com/developers/monetization/implementing-app-subscriptions#using-the-subscription-api). This API is available now.

### Monetization Documentation Updates

Premium Apps documentation was updated for these changes.
- New [Enabling Monetization](https://docs.discord.com/developers/monetization/enabling-monetization) page covers team setup, payout management, and enabling monetization for apps
- New [Managing SKUs](https://docs.discord.com/developers/monetization/managing-skus#creating-a-sku) page explains creating, updating, publishing, and promoting SKUs
- [Entitlement](https://docs.discord.com/developers/resources/entitlement), [SKU](https://docs.discord.com/developers/resources/sku), and [Subscription](https://docs.discord.com/developers/resources/subscription) resources were moved or added to the **Resources** section
- [Implementing App Subscriptions](https://docs.discord.com/developers/monetization/implementing-app-subscriptions) and [Implementing One-Time Purchases](https://docs.discord.com/developers/monetization/implementing-one-time-purchases) guides were updated

### Subscription Entitlement Migration Guide

Starting on **October 1, 2024**, Discord is migrating the existing entitlement system to behavior where **entitlements do not expire until explicitly canceled**. The guide describes developer-facing changes and how to manage them. With this update, subscription SKU entitlements no longer emit events when a new billing period begins. To detect renewal, use the new [Subscription API](https://docs.discord.com/developers/resources/subscription) and related [Subscription Gateway Events](https://docs.discord.com/developers/events/gateway-events#subscriptions).

### Current System

Subscription SKUs purchased through Discord currently have entitlements with:
- An `ends_at` date tied to the subscription interval, updated each billing cycle.
- An entitlement `type` value of `APPLICATION_SUBSCRIPTION` (type `8`).
- An `ENTITLEMENT_UPDATE` event at the start of each new subscription period.

### New System

After migration, entitlements for Subscription SKUs purchased through Discord will:
- Have no end date until cancellation (`ends_at` will be `null`).
- Use entitlement `type` value `PURCHASE` (type `1`).
- Stop sending `ENTITLEMENT_UPDATE` events until the subscription is canceled.

### Migration Timeline

- **Migration Start Date:** October 1, 2024
- **Migration End Date:** November 1, 2024

### Migration Impacts

### 1) Existing Entitlements Scheduled to Renew

- **During Migration Window:**
  - These entitlements transfer to the new system automatically.
  - A new `ENTITLEMENT_CREATE` event marks the migration, but does not represent a net new entitlement.
  - No further events are generated until the entitlement ends, which then triggers `ENTITLEMENT_UPDATE`.
  - In both the `ENTITLEMENT_UPDATE` event and the Entitlement API, `ends_at` gives the entitlement end timestamp.

### 2) Existing Entitlements Set to End

- **During Migration Window:**
  - These entitlements expire normally and do not renew under the new system.
  - No new entitlement events are generated for these cases.

### Developer Actions

- **Pre-Migration:**
  - Review the new entitlement event structure.
  - Update systems to handle `ends_at` as null, meaning the entitlement is indefinite.
  - Stop expecting type `APPLICATION_SUBSCRIPTION` (type `8`) for Discord-managed subscription entitlements.

- **Post-Migration:**
  - Watch for `ENTITLEMENT_CREATE`, `ENTITLEMENT_UPDATE`, `SUBSCRIPTION_CREATE`, and `SUBSCRIPTION_UPDATE` events.
  - Update references to entitlement `ends_at` timestamps so they represent the entitlement end. To find a subscription period end, use the [Subscription API](https://docs.discord.com/developers/resources/subscription) and related [Subscription Gateway Events](https://docs.discord.com/developers/events/gateway-events#subscriptions).

 August 26, 2024ActivitiesInteractions

### Launching Activities in Response to Interactions
