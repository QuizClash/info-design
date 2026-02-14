# Authorization Rules (proposal)

## Actors

| Actor | Authentication | Description                                                                                                |
|:------|:---------------|:-----------------------------------------------------------------------------------------------------------|
| **Moderator** | Telegram → backbone JWT | Authenticated user with a `ModeratorProfile`. Accesses backbone via EvaliquizModeratorBot mini-app.        |
| **Companion** | JWT signed with `Companion.secret` | Software on phone/host that relays buzzer events.                                                          |
| **Team member** | None (anonymous) | No direct backbone authentication. Interacts only through EvaliquizTeamBot.                                |
| **EvaliquizTeamBot** | Telegram → backbone JWT | Service account for read access to game state (rounds, attempts, teams) and to send attempt without buzzer |

## Scope rule

Moderators can only access entities that belong to them (own games, own quizzes, own companions).
There is no cross-moderator visibility.

## Entity authorization matrix

Legend: **O** = own only, **--** = no access, **?** = to be decided

### ModeratorProfile

| Operation | Moderator | Companion | Notes |
|:----------|:----------|:----------|:------|
| Create    |           | --        |       |
| Read      | O         | --        |       |
| Update    | O         | --        |       |
| Delete    |           |           |       |

### Quiz

| Operation | Moderator | Companion | Notes |
|:----------|:----------|:----------|:------|
| Create    | O         | --        |       |
| Read      | O         | --        |       |
| Update    | O         | --        |       |
| Delete    | O         | --        |       |

### Question

| Operation | Moderator | Companion | Notes |
|:----------|:----------|:----------|:------|
| Create    | O         | --        | Quiz must belong to moderator |
| Read      | O         | --        | Hint visible only to moderator |
| Update    | O         | --        |       |
| Delete    | O         | --        |       |

### Game

| Operation | Moderator | Companion | Notes |
|:----------|:----------|:----------|:------|
| Create    | O         | --        |       |
| Read      | O         | --        |       |
| Update    | O         | --        |       |
| Delete    |           | --        |       |

### Team

| Operation | Moderator | Companion | Notes |
|:----------|:----------|:----------|:------|
| Create    | O         | --        | Game must belong to moderator |
| Read      | O         | --        |       |
| Update    | O         | --        |       |
| Delete    | O         | --        |       |

### TeamMember

| Operation | Moderator | Companion | Notes |
|:----------|:----------|:----------|:------|
| Create    | O         | --        | Team must belong to moderator's game |
| Read      | O         | --        |       |
| Update    | O         | --        |       |
| Delete    | O         | --        |       |

### Round

| Operation | Moderator | Companion | Notes |
|:----------|:----------|:----------|:------|
| Create    | O         | --        | Game must belong to moderator |
| Read      | O         | --        |       |
| Update    | O         | --        | Status transitions only |
| Delete    |           | --        |       |

### Attempt

| Operation | Moderator | Companion | Notes |
|:----------|:----------|:----------|:------|
| Create    | --        | 0         | Created when captain buzzes in |
| Read      | O         | --        |       |
| Update    | O         | --        | Only `chosen` and `correct` fields |
| Delete    | O         | --        |       |

### Companion

| Operation | Moderator | Companion | Notes |
|:----------|:----------|:----------|:------|
| Create    | O         | --        |       |
| Read      | O         | --        |       |
| Update    | O         | --        |       |
| Delete    | O         | --        |       |

## Data consistency

- A moderator cannot delete a Game that is IN_PROGRESS (only CREATED/FINISHED)
- A moderator cannot delete a Quiz that has been used in a Game
- A moderator can update a Quiz/Question while a Game referencing it is IN_PROGRESS
