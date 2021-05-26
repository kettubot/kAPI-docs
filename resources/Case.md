# Cases

The easiest way to understand the case system is to think of a fairly typical punishment system, and then add a bunch of stuff on top.

Kettu provides the functionality to log any mod action like a punishment. So as well as mutes and bans, you can also have server lockdowns, purges, and other changes logged just like regular punishments. Obviously punishment is no longer a very good name for this system, so we're using the simpler 'cases' instead.

Although all cases belong to a specific guild, the documentation is separate due to the complexity of the system.

## Case Object

### Case Structure

A case is fairly abstract, as depending on the type, the fields can and will vary a lot.

Cases are composed of two main types of field.

**'global fields'** apply to all cases, while **'sub fields'** apply to only some cases, depending on the type

For this reason, all sub fields are nullable on a general case structure.

| field         | type            | level  | description                                        |
| ------------- | --------------- | ------ | -------------------------------------------------- |
| id            | integer         | global | case number, incrementing from zero per-guild      |
| guild_id      | snowflake       | global | guild this case belongs to                         |
| type          | string          | global | type of case - such as mute or lock channel        |
| reason?       | string          | global | reason, if given                                   |
| log?          | object          | global | case log message information, if sent              |
| context?      | object          | global | trigger message information, if triggered manually |
| moderator_id? | snowflake       | global | responsible moderator's id, if triggered manually  |
| user_id?      | snowflake       | sub    | target user                                        |
| channel_id?   | snowflake       | sub    | target channel                                     |
| user_dm?      | boolean, string | sub    | 'true', or a reason the dm failed to send          |
| strikes?      | integer         | sub    | strikes towards the target user                    |
| time?         | integer         | sub    | duration of the case                               |
| meta?*        | object          | sub    | other metadata about the case                      |

\* `meta` provides case-specific information about the case, and will change significantly depending on the case type. Documentation for each case type will specify the exact format, if included.

#### Case Log Message or Context Message Information Structure

| field      | type      | description               |
| ---------- | --------- | ------------------------- |
| channel_id | snowflake | channel id of log message |
| message_id | snowflake | log message id            |

## Case Types

### Lock Channel, Server, Category

**Sub Fields**: `channel_id?`, `time?`

When a channel, server or category is locked or unlocked.

### Ban

**Sub Fields**: `user_id`, `user_dm`, `strikes?`(?), `time?`

When a user is banned or tempbanned from a guild. Not sure if strikes is relevant since they're, like, banned.

### Kick

**Sub Fields**: `user_id`, `user_dm`, `strikes?`

When a user is kicked from a guild.

### Mute

**Sub Fields**: `user_id`, `user_dm`, `strikes?`, `time?`

When a user is muted within a guild.

### Raidmode

**Sub Fields**: `time?`

When raidmode is changed.

#### Raidmode Meta Structure

| field | type    | description                                          |
| ----- | ------- | ---------------------------------------------------- |
| state | boolean | whether raidmode was enabled (or otherwise disabled) |

### Purge

**Sub Fields**: `channel_id`

When messages in a channel are purged.

#### Purge Meta Structure

| field    | type                | description                        |
| -------- | ------------------- | ---------------------------------- |
| options  | object              | the options for the purge          |
| purged   | integer             | the number of messages purged      |
| messages | array of snowflakes | an array of the purged message ids |

### Edit Case, Delete Case

**Sub Fields**: none

When a case is edited or deleted.

#### Edit Case and Delete Case Meta Structure

| field | type    | description        |
| ----- | ------- | ------------------ |
| case  | integer | the edited case id |

### Slowmode

**Sub Fields**: `channel_id`, `time?`

When the slowmode in a channel is changed.

#### Slowmode Meta Structure

| field    | type    | description                    |
| -------- | ------- | ------------------------------ |
| original | integer | the original slowmode duration |
| new      | integer | the new slowmode duration      |