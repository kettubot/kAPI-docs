# Opcodes and Status Codes

This is probably the most unoriginal page in this entire set of documentation. In fact, this is probably the most original line in this entire file.

## Gateway

### Opcodes

| code | name      | action  | meaning                                        |
| ---- | --------- | ------- | ---------------------------------------------- |
| 0    | dispatch  | both    | all typical gateway events                     |
| 1    | heartbeat | both    | regular heartbeat to detect broken connections |
| 3    | hello     | receive | first received message after connecting        |
| 4    | identify  | send    | used to identify the connection                |
| 5    | resume    | send    | used to resume a previous connection           |

### Close Codes

When writing stuff like this, I've always been unsure whether to re-order the codes myself or just keep them the exact same and drop the ones that aren't necessary. For a short list like this, I kept them the same, but for longer ones like the JSON codes I redid them.

| code | description           | action  | meaning                                                |
| ---- | --------------------- | ------- | ------------------------------------------------------ |
| 4000 | unknown error         | resume  | unknown error, try reconnecting                        |
| 4001 | unknown opcode        | resume  | no idea what that is!                                  |
| 4002 | decode error          | resume  | plz speak stringified JSON                             |
| 4003 | not authenticated     | abort   | need to identify first                                 |
| 4004 | authentication failed | abort   | the token is invalid, probably                         |
| 4005 | already authenticated | resume  | no need to identify more than once                     |
| 4007 | invalid `seq`         | restart | you sent an invalid `seq` when resuming                |
| 4008 | ratelimited           | resume  | plz slow down kthx                                     |
| 4009 | session timed out     | restart | reconnect and start a new session                      |
| 4010 | server restarting     | restart | reconnect and start a new session (delay 3-6 seconds)  |
| 4012 | invalid api version   | abort   | that api version isn't supported                       |
| 4013 | invalid intents       | abort   | your intents are configured wrong                      |
| 4014 | disallowed intents    | abort   | you aren't allowed to use one or more of those intents |

Actions are only suggestions, and will depend on your implementation.

Some close codes will be preceded by opcode 6 & 7 'Invalid Session' and 'Reconnect' events respectively.

## HTTP

### HTTP Response Codes

| code | name              | cat                         | meaning                                 |
| ---- | ----------------- | --------------------------- | --------------------------------------- |
| 200  | OK                | [cat](https://http.cat/200) | request completed                       |
| 201  | CREATED           | [cat](https://http.cat/201) | something was created and worked        |
| 204  | NO CONTENT        | [cat](https://http.cat/204) | it worked and nothing was returned      |
| 304  | NOT MODIFIED      | [cat](https://http.cat/304) | no action was taken                     |
| 400  | BAD REQUEST       | [cat](https://http.cat/400) | incorrect format or something like that |
| 401  | UNAUTHORIZED      | [cat](https://http.cat/401) | you need an `Authorization` header      |
| 403  | FORBIDDEN         | [cat](https://http.cat/403) | you don't have permission               |
| 404  | NOT FOUND         | [cat](https://http.cat/404) | that does not exist. do u need a map    |
| 429  | TOO MANY REQUESTS | [cat](https://http.cat/429) | do that less please kthx                |
| 5xx  | SERVER ERROR      | [cat](https://http.cat/500) | sorry my bad                            |

## JSON

When you receive a 4xx HTTP response code, we will also send some error information.

```json
{
  "code": 50014,
  "message": "Invalid authentication token"
}
```

### JSON Error Codes

Unlike Discord, who are lazy, we actually document all our JSON error codes. Probably.

- **1xxxx**: unknown [something]
- **2xxxx**: some kind of restriction based on the context (requesting user and the specific resource)
- **3xxxx**: some kind of limit
- **4xxxx**: authorization or request issue
- **5xxxx**: invalid request configuration of some kind, similar to 2xxxx but more generic

| code  | meaning                               |
| ----- | ------------------------------------- |
| 0     | general error                         |
| 10001 | unknown user                          |
| 10002 | unknown guild                         |
| 10003 | unknown member                        |
| 10004 | unknown image                         |
| 10005 | unknown image category                |
| 10006 | unknown service                       |
| 10007 | unknown service event                 |
| 10008 | unknown service history               |
| 10009 | unknown interruption                  |
| 10010 | unknown interruption update           |
| 10011 | unknown kettu                         |
| 10012 | unknown kettu shard                   |
| 40001 | blacklisted                           |
| 40002 | unauthorized, invalid token           |
| 40003 | bots cannot use this endpoint         |
| 40004 | invalid route ratelimit configuration |
| 40005 | invalid simulation target             |
| 40006 | missing access token                  |
| 40007 | expired access token                  |
| 50001 | missing access                        |
| 50002 | insufficient permissions              |
| 50003 | invalid query                         |
| 50004 | invalid body                          |
| 50005 | invalid content type                  |
| 50006 | ip already verified                   |
| 50007 | invalid mfa code                      |
| 50008 | invalid ip for the mfa code           |

### 5xx Responses

Any 5xx errors encountered in the API are logged internally. In this case, we'll send a basic response.

The `report` value signfifies the unique error ID. You can probably ignore it, or store it, meh.

```json
{
  "message": "Internal Server Error",
  "report": "1-2-4732n-xbsjdwa"
}
```