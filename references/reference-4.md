# Reference (part 4)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

```
{
  "type": 5, // InteractionType.MODAL_SUBMIT
  ...additionalInteractionFields, // See the Interaction documentation for all fields

  "data": {
    "components": [
      {
          "component": {
              "custom_id": "file_upload",
              "id": 2,
              "type": 19,
              "values": [
                  "111111111111111111111"
              ]
          },
          "id": 1,
          "type": 18
      }
    ],
    "custom_id": "bug_submit_modal",
    "resolved": {
      "attachments": {
        "111111111111111111111": {
            "content_type": "image/png",
            "ephemeral": true,
            "filename": "bug.png",
            "height": 604,
            "id": "111111111111111111111",
            "placeholder": "/PcBAoBQydvKesabEIoMsdg=",
            "placeholder_version": 1,
            "proxy_url": "https://media.discordapp.net/ephemeral-attachments/2222222222222222222/111111111111111111111/bug.png?ex=68dc7ce1&is=68db2b61&hm=5954f90117ccf8716ffa6c7f97a778a0d039810c9584045f400d8a9fff590768&",
            "size": 241394,
            "url": "https://cdn.discordapp.com/ephemeral-attachments/2222222222222222222/111111111111111111111/bug.png?ex=68dc7ce1&is=68db2b61&hm=5954f90117ccf8716ffa6c7f97a778a0d039810c9584045f400d8a9fff590768&",
            "width": 2482
        }
      }
    }
  }
}

```

---

