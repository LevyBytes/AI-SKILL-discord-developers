# Opcodes And Status Codes (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

### Gateway

 Discord gateway events carry an opcode describing the payload type. Gateway connections can close; when they do, the close code identifies the reason.

###### Gateway Opcodes

| Code | Name | Client Action | Description |
| --- | --- | --- | --- |
| 0 | Dispatch | Receive | An event was dispatched. |
| 1 | Heartbeat | Send/Receive | Client heartbeat sent at regular intervals to keep the connection open. |
| 2 | Identify | Send | Opens a new session as part of the initial handshake. |
| 3 | Presence Update | Send | Update the client’s presence. |
| 4 | Voice State Update | Send | Moves the client into, out of, or between voice channels. |
| 6 | Resume | Send | Continues a disconnected session from its previous state. |
| 7 | Reconnect | Receive | Reconnect and resume the session immediately. |
| 8 | Request Guild Members | Send | Fetches offline guild member information for a large guild. |
| 9 | Invalid Session | Receive | The session was invalidated; reconnect, then identify/resume as appropriate. |
| 10 | Hello | Receive | Delivered right after connection and includes the `heartbeat_interval` value to use. |
| 11 | Heartbeat ACK | Receive | Acknowledges a heartbeat after it is received. |
| 31 | Request Soundboard Sounds | Send | Fetches soundboard sound information for a set of guilds. |
| 43 | Request Channel Info | Send | Fetches ephemeral channel data for channels in a guild. |

###### Gateway Close Event Codes

 Treat some gateway close codes as stop signals so clients do not enter broken reconnect loops. Causes include expired tokens and invalid identification. The table lists Discord's gateway close codes and flags the ones that should not be retried.
| Code | Description | Explanation | Reconnect |
| --- | --- | --- | --- |
| 4000 | Unknown error | Discord could not determine the failure. Reconnect may be worth trying. | true |
| 4001 | Unknown opcode | The client used an invalid Gateway opcode or sent a payload that does not match the opcode. | true |
| 4002 | Decode error | The client sent Discord a payload that is invalid. | true |
| 4003 | Not authenticated | A payload arrived before identification completed, or the session was invalidated. | true |
| 4004 | Authentication failed | The account token in the identify payload is wrong. | false |
| 4005 | Already authenticated | More than one identify payload was sent. | true |
| 4007 | Invalid `seq` | The resume sequence was invalid. Reconnect and open a new session. | true |
| 4008 | Rate limited | Payloads are being sent too quickly; slow down. Discord disconnects the client when this code is received. | true |
| 4009 | Session timed out | The session timed out. Reconnect and create a new session. | true |
| 4010 | Invalid shard | The identify payload used an invalid shard. | false |
| 4011 | Sharding required | The session would cover too many guilds, so the connection must be sharded before connecting. | false |
| 4012 | Invalid API version | The gateway version sent by the client is invalid. | false |
| 4013 | Invalid intent(s) | The Gateway Intent value was invalid; the bitwise value may have been calculated incorrectly. | false |
| 4014 | Disallowed intent(s) | The Gateway Intent value is not allowed, likely because the intent is not enabled or approved. | false |

### Voice

 Voice gateways use separate opcodes and close codes.

###### Voice Opcodes

| Code | Name | Sent By | Description | Binary |
| --- | --- | --- | --- | --- |
| 0 | Identify | client | Begin a voice websocket connection. |  |
| 1 | Select Protocol | client | Select the voice protocol. |  |
| 2 | Ready | server | Complete the websocket handshake. |  |
| 3 | Heartbeat | client | Keep the websocket connection alive. |  |
| 4 | Session Description | server | Describe the session. |  |
| 5 | Speaking | client and server | Indicate which users are speaking. |  |
| 6 | Heartbeat ACK | server | Acknowledges a client heartbeat that was received. |  |
| 7 | Resume | client | Resume a connection. |  |
| 8 | Hello | server | Heartbeat delay, in milliseconds. |  |
| 9 | Resumed | server | Acknowledge a successful session resume. |  |
| 11 | Clients Connect | server | One or more clients joined the voice channel. |  |
| 13 | Client Disconnect | server | A client left the voice channel. |  |
| 21 | DAVE Prepare Transition | server | A DAVE protocol downgrade is scheduled. |  |
| 22 | DAVE Execute Transition | server | Run the protocol transition that was previously announced. |  |
| 23 | DAVE Transition Ready | client | Acknowledge readiness previously announced transition |  |
| 24 | DAVE Prepare Epoch | server | A DAVE protocol version or group change is pending. |  |
| 25 | DAVE MLS External Sender | server | MLS external sender credential and public key. | X |
| 26 | DAVE MLS Key Package | client | MLS Key Package for a pending group member. | X |
| 27 | DAVE MLS Proposals | server | MLS Proposals queued for append or revocation. | X |
| 28 | DAVE MLS Commit Welcome | client | MLS Commit plus optional MLS Welcome messages. | X |
| 29 | DAVE MLS Announce Commit Transition | server | MLS Commit to process for the upcoming transition. | X |
| 30 | DAVE MLS Welcome | server | MLS Welcome for the group used in the upcoming transition. | X |
| 31 | DAVE MLS Invalid Commit Welcome | client | Report an invalid commit or welcome and request re-add. |  |

