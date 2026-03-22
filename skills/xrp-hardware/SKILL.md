---
name: xrp-hardware
description: >
  Physical hardware reference for MicroPython robotics platforms built on the
  RP2350 controller board. Use this skill whenever the user asks about GPIO pin
  assignments, wiring, physical port locations, motor or servo connectors, I2C
  bus configuration, power systems, battery voltage, expansion headers, sensor
  connections, or any question about what is physically connected to which pin.
  Also trigger for questions about the RP2350 processor, Qwiic connectors, solder
  jumpers, the DRV8411A motor driver, or the LSM6DSO IMU hardware.
---

# XRP Hardware — Pin Assignments & Physical Reference

## Processor

**RP2350** microcontroller with **Raspberry Pi Radio Module 2 (RM2)** providing WiFi and Bluetooth via SPI. MicroPython runs on the RP2350; the RM2 handles wireless communication.

---

## GPIO Pin Map

### Motor Ports

Each motor port has 2 PWM control pins + 2 encoder pins. Controlled by **DRV8411A** dual H-bridge drivers.

| Port | Motor Index | PWM Forward | PWM Backward | Encoder A | Encoder B | XRPLib Pin Names |
|------|-------------|-------------|--------------|-----------|-----------|------------------|
| Motor L | 1 (left) | GPIO 34 | GPIO 35 | GPIO 30 | GPIO 31 | `MOTOR_L_IN_1`, `MOTOR_L_IN_2`, `MOTOR_L_ENCODER_A`, `MOTOR_L_ENCODER_B` |
| Motor R | 2 (right) | GPIO 32 | GPIO 33 | GPIO 24 | GPIO 25 | `MOTOR_R_IN_1`, `MOTOR_R_IN_2`, `MOTOR_R_ENCODER_A`, `MOTOR_R_ENCODER_B` |
| Motor 3 | 3 | GPIO 20 | GPIO 21 | GPIO 22 | GPIO 23 | `MOTOR_3_IN_1`, `MOTOR_3_IN_2`, `MOTOR_3_ENCODER_A`, `MOTOR_3_ENCODER_B` |
| Motor 4 | 4 | GPIO 10 | GPIO 11 | GPIO 2 | GPIO 3 | `MOTOR_4_IN_1`, `MOTOR_4_IN_2`, `MOTOR_4_ENCODER_A`, `MOTOR_4_ENCODER_B` |

Motor L and Motor R are the primary drive motors for the standard XRP kit. Motor 3 and Motor 4 are expansion ports.

### Servo Ports

3-pin PWM connectors (signal, power, ground). Standard 50 Hz servo PWM.

| Port | Index | Signal GPIO | XRPLib Pin Name |
|------|-------|-------------|-----------------|
| Servo 1 | 1 | GPIO 6 | `SERVO_1` |
| Servo 2 | 2 | GPIO 9 | `SERVO_2` |
| Servo 3 | 3 | GPIO 7 | `SERVO_3` |
| Servo 4 | 4 | GPIO 8 | `SERVO_4` |

Servo 2 is used in the standard XRP kit (arm). Servo 3 and 4 availability depends on board revision (checked via `hasattr(Pin.board, "SERVO_3")`).

### Sensor Pins

| Function | GPIO | XRPLib Pin Name | Connector |
|----------|------|-----------------|-----------|
| Ultrasonic Trigger | GPIO 0 | `RANGE_TRIGGER` | DIST |
| Ultrasonic Echo | GPIO 1 | `RANGE_ECHO` | DIST |
| Line Follower Left | GPIO 44 | `LINE_L` | LINE (analog ADC) |
| Line Follower Right | GPIO 45 | `LINE_R` | LINE (analog ADC) |

### I2C Buses

| Bus | SCL GPIO | SDA GPIO | XRPLib Pin Names | Connector | Usage |
|-----|----------|----------|------------------|-----------|-------|
| I2C 0 | GPIO 5 | GPIO 4 | — | Qwiic 0 | Expansion (external sensors) |
| I2C 1 | GPIO 39 | GPIO 38 | `I2C_SCL_1`, `I2C_SDA_1` | Qwiic 1 | **IMU (LSM6DSO)** at 400 kHz |

The IMU is on **I2C bus 1**. I2C bus 0 is available for user-connected Qwiic/I2C devices.

### Board Control Pins

| Function | GPIO | XRPLib Pin Name | Description |
|----------|------|-----------------|-------------|
| User Button | GPIO 36 | `BOARD_USER_BUTTON` | Active low, internal pull-up |
| NeoPixel RGB LED | GPIO 37 | `BOARD_NEOPIXEL` | WS2812 single pixel |
| Onboard LED | — | `LED` | Standard status LED |
| Battery Voltage | — | `BOARD_VIN_MEASURE` | ADC input for motor power detection |

---

