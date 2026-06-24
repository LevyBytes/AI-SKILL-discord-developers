# Enabling Monetization, Implementing App Subscriptions

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Includes: Monetization enablement steps; Subscription types.

## Steps to Enable Monetization

Before monetization can be added, confirm that both the app and developer team satisfy the eligibility requirements.

### Steps to Enable Monetization

1. Configure the developer team and app so they can qualify for monetization
2. Satisfy the monetization eligibility criteria
3. Configure developer team payouts so revenue can be paid out
4. Create your premium offering
5. Implement monetization in your app
6. Start offering your premium features
 After those tasks are complete, use [create SKUs](https://docs.discord.com/developers/monetization/managing-skus#creating-a-sku) to model premium offerings, then follow [add support for your premium offering](https://docs.discord.com/developers/monetization/enabling-monetization#step-5-implement-monetization-in-your-app) while implementing them in the app.

### Step 1. Set Up Your Developer Team and App

 Monetization requires the following setup:
- A developer-portal [team](https://docs.discord.com/developers/topics/teams). If needed, [create one on the Teams page](https://discord.com/developers/teams)
- A [verified app](https://support-dev.discord.com/hc/en-us/articles/23926564536471-How-Do-I-Get-My-App-Verified) that the team owns
- Eligibility for monetization on both the app and team. Use the Eligibility Checklist below.

### Step 2. Complete the Eligibility Checklist

 The app and team must qualify for monetization before you create SKUs or offer paid features. When the team owner starts monetization setup, Discord walks them through checks for these criteria:

Eligibility Checklist

- App must be verified
- The app is owned by a developer team
- The team owner is 18 or older
- The team has verified emails and 2FA enabled
- The app uses slash commands, or it has approval for the privileged `Message Content` intent
- The app links to your Terms of Service
  - That document is the agreement between you and users for use of the app.

- The app links to your Privacy Policy
  - This document should clearly and accurately tell app users what user data you collect, how you use it, and how it is shared with third parties and us, while matching the Developer Terms of Service and the Developer Policy.

- The app name, description, commands, and role connection metadata must not include harmful or inappropriate language.
- Payouts are configured with a valid payment method
- Agreement to the [Monetization Terms](https://support.discord.com/hc/articles/5330075836311) and the [Discord Developer Policy](https://support-dev.discord.com/hc/en-us/articles/8563934450327-Discord-Developer-Policy).

### Step 3. Set Up Team Payouts

 Configure Team Payouts to receive revenue. Discord uses Stripe for payouts, so setup includes Stripe onboarding.
- Only the team owner can turn on payout settings for the team.
- The app becomes payout-eligible after earning its first $100.
- Discord reviews the setup; if approved, payouts begin for the team.

#### If You are Based in the United States, European Union, or United Kingdom

- On the Teams page, select [your team](https://discord.com/developers/teams).
- Select **Payout Settings**.
  - If **Payout Settings** is missing, you are not the team owner. Only the owner can enable team payout settings.

- Finish the Stripe onboarding flow.

#### If You are Based Outside of the United States, European Union, or United Kingdom

 Premium Apps is currently limited to these regions. Discord plans to expand availability. See the Help Center article [Premium Apps Payouts](https://support-dev.discord.com/hc/articles/17299902720919) for details.

### Step 4: Create Your Premium Offering

 You can now set up SKUs and add premium features. Use [Managing your Premium Offerings](https://docs.discord.com/developers/monetization/managing-skus#creating-a-sku) when creating app subscriptions and one-time purchases.

### Step 5: Implement Monetization in your App

 Once monetization is set up for the app, add code for premium features. Discord provides guides for these monetization patterns:

### Implementing App Subscriptions

Learn to begin and manage recurring subscriptions in the app.

### Implementing One-Time Purchases

Learn to add one-time purchase support to the app.

### Step 6: Start Offering Your Premium Features

 Monetization setup is complete. Users can purchase through a [link to your Store](https://docs.discord.com/developers/monetization/managing-skus#linking-to-your-store), a [link to a specific SKU](https://docs.discord.com/developers/monetization/managing-skus#linking-to-a-specific-sku), or a Message Components [premium styled button](https://docs.discord.com/developers/monetization/managing-skus#responding-with-a-premium-button).

## Types of Subscriptions

Use recurring user or guild subscriptions to charge for premium app features.
- Before subscriptions can be added, the app must have [Enable Monetization](https://docs.discord.com/developers/monetization/enabling-monetization) completed.
- After app and team eligibility is confirmed, create a [SKU](https://docs.discord.com/developers/resources/sku), or stock-keeping unit, for the subscription.

---

### Types of Subscriptions

 When creating subscriptions, choose either user subscriptions or guild subscriptions:
- **User Subscriptions**: Premium features apply to one user across any server where the app is installed.
- **Guild Subscriptions**: Premium benefits apply to every member of a specific server.

---

### How App Subscriptions Work

- When a subscription SKU is purchased, Discord creates an [Entitlement](https://docs.discord.com/developers/resources/entitlement) tied to the user (or guild) and that Subscription [SKU](https://docs.discord.com/developers/resources/sku).
- Discord sends an `ENTITLEMENT_CREATE` event through the Gateway.
- The entitlement appears through the `LIST Entitlements` API endpoint.
- `Interaction Payloads` from the entitled user include the entitlement; guild subscriptions include it on payloads from users in that guild.
- The subscription appears through the `LIST Subscriptions` API endpoint.
- The entitlement remains granted indefinitely until the user cancels; `ends_at` is null.
- Canceling a subscription does not cause the app to receive entitlement events.
- When the subscription ends, the subscription entitlement ends too. Developers receive `ENTITLEMENT_UPDATE` with an `ends_at` timestamp showing when it ended.

### Using Subscription Events for the Subscription Lifecycle

 Entitlements do not renew or update on cancellation, so subscription events can help follow the subscription lifecycle. This list is not exhaustive. Use entitlement presence to decide whether a user has premium access. The Subscription API and related events support reporting and lifecycle management and **are not the source of truth for premium-feature access**.
| Event Name | Subscription Behavior | Updated Fields |
| --- | --- | --- |
| `SUBSCRIPTION_CREATE` | Subscription is created | `status` is `0 (active)` when an entitlement has been granted, or `1 (inactive)` when it has not been granted yet |
| `SUBSCRIPTION_UPDATE` | Subscription is granted an entitlement | `status` is `0 (active)` |
| `SUBSCRIPTION_UPDATE` | Subscription is renewed | `current_period_start`, `current_period_end` timestamps updated |
| `SUBSCRIPTION_UPDATE` | Subscription is upgraded or downgraded | `sku_ids`, `entitlement_ids`, `renewal_sku_ids` may be updated |
| `SUBSCRIPTION_UPDATE` | Subscription is canceled | `canceled_at` timestamp updated, `status` is `2 (ending)` |
| `SUBSCRIPTION_UPDATE` | Subscription ends | `status` is `1 (inactive)`; this event is asynchronous and will not occur immediately |
| `SUBSCRIPTION_UPDATE` | Subscription is resumed or uncanceled by user | `status` is `0 (active)` |

---

### Working with Entitlements

 A subscription purchase creates an entitlement. [Entitlements](https://docs.discord.com/developers/resources/entitlement) describe access to premium app features. Depending on the app, combine [interaction payloads](https://docs.discord.com/developers/interactions/receiving-and-responding), the [Entitlement HTTP API](https://docs.discord.com/developers/resources/entitlement), and [Gateway events](https://docs.discord.com/developers/events/gateway-events#entitlements) to track user and guild entitlements and grant subscribed features.

### Accessing Entitlements with Gateway Events

 When users purchase in the app, Discord emits [Entitlement Gateway events](https://docs.discord.com/developers/events/gateway-events#entitlements). Subscription SKUs can produce these entitlement events:
| Event | Description |
| --- | --- |
| `ENTITLEMENT_CREATE` | When the user receives an entitlement for the app's subscription SKU |
| `ENTITLEMENT_UPDATE` | When entitlement access to a subscription SKU ends |
| `ENTITLEMENT_DELETE` | When a subscription refund, entitlement removal, or Test Entitlement deletion by a developer occurs |

### Accessing Entitlements with the HTTP API

 Apps that run background jobs or do more than interaction handling need entitlement tracking. Use [List Entitlements](https://docs.discord.com/developers/resources/entitlement#list-entitlements) to retrieve active and expired entitlements. Filter by user or guild with `?user_id=XYZ` or `?guild_id=XYZ`. For example, an app might store entitlements in a database and verify access to a particular SKU before running a cron job or another task.

### Accessing Entitlements on Interaction Payloads

 The `Interaction Payload` also includes entitlements when a user interacts with the app. Read them from the `entitlements` field on the `Interaction Payload` while [receiving and responding to interactions](https://docs.discord.com/developers/interactions/receiving-and-responding). That field can indicate whether the user or guild is subscribed.

### Accessing Entitlements with the Embedded App SDK

 An [Activity](https://docs.discord.com/developers/activities/overview) built with the [Embedded App SDK](https://docs.discord.com/developers/developer-tools/embedded-app-sdk) can also [access a user's entitlements](https://docs.discord.com/developers/developer-tools/embedded-app-sdk#getentitlements). For SDK monetization guidance, see [Implementing In-App Purchases for Activities](https://docs.discord.com/developers/monetization/implementing-iap-for-activities).

---

### Prompting Users to Subscribe

### Responding with a Premium Button

 [Responding with a premium button](https://docs.discord.com/developers/monetization/managing-skus#responding-with-a-premium-button) lets an app prompt a subscription when a user tries to use a premium feature. Send a message with a [button](https://docs.discord.com/developers/components/reference#button) using a [premium style](https://docs.discord.com/developers/components/reference#button-button-styles) and a `sku_id` for the upgrade offer.

### Starting a Purchase from an Activity

 An [Activity](https://docs.discord.com/developers/activities/overview) using the [Embedded App SDK](https://docs.discord.com/developers/developer-tools/embedded-app-sdk) can also open checkout for a specific SKU through the SDK. See [Implementing In-App Purchases for Activities](https://docs.discord.com/developers/monetization/implementing-iap-for-activities) for monetization with the Embedded App SDK.

### Purchasing from the Store Page

 Users can begin, upgrade, or downgrade subscriptions from the app's [Store](https://docs.discord.com/developers/monetization/managing-skus#viewing-your-store-page) page. Link directly to that page with Discord's [Application Directory Store URL scheme](https://docs.discord.com/developers/monetization/managing-skus#linking-to-your-store).

---

### Supporting Subscriptions

 To support subscriptions, [create a subscription SKU](https://docs.discord.com/developers/monetization/managing-skus#creating-a-sku) and handle these scenarios:

### Starting a new subscription

 When a user starts a new subscription, expect these events:
| Event | Event Trigger |
| --- | --- |
| `SUBSCRIPTION_CREATE` | when the subscription is first created; `status` is `0 (active)` when the entitlement has been granted, or `1 (inactive)` before entitlement grant |
| `ENTITLEMENT_CREATE` | when the user receives an entitlement for the new subscription |
| `SUBSCRIPTION_UPDATE` | when the subscription is updated with `entitlement_ids`, `renewal_sku_ids`, and `status` (`0 (active)`) |

### Cancelling an existing subscription

 Users may cancel from Subscription settings at any time. On cancellation, these events are sent:
| Event | Event Trigger |
| --- | --- |
| `SUBSCRIPTION_UPDATE` | when the subscription moves toward ending with `status` `2 (ending)` and `canceled_at` is set to the user's cancellation timestamp |

 The subscription and entitlement remain valid until `current_period_end`. If the user does not resume before `current_period_end`, the subscription ends and these events arrive:
| Event | Event Trigger |
| --- | --- |
| `ENTITLEMENT_UPDATE` | when the current entitlement ends and `ends_at` is updated with a timestamp |
| `SUBSCRIPTION_UPDATE` | when the subscription is updated to `status` `1 (inactive)` |

### Resuming a cancelled subscription

 Before `current_period_end`, users can resume the subscription from Subscription settings. When they resume, these events are sent:
| Event | Event Trigger |
| --- | --- |
| `SUBSCRIPTION_UPDATE` | when the subscription continues again with `status` `0 (active)` and `canceled_at` is reset to null |

---

### Supporting Upgrades and Downgrades

 If the app has multiple subscription tiers, users can upgrade or downgrade through the [Store page](https://docs.discord.com/developers/monetization/managing-skus#viewing-your-store-page) or App Subscription settings. Create each tier as its own [subscription SKU](https://docs.discord.com/developers/monetization/managing-skus#creating-a-sku), then support these cases:

### Upgrading an existing subscription

 When a user upgrades from a lower tier to a tier with the same or higher price, Discord charges the price difference and resets the subscription period at upgrade time. The lower-tier entitlement ends immediately, producing these events:
| Event | Event Trigger |
| --- | --- |
| `ENTITLEMENT_UPDATE` | when the current entitlement ends and `ends_at` is updated with a timestamp |
| `ENTITLEMENT_CREATE` | when Discord creates a new entitlement for the upgrade subscription SKU |
| `SUBSCRIPTION_UPDATE` | when the subscription is updated with new `entitlement_ids`, `sku_ids`, `current_period_start`, and `current_period_end` |

### Downgrading an existing subscription

 When a user downgrades from a higher tier to a lower tier, Discord does not charge immediately because the new price is lower than what has already been paid. The current plan remains valid until `subscription.current_period_end`, and this event is sent:
| Event | Event Trigger |
| --- | --- |
| `SUBSCRIPTION_UPDATE` | when the subscription is updated to show the renewal SKU ID in `subscription.renewal_sku_ids` |

 When the current subscription reaches `subscription.current_period_end`, these events are sent:
| Event | Event Trigger |
| --- | --- |
| `ENTITLEMENT_UPDATE` | when the current entitlement ends and `ends_at` is updated with a timestamp |
| `ENTITLEMENT_CREATE` | when Discord creates a new entitlement for the downgraded subscription SKU |
| `SUBSCRIPTION_UPDATE` | when the subscription is updated with new `entitlement_ids`, `sku_ids`, `current_period_start`, and `current_period_end` |

---

### Using the Subscription API

 For monetization, [Entitlements](https://docs.discord.com/developers/resources/entitlement) are the source of truth for a user's access to a SKU. The Subscription API is for reporting and lifecycle management outside direct user interaction. Use the [Subscription API](https://docs.discord.com/developers/resources/subscription) to inspect subscription status, list subscriptions by user for reports, and check subscription state without reading entitlements directly.
- [List SKU Subscriptions](https://docs.discord.com/developers/resources/subscription#list-sku-subscriptions): Lists all subscriptions tied to one SKU in the app.
- [Get SKU Subscription](https://docs.discord.com/developers/resources/subscription#get-sku-subscription): Retrieve one subscription in the app.
- [Subscription Gateway events](https://docs.discord.com/developers/events/gateway-events#subscriptions): Discord emits gateway events when subscriptions are created, updated, and, rarely, deleted.

---

### Testing Your App Subscription Implementation

### Using Test Entitlements

 Test by [creating](https://docs.discord.com/developers/resources/entitlement#create-test-entitlement) and [deleting](https://docs.discord.com/developers/resources/entitlement#delete-test-entitlement) test entitlements. They let you check subscribed and unsubscribed behavior for a user or guild, but do not run Discord checkout. Use [Using Live Entitlements](https://docs.discord.com/developers/monetization/implementing-app-subscriptions#using-live-entitlements) for the payment flow. Test Entitlements omit `starts_at` and `ends_at` because they remain valid until deleted.

### Using Live Entitlements

 To test the complete payment flow, use a [premium styled button](https://docs.discord.com/developers/monetization/implementing-app-subscriptions#prompting-users-to-subscribe) or the Store page. Team members linked to the app automatically receive a 100% subscription discount, so they can purchase without a live payment method charge. Checkout creates a live subscription that renews until canceled and supports subscription renewal testing. If canceled, access stays active through the billing period end, represented by `current_period_end` on the [Subscription](https://docs.discord.com/developers/resources/subscription#subscription-object). Only entitlements created with the [create entitlement](https://docs.discord.com/developers/resources/entitlement#create-test-entitlement) endpoint can be deleted. Use Test Entitlements when development requires toggling premium access.