###### Voice Close Event Codes

| Code | Description | Explanation |
| --- | --- | --- |
| 4001 | Unknown opcode | You sent an invalid opcode. |
| 4002 | Failed to decode payload | The identify payload sent to the Gateway was invalid. |
| 4003 | Not authenticated | A payload was sent before Gateway identification. |
| 4004 | Authentication failed | The token in the identify payload is wrong. |
| 4005 | Already authenticated | More than one identify payload was sent. |
| 4006 | Session no longer valid | The session is no longer valid. |
| 4009 | Session timeout | Your session has timed out. |
| 4011 | Server not found | Discord cannot find the server being joined. |
| 4012 | Unknown protocol | Discord did not recognize the submitted protocol. |
| 4014 | Disconnected | Disconnects one client, such as after a kick or main gateway session drop. Do not reconnect. |
| 4015 | Voice server crashed | The server crashed. Try resuming. |
| 4016 | Unknown encryption mode | Discord did not recognize the encryption value. |
| 4017 | E2EE/DAVE protocol required | The channel requires a client with E2EE support through the DAVE Protocol. |
| 4020 | Bad request | You sent a malformed request |
| 4021 | Disconnected: Rate Limited | Disconnect caused by exceeding a rate limit. Do not reconnect. |
| 4022 | Disconnected: Call Terminated | Disconnects all clients because the call ended, such as after channel deletion or a voice server change. Do not reconnect. |

### HTTP

 The API returns semantically valid HTTP response codes that reflect request success or failure. The table lists response codes the API may return.

###### HTTP Response Codes

| Code | Meaning |
| --- | --- |
| 200 (OK) | The request completed successfully. |
| 201 (CREATED) | The entity was created successfully. |
| 204 (NO CONTENT) | The request succeeded and the response has no body. |
| 304 (NOT MODIFIED) | The entity was unchanged, so no action occurred. |
| 400 (BAD REQUEST) | The request format was invalid or could not be understood by the server. |
| 401 (UNAUTHORIZED) | The `Authorization` header is absent or invalid. |
| 403 (FORBIDDEN) | The supplied `Authorization` token lacks access to the resource. |
| 404 (NOT FOUND) | No resource exists at the specified location. |
| 405 (METHOD NOT ALLOWED) | The HTTP method is not valid for the specified location. |
| 429 (TOO MANY REQUESTS) | The request is rate limited; see Rate Limits. |
| 502 (GATEWAY UNAVAILABLE) | No gateway was available to handle the request. Wait briefly, then retry. |
| 5xx (SERVER ERROR) | The server hit an error while processing the request; this should be rare. |

### JSON

 Besides the HTTP status, the API can return a more specific error in the JSON response's `code` key. The response includes a human-readable `message` key. Some errors include extra detail through an `errors` object using [Error Messages](https://docs.discord.com/developers/reference#error-messages).

###### JSON Error Codes

