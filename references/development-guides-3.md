# Multiplayer Experience, Networking, Production Readiness

> Discord Developers reference. Original prose; identifiers preserved verbatim.

## Overview

Covers: Activity Instance Management; Activity Proxy Considerations; Cache Busting.

## Activity Instance Management

### Activity Instance Management

 When someone selects `Join Application`, they expect to land in the same running experience as their friends. That may be a drawing canvas, board game, shared playlist, shooter, or another collaborative app, but all participants need access to the same shared state. These docs call that shared state an **application instance**. (Figure: join-application) The Embedded App SDK provides two-way communication between your app and the Discord Client. `instanceId` lets both your application and Discord identify the particular application instance being addressed.

#### Using instanceId

 The `instanceId` property is available immediately after SDK construction; the SDK does not need to receive a `ready` payload from the Discord client first.

```
import {DiscordSDK} from '@discord/embedded-app-sdk';
const discordSdk = new DiscordSDK(clientId);
// available immediately
const instanceId = discordSdk.instanceId;

```

 Use `instanceId` as the key for storing and retrieving the shared data for an application instance. That makes everyone in the same instance see the same data.

##### Semantics of instanceId

 An instance ID is created when a user launches an application. Other users who join that same application instance receive the same `instanceId`. After every user in that channel leaves or closes the application, the instance lifecycle ends and the ID is not reused. A later launch in that channel receives a new `instanceId`.

---

### Instance Participants

 Instance Participants are Discord users currently connected to the same Application Instance. Apps can fetch this data or subscribe to changes.

```
import {DiscordSDK, Events, type Types} from '@discord/embedded-app-sdk';

const discordSdk = new DiscordSDK('...');
await discordSdk.ready();

// Fetch
const participants = await discordSdk.commands.getInstanceConnectedParticipants();

// Subscribe
function updateParticipants(participants: Types.GetActivityInstanceConnectedParticipantsResponse) {
  // Do something really cool
}
discordSdk.subscribe(Events.ACTIVITY_INSTANCE_PARTICIPANTS_UPDATE, updateParticipants);
// Unsubscribe
discordSdk.unsubscribe(Events.ACTIVITY_INSTANCE_PARTICIPANTS_UPDATE, updateParticipants);

```

---

### Render Avatars and Names

 Detailed CDN and common image asset storage behavior is documented [here](https://docs.discord.com/developers/reference#image-formatting-cdn-endpoints). The following basic example retrieves a user's avatar and username.

```
// We'll be referencing the user object returned from authenticate
const {user} = await discordSdk.commands.authenticate({
  access_token: accessToken,
});

let avatarSrc = '';
if (user.avatar) {
  avatarSrc = `https://cdn.discordapp.com/avatars/${user.id}/${user.avatar}.png?size=256`;
} else {
  const defaultAvatarIndex = (BigInt(user.id) >> 22n) % 6n;
  avatarSrc = `https://cdn.discordapp.com/embed/avatars/${defaultAvatarIndex}.png`
}

const username = user.global_name ?? `${user.username}#${user.discriminator}`;

// Then in your HTML/JSX/etc...
<img alt="avatar" src={avatarSrc} />
<p>{username}</p>

```

#### Rendering guild-specific avatars and nicknames

 To read a user's guild-specific avatar and nickname, request the `guilds.members.read` scope. That scope only grants guild member information for the current application user. If you need to display guild-specific avatars or nicknames for every application user, share data obtained from `guilds.members.read` scoped API calls through your application's server. This example fetches the user's guild-specific avatar and nickname:

```
// We'll be referencing the user object returned from authenticate
const {user} = await discordSdk.commands.authenticate({
  access_token: accessToken,
});

// When using the proxy, you may instead replace `https://discord.com` with `/discord`
// or whatever url mapping you have chosen via the developer portal
fetch(`https://discord.com/api/users/@me/guilds/${discordSdk.guildId}/member`, {
  method: 'GET',
  headers: {
    Authorization: `Bearer ${accessToken}`,
  },
})
  .then((response) => {
    return response.json();
  })
  .then((guildsMembersRead) => {
    let guildAvatarSrc = '';
    // Retrieve the guild-specific avatar, and fallback to the user's avatar
    if (guildsMembersRead?.avatar) {
      guildAvatarSrc = `https://cdn.discordapp.com/guilds/${discordSdk.guildId}/users/${user.id}/avatars/${guildsMembersRead.avatar}.png?size=256`;
    } else if (user.avatar) {
      guildAvatarSrc = `https://cdn.discordapp.com/avatars/${user.id}/${user.avatar}.png?size=256`;
    } else {
      const defaultAvatarIndex = (BigInt(user.id) >> 22n) % 6n;
      avatarSrc = `https://cdn.discordapp.com/embed/avatars/${defaultAvatarIndex}.png`;
    }

    // Retrieve the guild-specific nickname, and fallback to the username#discriminator
    const guildNickname = guildsMembersRead?.nick ?? (user.global_name ?? `${user.username}#${user.discriminator}`);
  });

