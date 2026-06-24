# Guild (part 3)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

| Field | Type | Description | Default |
| --- | --- | --- | --- |
| name | string | role name, limited to 100 characters | ”new role” |
| permissions | string | bitfield representing permissions that are enabled or disabled | @everyone permissions in guild |
| color* | integer | **Deprecated** RGB color value | 0 |
| colors | role colors object | the role’s colors | default role colors object |
| hoist | boolean | whether this role appears in its own sidebar group | false |
| icon | ?image data | role icon image, available only when the guild includes the `ROLE_ICONS` feature | null |
| unicode_emoji | ?string | standard emoji used as the role unicode emoji, available only when the guild includes the `ROLE_ICONS` feature | null |
| mentionable | boolean | whether users can mention this role | false |

 * The API continues to include `color`, but requests should prefer the `colors` field.

### Modify Guild Role Positions

  Change ordering for one or more guild [role](https://docs.discord.com/developers/topics/permissions#role-object) objects. The caller must have `MANAGE_ROLES`. On success, the response contains every [role](https://docs.discord.com/developers/topics/permissions#role-object) object in the guild. The operation emits multiple [Guild Role Update](https://docs.discord.com/developers/events/gateway-events#guild-role-update) Gateway events, accepts `X-Audit-Log-Reason`, and expects a JSON array using this parameter shape:

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | role |
| position? | ?integer | role sort position; roles sharing a position are ordered by id |

### Modify Guild Role

  Update a guild role. The request requires `MANAGE_ROLES`, returns the modified [role](https://docs.discord.com/developers/topics/permissions#role-object), and emits a [Guild Role Update](https://docs.discord.com/developers/events/gateway-events#guild-role-update) Gateway event. Each endpoint parameter is optional and nullable. `X-Audit-Log-Reason` is supported.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name | string | role name, limited to 100 characters |
| permissions | string | bitfield representing permissions that are enabled or disabled |
| color* | integer | **Deprecated** RGB color value |
| colors | role colors object | the role’s colors |
| hoist | boolean | whether this role appears in its own sidebar group |
| icon | image data | role icon image, available only when the guild includes the `ROLE_ICONS` feature |
| unicode_emoji | string | standard emoji used as the role unicode emoji, available only when the guild includes the `ROLE_ICONS` feature |
| mentionable | boolean | whether users can mention this role |

 * The API continues to include `color`, but requests should prefer the `colors` field.

### Delete Guild Role

  Remove a guild role. The caller needs `MANAGE_ROLES`. A successful deletion returns an empty `204` response, emits a [Guild Role Delete](https://docs.discord.com/developers/events/gateway-events#guild-role-delete) Gateway event, and may include the `X-Audit-Log-Reason` header.

### Get Guild Prune Count

  Return an object containing a single `pruned` key with the member count that a prune would remove. This requires both `MANAGE_GUILD` and `KICK_MEMBERS`. Pruning excludes users with roles by default. Supplying `include_roles` lets specific roles participate; inactive users whose roles are a subset of the provided role or roles are counted, while users with other additional roles are excluded.

###### Query String Params

| Field | Type | Description | Default |
| --- | --- | --- | --- |
| days | integer | days of inactivity to include in the prune count, from 1 through 30 | 7 |
| include_roles | string containing a comma-separated snowflake array | role(s) to include | none |

### Begin Guild Prune

  Start a guild prune. The caller needs `MANAGE_GUILD` and `KICK_MEMBERS`. The response has one `pruned` key with the number of removed members; for large guilds, set `compute_prune_count` to `false` so `pruned` is forced to `null`. The action emits multiple [Guild Member Remove](https://docs.discord.com/developers/events/gateway-events#guild-member-remove) Gateway events. By default, members with roles are not pruned. Use `include_roles` to include selected roles: inactive users whose role set is contained within the supplied role or roles are pruned, while users with extra roles are not. The endpoint accepts `X-Audit-Log-Reason`.

###### JSON Params

| Field | Type | Description | Default |
| --- | --- | --- | --- |
| days | integer | number of days to prune (1-30) | 7 |
| compute_prune_count | boolean | whether to include `pruned` in the response; avoid enabling this for large guilds | true |
| include_roles | array of snowflakes | role(s) to include | none |
| reason? | string | reason for the prune (deprecated) |  |

### Get Guild Voice Regions

  Return the guild's [voice region](https://docs.discord.com/developers/resources/voice#voice-region-object) objects. This differs from the related `/voice` route because VIP-enabled guilds also receive VIP servers.

### Get Guild Invites

  Return [invite](https://docs.discord.com/developers/resources/invite#invite-object) objects for the guild. The request requires either `MANAGE_GUILD` or `VIEW_AUDIT_LOG`. With `MANAGE_GUILD`, the response also contains [Invite Metadata](https://docs.discord.com/developers/resources/invite#invite-metadata-object).

### Get Guild Integrations

  Return up to 50 guild [integration](https://docs.discord.com/developers/resources/guild#integration-object) objects. `MANAGE_GUILD` is required. Integrations beyond that maximum are not accessible through this endpoint.

### Delete Guild Integration

  Remove the guild's attached [integration](https://docs.discord.com/developers/resources/guild#integration-object). Related webhooks are deleted, and any associated bot is kicked. The call requires `MANAGE_GUILD`, returns an empty `204` on success, emits [Guild Integrations Update](https://docs.discord.com/developers/events/gateway-events#guild-integrations-update) and [Integration Delete](https://docs.discord.com/developers/events/gateway-events#integration-delete) Gateway events, and supports `X-Audit-Log-Reason`.

### Get Guild Widget Settings

  Return the [guild widget settings](https://docs.discord.com/developers/resources/guild#guild-widget-settings-object) object. The caller must have `MANAGE_GUILD`.

### Modify Guild Widget

  Update the guild's [guild widget settings](https://docs.discord.com/developers/resources/guild#guild-widget-settings-object) object. Any JSON attribute can be provided and changed. The operation requires `MANAGE_GUILD`, returns the updated [guild widget settings](https://docs.discord.com/developers/resources/guild#guild-widget-settings-object) object, may emit a [Guild Update](https://docs.discord.com/developers/events/gateway-events#guild-update) Gateway event, and supports `X-Audit-Log-Reason`.

### Get Guild Widget

  Return the guild [widget](https://docs.discord.com/developers/resources/guild#guild-widget-object). If an invite channel is configured and a new [Invite](https://docs.discord.com/developers/resources/invite#invite-object) is created, an [Invite Create](https://docs.discord.com/developers/events/gateway-events#invite-create) Gateway event is emitted.

### Get Guild Vanity URL

  Return a partial [invite](https://docs.discord.com/developers/resources/invite#invite-object) object for guilds where the vanity URL feature is enabled. `MANAGE_GUILD` is required. If the guild has no configured vanity url, `code` is null. Use this endpoint for the vanity invite usage count; the invite code itself is available as `vanity_url_code` on the [guild object](https://docs.discord.com/developers/resources/guild#guild-object) without requiring `MANAGE_GUILD`.

###### Example Partial Invite Object

```
{
  "code": "abc",
  "uses": 12
}

```

### Get Guild Widget Image

  Return the guild widget as a PNG image. Authentication and permissions are not required, and every parameter is optional.

###### Query String Params

| Field | Type | Description | Default |
| --- | --- | --- | --- |
| style | string | widget image style to return; see the options below | shield |

###### Widget Style Options

| Value | Description | Example |
| --- | --- | --- |
| shield | shield-style widget showing the Discord icon and the online guild member count | Example |
| banner1 | large widget image with the guild icon, guild name, online count, and `POWERED BY DISCORD` in the footer | Example |
| banner2 | compact widget with guild icon, name, and online count, plus a right-side split area containing the Discord logo | Example |
| banner3 | large widget image showing guild icon, name, and online count, with the Discord logo on the footer's left and `Chat Now` on the right | Example |
| banner4 | widget with a large Discord logo at the top, the guild icon, name, and online count in the center, and a `JOIN MY SERVER` button at the bottom | Example |

### Get Guild Welcome Screen

  Return the guild [Welcome Screen](https://docs.discord.com/developers/resources/guild#welcome-screen-object) object. When the welcome screen is disabled, the caller must have `MANAGE_GUILD`.

### Modify Guild Welcome Screen

  Update the guild's [Welcome Screen](https://docs.discord.com/developers/resources/guild#welcome-screen-object). The caller needs `MANAGE_GUILD`. A successful call returns the updated [Welcome Screen](https://docs.discord.com/developers/resources/guild#welcome-screen-object) object and may emit a [Guild Update](https://docs.discord.com/developers/events/gateway-events#guild-update) Gateway event. All parameters are optional and nullable, and `X-Audit-Log-Reason` is supported.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| enabled | boolean | whether the welcome screen is active |
| welcome_channels | array containing welcome screen channel objects | welcome screen channels and the display settings for each one |
| description | string | server description displayed on the welcome screen |

### Get Guild Onboarding

  Return the guild [Onboarding](https://docs.discord.com/developers/resources/guild#guild-onboarding-object) object.

### Modify Guild Onboarding

  Update the guild onboarding configuration. The response is a `200` with the guild [Onboarding](https://docs.discord.com/developers/resources/guild#guild-onboarding-object) object. The caller must have `MANAGE_GUILD` and `MANAGE_ROLES`. Enabled onboarding applies constraints: at least 7 Default Channels must exist, and at least 5 of those channels must allow the @everyone role to send messages. The `mode` field changes what is evaluated for these constraints. `X-Audit-Log-Reason` is supported, and every parameter is optional.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| prompts | array of onboarding prompt objects | prompts displayed during onboarding and customize community |
| default_channel_ids | array of snowflakes | channel IDs that members are automatically opted into |
| enabled | boolean | whether guild onboarding is active |
| mode | onboarding mode | Current mode of onboarding |

### Modify Guild Incident Actions

  Update guild incident actions. A successful call returns `200` with the guild [Incidents Data](https://docs.discord.com/developers/resources/guild#incidents-data-object) object. The request requires `MANAGE_GUILD`.

###### JSON Params

 Both `invites_disabled_until` and `dms_disabled_until` may be set no later than 24 hours ahead.
| Field | Type | Description |
| --- | --- | --- |
| invites_disabled_until? | ?ISO8601 timestamp * | timestamp when invites become enabled again |
| dms_disabled_until? | ?ISO8601 timestamp * | timestamp when direct messages become enabled again |

 * Supplying `null` disables the action.