| Code | Meaning |
| --- | --- |
| 0 | Generic failure, such as a malformed request body. |
| 10001 | Unknown account |
| 10002 | Unknown application |
| 10003 | Unknown channel |
| 10004 | Unknown guild |
| 10005 | Unknown integration |
| 10006 | Unknown invite |
| 10007 | Unknown member |
| 10008 | Unknown message |
| 10009 | Unknown permission overwrite |
| 10010 | Unknown provider |
| 10011 | Unknown role |
| 10012 | Unknown token |
| 10013 | Unknown user |
| 10014 | Unknown emoji |
| 10015 | Unknown webhook |
| 10016 | Unknown webhook service |
| 10020 | Unknown session |
| 10021 | Unknown Asset |
| 10026 | Unknown ban |
| 10027 | Unknown SKU |
| 10028 | Unknown Store Listing |
| 10029 | Unknown entitlement |
| 10030 | Unknown build |
| 10031 | Unknown lobby |
| 10032 | Unknown branch |
| 10033 | Unknown store directory layout |
| 10036 | Unknown redistributable |
| 10038 | Unknown gift code |
| 10049 | Unknown stream |
| 10050 | Unknown premium server subscribe cooldown |
| 10057 | Unknown guild template |
| 10059 | Unknown discoverable server category |
| 10060 | Unknown sticker |
| 10061 | Unknown sticker pack |
| 10062 | Unknown interaction |
| 10063 | Unknown application command |
| 10065 | Unknown voice state |
| 10066 | Unknown application command permissions |
| 10067 | Unknown Stage Instance |
| 10068 | Unknown Guild Member Verification Form |
| 10069 | Unknown Guild Welcome Screen |
| 10070 | Unknown Guild Scheduled Event |
| 10071 | Unknown Guild Scheduled Event User |
| 10087 | Unknown Tag |
| 10097 | Unknown sound |
| 10124 | Invite target users job is unknown because the invite has no target users. |
| 10129 | Invite target users are unknown because the invite has no target users. |
| 20001 | Bots cannot use this endpoint |
| 20002 | This endpoint is available to bots only. |
| 20009 | Explicit content cannot be delivered to the requested recipient(s). |
| 20012 | The caller is not authorized to take this action on the application. |
| 20015 | A premium subscription is required for this action. |
| 20016 | Slowmode rate limiting prevents this action. |
| 20018 | Only this account's owner may perform this action. |
| 20022 | Announcement rate limits prevent editing this message. |
| 20024 | Under minimum age |
| 20028 | The channel write rate limit has been hit for this write action. |
| 20029 | The server write rate limit has been hit for this write action. |
| 20031 | The Stage topic, server name, server description, or channel names include disallowed words. |
| 20035 | Guild premium subscription level too low |
| 30001 | Maximum number of guilds reached (100) |
| 30002 | Maximum number of friends reached (1000) |
| 30003 | The channel has reached its pin limit (50). |
| 30004 | Maximum number of recipients reached (10) |
| 30005 | The guild has reached its role limit (250). |
| 30007 | Maximum number of webhooks reached (15) |
| 30008 | Maximum number of emojis reached |
| 30010 | Maximum number of reactions reached (20) |
| 30011 | The group DM limit has been reached (10). |
| 30013 | The guild has reached its channel limit (500). |
| 30015 | The message has reached its attachment limit (10). |
| 30016 | Maximum number of invites reached (1000) |
| 30018 | The animated emoji limit has been reached. |
| 30019 | The server member limit has been reached. |
| 30030 | The server category limit has been reached (5). |
| 30031 | Guild already has a template |
| 30032 | The application command limit has been reached. |
| 30033 | The thread participant limit has been reached (1000). |
| 30034 | The daily application command creation limit has been reached (200). |
| 30035 | The maximum number of bans for non-guild members has been exceeded. |
| 30037 | The bans fetch limit has been reached. |
| 30038 | The uncompleted guild scheduled event limit has been reached (100). |
| 30039 | Maximum number of stickers reached |
| 30040 | The prune request limit has been reached. Try again later. |
| 30042 | The guild widget settings update limit has been reached. Try again later. |
| 30045 | The soundboard sound limit has been reached. |
| 30046 | The edit limit for messages older than 1 hour has been reached. Try again later. |
| 30047 | The forum channel has reached its pinned thread limit. |
| 30048 | The forum channel has reached its tag limit. |
| 30052 | The bitrate exceeds the limit for this channel type. |
| 30056 | The premium emoji limit has been reached (25). |
| 30058 | The guild has reached its webhook limit (1000). |
| 30060 | The channel permission overwrite limit has been reached (1000). |
| 30061 | This guild's channels are too large. |
| 40001 | Unauthorized. Send a valid token and retry. |
| 40002 | Account verification is required before this action can be performed. |
| 40003 | Direct messages are being opened too quickly. |
| 40004 | Send messages has been temporarily disabled |
| 40005 | The request entity is too large. Send smaller content. |
| 40006 | This feature is temporarily disabled server-side. |
| 40007 | The user is banned from this guild. |
| 40012 | Connection has been revoked |
| 40018 | Only consumable SKUs can be consumed. |
| 40019 | Sandbox entitlements are the only entitlements that can be deleted. |
| 40032 | The target user is not connected to voice. |
| 40033 | This message has already been crossposted. |
| 40041 | An application command already uses that name. |
| 40043 | Application interaction failed to send |
| 40058 | Messages cannot be sent in a forum channel. |
| 40060 | Interaction has already been acknowledged |
| 40061 | Tag names must be unique |
| 40062 | The service resource is being rate limited. |
| 40066 | No tags are available for non-moderators to set. |
| 40067 | Creating a forum post in this channel requires a tag. |
| 40074 | This resource already has an entitlement grant. |
| 40094 | This interaction has reached the follow-up message limit. |
| 40333 | Cloudflare blocked the request. Setting a proper User Agent often resolves it. |
| 50001 | Missing access |
| 50002 | Invalid account type |
| 50003 | Actions cannot be executed on a DM channel. |
| 50004 | Guild widget disabled |
| 50005 | A message authored by another user cannot be edited. |
| 50006 | Cannot send an empty message |
| 50007 | Messages cannot be sent to this user. |
| 50008 | Messages cannot be sent in a non-text channel. |
| 50009 | The channel verification level is too high to allow access. |
| 50010 | The OAuth2 application has no bot. |
| 50011 | OAuth2 application limit reached |
| 50012 | Invalid OAuth2 state |
| 50013 | Required permissions are missing for that action. |
| 50014 | Invalid authentication token provided |
| 50015 | Note was too long |
| 50016 | The delete request included too few or too many messages. Provide at least 2 and fewer than 100 messages. |
| 50017 | Invalid MFA Level |
| 50019 | A message can be pinned only in the channel where it was sent. |
| 50020 | The invite code is invalid or already taken. |
| 50021 | Actions cannot be executed on a system message. |
| 50024 | Actions cannot be executed on this channel type. |
| 50025 | Invalid OAuth2 access token provided |
| 50026 | Missing required OAuth2 scope |
| 50027 | Invalid webhook token provided |
| 50028 | Invalid role |
| 50033 | Invalid Recipient(s) |
| 50034 | One supplied message is too old for bulk delete. |
| 50035 | The form body is invalid for `application/json` or `multipart/form-data`, or the supplied `Content-Type` is invalid. |
| 50036 | The invite was accepted for a guild that does not contain the application's bot. |
| 50039 | Invalid Activity Action |
| 50041 | Invalid API version provided |
| 50045 | The uploaded file is larger than the maximum size. |
| 50046 | Invalid file uploaded |
| 50054 | Cannot self-redeem this gift |
| 50055 | Invalid Guild |
| 50057 | Invalid SKU |
| 50067 | Invalid request origin |
| 50068 | Invalid message type |
| 50070 | A payment source is required to redeem the gift. |
| 50073 | Cannot modify a system webhook |
| 50074 | A channel required for Community guilds cannot be deleted. |
| 50080 | Stickers inside a message cannot be edited. |
| 50081 | Invalid sticker sent |
| 50083 | The operation targeted an archived thread, such as editing a message or adding a user. |
| 50084 | Invalid thread notification settings |
| 50085 | The `before` value predates thread creation. |
| 50086 | Community server channels are required to be text channels. |
| 50091 | The event entity type does not match the entity being used to start the event. |
| 50095 | This server is unavailable in your location. |
| 50097 | This server must have monetization enabled before this action. |
| 50101 | This server needs additional boosts before this action. |
| 50109 | The request body contains invalid JSON. |
| 50110 | The provided file is invalid. |
| 50123 | The file type provided is invalid. |
| 50124 | The file duration is greater than the 5.2 seconds maximum. |
| 50131 | Owner cannot be pending member |
| 50132 | Ownership cannot be transferred to a bot user. |
| 50138 | Could not resize asset below the maximum size: 262144. |
| 50144 | Subscription and non subscription roles cannot be mixed for an emoji. |
| 50145 | Premium emoji and normal emoji cannot be converted into each other. |
| 50146 | Uploaded file not found. |
| 50151 | The specified emoji is invalid |
| 50159 | Voice messages cannot include additional content. |
| 50160 | Voice messages require exactly one audio attachment. |
| 50161 | Voice messages require supporting metadata. |
| 50162 | Voice messages cannot be edited. |
| 50163 | Cannot delete guild subscription integration |
| 50167 | Voice effects cannot be sent while the user is server muted, deafened, or suppressed. |
| 50173 | Voice messages cannot be sent in this channel. |
| 50178 | The user account must be verified first. |
| 50192 | The file does not have a valid duration. |
| 50278 | Messages cannot be sent to this user because there are no mutual guilds. |
| 50600 | Permission to send this sticker is missing. |
| 60003 | Two factor is required for this operation. |
| 80004 | No users with DiscordTag exist |
| 90001 | Reaction was blocked |
| 90002 | User cannot use burst reactions |
| 110000 | The index is not available yet. Try again later. |
| 110001 | The application is not available yet. Try again later. |
| 130000 | The API resource is overloaded. Try again a little later. |
| 150006 | The Stage is already open |
| 160002 | Replies require permission to read message history. |
| 160004 | This message already has a thread. |
| 160005 | Thread is locked |
| 160006 | The active thread limit has been reached. |
| 160007 | The active announcement thread limit has been reached. |
| 160014 | A message cannot be forwarded if its content cannot be read. |
| 170001 | The uploaded Lottie file contains invalid JSON. |
| 170002 | Uploaded Lotties cannot include rasterized images such as PNG or JPEG. |
| 170003 | Sticker maximum framerate exceeded |
| 170004 | The sticker frame count is above the 1000 frame maximum. |
| 170005 | Lottie animation maximum dimensions exceeded |
| 170006 | The sticker frame rate is outside the allowed range. |
| 170007 | The sticker animation duration is greater than the 5 seconds maximum. |
| 180000 | Cannot update a finished event |
| 180002 | Creating the stage required for the stage event failed. |
| 200000 | Automatic moderation blocked the message. |
| 200001 | Automatic moderation blocked the title. |
| 220001 | Forum-channel webhook posts require either `thread_name` or `thread_id`. |
| 220002 | Forum-channel webhook posts cannot include both `thread_name` and `thread_id`. |
| 220003 | Webhooks can create threads only in forum channels. |
| 220004 | Webhook services cannot be used in forum channels. |
| 240000 | The harmful links filter blocked the message. |
| 340015 | This user has limited access to joining new servers. |
| 350000 | Onboarding cannot be enabled because requirements are not met. |
| 350001 | Onboarding cannot be updated while requirements are unmet. |
| 400001 | File upload access has been limited for this guild. |
| 500000 | Failed to ban users |
| 520000 | Poll voting blocked |
| 520001 | Poll expired |
| 520002 | Poll creation does not support this channel type. |
| 520003 | Cannot edit a poll message |
| 520004 | An emoji included with the poll cannot be used. |
| 520006 | A non-poll message cannot be expired. |
| 530000 | The Discord application has not been granted permission to use provisional accounts. |
| 530001 | The provided ID token JWT is expired. Request a new one from the identity provider. |
| 530002 | The issuer in the provided ID token JWT differs from the configured issuer. |
| 530003 | The audience in the provided ID token JWT does not match the audience in the OIDC configuration. Update the configuration or supply an ID token issued to the application. |
| 530004 | The provided ID token was issued too long ago. Discord rejects ID tokens issued more than a week ago; request a new ID token from the identity provider. |
| 530006 | Discord could not generate a unique username within the allotted time. This is not terminal and should succeed after retrying. |
| 530007 | The client secret is invalid. Check the submitted value or regenerate the client secret. |

