# Reference — Reference (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

**Base URL**

```
https://discord.com/api

```

### API Versioning

 Some API and Gateway versions no longer work and are shown as discontinued in the table only for historical reference. Requests using those versions fail with 400 Bad Request. Discord offers multiple API versions[.](https://c.tenor.com/BuZl66EegkgAAAAC/westworld-dolores.gif) Choose a version by putting it in the route, for example `https://discord.com/api/v{version_number}`. If the path omits the version, Discord uses the current default version shown below. The newest API version change log is available [here](https://docs.discord.com/developers/change-log).  **API Versions**
| Version | Status | Default |
| --- | --- | --- |
| 10 | Available |  |
| 9 | Available |  |
| 8 | Deprecated |  |
| 7 | Deprecated |  |
| 6 | Deprecated |  |
| 5 | Discontinued |  |
| 4 | Discontinued |  |
| 3 | Discontinued |  |

### Error Messages

 Beginning with API v8, form error responses use a more detailed format. The response identifies the JSON key with the problem, provides the error code, and includes a human-readable message. Discord adds new error messages often, so a full catalog would become stale quickly; the following examples illustrate the structure instead: **Array Error**

```
{
  "code": 50035,
  "errors": {
    "activities": {
      "0": {
        "platform": {
          "_errors": [
            {
              "code": "BASE_TYPE_CHOICES",
              "message": "Value must be one of ('desktop', 'android', 'ios')."
            }
          ]
        },
        "type": {
          "_errors": [
            {
              "code": "BASE_TYPE_CHOICES",
              "message": "Value must be one of (0, 1, 2, 3, 4, 5)."
            }
          ]
        }
      }
    }
  },
  "message": "Invalid Form Body"
}

```

 **Object Error**

```
{
  "code": 50035,
  "errors": {
    "access_token": {
      "_errors": [
        {
          "code": "BASE_TYPE_REQUIRED",
          "message": "This field is required"
        }
      ]
    }
  },
  "message": "Invalid Form Body"
}

```

 **Request Error**

```
{
  "code": 50035,
  "message": "Invalid Form Body",
  "errors": {
    "_errors": [
      {
        "code": "APPLICATION_COMMAND_TOO_LARGE",
        "message": "Command exceeds maximum size (8000)"
      }
    ]
  }
}

```

### Authentication

 The Discord API supports two authentication approaches:
1. Use a bot token from the Bot page in the application's settings. For bot background, see [bots vs user accounts](https://docs.discord.com/developers/topics/oauth2#bot-vs-user-accounts).
2. Use an OAuth2 bearer token obtained through the [OAuth2 API](https://docs.discord.com/developers/topics/oauth2).
 Every authentication method uses the `Authorization` HTTP header with the form `Authorization: TOKEN_TYPE TOKEN`. **Example Bot Token Authorization Header**

```
Authorization: Bot YOUR_BOT_TOKEN

```

 **Example Bearer Token Authorization Header**

```
Authorization: Bearer YOUR_BEARER_TOKEN

```

### Encryption

 Discord API HTTP-level protocols and services, including HTTP and WebSocket, use TLS 1.2.

### Snowflakes

 Discord uses Twitter's [snowflake](https://github.com/twitter-archive/snowflake/tree/snowflake-2010) format for unique IDs. These IDs are unique throughout Discord except for certain special cases where child objects use the same ID as the parent. Snowflakes can be up to 64 bits, such as a uint64, so the HTTP API returns them as strings to avoid integer overflow in some languages. See [Gateway ETF/JSON](https://docs.discord.com/developers/events/gateway#encoding-and-compression) for Gateway encoding details. **Snowflake ID Broken Down in Binary**

```
111111111111111111111111111111111111111111 11111 11111 111111111111
64                                         22    17    12          0

```

 **Snowflake ID Format Structure (Left to Right)**
| Field | Bits | Number of bits | Description | Retrieval |
| --- | --- | --- | --- | --- |
| Timestamp | 63 to 22 | 42 bits | Elapsed milliseconds from Discord Epoch, the first second of 2015 or 1420070400000. | `(snowflake >> 22) + 1420070400000` |
| Internal worker ID | 21 to 17 | 5 bits |  | `(snowflake & 0x3E0000) >> 17` |
| Internal process ID | 16 to 12 | 5 bits |  | `(snowflake & 0x1F000) >> 12` |
| Increment | 11 to 0 | 12 bits | Incremented for each ID produced by that process | `snowflake & 0xFFF` |

### Convert Snowflake to DateTime

### Snowflake IDs in Pagination

 Many API routes use snowflake IDs for pagination. The common pagination model accepts `before` and `after` IDs together with `limit` to fetch the desired page. Always check the specific endpoint for exact behavior. Since a snowflake is numeric and includes a timestamp, you can generate a snowflake for a point in time when paginating from the beginning of time, from Discord Epoch where `0` also works, or around a particular timestamp. **Generating a snowflake ID from a Timestamp Example**

```
(timestamp_ms - DISCORD_EPOCH) << 22

```

### ID Serialization

 In some situations, the API or Gateway can return IDs in a format you might not expect. Discord stores IDs internally as integer snowflakes. During JSON serialization, `bigints` become strings. Since Discord IDs are snowflakes, clients should normally expect strings. However, if you send a value to the API or Gateway in an `id` field that is not `bigint` sized, Discord may serialize that same value back as an integer. One example is requesting `GUILD_MEMBERS_CHUNK` from the gateway:

```
// Send
{
  op: 8,
  d: {
    guild_id: '308994132968210433',
    user_ids: [ '123123' ]
  }
}

// Receive
{
  t: 'GUILD_MEMBERS_CHUNK',
  s: 3,
  op: 0,
  d: {
    not_found: [ 123123 ],
    members: [],
    guild_id: '308994132968210433'
  }
}

```

 In that example, the submitted `user_id` is not a `bigint`, so Discord does not convert it to a string when serializing it back to JSON. IDs generated by Discord do not behave this way. It can occur only when malformed request data is sent.

### ISO8601 Date/Time

 Most Date/Time values in Discord models use the [ISO8601 format](https://www.loc.gov/standards/datetime/iso-tc154-wg5_n0038_iso_wd_8601-1_2016-02-16.pdf). Documentation tables refer to this type as `ISO8601`.

### Nullable and Optional Resource Fields

 Fields that may be `null` use a type prefixed with a question mark. Optional fields use a question mark suffix on the field name. **Example Nullable and Optional Fields**
| Field | Type |
| --- | --- |
| optional_field? | string |
| nullable_field | ?string |
| optional_and_nullable_field? | ?string |

### Consistency

 At Discord's scale, strict consistency cannot be guaranteed. Many API and service operations are [eventually consistent](https://en.wikipedia.org/wiki/Eventual_consistency). Client actions therefore cannot be serialized and may run in any order, or may not run. Under these constraints, Discord events may:
- Never reach a client
- Reach the client exactly once
- Reach each client up to N times
 Clients should process API results and events in an idempotent manner wherever possible.

### HTTP API

### User Agent

 HTTP API clients must send a valid [User Agent](https://www.rfc-editor.org/rfc/rfc9110.html#section-10.1.5) that identifies the client library and version using this format: **User Agent Example**

```
User-Agent: DiscordBot ($url, $versionNumber)

```

 Clients may add more metadata to the end of that string. Requests without a valid User Agent may be blocked and can return a [Cloudflare error](https://support.cloudflare.com/hc/en-us/articles/360029779472-Troubleshooting-Cloudflare-1XXX-errors).

### Content Type

 HTTP API clients must provide a valid `Content-Type` header unless an endpoint states otherwise. Accepted values are `application/json`, `application/x-www-form-urlencoded`, or `multipart/form-data`. Missing or invalid content type can produce error `50035` "Invalid form body".

### Rate Limiting

 The HTTP API uses request limiting to control excessive traffic in line with [RFC 6585](https://tools.ietf.org/html/rfc6585#section-4). API users that repeatedly hit limits and ignore them can lose API keys and be blocked from the platform. See [Rate Limits](https://docs.discord.com/developers/topics/rate-limits) for more about request rate limiting.

### Boolean Query Strings

 Some API endpoints document boolean query string parameters. Because query strings have no universal boolean standard, Discord accepts `True`, `true`, or `1` for true and `False`, `false` or `0` for false.

### Array Query Strings

 Some endpoints accept arrays in query string parameters. Unless an endpoint says otherwise, Discord represents an array by repeating the same query parameter, such as `?id=123&id=456` for `["123", "456"]`.

### Gateway (WebSocket) API

 Discord's Gateway API maintains persistent, stateful websocket connections between clients and Discord servers. Clients use those connections to send and receive real-time events and keep local state current. The Gateway API uses secure websocket connections under [RFC 6455](https://tools.ietf.org/html/rfc6455). For connection setup, see [Gateway API](https://docs.discord.com/developers/events/gateway#connections).

### Message Formatting

 Discord clients render message content using a subset of markdown plus custom formats for features such as user and channel mentions. The supported forms are listed below:  **Formats**
| Type | Structure | Example |
| --- | --- | --- |
| User | `<@USER_ID>` | `<@80351110224678912>` |
| User * | `<@!USER_ID>` | `<@!80351110224678912>` |
| Channel | `<#CHANNEL_ID>` | `<#103735883630395392>` |
| Role | `<@&ROLE_ID>` | `<@&165511591545143296>` |
| Slash command | `</NAME:COMMAND_ID>` | `</airhorn:816437322781949972>` |
| Slash command with subcommand | `</NAME SUBCOMMAND:ID>` | `</foo bar:123456789012345678>` |
| Slash command with subcommand group | `</NAME SUBCOMMAND_GROUP SUBCOMMAND:ID>` | `</foo group bar:123456789012345678>` |
| Standard emoji | Unicode characters | 🪴 |
| Custom emoji | `<:NAME:ID>` | `<:mmLol:216154654256398347>` |
| Animated custom emoji | `<a:NAME:ID>` | `<a:b1nzy:392938283556143104>` |
| Unix timestamp | `<t:TIMESTAMP>` | `<t:1618953630>` |
| Styled unix timestamp | `<t:TIMESTAMP:STYLE>` | `<t:1618953630:d>` |
| Guild navigation | `<id:TYPE>` | See below |

 Markdown forms for users or roles create mentions and may notify targets depending on the sender's permissions and the `allowed_mentions` value used when creating the message. Standard emoji render with [Twemoji](https://github.com/jdecked/twemoji) on Desktop and Android, while iOS uses Apple's native emoji set. Timestamps are in **seconds** and render according to the viewer's timezone and locale. * User mentions that include an exclamation point are deprecated and should be treated like normal user mentions.  **Timestamp Styles**
| Style | Example Output | Description |
| --- | --- | --- |
| t | 16:20 | Short Time |
| T | 16:20:30 | Medium Time |
| d | 20/04/2021 | Short Date |
| D | April 20, 2021 | Long Date |
| f * | April 20, 2021 at 16:20 | Long Date, Short Time |
| F | Tuesday, April 20, 2021 at 16:20 | Full Date, Short Time |
| s | 20/04/2021, 16:20 | Short Date, Short Time |
| S | 20/04/2021, 16:20:30 | Short Date, Medium Time |
| R | 4 years ago | Relative Time |

 *default **Guild Navigation Types** Guild navigation types point users to the related resource inside the current server.
| Full Syntax | Linked Resource |
| --- | --- |
| `<id:customize>` | Channel & Roles tab showing Onboarding prompts |
| `<id:browse>` | Browse Channels tab |
| `<id:guide>` | Server Guide tab |
| `<id:linked-roles>` | Linked Roles tab |
| `<id:linked-roles:id>` | A particular linked role; clicking opens the connection modal, and the second `id` is the role id |

### Image Formatting

 **Image Base URL**

```
https://cdn.discordapp.com/

```

 Discord renders client images from ids and hashes. API calls such as [Get User](https://docs.discord.com/developers/resources/user#get-user) can return those hashes. The tables below describe image formats, size limits, and CDN endpoints. To choose the returned format, swap the [extension name](https://docs.discord.com/developers/reference#image-formatting-image-formats) at the URL suffix. Change returned size by adding `?size=desired_size`. Valid image sizes are powers of two from 16 through 4096. Animated WebP or AVIF uploads do not reliably become GIF files. Apps should request animated images as WebP because that works regardless of the uploaded format.  **Image Formats**
| Name | Extension |
| --- | --- |
| JPEG | .jpg, .jpeg |
| PNG | .png |
| WebP | .webp |
| GIF | .gif |
| Lottie | .json |

  **CDN Endpoints**
| Type | Path | Supports |
| --- | --- | --- |
| Custom Emoji | emojis/emoji_id.png ***** | PNG, JPEG, WebP, GIF |
| Guild Icon | icons/guild_id/guild_icon.png * | PNG, JPEG, WebP, GIF |
| Guild Splash | splashes/guild_id/guild_splash.png | PNG, JPEG, WebP |
| Guild Discovery Splash | discovery-splashes/guild_id/guild_discovery_splash.png | PNG, JPEG, WebP |
| Guild Banner | banners/guild_id/guild_banner.png * | PNG, JPEG, WebP, GIF |
| User Banner | banners/user_id/user_banner.png * | PNG, JPEG, WebP, GIF |
| Default User Avatar | embed/avatars/index.png ** *** | PNG |
| User Avatar | avatars/user_id/user_avatar.png * | PNG, JPEG, WebP, GIF |
| Guild Member Avatar | guilds/guild_id/users/user_id/avatars/member_avatar.png * | PNG, JPEG, WebP, GIF |
| Avatar Decoration | avatar-decoration-presets/avatar_decoration_data_asset.png | PNG |
| Application Icon | app-icons/application_id/icon.png | PNG, JPEG, WebP |
| Application Cover | app-icons/application_id/cover_image.png | PNG, JPEG, WebP |
| Application Asset | app-assets/application_id/asset_id.png | PNG, JPEG, WebP |
| Achievement Icon | app-assets/application_id/achievements/achievement_id/icons/icon_hash.png | PNG, JPEG, WebP |
| Store Page Asset | app-assets/application_id/store/asset_id | PNG, JPEG, WebP |
| Sticker Pack Banner | app-assets/710982414301790216/store/sticker_pack_banner_asset_id.png | PNG, JPEG, WebP |
| Team Icon | team-icons/team_id/team_icon.png | PNG, JPEG, WebP |
| Sticker | stickers/sticker_id.png *** **** | PNG, Lottie, GIF |
| Role Icon | role-icons/role_id/role_icon.png | PNG, JPEG, WebP |
| Guild Scheduled Event Cover | guild-events/scheduled_event_id/scheduled_event_cover_image.png | PNG, JPEG, WebP |
| Guild Member Banner | guilds/guild_id/users/user_id/banners/member_banner.png * | PNG, JPEG, WebP, GIF |
| Guild Tag Badge | guild-tag-badges/guild_id/badge_hash.png | PNG, JPEG, WebP |

 * For endpoints that support GIFs, an animated-format hash starts with `a_`, for example `a_1269e74af4df7417b13759eae50c83dc`. Retrieve those images as animated WebP with the `.webp` extension and the `?animated=true` query parameter. ** For the Default User Avatar endpoint, `index` depends on whether the user has migrated to the [new username system](https://docs.discord.com/developers/change-log#unique-usernames-on-discord). New username users set `index` to `(user_id >> 22) % 6`; legacy username users set `index` to `discriminator % 5`. *** For Default User Avatar and Sticker endpoints, image size is fixed and the "size" query parameter is ignored. **** For the Sticker endpoint, a sticker is available as PNG when its [`format_type`](https://docs.discord.com/developers/resources/sticker#sticker-object) is `PNG` or `APNG`, as GIF when `format_type` is `GIF`, and as [Lottie](https://airbnb.io/lottie/#/) when `format_type` is `LOTTIE`. ***** For Custom Emoji, Discord strongly recommends requesting WebP for performance and compatibility. Emoji uploads can be JPEG, PNG, GIF, WebP, or AVIF. WebP and AVIF uploads must be requested as WebP because they do not convert well to other formats. The Discord client displays all in-app emojis as WebP. See [Emoji Resource](https://docs.discord.com/developers/resources/emoji) for details. Sticker GIFs use `https://media.discordapp.net/stickers/<sticker_id>.gif` instead of the CDN base URL.

### Image Data

 Image data uses the [Data URI scheme](https://en.wikipedia.org/wiki/Data_URI_scheme) and supports JPG, GIF, and PNG. Example Data URI syntax:

```
data:image/jpeg;base64,BASE64_ENCODED_JPEG_IMAGE_DATA

```

 Use the content type that matches the supplied image data, such as `image/jpeg`, `image/png`, or `image/gif`.

### **Signed Attachment CDN URLs**

 Attachments stored on Discord's CDN, including user-uploaded and bot-uploaded images, use signed URLs with a fixed expiration. Discord refreshes attachment CDN URLs shown in the client automatically, so a signed URL received in a payload, such as after you [fetch a message](https://docs.discord.com/developers/resources/message#get-channel-message), is valid at receipt time. When an API field accepts a CDN URL, such as [`url` in an embed image object](https://docs.discord.com/developers/resources/message#embed-object-embed-image-structure) or [`avatar_url` for webhooks](https://docs.discord.com/developers/resources/webhook#execute-webhook-jsonform-params), the app can pass the CDN URL without parameters and Discord will render and refresh it. The [standard CDN endpoints](https://docs.discord.com/developers/reference#image-formatting-cdn-endpoints) above are unsigned and do not expire.

###### Example Attachment CDN URL

```
https://cdn.discordapp.com/attachments/1012345678900020080/1234567891233211234/my_image.png?ex=65d903de&is=65c68ede&hm=2481f30dd67f503f54d020ae3b5533b9987fae4e55f2b4e3926e08a3fa3ee24f&

```

###### Attachment CDN URL Parameters

| Parameter | Description |
| --- | --- |
| ex | Hex timestamp for when an attachment CDN URL expires |
| is | Hex timestamp for when the URL was created |
| hm | Unique signature valid until the URL expiration time |

### Uploading Files

 The upload limit applies per file in a request. The default is `10 MiB` for all users, with possible higher limits based on [Nitro](https://support.discord.com/hc/en-us/articles/115000435108-What-are-Nitro-Nitro-Basic) status or server [Boost Tier](https://support.discord.com/hc/en-us/articles/360028038352-Server-Boosting-FAQ-#h_419c3bd5-addd-4989-b7cf-c7957ef92583). The `attachment_size_limit` value [when working with interactions](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-object-interaction-structure) is the maximum of those values. Endpoints that accept uploads mention the `files[n]` parameter. To attach files, replace the normal `application/json` body with a `multipart/form-data` body. You may include the JSON message body as `payload_json`. Every `files[n]` part needs a valid `Content-Disposition` subpart header with `filename` and a unique `name`. File names must use the form `files[n]`, for example `files[0]`, `files[1]`, or `files[42]`. The `n` suffix is the snowflake placeholder referenced by the `attachments` field, which can be included in `payload_json` or [Callback Data Payloads](https://docs.discord.com/developers/interactions/receiving-and-responding#interaction-response-object-interaction-callback-data-structure). Embeds can reference uploaded images with `attachment://filename`. An example payload follows.

### Editing Message Attachments

 The `attachments` JSON parameter identifies every file to append to the message, including new files and their snowflake placeholders. In a `PATCH` request, only files present in `attachments` are appended; any previous files omitted from the parameter are removed. **Example Request Bodies (multipart/form-data)** These examples show small HTTP request fragments to demonstrate endpoint behavior. Client libraries set their own form boundaries; `boundary` is only an example. For details, see the [multipart/form-data spec](https://tools.ietf.org/html/rfc7578#section-4). The first example uses the endpoint without `payload_json`.

```
--boundary
Content-Disposition: form-data; name="content"

Hello, World!
--boundary
Content-Disposition: form-data; name="tts"

true
--boundary--

```

 This example uses the endpoint with `payload_json` and includes all content fields: `content`, `embeds`, and `files[n]`.

```
--boundary
Content-Disposition: form-data; name="payload_json"
Content-Type: application/json

{
  "content": "Hello, World!",
  "embeds": [{
    "title": "Hello, Embed!",
    "description": "This is an embedded message.",
    "thumbnail": {
      "url": "attachment://myfilename.png"
    },
    "image": {
      "url": "attachment://mygif.gif"
    }
  }],
  "message_reference": {
    "message_id": "233648473390448641"
  },
  "attachments": [{
      "id": 0,
      "description": "Image of a cute little cat",
      "filename": "myfilename.png"
  }, {
      "id": 1,
      "description": "Rickroll gif",
      "filename": "mygif.gif"
  }]
}
--boundary
Content-Disposition: form-data; name="files[0]"; filename="myfilename.png"
Content-Type: image/png

[image bytes]
--boundary
Content-Disposition: form-data; name="files[1]"; filename="mygif.gif"
Content-Type: image/gif

[image bytes]
--boundary--

```

 **Using Attachments within Embeds** A message can upload attachments and then use them inside an embed. Upload the files in the `multipart/form-data` body and ensure every uploaded file has a filename, because the payload must reference it. At this time, supported file types are `.jpg`, `.jpeg`, `.png`, `.webp`, and `.gif`; other file types are unsupported. In an embed object, set an image URL to the attachment scheme form `attachment://filename.png`. Example:

```
{
  "embeds": [{
    "image": {
      "url": "attachment://screenshot.png"
    }
  }]
}

```

### Locales
