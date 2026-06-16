# Overview & Hardware Stack

## Concept

A multi-unit autonomous tracked robot system designed to demonstrate coordinated
material transport without human intervention. The system consists of one scoop
unit and two conveyor units working in tandem to move material continuously from
point A to point B and back — a closed loop autonomous haulage demonstration.

This project applies embedded systems, robotics software, sensor fusion, and
autonomous coordination to a problem I spent 20 years solving manually underground.
The goal is a working proof of concept that demonstrates both the engineering
capability and the domain understanding behind it.

---

## Architecture

Pi4 acts as the central compute hub and WiFi access point. Unit 1 motors connect
directly to the Pi4 via motor driver. Units 2 and 3 each run an ESP32 that
connects back to the Pi4 over WiFi, receiving commands and reporting state.
All coordination logic runs on the Pi4.

---

## Hardware

### Unit 1 — Scoop (Pi4 direct)


| Component         | Details                                                                   |
| ----------------- | ------------------------------------------------------------------------- |
| Compute           | Raspberry Pi 4                                                            |
| Motor Driver      | v1321 dual IBT-2 carrier, 2x IBT-2                                        |
| Drive             | 12v tracked differential, 2x motors                                       |
| Proximity Sensors | VL53L0X/L1X ToF — front obstacle detection and conveyor dump positioning |
| Camera            | Pi Camera                                                                 |
| Network           | Pi4 WiFi AP host                                                          |

### Units 2 & 3 — Conveyors (ESP32 controlled)


| Component         | Details                             |
| ----------------- | ----------------------------------- |
| Controller        | ESP32                               |
| Motor Driver      | v1321 dual IBT-2 carrier, 2x IBT-2  |
| Drive             | 12v tracked differential, 2x motors |
| Proximity Sensors | VL53L0X/L1X ToF                     |
| Network           | WiFi client, connects to Pi4 AP     |

---

## Planned Units


| Unit               | Role                         | Controller   |
| ------------------ | ---------------------------- | ------------ |
| Unit 1 — Scoop    | Material pickup and deposit  | Pi4 (direct) |
| Unit 2 — Conveyor | Material transport segment 1 | ESP32        |
| Unit 3 — Conveyor | Material transport segment 2 | ESP32        |

---

## Goals

- Autonomous navigation and obstacle avoidance
- Multi-unit coordination without human input
- Continuous closed loop material transport
- Full ROS2 software stack
- ToF based proximity sensing for docking and dump positioning
- Lidar mapping — future phase

---

*Status: Unit 1 hardware assembly in progress.

[← Back](https://github.com/ChrisWells-Dev/autonomous-tracked-robot)
