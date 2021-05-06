# Users

A 'user' in Kettu's context is an extension of a Discord user. Every user has an id, which directly correlates to a Discord user's id.

The term 'user' encompasses three main types: `user`, `bot`, and `kettu`. This is, again, very similar to how Discord's users work (with the exception of kettu). We'll be referencing these different types frequently throughout this page.

It should be noted that even though bot users may not have any need at all for a corresponding Discord application to be linked, it is still a requirement, simply for the sake of consistency.

## User Object

- `public` fields are visible by anyone (assuming `settings.public` is set to true when `type` is `user`)
- `private` fields are only accessible by the user themselves, and anyone with `MANAGE_USERS` permissions
- `self` fields are only accessible by the user themselves
- `internal` fields are only stored in the database, and are linked through other points

> warn
> The above specifications apply to direct API access. For `public` and `private` fields, the website and bot may provide different visibility levels.

### User Structure

| field        | type      | user types | visibility | description                              |
| ------------ | --------- | ---------- | ---------- | ---------------------------------------- |
| id           | snowflake | all        | public     | the user's discord id                    |
| type         | string    | all        | public     | type of user, `normal`, `bot` or `kettu` |
| perms        | integer   | all        | private    | the permissions on a user's account      |
| allowed_ips? | array     | all        | self       | allowed IPs for API access               |

#### User Structure 'user' Extra Fields

| field     | type    | user types | visibility | description                                         |
| --------- | ------- | ---------- | ---------- | --------------------------------------------------- |
| flags     | integer | user       | public     | the flags on a normal user's account (kettu badges) |
| profile   | object  | user       | public     | a user's kettu profile                              |
| settings  | object  | user       | private    | a user's kettu settings                             |
| votes     | integer | user       | public     | the number of votes for this user                   |
| cache?    | object  | user       | internal   | discord user cache                                  |
| guilds?   | object  | user       | internal   | discord user.guilds cache                           |
| auth?     | object  | user       | internal   | discord user oauth2 access token cache              |
| internal? | object  | user       | internal   | cache for various other things                      |

#### User Structure 'bot' Extra Fields

| field           | type   | user types | visibility | description                             |
| --------------- | ------ | ---------- | ---------- | --------------------------------------- |
| allowed_guilds? | array  | bot        | private    | guilds this bot can access by snowflake |
| name?           | string | bot        | public     | display name for the bot                |

#### User Structure 'kettu' Extra Fields

| field          | type   | user types | visibility | description                               |
| -------------- | ------ | ---------- | ---------- | ----------------------------------------- |
| default_prefix | string | kettu      | public     | default prefix for kettu                  |
| options        | object | kettu      | private    | discord.js clientOptions for the Client   |
| secrets        | object | kettu      | self       | kettu secrets                             |
| token          | string | kettu      | self       | discord token for corresponding kettu bot |

> warn
> For Kettu `private` fields, `MANAGE_KETTU` is required instead of `MANAGE_USERS`

### Example User (normal user)

```json
{
  "id": "281665697593950209",
  "type": "user",
  "perms": 1,
  "flags": 85,
  "profile": {
    "bio": "hi!",
    "color": "#00ffdd",
    "pronouns": 1,
    "timezone": "Australia/Sydney"
  },
  "settings": {
    "social": 4096,
    "voteRM": false,
    "public": true
  },
  "votes": 12
}
```

### Example User (bot user)

```json
{
  "id": "691888529684299806",
  "type": "bot",
  "perms": 1792,
  "allowed_ips": [
    "*.*.*.*"
  ],
  "allowed_guilds": [
    "691889235518816276"
  ],
  "name": "Fiber"
}
```

### User Permissions

