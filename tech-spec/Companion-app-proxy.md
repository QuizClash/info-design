# Phone-as-gateway architecture or Companion app proxy pattern

ESP32 → BLE → Phone → Server

ESP32:

- runs BLE GATT service
- exposes TX/RX characteristics

Mobile app:
- connects over BLE
- reads/writes UART-like data
- forwards to server

This solution:
- ✅ Works well on both Android and iOS
- ✅ No cables needed
- ✅ No special certification needed

## App Technology Choices
  
Flutter

- ✅ Excellent BLE support
- ✅ Fast dev
- ✅ One codebase