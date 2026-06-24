# Embedded App Sdk (part 2)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

| Property | Type |
| --- | --- |
| user_id | string |
| nick? | string | null |
| guild_id | string |
| avatar? | string | null |
| avatar_decoration_data? | AvatarDecorationData | null |
| color_string? | string | null |

#### Image

| Property | Type |
| --- | --- |
| url? | string | null |
| proxy_url? | string | null |
| height? | number | null |
| width? | number | null |

#### InitiateImageUploadResponse

| Property | Type |
| --- | --- |
| image_url | string |

#### Message

| Property | Type |  |
| --- | --- | --- |
| id | string |  |
| channel_id | string |  |
| guild_id? | string | null |  |
| author? | User | null |  |
| member? | GuildMember | null |  |
| content | string |  |
| timestamp | string |  |
| edited_timestamp? | string | null |  |
| tts | boolean |  |
| mention_everyone | boolean |  |
| mentions | User[] |  |
| mention_roles | string[] |  |
| mention_channels | ChannelMention[] |  |
| attachments | Attachment[] |  |
| embeds | Embed[] |  |
| reactions? | Reaction[] | null |  |
| nonce? | string | number | null |
| pinned | boolean |  |
| webhook_id? | string | null |  |
| type | number |  |
| activity? | MessageActivity | null |  |
| application? | MessageApplication | null |  |
| message_reference? | MessageReference | null |  |
| flags? | number |  |
| stickers? | Sticker[] | null |  |
| referenced_message? | Message | null |  |

#### MessageActivity

| Property | Type |
| --- | --- |
| type | number |
| party_id? | string | null |

#### MessageApplication

| Property | Type |
| --- | --- |
| id | string |
| cover_image? | string | null |
| description | string |
| icon? | string | null |
| name | string |

#### MessageReference

| Property | Type |
| --- | --- |
| message_id? | string | null |
| channel_id? | string | null |
| guild_id? | string | null |

#### OpenExternalLinkRequest

| Property | Type |
| --- | --- |
| url | string |

#### OpenExternalLinkResponse

 Discord clients before December 2024 may not report whether opening a link succeeded; those clients return `{ opened: null }`.
| Property | Type |
| --- | --- |
| opened | boolean | null |

#### OpenShareMomentDialogRequest

| Property | Type |
| --- | --- |
| mediaUrl | string |

#### Party

| Property | Type |
| --- | --- |
| id? | string | null |
| size? | number[] | null |

#### Reaction

| Property | Type |
| --- | --- |
| count | number |
| me | boolean |
| emoji | Emoji |

#### Relationship

| Property | Type |
| --- | --- |
| type | number |
| user | User |

#### Secrets

| Property | Type |
| --- | --- |
| join? | string |
| match? | string |

#### SetActivityRequest

| Property | Type |
| --- | --- |
| activity | Activity |

#### SetConfigRequest

| Property | Type |
| --- | --- |
| use_interactive_pip | boolean |

#### SetConfigResponse

| Property | Type |
| --- | --- |
| use_interactive_pip | boolean |

#### SetOrientationLockStateRequest

| Property | Type |
| --- | --- |
| lock_state | OrientationLockState |
| picture_in_picture_lock_state | OrientationLockState |
| grid_lock_state | OrientationLockState |

#### ShareLinkRequest

| Property | Type |
| --- | --- |
| custom_id? | string |
| message | string |

#### ShareLinkResponse

| Property | Type |
| --- | --- |
| success | boolean |

#### Sku

| Property | Type |
| --- | --- |
| id | string |
| name | string |
| type | SkuTypeObject |
| price | SkuPrice |
| application_id | string |
| flags | number |
| release_date | string | null |

#### SkuPrice

| Property | Type |
| --- | --- |
| amount | number |
| currency | string |

#### StartPurchaseRequest

| Property | Type |
| --- | --- |
| sku_id | string |

#### StartPurchaseResponse

| Value |
| --- |
| Entitlement[] | null |

#### Timestamp

| Property | Type |
| --- | --- |
| start? | number |
| end? | number |

#### User

| Property | Type |
| --- | --- |
| id | string |
| username | string |
| discriminator | string |
| global_name? | string | null |
| avatar? | string | null |
| avatar_decoration_data | AvatarDecorationData | null |
| bot | boolean |
| flags? | number | null |
| premium_type? | number | null |

