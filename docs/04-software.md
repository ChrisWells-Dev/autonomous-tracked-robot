## Software Architecture

### Overview

The Pi4 acts as the central brain for all three units. All coordination logic, sensor processing, and unit communication runs on the Pi4. ESP32 controllers on Units 2 and 3 handle low level motor control and report state back to the Pi4 over WiFi.

---

### Network Architecture

Early consideration was given to running a dedicated router as the network hub for inter-unit communication. This was dropped in favour of running the Pi4 as a WiFi access point directly using hostapd and dnsmasq. Simpler stack, fewer failure points, and the Pi4 has the headroom to handle it alongside its other responsibilities.

```
Pi4 (brain + WiFi AP)
    ├── Unit 1 motors (direct via IBT-2)
    ├── Unit 2 ESP32 (WiFi client)
    └── Unit 3 ESP32 (WiFi client)
```

---

### Communication

MQTT runs over the local WiFi network for all inter-unit messaging. The Pi4 runs the broker. ESP32 units subscribe to command topics and publish state back. This keeps real time motor control on the ESP32s where latency matters while the Pi4 handles coordination logic above.

---

### Software Stack


| Component                 | Technology                  |
| ------------------------- | --------------------------- |
| Central compute           | Raspberry Pi 4, Linux       |
| WiFi AP                   | hostapd + dnsmasq           |
| Inter-unit comms          | MQTT                        |
| Robotics framework        | ROS2                        |
| Motor control (Unit 1)    | Direct PWM via IBT-2        |
| Motor control (Units 2/3) | ESP32 firmware over MQTT    |
| Proximity sensing         | VL53L0X/L1X ToF via I2C     |
| Environment mapping       | ROS2 + lidar (future phase) |

---

### Digital Twin (Planned)

The long term goal is to connect the system to a live digital twin. Physical unit positions, sensor data, and operational state represented in real time in a 3D environment using Blender or Unreal Engine. Component models are already being built in Fusion 360 and will feed directly into this pipeline.

This bridges the physical system with a visualization layer that mirrors real world state, enabling monitoring, simulation, and eventually predictive control from a 3D interface.

---

### Status

*Active development — documentation updated as software stack is implemented.*

---

[← Back](https://github.com/ChrisWells-Dev/autonomous-tracked-robot)
