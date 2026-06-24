# Change Log (part 4)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

**All existing code will continue to work without changes.** The `/.proxy/` path prefix remains fully supported and will be kept indefinitely. You can:
- Continue using existing `/.proxy/` URLs
- Move to the newer, simpler URLs
- Use both URL patterns in one application at the same time
**No migration is required.** This additive update increases what developers can do without breaking existing behavior. July 28, 2025HTTP API

### Guild Create Deprecation

Apps can no longer create guilds. The endpoint documentation has been removed, and the endpoints are no longer present in the OpenAPI specification. See the [earlier changelog entry](https://docs.discord.com/developers/change-log#deprecating-guild-creation-by-apps) for details. July 17, 2025ActivitiesEmbedded App SDK

### Clickable Links and Customizable Statuses in Rich Presence

Rich Presences now support more interactive and flexible experiences for people using your application. The update has two major pieces:
- Clickable links can be added to state text, details text, large image, and small image
- You can choose whether the name, state, or details field appears in user status text in the member list, such as showing "Listening to Rick Astley" instead of "Listening to MyMusic"
The new fields are documented in the [Activity Object](https://docs.discord.com/developers/events/gateway-events#activity-object) section of Gateway Events and are also exposed through the Embedded App SDK. July 02, 2025HTTP API

### Gradient Roles and Guild Tags

Gradient role colors and guild tags are now documented for the API. Guild tags let users show a favorite server with a 1-4 character badge beside their display name, available through the `primary_guild` field on the user object. Servers are no longer limited to one flat role color; they can configure gradients through the new `colors` field on the role object. The role `color` field is deprecated by this change, but it continues to work for backwards compatibility.

#### Gradient Role Colors

- The guild feature `ENHANCED_ROLE_COLORS` indicates whether a guild can assign gradient colors to roles.
- Guild roles now include `colors` in their [structure](https://docs.discord.com/developers/topics/permissions#role-object-role-structure).
- `color` on guild roles is deprecated, but the API still returns it and it remains supported for backwards compatibility.
- [Role color structure](https://docs.discord.com/developers/topics/permissions#role-object-role-colors-object)

#### Guild Tags

- Retrieve guild tags from the `primary_guild` field on the user object.
- [User Primary Guild](https://docs.discord.com/developers/resources/user#user-object-user-primary-guild)
 June 26, 2025Discord Social SDK

### Discord Social SDK Release 1.4

A Discord Social SDK release is available with these updates:

### Lobby Chat History

- Added [`Client::GetLobbyMessagesWithLimit`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a0586192e85caf548b8b321f1cb21301f) for fetching history for a specified lobby ID. A request can return no more than 200 messages and can look back at most 72 hours.
- Retrieval is limited to messages from lobbies where the user is currently a member.
- DM history support is planned soon too.

### Unified Friends List

- Added [`Client::GetRelationshipsByGroup`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a9f7898d3f3d1ec92b06c662df70746d5), which groups a user's relationships for friends-list rendering and sorts users according to the [Unified Friends List design guidelines](https://docs.discord.com/developers/discord-social-sdk/design-guidelines/unified-friends-list). Previously, developers had to call [`Client::GetRelationships`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ad481849835cd570f0e03adafcf90125d), split each relationship into the correct friend group, and implement their own sorting.
- Added [`Client::SetRelationshipGroupsUpdatedCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#af12441ef091298f968075b7190851098), which fires when a user change may invalidate a sorted friends list returned by [`Client::GetRelationshipsByGroup`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a9f7898d3f3d1ec92b06c662df70746d5). Call [`Client::GetRelationshipsByGroup`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a9f7898d3f3d1ec92b06c662df70746d5) again to refresh the list.
- Added `IsSpamRequest` to [`RelationshipHandle`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1RelationshipHandle.html#a7da36b15ad0b7d38ba658a622e9ded77), returning `true` when Discord identifies the request as spam.

### Audio Changes

- Added an experimental mobile audio mode that uses standard media audio streams instead of voice-specific processing. On iOS, the voice engine uses the Remote I/O Audio Unit instead of Voice Processing I/O; on Android, media stream types replace voice communication types. Enable it by creating a Client with a [`ClientCreateOptions`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1ClientCreateOptions.html#ae655ad66ba64f443496c158307cc77b4) parameter whose `experimentalAudioSystem` property is `AudioSystem::Game`. Also set [`Client::SetEngineManagedAudioSession`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ade08897214152b9acfa79c263e77e366) to true. Treat this as non-production functionality; Discord is looking for test feedback.
- Added [`Client::SetAecDump`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a3a05b2cafaa546d915a5249c63f4059f) for recording audio diagnostic data.

### Auth

- Publisher Auth
  - Publisher Auth is a new feature that simplifies authorization for publishers managing multiple games. This early release is currently limited to selected partners.
  - Added [`Client::ExchangeChildToken`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a59f5d9d14f79eb318bf4d57f4e87a5c1) to support child token exchange for public clients. Confidential clients need a server to server implementation, but this method can help during development.
  - The SDK can now see invites from sibling applications. Identify them through the `applicationId` field on the [`ActivityInvite`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1ActivityInvite.html#af980f140c1459e1cd8f6ef3f3c07547c) payload.
  - The SDK can now see messages sent by other sibling applications. Identify them with the `ApplicationId` method on [`MessageHandle`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1MessageHandle.html#ae25595b43bc74b0c4c92c5165d16382f).

- Added [`Client::RevokeToken`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a30ccea6366efaf0b884efcdcc28a6f2d) and [`Client::UnmergeIntoProvisionalAccount`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a2da21ae8a3015e0e5e42c1a7226b256f) so games using Public Clients can revoke tokens or unmerge directly from clients.

### Android

- The SDK now supports 16KB page size.

### Misc

- Activity serialization now avoids null or empty keys in the JSON payload.
 June 25, 2025HTTP API

### Paginated Pin Endpoints

New endpoints are available for managing paginated pins in channels. Get Channel Pins retrieves pinned messages with pagination support, which is more efficient for channels with many pinned messages. Pin and Unpin keep their behavior but use a new route. The older pinned-message endpoints are deprecated. The new endpoints should be straightforward to adopt because they keep similar functionality while adding pagination.

#### New Endpoints

**[Get Channel Pins](https://docs.discord.com/developers/resources/message#get-channel-pins)**: Retrieve paginated pinned messages in a channel: **[Pin Message](https://docs.discord.com/developers/resources/message#pin-message)**: Pin a message in a channel: **[Unpin Message](https://docs.discord.com/developers/resources/message#unpin-message)**: Unpin a message in a channel:

#### Deprecated Endpoints

**[Get Pinned Messages](https://docs.discord.com/developers/resources/message#get-pinned-messages-deprecated)**: **[Pin Message](https://docs.discord.com/developers/resources/message#pin-message-deprecated)**: **[Unpin Message](https://docs.discord.com/developers/resources/message#unpin-message-deprecated)**:  June 05, 2025Discord Social SDK

### Discord Social SDK Release 1.3

A Discord Social SDK release is available with these updates:

### Authentication

- Added an `APPLICATION_DEAUTHORIZED` webhook event configurable in the developer portal. When a user unlinks an account or otherwise revokes application authorization, the event is sent to configured webhooks with serialized user information in the payload. See [Webhook Events](https://docs.discord.com/developers/events/webhook-events) for webhook configuration details.

### PC

- Added configurable request timeouts for SDK HTTP client requests. This release supports PC, with console and mobile support planned for a later release. The timeout default is 30000ms (30 seconds) and can be changed through [`Client::SetHttpRequestTimeout`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ae5fe2518b0b1b05ee1745ab0a79096b9).
- Fixed a crash that could happen while handling certain failed HTTP requests.

### Mobile

- [`Client::SetSpeakerMode`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ac269ab57407e3b83e2bb2d30895e666d) is deprecated. Unless [`Client::SetEngineManagedAudioSession`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ade08897214152b9acfa79c263e77e366) is used, the SDK handles audio routing automatically.

#### Android

- Fixed game and voice audio routing when external audio devices connect or disconnect. [`Client::SetEngineManagedAudioSession`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ade08897214152b9acfa79c263e77e366) was added to indicate that the SDK should not manage audio routing or automatically enter and leave `MODE_IN_COMMUNICATION` when calls start and end.
- Fixed an Authorize method issue when an activity must restart after a device configuration change.

#### iOS

- Fixed several audio routing and session-management issues. In the Unity plugin, game audio no longer stops when a call ends. For standalone SDK use, [`Client::SetEngineManagedAudioSession`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#ade08897214152b9acfa79c263e77e366) was added to indicate that the SDK should not automatically start and stop `AVAudioSession` when joining and leaving calls.
- Corrected supported platform values in `Info.plist` for iOS .frameworks.

### Consoles

- Standalone archives now include only console-specific files, matching the Unity and Unreal Engine archives.

### Misc

- Fixed a thread-safety issue in [`Client::AddLogCallback`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#af78996cff24a40f5dc7066beed16692c).
- Added a [Flags] declaration for bit flag enums in C#.
 May 05, 2025Discord Social SDK

### Discord Social SDK Release 1.2

A Discord Social SDK release is available with these updates:

### Rich Presence

- Added custom button support on activity cards through [`Activity::AddButton`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Activity.html#aab07650fcff18565eb78a1e2df46627e).

### Packaging

- Unity and Unreal plugin artifacts now include only the extra files needed for console support, so they can be extracted over the base plugin.
- The Unity plugin now ships as a .zip to extract inside your project's Packages directory, enabling the behavior described above.
- Console archives now include a small README with console-specific documentation.

### Misc

- Added [`Client::OpenConnectedGamesSettingsInDiscord`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a24f268f5eebe9919a3f774354eb577e0), which deeplinks to Discord settings for connected games and gives players some control over who can DM them.
- Fixed a Linux hang that could occur in [`Client::RegisterLaunchCommand`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a024d7222931fdcb7d09c2b107642ecab) and [`Client::RegisterLaunchSteamApplication`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a45b2c791c5b06f77d457dacb53dfba40).
- [`Client::RegisterLaunchCommand`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a024d7222931fdcb7d09c2b107642ecab) and [`Client::RegisterLaunchSteamApplication`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a45b2c791c5b06f77d457dacb53dfba40) now function inside the Steam Runtime on Linux.
- Fixed a crash on exit that could occur while callbacks remained pending in the queue.
 April 29, 2025User AppsHTTP APIInteractions

### Raised Component Limits

The top level component limit is being removed, and messages using the [`IS_COMPONENTS_V2` message flag](https://docs.discord.com/developers/resources/message#message-object-message-flags) can now contain up to 40 components. The component count limit is also removed for a [Container Component](https://docs.discord.com/developers/components/reference#container). Legacy messages are unchanged and still allow up to 5 action rows.

#### What’s New

- **Total components**: A message can now contain up to 40 total components.
- **Top-level components**: Messages no longer have a top level component limit; the previous limit was 10.
- **[Container Component](https://docs.discord.com/developers/components/reference#container)**: Container Component no longer limits component count; the previous limit was 10.

#### Developer Resources

- See the [Component Reference](https://docs.discord.com/developers/components/reference) for full specifications for every available component.
- Use [Using Message Components](https://docs.discord.com/developers/components/using-message-components) to learn how to build rich component-based message layouts.
 April 22, 2025User AppsHTTP APIInteractions

### Introducing New Components for Messages!

New message components are available for apps, giving developers full control over message layout.

#### Why We Built Components V2

The earlier components system worked, but it had constraints:
- Content, attachments, embeds, and components were forced into fixed vertical positions.
- Visual styling options were limited
- Building cohesive visual experiences was difficult because message capabilities were expressed through separate, non-unified systems.
The new component system addresses those issues with composable components that can be ordered and arranged freely, enabling more flexible and visually coherent layouts.

#### What’s New

[Components V2](https://docs.discord.com/developers/components/overview) adds several component types for messages:

#### New Layout Components

- [**Section**](https://docs.discord.com/developers/components/reference#section) - Pair text with an accessory component for related elements.
- [**Container**](https://docs.discord.com/developers/components/reference#container) - Group content visually with a configurable accent color.
- [**Separator**](https://docs.discord.com/developers/components/reference#separator) - Insert spacing and dividers between components.

#### New Content Components

- [**Text Display**](https://docs.discord.com/developers/components/reference#text-display) - Place rich markdown-formatted text anywhere in messages.
- [**Thumbnail**](https://docs.discord.com/developers/components/reference#thumbnail) - Use an image inside a [section](https://docs.discord.com/developers/components/reference#section).
- [**Media Gallery**](https://docs.discord.com/developers/components/reference#media-gallery) - Show image and video collections in an organized grid.
- [**File**](https://docs.discord.com/developers/components/reference#file) - Include file attachments within the message layout.

#### Getting Started

To enable the new components for a message, send the `1 << 15` (`IS_COMPONENTS_V2`) message flag. Guides are available for implementing these features:
- [Using Message Components](https://docs.discord.com/developers/components/using-message-components) - Learn to create rich message layouts with components.
- [Using Modal Components](https://docs.discord.com/developers/components/using-modal-components) - Build interactive forms and dialogs.

#### Compatibility Notes

[Legacy component behavior](https://docs.discord.com/developers/components/reference#legacy-message-component-behavior) remains supported, so existing integrations keep working. When the Components V2 flag is used, these changes apply:
- `content` and `embeds` no longer work; use [Text Display](https://docs.discord.com/developers/components/reference#text-display) and [Container](https://docs.discord.com/developers/components/reference#container) instead.
- Attachments must be surfaced through components to appear. Use a [Media Gallery](https://docs.discord.com/developers/components/reference#media-gallery), [Thumbnail](https://docs.discord.com/developers/components/reference#thumbnail), or [File](https://docs.discord.com/developers/components/reference#file) component.
- The `poll` and `stickers` fields are disabled.
- Messages can have a max of 10 top-level components and 30 total components.

#### Developer Resources

See the [Component Reference](https://docs.discord.com/developers/components/reference) for detailed specifications covering all available components. April 21, 2025Discord Social SDK

### Discord Social SDK Release 1.1.8318

A Discord Social SDK release is available with these updates:

### Platforms

- Playstation standalone archives now include linker stubs

### Voice

- Fixed a Linux audio playback regression.
After enabling the Discord Social SDK for your application, download the Discord Social SDK binaries from the Developer Portal. To learn more, see the [Discord Social SDK Overview](https://docs.discord.com/developers/discord-social-sdk/overview). April 16, 2025Discord Social SDK

### Discord Social SDK Release 1.1

A Discord Social SDK release is available with these updates:

### Platforms

- Added support for Xbox One and PS4 consoles.

### Auth

- Added Unity Services support as an external auth provider.

### Voice

- [`Client::StartCallWithAudioCallbacks`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#abcaa891769f9e912bfa0e06ff7221b05) now allows sample data changes during record and playback for custom effects processing.
- Fixed a bug where a user's speaking state could remain stuck on.
- Added [`Call::GetPTTReleaseDelay`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Call.html#ab8dc6b1527728fecb17f266d5b3e9e6e)
- Voice engine initialization is now deferred until needed.
- Fixed a Linux PulseAudio backend deadlock where faulty audio devices could lock up the voice engine.

### Rich Presence

- Added desktop support so rich presence updates and invites can be sent without a Discord gateway connection.

### Misc

- Added Linux support for [`Client::RegisterLaunchCommand`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a024d7222931fdcb7d09c2b107642ecab) and [`Client::RegisterLaunchSteamApplication`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#a45b2c791c5b06f77d457dacb53dfba40).
- Fixed a crash when a Unity Editor scripting domain reload, such as entering or exiting play mode, occurs while an async completion callback is pending.
- Fixed [`Client::RemoveDiscordAndGameFriend`](https://discord.com/developers/docs/social-sdk/classdiscordpp_1_1Client.html#aa6d393a3d98ec5d06faef49a57d1a89b) so it no longer works only when you are Discord friends.
- Reduced log spam from desktop client RPC message handling.
After enabling the Discord Social SDK for your application, download the Discord Social SDK binaries from the Developer Portal. To learn more, see the [Discord Social SDK Overview](https://docs.discord.com/developers/discord-social-sdk/overview). April 15, 2025HTTP API

### Deprecating Guild Creation by Apps

### Breaking Change

Discord is deprecating application access to guild creation through the `Create Guild` endpoint to address security concerns.

#### What’s Changing

- Starting July 15, 2025, applications will be restricted from using the Create Guild endpoint (`POST /guilds`).
- Existing Guilds owned by bots will be transferred to ownership by a real user.
- The endpoint will no longer be available after the deprecation date.

#### Timeline

- **April 15, 2025**: Deprecation announcement
- **June 15, 2025**: Discord sends System DM/Email notices to impacted app owners and the designated guild members.
- **July 15, 2025**: `Create Guild` endpoint will no longer be available.
Affected apps will receive a migration plan through Discord System DM. Discord recognizes that this may affect legitimate use cases; contact the [Developer Support portal](https://support-dev.discord.com/hc) if you have questions or believe your application needs continued guild creation access. April 11, 2025ActivitiesEmbedded App SDK

### Custom Incentivized Links

### Custom Incentivized Links for Activities

Custom Incentivized Links control how incentivized link embeds appear to users. Create them in the developer portal or generate them from inside your activity. Incentivized Links can support referral links, promotions, deep links into an activity, and similar uses.
- shareLink now supports attaching custom params to shared game links with `custom_id`.
- `referrer_id` was removed from the shareLink API. Move any `referrer_id` usage to `custom_id`; passing `referrer_id` to shareLink silently fails.
Learn about [creating and managing Custom Incentivized Links](https://docs.discord.com/developers/activities/development-guides/growth-and-referrals#creating-and-managing-custom-incentivized-links) and [how to generate them from within your activity](https://docs.discord.com/developers/activities/development-guides/growth-and-referrals#generating-a-custom-link-within-your-activity) with the shareLink API. The Embedded App SDK is available through [npm](https://www.npmjs.com/package/@discord/embedded-app-sdk) and [GitHub](https://github.com/discord/embedded-app-sdk). Use the [installation guide and reference](https://docs.discord.com/developers/developer-tools/embedded-app-sdk) to get started. April 03, 2025HTTP APIInteractions

### Per-Attachment File Upload Behavior for Apps
