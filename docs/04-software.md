# Software Architecture

## Overview

The Pi4 acts as the central brain for all units. All coordination logic, sensor processing, and unit communication runs on the Pi4. ESP32 controllers on each unit handle low level motor control, sensor reads, and report state back to the Pi4 over WiFi.

---

## Network Architecture

```
Pi4 (ROS2 + Mosquitto MQTT Broker)
    ├── Scoop ESP32 (WiFi, motors + actuator + sensors)
    └── Coop ESP32 (WiFi, motors + conveyor + sensors)
```

---

## Software Stack

| Component           | Technology                                        |
| --------------------- | --------------------------------------------------- |
| OS                  | Ubuntu Server 24.04 LTS (arm64)                   |
| Robotics framework  | ROS2 Jazzy                                        |
| MQTT broker         | Mosquitto                                         |
| ROS2-MQTT bridge    | ros-jazzy-mqtt-client (ika-rwth-aachen)           |
| Motor control       | ESP32 firmware via PlatformIO (Arduino framework) |
| Proximity sensing   | 2x VL53L0X ToF via I2C (Adafruit library)         |
| IMU                 | MPU6050 accelerometer/gyroscope via I2C           |
| Motor feedback      | MT6701 magnetic encoder via analog                |
| Environment mapping | ROS2 + lidar (future phase)                       |

---

## Communication

MQTT runs over the local WiFi network for all inter-unit messaging. The Pi4 runs the Mosquitto broker. ESP32 units subscribe to command topics and publish sensor telemetry back at 200ms intervals.

### MQTT Topic Map

| MQTT Topic   | Direction    | Purpose                     |
| -------------- | -------------- | ----------------------------- |
| scoop/cmd    | Pi4 to Scoop | Motor and actuator commands |
| scoop/status | Scoop to Pi4 | Sensor telemetry (JSON)     |
| coop/cmd     | Pi4 to Coop  | Motor and conveyor commands |
| coop/status  | Coop to Pi4  | Sensor telemetry (JSON)     |

### Telemetry Format

Each ESP32 publishes a JSON status message every 200ms:

```json
{
  "d1": 342,
  "d2": 8190,
  "ax": -10568,
  "ay": -9464,
  "az": 8076,
  "gx": 549,
  "gy": 1038,
  "gz": 427,
  "le": 0,
  "lr": 0,
  "el": 1382
}
```

| Field      | Sensor       | Description                                    |
| ------------ | -------------- | ------------------------------------------------ |
| d1         | VL53L0X #1   | Front distance in mm (8190 = out of range)     |
| d2         | VL53L0X #2   | Bucket/hopper distance in mm                   |
| ax, ay, az | MPU6050      | Accelerometer raw values (x, y, z)             |
| gx, gy, gz | MPU6050      | Gyroscope raw values (x, y, z)                 |
| le         | Limit switch | Actuator extended (0 or 1)                     |
| lr         | Limit switch | Actuator retracted (0 or 1)                    |
| el         | MT6701       | Encoder analog value (0-4095 over 360 degrees) |

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

Both ESP32s connect automatically on power-up.

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

Both ESP32s are flashed via PlatformIO in VS Code using the Arduino framework. Source is in the `firmware/` directory.

### PlatformIO Config

```ini
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
lib_deps = 
  knolleary/PubSubClient@^2.8
  adafruit/Adafruit_VL53L0X@^1.2
  electroniccats/MPU6050@^1.3
monitor_speed = 115200
```

### Sensor Initialization

VL53L0X sensors share the I2C bus and require staggered XSHUT sequencing at boot to assign unique addresses. The MPU6050 initializes first while the bus is clean. Sensor initialization order:

1. MPU6050 (address 0x68, AD0 tied to ground)
2. VL53L0X #2 (XSHUT on GPIO 5, assigned address 0x31)
3. VL53L0X #1 (XSHUT on GPIO 16, assigned address 0x30)

---

## Digital Twin (Planned)

The long term goal is to connect the system to a live digital twin. Physical unit positions, sensor data, and operational state represented in real time in a 3D environment. Component models are already being built in Fusion 360.

---

[← Back](https://github.com/ChrisWells-Dev/autonomous-tracked-robot)

