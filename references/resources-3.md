# Guild Scheduled Event, Guild Template

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: Guild Scheduled Event Object; Guild Template Object.

## Guild Scheduled Event Object

Describes a scheduled event in a [guild](https://docs.discord.com/developers/resources/guild).

### Guild Scheduled Event Object

###### Guild Scheduled Event Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | scheduled event id |
| guild_id | snowflake | guild id that owns the scheduled event |
| channel_id ** | ?snowflake | channel id where the scheduled event is hosted, or `null` when the scheduled entity type is `EXTERNAL` |
| creator_id? * | ?snowflake | id of the user who created the scheduled event * |
| name | string | scheduled event name (1-100 characters) |
| description? | ?string | scheduled event description (1-1000 characters) |
| scheduled_start_time | ISO8601 timestamp | time when the scheduled event starts |
| scheduled_end_time ** | ?ISO8601 timestamp | time when the scheduled event ends; required when entity_type is `EXTERNAL` |
| privacy_level | privacy level | scheduled event privacy level |
| status | event status | scheduled event status |
| entity_type | scheduled entity type | scheduled event type |
| entity_id | ?snowflake | id of the entity tied to the guild scheduled event |
| entity_metadata ** | ?entity metadata | extra metadata for the guild scheduled event |
| creator? | user object | user who created the scheduled event |
| user_count? | integer | count of users subscribed to the scheduled event |
| image? | ?string | scheduled event cover image hash |
| recurrence_rule | ?recurrence rule | rule that defines how often the event recurs |

 * `creator_id` is null and `creator` is omitted for events created before October 25th, 2021, when `creator_id` was introduced and began being tracked. ** See [field requirements by entity type](https://docs.discord.com/developers/resources/guild-scheduled-event#guild-scheduled-event-object-field-requirements-by-entity-type) for how `entity_type` relates to `channel_id`, `entity_metadata`, and `scheduled_end_time`.

###### Guild Scheduled Event Privacy Level

| Level | Value | Description |
| --- | --- | --- |
| GUILD_ONLY | 2 | scheduled event is visible only to guild members |

###### Guild Scheduled Event Entity Types

| Type | Value |
| --- | --- |
| STAGE_INSTANCE | 1 |
| VOICE | 2 |
| EXTERNAL | 3 |

###### Field Requirements By Entity Type

 This table defines requirements by current entity type. `value` means the field must be non-null, `null` means the field must be null, and `-` means there is no strict requirement.
| Entity Type | channel_id | entity_metadata | scheduled_end_time |
| --- | --- | --- | --- |
| STAGE_INSTANCE | value | null | - |
| VOICE | value | null | - |
| EXTERNAL | null | value * | value |

 * Provide `entity_metadata` with a non-null `location`

###### Guild Scheduled Event Status

| Type | Value |
| --- | --- |
| SCHEDULED | 1 |
| ACTIVE | 2 |
| COMPLETED * | 3 |
| CANCELED * | 4 |

 * After `status` becomes `COMPLETED` or `CANCELED`, `status` cannot be updated again

###### Valid Guild Scheduled Event Status Transitions

 SCHEDULED —> ACTIVE ACTIVE --------> COMPLETED SCHEDULED —> CANCELED

###### Guild Scheduled Event Entity Metadata

| Field | Type | Description |
| --- | --- | --- |
| location? * | string | location of the event (1-100 characters) |

 * [required](https://docs.discord.com/developers/resources/guild-scheduled-event#guild-scheduled-event-object-guild-scheduled-event-entity-metadata) for events where `'entity_type': EXTERNAL`

### Guild Scheduled Event User Object

###### Guild Scheduled Event User Structure

| Field | Type | Description |
| --- | --- | --- |
| guild_scheduled_event_id | snowflake | scheduled event id the user subscribed to |
| user | user | user subscribed to the event |
| member? | guild member | guild member data for this user in the guild that owns the event, when present |

### Guild Scheduled Event Recurrence Rule Object

 Discord recurrence rules use a limited subset of behavior from the [iCalendar RFC](https://datatracker.ietf.org/doc/html/rfc5545) and [python's dateutil rrule](https://dateutil.readthedocs.io/en/stable/rrule.html). The system currently has many constraints; see System limitations below.

###### Guild Scheduled Event Recurrence Rule Structure

| Field | Type | Description |
| --- | --- | --- |
| start | ISO8601 timestamp | start time for the recurrence interval |
| end [1] | ?ISO8601 timestamp | end time for the recurrence interval |
| frequency | recurrence rule - frequency object | How often the event occurs |
| interval | int | spacing between events, as defined by `frequency`; for example, `frequency` of `WEEKLY` with an `interval` of `2` means every-other week |
| by_weekday | ?array of recurrence rule - weekday | specific weekdays on which the event recurs |
| by_n_weekday | ?array of recurrence rule - n_weekday objects | specific days in a specific week (1-5) on which the event recurs |
| by_month | ?array of recurrence rule - month | specific months in which the event recurs |
| by_month_day | ?array of int | specific dates in a month on which the event recurs |
| by_year_day [1] | ?array of int | specific days in a year on which the event recurs (1-364) |
| count [1] | ?int | total number of recurrences allowed before the event stops |

 [1] - Cannot be set externally currently.

System limitations

Recurrence rules are powerful, but Discord applies specific restrictions.

These limitations exist because recurring event data must be displayed in the client. Discord may loosen or remove these restrictions in the future.

###### The following fields cannot be set by the client / application

- `count`
- `end`
- `by_year_day`

###### The following combinations are mutually exclusive

- `by_weekday`
- `by_n_weekday`
- `by_month` + `by_month_day`

###### `by_weekday`

- Valid only for daily and weekly events (`frequency` of `DAILY` (`0`) or `WEEKLY` (`1`))
- For daily events (`frequency` is `DAILY` (`0`))
  - Values in the `by_weekday` event must match a known set of weekdays.
  - Current allowed sets are:
    - Monday - Friday (`[0, 1, 2, 3, 4]`)
    - Tuesday - Saturday (`[1, 2, 3, 4, 5]`)
    - Sunday - Thursday (`[6, 0, 1, 2, 3]`)
    - Friday & Saturday (`[4, 5]`)
    - Saturday & Sunday (`[5, 6]`)
    - Sunday & Monday (`[6, 0]`)

- For weekly events (`frequency` is `WEEKLY` (`1`))
  - The `by_weekday` array currently can only have a length of `1`
    - Meaning: a recurring weekly event can be placed on only a single day within the week
    - To recur on multiple days within a week, use a `frequency` of `DAILY`

  - See `interval` below for every-other week behavior

###### `by_n_weekday`

- Valid only for monthly events (`frequency` of `MONTHLY` (`1`))
- The `by_n_weekday` array currently can only have a length of `1`
  - Meaning: a recurring monthly event can be placed on only a single day within the month

###### `by_month` and `by_month_day`

- Valid only for annual events (`frequency` is `YEARLY` (`0`))
- `by_month` and `by_month_day` must both be provided
- The `by_month` and `by_month_day` arrays must each have a length of `1`
  - Meaning: annual events can use only a single date

###### `interval` can only be set to a value other than `1` when `frequency` is set to `WEEKLY` (`1`)

- Under that condition, `interval` may be set to `2`
- This allows every-other week events
- Because of `by_weekday` limits, every-other week scheduling can target only a single day.

Examples

Examples of common Recurrent Rules:

**Every weekday**

```
frequency = 3 // Frequency.DAILY
interval = 1
by_weekday = [0, 1, 2, 3, 4] // [Weekday.MONDAY, ..., Weekday.FRIDAY]

```

**Every Wednesday**

```
frequency = 2 // Frequency.WEEKLY
interval = 1
by_weekday = [2] // [Weekday.WEDNESDAY]

```

**Every other Wednesday**

```
frequency = 2 // Frequency.WEEKLY
interval = 2
by_weekday = [2] // [Weekday.WEDNESDAY]

```

**Monthly on the fourth Wednesday**

```
frequency = 1 // Frequency.MONTHLY
interval = 1
by_n_weekday = [{
  n: 4,
  day: 2 // Weekday.WEDNESDAY
}]

```

**Annually on July 24**

```
frequency = 0 // Frequency.YEARLY
interval = 1
by_month = [7] // [Month.JULY]
by_month_day = [24]

```

###### Guild Scheduled Event Recurrence Rule - Frequency

| Type | Value |
| --- | --- |
| YEARLY | 0 |
| MONTHLY | 1 |
| WEEKLY | 2 |
| DAILY | 3 |

###### Guild Scheduled Event Recurrence Rule - Weekday

| Type | Value |
| --- | --- |
| MONDAY | 0 |
| TUESDAY | 1 |
| WEDNESDAY | 2 |
| THURSDAY | 3 |
| FRIDAY | 4 |
| SATURDAY | 5 |
| SUNDAY | 6 |

###### Guild Scheduled Event Recurrence Rule - N_Weekday Structure

| Field | Type | Description |
| --- | --- | --- |
| n | int | The week to reoccur on. 1 - 5 |
| day | recurrence rule - weekday | day within the week when recurrence happens |

###### Guild Scheduled Event Recurrence Rule - Month

| Type | Value |
| --- | --- |
| JANUARY | 1 |
| FEBRUARY | 2 |
| MARCH | 3 |
| APRIL | 4 |
| MAY | 5 |
| JUNE | 6 |
| JULY | 7 |
| AUGUST | 8 |
| SEPTEMBER | 9 |
| OCTOBER | 10 |
| NOVEMBER | 11 |
| DECEMBER | 12 |

### List Scheduled Events for Guild

  Returns [guild scheduled event](https://docs.discord.com/developers/resources/guild-scheduled-event#guild-scheduled-event-object) objects for the specified guild.

###### Query String Params

| Field | Type | Description |
| --- | --- | --- |
| with_user_count? | boolean | whether to include the user subscription count for each event |

### Create Guild Scheduled Event

  Creates a guild scheduled event in the guild. On success, returns a [guild scheduled event](https://docs.discord.com/developers/resources/guild-scheduled-event#guild-scheduled-event-object) object. Also fires a [Guild Scheduled Event Create](https://docs.discord.com/developers/events/gateway-events#guild-scheduled-event-create) Gateway event. A guild can have no more than 100 events with `SCHEDULED` or `ACTIVE` status at once. This endpoint supports the `X-Audit-Log-Reason` header.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| channel_id? * | snowflake * | scheduled event channel id |
| entity_metadata? ** | entity metadata | scheduled event entity metadata |
| name | string | scheduled event name |
| privacy_level | privacy level | scheduled event privacy level |
| scheduled_start_time | ISO8601 timestamp | time when the scheduled event should start |
| scheduled_end_time? ** | ISO8601 timestamp | time when the scheduled event should end |
| description? | string | scheduled event description |
| entity_type | entity type | scheduled event entity type |
| image? | image data | scheduled event cover image |
| recurrence_rule? | recurrence rule | rule that defines how often the event recurs |

 * Optional for events where `'entity_type': EXTERNAL` ** Required for events where `'entity_type': EXTERNAL`

### Get Guild Scheduled Event

  Retrieves a guild scheduled event. On success, returns a [guild scheduled event](https://docs.discord.com/developers/resources/guild-scheduled-event#guild-scheduled-event-object) object.

###### Query String Params

| Field | Type | Description |
| --- | --- | --- |
| with_user_count? | boolean | whether to include the user subscription count for this event |

### Modify Guild Scheduled Event

  Updates a guild scheduled event and returns the modified [guild scheduled event](https://docs.discord.com/developers/resources/guild-scheduled-event#guild-scheduled-event-object) object on success. It fires a [Guild Scheduled Event Update](https://docs.discord.com/developers/events/gateway-events#guild-scheduled-event-update) Gateway event. To start or end an event, modify the event's [status](https://docs.discord.com/developers/resources/guild-scheduled-event#guild-scheduled-event-object-guild-scheduled-event-status) field through this endpoint. This endpoint supports the `X-Audit-Log-Reason` header, silently drops `entity_metadata` for non-`EXTERNAL` events, and treats every parameter as optional.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| channel_id? * | ?snowflake | scheduled event channel id; set to `null` when changing entity type to `EXTERNAL` |
| entity_metadata? | ?entity metadata | scheduled event entity metadata |
| name? | string | scheduled event name |
| privacy_level? | privacy level | scheduled event privacy level |
| scheduled_start_time? | ISO8601 timestamp | time when the scheduled event should start |
| scheduled_end_time? * | ISO8601 timestamp | time when the scheduled event should end |
| description? | ?string | scheduled event description |
| entity_type? * | event entity type | scheduled event entity type |
| status? | event status | scheduled event status |
| image? | image data | scheduled event cover image |
| recurrence_rule? | ?recurrence rule | rule that defines how often the event recurs |

 * When updating `entity_type` to `EXTERNAL`:
- `channel_id` is required and [must be null](https://docs.discord.com/developers/resources/guild-scheduled-event#guild-scheduled-event-object-field-requirements-by-entity-type)
- Provide `entity_metadata` with a `location` field
- Provide `scheduled_end_time`

### Delete Guild Scheduled Event

  Deletes a guild scheduled event. Success returns `204` and fires a [Guild Scheduled Event Delete](https://docs.discord.com/developers/events/gateway-events#guild-scheduled-event-delete) Gateway event.

### Get Guild Scheduled Event Users

  Gets users subscribed to a guild scheduled event and returns [guild scheduled event user](https://docs.discord.com/developers/resources/guild-scheduled-event#guild-scheduled-event-user-object) objects. Guild member data is included when it exists and the `with_member` query parameter is set.

###### Query String Params

| Field | Type | Description | Default |
| --- | --- | --- | --- |
| limit? | number | number of users to return, up to maximum 100 | 100 |
| with_member? | boolean | whether to include guild member data when available | false |
| before? * | snowflake | limit results to users before the provided user id | null |
| after? * | snowflake | limit results to users after the provided user id | null |

 * Use a user id in `before` or `after` for pagination. Users are always returned in ascending `user_id` order. If both `before` and `after` are supplied, only `before` is used. Fetching users between `before` and `after` is not supported.

### Guild Scheduled Event Status Update Automation

 NOTE: `status` and `entity_type` are shown by name instead of value for readability

### An active scheduled event for a stage channel where all users have left the stage channel will automatically end a few minutes after the last user leaves the channel

 When an event has `'status': ACTIVE` and `'entity_type': STAGE_INSTANCE`, and no users remain connected to the stage channel for a period on the order of minutes, Discord automatically changes event `status` to `COMPLETED`.

### An active scheduled event for a voice channel where all users have left the voice channel will automatically end a few minutes after the last user leaves the channel

 When an event has `'status': ACTIVE` and `'entity_type': VOICE`, and no users remain connected to the voice channel for a period on the order of minutes, Discord automatically changes event `status` to `COMPLETED`.

### An external event will automatically begin at its scheduled start time

 When an event with `'entity_type': EXTERNAL` reaches its `scheduled_start_time`, Discord automatically sets `status` to `ACTIVE`.

### An external event will automatically end at its scheduled end time

 When an event with `'entity_type': EXTERNAL` reaches its `scheduled_end_time`, Discord automatically sets `status` to `COMPLETED`.

### Any scheduled event which has not begun after its scheduled start time will be automatically cancelled after a few hours

 An event whose `'status': SCHEDULED` remains past `scheduled_start_time` by a period on the order of hours has its `status` automatically changed to `CANCELLED`.

### Guild Scheduled Event Permissions Requirements

 NOTE: `entity_type` is shown by name instead of value for readability. Users must belong to the guild to access its events unless the guild is public. Public guild events may be visible depending on event type and the permissions of associated channels.

### Permissions for events with entity_type: STAGE_INSTANCE

 The user's computed permissions in the event's `channel_id` must include:

#### Get Permissions

- `VIEW_CHANNEL`

#### Create Permissions

- `CREATE_EVENTS`
- `MANAGE_CHANNELS`
- `MUTE_MEMBERS`
- `MOVE_MEMBERS`

#### Modify/Delete Permissions

- If the current user created the event, either `CREATE_EVENTS` or `MANAGE_EVENTS`; otherwise `MANAGE_EVENTS`
- `MANAGE_CHANNELS`
- `MUTE_MEMBERS`
- `MOVE_MEMBERS`

### Permissions for events with entity_type: VOICE

 The user's computed permissions in the event's `channel_id` must include:

#### Get Permissions

- `VIEW_CHANNEL`

#### Create permissions

- `CREATE_EVENTS`
- `VIEW_CHANNEL`
- `CONNECT`

#### Modify/Delete Permissions

- If the current user created the event, either `CREATE_EVENTS` or `MANAGE_EVENTS`; otherwise `MANAGE_EVENTS`
- `VIEW_CHANNEL`
- `CONNECT`

### Permissions for events with entity_type: EXTERNAL

 The user's guild-level permissions must include:

#### Get Permissions

- *No permissions required*

#### Create permissions

- `CREATE_EVENTS`

#### Modify/Delete Permissions

- If the current user created the event, either `CREATE_EVENTS` or `MANAGE_EVENTS`; otherwise `MANAGE_EVENTS`

## Guild Template Object

### Guild Template Object

 A guild template code creates a guild from a snapshot of an existing guild.

###### Guild Template Structure

| Field | Type | Description |
| --- | --- | --- |
| code | string | the template code (unique ID) |
| name | string | template name |
| description | ?string | the description for the template |
| usage_count | integer | usage count for this template |
| creator_id | snowflake | ID of the user who created the template |
| creator | user object | user who created the template |
| created_at | ISO8601 timestamp | when this template was created |
| updated_at | ISO8601 timestamp | time when this template was last synced to the source guild |
| source_guild_id | snowflake | ID of the guild this template uses as its source |
| serialized_source_guild | partial guild object | guild snapshot stored in the template; placeholder IDs are integers |
| is_dirty | ?boolean | whether the template contains changes that have not been synced |

###### Example Guild Template Object

```
{
  "code": "hgM48av5Q69A",
  "name": "Friends & Family",
  "description": "",
  "usage_count": 49605,
  "creator_id": "132837293881950208",
  "creator": {
    "id": "132837293881950208",
    "username": "hoges",
    "avatar": "79b0d9f8c340f2d43e1f78b09f175b62",
    "discriminator": "0001",
    "public_flags": 129
  },
  "created_at": "2020-04-02T21:10:38+00:00",
  "updated_at": "2020-05-01T17:57:38+00:00",
  "source_guild_id": "678070694164299796",
  "serialized_source_guild": {
    "name": "Friends & Family",
    "description": null,
    "region": "us-west",
    "verification_level": 0,
    "default_message_notifications": 0,
    "explicit_content_filter": 0,
    "preferred_locale": "en-US",
    "afk_timeout": 300,
    "roles": [
      {
        "id": 0,
        "name": "@everyone",
        "permissions": 104324689,
        "color": 0,
        "hoist": false,
        "mentionable": false
      }
    ],
    "channels": [
      {
        "name": "Text Channels",
        "position": 1,
        "topic": null,
        "bitrate": 64000,
        "user_limit": 0,
        "nsfw": false,
        "rate_limit_per_user": 0,
        "parent_id": null,
        "permission_overwrites": [],
        "id": 1,
        "type": 4
      },
      {
        "name": "general",
        "position": 1,
        "topic": null,
        "bitrate": 64000,
        "user_limit": 0,
        "nsfw": false,
        "rate_limit_per_user": 0,
        "parent_id": 1,
        "permission_overwrites": [],
        "id": 2,
        "type": 0
      }
    ],
    "afk_channel_id": null,
    "system_channel_id": 2,
    "system_channel_flags": 0,
    "icon_hash": null
  },
  "is_dirty": null
}

```

### Get Guild Template

 GET/guilds/templates/[{template.code}](https://docs.discord.com/developers/resources/guild-template#guild-template-object) returns a [guild template](https://docs.discord.com/developers/resources/guild-template#guild-template-object) object for the provided code.

### Get Guild Templates

 GET/guilds/[{guild.id}](https://docs.discord.com/developers/resources/guild#guild-object)/templates returns [guild template](https://docs.discord.com/developers/resources/guild-template#guild-template-object) objects in an array. Requires the `MANAGE_GUILD` permission.

### Create Guild Template

 POST/guilds/[{guild.id}](https://docs.discord.com/developers/resources/guild#guild-object)/templates creates a template for the guild. Requires the `MANAGE_GUILD` permission and returns the created [guild template](https://docs.discord.com/developers/resources/guild-template#guild-template-object) object on success.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name | string | name of the template (1-100 characters) |
| description? | ?string | description for the template (0-120 characters) |

### Sync Guild Template

 PUT/guilds/[{guild.id}](https://docs.discord.com/developers/resources/guild#guild-object)/templates/[{template.code}](https://docs.discord.com/developers/resources/guild-template#guild-template-object) syncs the template with the guild's current state. Requires the `MANAGE_GUILD` permission and returns the [guild template](https://docs.discord.com/developers/resources/guild-template#guild-template-object) object on success.

### Modify Guild Template

 PATCH/guilds/[{guild.id}](https://docs.discord.com/developers/resources/guild#guild-object)/templates/[{template.code}](https://docs.discord.com/developers/resources/guild-template#guild-template-object) changes the template's metadata. Requires the `MANAGE_GUILD` permission and returns the [guild template](https://docs.discord.com/developers/resources/guild-template#guild-template-object) object on success.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name? | string | name of the template (1-100 characters) |
| description? | ?string | description for the template (0-120 characters) |

### Delete Guild Template

 DELETE/guilds/[{guild.id}](https://docs.discord.com/developers/resources/guild#guild-object)/templates/[{template.code}](https://docs.discord.com/developers/resources/guild-template#guild-template-object) deletes the template. Requires the `MANAGE_GUILD` permission and returns the deleted [guild template](https://docs.discord.com/developers/resources/guild-template#guild-template-object) object on success.
