## Motion & Control

### Concept

The Pi4 records all unit movement as a replayable path sequence, similar in concept to G-code in CNC machining. A unit navigates a route manually or semi-autonomously, the Pi4 logs every movement command with position data, and that sequence can then be replayed by any unit to retrace the exact path.

This means the scoop unit records the full route from staging area to load point once. That path is then transmitted to the ESP32 controlled conveyor units so they can navigate to their designated positions without needing to independently solve the routing problem.

---

### Operational Flow

```
1. Scoop navigates from staging to load point
   Pi4 records full movement sequence

2. Scoop arrives, loads material
   ToF sensors confirm dock position

3. Scoop signals ESP32 conveyor units
   Transmits recorded path via MQTT

4. Conveyor units replay path to position
   ToF sensors confirm arrival and alignment

5. Scoop dumps to conveyor
   Conveyor transports material to destination

6. Cycle repeats
```

---

### Movement Recording

The Pi4 logs all movement commands including direction, speed, and duration as a timestamped sequence. This gives an accurate dead reckoning record of where a unit has been and what it took to get there. ESP32 units receive this sequence over MQTT and execute it step by step.

---

### Docking

VL53L0X/L1X ToF sensors handle precision positioning at load and dump points. When a unit approaches a dock position the ToF sensor closes the loop, slowing and stopping the unit at the correct distance for material transfer regardless of minor path variation.

---

### Motor Control

Unit 1 motors connect directly to the Pi4 via a v1321 dual IBT-2 carrier. Units 2 and 3 are controlled by ESP32s receiving MQTT commands from the Pi4. All units use differential steering where left and right belt speed variation produces all required movement including forward, reverse, and turning.

---

### Sync & Calibration

Belt sync will require tuning. Two motors running at nominally the same PWM value will drift over time. Calibration runs will be used to characterize each unit and offset values baked into the movement profiles. Encoder feedback is a future addition for closed loop correction.

---

### Status

*Architecture defined — implementation begins with Unit 1 motor bring-up.*

---

[← Back](https://github.com/ChrisWells-Dev/autonomous-tracked-robot)
