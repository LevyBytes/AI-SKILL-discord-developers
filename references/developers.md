# Rich Presence, Social Commerce

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: Understanding Rich Presence Data; Discord Game Shops.

## Understanding Rich Presence Data

For an [Activity](https://docs.discord.com/developers/activities/overview), the [Embedded App SDK](https://docs.discord.com/developers/developer-tools/embedded-app-sdk) simplifies adding Rich Presence so Discord can show what a user is doing inside a game or social experience. Treat Rich Presence data as an extension of the Activity: a modest amount of extra context can make it more useful to the current user and to friends who may want to join. This guide covers the platform concepts and technical details needed to add Rich Presence to an existing Activity. If you are unsure whether the Embedded App SDK is the right integration path, read [Choosing an SDK](https://docs.discord.com/developers/rich-presence/overview#choosing-an-sdk). The remaining guidance assumes you already have an [app](https://docs.discord.com/developers/quick-start/overview-of-apps) capable of launching an Activity. If not, complete [building your first Activity](https://docs.discord.com/developers/activities/building-an-activity) first.

### Understanding Rich Presence Data

### Default Rich Presence Data

 When a user connects to an Activity, the default profile presence shows the app icon. If the profile viewer is allowed to join, Discord also shows an "Ask to Join" button. (Figure: Example of default Rich Presence data for an Activity) This default state works, but it gives little detail about the user's current in-Activity context. The next sections show how richer and more actionable presence can look.

### Custom Rich Presence Data

 Here is the kind of custom presence Discord can show after a user joins an Activity. The [types for these fields](https://docs.discord.com/developers/rich-presence/using-with-the-embedded-app-sdk#setactivity-fields) and [examples](https://docs.discord.com/developers/rich-presence/using-with-the-embedded-app-sdk#setactivity-example) are covered later; this section is only to orient you to the visible result. (Figure: Image of where Rich Presence data appears in Discord profiles for Activities) Important details from that image:
1. `large_image` and `small_image` both belong to the `assets` object shown in the [table below](https://docs.discord.com/developers/rich-presence/using-with-the-embedded-app-sdk#activity-partial-object). The image labels use the same object keys so it is easier to match each field to its profile placement.
2. App Name is not controlled by presence data; Discord uses the name from your [app’s settings](https://discord.com/developers/applications).
3. The state badge `(1 of max_party)` appears only when a `party` field is present. Without `party`, Discord renders `state` as text below `details`.

### Updating Presence

 With the Embedded App SDK, custom Rich Presence updates mainly go through **[`setActivity()`](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#setactivity)**. Internally, `setActivity()` uses the RPC [`SET_ACTIVITY` command](https://docs.discord.com/developers/topics/rpc#setactivity) with the fields and capabilities available to Activities.

Why does the word activity appear in so many places?

A short note on the platform history behind the different activity terms

Rich Presence documentation uses the term "activity" in several overlapping ways. The "activities" in parts of the docs such as the [RPC ones](https://docs.discord.com/developers/topics/rpc#setactivity) are not the same thing as Activities built with the Embedded App SDK. When Rich Presence first launched, the object holding presence details was named an "activity" before the Embedded App SDK existed; that is why the RPC [`SET_ACTIVITY` command](https://docs.discord.com/developers/topics/rpc#setactivity) uses that terminology. The Embedded App SDK method `setActivity()` wraps that RPC command, even though it is not changing the lifecycle state of the Activity you are building. The naming can be confusing, but the distinction usually matters less than knowing which object and SDK method you are using.

### rpc.activities.write Scope

 To show custom Rich Presence for a user, authorize the app for that user with the [`rpc.activities.write` scope](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-scopes). A request for that scope in [`authorize()`](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#authorize) can look like this:

Example requesting rpc.activities.write with authorize()

This example is limited to the `authorize()` call. For setup details such as installing and instantiating the Embedded App SDK, use the [Building an Activity guide](https://docs.discord.com/developers/activities/building-an-activity).

```
// Authorize with Discord Client
const { code } = await discordSdk.commands.authorize({
  client_id: import.meta.env.VITE_DISCORD_CLIENT_ID,
  response_type: "code",
  state: "",
  prompt: "none",
  scope: [
    "identify",
    "rpc.activities.write"
  ],
});

```

### setActivity Fields

 A call to [`setActivity()`](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#setactivity) receives a partial [activity object](https://docs.discord.com/developers/events/gateway-events#activity-object-activity-structure). The table below lists many available fields for that partial object. Fields with no effect for Activities are omitted.

#### Activity partial object

 Every field in the partial object is optional and nullable.
| field | type | description |
| --- | --- | --- |
| type | integer | Activity type, used to choose the header text shown for Rich Presence data |
| state | string | User’s current party status |
| details | string | The user's current action or status inside the Activity |
| timestamps | timestamps object | Unix timestamps used to show start time, end time, or both |
| assets | assets object | Images displayed in Rich Presence data, plus their hover text |
| party | party object | Details about the player's current party |

### setActivity Example

 Next is a more complete example. The Rich Presence shown below represents an Activity. (Figure: Example of a fake game's Rich Presence data) The matching `setActivity()` code would be:

Example calling setActivity() command

This example focuses only on `setActivity()`. For details such as creating the Embedded App SDK instance and authenticating users, follow the [Building an Activity guide](https://docs.discord.com/developers/activities/building-an-activity).

```
await discordSdk.commands.setActivity({
  activity: {
    type: 0,
    details: 'Traveling with a group',
    state: 'In Mainframe',
    assets: {
      large_image: 'main-game-image',
      large_text: 'in a group',
      small_image: 'map-mainframe',
      small_text: 'in mainframe'
    },
    timestamps: {
      start: 1723137832
    },
    party: {
      size: [2,4]
    }
  }
});

```

### Uploading Custom Assets

 To add custom Rich Presence art, open your [app's settings](https://discord.com/developers/applications), then select **Rich Presence** in the left sidebar. The [**Art Assets** page](https://discord.com/developers/applications/select/rich-presence/assets) supports up to 300 custom assets per app. Discord recommends **1024 x 1024** for all Rich Presence assets. Uploaded asset keys are automatically converted to lowercase, so reference the lowercase key in the `assets` object passed to [`setActivity()`](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#setactivity); for example, `Main-Game-Image` becomes `main-game-image`. For asset-selection guidance, see the [Rich Presence best practices guide](https://docs.discord.com/developers/rich-presence/best-practices#have-interesting-expressive-art).

### Using External Custom Assets

 Activities usually need to account for the proxy and [how to use external resources](https://docs.discord.com/developers/activities/development-guides/networking#using-external-resources). Rich Presence image URL fields do not require that extra handling. If 300 uploaded custom assets is not enough, or the images live elsewhere, set an external URL as `large_image` or `small_image` inside the `assets` object.

Example setting presence with an external asset

This example covers only the `setActivity()` usage. For creating the Embedded App SDK instance and authenticating users, refer to the [Building an Activity guide](https://docs.discord.com/developers/activities/building-an-activity).

```
await discordSdk.commands.setActivity({
  activity: {
    type: 2,
    state: 'Broken Hearts and Code (club edit)',
    details: 'DJ Wump',
    assets: {
      large_image: 'https://example.com/album-covers/dj-wump/broken-code-and-hearts-club-edit.jpg',
      large_text: 'Listening to a track',
    }
  }
});

```

 Assets uploaded through the developer portal are limited to PNG, JPEG, and WebP; animated uploaded assets are not supported. External URLs can also point to GIF, animated WebP, and AVIF. See [Activity Asset Image](https://docs.discord.com/developers/events/gateway-events#activity-object-activity-asset-image) for more detail.

## Discord Game Shops

(Figure: Discord Social Commerce Hero Image) Discord Social Commerce embeds a game's store inside Discord, where players already spend time. Game Shops let players browse items, buy them, and gift them to friends without being redirected to an outside store.

### Discord Game Shops

 A Game Shop is hosted inside the Discord server and exposes items in several Discord surfaces, including player profiles, DMs, and voice chats. Discovery can happen through social activity rather than only through deliberate store browsing. After a purchase, Discord processes the payment and creates an [entitlement](https://docs.discord.com/developers/resources/entitlement) that represents the item owed to the player. The game's backend receives the fulfillment webhook, verifies that the Discord account is linked to the game account, and grants the item.

### Gifting & Wishlists

 Game Shops make it possible for non-players to buy for players, which ordinary in-game shops usually cannot do. Friends, relatives, and community members can open the Game Shop, choose a wishlist item, and send it as a gift without owning or launching the game. For developers, each player in the Discord community can become a surface where items are discovered, and the player's social circle becomes a potential buyer audience. Wishlists improve this loop by letting players mark desired items, so gift-givers can choose something relevant instead of guessing. Birthdays, launches, seasonal moments, and other gift-driven events on Discord can all create shop revenue opportunities.

### Social Commerce APIs

 Social Commerce uses APIs that let a developer control catalog data, purchases, and fulfillment:
- **SKU & Storefront Management:** Create and maintain the item catalog, including cosmetics, bundles, and consumables. Manage pricing, availability, and metadata through the API to keep the store aligned with the game.
- **Entitlements:** Read and manage player ownership. When Discord completes a purchase, it creates an entitlement that the backend can check and use.
- **Fulfillment:** Receive webhook events immediately after purchase completion. Deliver the item in-game, then use the API to mark the entitlement fulfilled.
 Together, these APIs support an automated path from purchase through fulfillment, with Discord providing the payment experience and the game's backend delivering the item.

### Get Access to Social Commerce

 Social Commerce is in early access. Developers who want a Game Shop on Discord can contact Discord and will be followed up with as access expands.

### Request Access to Social Commerce

Submit the interest form, and Discord will contact studios as onboarding opens.
