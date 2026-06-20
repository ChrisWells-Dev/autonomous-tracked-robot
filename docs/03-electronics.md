# Electronics & Wiring

## Power Rails (Same for Both Units)

Each unit runs off a shared breadboard with two rails:

* Bench supply 12V+ to 12V bar
* Bench supply 12V- to ground bar
* ESP32 VN pin to 5V bar
* ESP32 GND to ground bar
* All grounds share one rail

**Do not connect 12V to the ESP32. ESP32 is powered via USB only.**

---

## Scoop Unit

### ESP32 Pin Map


| GPIO | Function                    |
| ---- | --------------------------- |
| 14   | Right Motor RPWM (IBT-2 #1) |
| 27   | Right Motor LPWM (IBT-2 #1) |
| 25   | Left Motor RPWM (IBT-2 #2)  |
| 26   | Left Motor LPWM (IBT-2 #2)  |
| 33   | Actuator AIN1 (TB6612)      |
| 32   | Actuator AIN2 (TB6612)      |
| VN   | 5V out to 5V bar            |
| GND  | Ground bar                  |

### IBT-2 #1 (Right Motor)


| IBT-2 Pin | Connection         |
| --------- | ------------------ |
| B+        | 12V bar            |
| B-        | Ground bar         |
| M+        | Right motor lead 1 |
| M-        | Right motor lead 2 |
| VCC       | 5V bar             |
| GND       | Ground bar         |
| R\_EN     | 5V bar             |
| L\_EN     | 5V bar             |
| RPWM      | ESP32 GPIO 14      |
| LPWM      | ESP32 GPIO 27      |
| R\_IS     | Not connected      |
| L\_IS     | Not connected      |

### IBT-2 #2 (Left Motor)


| IBT-2 Pin | Connection        |
| --------- | ----------------- |
| B+        | 12V bar           |
| B-        | Ground bar        |
| M+        | Left motor lead 1 |
| M-        | Left motor lead 2 |
| VCC       | 5V bar            |
| GND       | Ground bar        |
| R\_EN     | 5V bar            |
| L\_EN     | 5V bar            |
| RPWM      | ESP32 GPIO 25     |
| LPWM      | ESP32 GPIO 26     |
| R\_IS     | Not connected     |
| L\_IS     | Not connected     |

### TB6612 (Actuator)


| TB6612 Pin | Connection                               |
| ---------- | ---------------------------------------- |
| VM         | 12V bar                                  |
| VCC        | 5V bar                                   |
| GND (x3)   | Ground bar (all three must be connected) |
| STBY       | 5V bar                                   |
| PWMA       | 5V bar                                   |
| AIN1       | ESP32 GPIO 33                            |
| AIN2       | ESP32 GPIO 32                            |
| AO1        | Actuator black wire                      |
| AO2        | Actuator red wire                        |

---

## Coop Unit

### ESP32 Pin Map


| GPIO | Function                    |
| ---- | --------------------------- |
| 14   | Right Motor RPWM (IBT-2 #1) |
| 27   | Right Motor LPWM (IBT-2 #1) |
| 25   | Left Motor RPWM (IBT-2 #2)  |
| 26   | Left Motor LPWM (IBT-2 #2)  |
| 32   | Conveyor RPWM (IBT-2 #3)    |
| 33   | Conveyor LPWM (IBT-2 #3)    |
| VN   | 5V out to 5V bar            |
| GND  | Ground bar                  |

### IBT-2 #1 (Right Motor)

Same wiring pattern as Scoop IBT-2 #1.

### IBT-2 #2 (Left Motor)

Same wiring pattern as Scoop IBT-2 #2.

### IBT-2 #3 (Conveyor)


| IBT-2 Pin | Connection            |
| --------- | --------------------- |
| B+        | 12V bar               |
| B-        | Ground bar            |
| M+        | Conveyor motor lead 1 |
| M-        | Conveyor motor lead 2 |
| VCC       | 5V bar                |
| GND       | Ground bar            |
| R\_EN     | 5V bar                |
| L\_EN     | 5V bar                |
| RPWM      | ESP32 GPIO 32         |
| LPWM      | ESP32 GPIO 33         |
| R\_IS     | Not connected         |
| L\_IS     | Not connected         |

---

## Planned Additions

* VL53L0X/L1X ToF sensors (I2C on GPIO 21/22) for proximity and dump positioning
* Limit switches for actuator end-stop detection
* MPU6050 IMU for heading and tilt
* MT6701 magnetic encoders on drive motors for odometry

---

## Notes

* If a motor spins the wrong direction, swap M+ and M- on that IBT-2
* TB6612 has 3 GND pins. All 3 must connect to ground bar.
* ESP32s are powered via USB only. 12V never connects to the ESP32.

---

[← Back](https://github.com/ChrisWells-Dev/autonomous-tracked-robot)
