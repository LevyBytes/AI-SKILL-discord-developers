# Voice Connections (part 2)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

##### Authentication Tag

 AES128-GCM encryption produces an authentication tag that is shortened to 8 bytes. Libraries differ: some let callers request that tag size directly, while others return a 12-byte tag and require dropping the 4 least-significant bytes.

##### AEAD Additional Data

 OPUS frames are encrypted in their entirety, and no extra data is supplied to the AEAD.

### Transport Encryption and Sending Voice

 E2EE calls still keep transport encryption between the client and the selective forwarding unit (SFU). Audio sent to discord should be encoded as [Opus](https://www.opus-codec.org/) with stereo channels and a 48kHz sample rate. Voice Data uses a [RTP Header](https://www.rfcreader.com/#rfc3550_line548) followed by encrypted Opus audio data. For voice encryption, use the key from [Opcode 4 Session Description](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice); build the nonce from the 12 byte header plus 12 null bytes so it reaches the 24 bytes required by xsalsa20_poly1305. Discord performs encryption with the [libsodium](https://download.libsodium.org/doc/) encryption library.

### Transport Encryption Modes

| Mode | Key | Nonce | Status |
| --- | --- | --- | --- |
| AEAD AES256-GCM (RTP Size) | aead_aes256_gcm_rtpsize | 32-bit incremental integer value, added to the end of the payload | Available (Preferred) |
| AEAD XChaCha20 Poly1305 (RTP Size) | aead_xchacha20_poly1305_rtpsize | 32-bit incremental integer value, added to the end of the payload | Available (Required) |
| XSalsa20 Poly1305 Lite (RTP Size) | xsalsa20_poly1305_lite_rtpsize | 32-bit incremental integer value, added to the end of the payload | Deprecated |
| AEAD AES256-GCM | aead_aes256_gcm | 32-bit incremental integer value, added to the end of the payload | Deprecated |
| XSalsa20 Poly1305 | xsalsa20_poly1305 | Copy of RTP header | Deprecated |
| XSalsa20 Poly1305 Suffix | xsalsa20_poly1305_suffix | 24 random bytes | Deprecated |
| XSalsa20 Poly1305 Lite | xsalsa20_poly1305_lite | 32-bit incremental integer value, added to the end of the payload | Deprecated |

 The deprecated encryption modes above stop being supported on November 18th, 2024. Starting on that date, the voice gateway rejects connections that choose one of those deprecated modes. Remove the nonce from the payload before both encryption and decryption of audio data. RTP size variants compute the unencrypted RTP header length [the same way as SRTP](https://tools.ietf.org/html/rfc3711#section-3.1), treating CSRCs and, when present, the extension preamble as part of that unencrypted header. Deprecated variants instead use a fixed unencrypted RTP header size. [Opcode 2 Ready](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) tells the client which encryption modes the voice gateway supports. Voice-gateway-compatible modes always contain `aead_xchacha20_poly1305_rtpsize`; `aead_aes256_gcm_rtpsize` may be absent depending on hardware. Clients must implement `aead_xchacha20_poly1305_rtpsize`, and should choose `aead_aes256_gcm_rtpsize` when Discord offers it. Send the chosen mode in [Opcode 1 Select Protocol](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice). The voice server then returns [Opcode 4 Session Description](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) with the `mode` and `secret_key`; `secret_key` is a 32 byte array for [transport encryption and sending](https://docs.discord.com/developers/topics/voice-connections#transport-encryption-and-sending-voice) voice data:

###### Example Session Description Payload

```
{
  "op": 4,
  "d": {
      "mode": "aead_aes256_gcm_rtpsize",
      "secret_key": [ ...251, 100, 11...],
      "dave_protocol_version": 1
  }
}

```

 At this point, the client can encrypt audio and send voice packets across the UDP connection that was already established.

###### Voice Packet Structure

| Field | Type | Size |
| --- | --- | --- |
| Version + Flags | Single byte value of `0x80` | 1 byte |
| Payload Type | Single byte value of `0x78` | 1 byte |
| Sequence | Unsigned short (big endian) | 2 bytes |
| Timestamp | Unsigned integer (big endian) | 4 bytes |
| SSRC | Unsigned integer (big endian) | 4 bytes |
| Encrypted audio | Binary data | n bytes |

### Speaking

 Speaking updates change the client **speaking modes**. Set the mode with an [Opcode 5 Speaking](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) payload. Before transmitting audio, the client must send this payload at least once so the SSRC is updated and the initial speaking mode is set. The flags below can be combined as a bitwise mask; for instance, `5` means priority plus voice. Use a nonzero speaking mode when audio will be sent.
| Flag | Meaning | Value |
| --- | --- | --- |
| Microphone | Normal transmission of voice audio | `1 << 0` |
| Soundshare | Sends context audio for video without showing a speaking indicator | `1 << 1` |
| Priority | Priority speaker mode, which lowers audio from other speakers | `1 << 2` |

###### Example Speaking Payload

```
{
  "op": 5,
  "d": {
    "speaking": 5,
    "delay": 0,
    "ssrc": 1
  }
}

```

 Send at least one [Opcode 5 Speaking](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) payload before transmitting voice data; otherwise Discord disconnects the client with an invalid SSRC error. Bots using the voice gateway should set `delay` to `0`.

### Voice Data Interpolation

 When outgoing data pauses, do not just stop packet delivery. First transmit five silence frames (`0xF8, 0xFF, 0xFE`) so later transmissions do not trigger unwanted Opus interpolation.

### Resuming Voice Connection

 After the client notices that its connection was lost, it should create another WebSocket connection. Once that connection is open, send an [Opcode 7 Resume](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) payload:

###### Example Resume Connection Payload Since V8

```
{
  "op": 7,
  "d": {
    "server_id": "41771983423143937",
    "session_id": "my_session_id",
    "token": "my_token",
    "seq_ack": 10
  }
}

```

###### Example Resume Connection Payload Before V8

```
{
  "op": 7,
  "d": {
    "server_id": "41771983423143937",
    "session_id": "my_session_id",
    "token": "my_token"
  }
}

```

 On success, the Voice server answers with [Opcode 9 Resumed](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice), indicating that the client session has resumed:

###### Example Resumed Payload

```
{
  "op": 9,
  "d": null
}

```

 If resuming fails, such as when the session is invalid, the WebSocket closes with the relevant [close event code](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice-voice-close-event-codes). After that, reconnect by following the [Connecting](https://docs.discord.com/developers/topics/voice-connections#connecting-to-voice) flow.

### Buffered Resume

 In voice gateway version 8 and later, the gateway can replay buffered messages that were missed during resume. To make that possible, messages that may need replay include a sequence number.

###### Example Message With Sequence Number

```
{
  "op": 5,
  "d": {
    "speaking": 0,
    "delay": 0,
    "ssrc": 110
  },
  "seq": 10
}

```

 A client on voice gateway version 8 must put the last received sequence number in the data `d` object as `seq_ack` for both [Opcode 3 Heartbeat](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) and [Opcode 7 Resume](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice). If the client has not received any sequence-numbered messages, it may leave `seq_ack` out or send `-1`. The gateway uses a fixed-width sequence number and wraps it when needed. Because voice gateway messages are ordered, the client only has to remember the most recent sequence number. After a successful resume, the voice gateway sends [Opcode 9 Resumed](https://docs.discord.com/developers/topics/opcodes-and-status-codes#voice) and re-sends messages the client missed. Resume can fail when the session buffer no longer contains a missed message; in that situation the session closes and the client should reconnect with the [Connecting](https://docs.discord.com/developers/topics/voice-connections#connecting-to-voice) flow.

### IP Discovery

 Routers on the Internet commonly hide or rewrite UDP ports through NAT. Most voice implementations should use IP discovery to learn the external IP and port used to receive voice traffic. To obtain that external IP and port, send this UDP packet to the voice port; all numeric values are big endian:
| Field | Description | Size |
| --- | --- | --- |
| Type | 0x1 means request; 0x2 means response | 2 bytes |
| Length | Message length without the Type and Length fields; value is 70 | 2 bytes |
| SSRC | Unsigned integer | 4 bytes |
| Address | Null-terminated string in response | 64 bytes |
| Port | Unsigned short | 2 bytes |
