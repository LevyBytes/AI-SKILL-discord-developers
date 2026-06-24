# Assets And Metadata, Growth And Referrals, Layout

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

This reference groups Activity guidance for local builds, metadata setup, link-sharing prompts, and orientation behavior.

## Local Development

These guides outline development practices, SDK commands, and user flows to consider while building an Activity. They help keep the user experience consistent and easy to understand as users interact with your application.

### Local Development

### Run Your Application Locally

Run a local development application.

### Launch Your App from Discord

Launch your app from Discord on mobile and desktop clients.

### Setup URL Mapping

Configure the Discord proxy so network requests can reach required external endpoints.

### Logging

Use different logging levels while building your application.

### User Actions

### Open External Link

Open an external link from inside your app.

### Open Invite Dialog

Open the Application Channel Invite dialog in Discord.

### Open Share Moment Dialog

Open a dialog that shares media from your application to a channel, DM, or GDM.

### Setting Up an Entry Point Command

Configure a command that lets users start your Activity from the App Launcher.

### Encourage Hardware Acceleration

Open a dialog that helps users enable hardware acceleration for compute-intensive applications.

### Mobile

### Supporting Mobile Platforms

Update application settings so iOS and Android are supported.

### Mobile Safe Areas

Keep your app's assets inside mobile-safe areas.

### Mobile Thermal States

Handle thermal state changes reported by iOS and Android.

### Layout

### Application Orientation

Configure application orientation and subscribe to orientation changes.

### Application Layout Mode

Subscribe to layout mode updates so your application's user interface can adapt.

### Networking

### Activity Proxy Considerations

Working with our Activity Proxy

### Construct a Full URL

Build a full URL for network request work.

### Using External Resources

Permit network requests to external resources through the Discord proxy.

### Security Considerations

Protect your Activity with security-focused implementation choices.

### Multiplayer Experience

### Instance Management

Manage instances so users can join the same instance as their friends.

### Instance Participants

Fetch users currently connected to an instance with the SDK.

### Render Avatars and Names

Retrieve and render usernames and avatars for users connected to your application.

### Preventing Unwanted Activity Sessions

Validate that activity sessions come through a Discord client before adding them to an instance's session.

### Growth and Referrals

### Prompting Users to Share Incentivized Links

Encourage users to share Activity links by adding tracking and rewarding engagement.

### Creating and Managing Custom Incentivized Links

Use these for off-platform rewards, promotions, or limited time experiences.

### Generating a Custom Link Within Your Activity

API support for generating ephemeral links with customizable embeds.

### Assets & Metadata

### Setting Up Activity Metadata

Best practices for how your application appears in Discord.

### Setting Up Art Assets

Best practices for configuring your application's Discord presentation.

### Rich Presence

### Integrating Rich Presence

Show live Activity data on user profiles while users play your Activity.

### Rich Presence Best Practices

Design guidance and a launch checklist for a polished Rich Presence integration.

### Production Readiness

### Cache Busting

Handle asset caching for your application and the Discord Activity proxy.

### Handling Rate Limits

Keep your Activity within rate limits.

### Static IP Addresses

Production network routing guidance for Activities that need static IP addresses.

### Backward Compatibility

Keep your application forward-compatible and support new SDK commands as they are added.

## Setting Up Activity Metadata

### Setting Up Activity Metadata

 The Activity Shelf lists playable Activities and can display configurable metadata and art assets. To update app metadata in the Discord Developer Portal, open your app's `Settings -> General Information` tab.
- **Application Name:** The public name shown for your app.
- **Application Icon:** The public icon shown for your app.
- **Application Description:** The description displayed in the Activity Shelf Item view.
- **Max Participants:** Indicates the maximum number of players your application supports.
  - Max Participants appears above the name in the 1-up view as `Up to X participants`.
  - If this field is empty, Discord shows `Unlimited participants`.
  - Max Participants appears below the name in the 2-up view too.

 An app's application name and avatar can differ from its bot username and avatar. Both metadata sets are public and may appear in different user interactions with your app. The bot username is available on the `Settings -> Bot` tab.

