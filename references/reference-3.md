# Reference (part 3)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

```
{
  "type": 3, // InteractionType.MESSAGE_COMPONENT
  ...additionalInteractionFields, // See the Interaction documentation for all fields

  "data": {
    "component_type": 8, // ComponentType.CHANNEL_SELECT
    "id": 2,
    "custom_id": "notification_channel",
    "values": [
      "333333333333333333",
    ],
    "resolved": {
      "channels": {
        "333333333333333333": {
          "flags": 0,
          "guild_id": "44444444444444444",
          "id": "333333333333333333",
          "last_message_id": null,
          "name": "playtesting",
          "nsfw": false,
          "parent_id": "5555555555555555",
          "permissions": "4503599627370495",
          "position": 1,
          "rate_limit_per_user": 0,
          "topic": null,
          "type": 0  // ChannelType.TEXT
        }
      }
    }
  },
}

```

Modal Example

Modal create payload with a Channel Select component

(Figure: Example of a modal with a Channel Select)

```
{
  "type": 9,
  "data": {
    "custom_id": "channel_modal",
    "title": "Lockdown",
    "components": [
      {
        "type": 18,
        "label": "Which channel should be locked?",
        "component": {
          "type": 8,
          "custom_id": "channel_selected",
          "required": true
        }
      }
    ]
  }
}

```

Modal Submit Interaction Data Example

Once one user submits a modal covering a Channel Select

Once one user submits a modal that includes a Channel Select, this is the basic form for the interaction data payload you receive. The full payload is available within the [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure) reference.

```
{
  "type": 5, // InteractionType.MODAL_SUBMIT
  ...additionalInteractionFields, // See the Interaction documentation for all fields

  "data": {
    "custom_id": "channel_modal",
    "components": [
      {
        "component": {
          "custom_id": "channel_selected",
          "id": 2,
          "type": 8,
          "values": [
            "1357483683627663450"
          ]
        },
        "id": 1,
        "type": 18
      }
    ],
    "resolved": {
      "channels": {
        "1357483683627663450": {
          "flags": 0,
          "guild_id": "1111111111111111",
          "id": "1357483683627663450",
          "last_message_id": null,
          "name": "playtesting",
          "nsfw": false,
          "parent_id": "1357129309164404938",
          "permissions": "4503599627370495",
          "position": 1,
          "rate_limit_per_user": 0,
          "topic": null,
          "type": 0
        }
      }
    }
  }
}

```

---

