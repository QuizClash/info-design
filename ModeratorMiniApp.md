# Moderator Mini-App — Screen Catalog

Every screen of the EvaliquizModeratorBot mini-app, in enough detail to implement it.

Companion documents:
- [Game.md](Game.md) — game stages and state machines
- [Authorization.md](Authorization.md) — moderator access rules
- [Authentication.md](Authentication.md) — JWT flow
- [evaliquiz-model.jdl](evaliquiz-model.jdl) — data model
- [README.md](README.md) — system overview, SSE spec

---

## Overview

### Platform

The mini-app is a Telegram Web App (TWA) launched from the EvaliquizModeratorBot **menu button**. It uses the TWA SDK for:
- Theme colors (`themeParams`)
- Back button (`BackButton`)
- Haptic feedback
- `initData` for authentication (see [Authentication.md](Authentication.md))

### Authentication on launch

1. Mini-app obtains `initData` from the TWA SDK
2. `POST /bot_api/v1/auth` with `initData` → receives JWT
3. JWT stored in localStorage, attached as `Authorization: Bearer …` to all REST calls
4. On 401 → re-authenticate with fresh `initData`
5. Token expires after 12 hours

### Navigation model

List-based root with drill-down stack. No persistent tab bar.

The Telegram **Back Button** (`BackButton.show()` / `BackButton.hide()`) pops the stack. On the Home screen the Back Button is hidden — Telegram closes the mini-app on swipe-down.

```
Home
 ├── Quiz List → Quiz Detail → Question Edit
 ├── Game List → Game Create/Edit ──┐
 │                                  ↓
 │              Game Lobby → Live Game → Game Results
 └── Companion List → Companion Edit
```

### API conventions

- REST: `GET|POST|PUT|DELETE /bot_api/v1/rest/…` (mirrors backbone entity endpoints, moderator-scoped via JWT)
- SSE: `GET /api/games/{id}/events` (moderator JWT; delivers `round-status`, `attempt`, `attempt-update`, `game-status` events)

---

## Screen Catalog

### 1. Home

**Purpose:** Navigation root with a quick status glance.

**Entry:** Mini-app launch (after JWT is obtained).

**Data shown:**
- Moderator display name (from Telegram profile)
- Navigation items:
  - **Quizzes** — own quiz count
  - **Games** — own game count; badge if an IN_PROGRESS game exists
  - **Companions** — own companion count

**Actions:**

| Action | Result |
|--------|--------|
| Tap "Quizzes" | → Quiz List |
| Tap "Games" | → Game List |
| Tap "Companions" | → Companion List |

**States:** Only the initial load / skeleton state.

**Exits:** Quiz List, Game List, Companion List.

---

### 2. Quiz List

**Purpose:** Browse, create, and delete quizzes.

**Entry:** Home → "Quizzes".

**Data shown:**
- Paginated list of own quizzes (`GET /bot_api/v1/rest/quizzes?sort=createdAt,desc`)
- Each row: `title`, truncated `description`, question count, `createdAt`
- Badge if a quiz is referenced by any Game

**Actions:**

| Action | API | Notes |
|--------|-----|-------|
| Tap quiz | — | → Quiz Detail |
| "+" create | `POST /bot_api/v1/rest/quizzes` | Default title; → Quiz Detail |
| Swipe-delete | `DELETE /bot_api/v1/rest/quizzes/{id}` | Server returns 409 if referenced by any Game (show toast) |

**States:** Empty ("No quizzes yet — create your first quiz").

**Exits:** Home (Back), Quiz Detail (tap).

---

### 3. Quiz Detail / Edit

**Purpose:** Edit quiz metadata and manage its question list.

**Entry:** Quiz List → tap quiz.

**Data shown:**
- Editable: `title` (required), `description` (optional, multiline)
- Ordered question list (`GET /bot_api/v1/rest/questions?quizId.equals={id}&sort=orderIndex,asc`)
  - Each row: `orderIndex`, `title` (or first line of `text` as fallback), hint indicator