###### Example JSON Error Response

```
{
  "message": "Invalid authentication token",
  "code": 50014
}

```

### RPC

 RPC is the [local Discord server](https://docs.discord.com/developers/topics/rpc) that runs on localhost. Discord approval is required before accessing the RPC server.

###### RPC Error Codes

| Code | Name | Description |
| --- | --- | --- |
| 1000 | Unknown error | An unknown error occurred. |
| 4000 | Invalid payload | You sent an invalid payload. |
| 4002 | Invalid command | Invalid command name specified. |
| 4003 | Invalid guild | Invalid guild ID specified. |
| 4004 | Invalid event | Invalid event name specified. |
| 4005 | Invalid channel | Invalid channel ID specified. |
| 4006 | Invalid permissions | Required permissions for the resource are missing. |
| 4007 | Invalid client ID | Authorization or authentication used an invalid OAuth2 application ID. |
| 4008 | Invalid origin | Authorization or authentication used an invalid OAuth2 application origin. |
| 4009 | Invalid token | Authorization or authentication used an invalid OAuth2 token. |
| 4010 | Invalid user | The supplied user ID is invalid. |
| 5000 | OAuth2 error | A standard OAuth2 error occurred; inspect the data object for OAuth2 error details. |
| 5001 | Select channel timed out | An asynchronous `SELECT_TEXT_CHANNEL`/`SELECT_VOICE_CHANNEL` command timed out. |
| 5002 | `GET_GUILD` timed out | An asynchronous `GET_GUILD` command timed out. |
| 5003 | Select voice force required | The request attempted to join a user to a voice channel while that user was already in one. |
| 5004 | Capture shortcut already listening | The request attempted to capture more than one shortcut key at the same time. |

###### RPC Close Event Codes
