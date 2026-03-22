---
name: xrp-drive
description: >
  MicroPython motor and drivetrain API reference for robotics platforms using
  EncodedMotor, DifferentialDrive, MotorGroup, and Servo classes. Use this skill
  whenever the user asks about controlling motors, driving a robot, setting motor
  speed or effort, reading encoder positions, configuring servos, or writing any
  MicroPython code that moves actuators. Also trigger when the user mentions
  drivetrain, wheels, tank drive, arcade drive, motor groups, PWM motors, or
  encoder feedback. Covers both wheeled robots and pump/actuator applications.
---

# XRP Drive — Motor & Actuator API Reference

## Standard Import

```python
from XRPLib.defaults import *
```

Exposes: `left_motor`, `right_motor`, `motor_three`, `motor_four`, `drivetrain`, `servo_one`, `servo_two` (plus `servo_three`, `servo_four` on boards with those pins).

All default objects are **singletons** returned by factory classmethods. Never call constructors directly in normal use.

---

## EncodedMotor

DC motor + quadrature encoder with optional closed-loop speed control.

### Factory

```python
EncodedMotor.get_default_encoded_motor(index: int = 1) -> EncodedMotor
```

| Index | Default name   | Physical port |
|-------|---------------|---------------|
| 1     | `left_motor`  | Motor L       |
| 2     | `right_motor` | Motor R       |
| 3     | `motor_three` | Motor 3       |
| 4     | `motor_four`  | Motor 4       |

Automatically selects `DualPWMMotor` (RP2350) or `SinglePWMMotor` (RP2040 Beta) based on processor.

### Constants

```python
EncodedMotor.ZERO_EFFORT_BREAK = True   # Note: typo in source — means BRAKE
EncodedMotor.ZERO_EFFORT_COAST = False
```

### Methods

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `set_effort` | `(effort: float)` | `None` | Open-loop power, -1.0 to 1.0. Non-blocking. |
| `set_speed` | `(speed_rpm: float \| None = None)` | `None` | Activates passive closed-loop RPM control via internal 50 Hz timer. Pass `None` or `0` to disable. |
| `set_speed_controller` | `(new_controller: Controller)` | `None` | Replace default PID. Default: `PID(kp=0.035, ki=0.03, kd=0, max_integral=50)`. |
| `set_zero_effort_behavior` | `(brake_at_zero_effort: bool)` | `None` | `True` = brake at zero effort, `False` = coast. |
| `get_position` | `()` | `float` | Position in revolutions (accounts for `flip_dir`). |
| `get_position_counts` | `()` | `int` | Position in raw encoder counts. |
| `reset_encoder_position` | `()` | `None` | Zeros encoder. |
| `get_speed` | `()` | `float` | Current speed in RPM. |
| `brake` | `()` | `None` | Motor actively resists rotation. |
| `coast` | `()` | `None` | Motor spins freely. |

**Effort vs Speed**: `set_effort()` is fire-and-forget open-loop control. `set_speed()` starts a background PID loop (50 Hz timer) that continuously adjusts effort to maintain the target RPM. These are mutually exclusive — calling `set_effort()` while speed control is active will be overridden on the next timer tick.

---

## DifferentialDrive

Two-wheel drivetrain with optional IMU heading correction. Only for wheeled robots.

### Factory

```python
DifferentialDrive.get_default_differential_drive() -> DifferentialDrive
```

### Constructor (for custom configurations)

```python
DifferentialDrive(
    left_motor: EncodedMotor,
    right_motor: EncodedMotor,
    imu: IMU | None = None,
    wheel_diam: float = 6.0,    # cm
    wheel_track: float = 15.5   # cm
)
```

### Methods

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `set_effort` | `(left_effort: float, right_effort: float)` | `None` | Raw effort per motor, -1.0 to 1.0. Non-blocking. |
| `set_speed` | `(left_speed: float, right_speed: float)` | `None` | Speed per wheel in cm/s. |
| `arcade` | `(straight: float, turn: float)` | `None` | Arcade drive. `straight`: -1 to 1, `turn`: -1 to 1. Uses IMU for heading correction if available. |
| `straight` | `(distance: float, max_effort: float = 0.5, timeout: float \| None = None, main_controller: Controller \| None = None, secondary_controller: Controller \| None = None)` | `bool` | **Blocking.** Drives `distance` cm (positive=forward, negative=backward). Returns `True` if completed before timeout. |
| `turn` | `(turn_degrees: float, max_effort: float = 0.5, timeout: float \| None = None, main_controller: Controller \| None = None, secondary_controller: Controller \| None = None, use_imu: bool = True)` | `bool` | **Blocking.** Turns `turn_degrees` degrees. Uses IMU if available and `use_imu=True`, else uses encoder-based estimation. Returns `True` if completed before timeout. |
| `stop` | `()` | `None` | Stops both motors. |
| `set_zero_effort_behavior` | `(brake_at_zero_effort: bool)` | `None` | Applies to both motors. |
| `reset_encoder_position` | `()` | `None` | Resets both encoders. |
| `get_left_encoder_position` | `()` | `float` | Left wheel distance in cm. |
| `get_right_encoder_position` | `()` | `float` | Right wheel distance in cm. |

