# Companion app proxy pattern

Mobile app (Companion app proxy) is merely relaying the messages from devices to the cloud (and back)

ESP32 → BLE → Phone → Server

ESP32:

- runs BLE GATT service
- exposes TX/RX characteristics

Mobile app:
- connects over BLE
- reads/writes UART-like data
- forwards to server
- use long-polling to receive commands from the cloud

This solution:
- ✅ Works well on both Android and iOS
- ✅ No cables needed
- ✅ No special certification needed

## App Technology Choices
  
Flutter

- ✅ Excellent BLE support
- ✅ Fast dev
- ✅ One codebase
