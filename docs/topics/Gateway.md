# Gateway

Kettu's Gateway allows clients to receive real-time events & updates. It is fundamentally very similar to Discord's Gateway.

## Versions

The Gateway versions are consistent with the API versions, found in the main reference.

## Payload Structure

There is a consistent structure used for sending & receiving payloads across the entire Gateway, as follows.

| field | type          | description                                                |
| ----- | ------------- | ---------------------------------------------------------- |
| op    | integer       | opcode for the payload                                     |
| d     | ?mixed (JSON) | event data                                                 |
| s     | ?integer      | sequence number, increments for all dispatches (op = `0`)  |
| t     | ?string       | event name (for op = `0`)                                  |
| a     | ?boolean      | event marked as an acknowledgement                         |
| n     | ?string       | unique nonce, should be returned for bi-directional events |
| l     | ?integer      | optional time processing (in ms), used for metrics         |

The `op` key will always be provided. Depending on `op`, other keys may or may not be provided.

### Compression

At this point, all payloads are sent in plain-text JSON. Support for zlib compression is not currently high priority.

## Intents

### List of Intents

```txt
SHARDS (1 << 0)
  - SHARD_ONLINE
  - SHARD_OFFLINE
  - SHARD_RESUMING
  - SHARD_RESUMED

USERS (1 << 1)
  - USER_UPDATE
  - USER_DELETE
  - USER_REMINDER_TRIGGER
  - USER_VOTE

GUILDS (1 << 2)
  - GUILD_CREATE
  - GUILD_UPDATE
  - GUILD_DELETE

GUILD_CASES (1 << 3)
  - GUILD_CASE_CREATE
  - GUILD_CASE_UPDATE
  - GUILD_CASE_EXPIRE
  - GUILD_CASE_DELETE

GUILD_MEMBERS (1 << 4)
  - GUILD_MEMBER_UPDATE

IMAGES (1 << 5)
  - IMAGE_CREATE
  - IMAGE_UPDATE
  - IMAGE_DELETE

INTERNAL (1 << 6)
  - BLACKLIST_UPDATE
  - RESTART
```

- `SHARDS` is missing `CREATE` and `DELETE`, new shards will simple go `ONLINE`
- `USERS` is missing `CREATE` by design, as users always exist through REST
- `GUILD_MEMBERS` is missing `CREATE` and `DELETE` by design, for the same reason as users
- `INTERNAL` is private

### Websocket Rooms

> info
> Websocket Rooms are only relevant to internal functionality

Clients are only ever exposed to a certain selection of users, members, and guilds. Clients should only be sent events relating to users or guilds they interact with.

There should be different internal room subscriptions for a client receiving updates for every instance rather than individual instances.

As usual, Kettu has access to everything.

| intent group    | wildcard room     | specific room       | `user` client | `bot` client   | `kettu` client |
| --------------- | ----------------- | ------------------- | ------------- | -------------- | -------------- |
| `SHARDS`        | `SHARDS`          | -                   | ✓             | ✓              | ✓              |
| `USERS`         | `USERS:*`         | `USERS:id`          | ✓ *           | whitelist only | ✓              |
| `GUILDS`        | `GUILDS:*`        | `GUILDS:id`         | ✓ *           | whitelist only | ✓              |
| `GUILD_CASES`   | `GUILD_CASES:*`   | `GUILD_CASES:gid`   | ✓ *           | whitelist only | ✓              |
| `GUILD_MEMBERS` | `GUILD_MEMBERS:*` | `GUILD_MEMBERS:gid` | ✓ *           | whitelist only | ✓              |
| `IMAGES`        | `IMAGES`          | -                   | ✓             | whitelist only | ✓              |

\* `user` clients may only access updates for their own user and guilds they have `MANAGE_SERVER` permissions in

When using `wss.broadcast`, sending an event to clients in a specific room will also send that event to clients in the wildcard room. Events should not be sent directly with the wildcard room name.

## Gateway Commands

Commands are payloads sent to the Gateway in order to perform specific actions or request data.

| name      | description                                          |
| --------- | ---------------------------------------------------- |
| heartbeat | used to maintain connections and detect broken links |
| identify  | used to identify the user of the gateway connection  |
| resume    | used to resume a disconnected session                |

## Gateway Events

Events are payloads send from the API in order to provide certain data or respond to an action.

