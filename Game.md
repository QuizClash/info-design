# Game description

Overview of how the game is played and the rules.
For the UI, this project is heavily using Telegram bots and mini-app.
Thus, all the users are expected to have Telegram installed and
configured to play.

## Staged

1. Preliminary preparation (once)
2. Quiz preparation
3. Buzzers preparation
4. Create Game
5. In-place buzzer preparation
6. Introduction
7. Gameplay

## Preliminary preparation

This stage is done only once when a user decides to join the project.
The user should register as a moderator and configure buzzers if needed.

- Go to the [registration page](https://t.me/EvaliquizModeratorBot)
- Register as a moderator
- Configure buzzers if needed

TODO: describe the buzzers

## Quiz preparation

This stage is done independent of any game.
The moderator via a mini-app in EvaliquizModeratorBot can do the following actions:
- Create a new quiz (quiz has a mandatory title and an optional description)
- Add questions to the quiz (a question has title, text and hint only shown to the moderator)
- Edit existing quiz
- Edit questions in the existing quiz

Moderator only gets access to his own quizzes and questions.

## Buzzers preparation

TODO Configure a companion

## Create Game

The moderator via a mini-app in EvaliquizModeratorBot can do the following actions:
- Create a new game
- Assign an existing quiz to the game (optional)
- If buzzers are expected, select the companion to use for this game

When no quiz was assigned, the game goes with ad-hoc questions.
The moderator enters ad-hoc questions at the start of each round.
They are stored as text on the Round itself (`Round.questionText`) and cannot be selected from the database.
When a quiz is assigned, the round references a pre-made `Question` entity instead.

## In-place buzzer preparation

The moderator can do the following actions:
- power on a buzzer for every team
- power on the master station
- configure the master station (TODO)
- make sure the buzzer is working properly (TODO)

## Introduction

The teams are introduced to the game by the moderator. The captains may press the buzzer to test how it works.

The moderator via a mini-app in EvaliquizModeratorBot can do the following actions:
- order any buzzer to blink
- order a test run of the game (captains and teams can see how it works)

## Gameplay

The game consists of rounds.
The round has the following states:
- idle (buzzing in causes green blinking)
- ready (buzzing in is a false start, red blinking, and the round has failed for the captain)
- started (buzzing in indicates a valid attempt to answer the question)
- answered (buzzing in indicates a wrong answer, red blinking, and the round has failed for the captain)
- finished (buzzing in causes green blinking)
  
The moderator via a mini-app in EvaliquizModeratorBot can choose the state of the round.
The moderator selects 'ready' and reads the question.
The moderator selects 'started' and waits for the captains to answer.
The round transitions to ANSWERED when the 'Buzzed in done' condition is met (see Round state machine below).
The moderator can see the sequence of the captains.
The moderator can choose which captain is to answer the question.
When the answer was wrong, the moderator may either repeat the same question in another round
or select another captain to answer.

## State machines

### Game state machine

```
CREATED → IN_PROGRESS → FINISHED
```
Only one Game per moderator can be in progress at a time.
The Game is in progress, Buzzer may connect, and users may access the Game via Telegram bot

| From | To | Trigger                | Side effects                                       |
|:-----|:---|:-----------------------|:---------------------------------------------------|
| CREATED | IN_PROGRESS | Moderator action in UI |                                                    |
| IN_PROGRESS | FINISHED | Moderator action in UI |                                                    |

### Round state machine

```
IDLE → READY → STARTED → ANSWERED → FINISHED
```

The moderator explicitly creates a new round entity in the IDLE state.

Round is considered answered when (Buzzed in done):
- all captains buzzed in
- timeout reached and at least one captain buzzed in

If timeout is reached and no captain has buzzed in, the round remains in STARTED.
The moderator must explicitly transition it to FINISHED or READY.

The moderator can explicitly set the round to READY at any time, regardless of the current state (to repeat a failed round).
The moderator can explicitly set the round to FINISHED at any time, regardless of the current state.

| From | To | Trigger                | Side effects |
|:-----|:---|:-----------------------|:-------------|
| IDLE | READY | Moderator action in UI | |
| READY | STARTED | Moderator action in UI | |
| STARTED | ANSWERED | Buzzed in done         | |
| ANSWERED | FINISHED | Moderator action in UI | |
| IDLE | FINISHED | Moderator action in UI | |
| READY | FINISHED | Moderator action in UI | |
| STARTED | FINISHED | Moderator action in UI | |
| STARTED | READY | Moderator action in UI | Resets the round; existing attempts are deleted |
| ANSWERED | READY | Moderator action in UI | Replays the round; existing attempts are deleted |
| FINISHED | READY | Moderator action in UI | Reopens the round; existing attempts are deleted |

### Round timeout

`Game.roundTimeout` is in seconds.

The server tracks the timeout. When a round transitions to STARTED, the server records `Round.startedAt`.
The server monitors the elapsed time:

- If `roundTimeout` seconds have passed **and** at least one Attempt exists → the round auto-transitions to ANSWERED
- If `roundTimeout` seconds have passed **and** no Attempts exist → the round remains in STARTED; the moderator must manually transition it

The moderator's mini-app displays a countdown timer derived from `Game.roundTimeout` and `Round.startedAt`.

### Attempt creation

An Attempt is created when a captain buzzes in. There are two paths:

**Hardware path** (Companion): Button press → ESP-NOW → Master Station → Serial → Companion app → HTTP POST to backbone → Attempt created. The button `id` maps to a Team via `Team.index`. This mapping is established during in-place buzzer preparation.

**Mini-app path** (no hardware): Captain taps the buzzer button in the EvaliquizTeamBot mini-app → EvaliquizTeamBot service account sends POST to backbone → Attempt created. `Attempt.receivedAt` is set by the server on receipt. `Attempt.reaction` is null (no hardware timing).

In both cases:
- An Attempt can only be created when the round is in STARTED status
- Only one Attempt per team per round is allowed
- The moderator cannot create Attempts directly

## Scoring

The system does not enforce any scoring rules.
The score is completely driven and determined by the moderator outside the application.
The application only records attempts and the moderator's judgment (`Attempt.chosen` and `Attempt.correct`).
