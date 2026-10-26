# Aqua Autonomous Bot

A solar-powered, autonomous catamaran that collects **floating surface waste** (plastic and other debris) from lakes, ponds and rivers. It is built for small rural water bodies in Tamil Nadu, India, and supports SDG 6 (Clean Water and Sanitation) and SDG 14 (Life Below Water).

> **Scope:** surface-layer collection only. A front-mounted conveyor and mesh basket lift debris floating on the surface. The bot does not clean below the surface.

Built for **Hack for Social Cause** (theme: Environment & Sustainability) by students of Sri Sairam Engineering College, Chennai.

## Status

| Part | Status |
|---|---|
| Control software (manual, GPS waypoints, obstacle avoidance, e-stop) | Written, unit-tested and runnable in simulation |
| Wi-Fi web dashboard | Working (tested against the simulator) |
| Hardware drivers (motors, sonar, GPS, compass, camera) | Written, **not yet validated on the finished boat** |
| Water trials | Pending |
| YOLOv8n debris model | Stock `yolov8n.pt` loads; a floating-debris dataset and fine-tuning are still to do |

## Features

- Twin-thruster differential steering and a conveyor motor
- Browser dashboard for manual drive, conveyor control and emergency stop (phone friendly)
- GPS + compass waypoint navigation
- Ultrasonic obstacle stop and back-off
- Dead-man timeout on manual drive and a relay-based emergency stop
- Camera-based debris detection (YOLOv8n); the conveyor speeds up when debris is seen
- Full simulator, so everything runs on a laptop without hardware

## Repository layout

```
aqua_bot/          control software (controller, navigation, drivers, dashboard)
sample_data/       sample waypoints, NMEA GPS lines, sonar readings
tests/             pytest unit and simulation tests
docs/              technical design document (architecture, wiring, test plan)
```

## Hardware (prototype)

| Function | Part |
|---|---|
| Computer | Raspberry Pi (Pi Zero 2 W / 4 for the prototype; Pi 5 for the full-scale design) |
| Hulls | Two sealed PVC pipes (about 40-45 cm), flat deck, 20 cm open front channel |
| Thrusters | 2 DC motors with propellers or paddle wheels, one per hull |
| Collector | N20 gear motor driving a mesh conveyor |
| Motor drivers | 2 x TB6612FNG |
| Navigation | NEO-6M GPS, QMC5883L compass |
| Obstacle sensor | JSN-SR04T waterproof ultrasonic |
| Camera | Raspberry Pi camera or USB camera |
| Power | 2 x 18650 in 2S with BMS, LM2596 buck converter to 5 V, small solar panel with charge controller |
| Safety | Relay that cuts motor power (emergency stop) |

Wiring and the pin map are in [docs/TECHNICAL_DESIGN.md](docs/TECHNICAL_DESIGN.md).

## Setup

### Run the simulator (any computer)

```bash
git clone <your-repo-url> aqua-autonomous-bot
cd aqua-autonomous-bot
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements-dev.txt
python -m aqua_bot.main --simulate
```

Open <http://localhost:8000>, press **Auto** and watch the status panel.

### Run on the Raspberry Pi

1. Flash Raspberry Pi OS and enable I2C and the serial port: `sudo raspi-config` > Interface Options (I2C on; serial login shell off, serial hardware on).
2. Wire the boat as described in the design document.
3. Install:
   ```bash
   python3 -m venv .venv --system-site-packages && source .venv/bin/activate
   pip install -r requirements-pi.txt
   ```
4. Copy `.env.example` to `.env`, edit it, then load it: `set -a; source .env; set +a`
5. Put the Pi on the boat's Wi-Fi hotspot (or connect it to your phone hotspot) and run:
   ```bash
   python -m aqua_bot.main
   ```
6. Open `http://<pi-ip>:8000` on your phone.

### Calibrate before water trials

- **Compass:** rotate the boat through a full circle on the bench and set the X/Y offsets in `hardware.Compass`. Set `AQUA_MAG_DECLINATION_DEG` for your location.
- **GPS:** check that the receiver has a fix outdoors before using Auto mode.
- **Bench test:** lift the boat clear of the water and test every motor direction, the conveyor and the emergency stop first.

## Environment variables

| Variable | Default | Meaning |
|---|---|---|
| `AQUA_HOST` / `AQUA_PORT` | `0.0.0.0` / `8000` | Dashboard address |
| `AQUA_WAYPOINTS` | `sample_data/waypoints.json` | Mission waypoints file |
| `AQUA_GPS_PORT` / `AQUA_GPS_BAUD` | `/dev/serial0` / `9600` | GPS serial port |
| `AQUA_I2C_BUS` | `1` | I2C bus for the compass |
| `AQUA_MAG_DECLINATION_DEG` | `0.0` | Magnetic declination correction |
| `AQUA_OBSTACLE_STOP_CM` | `60` | Stop distance for obstacles |
| `AQUA_ARRIVAL_RADIUS_M` | `3.0` | Waypoint reached radius |
| `AQUA_BASE_SPEED` | `0.6` | Cruise throttle, 0-1 |
| `AQUA_CONVEYOR_SPEED` | `0.5` | Conveyor throttle, 0-1 |
| `AQUA_DEADMAN_S` | `1.0` | Manual drive stops if no command arrives within this time |
| `AQUA_YOLO_MODEL` | `yolov8n.pt` | Path to the YOLO weights |
| `AQUA_CAMERA_INDEX` | `0` | Camera device index |
| `AQUA_DETECT_CONF` | `0.4` | Detection confidence threshold |

## Tests

```bash
pip install -r requirements-dev.txt
python -m pytest -q
```

The tests cover geodesy maths, NMEA parsing (including a bad checksum and a no-fix sentence), navigation steering, the controller (dead-man timeout, obstacle handling, emergency stop, conveyor behaviour), the dashboard API, and a full simulated mission through the sample waypoints. Hardware test steps are in the design document.

## Safety

- Always keep a way to power the boat off by hand.
- Test on a bench, then in a small pool, before open water.
- Do not run Auto mode near people or swimmers.
- Wire the JSN-SR04T echo line through a voltage divider; the Pi GPIO is 3.3 V only.

## Contributing and licence

Issues and pull requests are welcome. Released under the [MIT License](LICENSE).
