# Components — Reference (part 2)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

* `min_values` must be omitted or set to at least `1` when `required` is omitted or `true`. ** The `required` field applies only to User Selects in modals and is ignored in messages. *** Setting `disabled` inside a modal causes an error because disabled components are not currently supported in modals.

###### Select Default Value Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | ID for a user, role, or channel |
| type | string | Value category represented by `id`; valid values are `"user"`, `"role"`, or `"channel"` |

###### User Select Interaction Response Structure

| Field | Type | Description |
| --- | --- | --- |
| type* | integer | `5` for a User Select |
| component_type* | integer | `5` for a User Select |
| id | integer | Unique identifier for the component |
| custom_id | string | Input identifier chosen by the developer; 1-100 characters |
| resolved | resolved data | Resolved entities from selected options |
| values | array of snowflakes | IDs of the selected users |

 * Message interaction responses include `component_type`; modal interaction responses include `type`.

###### Examples

Message Example

Payload for creating a message with a User Select component

(Figure: User Select example showing two people and an app in a server)

```
{
  "flags": 32768,
  "components": [
    {
      "type": 1,  // ComponentType.ACTION_ROW
      "components": [
        {
          "type": 5,  // ComponentType.USER_SELECT
          "custom_id": "user_select",
          "placeholder": "Select a user"
        }
      ]
    }
  ]
}

```

Message Interaction Data Example

A user chooses an option in a message User Select

When a user uses a User Select in a message, the interaction data payload has this basic shape. See the [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure) reference for the complete payload. If a user is selected, `resolved` can include both `members` and `users`.

```
{
  "type": 3, // InteractionType.MESSAGE_COMPONENT
  ...additionalInteractionFields, // See the Interaction documentation for all fields

  "data": {
    "component_type": 5, // ComponentType.USER_SELECT
    "id": 2,
    "custom_id": "user_select",
    "values": [
      "1111111111111111111",
    ],
    "resolved": {
      "members": {
        "1111111111111111111": {
          "avatar": null,
          "banner": null,
          "collectibles": null,
          "communication_disabled_until": null,
          "flags": 0,
          "joined_at": "2025-05-16T22:51:16.692000+00:00",
          "nick": null,
          "pending": false,
          "permissions": "2248473465835073",
          "premium_since": null,
          "roles": [
            "2222222222222222222"
          ],
          "unusual_dm_activity_until": null
        }
      },
      "users": {
        "1111111111111111111": {
          "avatar": "d54e87d20539fe9aad2f2cebe56809a2",
          "avatar_decoration_data": null,
          "bot": true,
          "clan": null,
          "collectibles": null,
          "discriminator": "9062",
          "display_name_styles": null,
          "global_name": null,
          "id": "1111111111111111111",
          "primary_guild": null,
          "public_flags": 524289,
          "username": "ExampleBot"
        }
      }
    }
  },
}

```

Modal Example

Payload for creating a modal with a User Select component

(Figure: Modal example containing a User Select)

```
{
  "type": 9,
  "data": {
    "custom_id": "user_modal",
    "title": "User Chooser",
    "components": [
      {
        "type": 18, // ComponentType.LABEL
        "label": "Choose your users",
        "component": {
          "type": 5, // ComponentType.USER_SELECT
          "custom_id": "user_selected",
          "max_values": 5,
          "required": true
        }
      }
    ]
  }
}

```

Modal Submit Interaction Data Example

A user submits a modal that includes a User Select

When a submitted modal includes a User Select, the interaction data payload follows this basic form. The complete payload is documented in the [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure) reference.

