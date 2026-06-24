# Getting Started (part 2)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

If the insertion point is unclear, refer to the complete implementation in `examples/app.js`.The code above handles several tasks:
1. Reads the request body to determine the slash-command caller ID (`userId`) and the selected option or object choice (`objectName`). For server interactions (`context === 0`), the user ID is under `member`; for a DM or Group DM, it is under `user`.
2. Creates an entry in `activeGames` keyed by the interaction ID. That active game stores both `userId` and `objectName`.
3. Replies in the channel with a button whose `custom_id` is `accept_button_<SOME_ID>`.
The sample stores state in an object in memory; production apps should replace that with a database.When a message includes [message components](https://docs.discord.com/developers/components/reference), each component payload goes in the `components` array. Interactive components such as buttons must be placed inside an [action row](https://docs.discord.com/developers/components/reference#action-row), as shown by the sample code.Pay attention to the distinct `custom_id` attached to message components here: `accept_button_` plus the active game ID. Use a `custom_id` to route the requests Discord sends when a user activates the component, as shown next.After running `/challenge` and choosing an option, the app posts a message with an **Accept** button. The next step is handling that button interaction.

Handling button interactions

Code for processing button clicks and replying with an ephemeral message

When someone uses a message component, Discord sends an interaction whose [interaction type](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-type) is `3`, or `MESSAGE_COMPONENT` when using `discord-interactions`.To handle the button, inspect the interaction `type`, then match the `custom_id`.Place this code below the `APPLICATION_COMMAND` type handler:

```
if (type === InteractionType.MESSAGE_COMPONENT) {
  // custom_id set in payload when sending message component
  const componentId = data.custom_id;

  if (componentId.startsWith('accept_button_')) {
    // get the associated game ID
    const gameId = componentId.replace('accept_button_', '');
    // Delete message with token in request body
    const endpoint = `webhooks/${process.env.APP_ID}/${req.body.token}/messages/${req.body.message.id}`;
    try {
      await res.send({
        type: InteractionResponseType.CHANNEL_MESSAGE_WITH_SOURCE,
        data: {
          // Indicates it'll be an ephemeral message
          flags: InteractionResponseFlags.EPHEMERAL | InteractionResponseFlags.IS_COMPONENTS_V2,
          components: [
            {
              type: MessageComponentTypes.TEXT_DISPLAY,
              content: 'What is your object of choice?',
            },
            {
              type: MessageComponentTypes.ACTION_ROW,
              components: [
                {
                  type: MessageComponentTypes.STRING_SELECT,
                  // Append game ID
                  custom_id: `select_choice_${gameId}`,
                  options: getShuffledOptions(),
                },
              ],
            },
          ],
        },
      });
      // Delete previous message
      await DiscordRequest(endpoint, { method: 'DELETE' });
    } catch (err) {
      console.error('Error sending message:', err);
    }
  }
  return;
}

```

The above code:
1. Looks for the `custom_id` originally sent by the app; in this example it begins with `accept_button_`. Because the active game ID is appended to that custom ID, the handler saves it in `gameID`.
2. [Deletes the original message](https://docs.discord.com/developers/interactions/receiving-and-responding#delete-original-interaction-response) by calling a webhook through `node-fetch` and including the unique interaction `token` in the request body. This removes clutter from the channel and prevents other users from pressing the button.
3. Answers the request with a message containing a select menu for the game's object choices. The payload is mostly the same as the earlier payload, except that it includes the `options` array and `flags: 64`, [which marks the message as ephemeral](https://docs.discord.com/developers/interactions/receiving-and-responding#create-followup-message).
`getShuffledOptions()` in `game.js` fills the `options` array by reshaping `RPSChoices` into the [string select options](https://docs.discord.com/developers/components/reference#string-select-select-option-structure) format.

Handling select menu interactions

Code for handling select menu interactions and applying the game-state update

The final addition is a select-menu interaction handler that posts the game result to the channel.Because select menus are message components too, their interaction handling is very close to the button path.Update the earlier code so it also handles the select menu:

```
if (type === InteractionType.MESSAGE_COMPONENT) {
  // custom_id set in payload when sending message component
  const componentId = data.custom_id;

  if (componentId.startsWith('accept_button_')) {
    // get the associated game ID
    const gameId = componentId.replace('accept_button_', '');
    // Delete message with token in request body
    const endpoint = `webhooks/${process.env.APP_ID}/${req.body.token}/messages/${req.body.message.id}`;
    try {
      await res.send({
        type: InteractionResponseType.CHANNEL_MESSAGE_WITH_SOURCE,
        data: {
          // Indicates it'll be an ephemeral message
          flags: InteractionResponseFlags.EPHEMERAL | InteractionResponseFlags.IS_COMPONENTS_V2,
          components: [
            {
              type: MessageComponentTypes.TEXT_DISPLAY,
              content: 'What is your object of choice?',
            },
            {
              type: MessageComponentTypes.ACTION_ROW,
              components: [
                {
                  type: MessageComponentTypes.STRING_SELECT,
                  // Append game ID
                  custom_id: `select_choice_${gameId}`,
                  options: getShuffledOptions(),
                },
              ],
            },
          ],
        },
      });
      // Delete previous message
      await DiscordRequest(endpoint, { method: 'DELETE' });
    } catch (err) {
      console.error('Error sending message:', err);
    }
  } else if (componentId.startsWith('select_choice_')) {
    // get the associated game ID
    const gameId = componentId.replace('select_choice_', '');

    if (activeGames[gameId]) {
      // Interaction context
      const context = req.body.context;
      // Get user ID and object choice for responding user
      // User ID is in user field for (G)DMs, and member for servers
      const userId = context === 0 ? req.body.member.user.id : req.body.user.id;
      const objectName = data.values[0];
      // Calculate result from helper function
      const resultStr = getResult(activeGames[gameId], {
        id: userId,
        objectName,
      });

      // Remove game from storage
      delete activeGames[gameId];
      // Update message with token in request body
      const endpoint = `webhooks/${process.env.APP_ID}/${req.body.token}/messages/${req.body.message.id}`;

      try {
        // Send results
        await res.send({
          type: InteractionResponseType.CHANNEL_MESSAGE_WITH_SOURCE,
          data: {
            flags: InteractionResponseFlags.IS_COMPONENTS_V2,
            components: [
              {
                type: MessageComponentTypes.TEXT_DISPLAY,
                content: resultStr
              }
            ]
            },
        });
        // Update ephemeral message
        await DiscordRequest(endpoint, {
          method: 'PATCH',
          body: {
            components: [
              {
                type: MessageComponentTypes.TEXT_DISPLAY,
                content: 'Nice choice ' + getRandomEmoji()
              }
            ],
          },
        });
      } catch (err) {
        console.error('Error sending message:', err);
      }
    }
  }

  return;
}

```

As in the previous handler, this code reads the interacting user's ID and selected object from the interaction request. It passes that data, together with the original user's ID and choice from `activeGames`, into `getResult()`. `getResult()` decides who won and creates a human-readable result string for the channel.The code also calls a webhook to [update the follow-up ephemeral message](https://docs.discord.com/developers/interactions/receiving-and-responding#edit-followup-message), since that message cannot be deleted.Finally, the app posts the result to the channel with the `CHANNEL_MESSAGE_WITH_SOURCE` interaction response type. Test the app to confirm the full flow works.

---

### Next steps

 You have now built a first Discord app. This walkthrough covered basic app configuration and interactive behavior; continue expanding the app or exploring other platform capabilities from here.

### Overview of Apps

Review the platform features and APIs available for Discord app development

### Explore developer tools

Browse first-party and community libraries and tools that can make development faster and simpler

### Developing User-Installable Apps

Guide for building interactions for user-installed apps and handling those interactions

### Discord Developers

Join the community to ask API questions, attend Discord platform-team events, and talk with other developers
