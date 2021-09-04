# Actions

In kAPI, some operations take time to complete. These operations may need to be kept track of, whether to prevent them happening again while still ongoing or for tracking purposes. In this case, these operations are called **actions**. The most simple example of an action would be resharding.

> info
> Actions are only relevant to admins

## Action Object

### Action Structure

| field       | type    | description                         |
| ----------- | ------- | ----------------------------------- |
| id          | string  | randomly generated id               |
| title       | string  | action title                        |
| created_at  | integer | when this action was first created  |
| status      | integer | action status                       |
| description | string  | general description                 |
| progress?   | float   | action progress                     |
| state?      | string  | current state description (changes) |

### Action Statuses

| value | meaning                             |
| ----- | ----------------------------------- |
| 0     | pending (yet to start)              |
| 1     | running                             |
| 2     | impacted (issues but still running) |
| 3     | failed (no longer running)          |
| 4     | aborted (failed -> completed)       |
| 5     | completed                           |

When an action has completed, it will be updated with the status `4` or `5`. At this point it will become ephemeral, meaning it is disposed on kAPI's end. It will receive no more updates, and will no longer be returned on the list actions endpoint.

Actions with the status of `3` remain inactive, until manually dismissed, at which point the status changes to `4`.

## List Actions % GET /kapi/actions

Lists all currently ongoing [actions](#docs:resources:action/action-structure).

## Dismiss Failed Action % POST /kapi/actions/{action.id#docs:resources:action/action-structure}/dismiss

Dismisses an action of type `3`. Returns a 204 `No Content` on success.