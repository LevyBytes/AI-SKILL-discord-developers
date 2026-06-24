# Rpc (part 3)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Takes no arguments. Requires the `relationships_read` [OAuth2 scope](https://docs.discord.com/developers/topics/oauth2#shared-resources-oauth2-scopes). Discord sends this event when a relationship is created, changed, such as a presence update, or deleted. If the relationship was deleted, `type` is `0` (`NONE`).

###### Relationship Update Dispatch Data Structure

| Field | Type | Description |
| --- | --- | --- |
| type | integer | relationship type |
| user | partial user object | the related user |
| presence | presence object | current presence for the related user |

###### Relationship Types

| Type | Value | Description |
| --- | --- | --- |
| NONE | 0 | relationship removed |
| FRIEND | 1 | user is a friend |
| BLOCKED | 2 | user is blocked |
| PENDING_INCOMING | 3 | incoming friend request |
| PENDING_OUTGOING | 4 | outgoing friend request |
| IMPLICIT | 5 | user shares a guild with the current user but is not a friend |

###### Presence Object

| Field | Type | Description |
| --- | --- | --- |
| status | string | user’s status (`online`, `idle`, `dnd`, `offline`) |
| activity | activity object | current activity for this application for the user (`null` when absent) |

###### Example Relationship Update Dispatch Payload

```
{
  "cmd": "DISPATCH",
  "data": {
    "type": 1,
    "user": {
      "id": "190320984123768832",
      "username": "test user 2",
      "discriminator": "0",
      "global_name": "test user 2",
      "avatar": "b004ec1740a63ca06ae2e14c5cee11f3",
      "bot": false,
      "flags": 0,
      "premium_type": 0
    },
    "presence": {
      "status": "online",
      "activity": null
    }
  },
  "evt": "RELATIONSHIP_UPDATE"
}

```

#### ENTITLEMENT_CREATE

 Takes no arguments. Sent when the user receives a new entitlement for this application.

###### Entitlement Create Dispatch Data Structure

| Field | Type | Description |
| --- | --- | --- |
| entitlement | entitlement object | the entitlement that was created |

###### Entitlement Object

| Field | Type | Description |
| --- | --- | --- |
| id | string | entitlement id |
| sku_id | string | SKU id associated with this entitlement |
| application_id | string | id of the application |
| user_id | string | id for the user who owns the entitlement |
| type | integer | entitlement type |
| deleted | boolean | whether the entitlement is marked deleted |
| starts_at? | ISO8601 | start date of the entitlement |
| ends_at? | ISO8601 | end date of the entitlement |
| guild_id? | string | id for the guild covered by the entitlement |
| consumed? | boolean | whether a consumable entitlement has been consumed |

###### Entitlement Types

| Type | Value | Description |
| --- | --- | --- |
| PURCHASE | 1 | purchased by a user |
| PREMIUM_SUBSCRIPTION | 2 | a Nitro subscription |
| DEVELOPER_GIFT | 3 | gifted by a developer |
| TEST_MODE_PURCHASE | 4 | purchased in test mode |
| FREE_PURCHASE | 5 | granted for free |
| USER_GIFT | 6 | gifted by another user |
| PREMIUM_PURCHASE | 7 | purchased as a premium feature |
| APPLICATION_SUBSCRIPTION | 8 | an app subscription |

###### Example Entitlement Create Dispatch Payload

```
{
  "cmd": "DISPATCH",
  "data": {
    "entitlement": {
      "id": "1019653849998299136",
      "sku_id": "1019475255913222144",
      "application_id": "192741864418312192",
      "user_id": "53908232506183680",
      "type": 8,
      "deleted": false,
      "starts_at": "2022-09-14T17:00:18.704163+00:00",
      "ends_at": "2022-10-14T17:00:18.704163+00:00"
    }
  },
  "evt": "ENTITLEMENT_CREATE"
}

```

#### ENTITLEMENT_DELETE

 Takes no arguments. Sent when this application loses an entitlement. The entitlement object in the payload contains the entitlement values from deletion time.

###### Entitlement Delete Dispatch Data Structure

| Field | Type | Description |
| --- | --- | --- |
| entitlement | entitlement object | the entitlement that was deleted |

###### Example Entitlement Delete Dispatch Payload

```
{
  "cmd": "DISPATCH",
  "data": {
    "entitlement": {
      "id": "1019653849998299136",
      "sku_id": "1019475255913222144",
      "application_id": "192741864418312192",
      "user_id": "53908232506183680",
      "type": 8,
      "deleted": true,
      "starts_at": "2022-09-14T17:00:18.704163+00:00",
      "ends_at": "2022-10-14T17:00:18.704163+00:00"
    }
  },
  "evt": "ENTITLEMENT_DELETE"
}

```