- "Used by N game(s)" informational notice (editing is allowed even when in use — per Authorization.md)

**Actions:**

| Action | API | Notes |
|--------|-----|-------|
| Edit title / description | `PUT /bot_api/v1/rest/quizzes/{id}` | Auto-save on blur |
| Tap question | — | → Question Edit |
| "+" add question | `POST /bot_api/v1/rest/questions` | Next `orderIndex`; → Question Edit |
| Drag-reorder | `PUT /bot_api/v1/rest/questions/{id}` (batch) | Updates `orderIndex` for affected questions |
| Swipe-delete question | `DELETE /bot_api/v1/rest/questions/{id}` | Confirm |
| Delete quiz (header action) | `DELETE /bot_api/v1/rest/quizzes/{id}` | Blocked if referenced by a Game; confirm; pops to Quiz List |

**States:** Empty question list ("Add your first question").

**Exits:** Quiz List (Back), Question Edit (tap / add).

---

### 4. Question Edit

**Purpose:** Edit a single question.

**Entry:** Quiz Detail → tap question or "+" add.

**Data shown:**
- Read-only: "Question #N" (`orderIndex`)
- Editable:
  - `title` (short label, optional)
  - `text` (full question body, multiline)
  - `hint` (moderator-only note, multiline) — labeled "Visible only to you during the game"

**Actions:**

| Action | API | Notes |
|--------|-----|-------|
| Edit fields | `PUT /bot_api/v1/rest/questions/{id}` | Auto-save on blur |
| Delete | `DELETE /bot_api/v1/rest/questions/{id}` | Confirm; pops to Quiz Detail |

**States:** None (question shell exists on entry).

**Exits:** Quiz Detail (Back).

---

### 5. Game List

**Purpose:** Browse games, create new ones, and navigate to the appropriate game screen by status.

**Entry:** Home → "Games".

**Data shown:**
- Games grouped by status, each group sorted by date:
  - **IN_PROGRESS** (at most one) — pinned at top, highlighted
  - **CREATED** — upcoming
  - **FINISHED** — past (collapsed by default)
- Each row: `label` (or quiz title fallback), status badge, team count, date (`startedAt` or `finishedAt`)

**Actions:**

| Action | Result | Notes |
|--------|--------|-------|
| Tap IN_PROGRESS game | → Live Game | |
| Tap CREATED game | → Game Lobby | |
| Tap FINISHED game | → Game Results | |
| "+" create | → Game Create | |
| Swipe-delete | `DELETE /bot_api/v1/rest/games/{id}` | Only CREATED or FINISHED; confirm |

**States:** Empty ("No games yet").

**Exits:** Home (Back), Game Create, Game Lobby, Live Game, Game Results.

---

### 6. Game Create / Edit

**Purpose:** Set up a new game or edit settings of a CREATED game.

**Entry:**
- Game List → "+" (create mode)
- Game Lobby → "Edit" header action (edit mode)

**Data shown (editable fields):**
- `label` (optional title — useful for ad-hoc games without a quiz)
- `roundTimeout` (seconds; numeric stepper; suggested default: 30)
- Quiz picker: own quizzes list or "No quiz (ad-hoc questions)"
- Companion picker: own companions list or "None (mini-app buzzers only)"

**Actions:**

| Action | API | Notes |
|--------|-----|-------|
| Save (create) | `POST /bot_api/v1/rest/games` | `status=CREATED`; → Game Lobby |
| Save (edit) | `PUT /bot_api/v1/rest/games/{id}` | Pop back to Game Lobby |
| Cancel | — | Pop back without saving |

**States:** None.

**Exits:** Game Lobby (save), previous screen (cancel / Back).

---

### 7. Game Lobby

**Purpose:** Manage teams and members; share invite links; start the game. Also serves as a read-only team viewer during an IN_PROGRESS game.

