# Autonomous Line-Following Car (Arduino Uno)

A beginner-friendly autonomous mobile robot that follows a high-contrast track using infrared (IR) sensors. The repository includes Arduino firmware, wiring guide, tuning instructions, test procedures, and a project report template.

> **Hardware note:** The photos show an Arduino Uno and a two-wheel robot chassis with a blue custom PCB. The exact pinout and onboard driver circuitry are not readable from the photos. This project therefore documents a **standard, testable reference wiring** using an Arduino Uno, a dual H-bridge motor driver (L298N or TB6612FNG configured for the pin map below), and three digital IR line sensors. If your blue PCB already integrates the sensors or motor driver, do not connect a second driver blindly—identify its pin labels first and update `config.h`.

## Features

- Three-sensor line tracking (left, centre, right)
- Proportional–derivative (PD) steering
- Adjustable motor speed and steering strength
- Handles common black-line-on-white-floor and white-line-on-black-floor sensor outputs
- Brief line-loss recovery using the last known direction
- Serial diagnostics for sensor readings and steering output
- Separate configuration file for pins and tuning values

## 1. Bill of materials

| Part | Quantity | Notes |
|---|---:|---|
| Arduino Uno / compatible ATmega328P board | 1 | Shown in the photos |
| Two-wheel robot chassis with DC geared motors | 1 | Shown in the photos |
| Dual H-bridge motor driver | 1 | L298N module or equivalent; use the matching driver wiring |
| Digital IR line sensors | 3 | Left, centre, right; a 3-channel array also works |
| Battery pack suitable for the motors and driver | 1 | Check motor voltage/current requirements |
| Jumper wires and switch | As needed | Secure exposed wiring |
| USB cable | 1 | Upload firmware and view diagnostics |

**Important:** Do not power motors from the Arduino 5 V pin. Use a motor supply appropriate for the motors, connect the driver and Arduino grounds together, and observe the driver's voltage/current limits. Disconnect power before rewiring.

## 2. Reference wiring (L298N)

| Arduino pin | Connect to |
|---|---|
| D2 | Left IR sensor digital output |
| D3 | Centre IR sensor digital output |
| D4 | Right IR sensor digital output |
| D5 | ENA (left motor PWM; remove ENA jumper if present) |
| D7 | IN1 |
| D8 | IN2 |
| D6 | ENB (right motor PWM; remove ENB jumper if present) |
| D9 | IN3 |
| D10 | IN4 |
| GND | Sensor GND, driver GND, battery negative (common ground) |
| 5V | Sensor VCC only if sensors are rated for 5 V |

Motor A connects to the left motor output; Motor B connects to the right motor output. If forward motion is reversed on a side, swap that motor's wires or change the corresponding direction setting in `config.h`.

The L298N's motor supply (`+12V`/`Vs`, label varies) connects to the motor battery positive, **not** to Arduino 5 V. Follow the module's own regulator/jumper guidance. Avoid connecting two different 5 V sources together.

## 3. Install and upload

1. Install the Arduino IDE from the official Arduino website.
2. Open `firmware/AutonomousLineFollower/AutonomousLineFollower.ino`.
3. Keep `config.h` in the same sketch folder.
4. Connect the Uno by USB, choose **Tools → Board → Arduino Uno**, and select the correct port.
5. Upload the sketch with the wheels lifted off the floor.
6. Open **Tools → Serial Monitor**, set to **115200 baud**.
7. Place the robot on the track and check that the three sensor readings change when each sensor passes over the line.

## 4. Sensor setup

This firmware uses **digital** IR sensor outputs. Many modules provide `DO` plus a small potentiometer. Adjust each potentiometer so the output switches reliably between the track and floor.

- By default, `LINE_IS_LOW = true`: a sensor reading LOW means it sees the line.
- If your module reports HIGH on the line, change `LINE_IS_LOW` to `false` in `config.h`.
- Test one sensor at a time and verify left / centre / right order.
- Use a matte black tape line on a light, non-glossy floor for initial testing.

If your board has analog-only sensors, this exact digital sketch needs adapting to use `analogRead()` and a calibrated threshold.

## 5. Tuning

Start slowly. In `config.h`:

- `BASE_SPEED`: overall forward speed (start around 100–130; range 0–255)
- `MAX_SPEED`: output ceiling (usually 170–210 for first tests)
- `KP`: steering response; increase if turns are too weak
- `KD`: damping; increase a little if it oscillates
- `LOST_LINE_LIMIT_MS`: how long to search using the last known direction before stopping

Recommended tuning sequence:
1. Start with `BASE_SPEED = 100`, `KP = 32`, `KD = 18`.
2. Run on a straight line.
3. Increase `KP` gradually until it follows bends but begins to wobble.
4. Add/increase `KD` to reduce wobble.
5. Increase speed in small steps only after it tracks reliably.

The PD controller is intentionally lightweight for an Uno. Three digital sensors give coarse position estimates, so sharp curves and intersections may need additional sensors or custom intersection logic.

## 6. Safety and testing

- First test with the robot raised so wheels cannot drive it off a table.
- Keep fingers, hair, loose clothing, and wires away from gears and wheels.
- Use a suitable protected battery pack; do not short the battery.
- Do not operate near stairs, roads, people, pets, or fragile objects.
- Stop immediately if a motor, driver, battery, or wire becomes unusually hot.
- Check local lab rules before using rechargeable lithium batteries.

## 7. Troubleshooting

| Symptom | Likely cause | What to check |
|---|---|---|
| Motors do not move | No motor supply, ENA/ENB jumper conflict, wrong wiring | Battery voltage, driver enable pins, common ground |
| Uno resets when motors start | Motor noise or supply sag | Separate motor and logic supplies; common ground; suitable decoupling |
| Robot moves backward | Motor polarity or direction mapping | Swap motor leads or change direction configuration |
| Always turns one way | Sensor order or active level wrong | Read Serial Monitor; verify L/C/R and `LINE_IS_LOW` |
| Sensors never change | Wrong sensor output pin or bad threshold | Sensor power, output pin, potentiometer |
| Wobbles on straight track | Excessive KP / speed, insufficient KD | Lower base speed; tune KP then KD |
| Loses the line on curves | Speed too high or sensor spacing unsuitable | Slow down; adjust sensor placement; use more sensors |
| Follows the background instead of line | Active level reversed | Toggle `LINE_IS_LOW` |

## 8. Repository structure

```text
autonomous-line-follower/
├── README.md
├── LICENSE
├── .gitignore
├── firmware/
│   └── AutonomousLineFollower/
│       ├── AutonomousLineFollower.ino
│       └── config.h
├── docs/
│   ├── WIRING.md
│   ├── TEST_PLAN.md
│   └── PROJECT_REPORT_TEMPLATE.md
└── media/
    └── README.md
```

## 9. Suggested GitHub project description

**Autonomous Line-Following Robot using Arduino Uno and IR Sensors**

A low-cost autonomous mobile robot that detects a track with infrared sensors and uses PD control to steer two DC motors. Includes configurable firmware, wiring documentation, calibration guidance, test plan, and troubleshooting notes.

Suggested topics: `arduino`, `robotics`, `embedded-systems`, `autonomous-robot`, `line-follower`, `control-systems`, `electronics`.

## 10. Before calling it complete

A project is only verified on your actual hardware after you upload the firmware, confirm sensor polarity and motor direction, and run the test plan. The code is a complete reference implementation, but the blue PCB in the photos may have a different pinout. Confirm its labels before connecting power or assuming the reference wiring matches it.
