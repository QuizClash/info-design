# General info

General information and tech design

## Overview

The QuizClash app is a quiz game that allows users to create and play quizzes.
Each quiz has a set of questions, and the user can react by pressing a provided
button device. The device will send a signal over ESP-NOW protocol to
the master station. The user who presses the first wins the right to answer the
question. The master station detects who was the first and sends a signal to the
button devices to expose the winner.
Moderator managers the quizzes and controls the master station. 
The players can see the results of the quizzes and the statistics of the game with their smartphones.

The master station is a microcontroller that runs on the ESP32 chip. It connects to the 
host computer via serial connection. The host computer is merely relaying the messages 
from devices to the cloud. It allows controlling the game from the cloud.

## Technologies for the devices

- ESP8266 for button devices
- ESP32 for master station
- ESP-NOW protocol between devices
- Serial (USB-to-UART) between the master station and the host computer connected to the internet
- HTTP between the host computer and the Quiz cloud

## Protocols

- ESP-NOW protocol between devices
- [Bluetooth](tech-spec/Companion-app-proxy.md) 
- Outdated [Serial (USB-to-UART)](tech-spec/Serial-protocol.md)
- [HTTP between the host computer and the Quiz cloud](/tech-spec/Remote-protocol.md)

## Overview Devices

![architecture](diagrams/QuizClash-Devices.drawio.png)

## Cloud Architecture

![cloud architecture](diagrams/QuizClash-Cloud.drawio.png)
