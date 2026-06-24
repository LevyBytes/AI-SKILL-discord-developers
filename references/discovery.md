# Best Practices, Enabling Discovery, Overview

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: The Elevator Pitch; Step 1: Verify Your App; App Directory.

## The Elevator Pitch

After creating a Discord app, you may be preparing to make it discoverable in the App Directory, or you may already have a listing that is not getting the results you want. This guide gives practical guidance from Discord Staff for improving an App Directory Product Page. The guidance refers to values you can configure in your [app's settings](https://discord.com/developers/applications). If you still need to create an app, start with the [Getting Started guide](https://docs.discord.com/developers/quick-start/getting-started).

### The Elevator Pitch

### Quickly tell your users what your app does

 Descriptions should quickly explain both what the app does and why someone should care. Keep the copy concise, plain, and engaging so a potential user understands the value and wants to add the app. Proofread spelling and grammar before publishing because the listing may be viewed by a very large audience. Discord lets you define several description fields from your [app's settings](https://discord.com/developers/applications).

#### App General Information Description

 The [**General Information** tab](https://discord.com/developers/applications/select/information) includes a general **Description** field with a 400 character maximum. That text is shown on the bot user's profile when someone opens the app in Discord. (Figure: App description on the General Information tab)

#### App Summary

 The [**App Directory** tab](https://discord.com/developers/applications/select/discovery) contains the **Summary** field, a 200 character maximum description shown in App Directory search results. Write the Summary to catch attention and communicate value quickly. A vague statement may be accurate but still fail to explain how the app improves or entertains a server. Stronger summaries open with a line that names a problem the user may want solved. (Figure: Poorly-written app Summary on the App Directory tab) (Figure: Well-written app Summary on the App Directory tab)

#### App Expanded Description

 The [**App Directory** tab](https://discord.com/developers/applications/select/discovery) also has an **Expanded Description** field near the page bottom. This copy appears after a user opens the app's App Directory profile. Use it to show why the app is worth installing and to highlight its best capabilities. Markdown formatting is supported. Start by making the app's value clear, then explain how a new user should begin after installation.

### Show Your Story

### Include as many relevant visuals as possible

#### Show Your App’s Functionality

 Use images and gifs on the Product Page to demonstrate the app's capabilities. A useful starting point is to show commands from the description and how they operate inside Discord. Consider the commands existing users rely on most and the normal ways they interact with the app. New users should be able to understand what the installed app will look like in a Discord server. Product Page media works best when it shows actual behavior and Discord presentation.

#### Include Your App’s Lore

 If the app has lore or a recognizable theme, related images can also help. For a "Hypesquad App," for example, the Product Page could show Hypesquad badges and explain what each badge represents.

#### Add a Video

 Once images are in place, add a short screen recording that demonstrates the app. Video is useful, but still include images because some users will not watch the entire clip.

### Find Your Users

### Utilize search to your advantage

 Choose as many as five words that describe the app and use them as tags. Think about the app's categories and the keywords people might enter when searching for apps to install on their servers. (Figure: App tags which help categorize apps and make them more searchable)

### Support Your Users

### Engage with your users directly

 A support server is an important part of the App Product Page. Provide a dedicated server and channel where users can communicate with the app's developers and maintainers. If the support server is not [discoverable](https://support.discord.com/hc/en-us/articles/360030843331-Enabling-Server-Discovery), add an invite in the "links" area of the App Directory Product Page. Useful support-server practices include:
- Ask users to provide feedback and describe their experience with the app.
- Provide technical help for people using the app.
- Post app updates in the support server, and revise the App's Product Page description when an update changes what the app does.
 For broader server-operation guidance, see the [Community page](https://discord.com/community).

### General Housekeeping

### Final checks to make sure you’re ready to go

 Before using the final "Enable Discovery" control, confirm the App Directory Product Page meets Discord's remaining Directory eligibility criteria, [app content requirements](https://support-dev.discord.com/hc/en-us/articles/9489299950487-App-Directory-App-Content-Requirements-Policy), Developer [Terms of Service](https://support-dev.discord.com/hc/en-us/articles/8562894815383), and Developer [Policy](https://support-dev.discord.com/hc/en-us/articles/8563934450327). For additional explanation of profile-page requirements, read the [Help Center article on Profile Pages](https://support-dev.discord.com/hc/en-us/articles/6378525413143-App-Directory-App-profile-pages). After those checks, publish the App Directory profile. If you have not joined yet, the official [Discord Developers server](https://discord.gg/discord-developers) provides more information about updating apps.

## Step 1: Verify Your App

When the app is ready for new users and server admins to find it, enabling **Discovery** makes it searchable and installable through the [App Directory](https://docs.discord.com/developers/discovery/overview#app-directory) and [App Launcher](https://docs.discord.com/developers/discovery/overview#app-launcher).

### Step 1: Verify Your App

 Before **Discovery** can be enabled, the team owner must complete identity verification and app verification. **App Verification** also unlocks [monetization features](https://docs.discord.com/developers/monetization/overview), including in-app purchases and subscriptions. To review **App Verification** requirements:
1. Open the [Developer Portal](https://discord.com/developers/applications) and choose the app.
2. In the left menu, open [App Verification](https://discord.com/developers/applications/select/verification-onboarding) to review the verification requirements.
3. Satisfy every App Verification qualification requirement for the app.
4. Submit the app for verification after the requirements are complete.
 For additional **App Verification** details, see the [Help Center article](https://support-dev.discord.com/hc/en-us/articles/23926564536471-How-Do-I-Get-My-App-Verified).

### Step 2: Opt into Discovery

 Once the app is verified, enable **Discovery** from the Developer Portal so it can appear in the App Directory and App Launcher. To opt in to **Discovery**:
1. Open the [Developer Portal](https://discord.com/developers/applications) and choose the app.
2. From the left menu, select [Discovery -> Discovery Status](https://discord.com/developers/applications/select/discovery/status) to view the enablement requirements.
3. Finish the listed **Discovery** qualification requirements for the app.
4. Under [Discovery -> Discovery Settings](https://discord.com/developers/applications/select/discovery/settings), add metadata and images that define how the app appears on discovery surfaces.
5. Enable **Discovery** after all requirements are satisfied.
 For more about **Discovery**, read the [Help Center article](https://support-dev.discord.com/hc/en-us/articles/21204493235991-How-Can-Users-Discover-and-Play-My-Activity). After **Discovery** is enabled, the app can take up to 24 hours to show in the App Directory and App Launcher.

### Search for your App in Discord

 Verify discoverability by searching for the app in the App Directory or App Launcher.
1. Go to the [App Directory](https://discord.com/application-directory) and search for the app name.
2. If the app appears, the setup is working.
3. If the app does not appear, confirm every earlier step is complete and allow up to 24 hours for it to show.

---

### Next Steps

 After **Discovery** is enabled, continue by tailoring the app's presentation in the App Directory and App Launcher.

### Discovery Best Practices

Read the guide for making the app more noticeable.

## App Directory

This page explains the discovery surfaces available for an app and how those surfaces behave. For a higher-level introduction to Discord app discovery, see the [App Discovery](https://docs.discord.com/developers/platform/discovery) platform page. App discovery is available for [Bots](https://docs.discord.com/developers/bots/overview) and [Activities](https://docs.discord.com/developers/activities/overview).

### App Directory

 After [enabled discovery for your app](https://docs.discord.com/developers/discovery/enabling-discovery), users can locate it in the App Directory, a searchable area for browsing apps by name, category, or collection.
- **Search**: Users can look up the app by name and install it.
- **App Directory Product Page**: Present app details such as descriptions, images, videos, and links.
 (Figure: Your Discord App Profile)

### App Launcher

 The App Launcher exposes apps from the app shapes icon throughout Discord, with both collections and search available. (Figure: App Launcher in Discord)

### App Launcher Collections

- **Recent Apps**: Recently used or installed apps are shown at the top of the App Launcher
- **Installed Apps**: Apps already installed for the account or server, when relevant
- **Curated Collections**: Recommended apps, staff selections, and user favorites grouped together
- **Partner Apps**: Apps made in partnership with Discord may show a "Partner" tag
- **Promoted Apps**: Apps with added visibility in the App Launcher or App Directory may carry a "Promoted" tag
 Search ranking uses query relevance and popularity derived from usage. If an app includes ads or in-app purchases, its details page displays "Ad-supported" and "In-App Purchases" tags.

### Editorial Recommendations

 Discord may place apps in curated collections or promotions in the App Directory or App Launcher. If Discord's editorial team wants to feature an app, they will contact the developer.

---

### Social Discovery

 Once an app is discoverable, Discord social activity can help users find it. If the app is used in a server, other users may see it, open the app profile, learn more, and install it.

### Sharing Links

 Create links that point to the app profile page, store page, or individual items. Those links can be shared in Discord, email, or a website. (Figure: A shared embed link in Discord for the Sandscape app)

### Rich Presence

 [Rich Presence](https://docs.discord.com/developers/rich-presence/overview) can show what users are doing in the app and encourage more discovery. (Figure: Examples of Rich Presence data on Discord user profiles)

---

### Next Steps

 Ready to turn on discovery for the app?

### Enabling Discovery

Turn on discovery so the app can appear in the App Directory and App Launcher.

### Discovery Best Practices

Review how to make the app more visible and attract additional users.
