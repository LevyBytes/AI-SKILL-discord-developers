# Audit Log

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Discord creates a guild audit-log entry whenever an administrative guild action occurs. Users need `VIEW_AUDIT_LOG` to view it, and apps can retrieve it with [`GET /guilds/{guild.id}/audit-logs`](https://docs.discord.com/developers/resources/audit-log#get-guild-audit-log); users can also see it in **Server Settings**. Entries are retained for 45 days. When an app performs an eligible API action, it may include `X-Audit-Log-Reason` to record the reason. See [audit log entry](https://docs.discord.com/developers/resources/audit-log#audit-log-entry-object) for details.

### Audit Log Object

###### Audit Log Structure

| Field | Type | Description |
| --- | --- | --- |
| application_commands | array of application commands objects | Application commands that the audit log references |
| audit_log_entries | array of audit log entry objects | Audit log entries ordered newest to oldest |
| auto_moderation_rules | array of auto moderation rule objects | Auto moderation rules referenced by the audit log |
| guild_scheduled_events | array of guild scheduled event objects | Guild scheduled events referenced by the audit log |
| integrations | array of partial integration objects | List of partial integration objects |
| threads | array of thread-specific channel objects | Threads referenced by the audit log* |
| users | array of user objects | Users referenced by the audit log |
| webhooks | array of webhook objects | Webhooks referenced by the audit log |

 * The threads map includes threads referenced by `THREAD_CREATE` and `THREAD_UPDATE` because clients may not retain archived threads in memory.

###### Example Partial Integration Object

```
{
  "id": "33590653072239123",
  "name": "A Name",
  "type": "twitch",
  "account": {
    "name": "twitchusername",
    "id": "1234567"
  },
  "application_id": "94651234501213162"
}

```

### Audit Log Entry Object

 An audit log entry records one administrative action or [event](https://docs.discord.com/developers/resources/audit-log#audit-log-entry-object-audit-log-events), identified by `action_type`. Most entries include a `changes` array with one or more modifications to a Discord entity such as a user, channel, guild, emoji, or another object. The content and shape of those changes depends on the entry type. For `MEMBER_ROLE_UPDATE`, a single change indicates that a role was added or removed. For `CHANNEL_CREATE`, many changes can appear, including channel name, channel type, and created permission overwrites. See the [change object](https://docs.discord.com/developers/resources/audit-log#audit-log-change-object) section for more information. Apps can explain an administrative API action by sending `X-Audit-Log-Reason`; Discord stores it in the entry `reason` field. `X-Audit-Log-Reason` accepts 1-512 URL-encoded UTF-8 characters. Users can see reasons in the client, and apps receive them when fetching audit log entries.

###### Audit Log Entry Structure

| Field | Type | Description |
| --- | --- | --- |
| target_id | ?string | ID for the affected entity, such as a webhook, user, or role |
| changes? | array of audit log change objects | Changes applied to the target_id |
| user_id | ?snowflake | User or app responsible for the change |
| id | snowflake | ID of the entry |
| action_type | audit log event | Type of action that occurred |
| options? | optional audit entry info | Extra details for particular event types |
| reason? | string | Reason for the change (1-512 characters) |

 For `APPLICATION_COMMAND_PERMISSION_UPDATE`, `target_id` is either the command ID or app ID because `changes` represents the full `permissions` property on the [guild permissions](https://docs.discord.com/developers/interactions/application-commands#application-command-permissions-object-guild-application-command-permissions-structure) object.

###### Audit Log Events

 The following table gives audit log events and their `action_type` values. **Object Changed** identifies which object may contribute values to the entry. In normal cases, keys in `changes` map to fields on that object, and the linked object documentation gives field types and descriptions. When no object is listed, the entry will not include `changes`, although `target_id` and often [`options` object](https://docs.discord.com/developers/resources/audit-log#audit-log-entry-object-optional-audit-entry-info) fields may still appear. Apps should be prepared for any changed-object field, but no field is guaranteed; usually only some fields appear in `changes`.
| Event | Value | Description | Object Changed |
| --- | --- | --- | --- |
| GUILD_UPDATE | 1 | Server settings were updated | Guild |
| CHANNEL_CREATE | 10 | Channel was created | Channel |
| CHANNEL_UPDATE | 11 | Channel settings were updated | Channel |
| CHANNEL_DELETE | 12 | Channel was deleted | Channel |
| CHANNEL_OVERWRITE_CREATE | 13 | A permission overwrite was created for a channel | Channel Overwrite |
| CHANNEL_OVERWRITE_UPDATE | 14 | A channel permission overwrite was changed | Channel Overwrite |
| CHANNEL_OVERWRITE_DELETE | 15 | A permission overwrite was removed from a channel | Channel Overwrite |
| MEMBER_KICK | 20 | Member was removed from server |  |
| MEMBER_PRUNE | 21 | Members were pruned from server |  |
| MEMBER_BAN_ADD | 22 | Member was banned from server |  |
| MEMBER_BAN_REMOVE | 23 | A member was unbanned from the server |  |
| MEMBER_UPDATE | 24 | Member was updated in server | Member |
| MEMBER_ROLE_UPDATE | 25 | A role was added to or removed from a member | Partial Role* |
| MEMBER_MOVE | 26 | A member was moved into another voice channel |  |
| MEMBER_DISCONNECT | 27 | A member was removed from a voice channel connection |  |
| BOT_ADD | 28 | A bot user joined the server |  |
| ROLE_CREATE | 30 | Role was created | Role |
| ROLE_UPDATE | 31 | Role was edited | Role |
| ROLE_DELETE | 32 | Role was deleted | Role |
| INVITE_CREATE | 40 | Server invite was created | Invite and Invite Metadata* |
| INVITE_UPDATE | 41 | Server invite was updated | Invite and Invite Metadata* |
| INVITE_DELETE | 42 | Server invite was deleted | Invite and Invite Metadata* |
| WEBHOOK_CREATE | 50 | Webhook was created | Webhook* |
| WEBHOOK_UPDATE | 51 | Webhook channel or properties changed | Webhook* |
| WEBHOOK_DELETE | 52 | Webhook was deleted | Webhook* |
| EMOJI_CREATE | 60 | Emoji was created | Emoji |
| EMOJI_UPDATE | 61 | Emoji name was updated | Emoji |
| EMOJI_DELETE | 62 | Emoji was deleted | Emoji |
| MESSAGE_DELETE | 72 | Single message was deleted |  |
| MESSAGE_BULK_DELETE | 73 | Multiple messages were deleted |  |
| MESSAGE_PIN | 74 | A message was pinned in a channel |  |
| MESSAGE_UNPIN | 75 | A message was removed from channel pins |  |
| INTEGRATION_CREATE | 80 | App was added to server | Integration |
| INTEGRATION_UPDATE | 81 | An app changed, for example through scope updates | Integration |
| INTEGRATION_DELETE | 82 | App was removed from server | Integration |
| STAGE_INSTANCE_CREATE | 83 | Stage instance was created (stage channel becomes live) | Stage Instance |
| STAGE_INSTANCE_UPDATE | 84 | Stage instance details were updated | Stage Instance |
| STAGE_INSTANCE_DELETE | 85 | A stage instance was removed, ending the live stage channel | Stage Instance |
| STICKER_CREATE | 90 | Sticker was created | Sticker |
| STICKER_UPDATE | 91 | Sticker details were updated | Sticker |
| STICKER_DELETE | 92 | Sticker was deleted | Sticker |
| GUILD_SCHEDULED_EVENT_CREATE | 100 | Event was created | Guild Scheduled Event |
| GUILD_SCHEDULED_EVENT_UPDATE | 101 | Event was updated | Guild Scheduled Event |
| GUILD_SCHEDULED_EVENT_DELETE | 102 | Event was cancelled | Guild Scheduled Event |
| THREAD_CREATE | 110 | A channel thread was created | Thread |
| THREAD_UPDATE | 111 | Thread was updated | Thread |
| THREAD_DELETE | 112 | Thread was deleted | Thread |
| APPLICATION_COMMAND_PERMISSION_UPDATE | 121 | Command permissions changed | Command Permission* |
| SOUNDBOARD_SOUND_CREATE | 130 | Soundboard sound was created | Soundboard Sound |
| SOUNDBOARD_SOUND_UPDATE | 131 | Soundboard sound was updated | Soundboard Sound |
| SOUNDBOARD_SOUND_DELETE | 132 | Soundboard sound was deleted | Soundboard Sound |
| AUTO_MODERATION_RULE_CREATE | 140 | Auto Moderation rule was created | Auto Moderation Rule |
| AUTO_MODERATION_RULE_UPDATE | 141 | Auto Moderation rule was updated | Auto Moderation Rule |
| AUTO_MODERATION_RULE_DELETE | 142 | Auto Moderation rule was deleted | Auto Moderation Rule |
| AUTO_MODERATION_BLOCK_MESSAGE | 143 | Auto Moderation blocked a message |  |
| AUTO_MODERATION_FLAG_TO_CHANNEL | 144 | Auto Moderation flagged a message |  |
| AUTO_MODERATION_USER_COMMUNICATION_DISABLED | 145 | Auto Moderation timed out a member |  |
| AUTO_MODERATION_QUARANTINE_USER | 146 | Auto Moderation quarantined a member |  |
| CREATOR_MONETIZATION_REQUEST_CREATED | 150 | Creator monetization request was created |  |
| CREATOR_MONETIZATION_TERMS_ACCEPTED | 151 | Creator monetization terms were accepted |  |
| ONBOARDING_PROMPT_CREATE | 163 | Guild Onboarding Question was created | Onboarding Prompt Structure |
| ONBOARDING_PROMPT_UPDATE | 164 | Guild Onboarding Question was updated | Onboarding Prompt Structure |
| ONBOARDING_PROMPT_DELETE | 165 | Guild Onboarding Question was deleted | Onboarding Prompt Structure |
| ONBOARDING_CREATE | 166 | Guild Onboarding was created | Guild Onboarding |
| ONBOARDING_UPDATE | 167 | Guild Onboarding was updated | Guild Onboarding |
| HOME_SETTINGS_CREATE | 190 | Guild Server Guide was created |  |
| HOME_SETTINGS_UPDATE | 191 | Guild Server Guide was updated |  |
| VOICE_CHANNEL_STATUS_CREATE | 192 | A user set the status for a voice channel |  |
| VOICE_CHANNEL_STATUS_DELETE | 193 | A user deleted a voice channel status |  |

 * This object has exceptions for available keys. See [exceptions](https://docs.discord.com/developers/resources/audit-log#audit-log-change-object-audit-log-change-exceptions).

###### Optional Audit Entry Info

| Field | Type | Description | Event Types |
| --- | --- | --- | --- |
| application_id | snowflake | ID of the app targeted by the permission change | APPLICATION_COMMAND_PERMISSION_UPDATE |
| auto_moderation_rule_name | string | Name of the triggered Auto Moderation rule | AUTO_MODERATION_BLOCK_MESSAGE & AUTO_MODERATION_FLAG_TO_CHANNEL & AUTO_MODERATION_USER_COMMUNICATION_DISABLED & AUTO_MODERATION_QUARANTINE_USER |
| auto_moderation_rule_trigger_type | string | Trigger type for the Auto Moderation rule that fired | AUTO_MODERATION_BLOCK_MESSAGE & AUTO_MODERATION_FLAG_TO_CHANNEL & AUTO_MODERATION_USER_COMMUNICATION_DISABLED & AUTO_MODERATION_QUARANTINE_USER |
| channel_id | snowflake | Channel where the entities were targeted | MEMBER_MOVE & MESSAGE_PIN & MESSAGE_UNPIN & MESSAGE_DELETE & STAGE_INSTANCE_CREATE & STAGE_INSTANCE_UPDATE & STAGE_INSTANCE_DELETE & AUTO_MODERATION_BLOCK_MESSAGE & AUTO_MODERATION_FLAG_TO_CHANNEL & AUTO_MODERATION_USER_COMMUNICATION_DISABLED & AUTO_MODERATION_QUARANTINE_USER & VOICE_CHANNEL_STATUS_CREATE & VOICE_CHANNEL_STATUS_DELETE |
| count | string | Count of targeted entities | MESSAGE_DELETE & MESSAGE_BULK_DELETE & MEMBER_DISCONNECT & MEMBER_MOVE |
| delete_member_days | string | Inactive-day threshold used before members were pruned | MEMBER_PRUNE |
| id | snowflake | ID of the overwritten entity | CHANNEL_OVERWRITE_CREATE & CHANNEL_OVERWRITE_UPDATE & CHANNEL_OVERWRITE_DELETE |
| members_removed | string | How many members the prune removed | MEMBER_PRUNE |
| message_id | snowflake | ID for the targeted message | MESSAGE_PIN & MESSAGE_UNPIN |
| role_name | string | Role name when type is `"0"`; omitted when type is `"1"` | CHANNEL_OVERWRITE_CREATE & CHANNEL_OVERWRITE_UPDATE & CHANNEL_OVERWRITE_DELETE |
| type | string | Overwritten entity type: role (`"0"`) or member (`"1"`) | CHANNEL_OVERWRITE_CREATE & CHANNEL_OVERWRITE_UPDATE & CHANNEL_OVERWRITE_DELETE |
| integration_type | string | Integration type that executed the action | MEMBER_KICK & MEMBER_ROLE_UPDATE |
| status | string | The new voice channel status | VOICE_CHANNEL_STATUS_CREATE |

### Audit Log Change Object

 Many audit log events include a `changes` array on their [entry object](https://docs.discord.com/developers/resources/audit-log#audit-log-entry-object-audit-log-entry-structure). The [audit log entry structure](https://docs.discord.com/developers/resources/audit-log#audit-log-entry-object-audit-log-entry-structure) and the [individual change structure](https://docs.discord.com/developers/resources/audit-log#audit-log-change-object-audit-log-change-structure) depend on the event type and changed object, so apps should not expect one universal audit-log handling pattern.

###### Audit Log Change Structure

 Some events differ from the usual audit-log shape; [the next section](https://docs.discord.com/developers/resources/audit-log#audit-log-change-object-audit-log-change-exceptions) describes those exceptions. If a change has `old_value` but lacks `new_value`, the property was reset or set to `null`. If `old_value` is absent, the property was previously `null`.
| Field | Type | Description |
| --- | --- | --- |
| new_value? | mixed (matches object field’s type) | New value of the key |
| old_value? | mixed (matches object field’s type) | Old value of the key |
| key | string | Changed entity name, except for specified exceptions |

###### Audit Log Change Exceptions

 For most changed objects, change keys can be any field on that object. The table below lists cases that do not follow that pattern.
| Object Changed | Change Key Exceptions | Change Object Exceptions |
| --- | --- | --- |
| Command Permission | snowflake as key | The `changes` array uses objects whose `key` is the affected command entity, such as a role, channel, or user ID; each object includes the prior permissions object under `old_value` and the updated permissions object under `new_value` |
| Invite and Invite Metadata | Extra `channel_id` key rather than the object's `channel.id` |  |
| Partial Role | `$add` and `$remove` as keys | `new_value` contains an array of objects with role `id` and `name` |
| Webhook | `avatar_hash` key (instead of `avatar`) |  |

### Get Guild Audit Log

  Returns the guild [audit log](https://docs.discord.com/developers/resources/audit-log#audit-log-object) object. The caller needs [`VIEW_AUDIT_LOG`](https://docs.discord.com/developers/topics/permissions#permissions-bitwise-permission-flags). Returned audit log entry ordering depends on `before` or `after`. With `before`, entries are sorted by audit log entry ID in **descending** order, newest first. With `after`, Discord reverses the list to **ascending** order, oldest first. If neither `before` nor `after` is provided, Discord behaves as if `before` is the current timestamp and returns the latest entries in descending ID order. Use `after=0` for the opposite, oldest-first behavior.

###### Query String Params

 Use these query parameters to filter the returned audit log entries and control result count.
| Field | Type | Description |
| --- | --- | --- |
| user_id? | snowflake | Entries created by a given user ID |
| action_type? | integer | Entries for one audit log event type |
| before? | snowflake | Entries whose ID is lower than a given audit log entry ID |
| after? | snowflake | Entries whose ID is higher than a given audit log entry ID |
| limit? | integer | Maximum entries to return, from 1-100; defaults to 50 |
