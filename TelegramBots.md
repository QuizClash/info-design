# Telegram Bots

## Backend

The mini aps are connected to the single backend (The Bot API). At the moment, the backend is implemented inside the backbone,
but with separate endpoints to allow separation from the backbone in the future.

The Bot API endpoints are:
- /bot_api/v1/auth/ to authenticate the Telegram user
- /bot_api/v1/rest/* to replicate all the endpoints of the backbone (the repetition is required to allow the use of the JWT token received from /bot_api/v1/auth/)


## Frontend

The frontend consists of the mini-apps that are connected to the Bot API.
- [mini app](ModeratorMiniApp.md) for Moderator
- mini app for Team Members
