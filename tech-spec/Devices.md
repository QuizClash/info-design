# Hardware devices

The captain can react by pressing a provided button device.
The device will send a signal over ESP-NOW protocol to
the master station. The master station will send the signal via the companion to the cloud.
Moderator can see the time when each captain indicated the readiness (or no answer within the timeout).

## Technologies for the devices

- ESP8266 for button devices
- ESP32 for master station
- ESP-NOW protocol between devices
- Companion app proxy
- Serial (USB-to-UART) between the master station and the host computer connected to the internet
- HTTP between the host computer and the Quiz cloud

## Protocols

- ESP-NOW protocol between devices
- Either [Bluetooth](tech-spec/Companion-app-proxy.md) or [Serial (USB-to-UART)](tech-spec/Serial-protocol.md)
- [HTTP between the host computer or companion and the Quiz cloud](/tech-spec/Remote-protocol.md)