```
{
  "type": 5, // InteractionType.MODAL_SUBMIT
  ...additionalInteractionFields, // See the Interaction documentation for all fields

  "data": {
    "custom_id": "user_modal",
    "components": [
      {
        "component": {
          "custom_id": "user_selected",
          "id": 2,
          "type": 5,
          "values": [
            "11111111111111111"
          ]
        },
        "id": 1,
        "type": 18
      }
    ],
    "resolved": {
      "members": {
        "11111111111111111": {
          "avatar": null,
          "banner": null,
          "collectibles": null,
          "communication_disabled_until": null,
          "flags": 0,
          "joined_at": "2025-04-02T23:07:21.476000+00:00",
          "nick": "Ant",
          "pending": false,
          "permissions": "4503599627370495",
          "premium_since": null,
          "roles": [
            "1357409927680889032"
          ],
          "unusual_dm_activity_until": null
        }
      },
      "users": {
        "11111111111111111": {
          "avatar": "a_b15bd8ee42e3c3d9a7de129fee60bc84",
          "avatar_decoration_data": null,
          "clan": null,
          "collectibles": {
            "nameplate": {
              "asset": "nameplates/spell/white_mana/",
              "expires_at": null,
              "label": "COLLECTIBLES_SPELL_WHITE_MANA_NP_A11Y",
              "palette": "bubble_gum",
              "sku_id": "1379220459203072050"
            }
          },
          "discriminator": "0",
          "display_name_styles": {
            "colors": [
              16777215
            ],
            "effect_id": 4,
            "font_id": 3
          },
          "global_name": "Anthony",
          "id": "11111111111111111",
          "primary_guild": null,
          "public_flags": 65,
          "username": "actuallyanthony"
        }
      }
    }
  }
}

```

---