```

 The example performs all work on the client, but a more typical implementation does this instead:
- Store the user's access token on the application server
- Fetch the user's guild-specific avatar and nickname from the application's server
- Serve every application's avatar and nickname through the application's server

---

### Preventing unwanted activity sessions

 Activities run inside iframes in the Discord app. The activity site is also publicly reachable at `<application_id>.discordsays.com`. Activities expect to communicate with Discord's web or mobile client through the Discord SDK's RPC protocol. If someone opens the activity website in a normal browser, no Discord RPC server is present, so the activity will usually fail. A malicious client could theoretically fake Discord's RPC protocol or load one activity site while launching another. Since the activity is loaded inside Discord, the RPC protocol is available and the activity may not detect the mismatch.

### Using the Activity Instance API

 To restrict activity access, use the `get_activity_instance` API at `discord.com/api/applications/<application_id>/activity-instances/<instance_id>'`. The route requires the application's Bot token. If the active activity instance exists for that application, the API returns a serialized representation; otherwise it returns 404. Two example responses follow:

```
curl https://discord.com/api/applications/1215413995645968394/activity-instances/i-1234567890-gc-912952092627435520-912954213460484116 -H 'Authorization: Bot <bot token>'
{"message": "404: Not Found", "code": 0}

curl https://discord.com/api/applications/1215413995645968394/activity-instances/i-1276580072400224306-gc-912952092627435520-912954213460484116 -H 'Authorization: Bot <bot token>'
{"application_id":"1215413995645968394","instance_id":"i-1276580072400224306-gc-912952092627435520-912954213460484116","launch_id":"1276580072400224306","location":{"id":"gc-912952092627435520-912954213460484116","kind":"gc","channel_id":"912954213460484116","guild_id":"912952092627435520"},"users":["205519959982473217"]}

```

 This API lets the activity backend confirm that a client really belongs to an instance of that activity before enabling meaningful gameplay. The exact `session verification` design is up to the developer; it can guard individual features or block serving the activity HTML unless the session is valid.

###### Validating Proxy Request Headers

 For applications that need extra request validation, Discord offers optional proxy authentication. When an embedded app sends traffic through Discord's proxy, requests can carry cryptographic headers proving authenticity. A proxy-authenticated request includes these headers:
- `X-Signature-Ed25519` containing the cryptographic signature
- `X-Signature-Timestamp` containing a Unix timestamp
- `X-Discord-Proxy-Payload` containing base64-encoded user context
 If you enable proxy authentication, validate those headers to confirm requests are authentic. Return `401` when signature validation fails.

Validating Proxy Headers

Example code for validating proxy authentication headers

The following code samples demonstrate validation of headers included with proxy-authenticated requests. **JavaScript**

```
const nacl = require("tweetnacl");

// Your public key can be found on your application in the Developer Portal
const PUBLIC_KEY = "APPLICATION_PUBLIC_KEY";

const signature = req.get("X-Signature-Ed25519");
const timestamp = req.get("X-Signature-Timestamp");
const payload = req.get("X-Discord-Proxy-Payload");

// Decode the base64 payload
const payloadBytes = Buffer.from(payload, "base64");
const payloadString = payloadBytes.toString("utf-8");
const payloadData = JSON.parse(payloadString);

// Verify timestamp matches payload
if (payloadData.created_at.toString() !== timestamp) {
    return res.status(401).end("invalid request timestamp");
}

// Check if token has expired
if (payloadData.expires_at < Math.floor(Date.now() / 1000)) {
    return res.status(401).end("expired proxy token");
}

// Verify the signature using tweetnacl
const isVerified = nacl.sign.detached.verify(
    payloadBytes,
    Buffer.from(signature, "base64"),
    Buffer.from(PUBLIC_KEY, "hex")
);

if (!isVerified) {
    return res.status(401).end("invalid request signature");
}

```

**Python**

```
import json
import base64
import time
from nacl.signing import VerifyKey
from nacl.exceptions import BadSignatureError

# Your public key can be found on your application in the Developer Portal
PUBLIC_KEY = 'APPLICATION_PUBLIC_KEY'

verify_key = VerifyKey(bytes.fromhex(PUBLIC_KEY))

signature = request.headers["X-Signature-Ed25519"]
timestamp = request.headers["X-Signature-Timestamp"]
payload = request.headers["X-Discord-Proxy-Payload"]

# Decode the base64 payload
payload_bytes = base64.b64decode(payload)
payload_string = payload_bytes.decode('utf-8')
payload_data = json.loads(payload_string)

# Verify timestamp matches payload
if str(payload_data['created_at']) != timestamp:
    abort(401, 'invalid request timestamp')

# Check if token has expired
if payload_data['expires_at'] < int(time.time()):
    abort(401, 'expired proxy token')

try:
    verify_key.verify(payload_bytes, bytes.fromhex(signature))
except BadSignatureError:
    abort(401, 'invalid request signature')

```

 Proxy authentication is optional and acts as an additional security layer for apps that use it. The flow diagram below shows a server returning the activity website only for valid users inside a valid activity instance: (Figure: activity-instance-validation)

## Activity Proxy Considerations

### Activity Proxy Considerations

 For security reasons, network traffic goes through the Discord Proxy. Internally this uses Cloudflare Workers, which introduces the limitations below.

