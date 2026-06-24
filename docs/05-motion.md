# Motion & Control

## Concept

The Pi4 records all unit movement as a replayable path sequence, similar in concept to G-code in CNC machining. A unit navigates a route manually or semi-autonomously, the Pi4 logs every movement command with position data, and that sequence can then be replayed by any unit to retrace the exact path.

The scoop unit records the full route from staging area to load point once. That path is then transmitted to the coop unit so it can navigate to its designated position without needing to independently solve the routing problem.

---

## Operational Flow

```
1. Scoop navigates from staging to load point
   Pi4 records full movement sequence

2. Scoop arrives, loads material
   ToF sensors confirm dock position

3. Scoop signals coop unit
   Transmits recorded path via MQTT

4. Coop replays path to position
   ToF sensors confirm arrival and alignment

5. Scoop dumps to conveyor
   Conveyor transports material to destination

6. Cycle repeats
```

---

## Movement Recording

The Pi4 logs all movement commands including direction, speed, and duration as a timestamped sequence. The MPU6050 gyroscope provides heading data and the MT6701 magnetic encoder tracks distance traveled, giving dead reckoning position throughout the path. ESP32 units receive recorded sequences over MQTT and execute them step by step.

---

## Docking

VL53L0X ToF sensors handle precision positioning at load and dump points. When a unit approaches a dock position the ToF sensor closes the loop, slowing and stopping the unit at the correct distance for material transfer regardless of minor path variation.

---

## Motor Control

Both units use ESP32 controllers receiving MQTT commands from the Pi4. All motor control is handled on the ESP32 for real-time responsiveness. The scoop uses two IBT-2 drivers for tracks and a TB6612 for the actuator. The coop uses three IBT-2 drivers for tracks and conveyor. All units use differential steering where left and right track speed variation produces all required movement including forward, reverse, and turning.

---

## Sync & Calibration

Belt sync will require tuning. Two motors running at nominally the same PWM value will drift over time. Calibration runs will be used to characterize each unit and offset values baked into the movement profiles. The MT6701 encoder provides closed loop feedback on one track for distance correction. A second encoder requires an ADS1115 external ADC since ADC2 pins are unavailable while WiFi is active.

---

## Status

Motor control and sensor telemetry operational on desk demo. Movement recording and path replay are next implementation targets.

---

[← Back](https://github.com/ChrisWells-Dev/autonomous-tracked-robot)