### Role Select

 A Role Select is an interactive component that lets users choose one or more roles in a message or modal. Discord fills the options automatically from roles available in the server. Role Selects support both single-select and multi-select configuration. After the user completes the selection, your app receives an [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure). Role Selects work in messages and modals. In messages they must be inside an [Action Row](https://docs.discord.com/developers/components/reference#action-row); in modals they must be inside a [Label](https://docs.discord.com/developers/components/reference#label).

###### Role Select Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `6` for role select |
| id? | integer | Optional identifier for component |
| custom_id | string | Select menu ID; 1-100 characters |
| placeholder? | string | Placeholder shown when no value is selected; maximum 150 characters |
| default_values? | array of default value objects | Default values for auto-populated select menu components; the count must fit between `min_values` and `max_values` |
| min_values?* | integer | Fewest items required; defaults to 1, minimum 0 (see note), maximum 25 |
| max_values? | integer | Most items allowed; defaults to 1, maximum 25 |
| required?** | boolean | Whether a modal requires an answer for the role select; defaults to `true` |
| disabled?*** | boolean | Whether the select menu is disabled in a message; defaults to `false` |

 * `min_values` must be omitted or set to at least `1` when `required` is omitted or `true`. ** The `required` field applies only to Role Selects in modals and is ignored in messages. *** Setting `disabled` inside a modal causes an error because disabled components are not currently supported in modals.

###### Role Select Interaction Response Structure

| Field | Type | Description |
| --- | --- | --- |
| type* | integer | `6` for a Role Select |
| component_type* | integer | `6` for a Role Select |
| id | integer | Unique identifier for the component |
| custom_id | string | Input identifier chosen by the developer; 1-100 characters |
| resolved | resolved data | Resolved entities from selected options |
| values | array of snowflakes | IDs of the selected roles |

 * Message interaction responses include `component_type`; modal interaction responses include `type`.

###### Examples

Message Example

Payload for creating a message with a Role Select component

(Figure: Role Select example allowing as many as 3 choices)

```
{
  "flags": 32768,
  "components": [
    {
      "type": 1,  // ComponentType.ACTION_ROW
      "components": [
        {
          "type": 6,  // ComponentType.ROLE_SELECT
          "custom_id": "role_ids",
          "placeholder": "Which roles?",
          "min_values": 1,
          "max_values": 3
        }
      ]
    }
  ]
}

```

Message Interaction Data Example

A user chooses an option in a message Role Select

When a user uses a Role Select in a message, the interaction data payload follows this basic structure. The complete payload is documented in the [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure) reference.

```
{
  "type": 3, // InteractionType.MESSAGE_COMPONENT
  ...additionalInteractionFields, // See the Interaction documentation for all fields

  "data": {
    "component_type": 6, // ComponentType.ROLE_SELECT
    "id": 2,
    "custom_id": "role_ids",
    "values": [
      "222222222222222222",
    ],
    "resolved": {
      "roles": {
        "222222222222222222": {
          "color": 12745742,
          "colors": {
            "primary_color": 12745742,
            "secondary_color": null,
            "tertiary_color": null
          },
          "description": null,
          "flags": 0,
          "hoist": false,
          "icon": null,
          "id": "222222222222222222",
          "managed": false,
          "mentionable": true,
          "name": "Developer",
          "permissions": "0",
          "position": 2,
          "unicode_emoji": "🔧"
        }
      }
    }
  },
}

```

Modal Example

Payload for creating a modal with a Role Select component

(Figure: Modal example containing a Role Select)

```
{
  "type": 9,
  "data": {
    "custom_id": "role_modal",
    "title": "Role Select",
    "components": [
      {
        "type": 18,
        "label": "Select which roles to assign",
        "component": {
          "type": 6,
          "custom_id": "roles_selected",
          "max_values": 10,
          "required": true
        }
      }
    ]
  }
}

```

Modal Submit Interaction Data Example

A user submits a modal that includes a Role Select

When a submitted modal includes a Role Select, the interaction data payload uses this basic shape. See the [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure) reference for the full payload.

```
{
  "type": 5, // InteractionType.MODAL_SUBMIT
  ...additionalInteractionFields, // See the Interaction documentation for all fields

  "data": {
    "custom_id": "role_modal",
    "components": [
      {
        "component": {
          "custom_id": "roles_selected",
          "id": 2,
          "type": 6,
          "values": [
            "1362213912946147499",
            "1357409927680889032"
          ]
        },
        "id": 1,
        "type": 18
      }
    ],
    "resolved": {
      "roles": {
        "1357409927680889032": {
          "color": 7419530,
          "colors": {
            "primary_color": 7419530,
            "secondary_color": null,
            "tertiary_color": null
          },
          "description": null,
          "flags": 0,
          "hoist": true,
          "icon": null,
          "id": "1357409927680889032",
          "managed": false,
          "mentionable": true,
          "name": "Player",
          "permissions": "2249596494938111",
          "position": 3,
          "unicode_emoji": "🎮"
        },
        "1362213912946147499": {
          "color": 11342935,
          "colors": {
            "primary_color": 11342935,
            "secondary_color": null,
            "tertiary_color": null
          },
          "description": null,
          "flags": 0,
          "hoist": false,
          "icon": null,
          "id": "1362213912946147499",
          "managed": false,
          "mentionable": false,
          "name": "Mod",
          "permissions": "0",
          "position": 1,
          "unicode_emoji": "🔨"
        }
      }
    }
  }
}

```

---

### Mentionable Select

 A Mentionable Select is an interactive component that lets users pick one or more mentionables in a message or modal. Discord automatically fills the options from mentionables available in the server. Mentionable Selects can be set up for single-select or multi-select behavior. When the user finishes choosing, your app receives an [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure). Mentionable Selects are supported in messages and modals. In messages they must be placed in an [Action Row](https://docs.discord.com/developers/components/reference#action-row); in modals they must be placed in a [Label](https://docs.discord.com/developers/components/reference#label).

###### Mentionable Select Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `7` for mentionable select |
| id? | integer | Optional identifier for component |
| custom_id | string | Select menu ID; 1-100 characters |
| placeholder? | string | Placeholder shown when no value is selected; maximum 150 characters |
| default_values? | array of default value objects | Default values for auto-populated select menu components; the count must fit between `min_values` and `max_values` |
| min_values?* | integer | Fewest items required; defaults to 1, minimum 0 (see note), maximum 25 |
| max_values? | integer | Most items allowed; defaults to 1, maximum 25 |
| required?** | boolean | Whether a modal requires an answer for the mentionable select; defaults to `true` |
| disabled?*** | boolean | Whether the select menu is disabled in a message; defaults to `false` |

 * `min_values` must be omitted or set to at least `1` when `required` is omitted or `true`. ** The `required` field applies only to Mentionable Selects in modals and is ignored in messages. *** Setting `disabled` inside a modal causes an error because disabled components are not currently supported in modals.

###### Mentionable Select Interaction Response Structure

| Field | Type | Description |
| --- | --- | --- |
| type* | integer | `7` for a Mentionable Select |
| component_type* | integer | `7` for a Mentionable Select |
| id | integer | Unique identifier for the component |
| custom_id | string | Input identifier chosen by the developer; 1-100 characters |
| resolved | resolved data | Resolved entities from selected options |
| values | array of snowflakes | IDs of the selected mentionables |

 * Message interaction responses include `component_type`; modal interaction responses include `type`.

###### Examples

Message Example

Payload for creating a message with a Mentionable Select component

(Figure: Mentionable Select example produced by the code below)

```
{
  "flags": 32768,
  "components": [
    {
      "type": 1,  // ComponentType.ACTION_ROW
      "components": [
        {
          "type": 7, // ComponentType.MENTIONABLE_SELECT
          "custom_id": "who_to_ping",
          "placeholder": "Who?",
        }
      ]
    }
  ]
}

```

Message Interaction Data Example

A user chooses an option in a message Mentionable Select

When a user uses a Mentionable Select in a message, the interaction data payload has this basic shape. The full payload is covered in the [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure) reference. If a user is selected, `resolved` may include both `members` and `users`.

```
{
  "type": 3, // InteractionType.MESSAGE_COMPONENT
  ...additionalInteractionFields, // See the Interaction documentation for all fields

  "data": {
    "component_type": 7, // ComponentType.MENTIONABLE_SELECT
    "id": 2,
    "custom_id": "who_to_ping",
    "values": [
      "111111111111111111",
      "222222222222222222",
    ],
    "resolved": {
      "members": {
        "1111111111111111111": {
          "avatar": null,
          "banner": null,
          "collectibles": null,
          "communication_disabled_until": null,
          "flags": 0,
          "joined_at": "2025-05-16T22:51:16.692000+00:00",
          "nick": null,
          "pending": false,
          "permissions": "2248473465835073",
          "premium_since": null,
          "roles": [
            "2222222222222222222"
          ],
          "unusual_dm_activity_until": null
        }
      },
      "users": {
        "1111111111111111111": {
          "avatar": "d54e87d20539fe9aad2f2cebe56809a2",
          "avatar_decoration_data": null,
          "bot": true,
          "clan": null,
          "collectibles": null,
          "discriminator": "9062",
          "display_name_styles": null,
          "global_name": null,
          "id": "1111111111111111111",
          "primary_guild": null,
          "public_flags": 524289,
          "username": "ExampleBot"
        }
      },
      "roles": {
        "222222222222222222": {
          "color": 12745742,
          "colors": {
            "primary_color": 12745742,
            "secondary_color": null,
            "tertiary_color": null
          },
          "description": null,
          "flags": 0,
          "hoist": false,
          "icon": null,
          "id": "222222222222222222",
          "managed": false,
          "mentionable": true,
          "name": "Developer",
          "permissions": "0",
          "position": 2,
          "unicode_emoji": "🔧"
        }
      }
    }
  },
}

```

Modal Example

Payload for creating a modal with a Mentionable Select component

(Figure: Mentionable Select example produced by the code below)

```
{
  "type": 9,
  "data": {
    "custom_id": "mentionable_modal",
    "title": "Unmentionables",
    "components": [
      {
        "type": 18,
        "label": "Who gets mentioned?",
        "component": {
          "type": 7,
          "custom_id": "mentionables_selected",
          "required": true
        }
      }
    ]
  }
}

```

Modal Submit Interaction Data Example

A user submits a modal that includes a Mentionable Select

When a submitted modal includes a Mentionable Select, the interaction data payload follows this basic structure. The complete payload is documented in the [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure) reference.

```
{
  "type": 5, // InteractionType.MODAL_SUBMIT
  ...additionalInteractionFields, // See the Interaction documentation for all fields

  "data": {
    "custom_id": "mentionable_modal",
    "components": [
      {
        "component": {
          "custom_id": "mentionables_selected",
          "id": 2,
          "type": 7,
          "values": [
            "1361539726405926952"
          ]
        },
        "id": 1,
        "type": 18
      }
    ],
    "resolved": {
      "roles": {
        "1361539726405926952": {
          "color": 12745742,
          "colors": {
            "primary_color": 12745742,
            "secondary_color": null,
            "tertiary_color": null
          },
          "description": null,
          "flags": 0,
          "hoist": false,
          "icon": null,
          "id": "1361539726405926952",
          "managed": false,
          "mentionable": true,
          "name": "Developer",
          "permissions": "0",
          "position": 2,
          "unicode_emoji": "🔧"
        }
      }
    }
  }
}

```

---

### Channel Select

 A Channel Select is an interactive component that lets users choose one or more channels in a message or modal. Discord automatically fills options from channels available in the server, and the list can be filtered by channel type. Channel Selects support single-select and multi-select configurations. After the user makes their choice, your app receives an [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure). Channel Selects work in messages and modals. In messages they must be placed in an [Action Row](https://docs.discord.com/developers/components/reference#action-row); in modals they must be placed in a [Label](https://docs.discord.com/developers/components/reference#label).

###### Channel Select Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | `8` for channel select |
| id? | integer | Optional identifier for component |
| custom_id | string | Select menu ID; 1-100 characters |
| channel_types? | array of channel types | Channel types that the channel select component should include |
| placeholder? | string | Placeholder shown when no value is selected; maximum 150 characters |
| default_values? | array of default value objects | Default values for auto-populated select menu components; the count must fit between `min_values` and `max_values` |
| min_values?* | integer | Fewest items required; defaults to 1, minimum 0 (see note), maximum 25 |
| max_values? | integer | Most items allowed; defaults to 1, maximum 25 |
| required?** | boolean | Whether a modal requires an answer for the channel select; defaults to `true` |
| disabled?*** | boolean | Whether the select menu is disabled in a message; defaults to `false` |

 * `min_values` must be omitted or set to at least `1` when `required` is omitted or `true`. ** The `required` field applies only to Channel Selects in modals and is ignored in messages. *** Setting `disabled` inside a modal causes an error because disabled components are not currently supported in modals.

###### Channel Select Interaction Response Structure

| Field | Type | Description |
| --- | --- | --- |
| type* | integer | `8` for a Channel Select |
| component_type* | integer | `8` for a Channel Select |
| id | integer | Unique identifier for the component |
| custom_id | string | Input identifier chosen by the developer; 1-100 characters |
| resolved | resolved data | Resolved entities from selected options |
| values | array of snowflakes | IDs of the selected channels |

 * Message interaction responses include `component_type`; modal interaction responses include `type`.

###### Examples

Message Example

Payload for creating a message with a Channel Select component

(Figure: Channel Select example limited to text channels)

```
{
  "flags": 32768,
  "components": [
    {
      "type": 1,  // ComponentType.ACTION_ROW
      "components": [
        {
          "type": 8,  // ComponentType.CHANNEL_SELECT
          "custom_id": "notification_channel",
          "channel_types": [0],  // ChannelType.TEXT
          "placeholder": "Which text channel?"
        }
      ]
    }
  ]
}

```

Message Interaction Data Example

A user chooses an option in a message Channel Select

When a user uses a Channel Select in a message, the interaction data payload follows this basic form. See the [interaction](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure) reference for the complete payload.
