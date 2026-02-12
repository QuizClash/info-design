# Host to Quiz Hall communication

Description of the communication protocol between Host and the Quiz Hall server.

## Endpoints

The Quiz Hall server exposes the following endpoints:

- '/edge'
- '/hall/<GAME UUID>'

## Authentication

All the requests from the host are authenticated using the JWT token.
The host must send the JWT token in the standard header:
```http
Authorization: Bearer <JWT token>
```
The subject is 'Evaliquiz'. The issuer is the host UUID.
The token might be generated using the following code:

```kotlin
Jwts.builder()
    .subject(SUBJECT)
    .issuer(hostId)
    .issuedAt(issuedAt)
    .id(uuid.toString())
    .signWith(key, Jwts.SIG.HS256)
    .compact()
```

### Edge endpoint

The Host uses the Edge endpoint to get the URL of the game room. This URL is created when the game is started.
(The game is started via another way, in a dedicated UI).
After authentication, this endpoint returns the URL of the game room (with its UUID) or 404 when the game is not started yet.
Once the UUID of the game is known, the Host can use the Hall endpoint to communicate with the game room.

### Hall endpoints

The Host uses the Hall endpoint to communicate with the game room.
The commands and events are defined in the [Serial protocol](Serial-protocol.md).
(The very same data structures are used for the HTTP and Serial connections.)
The messages from USB-to-UART are relayed to the HTTP server.

#### Push endpoint

The URL is `http://localhost:8080/api/v1/hall/push/<quiz-id>/`.
All the JSON events received via serial are pushed to this endpoint.

#### Long-polling endpoint

The URL is `http://localhost:8080/api/v1/hall/poll/<quiz-id>`.

The host keeps a long-polling HTTP connection to the QuizHall server to get
the commands to send them to the ESP32 master station over serial.
