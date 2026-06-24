# Implementing Iap For Activities, Implementing One Time Purchases

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Includes: Key Concepts; Types of One-Time Purchases.

## Key Concepts

[Activities](https://docs.discord.com/developers/activities/overview) can use In-App Purchases (IAP) so users buy subscriptions or premium items directly inside Discord. This guide explains how to implement monetization with the [Embedded App SDK](https://docs.discord.com/developers/developer-tools/embedded-app-sdk). Before an Activity can add premium products through the Embedded App SDK, the app must have [Enable Monetization](https://docs.discord.com/developers/monetization/enabling-monetization) completed.

---

### Key Concepts

 Before adding monetization, know the Discord features involved:
- **SKUs**: The premium products offered by your app.
- **Entitlements**: The user's access record for premium products.
- **Subscriptions**: A continuing payment arrangement that keeps an entitlement active until the user cancels.
 The [Monetization Overview](https://docs.discord.com/developers/monetization/overview) provides more detail on these terms.

---

### Working with SKUs

 A SKU is the premium product your app sells. Create one SKU for every premium product before implementing monetization. For creation, management, and SKU type details, use the [Managing SKUs](https://docs.discord.com/developers/monetization/managing-skus) guide.

### Publishing SKUs for Activities

 When publishing SKUs, choose between **Store and the API** and **API Only**. Base that choice on the purchase experience users should see.

#### Published to Store and the API

- Users can see these SKUs in the Discord client, either through your [app's store](https://docs.discord.com/developers/monetization/managing-skus#viewing-your-store-page) or an Activity storefront you build.
- Users may buy them from the Discord client even while your Activity is closed, so design the purchase flow with that in mind.

#### Published to API Only

- These SKUs do **not** appear in the user's [app's store](https://docs.discord.com/developers/monetization/managing-skus#viewing-your-store-page); purchases happen only through your Activity storefront.
 The next section describes creating and rendering a custom storefront inside the Activity.

---

### Building a Storefront

 After the SKUs exist, build a storefront in the Activity that shows the premium products. This requires listing SKUs, associating them with perks, and displaying prices correctly.

### Listing SKUs

 For Activity product listings, call [`getSkus()`](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#getskus) through the [Embedded App SDK](https://docs.discord.com/developers/developer-tools/embedded-app-sdk). The [HTTP API](https://docs.discord.com/developers/resources/sku#list-skus) can list SKUs as well.

### Mapping SKUs to Your Premium Perks

 Buying a SKU creates an entitlement, and that entitlement represents access to that SKU. Map each SKU to the premium perk it unlocks so purchasers receive the correct benefit. Keep these rules in mind:
- Map SKUs to products by the `id` attribute, **not** by mutable fields such as `name`
- SKUs cannot be deleted; after purchase, access cannot be revoked by deleting the SKU, so every SKU should continue mapping to a perk or product

### Formatting and Displaying Prices

 SKU listing methods localize price and currency. `sku.price` includes numeric `amount` and string `currency` values. Correct currency rendering is difficult across many currencies, so the Embedded App SDK includes `PriceUtils` to help.

```
import {PriceUtils} from '@discord/embedded-app-sdk';

const displayPrice = PriceUtils.formatPrice(sku.price);
console.log(`The price is ${displayPrice}!`);

```

---

### Working with Entitlements

 A SKU purchase creates an entitlement. [Entitlements](https://docs.discord.com/developers/resources/entitlement) describe user access to a premium product. Depending on app design, entitlement state can come from the [Entitlement HTTP API](https://docs.discord.com/developers/resources/entitlement), [interaction payloads](https://docs.discord.com/developers/interactions/receiving-and-responding), [Gateway events](https://docs.discord.com/developers/events/gateway-events#entitlements), and [Embedded App SDK events](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#getentitlements).

### Fetching Entitlements with the Embedded App SDK

 Call [`getEntitlements()`](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#getentitlements) to retrieve the user's entitlement list. The response contains entitlement objects representing access to premium products.

### Handling Subscription Entitlements

 A subscription SKU purchase creates an entitlement. For subscription entitlement behavior, see [Implementing App Subscriptions](https://docs.discord.com/developers/monetization/implementing-app-subscriptions).

### Handling One-Time Purchase Entitlements

 A one-time purchase SKU also creates an entitlement. For handling those entitlements, see [Implementing One-Time Purchases](https://docs.discord.com/developers/monetization/implementing-one-time-purchases).

#### One-Time Purchases: Consumable and Durable Items

 Activities often sell consumable or single-use items, such as a potion or power-up. Buying a consumable SKU creates an entitlement with `consumed: false`. Process the purchase and [consume the entitlement](https://docs.discord.com/developers/resources/entitlement#consume-an-entitlement) promptly so the user receives the item. For unlimited-duration perks, use a durable SKU rather than a consumable SKU. The [Implementing One-Time Purchases](https://docs.discord.com/developers/monetization/implementing-one-time-purchases) guide covers durable and consumable items in more detail.

---

### Initiating Purchases

 After SKUs are displayed in the custom storefront, start a purchase when the user chooses one. In an Activity, call [`startPurchase()`](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#startpurchase) from the Embedded App SDK with the selected SKU `id`. Discord opens a purchase modal for that SKU. See the [Embedded App SDK Reference](https://docs.discord.com/developers/developer-tools/embedded-app-sdk) for details on [`startPurchase()`](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#startpurchase). To learn when checkout finished, subscribe to `ENTITLEMENT_CREATE` events.

### Subscribing to Purchase Events with the Embedded App SDK

 After a purchase completes, Discord emits an `ENTITLEMENT_CREATE` event. Subscribe to `ENTITLEMENT_CREATE` with [`subscribe()`](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#subscribe) in the Embedded App SDK so the Activity detects a successful SKU purchase and granted entitlement. Example for `ENTITLEMENT_CREATE`:

```
import {DiscordSDK} from '@discord/embedded-app-sdk';
const discordSdk = new DiscordSDK(clientId);
await discordSdk.ready();

const handleEntitlementCreate = () => {
  // refetch entitlements from server using the Entitlement HTTP API endpoint
};
discordSdk.subscribe('ENTITLEMENT_CREATE', handleEntitlementCreate);

```

 The same `ENTITLEMENT_CREATE` event can be consumed through the [Gateway API](https://docs.discord.com/developers/events/gateway-events#entitlements) for backend processing, or you can retrieve user entitlements with the [List Entitlements](https://docs.discord.com/developers/resources/entitlement#list-entitlements) HTTP API endpoint.

---

### Data Security Considerations

 When an Activity uses SKUs and Entitlements, protect monetization data. Validate information received through [Embedded App SDK](https://docs.discord.com/developers/developer-tools/embedded-app-sdk) commands and events. An attacker could open a separate RPC connection and impersonate Discord to the client. That may be harmless for many SDK surfaces, but it matters for In-App purchases and premium perks. If entitlement decisions rely only on SDK data, the attacker could unlock paid products, features, or advantages without a purchase. [`getEntitlements()`](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#getentitlements) is one command where this risk is relevant.

### Use the Discord HTTP API for Verification

 Data returned by the Discord HTTP API from your backend servers is trustworthy and should be the source of truth. Use it to check any client-side data that looks inconsistent.

### Recommended Approach

- Use client-side SDK commands and events **optimistically** for SKUs and Entitlements.
- **Verify** those results from your backend through the Discord HTTP API.
 In short, trust the SDK for responsiveness, but verify with the API before protecting premium features.

---

### Testing Your In-App Purchases

 Testing Activity In-App Purchases requires following the guidance for both [One-Time Purchases](https://docs.discord.com/developers/monetization/implementing-one-time-purchases#testing-your-onetime-purchase-implementation) and [Subscriptions](https://docs.discord.com/developers/monetization/implementing-app-subscriptions#testing-your-app-subscription-implementation). After purchase-flow testing, confirm that the app granted the premium perks associated with the purchased SKUs.

---

### Next Steps

 Review the In-App Purchase example in the [SDK Playground Example Application](https://github.com/discord/embedded-app-sdk-examples/tree/main/sdk-playground). It includes client and server code and follows the [security considerations](https://docs.discord.com/developers/monetization/implementing-iap-for-activities#data-security-considerations) for SKUs and entitlements.

### Embedded App SDK Playground

This sample demonstrates Embedded App SDK commands and events, including In-App Purchases.

## Types of One-Time Purchases

One-time purchases let an app charge users for premium functionality through in-app items.
- Before offering one-time purchases, the app must have [Enable Monetization](https://docs.discord.com/developers/monetization/enabling-monetization) completed.
- After app and team eligibility is confirmed, create a [SKU](https://docs.discord.com/developers/resources/sku), or stock-keeping unit, for the one-time purchase.

---

### Types of One-Time Purchases

 One-time purchase items can be durable or consumable:
- **Durable Items**: A permanent one-time purchase that is not renewed and cannot be consumed, such as lifetime access to premium app features.
- **Consumable Items**: A non-renewing one-time purchase that grants access, for example a temporary game boost or power-up.

---

### How One-Time Purchases Work

#### For Durable SKUs

- When a durable SKU is purchased, Discord creates an [Entitlement](https://docs.discord.com/developers/resources/entitlement) for that purchaser and that [SKU](https://docs.discord.com/developers/resources/sku).
- Discord sends an `ENTITLEMENT_CREATE` event through the Gateway.
- The entitlement becomes available through the `LIST Entitlements` API endpoint.
- `Interaction Payloads` from the entitled user include that entitlement.

#### For Consumable SKUs

- When a consumable SKU is purchased, Discord creates an [Entitlement](https://docs.discord.com/developers/resources/entitlement) for the purchaser and that SKU.
- Discord sends an `ENTITLEMENT_CREATE` event through the Gateway.
- The entitlement becomes available through the `LIST Entitlements` API endpoint.
- For guild subscriptions, `Interaction Payloads` from entitled guild users include the entitlement; user subscriptions include it for the entitled user.
- The SKU cannot be bought again until the entitlement is consumed through the [Consume Entitlement API](https://docs.discord.com/developers/resources/entitlement#consume-an-entitlement) endpoint.
  - In [Test Mode](https://docs.discord.com/developers/monetization/implementing-one-time-purchases#using-application-test-mode), developers may repeat purchases without consuming the entitlement.

- After receiving `ENTITLEMENT_CREATE` for a consumable SKU, process the item in the app and consume the entitlement promptly.

---

### Working with Entitlements

 A one-time purchase SKU purchase creates an entitlement. [Entitlements](https://docs.discord.com/developers/resources/entitlement) define access to the consumable or durable item. Depending on the app, track that access through [interaction payloads](https://docs.discord.com/developers/interactions/receiving-and-responding), [Gateway events](https://docs.discord.com/developers/events/gateway-events#entitlements), and the [Entitlement HTTP API](https://docs.discord.com/developers/resources/entitlement).

### Accessing Entitlements with Gateway Events

 When a SKU is purchased, Discord emits [`ENTITLEMENT_CREATE`](https://docs.discord.com/developers/events/gateway-events#entitlements). The event includes the entitlement object for access to the SKU, allowing near-time entitlement tracking. For One-Time Purchases, `ENTITLEMENT_DELETE` may also arrive if access is revoked.

### Accessing Entitlements with the HTTP API

 Use [List Entitlements](https://docs.discord.com/developers/resources/entitlement#list-entitlements) to retrieve entitlements. Filter by user or SKU set with `?user_id=XYZ` or `?sku_ids=XYZ`.

### Accessing Entitlements on Interaction Payloads

 The `Interaction Payload` also carries entitlements when a user interacts with the app. Read them from the `entitlements` field on the `Interaction Payload` while [receiving and responding to interactions](https://docs.discord.com/developers/interactions/receiving-and-responding).

### Accessing Entitlements with the Embedded App SDK

 An [Activity](https://docs.discord.com/developers/activities/overview) built with the [Embedded App SDK](https://docs.discord.com/developers/developer-tools/embedded-app-sdk) can also [access a user's entitlements](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#getentitlements). For SDK-based monetization details, see [Implementing In-App Purchases for Activities](https://docs.discord.com/developers/monetization/implementing-iap-for-activities). Apps may combine these entitlement sources as needed.

---

### One-Time Purchase Considerations

 Consider these points when implementing one-time purchases:

### For Durable One-Time Purchases

 Durable items grant SKU access indefinitely. Durable items are not consumed, so access remains unless a refund removes it.

### For Consumable One-Time Purchases

 For consumables, a user may hold only one entitlement that has not been consumed. Process and store the item in the app or game, then call [Consume Entitlement](https://docs.discord.com/developers/resources/entitlement#consume-an-entitlement) so the user can buy that item later. After consumption, the entitlement's `consumed` field returns true. Decide how the app records consumable item state and quantity before choosing when to consume entitlements.

---

### Prompting Users to Purchase an Item

### Responding with a Premium Button

 [Responding with a premium button](https://docs.discord.com/developers/monetization/managing-skus#responding-with-a-premium-button) lets an app prompt a subscription offer when a user tries to use a premium feature. Send a message containing a [button](https://docs.discord.com/developers/components/reference#button) using [premium style](https://docs.discord.com/developers/components/reference#button-button-styles) plus a `sku_id` for the upgrade offer.

### Starting a Purchase from an Activity

 An [Activity](https://docs.discord.com/developers/activities/overview) that uses the [Embedded App SDK](https://docs.discord.com/developers/developer-tools/embedded-app-sdk) can launch checkout for a SKU through the SDK. See [Implementing In-App Purchases for Activities](https://docs.discord.com/developers/monetization/implementing-iap-for-activities) for monetization through the Embedded App SDK.

---

### Testing Your One-Time Purchase Implementation

 One-Time Purchases and App Subscriptions use different purchase-test paths. **Test Entitlements are not valid for One-Time Purchases.**

### Using Application Test Mode

 Application Test Mode allows free test purchases of One-Time Purchase SKUs for the app. You can test consumable and durable item purchases through the In-App Purchase flow without card charges, although a valid payment method still must be on file. It is not charged at checkout. To enable the mode, first confirm a payment method exists in `User Settings -> Billing`, then:
1. Open up the Discord app
2. Select the Settings cog in the lower-left corner
3. Open `Advanced` under App Settings
4. Turn **on** both Developer Mode and Application Test Mode, then enter the application ID. Leave other settings unchanged.
5. Exit user settings
 After Application Test Mode is enabled, the app name appears in an orange bar at the top of the screen. Use the Store page to buy one-time purchase items without being charged. Entitlements from Application Test Mode purchases have `type` value 4, representing `TEST_MODE_PURCHASE`. The Go To SKU button is currently unavailable; purchase the SKU from the Store page while testing.
