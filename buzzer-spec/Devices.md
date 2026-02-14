# Hardware devices

The captain can react by pressing a provided button device (buzzing in).
The device (buzzer) will send a signal over ESP-NOW protocol to
the master station. The master station will send the signal via the companion to the cloud.
Moderator can see the time when each captain sent an attempt (or no answer within the timeout).

## Technologies for the devices

- ESP8266 for button devices
- ESP32 for master station
- ESP-NOW protocol between devices
- Companion app proxy
- Serial (USB-to-UART) between the master station and the host computer connected to the internet
- HTTP between the host computer and the Quiz cloud

## Protocols

- ESP-NOW protocol between devices
- Either [Bluetooth](Companion-app-proxy.md) or [Serial (USB-to-UART)](Serial-protocol.md)
- [HTTP between the companion and the Quiz cloud](Companion-protocol.md)