**Entry:**
- Game List → tap CREATED game
- Live Game → "Teams" header action (limited mode, game is IN_PROGRESS)

**Data shown:**
- **Header:** `label`, quiz title (or "Ad-hoc"), `roundTimeout`, companion name (or "No companion"), game status
- **Team list** (`GET /bot_api/v1/rest/teams?gameId.equals={id}&sort=index,asc`):
  - Team `name`, `index`
  - Member list under each team, with captain badge
- **Invite links:**
  - Per-team: `https://t.me/EvaliquizTeamBot?start=team_{teamId}`
  - Moderator link (join any team during game): `https://t.me/EvaliquizTeamBot?start=moderator{moderatorUserId}`

#### Actions — CREATED game (full edit)

| Action | API | Notes |
|--------|-----|-------|
| Add team | `POST /bot_api/v1/rest/teams` | Auto-assigns next `index` |
| Edit team name | `PUT /bot_api/v1/rest/teams/{id}` | Inline edit |
| Delete team | `DELETE /bot_api/v1/rest/teams/{id}` | Confirm |
| Add member | `POST /bot_api/v1/rest/team-members` | Name entry; for Telegram-joined members the bot creates them automatically |
| Remove member | `DELETE /bot_api/v1/rest/team-members/{id}` | |
| Designate captain | `PUT /bot_api/v1/rest/team-members/{id}` `captain=true` | Service layer clears the previous captain on the same team |
| Copy team link | — | Clipboard |
| Copy moderator link | — | Clipboard |
| Edit game settings | — | → Game Create / Edit (edit mode) |
| **Start game** | `PUT /bot_api/v1/rest/games/{id}` `status=IN_PROGRESS` | Server sets `startedAt`; → Live Game. Blocked if another game is already IN_PROGRESS (server returns 409). |

#### Actions — IN_PROGRESS game (limited)

| Action | API | Notes |
|--------|-----|-------|
| Add member (late joiner) | `POST /bot_api/v1/rest/team-members` | Allowed per spec |
| Copy links | — | For late joiners |
| View teams / members | read-only | Cannot add/remove teams, remove members, or change captains |

**States:**
- Empty ("Add your first team")
- Not ready — some teams lack a captain (warning shown near Start button)
- Ready — all teams have a captain

**Exits:** Game List (Back from CREATED), Live Game (start), Game Create/Edit (edit settings).

---

### 8. Live Game

**Purpose:** Run the game — create and advance rounds, view attempts, judge answers. The moderator spends most of the game here.

**Entry:**
- Game Lobby → "Start game"
- Game List → tap IN_PROGRESS game (resume)

**SSE connection:** On entry subscribe to `GET /api/games/{id}/events`. Events: `round-status`, `attempt`, `attempt-update`, `game-status`. Auto-reconnect on disconnect; re-fetch current state on reconnect.

#### Layout

**Top bar:**
- Game label / quiz title
- IN_PROGRESS badge
- "Teams" action (→ Game Lobby, limited mode)
- "End game" action

**Current round panel** (the active round — last non-FINISHED round):
- Round number (`orderIndex`)
- Status badge with color coding:
  - IDLE — gray
  - READY — yellow
  - STARTED — green, pulsing
  - ANSWERED — blue
  - FINISHED — dark
- **Countdown timer** (STARTED state only): shows `roundTimeout − (now − startedAt)` seconds, ticking every second
- Question display:
  - Quiz game: `title`, `text`, `hint` from `Round.question`
  - Ad-hoc game: editable `questionText` field

**Attempt list** (visible when round is STARTED / ANSWERED / FINISHED):
- Ordered by `receivedAt` (fastest first)
- Each row: team name, `receivedAt` relative timestamp, `reaction` ms (if hardware path), `chosen` indicator, `correct` indicator
- Fastest attempt highlighted

**Round history** (collapsible, below current round):
- Previous rounds in reverse `orderIndex` order
- Each: question text, final status, attempts with judgments

