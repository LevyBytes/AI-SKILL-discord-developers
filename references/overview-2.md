# Components — Overview

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Components let an app place interactive UI in modals and in messages it sends. They are designed to be accessible, configurable, and straightforward to apply. For a primer on modal behavior and component types, see the [Components & Modals](https://docs.discord.com/developers/platform/components) platform page. (Figure: Image showing examples of components UI) Messages that use components must include the `IS_COMPONENTS_V2` flag (`1<<15`). When that flag is present, classic message content and embeds are unavailable; the message body must be represented entirely through components. [Legacy message component behavior](https://docs.discord.com/developers/components/reference#legacy-message-component-behavior) will **not** be deprecated and remains usable by apps per message. For new builds and features, use the new components model.

### Using Message Components

Explains how to send Message Components and includes examples.

### Using Modal Components

Explains how to send Modal Components and includes examples.

### Component Reference

Explore the Components reference documentation.

---

### Get Help & Join the Community

 Use this section when you need help or want to talk with other app developers.
- Join the [DDevs Discord Server](https://discord.gg/discord-developers) to ask the community for help, exchange best practices, and find additional ways to improve your apps.
