# Auto Moderation

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Auto Moderation lets each [guild](https://docs.discord.com/developers/resources/guild) define rules that fire when configured criteria are met. A rule might match a specific keyword in a message, then run configured actions automatically. For example, matching a prohibited keyword can cause the message to be blocked before posting.

### Auto Moderation Rule Object

###### Auto Moderation Rule Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | the id of this rule |
| guild_id | snowflake | id of the guild that owns this rule |
| name | string | the rule name |
| creator_id | snowflake | user who originally created the rule |
| event_type | integer | the rule event type |
| trigger_type | integer | the rule trigger type |
| trigger_metadata | object | the rule trigger metadata |
| actions | array of action objects | actions to run when the rule triggers |
| enabled | boolean | whether the rule is enabled |
| exempt_roles | array of snowflakes | role ids exempt from the rule, maximum 20 |
| exempt_channels | array of snowflakes | channel ids exempt from the rule, maximum 50 |

###### Example Auto Moderation Rule

```
{
  "id": "969707018069872670",
  "guild_id": "613425648685547541",
  "name": "Keyword Filter 1",
  "creator_id": "423457898095789043",
  "trigger_type": 1,
  "event_type": 1,
  "actions": [
    {
      "type": 1,
      "metadata": { "custom_message": "Please keep financial discussions limited to the #finance channel" }
    },
    {
      "type": 2,
      "metadata": { "channel_id": "123456789123456789" }
    },
    {
      "type": 3,
      "metadata": { "duration_seconds": 60 }
    }
  ],
  "trigger_metadata": {
    "keyword_filter": ["cat*", "*dog", "*ana*", "i like c++"],
    "regex_patterns": ["(b|c)at", "^(?:[0-9]{1,3}\\.){3}[0-9]{1,3}$"]
  },
  "enabled": true,
  "exempt_roles": ["323456789123456789", "423456789123456789"],
  "exempt_channels": ["523456789123456789"]
}

```

###### Trigger Types

 Describes which content category can cause the rule to trigger.
| Trigger Type | Value | Description | Max per Guild |
| --- | --- | --- | --- |
| KEYWORD | 1 | checks whether content includes words from a user-defined keyword list | 6 |
| SPAM | 3 | checks whether content is generic spam | 1 |
| KEYWORD_PRESET | 4 | checks whether content includes words from Discord's internal pre-defined wordsets | 1 |
| MENTION_SPAM | 5 | checks whether content exceeds the allowed number of unique mentions | 1 |
| MEMBER_PROFILE | 6 | checks whether a member profile includes words from a user-defined keyword list | 1 |

###### Trigger Metadata

 Extra data used to decide whether a rule triggers. The applicable fields depend on [trigger_type](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-trigger-types).
| Field | Type | Associated Trigger Types | Description |
| --- | --- | --- | --- |
| keyword_filter | array of strings * | KEYWORD, MEMBER_PROFILE | substrings searched for in content, maximum 1000 |
| regex_patterns | array of strings ** | KEYWORD, MEMBER_PROFILE | regular expression patterns matched against content, maximum 10 |
| presets | array of keyword preset types | KEYWORD_PRESET | internal pre-defined wordsets searched for in content |
| allow_list | array of strings *** | KEYWORD, KEYWORD_PRESET, MEMBER_PROFILE | substrings that should be ignored and not trigger the rule, maximum 100 or 1000 |
| mention_total_limit | integer | MENTION_SPAM | maximum total unique role and user mentions allowed in one message, up to 50 |
| mention_raid_protection_enabled | boolean | MENTION_SPAM | whether mention raid detection is automatic |

 * A keyword may be a multi-word phrase. [Wildcard symbols](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-keyword-matching-strategies) control how each keyword is matched. Each keyword is limited to 60 characters. ** Only Rust flavored regex is supported at this time; tools such as [Rustexp](https://rustexp.lpil.uk/) can test patterns. Each regex pattern is limited to 260 characters. *** Each `allow_list` keyword may also be a multi-word phrase. [Wildcard symbols](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-keyword-matching-strategies) control matching. Rules with `KEYWORD` [trigger_type](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-trigger-types) accept up to 100 keywords. Rules with `KEYWORD_PRESET` [trigger_type](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-trigger-types) accept up to 1000 keywords.

###### Trigger Metadata Field Limits

| Field | Trigger Types | MAX ARRAY LENGTH | MAX CHARACTERS PER STRING |
| --- | --- | --- | --- |
| keyword_filter | KEYWORD, MEMBER_PROFILE | 1000 | 60 |
| regex_patterns | KEYWORD, MEMBER_PROFILE | 10 | 260 |
| allow_list | KEYWORD, MEMBER_PROFILE | 100 | 60 |
| allow_list | KEYWORD_PRESET | 1000 | 60 |

###### Keyword Preset Types

| Preset Type | Value | Description |
| --- | --- | --- |
| PROFANITY | 1 | words that may count as swearing or cursing |
| SEXUAL_CONTENT | 2 | words referring to sexually explicit behavior or activity |
| SLURS | 3 | personal attacks or words that may qualify as hate speech |

###### Event Types

 Specifies the event context in which the rule is evaluated.
| Event Type | Value | Description |
| --- | --- | --- |
| MESSAGE_SEND | 1 | when a guild member sends or edits a message |
| MEMBER_UPDATE | 2 | when a guild member changes their profile |

###### Keyword Matching Strategies

 Place the wildcard symbol (`*`) at the start or end of a keyword to define its matching behavior. Keyword matching ignores case. **Prefix** - the word must begin with the keyword
| Keyword | Matches |
| --- | --- |
| cat* | **cat**ch, **Cat**apult, **CAt**tLE |
| tra* | **tra**in, **tra**de, **TRA**ditional |
| the mat* | **the mat**rix |

 **Suffix** - the word must finish with the keyword
| Keyword | Matches |
| --- | --- |
| *cat | wild**cat**, copy**Cat** |
| *tra | ex**tra**, ul**tra**, orches**TRA** |
| *the mat | brea**the mat** |

 **Anywhere** - the keyword may occur at any position in the content
| Keyword | Matches |
| --- | --- |
| *cat* | lo**cat**ion, edu**Cat**ion |
| *tra* | abs**tra**cted, ou**tra**ge |
| *the mat* | brea**the mat**ter |

 **Whole Word** - the keyword is a complete word or phrase and needs surrounding whitespace
| Keyword | Matches |
| --- | --- |
| cat | **cat** |
| train | **train** |
| the mat | **the mat** |

### Auto Moderation Action Object

 Action executed when a rule triggers.

###### Auto Moderation Action Structure

| Field | Type | Description |
| --- | --- | --- |
| type | action type | the type of action |
| metadata? * | action metadata | extra metadata required when executing this action type |

 * Depending on `type`, this can be omitted. Use the `Associated Action Types` column in [action metadata](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-action-object-action-metadata) to determine which `type` values require `metadata`.

###### Action Types

| Action Type | Value | Description |
| --- | --- | --- |
| BLOCK_MESSAGE | 1 | blocks a member message so it is not posted; an optional custom explanation can be shown to members when blocking occurs |
| SEND_ALERT_MESSAGE | 2 | records user content in a specified channel |
| TIMEOUT | 3 | times out a user for the specified duration * |
| BLOCK_MEMBER_INTERACTION | 4 | prevents a member from using text, voice, or other interaction surfaces |

 * A `TIMEOUT` action is valid only for `KEYWORD` and `MENTION_SPAM` rules. Using the `TIMEOUT` action type requires `MODERATE_MEMBERS`.

###### Action Metadata

 Extra data used while executing an action. The relevant fields depend on the [action type](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-action-object-action-types).
| Field | Type | Associated Action Types | Description | Constraints |
| --- | --- | --- | --- | --- |
| channel_id | snowflake | SEND_ALERT_MESSAGE | channel where user content is logged | existing channel |
| duration_seconds | integer | TIMEOUT | timeout duration in seconds | maximum of 2419200 seconds (4 weeks) |
| custom_message? | string | BLOCK_MESSAGE | extra explanation shown to members when their message is blocked | maximum of 150 characters |

### Auto Moderation Permission Requirements

 Access to all Auto Moderation resources requires the `MANAGE_GUILD` permission. Certain [action types](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-action-object-action-types) require more permissions; for example, `TIMEOUT` additionally requires `MODERATE_MEMBERS`.

### List Auto Moderation Rules for Guild

  Retrieve every rule currently configured for the guild. The response is a list of [auto moderation rule](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object) objects for that guild. The endpoint requires the `MANAGE_GUILD` [permission](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-permission-requirements).

### Get Auto Moderation Rule

  Retrieve one rule and return an [auto moderation rule](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object) object. The endpoint requires the `MANAGE_GUILD` [permission](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-permission-requirements).

### Create Auto Moderation Rule

  Create a rule. On success, the response contains an [auto moderation rule](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object), and Discord emits an [Auto Moderation Rule Create](https://docs.discord.com/developers/events/gateway-events#auto-moderation-rule-create) Gateway event. The endpoint requires the `MANAGE_GUILD` [permission](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-permission-requirements) and supports `X-Audit-Log-Reason`.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name | string | the rule name |
| event_type | integer | the event type |
| trigger_type | integer | the trigger type |
| trigger_metadata? * | object | the trigger metadata |
| actions | array of action objects | actions to run when the rule triggers |
| enabled? | boolean | whether the rule is enabled, default `False` |
| exempt_roles? | array of snowflakes | role ids exempt from the rule, maximum 20 |
| exempt_channels? | array of snowflakes | channel ids exempt from the rule, maximum 50 |

 * Depending on `trigger_type`, this may be omitted. Use the `Associated Trigger Types` column in [trigger metadata](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-trigger-metadata) to determine which `trigger_type` values require `trigger_metadata`. See [Trigger Types](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-trigger-types) for per-guild limits on rules of each trigger type.

### Modify Auto Moderation Rule

  Update an existing rule. A successful request returns an [auto moderation rule](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object) and emits an [Auto Moderation Rule Update](https://docs.discord.com/developers/events/gateway-events#auto-moderation-rule-update) Gateway event. The caller needs `MANAGE_GUILD` [permissions](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-permission-requirements). Every parameter is optional, and `X-Audit-Log-Reason` is supported.

###### JSON Params

| Field | Type | Description |
| --- | --- | --- |
| name | string | the rule name |
| event_type | integer | the event type |
| trigger_metadata? * | object | the trigger metadata |
| actions | array of action objects | actions to run when the rule triggers |
| enabled | boolean | whether the rule is enabled |
| exempt_roles | array of snowflakes | role ids exempt from the rule, maximum 20 |
| exempt_channels | array of snowflakes | channel ids exempt from the rule, maximum 50 |

 * Depending on `trigger_type`, this may be omitted. Use the `Associated Trigger Types` column in [trigger metadata](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-rule-object-trigger-metadata) to determine which `trigger_type` values require `trigger_metadata`.

### Delete Auto Moderation Rule

  Delete a rule. Success returns `204` and emits an [Auto Moderation Rule Delete](https://docs.discord.com/developers/events/gateway-events#auto-moderation-rule-delete) Gateway event. The endpoint requires the `MANAGE_GUILD` [permission](https://docs.discord.com/developers/resources/auto-moderation#auto-moderation-permission-requirements) and supports `X-Audit-Log-Reason`.