## IMU Hardware (LSM6DSO)

6-axis IMU (accelerometer + gyroscope) on I2C bus 1.

| Parameter | Value |
|-----------|-------|
| I2C Address | `0x6B` (default), `0x6A` (via solder jumper) |
| WHO_AM_I | `0x6C` |
| Accel ranges | ±2g, ±4g, ±8g, ±16g |
| Gyro ranges | ±125, ±250, ±500, ±1000, ±2000 dps |
| Sample rates | 0–6660 Hz (configurable) |
| Default conversion | 0.061 mg/LSB (2g), 4.375 mdps/LSB (125dps) |

---

## Encoder Hardware

Quadrature encoders on each motor, decoded via RP2 **PIO state machines** (hardware, not software).

| Parameter | Value |
|-----------|-------|
| Counts per motor shaft revolution | 12 |
| Gear ratio | 48.75 ((30/14)×(28/16)×(36/9)×(26/8)) |
| Counts per output shaft revolution | 585 |
| PIO state machine indices | 0–3 (one per motor) |

---

## Power System

| Rail | Voltage | Source | Powers | Indicator |
|------|---------|--------|--------|-----------|
| Motor | 5–11V | Barrel jack battery | DRV8411A motor drivers | MOT LED |
| System | 3.3V | Regulated from USB-C or barrel jack | RP2350, RM2, sensors, logic | SYS LED |

- **Power switch** controls barrel jack input only. USB-C bypasses the switch.
- **LOW VOLT LED** activates when supply drops below operating threshold.
- `board.are_motors_powered()` reads the `BOARD_VIN_MEASURE` ADC pin — returns `True` when ADC count > 20000 (battery connected and above threshold).
- Motors and servos will not function on USB power alone — the motor power rail requires the barrel jack battery.

---

## Motor Driver (DRV8411A)

Two DRV8411A dual H-bridge ICs. Each IC drives two motors.

| IC | Motors | Notes |
|----|--------|-------|
| IC 1 | Motor L, Motor R | Primary drive motors |
| IC 2 | Motor 3, Motor 4 | Expansion motors |

The RP2350 version uses **dual PWM** control (separate forward/backward PWM pins per motor). The RP2040 Beta used single PWM with a direction pin.

Solder jumpers `M-L CUR`, `M-R CUR`, `M-3 CUR`, `M-4 CUR` enable per-motor current sensing (disabled by default).

---

## Expansion Headers

Two 2×20 female headers expose RP2350 and RM2 GPIO pins. Inner rows replicate the **Raspberry Pi Pico 2 W** pinout (minus ADC pins). Outer rows contain RM2 pins and additional voltage rails.

This makes the board compatible with many Pico 2 W accessories and breakout boards.

---

## Qwiic / I2C Expansion

Two Qwiic connectors (SparkFun standard JST-SH 4-pin):

| Connector | I2C Bus | Available for user | Notes |
|-----------|---------|-------------------|-------|
| Qwiic 0 | I2C 0 (GPIO 4/5) | Yes | General expansion |
| Qwiic 1 | I2C 1 (GPIO 38/39) | Shared with IMU | IMU at 0x6B; other devices OK if different address |

Both buses have solder jumper pads for enabling/disabling pull-up resistors.

---

## Solder Jumpers Reference

| Jumper | Function | Default |
|--------|----------|---------|
| I2C0 | Enable/disable I2C0 pull-ups | Enabled |
| I2C1 | Enable/disable I2C1 pull-ups | Enabled |
| M-L/3 | Motor L/3 driver voltage reference | Connected |
| M-R/4 | Motor R/4 driver voltage reference | Connected |
| M-L/R/3/4 CUR | Per-motor current sense enable | Disabled |
| IMU ADDR | LSM6DSO address select (0x6B/0x6A) | 0x6B |
| VIN | Battery voltage measurement enable | Enabled |
| BYP | Fuse bypass | Open (fuse active) |
| SHLD | USB shield isolation | Connected |

---

## Standard XRP Kit Wheel Dimensions

| Parameter | Value | Used by |
|-----------|-------|---------|
| Wheel diameter | 6.0 cm | `DifferentialDrive` default |
| Wheel track (axle-to-axle) | 15.5 cm | `DifferentialDrive` default |

These are the defaults in `DifferentialDrive.__init__()`. Override in the constructor for custom wheel configurations.

---

## Platform Notes

- **Standard XRP and AgXRP use the same control board.** All pin assignments, power systems, and I2C buses are identical.
- The difference is in what's physically connected: standard XRP connects wheels, reflectance sensors, and a rangefinder; AgXRP connects pumps, soil moisture sensors, and temperature probes to the same motor and expansion ports.
- Motor port wiring is the same — the `EncodedMotor` class works identically whether driving a wheel or a pump.
- Servo ports and Qwiic connectors are available on both platforms for expansion.
