# Managing Skus, Overview

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: SKU creation and the parts that make up a Premium App.

## Creating a SKU

Premium items and subscriptions in an app are modeled as SKUs. **SKU** means Stock Keeping Unit, and each SKU uniquely identifies one premium offering. [SKUs](https://docs.discord.com/developers/resources/sku) form the base objects for premium offerings and are managed in the Developer Portal.

---

### Creating a SKU

 To add a SKU, open your [app's settings](https://discord.com/developers/applications), then go to [Monetization -> Manage SKUs](https://discord.com/developers/applications/select/skus). Select `Create SKU`; after `Create SKU` opens, the creation flow lets you choose one of these types:
- **User Subscription**: A renewing subscription whose benefits apply to one user across all servers
- **Guild Subscription**: A renewing subscription whose benefits apply to every user in one server
- **Consumable**: A one-time purchase that grants a temporary benefit and is used up when consumed.
- **Durable**: A one-time purchase that **grants** a permanent item, capability, or improvement.
 After choosing the SKU type, provide the SKU name to proceed.

### Creating Subscription Tiers

 Multiple subscription tiers can represent different benefit sets and prices. Each tier is backed by its own SKU. For upgrade and downgrade handling between tiers, use [Implementing App Subscriptions](https://docs.discord.com/developers/monetization/implementing-app-subscriptions#supporting-upgrades-and-downgrades). (Figure: Supporting multiple subscription tiers)

### SKU Limitations

 SKU creation has these limits:
- Each app can have at most 50 SKUs in total.
- An app may offer user subscription SKUs or guild subscription SKUs, but it cannot offer both at the same time.
- SKU pricing must come from Discord's available price options.
 When the 50 SKU cap is too small, consider a consumable in-app currency SKU that your app tracks and spends on items.

---

### Customizing Your SKUs

 After creating a SKU, adjust its presentation and benefits to fit the app. Customizable pieces include:
- Premium SKU name, up to 80 characters.
- Premium SKU description, up to 160 characters
- Premium SKU image
- Premium SKU price
 The benefit list appears on the app's Store page, in the App Directory, and in purchase and cancellation flows so users understand what the premium offer includes. Benefits may include:
- Up to 6 benefits
- An emoji, standard or custom
- A name, max 80 characters
- A description, max 160 characters
 (Figure: Example of SKU benefits)

### Pricing Your SKUs

 SKU prices are chosen from a predefined price list. Discord converts those prices into a user's local currency according to locale. Subscription SKUs renew monthly unless canceled. Changing a subscription SKU's price applies only to new subscribers; existing subscribers remain on the price active when they subscribed.

Using Unicode Emoji

For a standard Unicode emoji icon, put the emoji in `Unicode Emoji or Custom Emoji Name`. An emoji keyboard can help: MacOS uses `control + command + space bar`, and Windows uses `Windows + .`. (Figure: Set a unicode emoji)

Using a Custom Emoji

For a custom emoji, provide values in both fields:
- Name of your custom emoji
- ID of the custom emoji
To find an emoji ID in Discord, send the emoji escaped with a backslash `\`. For example, `\:uwu:` renders the emoji name and ID. (Figure: Set a custom emoji)

---

### Publishing and Unpublishing SKUs

 A newly created SKU starts in the `Unavailable` state and cannot yet be purchased. You can add pricing, benefits, and other details before publishing. In your [app's settings](https://discord.com/developers/applications) under [Monetization -> Manage SKUs](https://discord.com/developers/applications/select/skus), Discord provides several visibility and publishing controls:
- Publish SKU
- Unpublish SKU
- Delete SKU

### Publishing a SKU

 When you publish a SKU, choose either **Available via the Store and API** or **Available via the API Only**.

#### Publishing to Store & API

 Purchasable and visible from the app's store.

#### Publishing to API Only

 Only API calls or Embedded App SDK flows can grant entitlements for this SKU.

### Unpublishing a SKU

 **Danger:** Unpublishing a SKU can affect existing subscriptions and entitlements. It removes the SKU from both the Store and the API, so users cannot buy it. Effects of unpublishing include:
- Subscription SKUs stop renewing for subscribed users and guilds when their billing period ends.
- Users and guilds keep their SKU entitlement until the billing period finishes.
- Consumable and durable SKU entitlements remain for users who bought the SKU before it was unpublished.
- Does not delete a SKU.

### Deleting a SKU

 **Danger:** Deleting a SKU can affect existing subscriptions and entitlements. Deletion removes the SKU from the UI and prevents publishing, but deleted SKUs still appear in API responses from [List SKUs](https://docs.discord.com/developers/resources/sku#list-skus). Deletion has these effects:
- Subscription SKU purchasers, whether users or guilds, are unsubscribed immediately. Their entitlement remains valid through the billing period end.
- Consumable and durable SKU entitlements remain for users who bought the SKU before it was unpublished.

---

### Editing a Published SKU

 To modify a published SKU, first unpublish the active version. After unpublishing, it is no longer for sale, and existing subscribers do not renew at the end of the billing period. Keep providing the premium offering until each existing subscription expires.

### Changing a Subscription SKU Price

 Changing the price of a user or guild subscription SKU affects only new subscribers. Current subscribers continue paying the SKU price that was active when they subscribed.

---

### Integrating SKUs in Your App

 Once SKUs are published, implement the app's premium features. Start with the related guides.

### Implementing App Subscriptions

Learn to create and manage recurring subscriptions in an app.

### Implementing One-Time Purchases

Learn to add one-time purchase support to an app.

---

### Viewing your Store Page

 An app's Store page is reachable from the Bot User's server profile. Users can view available subscriptions and one-time purchases, open a subscription to inspect perks, benefits, and details, and buy directly from that Store page. Only items and subscriptions published to the Store are shown there.

#### Accessing your Store page from a Bot User’s Profile

 (Figure: Accessing the store as a user)

#### Subscriptions in Your Store page

 (Figure: Subscriptions in your Store View)

#### Items in Your Store page

 (Figure: Items in your Store View)

---

### Linking to a Specific SKU

 Use the Application Directory Store URL scheme to link to one SKU directly: `https://discord.com/application-directory/:appID/store/:skuID`
- In chat, this URL appears as a rich embed that can open a modal for SKU details or checkout
- As a browser URL, it opens the product page in the web Application Directory
 (Figure: Embed for direct link to SKU)

---

### Linking To Your Store

 Use the Application Directory Store URL scheme to link to the Store page directly: `https://discord.com/application-directory/:appID/store`
- In chat, this URL renders as a rich embed that can open a modal for the Store page
- As a browser URL, it opens your Store page in the web Application Directory
 (Figure: Embed for direct link to Store)

---

### Responding with a Premium Button

 Prompt purchases for item or subscription SKUs with a [button](https://docs.discord.com/developers/components/reference#button) that uses [premium style](https://docs.discord.com/developers/components/reference#button-button-styles) and `sku_id`. Premium buttons can be used wherever message components are valid, including command responses.

```
return new JsonResponse({
    type: 4, // InteractionResponseType.CHANNEL_MESSAGE_WITH_SOURCE
    data: {
        content: "This command requires Nelly Premium! Upgrade now to get access to these features!",
        components: [{
            type: MessageComponentTypes.ACTION_ROW,
            components: [
                {
                    type: MessageComponentTypes.BUTTON,
                    style: 6, // ButtonStyleTypes.PREMIUM
                    sku_id: '1234965026943668316',
                },
            ],
        }]
    },
});

```

 (Figure: A premium button)

## Components of a Premium App

(Figure: Monetizing Your Discord App) Use Discord's built-in checkout and payment flow to sell subscriptions and one-time purchases in an app. For a higher-level Premium Apps overview, see [Premium Apps & Activities](https://docs.discord.com/developers/platform/app-monetization). App Monetization applies to [Bots](https://docs.discord.com/developers/bots/overview) and [Activities](https://docs.discord.com/developers/activities/overview).

---

### Components of a Premium App

 A premium feature integration uses three main pieces from the Monetization API:
- [SKUs](https://docs.discord.com/developers/resources/sku) are the specific item or subscription offerings in the app. Each SKU identifies one distinct offer.
- [Entitlements](https://docs.discord.com/developers/resources/entitlement) show whether a user has access to a particular premium offering or SKU.
- [Subscriptions](https://docs.discord.com/developers/resources/subscription) model a continuing payment agreement for a recurring entitlement until cancellation.

### Types of SKUs

 Apps can create two SKU categories:

#### One-Time Purchase SKUs

 A one-time purchase SKU covers an item or feature bought once. Developers can offer two one-time purchase variants:
- Durable items: Items bought once and retained permanently. For example, a user could buy a `premium` upgrade that enables premium app features.
- Consumable items: Items bought once and then spent or depleted. For example, a user could buy a `boost` item that grants a temporary in-app boost.

#### Subscription SKUs

 A subscription SKU is a recurring purchase for a defined period. Developers can offer two subscription variants:
- User subscriptions: The purchasing user subscribes for themself, and only that user is entitled to the SKU.
- Guild subscriptions: The purchaser subscribes for a guild, and every member of that guild is entitled to the SKU.

---

### Next Steps

 To begin monetizing, follow [Enabling Monetization](https://docs.discord.com/developers/monetization/enabling-monetization) and the premium feature implementation guidance.