#### Actions — Round lifecycle

| Action | API | Precondition |
|--------|-----|--------------|
| **Create round** | `POST /bot_api/v1/rest/rounds` with `status=IDLE`, next `orderIndex` | No active (non-FINISHED) round |
| **Ready** | `PUT /bot_api/v1/rest/rounds/{id}` `status=READY` | Round is IDLE. Quiz game: question must be selected. Ad-hoc: `questionText` must be filled. |
| **Start** | `PUT /bot_api/v1/rest/rounds/{id}` `status=STARTED` | Round is READY. Server sets `startedAt`. Countdown begins. |
| **Finish** | `PUT /bot_api/v1/rest/rounds/{id}` `status=FINISHED` | Any state (override). Server sets `finishedAt`. |
| **Replay (→ READY)** | `PUT /bot_api/v1/rest/rounds/{id}` `status=READY` | From STARTED, ANSWERED, or FINISHED. **Deletes existing attempts** for the round. Confirm dialog. |
| Delete round | `DELETE /bot_api/v1/rest/rounds/{id}` | Secondary action (e.g. overflow menu). Confirm. |

> The round also auto-transitions STARTED → ANSWERED when `roundTimeout` elapses **and** at least one attempt exists (server-side). If timeout elapses with no attempts, the round stays STARTED and the moderator must act manually.

#### Actions — Question selection

| Context | Action | API |
|---------|--------|-----|
| Quiz game, IDLE/READY | Select question from quiz | `PUT /bot_api/v1/rest/rounds/{id}` with `question.id` — shows picker of unused quiz questions |
| Ad-hoc game, IDLE/READY | Enter question text | `PUT /bot_api/v1/rest/rounds/{id}` with `questionText` |

#### Actions — Attempt judging

Available when attempts exist (typically ANSWERED state, but also STARTED if attempts have arrived).

| Action | API | Notes |
|--------|-----|-------|
| Choose captain to answer | `PUT /bot_api/v1/rest/attempts/{id}` `chosen=true` | Tap on an attempt row |
| Mark correct | `PUT /bot_api/v1/rest/attempts/{id}` `correct=true` | Only on the `chosen` attempt |
| Mark incorrect | `PUT /bot_api/v1/rest/attempts/{id}` `correct=false` | Only on the `chosen` attempt |
| Clear judgment | `PUT /bot_api/v1/rest/attempts/{id}` `correct=null` | Reset |
| Delete attempt | `DELETE /bot_api/v1/rest/attempts/{id}` | Secondary action; confirm |

#### Actions — Game level

| Action | API | Notes |
|--------|-----|-------|
| End game | `PUT /bot_api/v1/rest/games/{id}` `status=FINISHED` | Confirm; server sets `finishedAt`; → Game Results |
| View teams | — | → Game Lobby (limited mode) |

**States:**
- No rounds yet — "Create first round" prompt
- Round in progress (STARTED) — countdown active, attempt list updating via SSE
- Round answered — attempts frozen, awaiting moderator judgment
- Between rounds (current round FINISHED) — "Create next round" prompt
- SSE disconnected — reconnection banner with manual retry

**Exits:** Game Results (end game), Game Lobby (teams), Game List (Back — game stays IN_PROGRESS for later resume).

---

### 9. Game Results

**Purpose:** Review a finished game's rounds and outcomes.

**Entry:**
- Live Game → "End game"
- Game List → tap FINISHED game

**Data shown:**
- **Header:** `label`, quiz title, `startedAt` → `finishedAt`, duration
- **Summary:** total rounds played, per-team correct-answer count
- **Team ranking:** ordered by correct-answer count descending
- **Round-by-round breakdown** (ordered by `orderIndex`):
  - Question (from `Round.question.title`/`text` or `Round.questionText`)
  - Attempts ordered by `receivedAt`: team name, reaction time, `chosen`/`correct` indicators

**Actions:**

