# Components — Reference (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

This reference covers every available component and divides them into three broad groups:
- **Layout Components** - Used to arrange and structure content (Action Rows, Sections, Containers)
- **Content Components** - Used to show static text, images, and files (Text Display, Media Gallery, Thumbnails)
- **Interactive Components** - Used for user input and interaction (Buttons, Select Menus, Text Input)
 To use these components, send the [message flag](https://docs.discord.com/developers/resources/message#message-object-message-flags) `1 << 15` (IS_COMPONENTS_V2) on the message. The flag is applied per message. After a message is sent with this flag, it cannot be removed from that message. Enabling the new component system changes message behavior as follows:
- The `content` and `embeds` fields stop working; use [Text Display](https://docs.discord.com/developers/components/reference#text-display) and [Container](https://docs.discord.com/developers/components/reference#container) instead
- Attachments are hidden unless components expose them
- The `poll` and `stickers` fields are unavailable
- A message can contain up to 40 components total
 [Legacy component behavior](https://docs.discord.com/developers/components/reference#legacy-message-component-behavior) still works, but gives less control and layout flexibility. For implementation guidance, see [Using Message Components](https://docs.discord.com/developers/components/using-message-components) and [Using Modal Components](https://docs.discord.com/developers/components/using-modal-components).

---

### What is a Component

 Components let you shape the layout and appearance of messages, modals, and interactions. They provide interactive UI pieces that can create richer Discord app experiences. Components are fields on the [message object](https://docs.discord.com/developers/resources/message#message-object) and [modal](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-response-object-modal). You can include them when creating messages or when responding to interactions such as an [application command](https://docs.discord.com/developers/interactions/application-commands).

### Component Object

###### Component Types

 The table below lists all supported components. Later sections describe each component in detail.
| Type | Name | Description | Style | Usage |
| --- | --- | --- | --- | --- |
| 1 | Action Row | Container that presents one row of interactive components | Layout | Message |
| 2 | Button | Button object | Interactive | Message |
| 3 | String Select | Select menu whose choices are predefined text options | Interactive | Message, Modal |
| 4 | Text Input | Text input object | Interactive | Modal |
| 5 | User Select | Select menu for users | Interactive | Message, Modal |
| 6 | Role Select | Select menu for roles | Interactive | Message, Modal |
| 7 | Mentionable Select | Select menu for mentionables, covering users *and* roles | Interactive | Message, Modal |
| 8 | Channel Select | Select menu for channels | Interactive | Message, Modal |
| 9 | Section | Container that pairs text with an accessory component | Layout | Message |
| 10 | Text Display | Markdown text | Content | Message, Modal |
| 11 | Thumbnail | Small image usable as an accessory | Content | Message |
| 12 | Media Gallery | Display images and other media | Content | Message |
| 13 | File | Displays an attached file | Content | Message |
| 14 | Separator | Component that inserts vertical spacing between components | Layout | Message |
| 17 | Container | Container that visually groups multiple components | Layout | Message |
| 18 | Label | Container that connects a label and description to a component | Layout | Modal |
| 19 | File Upload | Component for uploading files | Interactive | Modal |
| 21 | Radio Group | Single-choice set of options | Interactive | Modal |
| 22 | Checkbox Group | Multi-selectable group of checkboxes | Interactive | Modal |
| 23 | Checkbox | One checkbox representing a yes/no choice | Interactive | Modal |

---

### Anatomy of a Component

 Every component includes these fields:
| Field | Type | Description |
| --- | --- | --- |
| type | integer | The type of the component |
| id? | integer | optional 32 bit integer identifier for the component |

 The `id` field is optional and identifies components in interaction responses. Each `id` must be unique within the message. If omitted, Discord generates `id` values sequentially. Generated `id`s will not reuse an `id` already assigned to another component in the same message. Sending `id` as `0` is accepted, but the API treats it as empty and replaces it.

###### Custom ID

 Interactive components such as buttons and selects also require `custom_id`. Developers set this field in the component payload, and Discord returns it in the interaction payload when a user interacts with the component. For example, a button sent with `custom_id: click_me` produces an interaction containing `custom_id: click_me` when clicked. `custom_id` exists only on interactive components and must be unique for each component. Components in the same message cannot share a `custom_id`. The field is a 1 to 100 character string and can carry state or other important data.
| Field | Type | Description |
| --- | --- | --- |
| custom_id | string | Developer-defined identifier, 1-100 characters |

---

### Action Row

 An Action Row is a top-level layout component. It may contain one of these sets:
- Up to 5 contextually related [buttons](https://docs.discord.com/developers/components/reference#button)
- One select component: [string select](https://docs.discord.com/developers/components/reference#string-select), [user select](https://docs.discord.com/developers/components/reference#user-select), [role select](https://docs.discord.com/developers/components/reference#role-select), [mentionable select](https://docs.discord.com/developers/components/reference#mentionable-select), or [channel select](https://docs.discord.com/developers/components/reference#channel-select)
 For modals, prefer [Label](https://docs.discord.com/developers/components/reference#label) instead of Action Row. Action Row with Text Inputs in modals is deprecated.

###### Action Row Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `1` for action row component |
| id? | integer | Optional identifier for component |
| components | array containing the action row's child components | As many as 5 interactive buttons, or one select component |

###### Action Row Child Components

| Available Components | Description |
| --- | --- |
| Button | An Action Row may hold up to 5 Buttons |
| String Select | A single String Select |
| User Select | A single User Select |
| Role Select | A single Role Select |
| Mentionable Select | A single Mentionable Select |
| Channel Select | A single Channel Select |

###### Examples

Message Example

Payload for creating a message with an Action Row component

(Figure: Action Row example containing three buttons)

```
{
  "flags": 32768,
  "components": [
    {
      "type": 1,  // ComponentType.ACTION_ROW
      "components": [
        {
          "type": 2,  // ComponentType.BUTTON
          "custom_id": "click_yes",
          "label": "Accept",
          "style": 1
        },
        {
          "type": 2,  // ComponentType.BUTTON
          "label": "Learn More",
          "style": 5,
          "url": "http://watchanimeattheoffice.com/"
        },
        {
          "type": 2,  // ComponentType.BUTTON
          "custom_id": "click_no",
          "label": "Decline",
          "style": 4
        }
      ]
    }
  ]
}

```

---

### Button

 A Button is an interactive component available only in messages. It creates a clickable element for users, and clicking it sends an [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object) to your app. Buttons must be inside an [Action Row](https://docs.discord.com/developers/components/reference#action-row) or in a [Section](https://docs.discord.com/developers/components/reference#section)'s `accessory` field.

###### Button Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `2` for a button |
| id? | integer | Optional identifier for component |
| style | integer | A button style |
| label? | string | Text displayed on the button; maximum 80 characters |
| emoji? | partial emoji | `name`, `id`, and `animated` |
| custom_id? | string | Button identifier chosen by the developer; 1-100 characters |
| sku_id? | snowflake | Purchasable SKU identifier, available only for premium-style buttons |
| url? | string | URL used by link-style buttons; maximum 512 characters |
| disabled? | boolean | Whether the button is disabled; defaults to `false` |

 Button styles communicate different action types, and each style also determines which button fields are valid.
- Buttons that are neither link-style nor premium-style **must** include `custom_id`, and must not include `url` or `sku_id`.
- Link buttons **must** include `url`, and must not include `custom_id`
- Link buttons do not send your app an [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object) when clicked
- Premium buttons **must** include `sku_id`, and must not include `custom_id`, `label`, `url`, or `emoji`.
- Premium buttons do not send your app an [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object) when clicked

###### Button Styles

| Name | Value | Action | Required Field |
| --- | --- | --- | --- |
| Primary | 1 | Primary or recommended action among a set of choices | `custom_id` |
| Secondary | 2 | Alternative or supporting actions | `custom_id` |
| Success | 3 | Positive confirmation or completion actions | `custom_id` |
| Danger | 4 | An action with irreversible consequences | `custom_id` |
| Link | 5 | Navigates to a URL | `url` |
| Premium | 6 | Purchase | `sku_id` |

###### Examples

Message Example

Payload for creating a message with a Button component

```
{
  "flags": 32768,
  "components": [
    {
      "type": 1,  // ComponentType.ACTION_ROW
      "components": [
          {
            "type": 2,  // ComponentType.BUTTON,
            "custom_id": "click_me",
            "label": "Click me!",
            "style": 1
          }
      ]
    }
  ]
}

```

Message Interaction Response Example

A user clicks a Button in a message

When a user clicks a Button in a message, the interaction data payload follows this basic form. The complete payload is documented in the [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure) reference.

```
{
  "type": 3, // InteractionType.MESSAGE_COMPONENT
  ...additionalInteractionFields, // See the Interaction documentation for all fields

  "data": {
    "component_type": 2, // ComponentType.BUTTON
    "id": 2,
    "custom_id": "click_me",
  },
}

```

### Button Design Guidelines

###### General Button Content

- Maximum 34 characters when an icon or emoji is present.
- Maximum 38 characters when no icon or emoji is present.
- Make the text short and direct.
- Prefer plain, easy-to-understand wording. Avoid jargon and unnecessary technical language.
- Choose verbs that describe what the action will do.
- Keep wording and tone consistent across buttons.
- Plan for translation and test how text expands or contracts across languages.

###### Multiple Buttons

 Use button styles to establish hierarchy. Use only one `Primary` button in a group. (Figure: Example showing one primary button per button group) If several buttons carry equal importance, make each one `Secondary`. (Figure: Example showing multiple buttons in a group with equal significance)

###### Premium Buttons

 Premium buttons automatically include the following:
- Shop Icon
- SKU name
- SKU price
 (Figure: A premium button)

---

### String Select

 A String Select is an interactive component that lets users choose one or more supplied `options`. It can be configured for single-select or multi-select behavior. When selection is complete, your app receives an [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure). String Selects work in messages and modals. In messages they must be placed inside an [Action Row](https://docs.discord.com/developers/components/reference#action-row); in modals they must be placed inside a [Label](https://docs.discord.com/developers/components/reference#label).

###### String Select Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `3` for string select |
| id? | integer | Optional identifier for component |
| custom_id | string | Select menu ID; 1-100 characters |
| options | array of select options | Choices defined for the select menu; maximum 25 |
| placeholder? | string | Placeholder shown when no default or selection is present; maximum 150 characters |
| min_values?* | integer | Fewest items required; defaults to 1, minimum 0 (see note), maximum 25 |
| max_values? | integer | Most items allowed; defaults to 1, maximum 25 |
| required?** | boolean | Whether a modal requires an answer for the string select; defaults to `true` |
| disabled?*** | boolean | Whether the select menu is disabled in a message; defaults to `false` |

 * `min_values` must be omitted or set to at least `1` when `required` is omitted or `true`. ** The `required` field applies only to String Selects in modals and is ignored in messages. *** Setting `disabled` inside a modal causes an error because disabled components are not currently supported in modals.

###### Select Option Structure

| Field | Type | Description |
| --- | --- | --- |
| label | string | Option name shown to users; maximum 100 characters |
| value | string | Option value defined by the developer; maximum 100 characters |
| description? | string | Extra option description; maximum 100 characters |
| emoji? | partial emoji object | `id`, `name`, and `animated` |
| default? | boolean | Displays this option as selected by default |

###### String Select Interaction Response Structure

| Field | Type | Description |
| --- | --- | --- |
| type* | integer | `3` for a String Select |
| component_type* | integer | `3` for a String Select |
| id | integer | Unique identifier for the component |
| custom_id | string | Input identifier chosen by the developer; 1-100 characters |
| values | array of strings | Text values for the selected options |

 * Message interaction responses include `component_type`; modal interaction responses include `type`.

###### Examples

Message Example

Payload for creating a message with a String Select component

(Figure: String Select example with three options)

```
{
  "flags": 32768,
  "components": [
    {
      "type": 1, // ComponentType.ACTION_ROW,
      "id": 1,
      "components": [
        {
          "type": 3, // ComponentType.STRING_SELECT
          "id": 2,
          "custom_id": "favorite_bug",
          "placeholder": "Favorite bug?",
          "options": [
            {
              "label": "Ant",
              "value": "ant",
              "description": "(best option)",
              "emoji": {"name": "🐜"}
            },
            {
              "label": "Butterfly",
              "value": "butterfly",
              "emoji": {"name": "🦋"}
            },
            {
              "label": "Caterpillar",
              "value": "caterpillar",
              "emoji": {"name": "🐛"}
            }
          ]
        }
      ]
    }
  ]
}

```

Message Interaction Data Example

A user chooses an option from a message StringSelect

When a user uses a StringSelect in a message, the interaction data payload follows this basic structure. The complete payload is available in the [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure) reference.

```
{
  "type": 3, // InteractionType.MESSAGE_COMPONENT
  ...additionalInteractionFields, // See the Interaction documentation for all fields

  "data": {
    "component_type": 3, // ComponentType.STRING_SELECT
    "custom_id": "favorite_bug",
    "values": [
      "butterfly",
    ]
  },
}

```

Modal Example

Payload for creating a modal with a String Select component

(Figure: Modal example with a String Select containing three options)

```
{
  "type": 9, // InteractionCallbackType.MODAL
  "data": {
    "custom_id": "bug_modal",
    "title": "Bug Survey",
    "components": [
      {
        "type": 18, // ComponentType.LABEL
        "id": 1,
        "label": "Favorite bug?",
        "component": {
          "type": 3, // ComponentType.STRING_SELECT
          "id": 2,
          "custom_id": "favorite_bug",
          "placeholder": "Ants are the best",
          "options": [
            {
              "label": "Ant",
              "value": "ant",
              "description": "(best option)",
              "emoji": {"name": "🐜"}
            },
            {
              "label": "Butterfly",
              "value": "butterfly",
              "emoji": {"name": "🦋"}
            },
            {
              "label": "Caterpillar",
              "value": "caterpillar",
              "emoji": {"name": "🐛"}
            }
          ]
        }
      }
    ]
  }
}

```

Modal Submit Interaction Data Example

A user submits a modal that includes a String Select

When a submitted modal includes a String Select, the interaction data payload follows this basic form. See the [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure) reference for the complete payload.

```
{
  "type": 5, // InteractionType.MODAL_SUBMIT
  ...additionalInteractionFields, // See the Interaction documentation for all fields

  "data": {
    "custom_id": "bug_modal",
    "components": [
      {
        "type": 18, // ComponentType.LABEL
        "id": 1,
        "component": {
          "type": 3, // ComponentType.STRING_SELECT
          "id": 2,
          "custom_id": "favorite_bug",
          "values": [
            "butterfly",
          ]
        }
      }
    ]
  },
}

```

---

### Text Input

 Text Input is an interactive component for free-form text responses in modals. It supports both short single-line input and longer paragraph input. Text Inputs can be used only inside modals and must be placed in a [Label](https://docs.discord.com/developers/components/reference#label). Discord no longer recommends placing Text Input inside an [Action Row](https://docs.discord.com/developers/components/reference#action-row) in modals; all future Text Inputs should be placed inside a [Label](https://docs.discord.com/developers/components/reference#label) component.

###### Text Input Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `4` for a text input |
| id? | integer | Optional identifier for component |
| custom_id | string | Input identifier chosen by the developer; 1-100 characters |
| style | integer | The Text Input Style |
| min_length? | integer | Minimum text input length; minimum 0, maximum 4000 |
| max_length? | integer | Maximum text input length; minimum 1, maximum 4000 |
| required? | boolean | Whether filling this component is required; defaults to `true` |
| value? | string | Initial value filled into this component; maximum 4000 characters |
| placeholder? | string | Custom placeholder displayed when the input is empty; maximum 100 characters |

 The `label` field on Text Input is deprecated; use `label` and `description` on the [Label](https://docs.discord.com/developers/components/reference#label) component instead.

###### Text Input Styles

| Name | Value | Description |
| --- | --- | --- |
| Short | 1 | Single-line input |
| Paragraph | 2 | Multi-line input |

###### Text Input Interaction Response Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `4` for a Text Input |
| id | integer | Unique identifier for the component |
| custom_id | string | Input identifier chosen by the developer; 1-100 characters |
| value | string | The user’s input text |

###### Examples

Modal Example

Payload for creating a modal with a Text Input component

(Figure: Modal containing Text Input inside a Label)

```
{
  "type": 9, // InteractionCallbackType.MODAL
  "data": {
    "custom_id": "game_feedback_modal",
    "title": "Game Feedback",
    "components": [
      {
        "type": 18,  // ComponentType.LABEL
        "label": "What did you find interesting about the game?",
        "description": "Please give us as much detail as possible so we can improve the game!",
        "component": {
          "type": 4,  // ComponentType.TEXT_INPUT
          "custom_id": "game_feedback",
          "style": 2,
          "min_length": 100,
          "max_length": 4000,
          "placeholder": "Write your feedback here...",
          "required": true
        }
      }
    ]
  }
}

```

Modal Submit Interaction Data Example

A user submits a modal that includes a TextInput

When a submitted modal includes a TextInput, the interaction data payload follows this basic form. The complete payload is documented in the [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure) reference.

```
{
  "type": 5, // InteractionType.MODAL_SUBMIT
  ...additionalInteractionFields, // See the Interaction documentation for all fields

  "data": {
    "custom_id": "game_feedback_modal",
    "components": [
      {
        "type": 18, // ComponentType.LABEL
        "id": 1,
        "component": {
          "type": 4, // ComponentType.TEXT_INPUT
          "id": 2,
          "custom_id": "game_feedback",
          "value": "The recent changes to acceleration feel much better, but shadows still need help"
        }
      }
    ]
  },
}

```

---

### User Select

 A User Select is an interactive component that lets users choose one or more users in a message or modal. Discord automatically fills options from users available in the server. User Selects support both single-select and multi-select behavior. After the user finishes choosing, your app receives an [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure). User Selects work in messages and modals. In messages they must be placed in an [Action Row](https://docs.discord.com/developers/components/reference#action-row); in modals they must be placed in a [Label](https://docs.discord.com/developers/components/reference#label).

###### User Select Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `5` for user select |
| id? | integer | Optional identifier for component |
| custom_id | string | Select menu ID; 1-100 characters |
| placeholder? | string | Placeholder shown when no value is selected; maximum 150 characters |
| default_values? | array of default value objects | Default values for auto-populated select menu components; the count must fit between `min_values` and `max_values` |
| min_values?* | integer | Fewest items required; defaults to 1, minimum 0 (see note), maximum 25 |
| max_values? | integer | Most items allowed; defaults to 1, maximum 25 |
| required?** | boolean | Whether a modal requires an answer for the user select; defaults to `true` |
| disabled?*** | boolean | Whether the select menu is disabled in a message; defaults to `false` |
