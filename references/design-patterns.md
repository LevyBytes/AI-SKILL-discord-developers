# Design Patterns

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Before creating an Activity, consider these principles for building a strong user experience.

### Guiding Principles

### Interaction over Isolation

- Activities should not replace screen sharing, because Discord already provides that feature.
- Build interaction paths between users.
- One user's actions should affect what other participants experience.

### Expression over Monotony

- Discord users want experiences that produce memorable moments.
- Design spaces where users can make moments and respond in ways worth sharing.

### Accessibility over Exclusion

- Inclusivity is key.
- Lower the entry barrier for the Activity while rewarding deeper participation.

---

### Providing a “Solo-Plus” Experience

 Strong experiences support small group sessions ("Plus") while still being playable and enjoyable for single-player ("Solo") sessions.

### Solo or single-player

- Many Discord sessions happen with small friend groups. Design something worthwhile alone that becomes better with others.
- If solo play is unsupported, many users will still inspect the Activity alone. Provide a preview or reason for them to return with friends.

### Solo-Plus or small groups of users

- Small groups of 3-8 people tend to have stronger engagement and retention than single-player experiences.
- Players are more likely to return when they can join alongside friends.

### Large groups of users

- A "max participants" suggestion can be shown, but the practical cap is the number of people who can enter the Voice call.
- Plan how the Activity behaves when the call has 25 or more participants.

---

### User Presence & Privacy

 Inside a shared Activity, each player should be able to see other participants' presence and actions.

### Make actions and presence of the others visible to each player

- Users like spaces that visibly feel active.
- When a user has a server nickname or server avatar, use that server identity in game.
- Indicate whether a user is speaking in the voice call, active, or inactive.
 (Figure: Speech bubbles in Bobble League)

### Respect user privacy

- For Activity sessions that match users across multiple servers, default to anonymized usernames and avatars.
- Populate Discord personal identifiers in the Activity only after user confirmation.
- Use those identifiers only when the viewers already have access through the server or DM.

---

### Deliver a Quality Experience

 Make the app quick, simple to enter, and fun enough to become popular.

### Surprise and delight users

- Surprise and delight means paying attention to small details in the user's experience.
- Add the right emotional touch at unexpected moments to create the magic.
 (Figure: Bobble League)

### Keep load times as low as possible

