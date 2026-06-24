# Voice Connections (part 1)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Voice connections resemble a [Gateway](https://docs.discord.com/developers/events/gateway) connection, but they use their own payload set plus a separate UDP connection for carrying voice data. Since UDP handles both inbound and outbound voice traffic, the client *must* be able to receive UDP packets through any firewall or NAT; see [UDP Hole Punching](https://en.wikipedia.org/wiki/UDP_hole_punching). Discord Voice servers include [IP Discovery](https://docs.discord.com/developers/topics/voice-connections#ip-discovery), which can determine the local machine's remote UDP IP/Port and help with some network setups.

### Voice Gateway Versioning

 Use [version 8](https://docs.discord.com/developers/topics/voice-connections#voice-gateway-versioning-gateway-versions) for current behavior. Without it, the documented [Opcodes](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) may not match what arrives over the socket. The voice gateway previously defaulted to version 1 when the `v` query string was omitted. That fallback is deprecated; future connections must provide a voice gateway version or they will be rejected. Versions below 4 and the default-version fallback will be discontinued on November 18th, 2024. From that date, connections with no version or a version lower than version 4 will be rejected.

###### Gateway Versions

| Version | Status | WebSocket URL Append | Change |
| --- | --- | --- | --- |
| 8 | recommended | ?v=8 | Added buffering for server messages, with missed messages delivered again during resume |
| 7 | available | ?v=7 | Added channel options opcode |
| 6 | available | ?v=6 | Added code version opcode |
| 5 | available | ?v=5 | Added video sink wants opcode |
| 4 | available | ?v=4 | Changed speaking status from boolean to bitmask |
| 3 | deprecated | ?v=3 | Added video |
| 2 | deprecated | ?v=2 | Changed the server heartbeat reply to the heartbeat ACK opcode |
| 1 | deprecated | ?v=1 | Initial version |

### Connecting to Voice

### Retrieving Voice Server Information

 Connecting to a voice server, and therefore to a guild voice channel, starts by sending a request through the [Gateway](https://docs.discord.com/developers/events/gateway). The gateway returns the voice server information needed for the connection. Discord's voice platform is broadly distributed, so clients **should never** cache or persist the result of this call. To tell the gateway that voice connectivity is desired, send an [Opcode 4 Gateway Voice State Update](https://docs.discord.com/developers/topics/opcodes-and-status-codes#gateway):

###### Gateway Voice State Update Example

```
{
  "op": 4,
  "d": {
    "guild_id": "41771983423143937",
    "channel_id": "127121515262115840",
    "self_mute": false,
    "self_deaf": false
  }
}

```

 When the request succeeds, the gateway sends *two* events: a [Voice State Update](https://docs.discord.com/developers/events/gateway-events#voice-state-update) event and a [Voice Server Update](https://docs.discord.com/developers/events/gateway-events#voice-server-update) event. Your library must wait for both before continuing. The first includes a new `session_id`, and the second carries the voice server data needed to create the voice connection:

###### Example Voice Server Update Payload

```
{
  "t": "VOICE_SERVER_UPDATE",
  "s": 2,
  "op": 0,
  "d": {
    "token": "my_token",
    "guild_id": "41771983423143937",
    "endpoint": "sweetwater-12345.discord.media:2048"
  }
}

```

 After receiving these values, open the voice WebSocket connection. A voice state update that moves channels inside the same guild can produce a VOICE_SERVER_UPDATE with the same `endpoint` as the prior VOICE_SERVER_UPDATE. The `token` still changes, and the previous session cannot be reused during a channel change even when the endpoint is unchanged. Bot users obey the voice channel's user limit when one is set. If the channel is full, your own Voice State Update will not produce Voice State Update or Voice Server Update events. The `MOVE_MEMBERS` permission bypasses that limit and allows joining whether or not the channel is full.

### Establishing a Voice Websocket Connection

 After retrieving session_id, token, and endpoint data, connect to the voice server and complete the handshake over a second secure WebSocket. Unlike the gateway endpoint returned by the HTTP [Get Gateway](https://docs.discord.com/developers/events/gateway#get-gateway) request, the endpoint in the [Voice Server Update](https://docs.discord.com/developers/events/gateway-events#voice-server-update) payload does not include a URL protocol, so some libraries need `wss://` prepended manually. Once connected to the voice WebSocket endpoint, send an [Opcode 0 Identify](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) payload containing server_id, user_id, session_id, and token:

###### Example Voice Identify Payload

```
{
  "op": 0,
  "d": {
    "server_id": "41771983423143937",
    "user_id": "104694319306248192",
    "session_id": "my_session_id",
    "token": "my_token",
    "max_dave_protocol_version": 1
  }
}

```

 The voice server should answer with an [Opcode 2 Ready](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) payload that provides the expected `SSRC`, UDP IP/port, and supported encryption modes:

###### Example Voice Ready Payload

```
{
  "op": 2,
  "d": {
    "ssrc": 1,
    "ip": "127.0.0.1",
    "port": 1234,
    "modes": ["xsalsa20_poly1305", "xsalsa20_poly1305_suffix", "xsalsa20_poly1305_lite"],
    "heartbeat_interval": 1
  }
}

```

 The `heartbeat_interval` field shown here is incorrect and should be ignored. Use the `heartbeat_interval` value from the Hello payload instead.

### Heartbeating

 To keep the WebSocket connection alive, continue sending heartbeats at the interval supplied by [Opcode 8 Hello](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice):

###### Example Hello Payload

```
{
  "op": 8,
  "d": {
    "heartbeat_interval": 41250,
  }
}

```

 After [Opcode 8 Hello](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) arrives, send [Opcode 3 Heartbeat](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) with an integer nonce on each elapsed interval:

###### Example Heartbeat Payload since V8

```
{
  "op": 3,
  "d": {
    "t": 1501184119561,
    "seq_ack": 10
  }
}

```

 For voice gateway version 8 and later, heartbeat messages must include `seq_ack`, which carries the sequence number of the last numbered gateway message received. See [Buffered Resume](https://docs.discord.com/developers/topics/voice-connections#buffered-resume) for details.

###### Example Heartbeat Payload below V8

```
{
  "op": 3,
  "d": 1501184119561
}

```

 Discord replies with an [Opcode 6 Heartbeat ACK](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) containing the nonce that was sent:

###### Example Heartbeat ACK Payload since V8

```
{
  "op": 6,
  "d": {
    "t": 1501184119561
  }
}

```

###### Example Heartbeat ACK Payload below V8

```
{
  "op": 6,
  "d": 1501184119561
}

```

### Establishing a Voice UDP Connection

 After the [Opcode 2 Ready](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) payload provides the UDP voice server properties, the last voice-connection step is to create and handshake a UDP connection for voice data.

###### Example Ready Payload

```
{
  "op": 2,
  "d": {
    ...,
    "modes": ["aead_aes256_gcm_rtpsize", "aead_xchacha20_poly1305_rtpsize", "xsalsa20_poly1305_lite_rtpsize"]
  }
}

```

 Begin by opening a UDP connection to the IP and port from the Ready payload. If needed, run [IP Discovery](https://docs.discord.com/developers/topics/voice-connections#ip-discovery) through that connection. Once the external IP and UDP port are known, report them to the voice WebSocket and begin receiving and sending data by using [Opcode 1 Select Protocol](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice):

###### Example Select Protocol Payload

```
{
  "op": 1,
  "d": {
    "protocol": "udp",
    "data": {
      "address": "127.0.0.1",
      "port": 1337,
      "mode": "aead_aes256_gcm_rtpsize"
    }
  }
}

```

### End-to-End Encryption (DAVE Protocol)

 Since September 2024, Discord has been rolling end-to-end encryption (E2EE) into DM calls, Group DM calls, voice-channel conversations, and Go Live sessions. To meet long-term privacy goals, Discord will require E2EE after **March 1st, 2026** anywhere Discord carries audio or video in DMs, GDMs, voice channels, or Go Live streams.

 Implement DAVE support as early as possible for a smoother migration. This section gives a high-level view of Discord's audio & video end-to-end encryption (DAVE) protocol, focused on the voice gateway opcodes needed by the protocol. The most complete DAVE protocol documentation is the [Protocol Whitepaper](https://daveprotocol.com). The open-source [libdave](https://github.com/discord/libdave) library may also help with implementation. The exact DAVE protocol opcode format is covered in the [Voice Gateway Opcodes section of the protocol whitepaper](https://daveprotocol.com/#voice-gateway-opcodes). In an E2EE call, every member exchanges keys through a [Messaging Layer Security](https://www.rfc-editor.org/rfc/rfc9420.html) (MLS) group. That group derives per-sender ratcheted media keys, known only to group participants, for encrypting and decrypting media frames in the call.

### Binary Websocket Messages

 Some new DAVE protocol opcodes are binary rather than JSON text to reduce overhead. Use the binary column in the [Voice Opcodes Table](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) to identify them. Binary websocket messages use this layout:
| Field | Description | Size |
| --- | --- | --- |
| Sequence Number | OPTIONAL server -> client big-endian uint16 sequence number | 2 bytes |
| Opcode | Unsigned integer opcode value | 1 bytes |
| Payload | Binary payload for the message, with format determined by opcode | Variable bytes |

 Only server-to-client binary messages include sequence numbers, and every server-sent binary opcode requires that number. See [Resuming Voice Connection](https://docs.discord.com/developers/topics/voice-connections#resuming-voice-connection) for how sequence numbers are used when present.

### Indicating DAVE Protocol Support

 In [Opcode 0 Identify](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice), include the highest DAVE protocol version you support as `max_dave_protocol_version`. Sending version 0 or omitting `max_dave_protocol_version` indicates no DAVE protocol support. The voice gateway reports the starting protocol version in [Opcode 4 Session Description](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) as `dave_protocol_version`. It may choose any non-discontinued protocol version at or below the supported version. Clients must remain backwards-compatible with any non-discontinued DAVE protocol version. The voice gateway chooses the lowest protocol version shared by the call.

### Protocol Transitions

 The voice gateway coordinates protocol version changes and MLS group transitions so media continuity is preserved during the call. This can happen when a call moves to or from E2EE during the initial transition phase, changes protocol version, or changes MLS group membership. Some opcodes carry a transition ID. After preparing the local state needed for the transition, send [Opcode 23 DAVE Protocol Transition Ready](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) to tell the voice gateway you are ready to execute it. When every participant is ready or a timeout occurs, the voice gateway sends [Opcode 22 DAVE Protocol Execute Transition](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) to confirm execution. Execution tells media senders to begin using the new protocol context, such as no E2EE after downgrade, a new protocol version after a version change, or a new key ratchet after a participant change.

#### Downgrade

 Downgrades to protocol version 0 are announced with [Opcode 21 DAVE Protocol Prepare Transition](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice). This can happen during the transition phase when a client without protocol support joins the call. Once the transition executes, senders should stop sending media in the protocol format.

#### Protocol Version Change & Upgrade

 Protocol version transitions, including upgrades from protocol version 0, are announced through [Opcode 24 DAVE Protocol Prepare Epoch](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice). Along with `transition_id`, this opcode provides the `epoch` for the upcoming MLS epoch. Receiving [Opcode 24 DAVE Protocol Prepare Epoch](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) with `epoch = 1` means a new MLS group is being created. Participants must:
- prepare a local MLS group using parameters that match the DAVE protocol version
- generate and send [Opcode 26 DAVE MLS Key Package](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) so the voice gateway receives a new MLS key package
 When `epoch` is greater than 1, the existing MLS group is changing protocol version. After the transition executes, senders must send media with the new protocol context, such as the new protocol-version format or a new key ratchet.

#### MLS Group Changes

 When MLS group participants need to change, existing participants receive [Opcode 29 DAVE MLS Announce Commit Transition](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice), while new members being added receive [Opcode 30 DAVE MLS Welcome](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice). Both opcodes include the transition ID and a binary MLS Commit or MLS Welcome message. Existing group members prepare for the protocol transition by applying the commit and advancing their local MLS group to the correct next state. [Opcode 23 DAVE Protocol Transition Ready](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) is sent after the MLS commit is processed. Welcomed members send [Opcode 23 DAVE Protocol Transition Ready](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) after successfully joining the group from the MLS Welcome message.

### External Sender

 The voice gateway acts as the MLS group's external sender so it can submit MLS proposals for participant joins and leaves, such as adding a newly connected member or removing one that disconnected. DAVE protocol participants accept proposals only from that external sender, and the sender is limited to Add and Remove proposals. Through [Opcode 25 DAVE MLS External Sender Package](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice), the voice gateway delivers the sender credential and public key to MLS group participants. The package can arrive right after the voice gateway connects or later during an upgrade to DAVE protocol. Group creators include the gateway-provided external sender in the MLS group extensions, and welcomed members check that the expected extension is present before joining.

### Joining the MLS Group

 Except for creating the call's first group, joining the MLS group always follows receipt of [Opcode 30 DAVE MLS Welcome](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice).

#### Key Packages

 Pending members send an MLS key package through [Opcode 26 DAVE MLS Key Package](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) so they can be proposed for addition to the MLS group. Key packages are single-use, and each time a pending member waits to be added or re-added to the group, a new key package must be generated.

##### Identity Public Key

 Each MLS participant signs and authenticates MLS messages with an asymmetric keypair. Its public key appears in both the key package and the MLS tree, is visible to other call participants, and can be used for out-of-band identity verification. You may generate a fresh ephemeral keypair for each protocol call or reuse one persistent keypair.

#### Initial Group

 If an MLS group has not been established yet, for example during an upgrade from transport-only encryption or at the start of a fresh two-person call, every pending member builds a local group using the DAVE protocol version parameters and the external sender received from [Opcode 25 DAVE MLS External Sender Package](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice). Any pending member may produce the first commit that creates the MLS group at `epoch = 1`. The voice gateway sends each pending member Add proposals for the other pending participants; if another participant arrives before the group exists, that participant also receives the in-flight proposals. Proposal and commit handling is the same whether a group already exists or not. See [Proposals and Commits](https://docs.discord.com/developers/topics/voice-connections#proposals-and-commits).

#### Welcome

 Pending group members receive a welcome message from another group member that adds them to the MLS group. The voice gateway dispatches it with [Opcode 30 DAVE MLS Welcome](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice).

#### Invalid Group

 If the group from [Opcode 30 DAVE MLS Welcome](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) or [Opcode 29 DAVE MLS Announce Commit Transition](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) cannot be processed, the receiving member sends [Opcode 31 DAVE MLS Invalid Commit Welcome](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) to the voice gateway. The member also resets local group state, creates a new key package, and sends it to the voice gateway via [Opcode 26 DAVE MLS Key Package](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice). That causes the voice gateway to propose removing and re-adding the requesting member.

### Proposals and Commits

 The voice gateway dispatches proposals through [Opcode 27 DAVE MLS Proposals](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice), and members must append or revoke them. All members of the established or pending MLS group append or revoke received proposals, then create an MLS commit and, when committing add proposals for new members, optionally an MLS welcome message. They send those messages to the voice gateway through [Opcode 28 DAVE MLS Commit Welcome](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice). For each epoch, the voice gateway dispatches the winning commit through [Opcode 29 DAVE MLS Announce Commit Transition](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) and may dispatch associated welcome messages through [Opcode 30 DAVE MLS Welcome](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice). The first valid commit and welcome(s) seen for that epoch are broadcast, while later commits for the stale epoch are dropped. A valid commit must include every unrevoked proposal dispatched in that epoch. A valid welcome must welcome every member added during the epoch.

### Audio Frame E2EE

 Transport encryption works at the packet level, while DAVE protocol E2EE works at the frame level. When a call enables any DAVE protocol, call participants send and receive OPUS frames whose full contents are end-to-end encrypted.

#### ULEB128 Encoding

 Some protocol frame payload fields use [ULEB128 encoding](https://en.wikipedia.org/wiki/LEB128#Unsigned_LEB128), a variable-length code that stores arbitrarily large unsigned integers compactly.

#### Payload Format

| Field | Description | Size |
| --- | --- | --- |
| E2EE OPUS Frame | Ciphertext for the E2EE OPUS frame | Variable bytes |
| AES-GCM Auth. Tag | Truncated AES128-GCM AEAD Authentication Tag | 8 bytes |
| ULEB128 Nonce | ULEB128 synchronization nonce | Variable bytes |
| ULEB128 Unencrypted Ranges | ULEB128 offset/length pairs for unencrypted data in the E2EE frame | Variable bytes |
| Supplemental Data Size | Unsigned integer byte size for supplemental data | 1 byte |
| Magic Marker | 0xFAFA marker used to help identify protocol frames | 2 bytes |

 The E2EE OPUS frame contains the ciphertext output from the AES128-GCM AEAD process described below. The authentication tag stores the AEAD tag truncated to 8 bytes. The ULEB128 nonce encodes the encryption/decryption nonce as a compact variable-length value. For OPUS frames, the ULEB128 unencrypted ranges field has no contents (0 bytes), because the complete frame is encrypted. The supplemental data size is the total byte count needed for:
- 8-byte authentication tag
- Variable length ULEB128 nonce
- Variable length ULEB128 unencrypted ranges
- 1 byte supplemental data size
- 2 byte magic marker
 The magic marker is the constant 2-byte value 0xFAFA. Media receivers use it to detect protocol frames, and the SFU uses it to avoid forwarding protocol frames to receivers without protocol support during transition periods.

#### Frame Encryption

 Media frames use AES128-GCM for E2EE encryption.

##### Sender Key Derivation

 Every media sender has a ratcheted per-sender key, with a new ratchet for each MLS group epoch. The sender ratchet starts from a 16-byte secret exported by the MLS group. Key lookup uses a generation counter calculated from the high byte of the 4-byte nonce. For very long lived epochs, implementations must handle nonce wrap-around without allowing the generation to return to 0. See the [Sender Key Derivation section of the Protocol Whitepaper](https://daveprotocol.com/#sender-key-derivation) for the full process.

##### Nonce

 AES128-GCM encryption and decryption receive a full 12-byte nonce, but the protocol uses no more than 4-bytes. Expand a 4-byte truncated nonce into 12 bytes by setting the 8 most significant bytes to zero and storing the truncated nonce value in the 4 least significant bytes. The sender key ratchet generation comes from the most-significant byte of the 4-byte nonce, which is the 4th least significant byte of the full 12-byte nonce.
