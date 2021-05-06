# Guilds

A 'guild' in Kettu's context is an extension of a Discord guild. It primarily stores information relating to the guild's configuration and cases (or punishments).

The case system, although nested within the guild endpoints, is documented in a separate file due to its complexity.

## TODO

- Automod
- Example Guild
- Autopublish(?)

## Guild Object

### Guild Structure

| field   | type      | description                                                           |
| ------- | --------- | --------------------------------------------------------------------- |
| id      | snowflake | guild's discord id                                                    |
| configs | object    | guild configurations mapped by the bot ID                             |
| config  | object    | specific guild configuration (see below)                              |
| premium | snowflake | user id of the user who is responsible for the guild's premium status |

When a guild is requested from the API, the correct configuration is selected.

If a Kettu client is requesting, the corresponding config will be returned. Otherwise, the following criteria are used:

- The config specified by the `instance` query parameter
- The only available config
- Main Kettu's config

### Guild Configuration Structure

Guild configuration, in the global scope, only stores `prefix`. All other settings are further categorised under other objects.

| field    | type             | description                                 |
| -------- | ---------------- | ------------------------------------------- |
| prefix   | string           | guild prefix                                |
| disabled | array of strings | list of commands that are globally disabled |
| logs     | object           | logging configuration                       |
| mod      | object           | moderation configuration                    |
| roles    | object           | roles configuration (selfroles, reactroles) |
| social   | object           | social commands configuration               |

A configuration section is used when a certain area provides multiple levels of functionality across different areas.

For example, `automod` has multiple functions and could be considered a distinct category of the bot.

However, `autopurge` is an extension of the `purge` command, and is a singular feature. So rather than having its own category, it goes under the `util` category.

#### Guild Configuration Logs Structure

| field         | type      | description                                                           |
| ------------- | --------- | --------------------------------------------------------------------- |
| automod       | snowflake | log channel for automod                                               |
| mod           | snowflake | log channel for moderation actions                                    |
| join          | snowflake | join log channel                                                      |
| joinDM        | object    | join DM custom response                                               |
| joinMessage   | object    | join message custom response                                          |
| leave         | snowflake | leave log channel                                                     |
| leaveMessage  | object    | leave message custom response                                         |
| messageEdit   | snowflake | log channel for message edits                                         |
| messageDelete | snowflake | log channel for message deletes                                       |
| nickname      | snowflake | log channel for nickname changes                                      |
| role          | snowflake | log channel for roles (giddy: general roles or selfroles/reactroles?) |
| vc            | snowflake | log channel for vc actions                                            |
| type          | integer   | message style for logs                                                |

##### Custom Response

Custom response objects have the following format:

| field    | type      | description                              |
| -------- | --------- | ---------------------------------------- |
| message? | string    | message to send                          |
| embed?   | object    | embed to send                            |
| channel? | snowflake | channel to send in, for non-dm responses |

Note: one of `message` or `embed` must be present.

##### Message Style Types

Giddy still needs to plan this.

#### Guild Configuration Mod Structure

| field     | type    | description                                                    |
| --------- | ------- | -------------------------------------------------------------- |
| confirm   | string  | level at which to confirm mod actions: `none`, `mass` or `all` |
| type      | integer | message style for mod actions                                  |
| deleteCmd | boolean | whether to delete mod commands                                 |
| deleteErr | boolean | whether to (auto?) delete mod command errors                   |
| dm        | object  | mod dm configuration                                           |
| role      | object  | moderation role IDs                                            |
| action    | object  | which actions are counted as cases                             |

##### Guild Configuration Mod DM Structure

| field   | type    | description                          |
| ------- | ------- | ------------------------------------ |
| dm      | boolean | whether DMs are enabled              |
| showMod | boolean | whether the responsible mod is shown |
| appeals | string  | link to appeals page                 |

##### Guild Configuration Mod Roles Structure

The mod roles structure is an object of role types mapped to their corresponding IDs.

| role type | purpose                             |
| --------- | ----------------------------------- |
| mute      | role assigned to muted users        |
| trusted   | users with a lower automod strength |
| bypass    | users with no automod               |
| helper    | role for helpers                    |
| mod       | role for mods, step up from helpers |
| admin     | role for admins, step up from mods  |

##### Guild Configuration Mod Actions Structure

Determines whether certain actions are counted as new cases or not. Action types are mapped to boolean values. Default-on types are indicated below.

Action types: `lockchannel`, `lockcategory`, `lockserver` (on), `raidmode` (on), `purge`, `deletecase` (on), `editcase`

#### Guild Configuration Roles Structure

| field      | type  | description                                                  |
| ---------- | ----- | ------------------------------------------------------------ |
| selfroles  | array | list of snowflakes of role IDs that can be used as selfroles |
| reactroles | array | list of reactrole objects                                    |

##### Guild Configuration Reaction Role Structure

| field           | type       | description                          |
| --------------- | ---------- | ------------------------------------ |
| channelID       | snowflake  | channel id                           |
| messageID       | snowflake  | message id                           |
| roleID          | snowflake  | role id                              |
| emoji.id        | ?snowflake | emoji id for reacting                |
| emoji.name      | ?string    | ASCII emoji, or custom emoji name    |
| emoji.animated? | boolean    | whether the emoji is animated or not |

For more information on the `emoji` object, check out the [Discord docs](https://discord.com/developers/docs/resources/emoji#emoji-object-gateway-reaction-standard-emoji-example).

#### Guild Configuration Social Structure

| field     | type             | description                                         |
| --------- | ---------------- | --------------------------------------------------- |
| sDelete   | boolean          | whether social command trigger messages are deleted |
| blacklist | array of objects | list of blacklisted social images                   |

##### Guild Configuration Social Blacklist Structure

| field    | type    | description    |
| -------- | ------- | -------------- |
| id       | integer | image id       |
| category | string  | image category |

## Get Guild % GET /guilds/{guild.id#docs:resources:guild/guild-structure} % REQUIRES `READ_GUILDS`

Returns the corresponding [guild](#docs:resources:guild/guild-structure) object.

Requires `READ_GUILDS` with `MANAGE_SERVER` permissions (or `allowed_guilds` for `bot`s) or `READ_GUILDS_GLOBAL`.

## Ping Guild % POST /guilds/{guild.id#docs:resources:guild/guild-structure}/ping

Pings a specific guild, sending out a `GUILD_PING` websocket event to all connected clients in the guild.