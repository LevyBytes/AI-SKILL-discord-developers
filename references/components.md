# Using Message Components, Using Modal Components

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: Overview.

## Overview

### Overview

 Message components let messages contain interactive UI, so users can act on your content directly. When components are sent through a [webhook](https://docs.discord.com/developers/resources/webhook), include the [`?with_components=true`](https://docs.discord.com/developers/resources/webhook#execute-webhook-query-string-params) query parameter or Discord ignores them.

### Prerequisites

- You need a Discord account with access to the Discord Developer Portal.
- Create a Discord application in the Discord Developer Portal before using this flow.
- The app or user sending the message needs permission to send messages in the target channel.

---

### Sending a Message with a Component

 A message that uses components must set the `IS_COMPONENTS_V2` flag (`1<<15`) on the message `flags` field. You can set it through [Message Create](https://docs.discord.com/developers/resources/message#create-message), [Execute Webhook](https://docs.discord.com/developers/resources/webhook#execute-webhook), or when [responding to an interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#create-followup-message). After a message is created with `IS_COMPONENTS_V2`, editing the message cannot remove that flag. The flag marks the message as component-based, turns off the traditional content and embeds format, and requires all visible content to be represented through components.

```
{
  "flags": 32768,
  "components": [
    {
      "type": 10,
      "content": "This is a message using the Text Display component"
    }
  ]
}

```

### Sending a Message with Multiple Components

 To include more than one component in a message, put multiple component objects in the message `components` array.

```
{
  "flags": 32768,
  "components": [
    {
      "type": 10,
      "content": "This is a Text Display component."
    },
    {
      "type": 10,
      "content": "This is another Text Display component!"
    }
  ]
}

```

### Nesting Components with Layout Components

 Layout components can contain other components, giving you control over how information, images, and interactive controls are arranged for users. The [list of components](https://docs.discord.com/developers/components/reference#component-object-component-types) documents every available layout component. For instance, an Action Row component can wrap several Button components.

```
{
  "flags": 32768,
  "components": [
    {
      "type": 10,
      "content": "This is a message with v2 components"
    },
    {
      "type": 1,
      "components": [
        {
          "type": 2,
          "style": 1,
          "label": "Click Me",
          "custom_id": "click_me_1"
        },
        {
          "type": 2,
          "style": 2,
          "label": "Click Me Too",
          "custom_id": "click_me_2"
        }
      ]
    }
  ]
}

```

### Using Message Components with Interactions

 When someone uses an interactive message component, your app [receives an interaction event](https://docs.discord.com/developers/interactions/overview). The event includes details such as the interaction type and the component that was used. The [list of supported component types](https://docs.discord.com/developers/components/reference#component-object-component-types) maps interactive message components to their interaction event payloads. Use that data to reply, edit the message, or take another action, such as opening a modal based on the user's input. The [Interactions documentation](https://docs.discord.com/developers/interactions/overview) explains how to process interactions and respond to input from interactive components.

## Overview

### Overview

 Modal components are useful when your app needs freeform input from users.

### Prerequisites

- You need a Discord account with access to the Discord Developer Portal.
- Create a Discord application in the Discord Developer Portal first.

---

### Displaying a Modal

 Show a modal as the response to an [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding). Use an [interaction response](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-response-object) with the [`MODAL`](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-response-object-interaction-callback-type) interaction callback type plus the required [modal fields](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-response-object-modal). The following example uses a [String Select](https://docs.discord.com/developers/components/reference#string-select) and a [Text Input](https://docs.discord.com/developers/components/reference#text-input), each wrapped in [Labels](https://docs.discord.com/developers/components/reference#label):

```
{
  "type": 9,
  "data": {
    "custom_id": "bug_modal",
    "title": "Bug Report",
    "components": [
      {
        "type": 18,
        "label": "What's your favorite bug?",
        "component": {
          "type": 3,
          "custom_id": "bug_string_select",
          "placeholder": "Choose...",
          "options": [
            {
              "label": "Ant",
              "value": "ant",
              "description": "(best option)",
              "emoji": {
                "name": "🐜"
              }
            },
            {
              "label": "Butterfly",
              "value": "butterfly",
              "emoji": {
                "name": "🦋"
              }
            },
            {
              "label": "Caterpillar",
              "value": "caterpillar",
              "emoji": {
                "name": "🐛"
              }
            }
          ]
        }
      },
      {
        "type": 18,
        "label": "Why is it your favorite?",
        "description": "Please provide as much detail as possible!",
        "component": {
          "type": 4,
          "custom_id": "bug_explanation",
          "style": 2,
          "min_length": 1000,
          "max_length": 4000,
          "placeholder": "Write your explanation here...",
          "required": true
        }
      }
    ]
  }
}

```

 (Figure: Example of an Action Row with three buttons)
