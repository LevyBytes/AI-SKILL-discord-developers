# Change Log (part 10)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

### Inline Replies

Inline Replies are now covered in the docs. Their behavior is not the same in v6 and v8, so implement them carefully:
- Inline replies use type `19` in v8, while v6 still represents them as type `0`
- Message creation can now include `message_reference` to create a reply
- The [Message Object](https://docs.discord.com/developers/resources/message#message-object) now has a `referenced_message` field
- The [Allowed Mentions Object](https://docs.discord.com/developers/resources/message#allowed-mentions-object) now has a `replied_user` field
- [Message Create](https://docs.discord.com/developers/events/gateway-events#message-create) gateway event includes `referenced_message` when the created message is a reply; outside replies, that field is not guaranteed.
 November 13, 2020

### Stickers

Stickers are now described on the [message](https://docs.discord.com/developers/resources/message#message-object) object. October 27, 2020

### Gateway v6 Intent Restrictions

Gateway v6 now enforces gateway intent restrictions. The newer chunking limits apply regardless of which intents are in use; see [Request Guild Members](https://docs.discord.com/developers/events/gateway-events#request-guild-members). Bots without privileged intents enabled in the application dashboard will not receive the events covered by those intents. Other intents remain enabled by default unless the identify payload says otherwise. Discord also published [Gateway Update FAQ](https://dis.gd/gwupdate) with more detail about the changes and common issues. September 24, 2020

### API and Gateway V8

API and Gateway v8 are now introduced. The documentation marks the changes, and [this commit in our docs repo](https://github.com/discord/discord-api-docs/commit/545ff4a7883e5eee7ee91d19a5e5d760a0730033) shows the full diff. The changes are:
- API and Gateway v8 are available now; v6 remains the default for the moment.
- [Gateway Intents](https://docs.discord.com/developers/events/gateway#gateway-intents) must now be used
- `guild_subscriptions` in identify was removed and replaced by [Gateway Intents](https://docs.discord.com/developers/events/gateway#gateway-intents).
- Permission values now use string-serialized numbers everywhere, so `permissions_new`, `allow_new`, and `deny_new` were removed
- The `game` field was removed; use the first item in `activities` when you need an equivalent value
- Channel Permission Overwrite `type`s now use numeric values 0 and 1 instead of string values `role` and `member`. Because of a current technical constraint, audit log `options` still expose them as string-serialized numbers.
- `embed_enabled` and `embed_channel_id` were removed; use `widget_enabled` and `widget_channel_id` instead.
- Form body validation errors now return clearer messages. [See more here](https://docs.discord.com/developers/reference#error-messages)
- `Retry-After` header values and `retry_after` response body values now use seconds, not milliseconds; for example, `123` means 123 seconds
- `X-RateLimit-Precision` is ignored now. `X-RateLimit-Reset` and `X-RateLimit-Reset-After` always use millisecond precision, such as `123.456` rather than `124`
- Bots no longer receive [Channel Create Gateway Event](https://docs.discord.com/developers/events/gateway-events#channel-create) for DMs
- `delete-message-days` is no longer supported; use `delete_message_days`.
- [Presence Update Gateway Event](https://docs.discord.com/developers/events/gateway-events#presence-update) no longer includes `roles`, `premium_since`, or `nick`
- Some [integration object](https://docs.discord.com/developers/resources/guild#integration-object) fields were removed for Discord application integrations
- [Get Guild Integrations](https://docs.discord.com/developers/resources/guild#get-guild-integrations) no longer accepts `include_applications`; application integrations are always returned.
- These deprecated routes were removed in favor of clearer route names:
Replaced by `/guilds/<guild_id>/widget`:
- `/guilds/<guild_id>/embed`
Replaced by `/guilds/<guild_id>/widget.json`:
- `/servers/<guild_id>/embed.json`
- `/servers/<guild_id>/widget.json`
- `/guilds/<guild_id>/embed.json`
Replaced by `/guilds/<guild_id>/widget.png`:
- `/guilds/<guild_id>/embed.png`
Replaced by `/channels/<channel_id>/messages/bulk-delete`:
- `/channels/<channel_id>/messages/bulk_delete/`
Replaced by `/invites/<code>/`:
- `/invite/<code>/`
 July 28, 2020

### New Permission Fields

Documented `permissions_new`, `allow_new`, and `deny_new` as string-serialized permission bitfields. May 11, 2020

### Legacy Mention Behavior Deprecation

Legacy bot mention behavior has been removed, and mention control should now be handled through the [Allowed Mentions](https://docs.discord.com/developers/resources/message#allowed-mentions-object) API. April 24, 2020

### New Properties on Guild Members Chunk Event

[Guild Members Chunk](https://docs.discord.com/developers/events/gateway-events#guild-members-chunk) gateway events now include `chunk_index` and `chunk_count`. Use them to track how many chunk events remain after a [Request Guild Members](https://docs.discord.com/developers/events/gateway-events#request-guild-members) command. March 03, 2020

### New Allowed Mentions Object

Discord added a more precise way to control mentions. The change also starts a 60 day deprecation period for legacy mention behavior. Read more:
- [Allowed mentions object](https://docs.discord.com/developers/resources/message#allowed-mentions-object)
 March 02, 2020

### New Invite Events and Reactions Endpoint

A new endpoint can delete every reaction for a specific emoji on a message, and new invite and reaction gateway events were added. Read more:
- [Delete All Reactions for Emoji](https://docs.discord.com/developers/resources/message#delete-all-reactions-for-emoji)
- [Invite Create](https://docs.discord.com/developers/events/gateway-events#invite-create)
- [Invite Delete](https://docs.discord.com/developers/events/gateway-events#invite-delete)
- [Message Reaction Remove Emoji](https://docs.discord.com/developers/events/gateway-events#message-reaction-remove-emoji)
 February 26, 2020

### Rich Presence Spectate Approval

Rich Presence [Spectate](https://docs.discord.com/developers/developer-tools/game-sdk#onactivityspectate) no longer needs approval or whitelisting. February 14, 2020

### Gateway Intents

Documentation now covers [Gateway Intents!](https://docs.discord.com/developers/events/gateway#gateway-intents) Gateway Intents let you declare the gateway events your app wants, reducing bandwidth and CPU usage. Intents alter some existing event and command behavior, so refer to:
- [Guild Create](https://docs.discord.com/developers/events/gateway-events#guild-create)
- [Request Guild Members](https://docs.discord.com/developers/events/gateway-events#request-guild-members)
- [Guild Member Add](https://docs.discord.com/developers/events/gateway-events#guild-member-add)
- [Guild Member Remove](https://docs.discord.com/developers/events/gateway-events#guild-member-remove)
- [Guild Member Update](https://docs.discord.com/developers/events/gateway-events#guild-member-update)
- [Presence Update](https://docs.discord.com/developers/events/gateway-events#presence-update)
- [List Guild Members](https://docs.discord.com/developers/resources/guild#list-guild-members)
 December 06, 2019

### IP Discovery Updates

The [IP discovery message](https://docs.discord.com/developers/topics/voice-connections#ip-discovery) was updated. The old format is deprecated and will be removed later. November 27, 2019

### GameSDK Version 2.5.6

Fixed a 2.5.5 regression where network handshakes failed and networking data was not sent. After updating, the networking manager and integrated lobby networking should work again. November 14, 2019

### GameSDK Version 2.5.5

GameSDK updates shipped with Linux support and Unity IL2CPP backend support. The release also fixed an issue in [`SetUserAchievement()`](https://github.com/discord/discord-api-docs/blob/legacy-gamesdk/docs/game_sdk/Achievements.md#setuserachievement). Download the latest version from the top of [Getting Started](https://docs.discord.com/developers/developer-tools/game-sdk#step-1-get-the-sdk). For GameSDK help or bug reports, join the [official Discord](https://discord.gg/discord-developers). August 22, 2019

### Changes to Special Channels

News Channels were renamed to Announcement Channels. Developer License owners retain access to existing and newly created channels. The underlying channel type remains `GUILD_NEWS = 5`. August 12, 2019

### More Precise Rate Limits

A new request header provides more exact rate limit reset times. See the [rate limits](https://docs.discord.com/developers/topics/rate-limits) docs for details. July 18, 2019

### Bot Tokens for Achievements

Bot tokens can now authorize HTTP API requests for [Achievements](https://github.com/discord/discord-api-docs/blob/legacy-gamesdk/docs/game_sdk/Achievements.md#the-api-way). June 19, 2019

### Additional Team Information

More Teams information was added to the API and docs. The [Teams](https://docs.discord.com/developers/topics/teams) page now describes team and team member objects. [Get Current Application Information](https://docs.discord.com/developers/topics/oauth2#get-current-bot-application-information) now returns a `team` object for team-owned applications, and the application docs now include missing fields for games sold on Discord. May 29, 2019

### Added Info Around Nitro Boosting Experiment

Documentation was expanded for the [Server Nitro Boosting](https://support.discord.com/hc/en-us/articles/360028038352-Server-Boosting) experiment. The update adds [message types](https://docs.discord.com/developers/resources/message#message-object-message-types) and [new fields on guilds](https://docs.discord.com/developers/resources/guild#guild-object-premium-tier). This feature remains experimental, so the fields can change. April 29, 2019

### Deprecation of Discord-RPC Rich Presence SDK

[Discord-RPC](https://github.com/discord/discord-rpc) Rich Presence is deprecated in favor of Discord's GameSDK. To use Rich Presence, read the [SDK Starter Guide](https://docs.discord.com/developers/developer-tools/game-sdk#getting-started) and review the [Activity Manager](https://docs.discord.com/developers/developer-tools/game-sdk#activities) functions. April 18, 2019

### New Invite Object Fields

The [Invite Object](https://docs.discord.com/developers/resources/invite#invite-object) now has `target_user` and `target_user_type`. January 14, 2019

### Ask to Join & Rich Presence SDK

Ask to Join can now be used without approval or whitelisting. You may build in-game UI, while Discord overlay also handles Ask to Join requests. Rich Presence SDK docs gained a few additions, including the formerly undocumented `UpdateHandlers()` function. December 11, 2018

### Documentation: Dispatch Store Listings

Dispatch store listing documentation was removed. Discord Store pages are now managed entirely in the [Developer Portal](https://discord.com/developers). November 30, 2018

### Enhancement: User Object

The [User object](https://docs.discord.com/developers/resources/user#user-object) now includes `premium_type` and `flags`, useful for Nitro status and HypeSquad house detection. June 19, 2018

### Documentation Fix: List of Open DMS in Certain Payloads

Docs now correctly state that `private_channels` in [Ready](https://docs.discord.com/developers/events/gateway-events#ready), and the `/users/@me/channels` response for a bot user, should be an empty array. The behavior existed already; the documentation lagged behind. June 11, 2018

### Deprecation: RPC online member count and members list

Server changes allow guilds to expose incomplete member lists in clients, making RPC member lists and online counts inaccurate. Those fields are deprecated and will return an empty members array and an online count of 0 going forward. February 05, 2018

### Enhancement: New Message Properties

The [Message](https://docs.discord.com/developers/resources/message#message-object) object now includes `activity` and `application` fields, with related object docs, for the new [Spotify integration](https://support.discord.com/hc/en-us/articles/360000167212-Discord-Spotify-Connection) and earlier Rich Presence improvements. January 30, 2018

### Enhancement: Get Guild Emoji Endpoint

[Get Guild Emoji](https://docs.discord.com/developers/resources/emoji#get-guild-emoji) now returns a user object when a user added the emoji. January 23, 2018

### Deprecation: Accept Invite Endpoint

[Accept Invite](https://docs.discord.com/developers/resources/invite) is deprecated as of today and will be removed on March 23, 2018. Use [Add Guild Member](https://docs.discord.com/developers/resources/guild#add-guild-member) instead. January 03, 2018Breaking Changes

### Semi-Breaking Change: Very Large Bot Sharding

Additional sharding guidance was added for bots in more than 100,000 guilds. Affected bots must make a small shard-count adjustment. See the [documentation](https://docs.discord.com/developers/events/gateway#sharding-for-large-bots). November 09, 2017

### New Feature: Rich Presence

Rich Presence is live for all developers. It lets developers integrate games with Discord in several new ways, including:
- Showing richer details about a user's current game on their profile
- Letting users send chat invites to join a party or spectate a game
- Showing `Spectate` and `Ask to Join` buttons on user profiles
For details, see the [Rich Presence site](https://discord.com/rich-presence). To begin development, [read the docs](https://docs.discord.com/developers/rich-presence/overview). October 16, 2017Breaking Changes

### Breaking Change: API & Gateway Below v6 Discontinued

[API](https://docs.discord.com/developers/reference#api-versioning) and Gateway versions earlier than v6 are discontinued after prior deprecation. Version 6 is now the default for both API and Gateway. Requests using versions below 6 return errors. September 20, 2017

### New Feature: Channel Categories

Docs were updated for Discord channel categories. The base [channel](https://docs.discord.com/developers/resources/channel#channel-object) object now has `parent_id`, and a new [channel category example](https://docs.discord.com/developers/resources/channel#channel-object-example-channel-category) was added. September 10, 2017

### New Feature: Emoji Endpoints

[Emoji endpoints](https://docs.discord.com/developers/resources/emoji) were added to the API. Bots can now manage guild emojis. August 16, 2017Breaking Changes

### Breaking Change: Presence Activity Objects

The `type` field in the [activity object](https://docs.discord.com/developers/events/gateway-events#activity-object) is now required when that object is not null in [Gateway Status Update](https://docs.discord.com/developers/events/gateway-events#update-presence) and [Presence Update](https://docs.discord.com/developers/events/gateway-events#presence-update) payloads. August 03, 2017Breaking Changes

### Breaking Change: Default Channels

Default channel behavior changed. For a given user, the `default` channel is now the highest-position channel they have permission to view. New guilds no longer get a default channel whose id matches the guild id, while existing guilds keep their #general channel id. A user may have no default channel if permissions allow no visible channel. Servers that previously repurposed #general as a read-only announcement channel after clearing its history can now delete and recreate that channel with desired permissions. The change also supports permission-based dynamic defaults. Discord is also rolling out remembered last-visited channels across sessions: new members go to the guild's default channel on first join, while existing members return to their previous channel. July 24, 2017

### New Feature: Audit Logs

Audit logs now have [documentation](https://docs.discord.com/developers/resources/audit-log). Review the feature carefully before using it. July 19, 2017Breaking Changes

### Breaking Change: Version 6

- [Channel](https://docs.discord.com/developers/resources/channel#channel-object) Object
  - `is_private` removed
  - [`type`](https://docs.discord.com/developers/resources/channel#channel-object-channel-types) now uses an integer
  - `recipient` changed to `recipients`, an array of [user](https://docs.discord.com/developers/resources/user#user-object) objects

- [Message](https://docs.discord.com/developers/resources/message#message-object) Object
  - [`type`](https://docs.discord.com/developers/resources/message#message-object-message-types) added to support system messages

- [Status Update](https://docs.discord.com/developers/events/gateway-events#update-presence-gateway-presence-update-structure) Object
  - `idle_since` renamed to `since`

⌘I
