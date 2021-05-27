# Members

Members store kettu information relating to a specific user in a specific guild.

## Member Object

### Member Structure

| field         | type      | description                             |
| ------------- | --------- | --------------------------------------- |
| id            | snowflake | member id                               |
| nickname      | string    | last cached nickname                    |
| strikes       | integer   | total number of strikes for this member |
| premium_since | integer   | last cached premium_since value         |

- `nickname` and `premium_since` are regularly updated cached values
- `strikes` is calculated by adding the strikes for every case the member has been involved in

## Get Guild Member % GET /guilds/{guild.id#docs:resources:guild/guild-structure:id}/members/{member.id#docs:resources:member/member-structure:id} % REQUIRES `MANAGE_GUILDS`

Returns the corresponding [member](#docs:resources:member/member-structure) object.

If the member doesn't exist, the default object will be returned with a `default` key set to `true`.

## Modify Guild Member % PATCH /guilds/{guild.id#docs:resources:guild/guild-structure:id}/members/{member.id#docs:resources:member/member-structure:id} % REQUIRES `KETTU`

Modifies and returns the new [member](#docs:resources:member/member-structure) object.

Depending on if and how the `premium_since` value has changed, a `GUILD_MEMBER_BOOST` event may be broadcasted.

### Sync Guild Member JSON Body

| field         | type    | description                        |
| ------------- | ------- | ---------------------------------- |
| nickname      | string  | current cached nickname            |
| premium_since | integer | current cached premium_since value |