| Action | API | Notes |
|--------|-----|-------|
| Delete game | `DELETE /bot_api/v1/rest/games/{id}` | Confirm; pops to Game List |

**States:** Always has data (a finished game has at least the game entity).

**Exits:** Game List (Back).

---

## Companion Management

Accessible from Home → "Companions". Simple CRUD — not a game-session screen.

### Companion List

**Data:** Own companions (`GET /bot_api/v1/rest/companions`). Each row: `name`, `uuid`, `edition`.

| Action | API |
|--------|-----|
| Tap companion | → Companion Edit |
| "+" add | `POST /bot_api/v1/rest/companions` → Companion Edit |
| Swipe-delete | `DELETE /bot_api/v1/rest/companions/{id}` |

### Companion Edit

**Editable:** `name` (required), `description`, `edition`.
**Read-only:** `uuid` (auto-generated), `secret` (shown once on create, then masked).

| Action | API |
|--------|-----|
| Save | `PUT /bot_api/v1/rest/companions/{id}` |
| Delete | `DELETE /bot_api/v1/rest/companions/{id}` |

**Exits:** Companion List (Back).

---

## Coverage Verification

### Authorization.md — every moderator capability is reachable

| Entity | Create | Read | Update | Delete | Screen(s) |
|--------|--------|------|--------|--------|-----------|
| TelegramProfile | — | Home (display name) | — (self-disable via bot `/disable`, not mini-app) | — | 1 |
| Quiz | Quiz List | Quiz List, Quiz Detail | Quiz Detail | Quiz List, Quiz Detail | 2, 3 |
| Question | Quiz Detail | Quiz Detail, Question Edit | Question Edit | Quiz Detail, Question Edit | 3, 4 |
| Game | Game Create | Game List, Lobby, Live, Results | Game Create/Edit, Live Game (status) | Game List, Game Results | 5–9 |
| Team | Game Lobby | Game Lobby | Game Lobby (name) | Game Lobby | 7 |
| TeamMember | Game Lobby | Game Lobby | Game Lobby (captain) | Game Lobby | 7 |
| Round | Live Game | Live Game, Game Results | Live Game (status transitions) | Live Game | 8, 9 |
| Attempt | — (created by bot/companion) | Live Game, Game Results | Live Game (chosen, correct) | Live Game | 8, 9 |
| Companion | Companion Edit | Companion List | Companion Edit | Companion List/Edit | Home → Companions |

### Game.md — every stage maps to a screen

| Stage | Screen |
|-------|--------|
| Preliminary preparation | Bot registration (outside mini-app) |
| Quiz preparation | Quiz List (2), Quiz Detail (3), Question Edit (4) |
| Buzzers preparation | Companion List, Companion Edit |
| Create Game | Game Create / Edit (6) |
| In-place buzzer preparation | Game Lobby (7) — companion is assigned to game |
| Introduction | Game Lobby (7) — teams formed, links shared |
| Gameplay | Live Game (8) |
| Results | Game Results (9) |

### Data consistency rules enforced in UI

| Rule (from JDL) | UI enforcement |
|------------------|---------------|
| One captain per team | Game Lobby: captain toggle; server enforces single captain |
| Quiz cannot be deleted if referenced by a Game | Quiz List / Quiz Detail: server returns 409, UI shows toast |
| Game delete only CREATED or FINISHED | Game List: swipe-delete disabled for IN_PROGRESS |
| Teams cannot be added/removed while IN_PROGRESS | Game Lobby limited mode: add/delete hidden |
| Members cannot be removed while IN_PROGRESS | Game Lobby limited mode: remove hidden |
| One IN_PROGRESS game per moderator | Game Lobby: server returns 409 on Start if violated |
| One attempt per team per round | Server enforces; UI does not create attempts |
| `Round.question` and `Round.questionText` mutually exclusive | Live Game: quiz-game shows question picker, ad-hoc shows text field — never both |
