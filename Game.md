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
- ready (buzzing in is a false start, red blinking, and the round is skipped unless allowed by the moderator)
- started (buzzing in indicates a valid attempt to answer the question)
  
The moderator via a mini-app in EvaliquizModeratorBot can choose the state of the round.
The moderator selects 'ready' and reads the question.
The moderator selects 'started' and waits for the captains to answer.
The started round finishes when all the captains pressed the buzzer or timeout is reached.
The moderator can see the sequence of the captains.
The moderator can choose which captain is to answer the question.
When the answer was wrong, the moderator may either repeat the same question in another round
or select another captain to answer.


