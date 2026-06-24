# Gateway (part 3)

> Discord Developers reference. Original prose; identifiers preserved verbatim.

```
shard_id: 0, rate limit key (0 % 16): 0
shard_id: 1, rate limit key (1 % 16): 1
shard_id: 2, rate limit key (2 % 16): 2
shard_id: 3, rate limit key (3 % 16): 3
shard_id: 4, rate limit key (4 % 16): 4
shard_id: 5, rate limit key (5 % 16): 5
shard_id: 6, rate limit key (6 % 16): 6
shard_id: 7, rate limit key (7 % 16): 7
shard_id: 8, rate limit key (8 % 16): 8
shard_id: 9, rate limit key (9 % 16): 9
shard_id: 10, rate limit key (10 % 16): 10
shard_id: 11, rate limit key (11 % 16): 11
shard_id: 12, rate limit key (12 % 16): 12
shard_id: 13, rate limit key (13 % 16): 13
shard_id: 14, rate limit key (14 % 16): 14
shard_id: 15, rate limit key (15 % 16): 15
shard_id: 16, rate limit key (16 % 16): 0
shard_id: 17, rate limit key (17 % 16): 1
shard_id: 18, rate limit key (18 % 16): 2
shard_id: 19, rate limit key (19 % 16): 3
shard_id: 20, rate limit key (20 % 16): 4
shard_id: 21, rate limit key (21 % 16): 5
shard_id: 22, rate limit key (22 % 16): 6
shard_id: 23, rate limit key (23 % 16): 7
shard_id: 24, rate limit key (24 % 16): 8
shard_id: 25, rate limit key (25 % 16): 9
shard_id: 26, rate limit key (26 % 16): 10
shard_id: 27, rate limit key (27 % 16): 11
shard_id: 28, rate limit key (28 % 16): 12
shard_id: 29, rate limit key (29 % 16): 13
shard_id: 30, rate limit key (30 % 16): 14
shard_id: 31, rate limit key (31 % 16): 15

```

 For this situation, shard buckets must be launched **in "order"**. You may start shard 0 -> shard 15 at the same time, then proceed with shard 16 -> shard 31.

### Sharding for Large Bots

 Bots present in more than 150,000 guilds have extra sharding requirements. As a bot approaches the large bot sharding threshold, Discord moves it to large bot sharding. The bot owner(s) receive a system DM and an email after the migration is complete, including the assigned shard number. The shard count you run must be a multiple of the shard number Discord provides during that contact. Starting the bot with a shard count that is not valid causes the Gateway connection to close with `4010` Invalid Shard. The [Get Gateway Bot endpoint](https://docs.discord.com/developers/events/gateway#get-gateway-bot) always reports the correct shard count, so implementations that already use that endpoint to choose shard count should not need changes. For these bots, the daily session start limit also rises from 1000 to `max(2000, (guild_count / 1000) * 5)`. Discord also increases `max_concurrency`, which is the number of [shards you can concurrently start](https://docs.discord.com/developers/events/gateway#session-start-limit-object).

### Get Gateway

  Authentication is not required for this endpoint. It returns an object containing a valid WSS URL that the app can use when [Connecting](https://docs.discord.com/developers/events/gateway#connecting) to the Gateway. Apps should keep the value cached and request another URL only when the cached URL cannot establish a connection correctly.

###### Example Response

```
{
  "url": "wss://gateway.discord.gg/"
}

```

### Get Gateway Bot

  This endpoint requires authentication with a valid bot token. The response contains the data from [Get Gateway](https://docs.discord.com/developers/events/gateway#get-gateway) plus additional metadata useful for operating large or [sharded](https://docs.discord.com/developers/events/gateway#sharding) bots. Unlike [Get Gateway](https://docs.discord.com/developers/events/gateway#get-gateway), this route should not be cached for long periods because the returned value may differ between calls and changes as the bot joins or leaves guilds.

###### JSON Response

| Field | Type | Description |
| --- | --- | --- |
| url | string | WSS URL for connecting to the Gateway |
| shards | integer | Suggested shard count for the connection |
| session_start_limit | session_start_limit object | Current session start limit details |

###### Example Response

```
{
  "url": "wss://gateway.discord.gg/",
  "shards": 9,
  "session_start_limit": {
    "total": 1000,
    "remaining": 999,
    "reset_after": 14400000,
    "max_concurrency": 1
  }
}

```

### Session Start Limit Object

###### Session Start Limit Structure

| Field | Type | Description |
| --- | --- | --- |
| total | integer | Total session starts permitted for the current user |
| remaining | integer | Session starts still available to the current user |
| reset_after | integer | Milliseconds until the limit resets |
| max_concurrency | integer | Identify requests permitted per 5 seconds |