### Radio Group

 A Radio Group lets a user choose exactly one item from a defined option set. Radio Groups can be used in modals and must be nested inside a [Label](https://docs.discord.com/developers/components/reference#label).

### Radio Group Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `21` for radio group |
| id? | integer | Optional identifier for component |
| custom_id | string | developer-provided input identifier, from 1 to 100 characters |
| options | array of radio group options | options displayed to the user; minimum 2 and maximum 10 |
| required? | boolean | whether modal submission requires a selected option; defaults to `true` |

### Radio Group Option Structure

| Field | Type | Description |
| --- | --- | --- |
| value | string | developer-provided option value, up to 100 characters |
| label | string | option label shown to the user, up to 100 characters |
| description? | string | optional option description, up to 100 characters |
| default? | boolean | whether this option is initially selected |

### Radio Group Interaction Response Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `21` for a Radio Group |
| id | integer | Unique identifier for the component |
| custom_id | string | developer-provided input identifier, from 1 to 100 characters |
| value | ?string | selected option value, or `null` when nothing is selected |

### Examples

Modal Example

Payload for creating a modal that contains a Radio Group component

(Figure: Example of a modal with a Radio Group)

```
{
  "type": 9,
  "data": {
    "custom_id": "class_selection_modal",
    "title": "Class Selection",
    "components": [
      {
        "type": 18, // ComponentType.LABEL
        "label": "Choose your class",
        "description": "Your class detertmines the style of play for your character.",
        "component": {
          "type": 21, // ComponentType.RADIO_GROUP
          "custom_id": "class_radio",
          "options": [
            {"value": "warrior", "label": "Warrior", "description": "Strong and brave"},
            {"value": "rogue", "label": "Rogue", "description": "Weak and squishy"},
            {"value": "wizard", "label": "Wizard", "description": "Nerd"},
            {"value": "bard", "label": "Bard", "description": "Annoys everyone"},
            {"value": "witch_doctor", "label": "Witch Doctor", "description": "Actually a pretty cool option"}
          ]
        }
      }
    ]
  }
}

```

Modal Submit Interaction Data Example

User submission data for a modal that includes a Radio Group

```
{
  "type": 5, // InteractionType.MODAL_SUBMIT
  ...additionalInteractionFields, // See the Interaction documentation for all fields

  "data": {
    "custom_id": "class_selection_modal",
    "components": [
      {
        "id": 1,
        "type": 18, // ComponentType.LABEL
        "component": {
          "custom_id": "class_radio",
          "id": 2,
          "type": 21, // ComponentType.RADIO_GROUP
          "value": "warrior"
        }
      }
    ]
  }
}

```

---

### Checkbox Group

 A Checkbox Group lets a user select one or more options with checkboxes. Checkbox Groups are supported in modals and must be nested inside a [Label](https://docs.discord.com/developers/components/reference#label).

### Checkbox Group Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `22` for checkbox group |
| id? | integer | Optional identifier for component |
| custom_id | string | developer-provided input identifier, from 1 to 100 characters |
| options | array of checkbox group options | options displayed to the user; minimum 1 and maximum 10 |
| min_values?* | integer | minimum selections required; min 0, max 10, default 1; |
| max_values? | integer | maximum selections allowed; min 1, max 10, default equals the option count |
| required? | boolean | whether the group must be completed; defaults to `true` |

 * If `required` is omitted or set to `true`, `min_values` must be omitted or have a value of at least `1`.

### Checkbox Group Option Structure

| Field | Type | Description |
| --- | --- | --- |
| value | string | developer-provided option value, up to 100 characters |
| label | string | option label shown to the user, up to 100 characters |
| description? | string | optional option description, up to 100 characters |
| default? | boolean | whether this option is initially selected |

### Checkbox Group Interaction Response Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `22` for a Checkbox Group |
| id | integer | Unique identifier for the component |
| custom_id | string | developer-provided input identifier, from 1 to 100 characters |
| values | array of strings | selected option values, or an empty array `[]` if none are selected |

### Examples

Modal Example

Payload for creating a modal that contains a Checkbox Group component

(Figure: Example of a modal with a Checkbox Group)

```
{
  "type": 9,
  "data": {
    "custom_id": "day_selection_modal",
    "title": "Study Days",
    "components": [
      {
        "type": 18, // ComponentType.LABEL
        "label": "Which days are you free?",
        "description": "Choose all of the days you're able to meet up.",
        "component": {
          "type": 22, // ComponentType.CHECKBOX_GROUP
          "custom_id": "event_checkbox",
          "options": [
            {"value": "march-4", "label": "March 4th"},
            {"value": "march-5", "label": "March 5th"},
            {"value": "march-7", "label": "March 7th", "description": "I know this is a Saturday and is tough"},
            {"value": "march-9", "label": "March 9th"},
            {"value": "march-10", "label": "March 10th"}
          ]
        }
      }
    ]
  }
}

```

Modal Submit Interaction Data Example

User submission data for a modal that includes a Checkbox Group

```
{
  "type": 5, // InteractionType.MODAL_SUBMIT
  ...additionalInteractionFields, // See the Interaction documentation for all fields

  "data": {
    "custom_id": "day_selection_modal",
    "components": [
      {
        "id": 1,
        "type": 18, // ComponentType.LABEL
        "component": {
          "custom_id": "event_checkbox",
          "id": 2,
          "type": 22, // ComponentType.CHECKBOX_GROUP
          "values": [
            "march-5",
            "march-10",
            "march-4"
          ]
        }
      }
    ]
  }
}

```

---

### Checkbox

 A Checkbox is one interactive control for yes/no-style input. Checkboxes are supported in modals and must be nested inside a [Label](https://docs.discord.com/developers/components/reference#label).

### Checkbox Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `23` for checkbox |
| id? | integer | Optional identifier for component |
| custom_id | string | developer-provided input identifier, from 1 to 100 characters |
| default? | boolean | whether the checkbox starts checked |

 A checkbox cannot be marked required directly. For similar behavior, use a [Checkbox Group](https://docs.discord.com/developers/components/reference#checkbox-group) containing one option and set `required`.

### Checkbox Interaction Response Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `23` for a Checkbox |
| id | integer | Unique identifier for the component |
| custom_id | string | developer-provided input identifier, from 1 to 100 characters |
| value | boolean | checkbox state: `true` when checked, `false` when unchecked |

### Examples

Modal Example

Payload for creating a modal that contains a Checkbox component

(Figure: Example of a modal with a Checkbox)

```
{
  "type": 9,
  "data": {
    "custom_id": "secret_note_modal",
    "title": "Secret Note",
    "components": [
      {
        "type": 18, // ComponentType.LABEL
        "label": "Do you like me?",
        "description": "😳😳😳",
        "component": {
          "type": 23, // ComponentType.CHECKBOX
          "custom_id": "like_checkbox"
        }
      }
    ]
  }
}

```

Modal Submit Interaction Data Example

User submission data for a modal that includes a Checkbox

```
{
  "type": 5, // InteractionType.MODAL_SUBMIT
  ...additionalInteractionFields, // See the Interaction documentation for all fields

  "data": {
    "custom_id": "secret_note_modal",
    "components": [
      {
        "id": 1,
        "type": 18, // ComponentType.LABEL
        "component": {
          "custom_id": "like_checkbox",
          "id": 2,
          "type": 23, // ComponentType.CHECKBOX
          "value": true
        }
      }
    ]
  }
}

```

---

### Unfurled Media Item

 An Unfurled Media Item represents media by URL for use inside a component. It may refer to media uploaded to Discord or to an external asset by **a direct link**. The structure below shows the complete object, but developers can set only `url` in requests using this structure. Discord fills every other field automatically.

###### Unfurled Media Item Structure

| Field | Type | Description |
| --- | --- | --- |
| url | string | Supports arbitrary urls and `attachment://<filename>` references |
| proxy_url? * | string | proxied URL for the media item |
| height? * | ?integer | media item height, when the media is an image or video |
| width? * | ?integer | media item width, when the media is an image or video |
| placeholder? * | string | thumbhash placeholder, when the media is an image or video |
| placeholder_version? * | integer | placeholder version, when the media is an image or video |
| content_type? * | string | content media type |
| flags? * | integer | bitfield combining Unfurled media item flags |
| attachment_id? * ** | snowflake | uploaded attachment id |

 * The API ignores this field in requests and supplies it in responses. ** Present only when the media item came from an uploaded attachment.

###### Unfurled Media Item Flags

| Flag | Value | Description |
| --- | --- | --- |
| IS_ANIMATED | `1 << 0` | This image is animated |

### Uploading a file

 To attach a file to a message, send the request as `multipart/form-data` instead of `application/json`, and include the file with a valid filename in the payload. Upload details and examples are in the [API Reference](https://docs.discord.com/developers/reference#uploading-files).

### Legacy Message Component Behavior

 Before `IS_COMPONENTS_V2` was introduced ([see changelog](https://docs.discord.com/developers/change-log#introducing-new-components-for-messages)), message components were sent together with message content. Apps could send messages with some available components without setting `IS_COMPONENTS_V2`; those components appeared alongside `content` and `embeds`. Components on pre-components V2 messages also have an `id` of `0`. Apps using this Legacy Message Component behavior still function, but new apps or features should prefer `IS_COMPONENTS_V2` for richer layout and customization. In legacy messages, the top-level component list can contain as many as 5 action rows. Legacy Message Component Example

```
{
  "content": "This is a message with legacy components",
  "components": [
    {
      "type": 1,
      "components": [
        {
          "type": 2,
          "style": 1,
          "label": "Click Me",
          "custom_id": "click_me_1"
        }
      ]
    }
  ]
}

```
