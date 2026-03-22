---
name: xrp-sensors
description: >
  MicroPython sensor and control algorithm API reference for robotics platforms
  using IMU, Reflectance, Rangefinder, PID, and Timeout classes. Use this skill
  whenever the user asks about reading sensors, getting heading/pitch/roll/yaw,
  measuring distance, line following, reflectance values, gyroscope or
  accelerometer data, PID tuning, control loops, or timeout utilities. Also
  trigger when the user needs help with sensor calibration, obstacle avoidance,
  or any closed-loop control pattern that combines sensor input with motor output.
---

# XRP Sensors — Sensor & Control API Reference

## Standard Import

```python
from XRPLib.defaults import *
```

Exposes: `imu`, `rangefinder`, `reflectance`.

`PID`, `Controller`, and `Timeout` must be imported separately:

```python
from XRPLib.pid import PID
from XRPLib.controller import Controller
from XRPLib.timeout import Timeout
```

---

## IMU (LSM6DSO 6-DoF)

Accelerometer + gyroscope with integrated orientation tracking. Connected via I2C bus 1 at address `0x6B`.

### Factory

```python
IMU.get_default_imu() -> IMU
```

Auto-calibrates on first instantiation (1 second, Z-axis vertical).

### Constructor (custom config)

```python
IMU(scl_pin: int|str = "I2C_SCL_1", sda_pin: int|str = "I2C_SDA_1", addr: int = 0x6B)
```

I2C frequency: 400 kHz.

### Orientation Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `get_pitch()` | `float` | Pitch in degrees (unbounded, integrated from gyro). |
| `get_roll()` | `float` | Roll in degrees (unbounded). |
| `get_yaw()` | `float` | Yaw in degrees (unbounded). |
| `get_heading()` | `float` | Yaw bounded to [0, 360). |
| `set_pitch(pitch: float)` | `None` | Override current pitch value. |
| `set_roll(roll: float)` | `None` | Override current roll value. |
| `set_yaw(yaw: float)` | `None` | Override current yaw value. |
| `reset_pitch()` | `None` | Zero pitch. |
| `reset_roll()` | `None` | Zero roll. |
| `reset_yaw()` | `None` | Zero yaw. |

Orientation values are **integrated from gyro rates** — they accumulate drift over time. `get_heading()` wraps yaw to [0, 360) but the underlying value still drifts.

### Accelerometer Methods

| Method | Returns | Unit | Description |
|--------|---------|------|-------------|
| `get_acc_x()` | `int` | mg | X-axis acceleration. |
| `get_acc_y()` | `int` | mg | Y-axis acceleration. |
| `get_acc_z()` | `int` | mg | Z-axis acceleration. |
| `get_acc_rates()` | `list[int, int, int]` | mg | [x, y, z] acceleration. |

### Gyroscope Methods

| Method | Returns | Unit | Description |
|--------|---------|------|-------------|
| `get_gyro_x_rate()` | `int` | mdps | X-axis angular rate (millidegrees/sec). |
| `get_gyro_y_rate()` | `int` | mdps | Y-axis angular rate. |
| `get_gyro_z_rate()` | `int` | mdps | Z-axis angular rate. |
| `get_gyro_rates()` | `list[int, int, int]` | mdps | [x, y, z] angular rates. |
| `get_acc_gyro_rates()` | `list[list, list]` | mg, mdps | [[acc_x, acc_y, acc_z], [gyro_x, gyro_y, gyro_z]]. |

### Configuration Methods

**Accelerometer scale** — `acc_scale(value: str | None = None) -> str`

Valid values: `'2g'`, `'4g'`, `'8g'`, `'16g'`. Pass `None` to read current. Default: `'2g'`.

**Gyroscope scale** — `gyro_scale(value: str | None = None) -> str`

Valid values: `'125dps'`, `'250dps'`, `'500dps'`, `'1000dps'`, `'2000dps'`. Pass `None` to read current. Default: `'250dps'`.

**Accelerometer rate** — `acc_rate(value: str | None = None) -> str`

Valid values: `'0Hz'`, `'12.5Hz'`, `'26Hz'`, `'52Hz'`, `'104Hz'`, `'208Hz'`, `'416Hz'`, `'833Hz'`, `'1660Hz'`, `'3330Hz'`, `'6660Hz'`. Default: `'104Hz'`.

**Gyroscope rate** — `gyro_rate(value: str | None = None) -> str`

Same valid values as `acc_rate`. Default: `'104Hz'`.

### Utility Methods

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `calibrate` | `(calibration_time: float = 1, vertical_axis: int = 2)` | `None` | Samples offsets while stationary. `vertical_axis`: 0=X, 1=Y, 2=Z points up. |
| `is_connected` | `()` | `bool` | Checks WHO_AM_I register (expects `0x6C`). |
| `reset` | `(wait_for_reset: bool = True, wait_timeout_ms: int = 100)` | `bool` | Resets to factory defaults. Returns `False` on timeout. |
| `temperature` | `()` | `float` | Sensor die temperature in Celsius. |

---

## Reflectance

