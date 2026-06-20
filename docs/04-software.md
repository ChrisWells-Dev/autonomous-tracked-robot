# Software Architecture

## Overview

The Pi4 acts as the central brain for all units. All coordination logic, sensor processing, and unit communication runs on the Pi4. ESP32 controllers on each unit handle low level motor control and report state back to the Pi4 over WiFi.

---

## Network Architecture

Early consideration was given to running a dedicated router as the network hub for inter-unit communication. This was dropped in favour of running the Pi4 as the network coordinator directly. Simpler stack, fewer failure points, and the Pi4 has the headroom to handle it alongside its other responsibilities.

```
Pi4 (ROS2 + Mosquitto MQTT Broker)
    ├── Scoop ESP32 (WiFi client, motor control + actuator)
    └── Coop ESP32 (WiFi client, motor control + conveyor)
```

---

## Software Stack


| Component           | Technology                                        |
| ------------------- | ------------------------------------------------- |
| OS                  | Ubuntu Server 24.04 LTS (arm64)                   |
| Robotics framework  | ROS2 Jazzy                                        |
| MQTT broker         | Mosquitto                                         |
| ROS2-MQTT bridge    | ros-jazzy-mqtt-client (ika-rwth-aachen)           |
| Motor control       | ESP32 firmware via PlatformIO (Arduino framework) |
| Proximity sensing   | VL53L0X/L1X ToF via I2C (planned)                 |
| Environment mapping | ROS2 + lidar (future phase)                       |

---

## Communication

MQTT runs over the local WiFi network for all inter-unit messaging. The Pi4 runs the Mosquitto broker. ESP32 units subscribe to command topics and publish status back. This keeps real time motor control on the ESP32s where latency matters while the Pi4 handles coordination logic above.

### MQTT Topic Map


| MQTT Topic   | Direction    | Purpose                       |
| ------------ | ------------ | ----------------------------- |
| scoop/cmd    | Pi4 to Scoop | Motor and actuator commands   |
| scoop/status | Scoop to Pi4 | Heartbeat and state reporting |
| coop/cmd     | Pi4 to Coop  | Motor and conveyor commands   |
| coop/status  | Coop to Pi4  | Heartbeat and state reporting |

### MQTT Bridge Config

Located at `~/ros2_ws/config/mqtt_bridge.yaml` on the Pi4:

```yaml
/**/*:
  ros__parameters:
    broker:
      host: localhost
      port: 1883
    bridge:
      ros2mqtt:
        ros_topics:
          - /scoop/cmd
          - /coop/cmd
        /scoop/cmd:
          mqtt_topic: scoop/cmd
          primitive: true
        /coop/cmd:
          mqtt_topic: coop/cmd
          primitive: true
      mqtt2ros:
        mqtt_topics:
          - scoop/status
          - coop/status
        scoop/status:
          ros_topic: /scoop/status
          primitive: true
        coop/status:
          ros_topic: /coop/status
          primitive: true
```

### Mosquitto Remote Access

Located at `/etc/mosquitto/conf.d/remote.conf`:

```
listener 1883 0.0.0.0
allow_anonymous true
```

---

## System Startup

```bash
# Start the MQTT bridge (Pi4)
ros2 launch mqtt_client standalone.launch.xml params_file:=$HOME/ros2_ws/config/mqtt_bridge.yaml

# Verify units are connected
ros2 topic echo /scoop/status
ros2 topic echo /coop/status
```

Both ESP32s connect automatically on power-up. The `alive` heartbeat confirms connectivity every 2 seconds.

---

## Command Reference

### Scoop Unit

```bash
# Left motor
ros2 topic pub /scoop/cmd std_msgs/msg/String "data: 'left_fwd'" --once
ros2 topic pub /scoop/cmd std_msgs/msg/String "data: 'left_rev'" --once
ros2 topic pub /scoop/cmd std_msgs/msg/String "data: 'left_stop'" --once

# Right motor
ros2 topic pub /scoop/cmd std_msgs/msg/String "data: 'right_fwd'" --once
ros2 topic pub /scoop/cmd std_msgs/msg/String "data: 'right_rev'" --once
ros2 topic pub /scoop/cmd std_msgs/msg/String "data: 'right_stop'" --once

# Actuator
ros2 topic pub /scoop/cmd std_msgs/msg/String "data: 'scoop_up'" --once
ros2 topic pub /scoop/cmd std_msgs/msg/String "data: 'scoop_down'" --once
ros2 topic pub /scoop/cmd std_msgs/msg/String "data: 'scoop_stop'" --once

# Kill all
ros2 topic pub /scoop/cmd std_msgs/msg/String "data: 'stop'" --once
```

### Coop Unit

```bash
# Left motor
ros2 topic pub /coop/cmd std_msgs/msg/String "data: 'left_fwd'" --once
ros2 topic pub /coop/cmd std_msgs/msg/String "data: 'left_rev'" --once
ros2 topic pub /coop/cmd std_msgs/msg/String "data: 'left_stop'" --once

# Right motor
ros2 topic pub /coop/cmd std_msgs/msg/String "data: 'right_fwd'" --once
ros2 topic pub /coop/cmd std_msgs/msg/String "data: 'right_rev'" --once
ros2 topic pub /coop/cmd std_msgs/msg/String "data: 'right_stop'" --once

# Conveyor
ros2 topic pub /coop/cmd std_msgs/msg/String "data: 'conv_fwd'" --once
ros2 topic pub /coop/cmd std_msgs/msg/String "data: 'conv_rev'" --once
ros2 topic pub /coop/cmd std_msgs/msg/String "data: 'conv_stop'" --once

# Kill all
ros2 topic pub /coop/cmd std_msgs/msg/String "data: 'stop'" --once
```

---

## ESP32 Firmware

Both ESP32s are flashed via PlatformIO in VS Code using the Arduino framework. PubSubClient handles MQTT. Each unit runs independently with its own command set. Source is in the `firmware/` directory.

### PlatformIO Config (both units)

```ini
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
lib_deps = knolleary/PubSubClient@^2.8
monitor_speed = 115200
```

---

## Digital Twin (Planned)

The long term goal is to connect the system to a live digital twin. Physical unit positions, sensor data, and operational state represented in real time in a 3D environment using Blender or Unreal Engine. Component models are already being built in Fusion 360 and will feed directly into this pipeline.

This bridges the physical system with a visualization layer that mirrors real world state, enabling monitoring, simulation, and eventually predictive control from a 3D interface.

---

[← Back](https://github.com/ChrisWells-Dev/autonomous-tracked-robot)
