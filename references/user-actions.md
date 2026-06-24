# User Actions

> Discord Developers reference. Original prose; identifiers preserved verbatim.

### Open External Link

 Because Activities run in a sandbox, an app must call the command flow before a user can open a link outside Discord. Discord shows an in-client confirmation prompt before navigating to that external URL.

#### Usage

```
import {DiscordSDK} from '@discord/embedded-app-sdk';
const discordSdk = new DiscordSDK(clientId);
await discordSdk.ready();
// Once the sdk has established the connection with the discord client, external
// links can be launched
discordSdk.commands.openExternalLink({
  url: 'https://google.com',
});

```

#### User Experience

 (Figure: external-link-modal) Discord displays a modal that asks the user whether to continue. Choosing ***Trust this Domain*** suppresses future prompts for that same domain.

---

### Open Invite Dialog

 An Application Channel Invite can be fetched as described in [these docs](https://docs.discord.com/developers/resources/invite#get-invite), but no OAuth scope grants that ability directly. The SDK still provides `openInviteDialog`, which launches the Application Channel Invite UI in the Discord client without extra OAuth scopes. Calling it from DM (Direct Message) contexts returns an error, so restrict usage to Guild Voice or Text Channels. It also errors when the user lacks valid channel permissions; checking with `getChannelPermissions` first is strongly recommended, and that check requires OAuth scope `'guilds.members.read'`.

#### Usage

```
import {DiscordSDK, Permissions, PermissionUtils} from '@discord/embedded-app-sdk';
const discordSdk = new DiscordSDK(clientId);
await discordSdk.ready();

try {
  const {permissions} = await discordSdk.commands.getChannelPermissions();
  if (PermissionUtils.can(Permissions.CREATE_INSTANT_INVITE, permissions)) {
    await discordSdk.commands.openInviteDialog();
    // successfully opened dialog
  } else {
    console.warn('User does not have CREATE_INSTANT_INVITE permissions');
  }
} catch (err) {
  // failed to fetch permissions or open dialog
  console.warn(err.message);
}

```

 User Experience (Figure: Invite Dialog UI) The Discord client shows a modal where the user can send an invite to a channel or friend, or copy an invite link for manual sharing.

---

### Open Share Moment Dialog

 Use `openShareMomentDialog` when an application needs to share media into a channel or DM with the least custom UI. The command takes a Discord CDN `mediaUrl` such as `https://cdn.discordapp.com/attachments/...` and opens a Discord client dialog where the user can pick channels, DMs, or GDMs. No additional OAuth scopes are needed, but the application must already be authenticated. Since `mediaUrl` has to point at Discord CDN, the recommended path is the activities attachment API endpoint, `discord.com/api/applications/${applicationId}/attachment`, which creates an ephemeral CDN URL. That endpoint accepts bearer tokens for any scopes, so the application client may call it with the authorized user's bearer token. Its response is a serialized attachment containing a `url` attribute; pass that value to the DiscordSDK command as `mediaUrl`.

#### Usage

```
import {discordSdk} from './wherever-you-initialize-your-sdk';
import {accessToken} from './wherever-you-store-your-access-token';

// some image
const imageURL = 'https://i.imgur.com/vaSWuKr.gif';

// get image data
const response = await fetch(imageURL);
const blob = await response.blob();
const mimeType = blob.type;

// image data as buffer
const buf = await blob.arrayBuffer();

// image as file
const imageFile = new File([buf], 'example.gif', {type: mimeType});

const body = new FormData();
body.append('file', imageFile);

const attachmentResponse = await fetch(`${env.discordAPI}/applications/${env.applicationId}/attachment`, {
  method: 'POST',
  headers: {
    Authorization: `Bearer ${accessToken}`,
  },
  body,
});
const attachmentJson = await attachmentResponse.json();

// mediaUrl is an ephemeral Discord CDN URL
const mediaUrl = attachmentJson.attachment.url;

// opens dialog in Discord client
await discordSdk.commands.openShareMomentDialog({mediaUrl});

```

 User Experience (Figure: Example of the sharing dialog)

---

### Setting Up an Entry Point Command

 Users need an [Entry Point command](https://docs.discord.com/developers/interactions/application-commands#entry-point-commands) before they can start your Activity from Discord's [App Launcher menu](https://support.discord.com/hc/articles/21334461140375-Using-Apps-on-Discord#h_01HRQSA6C8TRHS722P1H3HW1TV). Enabling Activities in your [app's settings](http://discord.com/developers/applications) automatically creates a [default Entry Point command](https://docs.discord.com/developers/interactions/application-commands#default-entry-point-command). That default uses the `DISCORD_LAUNCH_ACTIVITY` (`2`) [handler type](https://docs.discord.com/developers/interactions/application-commands#application-command-object-entry-point-command-handler-types), so Discord launches the Activity for the user and posts a follow-up message in the launch channel. To send messages yourself, change the handler to `APP_HANDLER` (`1`). Handler behavior is covered in the [Entry Point command documentation](https://docs.discord.com/developers/interactions/application-commands#entry-point-handlers).

#### Customizing the Default Entry Point Command

 Entry Point commands use the same customization model as other [commands](https://docs.discord.com/developers/interactions/application-commands). They can only be [global](https://docs.discord.com/developers/interactions/application-commands#making-a-global-command), so use the global command HTTP endpoints:
- **Change the name or details of the current Entry Point command** with the [Edit Global Application Command](https://docs.discord.com/developers/interactions/application-commands#edit-global-application-command) endpoint. If the Entry Point command ID is unknown, retrieve it with [Get Global Application Commands](https://docs.discord.com/developers/interactions/application-commands#get-global-application-commands).
- **Promote another command without options to be the Entry Point command** by setting its [command `type`](https://docs.discord.com/developers/interactions/application-commands#application-command-object-application-command-types) to `PRIMARY_ENTRY_POINT` (type `4`). An app may have only one Entry Point command, so remove the existing one with [delete](https://docs.discord.com/developers/interactions/application-commands#delete-global-application-command) or change its [command `type`](https://docs.discord.com/developers/interactions/application-commands#application-command-object-application-command-types) through [update](https://docs.discord.com/developers/interactions/application-commands#edit-global-application-command) first.

#### Creating an Entry Point Command

 To add an Entry Point command from scratch, call [Create Global Application Command](https://docs.discord.com/developers/interactions/application-commands#create-global-application-command) and set the [command `type`](https://docs.discord.com/developers/interactions/application-commands#application-command-object-application-command-types) to `PRIMARY_ENTRY_POINT` (type `4`). A payload might be shaped like this:

```
{
  "name": "launch",
  "description": "Launch Realms of Wumpus",
  // PRIMARY_ENTRY_POINT is type 4
  "type": 4,
  // DISCORD_LAUNCH_ACTIVITY is handler value 2
  "handler": 2,
  // integration_types and contexts define where your command can be used (see below)
  "integration_types": [0, 1],
  "contexts": [0, 1, 2]
}

```

 Besides `type` and `handler`, the command payload contains `integration_types` and `contexts`, which control the places where the command is available and usable:
- `integration_types` specifies the [installation contexts](https://docs.discord.com/developers/resources/application#installation-context) where the command appears: server installs, user-account installs, or both. If command creation omits `integration_types`, Discord defaults it to the app's [currently-supported installation contexts](https://docs.discord.com/developers/resources/application#setting-supported-installation-contexts).
- `contexts` specifies the [interaction contexts](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-context-types) in which Discord permits the command to run. The choices cover guild context, a direct message with your app, and direct or Group DM conversations with other users.
 The [command contexts](https://docs.discord.com/developers/interactions/application-commands#contexts) documentation gives the complete details for both fields.

---

### Encourage Hardware Acceleration

 Compute-heavy Activities may perform better when hardware acceleration is enabled. When the app calls `encourageHardwareAcceleration`, Discord returns the setting's current state and, when relevant, prompts the user to change it. If Hardware Acceleration is disabled, the Discord client shows a modal recommending the change. Selecting **Don't show me this again** prevents that modal from appearing for *any application* on that device.

#### Best Practices

 Changing Hardware Acceleration makes the Discord client quit and restart, so call this command early enough that users have not already begun the app experience. A good placement is immediately after `await discordSdk.ready()`.

#### Usage

```
import {DiscordSDK} from '@discord/embedded-app-sdk';
const discordSdk = new DiscordSDK(clientId);
await discordSdk.ready();
const {enabled} = await discordSdk.commands.encourageHardwareAcceleration();
console.log(`Hardware Acceleration is ${enabled === true ? 'enabled' : 'disabled'}`);

```

#### User Experience

 (Figure: encourage-hardware-acceleration-modal)