#### UserSettingsGetLocaleResponse

| Property | Type |
| --- | --- |
| locale | string |

#### UserVoiceState

| Property | Type |
| --- | --- |
| mute | boolean |
| nick | string |
| user | User |
| voice_state | VoiceState |
| volume | number |

#### Video

| Property | Type |
| --- | --- |
| url? | string | null |
| height? | number | null |
| width? | number | null |

#### VoiceState

| Property | Type |
| --- | --- |
| mute | boolean |
| deaf | boolean |
| self_mute | boolean |
| self_deaf | boolean |
| suppress | boolean |

### SDK Enums

#### ChannelTypesObject

| Name | Value |
| --- | --- |
| UNHANDLED | -1 |
| DM | 1 |
| GROUP_DM | 3 |
| GUILD_TEXT | 0 |
| GUILD_VOICE | 2 |
| GUILD_CATEGORY | 4 |
| GUILD_ANNOUNCEMENT | 5 |
| GUILD_STORE | 6 |
| ANNOUNCEMENT_THREAD | 10 |
| PUBLIC_THREAD | 11 |
| PRIVATE_THREAD | 12 |
| GUILD_STAGE_VOICE | 13 |
| GUILD_DIRECTORY | 14 |
| GUILD_FORUM | 15 |

#### ConsoleLevel

| Value |
| --- |
| ’error' |
| 'log' |
| 'warn' |
| 'debug' |
| 'info’ |

#### OrientationLockStateTypeObject

| Name | Value |
| --- | --- |
| UNHANDLED | -1 |
| UNLOCKED | 1 |
| PORTRAIT | 2 |
| LANDSCAPE | 3 |

#### ThermalStateTypeObject

| Name | Value |
| --- | --- |
| UNHANDLED | -1 |
| NOMINAL | 0 |
| FAIR | 1 |
| SERIOUS | 2 |
| CRITICAL | 3 |

#### OrientationTypeObject

| Name | Value |
| --- | --- |
| UNHANDLED | -1 |
| PORTRAIT | 0 |
| LANDSCAPE | 1 |

#### LayoutModeTypeObject

| Name | Value |
| --- | --- |
| UNHANDLED | -1 |
| FOCUSED | 0 |
| PIP | 1 |
| GRID | 2 |

#### OAuthScopes

| Value |
| --- |
| ’bot' |
| 'rpc' |
| 'identify' |
| 'connections' |
| 'email' |
| 'guilds' |
| 'guilds.join' |
| 'guilds.members.read' |
| 'gdm.join' |
| 'messages.read' |
| 'rpc.notifications.read' |
| 'rpc.voice.write' |
| 'rpc.voice.read' |
| 'rpc.activities.write' |
| 'webhook.incoming' |
| 'applications.commands' |
| 'applications.builds.upload' |
| 'applications.builds.read' |
| 'applications.store.update' |
| 'applications.entitlements' |
| 'relationships.read' |
| 'activities.read' |
| 'activities.write' |
| 'dm_channels.read’ |

#### RPCCloseCodes

| Name | Code |
| --- | --- |
| CLOSE_NORMAL | 1000 |
| CLOSE_UNSUPPORTED | 1003 |
| CLOSE_ABNORMAL | 1006 |
| INVALID_CLIENTID | 4000 |
| INVALID_ORIGIN | 4001 |
| RATELIMITED | 4002 |
| TOKEN_REVOKED | 4003 |
| INVALID_VERSION | 4004 |
| INVALID_ENCODING | 4005 |

#### SkuTypeObject

| Name | Value |
| --- | --- |
| UNHANDLED | -1 |
| APPLICATION | 1 |
| DLC | 2 |
| CONSUMABLE | 3 |
| BUNDLE | 4 |
| SUBSCRIPTION | 5 |

#### Relationship Types

| Value | Name | Description |
| --- | --- | --- |
| 0 | None | No relationship exists between the user and the other user. |
| 1 | Friend | The user and the other user are friends. |
| 2 | Blocked | The current user blocked the target user. |
| 3 | Pending Incoming | The target user sent the current user a friend request that is still pending. |
| 4 | Pending Outgoing | The current user sent the target user a friend request that is still pending. |
| 5 | Implicit | Implicit is listed so the type is visible, but SDK code should not use it. |
| 6 | Suggestion | Suggestion is listed so the type is visible, but SDK code should not use it. |
