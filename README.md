# General info

Evaliquiz - General information and tech design

## Overview

The Evaliquiz application is a quiz game that allows players to create and play quizzes with a moderator.
A few players form a team (as TeamMember), and they have to answer questions after a start is announced by the moderator.
The teams choose a captain who will be buzzing in to indicate their readiness to answer.
The moderator can see the sequence in which captains buzzed in and choose the one to answer.
The chosen team gives an answer to the question.
Moderator takes the answer and decides if it was right or wrong.

Roles:
- moderator: the person who creates and manages the quizzes and teams
- teams: the players who play the quizzes
- captain: the person who is buzzing in
- team member: the users who play the quizzes and help the captain answer the questions

The game is described in the [Game](Game.md) section.

### Roles

Moderator is driving the whole game:
- creates and manages the quizzes and questions
- creates the teams
- designates captains for each team
- announces the questions and the start of each question
- decides who will answer the question
- decides if the answer was right or wrong
- takes the results of the questions and displays them

Team:
- has fun playing the game

Captain:
- buzzes in

Team Member:
- answer questions
- observe the results of buzzing in

### Buzzing in

Captains may use either a physical button (buzzer) or a smartphone Telegram mini-app to indicate their readiness.

### Hardware devices

Buzzers and master station are optional for the game. When used, it must be connected and
configured before the game starts.
[Hardware devices](tech-spec/Devices.md) make the game more attractive.

## Overview Use case

![use-case](diagrams/Evaliquiz-UseCase.drawio.png)

