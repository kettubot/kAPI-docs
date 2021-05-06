# Cases

The easiest way to understand the case system is to think of a fairly typical punishment system, and then add a bunch of stuff on top.

Kettu provides the functionality to log any mod action like a punishment. So as well as mutes and bans, you can also have server lockdowns, purges, and punishment modifications logged just like other punishments. Obviously punishment is no longer a very good name for this system, so we're using the simpler 'cases' instead.

A case is fairly abstract, as depending on the context, the fields can and will vary a lot.

## Case Object

### Case Structure

Cases are composed of three types of field.

**Global Fields**: Apply to all cases.
**Sub Fields**: Apply to some cases, depending on the type. You can think of these as 'if applicable'

For this reason, all sub-global and local fields are nullable on a general case structure.

| field         | type            | level  | description                                        |
| ------------- | --------------- | ------ | -------------------------------------------------- |
| id            | integer         | global | case number, incrementing from zero per-guild      |
| guild_id      | snowflake       | global | guild this case belongs to                         |
| type          | string          | global | type of case - such as mute or lock channel        |
| log?          | object          | global | case log message information, if sent              |
| context?      | object          | global | trigger message information, if triggered manually |
| moderator_id? | snowflake       | global | responsible moderator's id, if triggered manually  |
| user_id?      | snowflake       | sub    | target user                                        |
| channel_id?   | snowflake       | sub    | target channel                                     |
| user_dm?      | boolean, string | sub    | 'true', or a reason the dm failed to send          |
| strikes?      | integer         | sub    | strikes towards the target user                    |
| time?         | integer         | sub    | duration of the case                               |

#### Case Log Message or Context Message Information Structure

| field      | type      | description               |
| ---------- | --------- | ------------------------- |
| channel_id | snowflake | channel id of log message |
| message_id | snowflake | log message id            |

## Case Types

### Lock Channel, Server, Category

**Sub-globals:** `channel_id`, `time`

When a channel, server or category is locked or unlocked.

### Ban

**Sub Fields**: `user_id`, `user_dm`, `strikes`(?), `time`

When a user is banned or tempbanned from a guild. Not sure if strikes is relevant since they're, like, banned.

### Kick

**Sub Fields**: `user_id`, `user_dm`, `strikes`

When a user is kicked from a guild.

### Mute

**Sub Fields**: `user_id`, `user_dm`, `strikes`, `time`

When a user is muted within a guild.

