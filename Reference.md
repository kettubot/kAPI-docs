# Reference

kAPI, or the Kettu API, is much like Discord's. We have a restful API for doing stuff, and a WebSocket Gateway for receiving stuff. Sometimes we use WebSocket for doing stuff as well.

The API operates from the following base URL:

```
https://api.kettu.cc/v{version}
```

> warn
> This documentation is primarily for kAPI developers. Many private routes are therefore documented, even though a typical user would have no access to them. Is this kind of a security risk? Probably. But they're protected, and we keep an eye on things.

## Versioning

kAPI's gateway & api are both versioned inline with the bot itself. As kAPI is being introduced for Kettu v4, v4 is the earliest version of both.

| version | status            |
| ------- | ----------------- |
| 4       | under development |

For the API, the version can be appended after the base url. For the Gateway, the version is appended as a query parameter in the WebSocket URL. See the Gateway topic for more information.

> warn
> An API version is required

## Clients

There are three main types of clients that can interact with kAPI

| type  | availability | permissions                                                   | token prefix   |
| ----- | ------------ | ------------------------------------------------------------- | -------------- |
| user  | public       | [see base permissions](#docs:resources:user/user-permissions) | `Bearer x.y.z` |
| bot   | private      | variable (default none)                                       | `Bot x.y.z`    |
| kettu | private      | almost all (`KETTU`)                                          | `Bot x.y.z`    |

'User' clients are publically accessible by a user's authentication token on Kettu's main website (stored in the `sess` cookie). These provide access to the user's personal settings, and also the endpoints for any guild they have `MANAGE_SERVER` permissions in (any server on their Kettu dashboard). Any actions performed by a user token are marked as such. A Bot client is required to execute actions on behalf of other users.

'Bot' clients are only generally implemented for system consistency and testing. They are also used for bots that integrate with Kettu's API but are not Kettu, such as Sir Foxy. Access to these is not publically available, for the sake of our privacy policy.

## Authorization

For authenticating with the REST API, the Authorization header is used. The format should be something like this:

```txt
--- Users
Authorization: Bearer x.y.z

--- Bots (and Kettu)
Authorization: Bot x.y.z
```

The gateway has it's own process for authenticating, which can be found in the Gateway section.

If no Authorization header is provided, the `sess` cookie is also checked for a token. This is used for the web dashboard, and also allows users to access any GET-based endpoints through their browser.

## Simulation

Bots & Kettu have the ability to 'simulate' users, where the request is performed on behalf of a user. This is the key difference between bots and users, as bots can create cases in a server on behalf of a user and have that user marked as the author.

Simulation is done by setting the `X-Simulating` header to a user id.

> info
> Simulation is not yet available for bots.

> warn
> Simulation does not allow access to private endpoints, such as /users/@me

## Resource Fields

Yeah look I'm just gonna copy this table from the Discord docs.

| field                        | type    |
| ---------------------------- | ------- |
| optional_field?              | string  |
| nullable_field               | ?string |
| optional_and_nullable_field? | ?string |

## Other Notes

- Encryption should always be TLS 1.2, https/wss
- Where we use snowflakes, they will *always* be strings
- We store all times as millisecond values since the JS epoch (1st Jan 1970 00:00:00), with the exception of those embedded in snowflakes of course
- No one cares about user agents
- Everyone gets rate limits ~~except Kettu~~
- The API is not accessible through web browsers, kind of
