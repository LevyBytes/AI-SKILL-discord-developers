# Embedded App Sdk (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

The Embedded App SDK manages RPC calls between your application and Discord. It supports developers building interactive Activities, including games. For more about Activities, read [Building an Activity](https://docs.discord.com/developers/activities/building-an-activity) or browse the [Sample Projects](https://docs.discord.com/developers/activities/overview#sample-projects).

---

### Install the SDK

 You can get the Embedded App SDK from **[npm](https://www.npmjs.com/package/@discord/embedded-app-sdk)** or **[GitHub](https://github.com/discord/embedded-app-sdk)**. Install it from the frontend JavaScript project directory with whichever package manager you use.

```
npm install @discord/embedded-app-sdk

```

 Once installation is complete, import the SDK and create an instance in your project.

```
import { DiscordSDK } from "@discord/embedded-app-sdk";

const discordSdk = new DiscordSDK(DISCORD_CLIENT_ID);

```

---

### SDK Methods

| Name | Description |
| --- | --- |
| ready | Resolves after your app connects successfully to the Discord client |
| subscribe | Subscribes to an Embedded App SDK Event |
| unsubscribe | Unsubscribes from an Embedded App SDK Event |
| close | Close an Embedded App |

### ready()

 Resolves after your app connects successfully to the Discord client.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ✅ | ✅ |

#### Required Scopes

 No scopes required

#### Signature

#### SDK Usage

```
async function setup() {
  await discordSdk.ready();
  // The rest of your app logic
}

```

---

### subscribe()

 Subscribes to one event from the [SDK Events](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#sdk-events) list.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ✅ | ✅ |

#### Required Scopes

 Scope requirements depend on the event. Check the Required Scopes section for the event being subscribed to.

#### Signature

#### Usage

```
await discordSdk.subscribe("SDK_EVENT_NAME", eventHandler, args);

```

---

### unsubscribe()

 Unsubscribes from [SDK Events](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#sdk-events) that the app previously subscribed to.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ✅ | ✅ |

#### Required Scopes

 No scopes required

#### Signature

 *`EventPayloadData` depends on the event being unsubscribed from. See that [event](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#sdk-events) for its payload details.*

#### Usage

```
await discordSdk.unsubscribe("SDK_EVENT_NAME");

```

---

### close()

 Closes your app with the supplied code and reason.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ✅ | ✅ |

#### Required Scopes

 No scopes required

#### Signature

#### SDK Usage

```
discordSdk.close(RPCCloseCodes.CLOSE_NORMAL, "You exited from app");

```

---

### SDK Commands

 These commands let developers interact with the Discord client. Each SDK command below uses the `.commands` prefix, such as `discordSDK.commands.authenticate`.
| Name | Description |
| --- | --- |
| authenticate | Authenticates an existing client with your app |
| authorize | Authorizes a new client with your app |
| captureLog | Sends logs to your own logger |
| encourageHardwareAcceleration | Shows a modal dialog that lets the user enable hardware acceleration |
| getChannel | Gets channel information for the channel_id |
| getChannelPermissions | Gets the current user's permissions in the currently connected channel |
| getEntitlements | Gets the current user's entitlement list |
| getInstanceConnectedParticipants | Gets every participant connected to the instance |
| getPlatformBehaviors | Returns information about supported platform behaviors |
| getRelationships | Lets your app read a user's Discord Friends list, pending requests, and blocked users. Access is handled through the Social SDK, requires submission for access, and is subject to Social SDK Terms, including Section 5(a)(ii), for obtained data |
| getSkus | Gets your app's SKU list |
| initiateImageUpload | Opens the Discord client file upload flow |
| openExternalLink | Lets the app open an external link inside the Discord client |
| openInviteDialog | Shows a modal dialog with Channel Invite UI and does not require additional OAuth scopes |
| openShareMomentDialog | Shows a modal dialog for sharing media to a channel or DM |
| setActivity | Changes how the activity's rich presence appears in the Discord client |
| setConfig | Sets whether PIP (picture-in-picture) can be interactive |
| setOrientationLockState | Sets orientation and picture-in-picture (PIP) options |
| shareLink | Shows a modal so the user can share an activity link with custom query params |
| startPurchase | Starts the purchase flow for a specific SKU, using the sku_id |
| userSettingsGetLocale | Gets the current user's locale |

### authenticate()

 Authenticates an existing client with your app.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ✅ | ✅ |

#### Required Scopes

 No scopes required

#### Signature

#### Usage

```
await discordSdk.commands.authenticate({
  access_token: 'ACCESS_TOKEN_STRING'
});

```

---

### authorize()

 Authorizes a new client with your app.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ✅ | ✅ |

#### Required Scopes

 No scopes required

#### Signature

#### Usage

```
await discordSdk.commands.authorize({
  client_id: DISCORD_CLIENT_ID,
  response_type: "code",
  state: "",
  prompt: "none",
  scope: [
    // "applications.builds.upload",
    // "applications.builds.read",
    // "applications.store.update",
    // "applications.entitlements",
    // "bot",
    "identify",
    // "connections",
    // "email",
    // "gdm.join",
    "guilds",
    // "guilds.join",
    // "guilds.members.read",
    // "messages.read",
    // "relationships.read",
    // 'rpc.activities.write',
    // "rpc.notifications.read",
    // "rpc.voice.write",
    // "rpc.voice.read",
    // "webhook.incoming",
  ],
});

```

---

### captureLog()

 Sends logs to your own logger.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ✅ | ✅ |

#### Required Scopes

 No scopes required

#### Signature

#### Usage

```
await discordSdk.commands.captureLog({
  level: 'log',
  message: 'This is my log message!'
});

```

---

### encourageHardwareAcceleration()

 Shows a modal dialog that lets the user enable hardware acceleration.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ⛔️ | ⛔️ |

#### Required Scopes

 No scopes required

#### Signature

#### Usage

```
await discordSdk.commands.encourageHardwareAcceleration();

```

---

### getChannel()

 Gets information about the channel for the given channel ID.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ✅ | ✅ |

#### Required Scopes

- [guilds] for guild channels
- [guilds, dm_channels.read] is required for GDM channels. dm_channels.read requires Discord approval.

#### Signature

#### Usage

```
await discordSdk.commands.getChannel({
  channel_id: discordSdk.channelId,
});

```

---

### getChannelPermissions()

 Gets the current user's permissions in the currently connected channel.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ✅ | ✅ |

#### Required Scopes

- guilds.members.read

#### Signature

#### Usage

```
await discordSdk.commands.getChannelPermissions();

```

---

### getEntitlements()

 Gets the current user's entitlement list.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ✅ | ✅ |

#### Required Scopes

 No scopes required

#### Signature

#### Usage

```
await discordSdk.commands.getEntitlements();

```

---

### getInstanceConnectedParticipants()

 Gets every participant connected to the instance.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ✅ | ✅ |

#### Required Scopes

 No scopes required

#### Signature

#### Usage

```
await discordSdk.commands.getInstanceConnectedParticipants();

```

---

### getPlatformBehaviors()

 Returns information about supported platform behaviors.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ✅ | ✅ |

#### Required Scopes

 No scopes required

#### Signature

#### Usage

```
await discordSdk.commands.getPlatformBehaviors();

```

---

### getRelationships()

 Gets the current user's relationships.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ✅ | ✅ |

#### Required Scopes

- relationships.read
 This scope belongs to the Social SDK - [submit for access here](https://discord.com/developers/applications/select/social-sdk/getting-started). [Social SDK Terms apply](https://support-dev.discord.com/hc/en-us/articles/30225844245271-Discord-Social-SDK-Terms), including Section 5(a)(ii), to the data you obtain

#### Signature

#### Usage

```
await discordSdk.commands.getRelationships();

```

---

### getSkus()

 Gets SKU objects, with SKUs that have no prices filtered out automatically.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ✅ | ✅ |

#### Required Scopes

 No scopes required

#### Signature

#### Usage

```
await discordSdk.commands.getSkus();

```

---

### initiateImageUpload()

 Opens the file upload flow in the Discord client.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ✅ | ✅ |

#### Required Scopes

 No scopes required

#### Signature

#### Usage

```
await discordSdk.commands.initiateImageUpload();

```

---

### openExternalLink()

 Lets the app open an external link inside the Discord client.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ✅ | ✅ |

#### Required Scopes

 No scopes required

#### Signature

#### Usage

```
await discordSdk.commands.openExternalLink({
  url: 'string url'
});

```

---

### openInviteDialog()

 Shows a modal dialog with Channel Invite UI and does not require additional OAuth scopes.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ✅ | ✅ |

#### Required Scopes

 No scopes required

#### Signature

#### Usage

```
await discordSdk.commands.openInviteDialog();

```

---

### openShareMomentDialog()

 Shows a modal dialog for sharing media to a channel or direct message.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ⛔️ | ⛔️ |

#### Required Scopes

 No scopes required

#### Signature

#### Usage

```
await discordSdk.commands.openShareMomentDialog({
  mediaUrl: 'DISCORD_CDN_URL'
});

```

---

### setActivity()

 Changes how your Activity's Rich Presence data appears in the Discord client. The inner `activity` field is a partial [Activity object](https://docs.discord.com/developers/events/gateway-events#activity-object-activity-structure). For more details, read [Using Rich Presence with the Embedded App SDK](https://docs.discord.com/developers/rich-presence/using-with-the-embedded-app-sdk).

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ✅ | ✅ |

#### Required Scopes

- rpc.activities.write

#### Signature

#### Usage

```
await discordSdk.commands.setActivity({
  activity: {
    type: 0,
    details: 'Details',
    state: 'Playing'
  }
});

```

---

### setConfig()

 Sets whether PIP (picture-in-picture) can be interactive.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ⛔️ | ⛔️ |

#### Required Scopes

 No scopes required

#### Signature

#### Usage

```
await discordSdk.commands.setConfig({
  use_interactive_pip: true
})

```

---

### setOrientationLockState()

 Restricts the application to selected orientations for each supported layout mode.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ⛔️ | ✅ | ✅ |

#### Required Scopes

 No scopes required

#### Signature

#### Usage

```
import {Common} from '@discord/embedded-app-sdk';

await discordSdk.commands.setOrientationLockState({
  lock_state: Common.OrientationLockStateTypeObject.LANDSCAPE,
  picture_in_picture_lock_state: Common.OrientationLockStateTypeObject.LANDSCAPE,
  grid_lock_state: Common.OrientationLockStateTypeObject.UNLOCKED
});

```

---

### shareLink()

 Shows the user a modal for sharing a link

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ✅ | ✅ |

#### Required Scopes

 No scopes required

#### Signature

#### Usage

```
const { success } = await discordSdk.commands.shareLink({
  message: 'This message is shared alongside the link!',
  custom_id: 'some_custom_id',
});
success ? console.log('User shared link!') : console.log('User did not share link!');

```

---

### startPurchase()

 Starts the purchase flow for a specific SKU ID.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ⛔️ | ⛔️ |

#### Required Scopes

 No scopes required

#### Signature

#### Usage

```
await discordSdk.commands.startPurchase({sku_id: skuId});

```

---

### userSettingsGetLocale()

 Gets the current user's locale.

#### Supported Platforms

| Web | iOS | Android |
| --- | --- | --- |
| ✅ | ✅ | ✅ |

#### Required Scopes

- identify

#### Signature

#### Usage

```
await discordSdk.commands.userSettingsGetLocale();

```

---

### SDK Events

 Developers can pair these events with the `subscribe()` SDK method to receive events from Discord and supported devices.
| Name | Description |
| --- | --- |
| READY | non-subscription event emitted right after connection and carrying server information |
| ERROR | non-subscription event emitted for errors, including command responses |
| VOICE_STATE_UPDATE | emitted after subscribed voice-channel state changes for a user, such as mute or volume updates |
| SPEAKING_START | emitted when a user in a subscribed voice channel begins speaking |
| SPEAKING_STOP | emitted when a user in a subscribed voice channel stops speaking |
| ACTIVITY_LAYOUT_MODE_UPDATE | Received after a user changes the layout mode in the Discord client |
| ORIENTATION_UPDATE | Received when screen orientation changes |
| CURRENT_USER_UPDATE | Received after the current user object changes |
| CURRENT_GUILD_MEMBER_UPDATE | Received after the current guild member object changes |
| THERMAL_STATE_UPDATE | Received when Android or iOS thermal states are exposed to the Discord app |
| ACTIVITY_INSTANCE_PARTICIPANTS_UPDATE | Received after the instance participant count changes |
| RELATIONSHIP_UPDATE | Received after one of the current user's relationships changes |
| ENTITLEMENT_CREATE | Received after an entitlement is created for a SKU |

### READY

 Non-subscription event emitted right after connection and carrying server information.

#### Required Scopes

 No scopes required

#### Sample Event Payload

```
{
  "v": 1,
  "config": {
      "cdn_host": "cdn.discordapp.com",
      "api_endpoint": "//discord.com/api",
      "environment": "production"
  }
}

```

### ERROR

 Non-subscription event emitted for errors, including command responses.

#### Required Scopes

 No scopes required

#### Sample Event Payload

```
{
  "code": 4006,
  "message": "Not authenticated or invalid scope"
}

```

---

### VOICE_STATE_UPDATE

 Received after subscribed voice-channel state changes for a user, such as mute or volume updates.

#### Required Scopes

- rpc.voice.read

#### Sample Event Payload

```
{
  "voice_state": {
    "mute": false,
    "deaf": false,
    "self_mute": false,
    "self_deaf": false,
    "suppress": false
  },
  "user": {
    "id": "190320984123768832",
    "username": "test 2",
    "discriminator": "7479",
    "avatar": "b004ec1740a63ca06ae2e14c5cee11f3",
    "bot": false
  },
  "nick": "test user 2",
  "volume": 110,
  "mute": false,
  "pan": {
    "left": 1.0,
    "right": 1.0
  }
}

```

---

### SPEAKING_START

 Received after a user in a subscribed voice channel begins speaking.

#### Required Scopes

- rpc.voice.read

#### Sample Event Payload

```
{
  "channel_id": "7173758092142710784",
  "user_id": "7173758143913005056"
}

```

---

### SPEAKING_STOP

 Received after a user in a subscribed voice channel stops speaking.

#### Required Scopes

- rpc.voice.read

#### Sample Event Payload

```
{
  "channel_id": "7173758211307081728",
  "user_id": "7173758261412237312"
}

```

---

### ACTIVITY_LAYOUT_MODE_UPDATE

 Received after a user changes the layout mode in the Discord client.

#### Required Scopes

 No scopes required

#### Sample Event Payload

```
{
  "layout_mode": 1
}

```

---

### ORIENTATION_UPDATE

 Received when screen orientation changes.

#### Required Scopes

 No scopes required

#### Sample Event Payload

```
{
  "screen_orientation": 1
}

```

---

### CURRENT_USER_UPDATE

 Received after the current user object changes.

#### Required Scopes

- identify

#### Sample Event Payload

```
{
  "id": "7173771622812225536",
  "username": "beef_supreme",
  "discriminator": "0",
  "global_name": "Dis Cord",
  "avatar": "abcdefg",
  "avatar_decoration_data": {
    "asset": "abcdefg",
    "sku_id": "123456789"
  },
  "bot": false,
  "flags": 1,
  "premium_type": 2
}

```

---

### CURRENT_GUILD_MEMBER_UPDATE

 Received after the current guild member object changes.

#### Required Scopes

- identify
- guilds.members.read

#### Sample Event Payload

```
{
  "user_id": "7173771622812225536",
  "nick": "beef_supreme",
  "guild_id": "613425648685547541"
  "avatar": "abcdefg",
  "avatar_decoration_data": {
    "asset": "abcdefg",
    "sku_id": "123456789"
  },
  "color_string": "#ffff00"
}

```

---

### THERMAL_STATE_UPDATE

 Received when Android or iOS thermal states are exposed to the Discord mobile app.

#### Required Scopes

 No scopes required

#### Sample Event Payload

```
{
  thermal_state: 0
}

```

---

### ACTIVITY_INSTANCE_PARTICIPANTS_UPDATE

 Received after the instance participant count changes.

#### Required Scopes

 No scopes required

#### Sample Event Payload

```
{
  "participants": [
    {
      "id": "7173771622812225536",
      "username": "beef_supreme",
      "discriminator": "0",
      "global_name": "Dis Cord",
      "avatar": "abcdefg",
      "avatar_decoration_data": {
        "asset": "abcdefg",
        "sku_id": "123456789"
      },
      "bot": false,
      "flags": 1,
      "premium_type": 2
    }
  ]
}

```

---

### RELATIONSHIP_UPDATE

 Received after one of the current user's relationships changes.

#### Required Scopes

- relationships.read
 This scope belongs to the Social SDK - [submit for access here](https://discord.com/developers/applications/select/social-sdk/getting-started). [Social SDK Terms apply](https://support-dev.discord.com/hc/en-us/articles/30225844245271-Discord-Social-SDK-Terms), including Section 5(a)(ii), to the data you obtain

#### Sample Event Payload

```
{
  "type": 1,
  "user": {
      "id": "7173771622812225536",
      "username": "beef_supreme",
      "discriminator": "0",
      "global_name": "Dis Cord",
      "avatar": "abcdefg",
      "avatar_decoration_data": {
        "asset": "abcdefg",
        "sku_id": "123456789"
      },
      "bot": false,
      "flags": 1,
      "premium_type": 2
  }
}

```

---

### ENTITLEMENT_CREATE

 Coming soon! Not available during Developer Preview

### SDK Interfaces

#### Activity

| Property | Type |
| --- | --- |
| name | string |
| type | number |
| url? | string | null |
| created_at? | number | null |
| timestamps? | Timestamp | null |
| application_id? | string | null |
| details? | string | null |
| details_url? | string | null |
| state? | string | null |
| state_url? | string | null |
| emoji? | Emoji | null |
| party? | Party | null |
| assets? | Assets | null |
| secrets? | Secrets | null |
| instance? | boolean | null |
| flags? | number | null |

#### Assets

| Property | Type |
| --- | --- |
| large_image? | string | null |
| large_text? | string | null |
| large_url? | string | null |
| small_image? | string | null |
| small_text? | string | null |
| small_url? | string | null |

#### Application

| Property | Type |
| --- | --- |
| description | string |
| icon? | string | null |
| id | string |
| rpc_origins? | string[] |
| name | string |

#### Attachment

| Property | Type |
| --- | --- |
| id | string |
| filename | string |
| size | number |
| url | string |
| proxy_url | string |
| height? | number | null |
| width? | number | null |

#### AuthenticateRequest

| Property | Type |
| --- | --- |
| access_token? | string | null |

#### AuthenticateResponse

| Property | Type |
| --- | --- |
| access_token | string |
| user | User |
| scopes | string[] |
| expires | string |
| application | Application |

#### AuthorizeRequest

| Property | Type |
| --- | --- |
| client_id | string |
| scope | OAuthScopes[] |
| response_type? | ‘code’ |
| code_challenge? | string |
| state? | string |
| prompt? | ‘none’ |
| code_challenge_method? | ‘S256’ |

#### AuthorizeResponse

| Property | Type |
| --- | --- |
| code | string |

#### AvatarDecorationData

| Property | Type |
| --- | --- |
| asset | string |
| sku_id? | string | null |

#### CaptureLogRequest

| Property | Type |
| --- | --- |
| level | ConsoleLevel |
| message | string |

#### ChannelMention

| Property | Type |
| --- | --- |
| id | string |
| guild_id | string |
| type | number |
| name | string |

#### Embed

| Property | Type |
| --- | --- |
| title? | string | null |
| type? | string | null |
| description? | string | null |
| url? | string | null |
| timestamp? | string | null |
| color? | number | null |
| footer? | EmbedFooter | null |
| image? | Image | null |
| thumbnail? | Image | null |
| video? | Video | null |
| provider? | EmbedProvider | null |
| author? | EmbedAuthor | null |
| fields? | EmbedField[] | null |

#### EmbedAuthor

| Property | Type |
| --- | --- |
| name? | string | null |
| url? | string | null |
| icon_url? | string | null |
| proxy_icon_url? | string | null |

#### EmbedField

| Property | Type |
| --- | --- |
| name | string |
| value | string |
| inline | boolean |

#### EmbedFooter

| Property | Type |
| --- | --- |
| text | string |
| icon_url? | string | null |
| proxy_icon_url? | string | null |

#### EmbedProvider

| Property | Type |
| --- | --- |
| name? | string | null |
| url? | string | null |

#### Emoji

| Property | Type |
| --- | --- |
| id | string |
| name? | string | null |
| roles? | string[] | null |
| user? | User | null |
| require_colons? | boolean | null |
| managed? | boolean | null |
| animated? | boolean | null |
| available? | boolean | null |

#### EncourageHardwareAccelerationResponse

| Property | Type |
| --- | --- |
| enabled | boolean |

#### Entitlement

| Property | Type |
| --- | --- |
| id | string |
| sku_id | string |
| application_id | string |
| user_id | string |
| gift_code_flags | number |
| type | string | number |
| gifter_user_id? | string | null |
| branches? | string[] | null |
| starts_at? | string | null |
| ends_at? | string | null |
| parent_id? | string | null |
| consumed? | boolean | null |
| deleted? | boolean | null |
| gift_code_batch_id? | string | null |

#### GetChannelPermissionsResponse

| Property | Type |
| --- | --- |
| permissions | bigint | string |

#### GetChannelRequest

| Property | Type |
| --- | --- |
| channel_id | string |

#### GetChannelResponse

| Property | Type |
| --- | --- |
| id | string |
| type | ChannelTypesObject |
| guild_id? | string | null |
| name? | string | null |
| topic? | string | null |
| bitrate? | number | null |
| user_limit? | number | null |
| position? | number | null |
| voice_states | UserVoiceState[] |
| messages | Message[] |

#### GetEntitlementsResponse

| Property | Type |
| --- | --- |
| entitlements | Entitlement[] |

#### GetInstanceConnectedParticipantsResponse

| Property | Type |
| --- | --- |
| participants | User[] |

#### GetPlatformBehaviorsResponse

| Property | Type |
| --- | --- |
| iosKeyboardResizesView? | boolean |

#### GetRelationshipsResponse

| Property | Type |
| --- | --- |
| relationships | Relationship[] |

#### GetSkusResponse

| Property | Type |
| --- | --- |
| skus | Sku[] |

#### GuildMember

| Property | Type |
| --- | --- |
| user | User |
| nick? | string | null |
| roles | string[] |
| joined_at | string |
| deaf | boolean |
| mute | boolean |

#### GuildMemberRPC
