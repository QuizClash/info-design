# Serial (USB-to-UART) communication

Description of the serial communication protocol between Host and Master Station.

## Commands

Commands are messages from Host to Station. Commands are formatted as ASCII strings in the upper case.
The commands end with a line break. A command may have optional space-separated parameters.

- STATUS: Get the current state
- RESET: Reregister the button slaves
- IDLE: wait
- READY: report false start
- START: wait for a button press
- SHOW 2: show the second button as the winner

## Events

Events are messages from Station to Host. The events should be formatted as JSON.
The data coming from serial is parsed as JSON. When the format is correct, it is considered to be a valid event,
otherwise it is considered to be a log message from the master station, and it is printed to the console.
The first field of the event is the event kind.

The event kinds are:
1. STATION: when it is sent from the master station to the host.
2. BUTTON: it is sent when a button is pressed or its status is asked.

The JSON events do not have line breaks, but for better readability they are added in the examples with line breaks.

### Format for STATION

The event is sent when the master station starts (or restarts) or when it is requested via STATUS command.

| Field     | Type   | Presence | Description                             |
|:----------|:-------|:---------|:----------------------------------------|
| `kind`    | String | required | The kind/type of message ("STATION")    |
| `version` | String | required | Serial Protocol version ("1.0-beta1")   |
| `bssid`   | String | required | MAC address of the Master station       |
| `millis`  | Long   | required | Current uptime in milliseconds          |
| `mode`    | Int    | required | StationMode (see below)                 |
| `buttons` | Object | required | Maps button's number to its MAC address |
| `results` | Object | required | Maps button's number to elapsed time    |
| `error`   | Int    | optional | Error code when present (0 - Ok)        |

**Example STATION event:**
```json
{
  "kind": "STATION",
  "version": "1.0-beta1",
  "bssid": "00:70:07:18:D6:D8",
  "mode": "10",
  "millis": 24821,
  "buttons": {
    "1": "B4:E6:2D:72:98:56",
    "2": "48:3F:DA:5D:DB:02",
    "3": "EC:FA:BC:7B:CB:8E"
  },
  "results": {
    "1": 1872
  }
}
```

#### Master Station Modes

This is how the master station modes are defined in ESP32 code:
```c++
/* Station modes */
enum StationMode : uint8_t {
ERROR_MODE = 0,     // error detected
DETECT_MODE = 10,   // register the buttons
IDLE_MODE = 20,     // ping buttons
READY_MODE = 30,    // wait for start
STARTED_MODE = 40,  // started
};
```

### Format for BUTTON

The event is sent when the master station receives a button action or when it is requested via STATUS command.

| Field       | Type   | Presence | Description                         |
|:------------|:-------|:---------|:------------------------------------|
| `kind`      | String | required | The kind/type of message ("BUTTON") |
| `id`        | Int    | required | Registered number of the button     |
| `elapsed`   | Int    | required | Elapsed time for the button         |
| `condition` | Int    | required | Condition of the button (see below) |
| `error`     | Int    | optional | Error code when present (0 - Ok)    |

**Example BUTTON event:**

```json
{
  "kind": "BUTTON",
  "id": 2,
  "elapsed": 0,
  "condition": 30,
  "error": 0
}
```
#### Button conditions

This is how the master station modes are defined in ESP32 code:
```c++
/* Possible conditions (states/modes) of the buttons */
enum ButtonCondition : uint8_t {
  ERROR_CONDITION = 0,
  NO_MASTER_CONDITION = 10,
  SLEEP_CONDITION = 15, // deep sleep to save energy, not implemented
  IDLE_CONDITION = 20,
  READY_CONDITION = 30, // detect false start
  STARTED_CONDITION = 40, // start broadcast received
  FINISHED_CONDITION = 50, // button pressed
  SHOW_CONDITION = 60 // show result
};
```