---

### Setting Up Activity Art Assets

 The Activity Shelf lists playable Activities and supports configurable metadata and art assets. To update embedded-specific art assets for your app in the Discord Developer Portal, open [Activities -> Art Assets](https://discord.com/developers/applications/select/embedded/assets).

### Embedded Background

 Used as a Grid view background overlay. Keep artwork clustered near the image edges, leaving the center clear so it does not conflict with the UI.

#### Specifications

- 16:9 aspect ratio
- At least 1024 pixels wide

### Cover Art

 Used as the primary Activity Shelf image. The recommended composition includes the title with background artwork.

#### Specifications:

- The image may be displayed at both 16:9 and 13:11 aspect ratios
- At least 1024 pixels wide

### App Tile

 Application tiles have two presentations: the regular 2-up tile and the larger "featured" 1-up tile.

### Video Preview

 Hovering over the cover image should play a preview video for the Application. Preview videos should be no longer than 10 seconds. If no video is supplied, hover does nothing.

#### Specifications: 640 x 360, mp4 format, under 10 seconds long, under 1 MB in size

## Prompting Users to Share Incentivized Links

### Prompting Users to Share Incentivized Links

 Incentivized sharing can grow your Activity through network effects. Links can support several patterns:
- **Referral links.** Users can copy referral links inside your Activity, including their Discord user ID (`https://discord.com/activities/<your Activity ID>?referrer_id=123456789`), and send them to friends. If a friend accepts and starts playing your game, grant the referrer an in-game reward.
- **Promotions.** Run a temporary promotion on social media with a reward for starting now. Share a custom link (`https://discord.com/activities/<your Activity ID>?custom_id=social012025` ) on social media; anyone who clicks that link can receive an in-game reward.
- **Social deep-links.** Instead of sending every Activity launch to the same location, use contextual deep-links. For example, user A can copy a link inside your Activity (`https://discord.com/activities/<your Activity ID>?referrer_id=123456789&custom_id=visit-location`) and send it in a DM or channel, so user B opens directly at user A's location.
- **Turn-based deep-links.** A DM that tells a user "it's your turn" can include a link that opens the correct game instance and turn.
- **Affiliate marketing.** Affiliates such as influencers or companies can promote your game and receive rewards through a custom link (`https://discord.com/activities/<your Activity ID>?custom_id=influencer1`). For each user who starts playing through that influencer, pay out the agreed reward.
- **Source attribution.** Use the `custom_id` parameter to measure traffic from different marketing sources.
 This guide implements a referral link with rewards for both users who share links and users who click them.

#### Implementation Overview

1. Build a tracked promotional referral link, then show users a prompt for sharing it
2. Process incoming referrals and grant valid rewards

#### Sharing Links

 When adding sharing, you need to:
1. Generate a unique ID for tracking the promotion
2. Call the [`shareLink`](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#sharelink) command
3. Track the share attempt

```
// Generate a unique ID for this promotion
// This could be per-campaign, per-user, or per-share depending on your needs
const customId = await createPromotionalCustomId();

try {
    const { success } = await discordSdk.commands.shareLink({
        message: 'Click this link to redeem 5 free coins!',
        custom_id: customId,
    });

    if (success) {
        // Track successful share for analytics/limiting
        await trackSuccessfulShare(customId);
    }
} catch (error) {
    // Handle share failures appropriately
    console.error('Failed to share link:', error);
}

```

#### Handling Incoming Referrals

 When a user opens a shared link, the activity launches with referral data available through the SDK:

```
// Early in your activity's initialization
async function handleReferral() {
    // Validate the referral data
    if (!discordSdk.customId || !discordSdk.referrerId) {
        return;
    }

    try {
        // Verify this is a valid promotion and hasn't expired
        const promotion = await validatePromotion(discordSdk.customId);
        if (!promotion) {
            console.log('Invalid or expired promotion');
            return;
        }

        // Prevent self-referrals
        if (discordSdk.referrerId === currentUserId) {
            console.log('Self-referrals not allowed');
            return;
        }

        // Grant rewards to both users
        await grantRewards({
            promotionId: discordSdk.customId,
            referrerId: discordSdk.referrerId,
            newUserId: currentUserId
        });
    } catch (error) {
        console.error('Failed to process referral:', error);
    }
}

```

#### Link Sharing Best Practices

- Generate unique, non-guessable `customId`s
- Track and validate referrals to reduce abuse
- Handle edge cases like expired promotions gracefully
- Consider cool-down periods between shares
- Do not write the `referrer_id` query parameter directly. When present, `referrer_id` should be a Discord snowflake-type user ID; otherwise it is set to the message's author id.

---

### Creating and Managing Custom Incentivized Links

 This guide explains how to create a customizable [Incentivized Link](https://docs.discord.com/developers/activities/development-guides/growth-and-referrals#prompting-users-to-share-incentivized-links) in the dev portal and retrieve it for off-platform sharing. Incentivized Links control how the embed appears to users.

#### Creating a Link

1. In your Application's portal, open the [Custom Links page](https://discord.com/developers/applications/select/embedded/custom-links) under Activities in the navigation pane.
2. On the Custom Links page, select `Create New` to create a link.
3. Upload an image with a 43:24 aspect ratio.
4. Title and description are required.
5. `custom_id` is optional; an explicit `custom_id` query parameter on the link always overrides the configured `custom_id`.
6. Click Save.

#### Editing a Link

1. Select a row to open the modal with its data loaded for editing.
2. Replace the description with different text.
3. Click Update.

#### Copying a Link

 When the edits are ready, select the copy icon on the row. It turns green after copying the link to your clipboard. You can share the link anywhere. The link uses this format: `https://discord.com/activities/<your Activity ID>?link_id=0-123456789`. If you configured a `custom_id`, it is not shown directly in the link, but it loads when a user clicks the link. You may shorten the URL afterward.

#### Deleting a Link

1. Select the trash icon on the row for the link you want to delete.
2. A confirmation dialog appears.
 Deletion happens immediately and cannot be undone. Before deleting, confirm that the link is not actively used and that your activity handles any later click-throughs gracefully.

#### Best Practices

- Generate unique, non-guessable `customId`s
- Track and validate referrals to reduce abuse
- Gracefully handle expirations in your activity for custom links that are time-limited but remain live off-platform.

#### User Experience

 (Figure: custom-link-embed) Users see an embed with your information. Selecting "Play" opens the activity and passes through the configured `custom_id`. A `referrer_id` is present for links shared on Discord.

---

### Generating a Custom Link Within Your Activity

 This guide covers creating a customizable [Incentivized Link](https://docs.discord.com/developers/activities/development-guides/growth-and-referrals#prompting-users-to-share-incentivized-links) inside your activity and sharing it with the `shareLink` API.
- Lets you customize how the embed presents the link to users
- Can be generated on demand inside your activity
- Ephemeral, 30 day TTL
- Does not appear in the developer portal

#### Generating a Link

```
// Convert an image array buffer to base64 string
const image = base64EncodedImage;

// Generate the quick activity link
const linkIdResponse = await fetch(`${env.discordAPI}/applications/${env.applicationId}/quick-links/`, {
  method: 'POST',
  headers: {
    Authorization: `Bearer ${accessToken}`,
    'Content-Type': 'application/json',
  },
  body: {
    custom_id: 'user_123/game_456',
    description: 'I just beat level 10 with a perfect score',
    title: 'Check out my high score!',
    image,
  }
});
const {link_id} = await linkIdResponse.json();

// Open the Share modal with the generated link
const {success} = await discordSdk.commands.shareLink({
  message: 'Check out my high score!',
  link_id,
});
success ? console.log('User shared link!') : console.log('User did not share link!');

```

## Application Orientation

### Application Orientation

#### Locking Application Orientation

 These SDK APIs lock the application to selected orientations. Supported lock states are `UNLOCKED`, `PORTRAIT`, and `LANDSCAPE`. `lock_state` is the default and controls orientation while the application is focused. `picture_in_picture_lock_state` controls the PIP aspect ratio, and `grid_lock_state` controls the grid tile aspect ratio. If `picture_in_picture_lock_state` is unset, the application PIP uses `lock_state` as its fallback. If `grid_lock_state` is unset, the grid tile uses `picture_in_picture_lock_state`; if that is unset too, it uses `lock_state`. Calling `setOrientationLockState` with `undefined` or with an omitted value for `picture_in_picture_lock_state` or `grid_lock_state` leaves the corresponding lock state unchanged. A later `setOrientationLockState` call can still set `picture_in_picture_lock_state` explicitly. Passing null for `picture_in_picture_lock_state` or `grid_lock_state` clears that layout mode's lock state so it follows the fallback chain.

```
import {DiscordSDK, Common} from '@discord/embedded-app-sdk';
const discordSdk = new DiscordSDK(clientId);
await discordSdk.ready();

// Set a default lock state
discordSdk.commands.setOrientationLockState({lock_state: Common.OrientationLockStateTypeObject.LANDSCAPE});

// or set both a default lock state and a picture-in-picture lock state
discordSdk.commands.setOrientationLockState({
  lock_state: Common.OrientationLockStateTypeObject.PORTRAIT,
  picture_in_picture_lock_state: Common.OrientationLockStateTypeObject.LANDSCAPE,
  grid_lock_state: Common.OrientationLockStateTypeObject.LANDSCAPE,
});

```

#### Configuring Default Orientation Lock State Through the Developer Portal

 You can configure a default orientation lock state for an application in the Developer Portal. With this setting, the Discord app applies the orientation lock while launching the application, before the SDK initializes. That can make startup smoother because the app opens in the correct orientation instead of switching after an SDK orientation-lock request. The Developer Portal supports different default orientation lock states for phones and tablets. (Figure: default-orientation-lock-state)

#### Subscribing to Screen Orientation Updates

 Subscribe to the `ORIENTATION_UPDATE` event to receive screen orientation, which can differ from physical device orientation. Discord sends the current orientation when the subscription is created and sends later orientation changes.

```
const handleOrientationUpdate = (update: {screen_orientation: number}) => {
  switch (update.screen_orientation) {
      case Common.OrientationTypeObject.PORTRAIT:
        ...
      case Common.OrientationTypeObject.LANDSCAPE:
        ...
      default:
        ...
    }
}

discordSdk.subscribe('ORIENTATION_UPDATE', handleOrientationUpdate);

```

---

### Application Layout Mode

 An application can use three layout modes: focused, picture-in-picture (PIP), or grid mode. Activities can subscribe to layout mode updates and adjust their layouts for each mode. Older Discord clients support only `ACTIVITY_PIP_MODE_UPDATE`, while newer clients support both `ACTIVITY_PIP_MODE_UPDATE` and `ACTIVITY_LAYOUT_MODE_UPDATE`. Use `subscribeToLayoutModeUpdatesCompat` and `unsubscribeFromLayoutModeUpdatesCompat` to subscribe to both events while staying compatible with older clients that only support `ACTIVITY_PIP_MODE_UPDATE`. Example using React:

```
export default function LayoutMode() {
  const handleLayoutModeUpdate = React.useCallback((update: {layout_mode: number}) => {
       ...
  }, []);

  React.useEffect(() => {
    discordSdk.subscribeToLayoutModeUpdatesCompat(handleLayoutModeUpdate);
    return () => {
      discordSdk.unsubscribeFromLayoutModeUpdatesCompat(handleLayoutModeUpdate);
    };
  }, [handleLayoutModeUpdate]);
}

```