#### WebTransport

 Only websockets are currently supported, but Discord is working with upstream providers to make WebTransport possible.

#### WebRTC

 WebRTC is not supported.

---

### Construct A Full URL

 Sometimes a network request needs a full URL instead of a relative URL such as `/path/to/my/thing`. Build that URL from these parts:
1. The protocol to use
2. Your application’s client id
3. The discord proxy domain
4. Whatever you need to list
 This example builds a full URL with the URL constructor:

```
const protocol = `https`;
const clientId = '<YOUR CLIENT ID>';
const proxyDomain = 'discordsays.com';
const resourcePath = '/foo/bar.jpg';
const url = new URL(`${protocol}://${clientId}.${proxyDomain}${resourcePath}`);

```

 For example, with application client id `12345678`
| Relative Path | Full Path |
| --- | --- |
| /foo/bar.jpg | https://12345678.discordsays.com/foo/bar.jpg |

---

### Using External Resources

 Discord activities are sandboxed through a Discord proxy. The proxy hides user IP addresses and blocks URLs associated with malicious endpoints. In the Developer Portal, applications configure [URL Mappings](https://docs.discord.com/developers/activities/development-guides/local-development#url-mapping) for this proxy behavior. One edge case is third-party NPM modules or other assets that reference external, unmapped URLs. If a module tries to request [https://foo.library.com](https://foo.library.com), for example, the request fails with `blocked:csp`. Possible ways to handle this include:
- Fork the library so it uses mapped urls
- Apply a post-install tool such as [patch-package](https://www.npmjs.com/package/patch-package)
- Call the Embedded App SDK's `patchUrlMappings` API
 For that scenario, `patchUrlMappings` is the recommended option because it helps the app move cleanly from an unsandboxed development environment to production. The call accepts an array of `mappings` that rewrite external network requests to the mappings you define. See this example:
- In this example, assume a third-party library makes an HTTP request to foo.com
- In the developer portal, define this mapping: `/foo` -> `foo.com`
- Then initialize the SDK and make the function call in your code.

```
import {patchUrlMappings} from '@discord/embedded-app-sdk';
const isProd = process.env.NODE_ENV === 'production'; // Actual dev/prod env check may vary for you

async function setupApp() {
  if (isProd) {
    patchUrlMappings([{prefix: '/foo', target: 'foo.com'}]);
  }
  // start app initialization after this....
}

```

 Note: `patchUrlMappings` changes the browser globals `fetch`, `WebSocket`, and `XMLHttpRequest.prototype.open`. Depending on the library, that helper can cause side effects. Use it only when you need it.

---

### Security Considerations

#### Trusting Client Data

 Treat data from the Discord client as untrusted. It may be useful locally in your app, but assume anything from the Discord Client can be forged, including the current user, Nitro status, current channel, and similar context. For trusted values, call the Discord API from your application server using the OAuth2 user token. Also remember that Discord client data is not pre-sanitized. Usernames, channel names, and similar strings are arbitrary user input, so sanitize them or render them safely with `.textContent`, for example.

#### Using Cookies

 For cookies sent through the proxy, set the cookie domain to your app's full `{clientId}.discordsays.com` domain. Also set `SameSite=None Partitioned`. `SameSite=None` is required because browsers otherwise reject or withhold cookies with stricter settings during iframe navigation. `Partitioned` restricts the cookie to Discord iframes. Other activities cannot use your activity's cookie because the Content Security Policy (CSP) limits requests to your app's own proxy.

## Cache Busting

### Cache Busting

 Application assets follow [cache headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control). The exception is Discord's application proxy, which strips cache headers from assets whose `content-type` includes `text/html`. For non-`text/html` content, use cache busting. Build tooling often provides this; if JavaScript or CSS uses static filenames, add cache-busting behavior, such as [webpack enables creating dynamic file names](https://webpack.js.org/guides/caching/).

---

### Handling Rate Limits

 Ensure your application's client and server requests can honor Discord API rate limits [as described here](https://docs.discord.com/developers/topics/rate-limits). [This implementation in the Activity Starter project](https://github.com/discord/embedded-app-sdk-examples/blob/main/discord-activity-starter/packages/server/src/utils.ts) shows one way to handle the `retry_after` header after a 429 response.

---

### Static IP Addresses

 If the application server uses a dynamically assigned IP address, as cloud functions commonly do, it may inherit an address previously banned by Cloudflare. When that happens, outbound traffic from that IP to Discord's API can be blocked for up to an hour. Mitigate this by routing all application server egress through a static IP address.

---

### Backward Compatibility

#### New Commands

 When the embedded-app-sdk adds commands, older Discord app versions may not support them. If an application calls a new command against an older Discord client, Discord responds with `INVALID_COMMAND`, which the application can handle like this:

```
try {
  const {permissions} = await discordSdk.commands.getChannelPermissions();

  // check permissions
  ...
} catch (error) {
  if (error.code == RPCErrorCodes.INVALID_COMMAND) {
    // This is an expected error. The Discord client doesn't support this command
    ...
  } else {
    // Unexpected error
    ...
  }
}

```
