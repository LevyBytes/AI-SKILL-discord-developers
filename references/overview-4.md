# Events — Overview

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Applications can observe activity in Discord so they remain synchronized with changes involving servers, users, and the app itself. The sections below introduce the supported event transport options and point to the relevant documentation for each path.

### Receiving Events

 Different categories of events are available through different delivery mechanisms:
- **[Gateway events](https://docs.discord.com/developers/events/overview#using-gateway)** arrive over a WebSocket session between Discord and your app, and they are the main event transport. **Most events require Gateway connections.**
- **[Webhook events](https://docs.discord.com/developers/events/overview#using-webhooks)** are delivered by HTTP requests to the app's Webhook Event URL.
- **[SDK events](https://docs.discord.com/developers/events/overview#using-the-embedded-app-sdk)** are emitted to an app that is built with the Embedded App SDK.
 The following sections describe each event-receiving approach in more detail.

### Using Gateway

 Gateway events are the central mechanism for apps to receive and publish event traffic. Resource changes in Discord, including channel, guild, role, and message updates, are generally exposed only through the [Gateway](https://docs.discord.com/developers/events/gateway). Discord sends Gateway events across a WebSocket-backed [Gateway connection](https://docs.discord.com/developers/events/gateway#connections) maintained by your app. To consume these events, the app needs to establish and keep a persistent Gateway connection; connection behavior is documented in [Gateway documentation](https://docs.discord.com/developers/events/gateway#connections). A [developer library](https://docs.discord.com/developers/developer-tools/community-resources#libraries) can simplify connection setup, lifecycle management, and common concerns such as [rate limits](https://docs.discord.com/developers/events/gateway#rate-limiting). For receiving behavior, see the [Gateway documentation](https://docs.discord.com/developers/events/gateway#receiving-events); for the complete event catalog, see [Gateway Events documentation](https://docs.discord.com/developers/events/gateway-events).

### Using Webhooks

 Webhook events provide HTTP delivery for a limited set of events. Although many event types cannot be received this way, some, such as [Application Authorized](https://docs.discord.com/developers/events/webhook-events#application-authorized), which fires when an app is installed for a user or server, are not exposed through transports such as Gateway. The [Webhook Events documentation](https://docs.discord.com/developers/events/webhook-events) explains subscriptions and handling, and its [event types](https://docs.discord.com/developers/events/webhook-events#event-types) section lists the supported webhook events.

### Using the Embedded App SDK

 For [Activities](https://docs.discord.com/developers/activities/overview), the Embedded App SDK can emit [SDK events](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#sdk-events), including changes to a user's voice state or screen orientation. Subscribe by calling [`subscribe()`](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#subscribe) with the SDK event name. The [Embedded App SDK documentation](https://docs.discord.com/developers/developer-tools/embedded-app-sdk) covers the event-listening model for this SDK.

### Next Steps

### Gateway Events

Gateway is the main event channel for Discord apps. Use this material to understand Gateway connections and Gateway Events.

### Webhook Events

Details for receiving and answering the limited set of Webhook Events delivered over HTTP.

### Embedded App SDK Events

Instructions for subscribing to Activity-specific events and handling them through the Embedded App SDK.