| name            | description                                                                                     |
| --------------- | ----------------------------------------------------------------------------------------------- |
| hello           | used to initiate the connection and heartbeating                                                |
| heartbeat       | used to acknowledge or request a heartbeat from the client                                      |
| invalid session | invalid session, client should disconnect and reconnect with a fresh session                    |
| reconnect       | indicates that the client should disconnect and try to resume after a short delay (few seconds) |

## Gateway Dispatches

Dispatches are specific types of events that are continuously streamed to the client based on user and automated actions.

Sometimes, dispatches may require or request a response from the client.

| name            | `t`             | description                                                                   |
| --------------- | --------------- | ----------------------------------------------------------------------------- |
| ready           | READY           | used to indicate that the client is ready and will start receiving dispatches |
| resumed         | RESUMED         | same as ready, but for resumed connections                                    |
| user update     | USER_UPDATE     | a user was updated                                                            |
| guild ping      | GUILD_PING      | a guild ping                                                                  |
| guild update    | GUILD_UPDATE    | a guild was updated                                                           |
| image create    | IMAGE_CREATE    | an image was created                                                          |
| image update    | IMAGE_UPDATE    | an image was updated                                                          |
| image delete    | IMAGE_DELETE    | an image was deleted                                                          |
| retrieve guilds | RETRIEVE_GUILDS | the server is requesting information on guilds from the client                |

## All Commands, Events and Dispatches

### Heartbeat (command)

Sent every `heartbeat_interval` ms, from the opcode-3 hello message. A very basic `{ op: 1 }`

### Identify

Sent by the client in order to identify a connection.

| field      | type    | description                                   |
| ---------- | ------- | --------------------------------------------- |
| token      | string  | user token                                    |
| intents    | integer | gateway intents                               |
| properties | object  | connection properties (idc about this really) |

If identification is successful, the gateway will respond with a `READY` dispatch event.

### Resume

Sent by the client in order to resume a connection.

| field      | type    | description                                |
| ---------- | ------- | ------------------------------------------ |
| token      | string  | user token                                 |
| session_id | string  | id of session to be resumed                |
| seq        | integer | last recorded `seq` (for replaying events) |

If resuming is successful, the gateway will start to progressively replay missed dispatch events. Completion will be marked by a `RESUMED` dispatch event, and all dispatches after will be in real time.

### Hello

Received by the client upon connection. The inner payload contains a `heartbeat_interval` value, instructing the client how often to send heartbeats (in ms).

### Heartbeat (event)

Received by the client to request or acknowledge a heartbeat.

If the payload ACK key (`a`) is set to true, the event is an acknowledgement of a client heartbeat, and no action is required.

Otherwise, the client should send an opcode-1 heartbeat to the server, independent of the standard heartbeat interval.

### Reconnect

Received by the client when the server needs to close the connection for whatever reason.

The client should disconnect and wait a short time (a couple of seconds, maybe randomise it) before attempting to resume the previous connection.

### Invalid Session

Received by the client to indicate a failure in identifying or resuming, or that the session became somehow invalidated.

The client should disconnect and wait a short time before attemping to reconnect and start a fresh session.

### Retrieve Guilds

Received by the client when the server wishes to retrieve specific data from guilds.

A retrieve guilds body will contain an array of Guild Data Query objects, specifying the data to be returned.

If all data is available in the cache, the client should respond directly with an opcode-10 retrieve guilds response, with the requested data substituted with the corresponding objects. Guilds that are irrelevant to the shard can be safely dropped or nullified.

If all data is not available, the client should response with an opcode-10 retrieve guilds ACK (the `a` key set to `true`), to indicate the data will be returned soon. Once the data is fetched, it should be sent as it would be if already cached.

If certain data cannot be resolved or fetched, it should be returned in a basic object like so: `{ id: '1234', exists: false }`

#### Guild Data Query Structure

These are the object values for the `d` payload structure in a Retrieve Guilds event, where the keys are guild ids.

| field    | type                          | description                               |
| -------- | ----------------------------- | ----------------------------------------- |
| id       | snowflake                     | the guild id                              |
| meta     | boolean                       | whether guild metadata should be returned |
| members  | array of snowflakes           | requested members                         |
| roles    | array of snowflakes or `true` | requested roles, or just all roles        |
| channels | array of snowflakes or `true` | requested channels, or just all channels  |