Dual analog line-following sensors. Return 0.0 (white/reflective surface) to 1.0 (black/absorbing surface).

### Factory

```python
Reflectance.get_default_reflectance() -> Reflectance
```

### Constructor

```python
Reflectance(leftPin: int|str = "LINE_L", rightPin: int|str = "LINE_R")
```

### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `get_left()` | `float` | Left sensor, 0.0–1.0. |
| `get_right()` | `float` | Right sensor, 0.0–1.0. |

ADC conversion: `raw_adc / 65536`.

---

## Rangefinder (HC-SR04)

Ultrasonic distance sensor. Trigger/echo pulse timing.

### Factory

```python
Rangefinder.get_default_rangefinder() -> Rangefinder
```

### Constructor

```python
Rangefinder(
    trigger_pin: int|str = "RANGE_TRIGGER",
    echo_pin: int|str = "RANGE_ECHO",
    timeout_us: int = 30000   # 500 * 2 * 30
)
```

### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `distance()` | `float` | Distance in cm. Returns `65535` on timeout (no echo). |

Effective range: 2–400 cm. Results cached for 3000 us to prevent re-triggering too fast.

---

## PID Controller

Concrete PID implementation inheriting from abstract `Controller`.

### Constructor

```python
PID(
    kp: float = 1.0,
    ki: float = 0.0,
    kd: float = 0.0,
    min_output: float = 0.0,
    max_output: float = 1.0,
    max_derivative: float | None = None,  # rate limiter
    max_integral: float | None = None,    # windup cap
    tolerance: float = 0.1,
    tolerance_count: int = 1
)
```

### Methods

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `update` | `(error: float, debug: bool = False)` | `float` | Computes PID output. `debug=True` prints P/I/D terms. |
| `is_done` | `()` | `bool` | `True` if error within `tolerance` for `tolerance_count` consecutive calls. |
| `clear_history` | `()` | `None` | Resets integral sum, derivative state, and done counter. |

Output is clamped to [`min_output`, `max_output`]. Time delta is computed internally using `time.ticks_ms()`.

### Controller (Abstract Base)

```python
class Controller:
    def update(self, input: float) -> float: ...
    def is_done(self) -> bool: ...
    def clear_history(self) -> None: ...
```

Any class implementing this interface can be passed to `EncodedMotor.set_speed_controller()` or `DifferentialDrive.straight()/turn()` as `main_controller`/`secondary_controller`.

---

## Timeout

Simple timer utility for loop control.

### Constructor

```python
Timeout(timeout: float)  # seconds
```

### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `is_done()` | `bool` | `True` if `timeout` seconds have elapsed since construction. Always returns `True` if timeout was `None`. |

Uses `time.ticks_ms()` internally.

---

## Code Examples

### 1. Line following with PID

```python
from XRPLib.defaults import *
from XRPLib.pid import PID

base_effort = 0.4
pid = PID(kp=0.6, ki=0.0, kd=0.1, min_output=-0.4, max_output=0.4)

board.wait_for_button()
while True:
    error = reflectance.get_left() - reflectance.get_right()
    correction = pid.update(error)
    drivetrain.set_effort(base_effort + correction, base_effort - correction)
```

### 2. Obstacle avoidance

```python
from XRPLib.defaults import *

board.wait_for_button()
while True:
    dist = rangefinder.distance()
    if dist < 15:  # cm
        drivetrain.stop()
        drivetrain.turn(90)
    else:
        drivetrain.set_effort(0.4, 0.4)
```

### 3. IMU heading hold

```python
from XRPLib.defaults import *
from XRPLib.pid import PID

target_heading = 0.0
pid = PID(kp=0.02, ki=0.001, kd=0.005, min_output=-0.5, max_output=0.5)

imu.reset_yaw()
board.wait_for_button()
while True:
    error = target_heading - imu.get_yaw()
    correction = pid.update(error)
    drivetrain.set_effort(0.3 + correction, 0.3 - correction)
```

### 4. Timed sensor logging

```python
from XRPLib.defaults import *
from XRPLib.timeout import Timeout
import time

timer = Timeout(10)  # 10 seconds
while not timer.is_done():
    print(f"Heading: {imu.get_heading():.1f}, Range: {rangefinder.distance():.1f} cm")
    time.sleep(0.1)
```

### 5. Custom PID for drivetrain.straight()

```python
from XRPLib.defaults import *
from XRPLib.pid import PID

# Tighter distance control with faster settling
distance_pid = PID(kp=0.1, ki=0.01, kd=0.05, tolerance=0.5, tolerance_count=3)
drivetrain.straight(50, max_effort=0.6, main_controller=distance_pid)
```

---

## Platform Notes

- **Standard XRP**: All sensors in this skill (`imu`, `rangefinder`, `reflectance`) are part of the standard XRP kit and are accessed through `XRPLib.defaults`.
- **AgXRP**: Uses different sensors (soil moisture, temperature probes, etc.) that are **not part of XRPLib**. The IMU may still be present on the shared control board, but `Reflectance` and `Rangefinder` are typically not connected on AgXRP builds. The `PID`, `Controller`, and `Timeout` classes are platform-independent and work on both.
