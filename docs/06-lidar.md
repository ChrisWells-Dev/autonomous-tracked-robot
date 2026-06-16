## Lidar & Mapping

### Current Sensing

Proximity sensing on all units is handled by VL53L0X/L1X ToF sensors via I2C. These provide short range distance measurement used for obstacle detection and precision docking at load and dump points.

---

### Planned: Survey Drone Integration

The longer term vision includes an autonomous survey drone equipped with lidar and a camera providing overhead situational awareness back to the Pi4 command hub. The drone surveys the operational area and feeds real time positional and environmental data to the central brain, giving the system the kind of eyes-on capability that underground operators currently rely on direct observation for.

This data feeds into the digital twin layer, combining ground unit telemetry with aerial survey data into a unified operational picture.

---

### Status

*ToF proximity sensing active in hardware planning. Drone integration is a future phase.*

---

[← Back](https://github.com/ChrisWells-Dev/autonomous-tracked-robot)
