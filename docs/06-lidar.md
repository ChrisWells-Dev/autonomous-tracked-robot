# Lidar & Mapping

## Current Sensing

Proximity sensing on all units is handled by dual VL53L0X ToF sensors via I2C using the Adafruit library. These provide short range distance measurement (up to 2m) used for obstacle detection and precision docking at load and dump points. Each unit runs two sensors on a shared I2C bus with XSHUT-based address assignment at boot.

---

## Planned: SLAM Lidar

The RPLIDAR C1 (DTOF, 360 degree, 12m range) is the target sensor for full SLAM mapping. It connects directly to the Pi4 via USB and publishes a `/scan` LaserScan topic through the `sllidar_ros2` driver. The DTOF method is better suited for dusty underground environments than triangulation-based alternatives. This is a post-incorporation purchase.

---

## Planned: Survey Drone Integration

The longer term vision includes an autonomous survey drone equipped with lidar and a camera providing overhead situational awareness back to the Pi4 command hub. The drone surveys the operational area and feeds real time positional and environmental data to the central brain, giving the system the kind of eyes-on capability that underground operators currently rely on direct observation for.

This data feeds into the digital twin layer, combining ground unit telemetry with aerial survey data into a unified operational picture.

---

## Status

ToF proximity sensing operational on desk demo. SLAM lidar and drone integration are future phases.

---

[← Back](https://github.com/ChrisWells-Dev/autonomous-tracked-robot)