| value   | internal name        | description                                                        | base | AD  | mfa |
| ------- | -------------------- | ------------------------------------------------------------------ | ---- | --- | --- |
| 1 << 0  | OWNER                | you are literally a god                                            |      | ✓   | ✓   |
| 1 << 1  | MANAGE_KETTU         | manage kettu, essential `MANAGE_USERS` but for kettu               |      | ✓   | ✓   |
| 1 << 2  | KETTU                | you are kettu. what?                                               |      |     | ✓   |
| 1 << 3  | ADMIN                | you are almost a god                                               |      | ✓   | ✓   |
| 1 << 4  | MANAGE_USERS         | manage other users besides self, overrides private user visibility |      | ✓   | ✓   |
| 1 << 5  | MANAGE_IMAGES        | manage all images                                                  |      | ✓   | ✓   |
| 1 << 6  | MANAGE_GUILDS_GLOBAL | manage all guilds                                                  |      | ✓   | ✓   |
| 1 << 7  | READ_GUILDS_GLOBAL   | read all guilds                                                    |      | ✓   | ✓   |
| 1 << 8  | READ_IMAGES          | read all images                                                    |      | ✓   |     |
| 1 << 9  | READ_METRICS         | view all the metric data                                           |      | ✓   |     |
| 1 << 10 | BE_IMPRESSED         | view all the fancy stuff                                           |      | ✓   |     |
| 1 << 11 | MANAGE_GUILDS        | manage guilds (mod roles allowing)                                 | ✓    |     |     |
| 1 << 12 | READ_GUILDS          | read guilds (mod roles allowing)                                   | ✓    |     |     |
| 1 << 13 | READ_USERS           | read all (public) users                                            | ✓    |     |     |

- `base` permissions are granted to all 'user' users, but can be removed from suspicious users or whatever
- `AD` permissions grant access to (certain areas of, depending on which perms) the admin dashboard
- `mfa` permissions require the user to use ip-based mfa

> warn
> By default, no permissions are granted to 'bot' users.

#### User Permission Implications

Some permissions automatically trickle down. For example, the `OWNER` permission means that user also has a number of other permissions by default, etc.

The order of this list matters - implications can cause further implications, given that they are further down on the list.

| name                 | implications                                                                   |
| -------------------- | ------------------------------------------------------------------------------ |
| OWNER                | ADMIN, MANAGE_KETTU                                                            |
| KETTU                | ADMIN                                                                          |
| ADMIN                | MANAGE_USERS, MANAGE_IMAGES, MANAGE_GUILDS_GLOBAL, READ_METRICS, MANAGE_GUILDS |
| MANAGE_GUILDS_GLOBAL | READ_GUILDS_GLOBAL                                                             |
| MANAGE_USERS         | READ_USERS                                                                     |
| MANAGE_IMAGES        | READ_IMAGES                                                                    |
| MANAGE_GUILDS        | READ_GUILDS                                                                    |

### User Flags

| value   | internal name  | description                           |
| ------- | -------------- | ------------------------------------- |
| 1 << 0  | OWNER          | Kettu owner                           |
| 1 << 1  | KETTU_STAFF    | Staff in Kettu's Lair                 |
| 1 << 2  | LEAD_DEVELOPER | Lead developers                       |
| 1 << 3  | DEVELOPER      | Developers                            |
| 1 << 4  | DONOR_SUPER    | 'Super' donator                       |
| 1 << 5  | DONOR          | Donator                               |
| 1 << 6  | BOOSTER        | Booster of Kettu's Lair               |
| 1 << 7  | CONTRIBUTOR    | A contributor to Kettu's development  |
| 1 << 8  | BUGHUNTER      | Someone who has squished some buggies |
| 1 << 9  | ARTHUNTER      | Someone who has found art for Kettu   |
| 1 << 10 | VOTER          | Anyone with more than 200 votes       |

### User Profile Structure

| field     | type       | description                                        |
| --------- | ---------- | -------------------------------------------------- |
| bio?      | string     | user's biography                                   |
| color?    | hex string | user's profile color (embed border, etc)           |
| pronouns  | integer    | user's main preferred pronouns for social commands |
| timezone? | string     | IANA time zone string                              |

#### User Pronouns

| value | description     |
| ----- | --------------- |
| 0     | they/them/their |
| 1     | she/her/hers    |
| 2     | he/him/his      |

