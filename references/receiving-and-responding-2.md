# Receiving And Responding (part 2)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

DELETE/webhooks/[{application.id}](https://docs.discord.com/developers/resources/application#application-object)/[{interaction.token}](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object)/messages/@original Removes the original Interaction response. A successful request returns `204 No Content`.

### Create Followup Message

 POST/webhooks/[{application.id}](https://docs.discord.com/developers/resources/application#application-object)/[{interaction.token}](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object) If an interaction comes from a user-installed app that is not installed in the server, apps may create no more than 5 followup messages for that interaction; then the [authorizing integration owners object](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-authorizing-integration-owners-object) contains only `USER_INSTALL`. This endpoint creates a followup message for an Interaction. It behaves like [Execute Webhook](https://docs.discord.com/developers/resources/webhook#execute-webhook), except `wait` is always true. Interaction followups do not support the `thread_id`, `avatar_url`, or `username` parameters on this endpoint. Set the `EPHEMERAL` [message flag](https://docs.discord.com/developers/resources/message#message-object-message-flags) `1 << 6` (64) when the message should be visible only to the user. Set the `IS_COMPONENTS_V2` [message flag](https://docs.discord.com/developers/resources/message#message-object-message-flags) `1 << 15` (32768) to send a [component](https://docs.discord.com/developers/components/reference)-based message. For backward compatibility, calling this endpoint immediately after a `DEFERRED_CHANNEL_MESSAGE_WITH_SOURCE` interaction response acts as [Edit Original Interaction Response](https://docs.discord.com/developers/interactions/receiving-and-responding#edit-original-interaction-response). Under that legacy behavior, Discord edits the loading message instead of creating a separate message. The ephemeral setting from this followup call is ignored; Discord keeps the value sent in the original defer response because an existing message cannot switch ephemeral state. This compatibility behavior is deprecated; use [Edit Original Interaction Response](https://docs.discord.com/developers/interactions/receiving-and-responding#edit-original-interaction-response) for that case.

### Get Followup Message

 GET/webhooks/[{application.id}](https://docs.discord.com/developers/resources/application#application-object)/[{interaction.token}](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object)/messages/[{message.id}](https://docs.discord.com/developers/resources/message#message-object) Fetches an Interaction followup message. Its behavior matches [Get Webhook Message](https://docs.discord.com/developers/resources/webhook#get-webhook-message).

### Edit Followup Message

 PATCH/webhooks/[{application.id}](https://docs.discord.com/developers/resources/application#application-object)/[{interaction.token}](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object)/messages/[{message.id}](https://docs.discord.com/developers/resources/message#message-object) Updates an Interaction followup message. It follows the same rules as [Edit Webhook Message](https://docs.discord.com/developers/resources/webhook#edit-webhook-message).

### Delete Followup Message

 DELETE/webhooks/[{application.id}](https://docs.discord.com/developers/resources/application#application-object)/[{interaction.token}](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object)/messages/[{message.id}](https://docs.discord.com/developers/resources/message#message-object) Removes an Interaction followup message. Success is indicated by `204 No Content`.
