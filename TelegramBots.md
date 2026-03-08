# Telegram Bots

Operational and configuration reference for the two Evaliquiz Telegram bots.

Companion documents:
- [Authentication.md](Authentication.md) — JWT flow and initData verification
- [Authorization.md](Authorization.md) — entity access rules per actor
- [ModeratorMiniApp.md](ModeratorMiniApp.md) — moderator mini-app screen catalog
- [README.md](README.md) — system overview, SSE spec, game flows

---

## Bots

### @EvaliquizModeratorBot

**Purpose:** Moderator registration and mini-app launcher.

**Commands:**
| Command | Description |
|---------|-------------|
| `/start` | Register moderator profile |

**Menu Button:** Opens the moderator mini-app (see [ModeratorMiniApp.md](ModeratorMiniApp.md)).

**Inline mode:** Off.
**Groups / channels:** Disabled.

### @EvaliquizTeamBot

**Purpose:** Team member registration via deep links and team mini-app launcher.

**Commands:**
| Command | Description |
|---------|-------------|
| `/start` | Entry point for deep-link joining |

**Menu Button:** None (team members enter via deep links, not via the menu).

**Inline mode:** Off.
**Groups / channels:** Disabled.

---

## Setup — BotFather

### EvaliquizModeratorBot

1. `/newbot` — create the bot, choose display name and username
2. Copy the bot token (see [Backend setup](#setup--backend) below)
3. `/setcommands` for @EvaliquizModeratorBot:
   ```
   start - Register your moderator profile
   ```
4. `/setmenubutton` — set the Web App URL to the moderator mini-app (e.g. `https://app.evaliquiz.example/moderator`)
5. Bot Settings → Domain — add the mini-app domain so Telegram allows the Web App to open

### EvaliquizTeamBot

1. `/newbot` — create the bot, choose display name and username
2. Copy the bot token (see [Backend setup](#setup--backend) below)
3. `/setcommands` for @EvaliquizTeamBot:
   ```
   start - Join a game
   ```
4. Bot Settings → Domain — add the team mini-app domain

---

## Setup — Backend

Bot tokens are configured via environment variables and read by `BotApiProperties.java` under the prefix `evaliquiz.bot-api`.

### Environment variables

| Variable | Maps to | Description |
|----------|---------|-------------|
| `EVALIQUIZ_MODERATOR_BOT_TOKEN` | `evaliquiz.bot-api.moderator-bot.token` | Token for @EvaliquizModeratorBot |
| `EVALIQUIZ_TEAM_BOT_TOKEN` | `evaliquiz.bot-api.team-bot.token` | Token for @EvaliquizTeamBot |

### Configuration files

**`application-dev.yml`** provides dev defaults with env-var overrides:
```yaml
evaliquiz:
  bot-api:
    moderator-bot:
      token: ${EVALIQUIZ_MODERATOR_BOT_TOKEN:test-moderator-bot-token}
    team-bot:
      token: ${EVALIQUIZ_TEAM_BOT_TOKEN:test-team-bot-token}
```

The dev defaults (`test-moderator-bot-token`, `test-team-bot-token`) allow unit tests to run without real tokens. For actual Telegram interaction, set the environment variables to real BotFather tokens.

**Production:** Define the environment variables in your deployment environment. Never commit real tokens to git.

### Code reference

- `BotApiProperties.java` — `@ConfigurationProperties(prefix = "evaliquiz.bot-api")`, exposes `moderatorBot.token` and `teamBot.token`
- `BotApiAuthController.java` — `/bot_api/v1/auth` endpoint; selects the bot token based on the `bot` field in the request (`"moderator"` or `"team"`)

---

## Backend

The mini-apps connect to a single backend (the Bot API). The Bot API is implemented inside the backbone with separate endpoints to allow future separation.

### Endpoints

| Path | Purpose |
|------|---------|
| `POST /bot_api/v1/auth` | Authenticate a Telegram user — verifies `initData` signature against the selected bot token, returns a JWT |
| `/bot_api/v1/rest/*` | Entity CRUD — mirrors backbone endpoints, authenticated with the JWT from `/auth` |
| `GET /api/games/{id}/events` | SSE — moderator game events (requires moderator JWT) |
| `GET /api/games/{id}/team-events` | SSE — team game events (requires team JWT; question hints excluded) |

### Auth endpoint — bot selection

The `POST /bot_api/v1/auth` request body includes a `bot` field:
- `"moderator"` — validates `initData` against the moderator bot token
- `"team"` — validates `initData` against the team bot token

See [Authentication.md](Authentication.md) for the full JWT flow (initData verification, token structure, expiry).

### Access rules

See [Authorization.md](Authorization.md) for the entity access matrix per actor (moderator, team bot, companion, admin).

---

## Frontend

The frontend consists of Telegram Web Apps (mini-apps) connected to the Bot API.

### Hosting requirements

- Served over HTTPS with a valid TLS certificate
- Domain registered in BotFather Bot Settings (see [Setup — BotFather](#setup--botfather))
- Must be accessible from the public internet (Telegram loads the Web App in an iframe)

### TWA SDK integration

Both mini-apps use the Telegram Web App (TWA) SDK for:
- `initData` — passed to `/bot_api/v1/auth` for authentication
- `themeParams` — adaptive theming
- `BackButton` — navigation stack
- Haptic feedback

### Mini-apps

| Mini-app | Bot | Spec |
|----------|-----|------|
| Moderator | @EvaliquizModeratorBot | [ModeratorMiniApp.md](ModeratorMiniApp.md) |
| Team | @EvaliquizTeamBot | Spec TBD |

---

## Deep Links

### Team link (join a specific team)

```
https://t.me/EvaliquizTeamBot?start=team_{teamId}
```

The team member opens the link, the bot finds or creates a `TelegramProfile` + `User`, registers a `TeamMember` (`captain=false`) linked to that team, and returns the team mini-app link.

### Moderator link (join any team in the active game)

```
https://t.me/EvaliquizTeamBot?start=moderator{moderatorUserId}
```

The team member opens the link, the bot finds or creates a `TelegramProfile` + `User`, looks up the moderator's IN_PROGRESS game, and returns the team mini-app link. The team member selects a team within the mini-app.

Both link formats are displayed in the moderator mini-app's Game Lobby screen (see [ModeratorMiniApp.md](ModeratorMiniApp.md#7-game-lobby)).