please don't attack me about the order im not sexist ;-;

### User Settings Structure

| field          | type    | description                                       |
| -------------- | ------- | ------------------------------------------------- |
| socialDisabled | boolean | whether the user has disabled all social commands |
| socialPrefs?   | integer | user's social command preferences                 |
| animalDisabled | boolean | whether the user has disabled allanimal DMs       |
| animalPrefs?   | integer | user's animal command preferences                 |
| voteRM?        | boolean | vote reminders toggle                             |
| public?        | boolean | whether the user's profile is publically visible  |

#### User Social Command Preference Flags

These flags designate social commands the used **does not** want to allow. This also prevents them from using the command.

| value   | internal name |
| ------- | ------------- |
| 1 << 0  | BAP           |
| 1 << 1  | BELLYRUB      |
| 1 << 2  | BOOP          |
| 1 << 3  | COOKIE        |
| 1 << 4  | HUG           |
| 1 << 5  | KISS          |
| 1 << 6  | LICK          |
| 1 << 7  | NOM           |
| 1 << 8  | NUZZLE        |
| 1 << 9  | PAT           |
| 1 << 10 | POUNCE        |
| 1 << 11 | SNUGGLE       |
| 1 << 12 | ZAP           |

#### User Animal Command Preference Flags

These flags designate animal commands the used **does not** want to allow. This also prevents them from using the command.

| value   | internal name |
| ------- | ------------- |
| 1 << 0  | BIRD          |
| 1 << 1  | CAT           |
| 1 << 2  | DOG           |
| 1 << 3  | DUCK          |
| 1 << 4  | FOX           |
| 1 << 5  | KOALA         |
| 1 << 6  | OTTER         |
| 1 << 7  | OWL           |
| 1 << 8  | PANDA         |
| 1 << 9  | RABBIT        |
| 1 << 10 | REDPANDA      |
| 1 << 11 | SNAKE         |
| 1 << 12 | TURTLE        |
| 1 << 13 | WOLF          |

## Get Current User % GET /users/@me

Returns a [user](#docs:resources:user/user-structure) object of the requester's account.

Returns all values aside from internal properties.

## Modify Current User % PATCH /users/@me

Modify the current user's settings. Returns the new [user](#docs:resources:user/user-structure) object on success.

**For users,** this endpoint allows modification of `profile`, `settings`, and `allowed_ips`.

**For bots,** this endpoint allows modification of `name` and `allowed_ips`.

**For kettu,** this endpoint does not allow modification of any properties.

## Create MFA Direct Message % POST /users/@me/mfa

Sends a MFA code to the current user for authorizing the current IP. Must be sent from a non-verified IP.

Returns 204 No Content on success.

## Verify MFA Code % POST /users/@me/mfa/verify

Verifies an MFA code to verify an IP. Requires the following:

- The query parameter `code` should be the code to verify (most recent received)
- The requesting IP should be the same as was used for the `/users/@me/mfa` request

Returns 204 No Content on success.

## Get User % GET /users/{user.id#docs:resources:user/user-structure}

Returns the corresponding [user](#docs:resources:user/user-structure) object.

With `READ_USERS` permissions, and assuming the user's `settings.public` setting is true, all `public` properties will be returned.

With `MANAGE_USERS` permissions, all `public` and `private` properties will be returned.

## Modify User % PATCH /users/{user.id#docs:resources:user/user-structure} % REQUIRES `MANAGE_USERS`

Modify the corresponding user's settings. Returns the new [user](#docs:resources:user/user-structure) object on success.

### Modify User Permissions

| user type | `MANAGE_USERS`                 | `MANAGE_KETTU`                         | `OWNER`                   |
| --------- | ------------------------------ | -------------------------------------- | ------------------------- |
| user      | `flags`, `profile`, `settings` |                                        | `perms`                   |
| bot       | `allowed_ips`, `name`          |                                        | `perms`, `allowed_guilds` |
| kettu     |                                | `default_prefix`, `options`, `secrets` | `perms`, `token`          |