- Low load times support drop-in and drop-out use, especially for Discord's many mobile users.
- See [Quality & Testing Recommendations](https://docs.discord.com/developers/activities/design-patterns#quality-and-testing) for minimum quality areas and recommended testing.
- See [Technical Considerations](https://docs.discord.com/developers/activities/design-patterns#technical-considerations) for guidance on load partitioning and development-tool choices that reduce loading time.
- Account for desktop and mobile screen sizes and orientations, and ensure UI elements scale correctly.

### Support drop-in, drop-out behavior

- Activities are easy to discover and low-friction to join, so users may enter while an experience is already underway. Give late joiners something useful to do, even if they only spectate until participation would not disrupt the session. Users may also leave suddenly or become afk (away from keyboard), so handle those cases cleanly.
- Support users who joined the call but have not started playing or interacting yet. Let them "spectate" active users, which is especially useful when the Activity has an ideal participant count.
 (Figure: Support drop-in, drop-out behavior in your Activity)

### Make your app as available as possible

#### Cross-Platform Support

- Desktop and mobile support gives access to the broadest user base.
- Discord usage is divided across mobile and desktop.

#### Discord is a Global Audience

- Support many languages and cultures so the app works well for more Discord users.

#### Implement Invites

- The Embedded App SDK can send invites from inside an Activity to friended users.
- Place invite buttons deliberately within screens or flows, not only at startup or the first moment. Useful invite prompts include:
  - Moments when players leave the activity session.
  - Moments when the minimum participant count is unmet, such as an activity requiring 2+ players, an experience that improves with more people, or a game like Chess that needs an opponent.

#### Implement Sharing

 Discord is a social platform built around conversation. Sharing and invites let an app appear in shared spaces, making it visible and reachable to people who are not currently playing. They are important ways to enter private social contexts and bring in more players.
- The Embedded App SDK can send an image or gif from inside an Activity.
  - Share photos or GIFs that capture fun, memorable, or brag-worthy moments. Do not make content shareable merely to advertise the activity.
  - A high score by itself may be less compelling than a strong play, a collaborative drawing, or another shared moment that starts conversation and makes others want to join.

 (Figure: Shared Moment from Chess in the Park)

#### Activities in Text Channels

- Activity UI, copy, and flows should not depend on voice participants to explain, organize, clarify, or teach the experience.
- Controls, CTAs, and instructions need to be clear enough that people in a text channel, especially first-time users, can begin quickly without voice explanation.
- Use dynamic first-time user experiences, "How To Play" guidance, toast messages, Call To Action buttons, and similar aids, but avoid cluttering the interface with too much copy.

### Monetization Considerations

 Monetization for Activities is coming soon. Keep these points in mind while designing an Activity.
- Do not put restrictive participation barriers such as a login wall or paywall early in the user journey.
- Avoid paid unlocks that create an unfair advantage over non-paying players or users, including "pay to win" designs.
- Use Discord's social context to offer cosmetic options such as skins, themes, avatars, and similar customization.

---

### Game Design Considerations

### Build games that are easy to learn, hard to master

- New players can feel pressure when they join a multiplayer game over voice/video in the moment.
- A simple, quick-reference control scheme helps players enter without feeling overwhelmed.
- The strongest pattern is an understandable game loop paired with controls that are easy to absorb. If a tutorial is required before the game makes sense, joining in the moment can feel intimidating.

### Discord users who engage with voice are among our most engaged users

- Users in voice calls with friends show higher Activity engagement and retention.

### Consider existing game expectations and tropes

- Games are more likely to attract users who already identify as gamers.
- Use familiar tropes and expectations from the genre of the game being developed.
- Lean into the interests served by the game or genre, and support them in ways that encourage return play.

### Activities close when the last participant in the Activity leaves it

- When launched again, even in the same DM or Voice Channel, the Activity is a new instance and may include a different participant set.

### If a game is launched, what % of the time do users reach different phases of the game?

- Track launch -> start because many users may drop between those points, especially without solo play or small-group support.
- Also track checkpoints across the game, including the percentage of started sessions that reach the end.
- If the Activity has multiple starting settings, compare popularity and success by setting; defaults may need adjustment.
  - For non-game Activities, analyze a relevant action instead, such as queueing a video or song for a co-watching or co-listening experience.

- High drop-off at one point should prompt investigation and possible flow changes.

### How many games are played in a session?

- More games per session is not always better, since some Activities are designed around one long game, but it is a useful baseline.
- If repeat plays per session are expected but uncommon, investigate the cause.

### How does the group size impact various key metrics?

- For example, compare whether larger groups are more or less likely to finish a game, replay, and so on.
- This can reveal weak points by group size, such as a game taking too long with many people or feeling thin with only two.
- Not every Activity needs broad group-size support, but if X number of players is an option, verify that each supported size is enjoyable.

---

### Co-Watching / Co-Listening

- Co-watching and co-listening experiences should add more than screen sharing. Find ways for each participant to affect the others' experience.
- Host controls can prevent chaotic playback. When host controls exist, still let everyone contribute somehow, even if that means suggesting content to the host.
- If content is unavailable to some participants, such as because of geo-restrictions, communicate that status when the content is queued and while it plays. Hosts generally want a good experience for friends. Keep playback synchronized with other participants in the local channel or call.
- Familiar patterns help users participate immediately. Most Discord users already understand media-browsing experiences such as Spotify, YouTube, and Netflix, so following those patterns makes content easier to understand.
- Mobile volume can use the device's built-in side buttons and should behave consistently on desktop, mobile, and browsers.

---

### Technical Considerations

### Developing for the iframe

- Remember that the Activity shares CPU, RAM, and GPU resources with the still-running Discord Client.
- Prioritize time-to-first-interaction.
- Keep Web Assembly (WASM) usage in the Activity as small as possible.
  - Older iOS hardware is especially sensitive to WebKit's WASM optimization compilation pass during the first minutes of Activity use. The result can include visible stutter, thermal problems, and possible reduction in Discord AV quality during that early period.

- For security reasons, all network traffic goes through the Discord Proxy.
- Create separate Activity versions in the Developer Portal for development stability levels, such as Development, Staging, and Production. This lets you test and display the last stable build while continuing work in development and staging environments.

### Developing in Unity

 Unity can be used for iframe development, but meeting performance and quality targets will likely require heavy optimization across the game. Web-first game engines usually perform better by default.

---

### Quality and Testing

### Verify User Interface and User Experience

- Verify that buttons and UI elements work correctly and do not block normal Activity use or gameplay.
- Test Discord username display so the chosen format, such as Discord username or server identity, is handled properly.

### Test for Performance

- Measure performance across phones, tablets, desktop computers, and multiple operating systems to confirm it is acceptable for most users.
- Lag and slowness should not meaningfully degrade the Activity or gameplay.

### Test for Audio

- Verify that sound effects and background music play correctly and that volume controls behave as intended.
- Users need to be able to hear while playing in a voice call.

### Validate Activity Controls

- Confirm game controls such as clicks, dragging, keyboard controls, swipes, and hover states behave correctly and fit the platform.
- Test tutorials and instructions inside the app for correctness and relevance to the current Activity behavior and platform, desktop or mobile.

### Mobile Device UI Considerations

- Check for devices with top notches or corner cameras that could cover or cut off UI elements and buttons, making them impossible to tap.
- Account for curved Android screen edges that may hide elements or make buttons unreachable.
- Consider iOS/Android swipe gestures that can conflict with buttons or UI at the bottom of the screen.
- Ensure Android back buttons do not overlap lower-screen elements or buttons in a way that prevents taps.
- Honor the platform safe area so content and buttons are not clipped or disabled.

### Desktop Accessibility Guidelines

- On desktop, allow repeated TAB presses to move focus through Discord buttons and fields, and make sure the activity does not capture that control.
- Make ESC close any open in-activity modals for a consistent user experience.
- Use high-contrast colors that work for color-blind users, especially for gameplay elements that must be distinguished by color.
- Avoid essential gameplay objects that differ only by similar colors, because color-blind users may not be able to tell them apart.

[Discovery Best PracticesPrevious](https://docs.discord.com/developers/discovery/best-practices)⌘I
