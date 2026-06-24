# Guild (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Guilds in Discord represent an isolated collection of user accounts and channels, and are often referred to as “servers” within the UI.

### Guild Object

###### Guild Structure

 Fields specific to that `GUILD_CREATE` event are listed within the [Gateway Events documentation](https://docs.discord.com/developers/events/gateway-events#guild-create).
| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | guild id |
| name | string | Guild name (2-100 characters, excluding trailing and leading whitespace) |
| icon | ?string | icon hash |
| icon_hash? | ?string | icon hash, provided once within the template object |
| splash | ?string | splash hash |
| discovery_splash | ?string | discovery splash hash; only present for guilds using the “DISCOVERABLE” feature |
| owner? * | boolean | true once the user account is the owner for the guild |
| owner_id | snowflake | id of owner |
| permissions? * | string | total permissions for the user account within that server (excludes overwrites and implicit permissions) |
| region? ** | ?string | voice region id for that server (deprecated) |
| afk_channel_id | ?snowflake | id of afk channel |
| afk_timeout | integer | afk timeout in seconds |
| widget_enabled? | boolean | true once the server widget is enabled |
| widget_channel_id? | ?snowflake | Channel id that the widget can generate an invite to, or `null` once set to no invite |
| verification_level | integer | verification level required for the guild |
| default_message_notifications | integer | default message notifications level |
| explicit_content_filter | integer | explicit content filter level |
| roles | array of role objects | roles in the guild |
| emojis | array of emoji objects | custom guild emojis |
| features | array of guild feature strings | enabled guild features |
| mfa_level | integer | required MFA level for the guild |
| application_id | ?snowflake | application id for that server creator once it is bot-created |
| system_channel_id | ?snowflake | Channel id used for server notices such as welcome messages and boost events are posted |
| system_channel_flags | integer | system channel flags |
| rules_channel_id | ?snowflake | Channel id used for Community guilds can display rules and/or guidelines |
| max_presences? | ?integer | Maximum count of presences for that server (`null` is always returned, apart from that largest of guilds) |
| max_members? | integer | Maximum count of member records for the guild |
| vanity_url_code | ?string | Vanity url code for the guild |
| description | ?string | the description of a guild |
| banner | ?string | banner hash |
| premium_tier | integer | premium tier (Server Boost level) |
| premium_subscription_count? | integer | Count of boosts this server currently has |
| preferred_locale | string | Preferred locale of one Community guild; used in server discovery and notices from Discord, and delivered in interactions; default is “en-US” |
| public_updates_channel_id | ?snowflake | Channel id used for admins and moderators of Community guilds get notices from Discord |
| max_video_channel_users? | integer | Maximum amount of user accounts in a video channel |
| max_stage_video_channel_users? | integer | Maximum amount of user accounts in a stage video channel |
| approximate_member_count? | integer | approximate count of member records in this guild, provided from that `GET /guilds/<id>` and `/users/@me/guilds` endpoints once `with_counts` is `true` |
| approximate_presence_count? | integer | approximate count of non-offline member records in this guild, provided from that `GET /guilds/<id>` and `/users/@me/guilds` endpoints once `with_counts` is `true` |
| welcome_screen? | welcome screen object | Welcome screen of a Community guild, shown to new members, provided in an Invite’s server object |
| nsfw_level | integer | guild age-restriction level |
| stickers? | array of sticker objects | custom guild stickers |
| premium_progress_bar_enabled | boolean | Indicates whether that server includes the boost progress bar enabled |
| safety_alerts_channel_id | ?snowflake | Channel id used for admins and moderators of Community guilds get safety alerts from Discord |
| incidents_data | ?incidents data object | Incidents data for this guild |

 * These fields are only emitted once using the [GET Current User Guilds](https://docs.discord.com/developers/resources/user#get-current-user-guilds) route and are relative to that requested user ** This field is deprecated plus is replaced by [channel.rtc_region](https://docs.discord.com/developers/resources/channel#channel-object-channel-structure)

###### Default Message Notification Level

| Key | Value | Description |
| --- | --- | --- |
| ALL_MESSAGES | 0 | Members get notifications for all messages by default |
| ONLY_MENTIONS | 1 | Members get notifications only for messages that @mention them by default |

###### Explicit Content Filter Level

| Level | Integer | Description |
| --- | --- | --- |
| DISABLED | 0 | Media content can not be scanned |
| MEMBERS_WITHOUT_ROLES | 1 | Media content emitted by member records without roles may be scanned |
| ALL_MEMBERS | 2 | Media content emitted by all member records may be scanned |

###### MFA Level

| Level | Integer | Description |
| --- | --- | --- |
| NONE | 0 | Guild includes no MFA/2FA requirement for moderation actions |
| ELEVATED | 1 | Guild includes a 2FA requirement for moderation actions |

###### Verification Level

| Level | Integer | Description |
| --- | --- | --- |
| NONE | 0 | unrestricted |
| LOW | 1 | must have verified email on account |
| MEDIUM | 2 | must be registered on Discord for longer than 5 minutes |
| HIGH | 3 | must be one member for the server for longer than 10 minutes |
| VERY_HIGH | 4 | must have a verified phone number |

###### Guild Age-Restriction Level

| Level | Value |
| --- | --- |
| DEFAULT | 0 |
| EXPLICIT | 1 |
| SAFE | 2 |
| AGE_RESTRICTED | 3 |

###### Premium Tier

| Level | Integer | Description |
| --- | --- | --- |
| NONE | 0 | Guild includes not unlocked any Server Boost perks |
| TIER_1 | 1 | guild has unlocked Server Boost level 1 perks |
| TIER_2 | 2 | guild has unlocked Server Boost level 2 perks |
| TIER_3 | 3 | guild has unlocked Server Boost level 3 perks |

###### System Channel Flags

| Flag | Value | Description |
| --- | --- | --- |
| SUPPRESS_JOIN_NOTIFICATIONS | `1 << 0` | Suppress member join notifications |
| SUPPRESS_PREMIUM_SUBSCRIPTIONS | `1 << 1` | Suppress server boost notifications |
| SUPPRESS_GUILD_REMINDER_NOTIFICATIONS | `1 << 2` | Suppress server setup tips |
| SUPPRESS_JOIN_NOTIFICATION_REPLIES | `1 << 3` | Hide member join sticker reply buttons |
| SUPPRESS_ROLE_SUBSCRIPTION_PURCHASE_NOTIFICATIONS | `1 << 4` | Suppress role subscription purchase and renewal notifications |
| SUPPRESS_ROLE_SUBSCRIPTION_PURCHASE_NOTIFICATION_REPLIES | `1 << 5` | Hide role subscription sticker reply buttons |

###### Guild Features

| Feature | Description |
| --- | --- |
| ANIMATED_BANNER | Guild includes access to set an animated server banner image |
| ANIMATED_ICON | Guild includes access to set an animated server icon |
| APPLICATION_COMMAND_PERMISSIONS_V2 | Guild is using the old permissions configuration behavior |
| AUTO_MODERATION | guild has set up auto moderation rules |
| BANNER | Guild includes access to set a server banner image |
| COMMUNITY | Guild can enable welcome screen, Membership Screening, stage channel records and discovery, and receives community updates |
| CREATOR_MONETIZABLE_PROVISIONAL | guild has enabled monetization |
| CREATOR_STORE_PAGE | Guild includes enabled the role subscription promo page |
| DEVELOPER_SUPPORT_SERVER | Guild includes been set as a support server on the App Directory |
| DISCOVERABLE | Guild is able to be discovered within the directory |
| FEATURABLE | Guild is able to be featured within the directory |
| INVITES_DISABLED | Guild includes paused invites, preventing new user accounts from joining |
| INVITE_SPLASH | Guild includes access to set an invite splash background |
| MEMBER_VERIFICATION_GATE_ENABLED | guild has enabled Membership Screening |
| MORE_SOUNDBOARD | guild has increased custom soundboard sound slots |
| MORE_STICKERS | guild has increased custom sticker slots |
| NEWS | Guild includes access to create announcement channels |
| PARTNERED | guild is partnered |
| PREVIEW_ENABLED | Guild may be previewed before joining via Membership Screening or the directory |
| RAID_ALERTS_DISABLED | Guild includes disabled alerts for join raids within the configured safety alerts channel |
| ROLE_ICONS | Guild is able to set role icons |
| ROLE_SUBSCRIPTIONS_AVAILABLE_FOR_PURCHASE | Guild includes role subscriptions that may be purchased |
| ROLE_SUBSCRIPTIONS_ENABLED | guild has enabled role subscriptions |
| SOUNDBOARD | guild has created soundboard sounds |
| TICKETED_EVENTS_ENABLED | guild has enabled ticketed events |
| VANITY_URL | Guild includes access to set a vanity URL |
| VERIFIED | guild is verified |
| VIP_REGIONS | Guild includes access to set 384kbps bitrate in voice (previously VIP voice servers) |
| WELCOME_SCREEN_ENABLED | Guild includes enabled the welcome screen |
| GUESTS_ENABLED | Guild includes access to guest invites |
| GUILD_TAGS | Guild includes access to set server tags |
| ENHANCED_ROLE_COLORS | Guild is able to set gradient colors to roles |

###### Mutable Guild Features

| Features | Required Permissions | Effects |
| --- | --- | --- |
| COMMUNITY | Administrator | Enables Community Features within the guild |
| DISCOVERABLE | Administrator* | Enables discovery within the guild, making it publicly listed |
| INVITES_DISABLED | Manage Guild | Pauses all invites/access to that server |
| RAID_ALERTS_DISABLED | Manage Guild | Disables alerts for join raids |

 * Server also must be passing all discovery requirements

###### Example Guild

```
{
  "id": "197038439483310086",
  "name": "Discord Testers",
  "icon": "f64c482b807da4f539cff778d174971c",
  "description": "The official place to report Discord Bugs!",
  "splash": null,
  "discovery_splash": null,
  "features": [
    "ANIMATED_ICON",
    "VERIFIED",
    "NEWS",
    "VANITY_URL",
    "DISCOVERABLE",
    "MORE_EMOJI",
    "INVITE_SPLASH",
    "BANNER",
    "COMMUNITY"
  ],
  "emojis": [],
  "banner": "9b6439a7de04f1d26af92f84ac9e1e4a",
  "owner_id": "73193882359173120",
  "application_id": null,
  "region": null,
  "afk_channel_id": null,
  "afk_timeout": 300,
  "system_channel_id": null,
  "widget_enabled": true,
  "widget_channel_id": null,
  "verification_level": 3,
  "roles": [],
  "default_message_notifications": 1,
  "mfa_level": 1,
  "explicit_content_filter": 2,
  "max_presences": 40000,
  "max_members": 250000,
  "vanity_url_code": "discord-testers",
  "premium_tier": 3,
  "premium_subscription_count": 33,
  "system_channel_flags": 0,
  "preferred_locale": "en-US",
  "rules_channel_id": "441688182833020939",
  "public_updates_channel_id": "281283303326089216",
  "safety_alerts_channel_id": "281283303326089216"
}

```

### Unavailable Guild Object

 A partial [guild](https://docs.discord.com/developers/resources/guild#guild-object) object. Represents an Offline Guild, or one Guild whose information has not already been provided through [Guild Create](https://docs.discord.com/developers/events/gateway-events#guild-create) events during this Gateway connect.

###### Example Unavailable Guild

```
{
  "id": "41771983423143937",
  "unavailable": true
}

```

### Guild Preview Object

###### Guild Preview Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | guild id |
| name | string | guild name (2-100 characters) |
| icon | ?string | icon hash |
| splash | ?string | splash hash |
| discovery_splash | ?string | discovery splash hash |
| emojis | array of emoji objects | custom guild emojis |
| features | array of guild feature strings | enabled guild features |
| approximate_member_count | integer | approximate count of member records in this guild |
| approximate_presence_count | integer | approximate count of online member records in this guild |
| description | ?string | the description for the guild |
| stickers | array of sticker objects | custom guild stickers |

###### Example Guild Preview

```
{
  "id": "197038439483310086",
  "name": "Discord Testers",
  "icon": "f64c482b807da4f539cff778d174971c",
  "splash": null,
  "discovery_splash": null,
  "emojis": [],
  "features": [
    "DISCOVERABLE",
    "VANITY_URL",
    "ANIMATED_ICON",
    "INVITE_SPLASH",
    "NEWS",
    "COMMUNITY",
    "BANNER",
    "VERIFIED",
    "MORE_EMOJI"
  ],
  "approximate_member_count": 60814,
  "approximate_presence_count": 20034,
  "description": "The official place to report Discord Bugs!",
  "stickers": []
}

```

### Guild Widget Settings Object

###### Guild Widget Settings Structure

| Field | Type | Description |
| --- | --- | --- |
| enabled | boolean | whether the widget is enabled |
| channel_id | ?snowflake | the widget channel id |

###### Example Guild Widget Settings

```
{
  "enabled": true,
  "channel_id": "41771983444115456"
}

```

### Guild Widget Object

###### Guild Widget Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | guild id |
| name | string | guild name (2-100 characters) |
| instant_invite | ?string | instant invite for the guilds specified widget invite channel |
| channels | array of partial channel objects | voice and stage channel records which are accessible by @everyone |
| members | array of partial user objects | special widget user object records that includes user accounts presence (Limit 100) |
| presence_count | integer | Count of online member records in this guild |

 The fields `id`, `discriminator` and `avatar` are anonymized to prevent abuse.

###### Example Guild Widget

```
{
  "id": "290926798626999250",
  "name": "Test Server",
  "instant_invite": "https://discord.com/invite/abcdefg",
  "channels": [
    {
      "id": "705216630279993882",
      "name": "elephant",
      "position": 2
    },
    {
      "id": "669583461748992190",
      "name": "groovy-music",
      "position": 1
    }
  ],
  "members": [
    {
      "id": "0",
      "username": "1234",
      "discriminator": "0000",
      "avatar": null,
      "status": "online",
      "avatar_url": "https://cdn.discordapp.com/widget-avatars/FfvURgcr3Za92K3JtoCppqnYMppMDc5B-Rll74YrGCU/C-1DyBZPQ6t5q2RuATFuMFgq0_uEMZVzd_6LbGN_uJKvZflobA9diAlTjhf6CAESLLeTuu4dLuHFWOb_PNLteooNfhC4C6k5QgAGuxEOP12tVVVCvX6t64k14PMXZrGTDq8pWZhukP40Wg"
    }
  ],
  "presence_count": 1
}

```

### Guild Member Object

###### Guild Member Structure

| Field | Type | Description |
| --- | --- | --- |
| user? | user object | User this server member represents |
| nick? | ?string | this user’s guild nickname |
| avatar? | ?string | Member’s server avatar hash |
| banner? | ?string | Member’s server banner hash |
| roles | array of snowflakes | array of role object ids |
| joined_at | ?ISO8601 timestamp | timestamp for once the user account joined the guild |
| premium_since? | ?ISO8601 timestamp | timestamp for once the user account started boosting the guild |
| deaf | boolean | Indicates whether the user account is deafened in voice channels |
| mute | boolean | Indicates whether the user account is muted in voice channels |
| flags | integer | Guild member flags represented as a bit set, default is `0` |
| pending? | boolean | Indicates whether the user account has not yet passed the guild’s Membership Screening requirements |
| permissions? | string | total permissions for that member within the channel, covering overwrites, provided once within the interaction object |
| communication_disabled_until? | ?ISO8601 timestamp | timestamp for once the user’s timeout can expire and the user account can communicate within that server again, null or a time within the past once the user account is not timed out |
| avatar_decoration_data? | ?avatar decoration data object | data for the member’s server avatar decoration |
| collectibles? | ?collectibles object | data for the member’s collectibles |

 The field `user` won’t be included within that member object record attached to `MESSAGE_CREATE` and `MESSAGE_UPDATE` gateway events. In `GUILD_` events, `pending` can always be included as true or else false. Inside non `GUILD_` events which may only be triggered by non-`pending` users, `pending` can not be included. Member object records retrieved from `VOICE_STATE_UPDATE` events can have `joined_at` set as `null` once that member was invited as a guest.

###### Example Guild Member

```
{
  "user": {},
  "nick": "NOT API SUPPORT",
  "avatar": null,
  "banner": null,
  "roles": [],
  "joined_at": "2015-04-26T06:26:56.936000+00:00",
  "deaf": false,
  "mute": false
}

```

###### Guild Member Flags

| Flag | Value | Description | Editable |
| --- | --- | --- | --- |
| DID_REJOIN | `1 << 0` | Member has left and rejoined the guild | false |
| COMPLETED_ONBOARDING | `1 << 1` | Member has completed onboarding | false |
| BYPASSES_VERIFICATION | `1 << 2` | Member is exempt from server verification requirements | true |
| STARTED_ONBOARDING | `1 << 3` | Member has started onboarding | false |
| IS_GUEST | `1 << 4` | Member is a guest and can only access this voice channel they were invited to | false |
| STARTED_HOME_ACTIONS | `1 << 5` | Member has started Server Guide new member actions | false |
| COMPLETED_HOME_ACTIONS | `1 << 6` | Member has completed Server Guide new member actions | false |
| AUTOMOD_QUARANTINED_USERNAME | `1 << 7` | Member’s username, display name, or nickname is blocked by AutoMod | false |
| DM_SETTINGS_UPSELL_ACKNOWLEDGED | `1 << 9` | Member has dismissed the DM settings upsell | false |
| AUTOMOD_QUARANTINED_GUILD_TAG | `1 << 10` | Member’s server tag is blocked by AutoMod | false |

 BYPASSES_VERIFICATION permits one member who does not meet verification requirements to participate in a server.

### Integration Object

###### Integration Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | integration id |
| name | string | integration name |
| type | string | integration type (twitch, youtube, discord, or guild_subscription) |
| enabled | boolean | is this integration enabled |
| syncing? * | boolean | is this integration syncing |
| role_id? * | snowflake | id that this integration uses for “subscribers” |
| enable_emoticons? * | boolean | Indicates whether emoticons must be synced for this integration (twitch only currently) |
| expire_behavior? * | integration expire behavior | the behavior of expiring subscribers |
| expire_grace_period? * | integer | Grace period (in days) before expiring subscribers |
| user? | user object | user for this integration |
| account | account object | integration account information |
| synced_at? * | ISO8601 timestamp | timestamp for once this integration was last synced |
| subscriber_count? * | integer | how many subscribers this integration has |
| revoked? * | boolean | has this integration been revoked |
| application? | application object | The bot/OAuth2 application for discord integrations |
| scopes? | array of OAuth2 scopes | Scopes the application has been authorized for |

 * The current fields remain not provided for discord bot integrations. Some older integrations may not have one attached user.

###### Integration Expire Behaviors

| Value | Name |
| --- | --- |
| 0 | Remove role |
| 1 | Kick |

### Integration Account Object

###### Integration Account Structure

| Field | Type | Description |
| --- | --- | --- |
| id | string | id of the account |
| name | string | name of the account |

### Integration Application Object

###### Integration Application Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | the id of the app |
| name | string | the name of the app |
| icon | ?string | Icon hash for the app |
| description | string | the description of the app |
| bot? | user object | Bot associated with this application |

### Ban Object

###### Ban Structure

| Field | Type | Description |
| --- | --- | --- |
| reason | ?string | the reason for the ban |
| user | user object | the banned user |

###### Example Ban

```
{
  "reason": "mentioning b1nzy",
  "user": {
    "username": "Mason",
    "discriminator": "9999",
    "id": "53908099506183680",
    "avatar": "a_bab14f271d565501444b2ca3be944b25",
    "public_flags": 131141
  }
}

```

### Welcome Screen Object

###### Welcome Screen Structure

| Field | Type | Description |
| --- | --- | --- |
| description | ?string | Server description shown within the welcome screen |
| welcome_channels | List of welcome screen channel objects | Channels shown within the welcome screen, up to 5 |

###### Welcome Screen Channel Structure

| Field | Type | Description |
| --- | --- | --- |
| channel_id | snowflake | the channel’s id |
| description | string | Description shown for the channel |
| emoji_id | ?snowflake | Emoji id, once the emoji is custom |
| emoji_name | ?string | Emoji name once custom, the unicode character once standard, or `null` once no emoji is set |

###### Example Welcome Screen

```
{
  "description": "Discord Developers is a place to learn about Discord's API, bots, and SDKs and integrations. This is NOT a general Discord support server.",
  "welcome_channels": [
    {
      "channel_id": "697138785317814292",
      "description": "Follow for official Discord API updates",
      "emoji_id": null,
      "emoji_name": "📡"
    },
    {
      "channel_id": "697236247739105340",
      "description": "Get help with Bot Verifications",
      "emoji_id": null,
      "emoji_name": "📸"
    },
    {
      "channel_id": "697489244649816084",
      "description": "Create amazing things with Discord's API",
      "emoji_id": null,
      "emoji_name": "🔬"
    },
    {
      "channel_id": "613425918748131338",
      "description": "Integrate Discord into your game",
      "emoji_id": null,
      "emoji_name": "🎮"
    },
    {
      "channel_id": "646517734150242346",
      "description": "Find more places to help you on your quest",
      "emoji_id": null,
      "emoji_name": "🔦"
    }
  ]
}

```

### Guild Onboarding Object

 Represents the [onboarding](https://support.discord.com/hc/en-us/articles/11074987197975-Community-Onboarding-FAQ) flow for a guild.

###### Guild Onboarding Structure

| Field | Type | Description |
| --- | --- | --- |
| guild_id | snowflake | ID for that server this onboarding is part of |
| prompts | array of onboarding prompt objects | Prompts shown during onboarding and in customize community |
| default_channel_ids | array of snowflakes | Channel IDs that member records get opted into automatically |
| enabled | boolean | Whether onboarding is enabled within the guild |
| mode | onboarding mode | Current mode of onboarding |

###### Onboarding Prompt Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | ID of the prompt |
| type | prompt type | Type of prompt |
| options | array of prompt option objects | Options available within the prompt |
| title | string | Title of the prompt |
| single_select | boolean | Indicates indicates whether user accounts are limited to selecting one option for the prompt |
| required | boolean | Indicates Indicates whether the prompt is required before one user completes the onboarding flow |
| in_onboarding | boolean | Indicates Indicates whether the prompt is present within the onboarding flow. Once `false`, the prompt can only appear within the Channels & Roles tab |

###### Prompt Option Structure

| Field | Type | Description |
| --- | --- | --- |
| id | snowflake | ID of the prompt option |
| channel_ids | array of snowflakes | IDs for channel records one member is added to timestamp for once the option is selected |
| role_ids | array of snowflakes | IDs for roles assigned to one member timestamp for once the option is selected |
| emoji? | emoji object | Emoji for the option (see below) |
| emoji_id? | snowflake | Emoji ID for the option (see below) |
| emoji_name? | string | Emoji name for the option (see below) |
| emoji_animated? | boolean | Whether the emoji is animated (see below) |
| title | string | Title of the option |
| description | ?string | Description of the option |

 Once creating or updating a prompt option, this `emoji_id`, `emoji_name`, and `emoji_animated` fields must be used instead for the emoji object.

###### Onboarding Mode

 Defines the criteria used for satisfy Onboarding constraints that are required for enabling.
| Name | Value | Description |
| --- | --- | --- |
| ONBOARDING_DEFAULT | 0 | Counts only Default Channels towards constraints |
| ONBOARDING_ADVANCED | 1 | Counts Default Channels and Questions towards constraints |

###### Prompt Types

| Name | Value |
| --- | --- |
| MULTIPLE_CHOICE | 0 |
| DROPDOWN | 1 |

###### Example Guild Onboarding