**`straight()` and `turn()` are blocking** — they run a control loop internally and do not return until the target is reached or timeout expires. Use `set_effort()` for non-blocking control.

---

## MotorGroup

Treats multiple `EncodedMotor` instances as a single unit. Inherits from `EncodedMotor`.

### Constructor

```python
MotorGroup(*motors: EncodedMotor)
```

### Methods

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `add_motor` | `(motor: EncodedMotor)` | `None` | Add motor to group. |
| `remove_motor` | `(motor: EncodedMotor)` | `None` | Remove motor from group. |
| `set_effort` | `(effort: float)` | `None` | Sets effort on all motors. |
| `get_position` | `()` | `float` | Average position in revolutions. |
| `get_position_counts` | `()` | `int` | Average position in counts (rounded). |
| `reset_encoder_position` | `()` | `None` | Resets all encoders. |
| `get_speed` | `()` | `float` | Average speed in RPM. |
| `set_speed` | `(target_speed_rpm: float \| None = None)` | `None` | Sets speed on all motors. |

---

## Servo

PWM servo control, 50 Hz.

### Factory

```python
Servo.get_default_servo(index: int) -> Servo
```

| Index | Default name   | Pin        |
|-------|---------------|------------|
| 1     | `servo_one`   | `SERVO_1`  |
| 2     | `servo_two`   | `SERVO_2`  |
| 3     | `servo_three` | `SERVO_3`  |
| 4     | `servo_four`  | `SERVO_4`  |

Indices 3–4 only available on boards with those pins (checked via `hasattr(Pin.board, "SERVO_3")`).

### Methods

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `set_angle` | `(degrees: float)` | `None` | Set servo position. Range: 0–200 degrees. |
| `free` | `()` | `None` | Disables PWM signal — servo stops holding position. |

---

## Encoder

Low-level quadrature decoder using RP2 PIO state machines. Normally accessed through `EncodedMotor`, not directly.

### Constructor

```python
Encoder(index: int, encAPin: int|str, encBPin: int|str)
```

`index`: PIO state machine index 0–3.

### Constants

```python
Encoder.resolution = 585.0  # counts per output shaft revolution
# Gear ratio: (30/14) * (28/16) * (36/9) * (26/8) = 48.75
# Counts per motor shaft revolution: 12
# 12 * 48.75 = 585
```

### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `get_position()` | `float` | Position in revolutions. |
| `get_position_counts()` | `int` | Position in raw counts (signed 32-bit). |
| `reset_encoder_position()` | `None` | Zeros position. |

---

## DualPWMMotor / SinglePWMMotor

Low-level motor drivers. Normally accessed through `EncodedMotor`, not directly.

**DualPWMMotor** (RP2350 — official XRP):
```python
DualPWMMotor(in1_pwm_forward: int|str, in2_pwm_backward: int|str, flip_dir: bool = False)
```

**SinglePWMMotor** (RP2040 — XRP Beta):
```python
SinglePWMMotor(in1_direction_pin: int|str, in2_speed_pin: int|str, flip_dir: bool = False)
```

Both expose: `set_effort(effort: float)`, `brake()`, `coast()`.

---

## Code Examples

### 1. Drive forward 30 cm and turn 90 degrees

```python
from XRPLib.defaults import *

board.wait_for_button()
drivetrain.straight(30, max_effort=0.5)
drivetrain.turn(90, max_effort=0.4)
```

### 2. Non-blocking tank drive with effort

```python
from XRPLib.defaults import *
import time

board.wait_for_button()
drivetrain.set_effort(0.5, 0.5)  # forward
time.sleep(2)
drivetrain.set_effort(0.5, -0.5)  # spin right
time.sleep(1)
drivetrain.stop()
```

### 3. Closed-loop speed control on a single motor

```python
from XRPLib.defaults import *
import time

left_motor.set_speed(60)  # 60 RPM
time.sleep(5)
left_motor.set_speed()    # disable speed control
left_motor.coast()
```

### 4. Motor group for synchronized pump control

```python
from XRPLib.defaults import *
from XRPLib.motor_group import MotorGroup

pump_group = MotorGroup(motor_three, motor_four)
pump_group.set_effort(0.7)   # both pumps at 70%
# ... later
pump_group.set_effort(0)
```

### 5. Servo sweep

```python
from XRPLib.defaults import *
import time

for angle in range(0, 180, 10):
    servo_one.set_angle(angle)
    time.sleep(0.1)
servo_one.free()
```

---

## Platform Notes

- **Standard XRP**: Uses `DifferentialDrive` for wheeled navigation. `left_motor` and `right_motor` are the drive motors; `motor_three` and `motor_four` are expansion.
- **AgXRP**: Uses `EncodedMotor` directly to drive pumps and actuators. **Do not use `DifferentialDrive` for AgXRP** — it assumes a two-wheeled robot with distance/heading semantics that do not apply to pumps. Use individual `EncodedMotor` instances or `MotorGroup` instead.
- Both platforms share the same `EncodedMotor`, `Servo`, and `MotorGroup` classes. Only `DifferentialDrive` is XRP-specific.