### Section

 A Section acts as a top-level layout component that permits you to contextually associate content with an accessory component. This typical use-case is to contextually associate [text content](https://docs.discord.com/developers/components/reference#text-display) with one [accessory](https://docs.discord.com/developers/components/reference#section-section-accessory-components). Sections are currently works only in messages. Messages can use this component only once they include the [message flag](https://docs.discord.com/developers/resources/message#message-object-message-flags) `1 << 15` (IS_COMPONENTS_V2) and that flag is enabled per message.

###### Section Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `9` for section component |
| id? | integer | Optional identifier for component |
| components | array of section child components | One to three child components representing the content for the section that is contextually associated to that accessory |
| accessory | section accessory component | A component that is contextually associated to that content for the section |

 Don’t hardcode `components` to contain only text current components. We may add other components within the future. Similarly, `accessory` may be expanded to include other components within the future.

###### Section Child Components

| Available Components |
| --- |
| Text Display |

###### Section Accessory Components

| Available Components |
| --- |
| Button |
| Thumbnail |

###### Examples

Message Example

Message create payload with a Section (and Thumbnail) component

(Figure: Example of a Section showing one fake game changelog and a thumbnail)

```
{
  "flags": 32768,
  "components": [
    {
      "type": 9,  // ComponentType.SECTION
      "components": [
        {
          "type": 10,  // ComponentType.TEXT_DISPLAY
          "content": "# Real Game v7.3"
        },
        {
          "type": 10,  // ComponentType.TEXT_DISPLAY
          "content": "Hope you're excited, the update is finally here! Here are some of the changes:\n- Fixed a bug where certain treasure chests wouldn't open properly\n- Improved server stability during peak hours\n- Added a new type of gravity that will randomly apply when the moon is visible in-game\n- Every third thursday the furniture will scream your darkest secrets to nearby npcs"
        },
        {
          "type": 10,  // ComponentType.TEXT_DISPLAY
          "content": "-# That last one wasn't real, but don't use voice chat near furniture just in case..."
        }
      ],
      "accessory": {
        "type": 11,  // ComponentType.THUMBNAIL
        "media": {
          "url": "https://websitewithopensourceimages/gamepreview.webp"
        }
      }
    }
  ]
}

```

---

### Text Display

 A Text Display is a content component that permits you to add markdown formatted text, covering mentions (users, roles, etc) and emojis. This behavior of this component is extremely similar to that [`content` field of a message](https://docs.discord.com/developers/resources/message#message-object), but permits you to add multiple text components, controlling the layout of your message. Once delivered in a message, pingable mentions (@user, @role, etc) present in this component can ping and send notifications based on the value for this [allowed mention object](https://docs.discord.com/developers/resources/message#allowed-mentions-object) set inside [`message.allowed_mentions`](https://docs.discord.com/developers/resources/message#message-object). Messages can use this component only once they include the [message flag](https://docs.discord.com/developers/resources/message#message-object-message-flags) `1 << 15` (IS_COMPONENTS_V2) and that flag is enabled per message.

###### Text Display Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `10` for text display |
| id? | integer | Optional identifier for component |
| content | string | Text that may be displayed similar to a message |

###### Text Display Interaction Response Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `10` for a Text Display |
| id | integer | Unique identifier for the component |

###### Examples

Message Example

Message create payload with a Text Display component

(Figure: Example of a Text Display with markdown)

```
{
  "flags": 32768,
  "components": [
    {
      "type": 10,  // ComponentType.TEXT_DISPLAY
      "content": "# This is a Text Display\nAll the regular markdown rules apply\n- You can make lists\n- You can use `code blocks`\n- You can use [links](http://watchanimeattheoffice.com/)\n- Even :blush: :star_struck: :exploding_head:\n- Spoiler alert: ||these too!||"
    }
  ]
}

```

Modal Example

Modal create payload with a Text Display component

(Figure: Example of a Text Display from that code below)

```
{
  "type": 9,
  "data": {
    "custom_id": "jail_modal",
    "title": "Jail",
    "components": [
      {
        "type": 10,
        "content": "This action will move the selected user to the selected voice channel and take away all their permissions **for 1 hour**."
      },
      {
        "type": 18,
        "label": "Choose a user",
        "component": {
          "type": 5,
          "custom_id": "user_selected",
          "required": true
        }
      },
      {
        "type": 18,
        "label": "Where should they be sent?",
        "component": {
          "type": 8,
          "custom_id": "channel_selected",
          "channel_types": [
            2
          ],
          "required": true
        }
      }
    ]
  }
}

```

---

### Thumbnail

 A Thumbnail presents a content component that displays visual media in one small form-factor. It is intended as one accessory for to other content, plus is primarily usable with [sections](https://docs.discord.com/developers/components/reference#section). This media displayed is defined by this [unfurled media item](https://docs.discord.com/developers/components/reference#unfurled-media-item) structure, which supports both uploaded media plus externally hosted media. Thumbnails are currently works only in messages as an accessory in one [section](https://docs.discord.com/developers/components/reference#section). Thumbnails currently only support images, covering animated formats like GIF plus WEBP. Videos are not supported at the this time. To use this component, you need to send this [message flag](https://docs.discord.com/developers/resources/message#message-object-message-flags) `1 << 15` (IS_COMPONENTS_V2), and that flag is enabled per message.

###### Thumbnail Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `11` for thumbnail component |
| id? | integer | Optional identifier for component |
| media | unfurled media item | A url or attachment provided as an unfurled media item |
| description? | ?string | Alt text for the media, max 1024 characters |
| spoiler? | boolean | Whether the thumbnail must be a spoiler (or blurred out). Default value is `false` |

###### Examples

Message Example

Message create payload with a Thumbnail component (via Section)

(Figure: Example of a Thumbnail in a Section from that code below)

```
{
  "flags": 32768,
  "components": [
    {
      "type": 9,  // ComponentType.SECTION
      "components": [
        {
          "type": 10,  // ComponentType.TEXT_DISPLAY
          "content": "# Real Game v7.3"
        },
        {
          "type": 10,  // ComponentType.TEXT_DISPLAY
          "content": "Hope you're excited, the update is finally here! Here are some of the changes:\n- Fixed a bug where certain treasure chests wouldn't open properly\n- Improved server stability during peak hours\n- Added a new type of gravity that will randomly apply when the moon is visible in-game\n- Every third thursday the furniture will scream your darkest secrets to nearby npcs"
        },
        {
          "type": 10,  // ComponentType.TEXT_DISPLAY
          "content": "-# That last one wasn't real, but don't use voice chat near furniture just in case..."
        }
      ],
      "accessory": {
        "type": 11,  // ComponentType.THUMBNAIL
        "media": {
          "url": "https://websitewithopensourceimages/gamepreview.webp"
        }
      }
    }
  ]
}

```

---

### Media Gallery

 A Media Gallery provides a top-level content component that permits you to display 1-10 media attachments in one organized gallery format. Each item can have optional descriptions and may be marked as spoilers. Media Galleries are currently works only in messages. Messages can use this component only once they include the [message flag](https://docs.discord.com/developers/resources/message#message-object-message-flags) `1 << 15` (IS_COMPONENTS_V2) and that flag is enabled per message.

###### Media Gallery Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `12` for media gallery component |
| id? | integer | Optional identifier for component |
| items | array of media gallery items | 1 to 10 media gallery items |

###### Media Gallery Item Structure

| Field | Type | Description |
| --- | --- | --- |
| media | unfurled media item | A url or attachment provided as an unfurled media item |
| description? | ?string | Alt text for the media, max 1024 characters |
| spoiler? | boolean | Whether the media must be a spoiler (or blurred out). Default value is `false` |

###### Examples

Message Example

Message create payload with a Media Gallery component

(Figure: Example of a Media Gallery showing screenshots from live webcam feeds)

```
{
  "flags": 32768,
  "components": [
    {
      "type": 10,  // ComponentType.TEXT_DISPLAY
      "content": "Live webcam shots as of 18-04-2025 at 12:00 UTC"
    },
    {
      "type": 12,  // ComponentType.MEDIA_GALLERY
      "items": [
        {
          "media": {"url": "https://livevideofeedconvertedtoimage/webcam1.webp"},
          "description": "An aerial view looking down on older industrial complex buildings. The main building is white with many windows and pipes running up the walls."
        },
        {
          "media": {"url": "https://livevideofeedconvertedtoimage/webcam2.webp"},
          "description": "An aerial view of old broken buildings. Nature has begun to take root in the rooftops. A portion of the middle building's roof has collapsed inward. In the distant haze you can make out a far away city."
        },
        {
          "media": {"url": "https://livevideofeedconvertedtoimage/webcam3.webp"},
          "description": "A street view of a downtown city. Prominently in photo are skyscrapers and a domed building"
        }
      ]
    }
  ]
}

```

---

### File

 A File provides a top-level content component that permits you to display one [uploaded file](https://docs.discord.com/developers/components/reference#uploading-a-file) as an attachment to that message and reference it within this component. Each file component may only display 1 attached file, but you can upload multiple files plus add them to different file components within your payload. Files are currently works only in messages. The File component only supports using this `attachment://` protocol in [unfurled media item](https://docs.discord.com/developers/components/reference#unfurled-media-item) Messages can use this component only once they include the [message flag](https://docs.discord.com/developers/resources/message#message-object-message-flags) `1 << 15` (IS_COMPONENTS_V2) and that flag is enabled per message.

###### File Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `13` for a file component |
| id? | integer | Optional identifier for component |
| file | unfurled media item | This unfurled media item is unique inside that it **only** supports attachment references using this `attachment://<filename>` syntax |
| spoiler? | boolean | Whether the media must be a spoiler (or blurred out). Default value is `false` |
| name? | string | The name for the file. This field is ignored and provided by this API as part for the response |
| size? | integer | The size for the file in bytes. This field is ignored and provided by this API as part for the response |

###### Examples

Message Example

Message create payload with a File component

(Figure: Example of a File showing a download for one game and manual)This example makes use for this `attachment://` protocol functionality in [unfurled media item](https://docs.discord.com/developers/components/reference#unfurled-media-item).

```
{
  "flags": 32768,
  "components": [
    {
      "type": 10,  // ComponentType.TEXT_DISPLAY
      "content": "# New game version released for testing!\nGrab the game here:"
    },
    {
      "type": 13,  // ComponentType.FILE
      "file": {
        "url": "attachment://game.zip"
      }
    },
    {
      "type": 10,  // ComponentType.TEXT_DISPLAY
      "content": "Latest manual artwork here:"
    },
    {
      "type": 13,  // ComponentType.FILE
      "file": {
        "url": "attachment://manual.pdf"
      }
    }
  ]
}

```

---

### Separator

 A Separator provides one top-level layout component that adds vertical padding plus visual division between other components. Separators are currently works only in messages. Messages can use this component only once they include the [message flag](https://docs.discord.com/developers/resources/message#message-object-message-flags) `1 << 15` (IS_COMPONENTS_V2) and that flag is enabled per message.

###### Separator Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `14` for separator component |
| id? | integer | Optional identifier for component |
| divider? | boolean | Whether a visual divider must be displayed within the component. Default value is `true` |
| spacing? | integer | Size for separator padding—`1` for small padding, `2` for this large padding. Defaults to `1` |

###### Examples

Message Example

Message create payload with a Separator component

(Figure: Example of a separator with large spacing dividing content)

```
{
  "flags": 32768,
  "components": [
    {
      "type": 10,  // ComponentType.TEXT_DISPLAY
      "content": "It's dangerous to go alone!"
    },
    {
      "type": 14,  // ComponentType.SEPARATOR
      "divider": true,
      "spacing": 1
    },
    {
      "type": 10,  // ComponentType.TEXT_DISPLAY
      "content": "Take this."
    }
  ]
}

```

---

### Container

 A Container is a top-level layout component. Containers offer this ability to visually encapsulate one collection of components and have one optional customizable accent color bar. Containers are currently works only in messages. Messages can use this component only once they include the [message flag](https://docs.discord.com/developers/resources/message#message-object-message-flags) `1 << 15` (IS_COMPONENTS_V2) and that flag is enabled per message.

###### Container Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `17` for container component |
| id? | integer | Optional identifier for component |
| components | array of container child components | Child components that are encapsulated within the Container |
| accent_color? | ?integer | Color for the accent on this container as RGB from `0x000000` to `0xFFFFFF` |
| spoiler? | boolean | Whether the container must be a spoiler (or blurred out). Default value is `false`. |

###### Container Child Components

| Available Components |
| --- |
| Action Row |
| Text Display |
| Section |
| Media Gallery |
| Separator |
| File |

###### Examples

Message Example

Message create payload with a Container component

(Figure: Example of one container showing text, image, and buttons for a wild enemy encounter)

```
{
  "flags": 32768,
  "components": [
    {
      "type": 17,  // ComponentType.CONTAINER
      "accent_color": 703487,
      "components": [
        {
          "type": 10,  // ComponentType.TEXT_DISPLAY
          "content": "# You have encountered a wild coyote!"
        },
        {
          "type": 12,  // ComponentType.MEDIA_GALLERY
          "items": [
            {
              "media": {"url": "https://websitewithopensourceimages/coyote.webp"},
            }
          ]
        },
        {
          "type": 10,  // ComponentType.TEXT_DISPLAY
          "content": "What would you like to do?"
        },
        {
          "type": 1,  // ComponentType.ACTION_ROW
          "components": [
            {
              "type": 2,  // ComponentType.BUTTON
              "custom_id": "pet_coyote",
              "label": "Pet it!",
              "style": 1
            },
            {
              "type": 2,  // ComponentType.BUTTON
              "custom_id": "feed_coyote",
              "label": "Attempt to feed it",
              "style": 2
            },
            {
              "type": 2,  // ComponentType.BUTTON
              "custom_id": "run_away",
              "label": "Run away!",
              "style": 4
            }
          ]
        }
      ]
    }
  ]
}

```

---

### Label

 A Label is one top-level layout component. Labels wrap modal components with text as one label and optional description. This `description` may display above or below the `component` depending on the platform.

###### Label Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `18` for a label |
| id? | integer | Optional identifier for component |
| label | string | The label text; max 45 characters |
| description? | string | An optional description text for the label; max 100 characters |
| component | label child component | The component within the label |

###### Label Child Components

| Available Components |
| --- |
| Text Input |
| String Select |
| User Select |
| Role Select |
| Mentionable Select |
| Channel Select |
| File Upload |
| Radio Group |
| Checkbox Group |
| Checkbox |

###### Label Interaction Response Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `18` for a Label |
| id | integer | Unique identifier for the component |
| component | label interaction response child component | The component within the label |

###### Label Interaction Response Child Components

| Available Components |
| --- |
| Text Input |
| String Select |
| User Select |
| Role Select |
| Mentionable Select |
| Channel Select |
| File Upload |
| Radio Group |
| Checkbox Group |
| Checkbox |

###### Examples

Modal Example

Modal create payload with a Label component (wrapping a Text Input)

(Figure: A modal with Text Input in a Label)

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

---

### File Upload

 File Upload is an interactive component that permits user accounts to upload files in modals. File Uploads may be configured to have a minimum and maximum count of files between 0 and 10, along with `required` for once the upload is required to submit the modal. The max file size one user can upload is based on this user’s upload limit inside which channel. File Uploads are available on modals. They must be placed inside one [Label](https://docs.discord.com/developers/components/reference#label).

###### File Upload Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `19` for file upload |
| id? | integer | Optional identifier for component |
| custom_id | string | ID for the file upload; 1-100 characters |
| min_values?* | integer | Minimum count of items that must be uploaded (default is 1); min 0 (see note), max 10 |
| max_values? | integer | Maximum count of items that may be uploaded (default is 1); max 10 |
| required? | boolean | Whether this file upload requires files to be uploaded before submitting the modal (default is `true`) |

 * `min_values` must be either omitted or at least `1` once `required` is omitted or `true`.

###### File Upload Interaction Response Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `19` for a File Upload |
| id | integer | Unique identifier for the component |
| custom_id | string | Developer-defined identifier for the input; 1-100 characters |
| values | array of snowflakes | IDs for the uploaded files found within the resolved data |

###### Examples

Modal Example

Modal create payload with a File Upload component

(Figure: Example of a modal with a File Upload)

```
{
  "type": 9,
  "data": {
    "custom_id": "bug_submit_modal",
    "title": "Bug Submission",
    "components": [
      {
        "type": 18, // ComponentType.LABEL
        "label": "File Upload",
        "description": "Please upload a screenshot or other image that shows the bug you encountered.",
        "component": {
          "type": 19, // ComponentType.FILE_UPLOAD
          "custom_id": "file_upload",
          "min_values": 1,
          "max_values": 10,
          "required": true
        }
      }
    ]
  }
}

```

Modal Submit Interaction Data Example

Once one user submits a modal covering a File Upload

Once one user submits a modal that includes a File Upload, this is the basic form for the interaction data payload you receive. The full payload is available within the [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure) reference.
