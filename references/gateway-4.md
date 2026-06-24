# Getting Started With Privileged Intent Review, You Might Not Need A Privileged Intent

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Includes sections on Privileged Intents and which apps this guidance covers.

## What are Privileged Intents?

If you received a notice about [Privileged Intents](https://docs.discord.com/developers/events/gateway#privileged-intents), this guide explains the situation. Your app may have grown to the review point, may need to renew access that was previously approved, or may have lost behavior that depended on gated data. Use this page to understand the change, its impact on your app, and the next action to take.

---

### What are Privileged Intents?

 Discord restricts some Gateway data through [Privileged Intents](https://docs.discord.com/developers/events/gateway#privileged-intents). The current privileged intents are:
- **Guild Presences**: user status such as online or offline, activity details, and platform information
- **Guild Members**: events for member joins, removals, and updates, plus support for listing every member in a server
- **Message Content**: message object fields named `content`, `embeds`, `attachments`, `components`, and `poll`
 Because these intents expose sensitive categories of data, they start turned off and should be requested only when the app requires them. Without an enabled intent, the app simply does not receive the data controlled by that intent.

---

### What’s changed?

 Beginning June 10th, 2026, Discord updated the Privileged Intents review workflow. See the [original announcement here](https://support-dev.discord.com/hc/articles/40281523410967) for additional context. (Figure: Privileged Intents Requirements)

### 1. The review threshold is now based on users, not servers

 The old threshold used server count: fewer than 100 servers could use Developer Portal toggles, while 100+ servers required an application. The new threshold is the count of unique users who can access your app across every server where it is installed. When that reach is more than 10,000 users, the app must request Privileged Intent access. A bot in 50 high-population servers can therefore cross the line, while a bot in 200 small servers may remain below it. The metric is user reach, not installation count.

### 2. Apps must reapply annually for continued access

 Apps with Privileged Intent approval from an earlier review must renew that access once per year in the Developer Portal. Discord sends advance notification before the renewal deadline. The renewal gives you a chance to refresh your use-case documentation and show that the app still needs the requested data.

### 3. Apps can continue to use intents and join servers while in review

 During review, the app can keep operating with the privileged intents it has requested. It can also continue being added to more servers while Discord evaluates the submission.

---

### Step 1: Figure out where your app stands

 Discord counts unique users who can reach your app through all servers where the app is installed.

### Under 10,000 users

 Below 10,000 users, you may enable or disable privileged intents directly in the Developer Portal without submitting an access request. Open the app's **Bot** page, find **Privileged Gateway Intents**, and turn on only the intents needed for the app to work. This permission to toggle does not mean every intent should be enabled; use the minimum data required. If the app later exceeds 10,000 users, you will need to submit review details for each requested intent and prove the access is necessary. Starting with fewer intents reduces future migration work. For cases where privileged access may be avoidable, see [You Might Not Need a Privileged Intent](https://docs.discord.com/developers/gateway/you-might-not-need-a-privileged-intent).

### Approaching or over 10,000 users

 When an app reaches 10,000 users and has a Privileged Intent enabled, or tries to enable one, the Developer Portal will notify you that review is required. The application window is **90 days** from the notification date. While the request is pending, the app may keep joining servers and reaching additional users. If you miss the 90-day window, existing Privileged Intents access is removed, but you may submit a new access request later.

### Access already granted

 An app that already passed Privileged Intent review can keep its current access for now, but annual reapplication is required. Discord will notify you when renewal time arrives. From that notice, you have 90 days to submit the continued-access review. If Discord grants the renewal, you will receive another notice before the next yearly review. Keep current notes for each intent's use case as the app changes, so the next submission is easier to complete. Missing the 90-day renewal window removes current Privileged Intents access, though a new request can still be filed afterward.

---

### Step 2: Audit what Privileged Intents your app uses

 Before requesting access, inventory the data your bot truly needs. This audit is essential and is the step developers most often skip. Confirm that the feature requires Privileged Intent access and that the use case complies with the [Developer Policy](https://support-dev.discord.com/hc/en-us/articles/8563934450327-Discord-Developer-Policy) and [Developer Terms of Service](https://support-dev.discord.com/hc/en-us/articles/8562894815383-Discord-Developer-Terms-of-Service). Use [You Might Not Need a Privileged Intent](https://docs.discord.com/developers/gateway/you-might-not-need-a-privileged-intent) to review each intent and compare possible alternatives.

---

### Step 3: Decide whether to adopt an alternative or apply for access

 For every intent currently used by the app, choose one of two approaches:

### Path A: Remove the dependency

 If the app can deliver the same feature without a Privileged Intent, remove the dependency. This path avoids review submissions, yearly renewal, and the risk of losing access. Common migrations include:
- **Text commands to slash commands**: Many Message Content requests come from prefix-command bots. [Slash commands](https://docs.discord.com/developers/interactions/application-commands) are the preferred replacement and do not need privileged intents.
- **Full member list to on-demand lookups**: If the app stores all server members but uses only a small number, fetch individual members through the API when needed instead.
 For more replacement patterns, read [You Might Not Need a Privileged Intent](https://docs.discord.com/developers/gateway/you-might-not-need-a-privileged-intent).

### Path B: Apply for Privileged Intent access

 When a compliant feature truly depends on data exposed by Privileged Intents and no suitable substitute exists, submit an access request. Prepare the request carefully: **Be specific about your use case:** Avoid a broad claim such as "My bot needs message content." A stronger explanation could say the bot runs automated moderation and checks messages for phishing URLs plus known scam signals. Describe what the app does with the data and why interactions cannot replace it. **Only request what you need:** If moderation needs Message Content but Guild Presences are irrelevant, leave Guild Presences out. Asking for unsupported intents weakens the submission. **Describe responsible data handling:** If data is stored, explain why storage is needed and describe retention and security controls. If the app processes data in memory and discards it, state that. **Think about annual review from the start:** Yearly renewal is required, so keep responses clear, easy to update, and backed by records as the app changes. **Submit complete, accurate, and understandable information; otherwise Discord may delay review or deny the request.** Once access is approved, enable the granted intents as described in Step 4.

---

### Step 4: Enable the Intent in Dev Portal and Your Code

 A privileged intent with approved access must be enabled in **two places**. Missing either one can cause an error.

### 1. The Developer Portal

 Open the Developer Portal, choose the app, [click the **Bot** tab](https://discord.com/developers/applications/select/bot), and go to **Privileged Gateway Intents**. Turn on the intents that the app needs and, for reviewed apps, the intents that have been approved.

### 2. Your code

 The portal toggle only authorizes the app to receive that data. The bot must also request the intent when it connects to the Gateway. The exact code depends on the library, or you can define Gateway Intents manually. If code and portal settings do not match, the app receives a `[DisallowedIntents]` error when it connects.

---

### Step 5: Test and verify

 After changes are made, verify the behavior:
1. **Restart your bot.** Intent settings apply only on a new Gateway connection; an already-running bot will not see portal changes until it reconnects.
2. **Check your error logs.** A `DisallowedIntents` error usually means the requested intents in code do not match the Developer Portal settings or approval state.
3. **Test affected features.** When a Privileged Intent is removed, confirm the migrated implementation still works. If prefix commands moved to slash commands, check that they are registered and respond properly.
4. **Test with a second account.** Behavior can change when the author is the bot instead of another user. Use a separate Discord account to test the experience as a normal user.

---

### Common situations and what to do

### ”My bot’s commands stopped working”

 If a prefix-command bot such as `!help` depended on Message Content, it can still receive `MESSAGE_CREATE`, but the `content` field is empty. The bot never sees the prefix. **Fix:** Move the feature to [slash commands](https://docs.discord.com/developers/gateway/developers/interactions/application-commands).

### ”My welcome messages stopped firing”

 Real-time welcome messages require `GUILD_MEMBER_ADD` events when users join. **Fix:** Enable and request the Guild Members intent, and show that the access is required for the app's stated functionality.

### ”I got a DisallowedIntents error”

 The bot is requesting an intent that is not enabled in the Developer Portal or has not been approved for the app. **Fix:** Review the portal settings. If the toggle is off, enable it when needed. If the app is above 10,000 users and the toggle is locked, submit a Privileged Intent access request.

### ”My intent application was denied”

 A request may be denied if the submission does not prove the stated feature requires access, such as when interactions or other non-privileged methods can provide the behavior, or if the submission fails Discord requirements. Examples include unclear or incomplete information or a use case that violates policy. Sometimes Discord may give extra time to resubmit updated details or make required changes while current access remains active. In other cases access is removed, but a new request can still be submitted later. Review the notice Discord sent for the exact outcome. **Fix:** Reassess whether the intent is truly required and whether the use case meets the requirements. If it does, resubmit with clearer, more complete details and include evidence such as screenshots or video showing the feature that requires access. If slash commands, components, modals, or another alternative can provide the feature, rebuild on that path and stop requesting the intent. Discord also provides a guide that explains each intent and possible replacements.

### ”I already have access granted, and it’s been a year or more than a year”

 With annual review, Discord sends advance notification when reapplication is due. When that notice arrives, use the Developer Portal to submit the same kind of request used for the initial review. If you believe a yearly review is due but have not received a notice, wait for the notice that says it is time to re-apply.

### ”I’m not sure if I need an intent or not”

 Use [You Might Not Need a Privileged Intent](https://docs.discord.com/developers/gateway/you-might-not-need-a-privileged-intent). That companion guide reviews each intent with decision checklists and alternatives so you can decide whether the app needs access.

[You Might Not Need a Privileged IntentPrevious](https://docs.discord.com/developers/gateway/you-might-not-need-a-privileged-intent)Ã¢Å’ËœI

## Who does this apply to?

(Figure: Privileged Intents Requirements) [Privileged Intents](https://docs.discord.com/developers/events/gateway#privileged-intents) allow a Discord app to receive extra server data after it is added to a server. That data can be useful, but it also carries additional responsibility and review requirements. Before enabling one, evaluate the need carefully:

>  Do I actually need this?

 **Use this guide to evaluate each privileged intent and decide whether the app needs it or can use another design.** Developers sometimes enable privileged intents out of habit or because an example did so, then later discover Discord APIs can support the same feature with less access. Apps may access only the data required for their functionality. Review the [Developer Policy](https://support-dev.discord.com/hc/en-us/articles/8563934450327-Discord-Developer-Policy), the [Developer Terms of Service](https://support-dev.discord.com/hc/en-us/articles/8562894815383-Discord-Developer-Terms-of-Service), and this guide to confirm the app and its intended privileged-intent use fit those rules.

### Who does this apply to?

 This applies to any app that has enabled privileged intents or is considering them. Apps with fewer than 10,000 server-installed users can manage the toggles in the Developer Portal. After the app passes 10,000 users, continued access requires a review request with details about the app and the use case for each requested intent. Planning early can prevent a later migration.

---

### The Three Privileged Intents

| Intent | What it controls |
| --- | --- |
| **Guild Presences** | Provides presence update events, including users going online or offline, status changes, and Rich Presence activity updates. |
| **Guild Members** | Provides guild member join, update, and leave events, and supports requesting complete member lists. |
| **Message Content** | Provides message object fields named `content`, `embeds`, `attachments`, `components`, and `poll` in Gateway events and API responses. This one differs from the others because it governs fields inside message objects rather than a specific event family. |

### Guild Members Intent

### What you get with the intent

 The Guild Members intent enables three broad capabilities:
- **Member events via the Gateway:** `GUILD_MEMBER_ADD`, `GUILD_MEMBER_UPDATE`, and `GUILD_MEMBER_REMOVE` are sent when users join, change profile or role data, or leave a server.
- **Full member list:** The app can request every member in a server through the Gateway [`Request Guild Members` opcode](https://docs.discord.com/developers/events/gateway-events#request-guild-members) or the [List Guild Members API](https://docs.discord.com/developers/resources/guild#list-guild-members) endpoint.

### What you can do without the intent

 If the app does not require every member or real-time member events, several REST endpoints work **without** the Guild Members intent:
- **[Get Guild Member](https://docs.discord.com/developers/resources/guild#get-guild-member):** Retrieve one member by user ID. When you already know *who* you need, enumerating the server is unnecessary.
- **[Search Guild Members](https://docs.discord.com/developers/resources/guild#search-guild-members):** Find members by username or nickname prefix. This suits autocomplete, lookup commands, and targeted searches.
- **[Get User](https://docs.discord.com/developers/resources/user#get-user):** Retrieve basic user details such as username and avatar without guild context.
- **[Interaction-provided member data](https://docs.discord.com/developers/interactions/receiving-and-responding#message-interaction-object):** Slash commands, buttons, select menus, and modals include a member object in the interaction payload. That gives roles, nickname, permissions, and related data without the intent.

### Decision checklist

 Ask yourself these questions:
1. Does the app need information about all members, or only particular members that can be fetched by ID or search?
2. Does the app need live join, leave, or update events, or can it request member data when a user interacts with the bot through interactions?
 If the answer is closer to "particular members" and "on demand," the Guild Members intent is probably unnecessary because REST endpoints and interaction payloads cover those needs.

---

### Guild Presences Intent

### What you get with the intent

 The Guild Presences intent allows the app to receive `PRESENCE_UPDATE` events containing:
- `status`: The user state, such as online, idle, do not disturb, or offline.
- `activities`: The user's current playing, streaming, listening, or watching activity, including Rich Presence data.
- `client_status`: The active platform or platforms, such as desktop, mobile, or web.

### What you can do without the intent

 Individual presence data is not available through another API. Before building around it, decide whether the app truly needs real-time presence per user. These alternatives do not require the Guild Presences intent:
- **[Get User](https://docs.discord.com/developers/resources/user#get-user):** Fetch profile fields such as username, avatar, and banner without using this intent. That is enough for many "user info" command designs.
- **[Guild Object - approximate_presence_count](https://docs.discord.com/developers/resources/guild#get-guild):** The Guild object exposes an approximate count of online members through the API without revealing individual presences.

### Decision checklist

1. Does the bot use presence data for a real feature, or only show it in a "user info" command?
2. Would `approximate_presence_count` satisfy the feature instead of per-user presence access?
3. Is this intent being requested for a specific core dependency, or merely "just in case"?

---

### Message Content Intent

### What you get with the intent

 The Message Content intent governs several fields in message objects across Gateway events and REST API responses:
- `content`: The message's text body
- `embeds`: Embedded content
- `attachments`: Files attached to the message
- `components`: User-sent message components such as buttons and selects
- `poll`: Poll data
 Without this intent, those fields are delivered as empty strings or empty arrays, except for several important cases.

### Exceptions: when you get message content without the privileged intent

 The app does **not** need Message Content intent to read message content in these cases:
- **Messages your app sends**
- **Direct Messages sent to your app**
- **Messages that @mention your app**
- **Replies to your app's messages.** This covers normal bot-message replies made with Discord's reply feature when "ping on reply" is enabled for the user. It does not include replies to slash command responses.
 When the app's feature can be designed around this available access, do not request Message Content intent.

### What you can do without the intent

 Discord provides interaction primitives so most bots can avoid Message Content intent: **[Slash Commands](https://docs.discord.com/developers/gateway/developers/interactions/application-commands)** are the most direct replacement for text commands. Users enter `/command`, and Discord handles argument parsing, validation, and autocomplete. Slash command options include `string`, `integer`, `number`, `boolean`, `user`, `channel`, `role`, `mentionable`, and `attachment`, which cover most input needs. **[Message Context Menu Commands](https://docs.discord.com/developers/interactions/application-commands#message-commands)** appear from a message context action such as right-click or long-press. The bot receives the full message object, including content, which fits features that act on a selected message, such as reporting, translating, pinning, or saving, without reading every channel message. **[Message Components](https://docs.discord.com/developers/components/overview)** use buttons and select menus to collect structured input for polls, verification, role selection, confirmations, and multi-step flows. **[Modals with Text Inputs](https://docs.discord.com/developers/components/using-modal-components)** present forms for freeform text, checkboxes, and other structured input, suitable for applications, feedback, forms, and prompts where users must type a specific response.

### Decision checklist

1. Does the bot rely on prefix commands such as `!help` or `?play` that could become slash commands?
2. Could users explicitly trigger the feature through a command, context menu, or other interaction?
3. Does the bot need the message's content, or only the fact that a message was created?
4. If the intended use is moderation, does the bot provide capabilities beyond the AutoMod API? Recreating functionality already supported by the AutoMod API generally is not a strong reason to grant access.

### Further Reading

- [Guide: Getting Started with Privileged Intent Review](https://docs.discord.com/developers/gateway/getting-started-with-privileged-intent-review)
