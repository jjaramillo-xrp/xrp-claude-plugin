---
name: xrp-board
description: >
  MicroPython Board, Webserver, and system-level API reference for robotics
  platforms. Use this skill whenever the user asks about the board LED, RGB LED,
  NeoPixel, user button, motor power detection, WiFi access point, web-based
  remote control, webserver buttons, data logging to the web UI, the defaults
  import pattern, singleton pattern, or how to structure an XRP MicroPython
  program. Also trigger when the user asks about button-to-start patterns,
  network configuration, secrets.json, or the Gamepad Bluetooth controller.
---

# XRP Board — Board, Webserver & System API Reference

## Standard Import

```python
from XRPLib.defaults import *
```

This single import exposes all pre-configured singleton instances:

| Name | Class | Description |
|------|-------|-------------|
| `left_motor` | `EncodedMotor` | Left drive motor (index 1) |
| `right_motor` | `EncodedMotor` | Right drive motor (index 2) |
| `motor_three` | `EncodedMotor` | Expansion motor (index 3) |
| `motor_four` | `EncodedMotor` | Expansion motor (index 4) |
| `drivetrain` | `DifferentialDrive` | Two-wheel drivetrain |
| `imu` | `IMU` | 6-DoF inertial sensor |
| `rangefinder` | `Rangefinder` | Ultrasonic distance sensor |
| `reflectance` | `Reflectance` | Dual line sensors |
| `servo_one` | `Servo` | Servo port 1 |
| `servo_two` | `Servo` | Servo port 2 |
| `servo_three` | `Servo` | Servo port 3 (if pin exists on board) |
| `servo_four` | `Servo` | Servo port 4 (if pin exists on board) |
| `webserver` | `Webserver` | HTTP control interface |
| `board` | `Board` | LED, button, power sensing |

`servo_three` and `servo_four` are conditionally created — only if the board definition includes `SERVO_3`/`SERVO_4` pins.

## Singleton Pattern

Every default object is a **singleton** returned by a factory classmethod (e.g., `Board.get_default_board()`). Calling the factory method multiple times returns the same instance. The `defaults` import simply calls these factories and assigns the results to module-level names.

**Never instantiate classes directly** in normal use. This avoids duplicate hardware initialization and conflicting pin assignments.

```python
# CORRECT
from XRPLib.defaults import *
board.led_on()

# CORRECT (same result, explicit factory)
from XRPLib.board import Board
b = Board.get_default_board()
b.led_on()

# WRONG — creates a second Board instance with conflicting pins
from XRPLib.board import Board
b = Board()  # Don't do this
```

---

## Board

Power sensing, user button, and LED control.

### Factory

```python
Board.get_default_board() -> Board
```

### Constructor

```python
Board(
    vin_pin: str = "BOARD_VIN_MEASURE",
    button_pin: str = "BOARD_USER_BUTTON",
    rgb_led_pin: str = "BOARD_NEOPIXEL",
    led_pin: str = "LED"
)
```

### Methods

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `are_motors_powered` | `()` | `bool` | `True` if battery voltage ADC > 20000 counts. Use as safety check before running motors. |
| `is_button_pressed` | `()` | `bool` | Current button state (button is PULL_UP, active low). |
| `wait_for_button` | `()` | `None` | **Blocking.** Waits for button press then release. Standard "press to start" pattern. |
| `led_on` | `()` | `None` | Turns LED on. Stops blinking timer if active. |
| `led_off` | `()` | `None` | Turns LED off. Stops blinking timer if active. |
| `led_blink` | `(frequency: int = 0)` | `None` | Blink at `frequency` Hz. Pass 0 to stop. Uses a virtual `Timer`. |
| `set_rgb_led` | `(r: int, g: int, b: int)` | `None` | Set NeoPixel RGB color. **Raises `NotImplementedError` on XRP Beta** (no NeoPixel hardware). |

---

## Webserver

HTTP-based remote control interface. Runs on the XRP's WiFi radio (RM2 module). Supports access point mode (robot creates its own network) or bridge mode (robot joins an existing network).

### Factory

```python
Webserver.get_default_webserver() -> Webserver
```

### Network Setup

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `start_network` | `(ssid: str \| None = None, robot_id: int \| None = None, password: str \| None = None)` | `None` | **Access point mode.** Creates a WiFi network. If params are `None`, reads from `secrets.json`. `{robot_id}` in SSID is replaced with the integer. |
| `connect_to_network` | `(ssid: str \| None = None, password: str \| None = None, timeout: int = 10)` | `bool` | **Bridge mode.** Joins existing WiFi. Returns `True` on success. Reads `secrets.json` if params are `None`. |
| `start_server` | `()` | `None` | Starts HTTP server + DNS catchall. **Must call `start_network` or `connect_to_network` first.** |
| `stop_server` | `()` | `None` | Shuts down server and network. |

**`secrets.json`** format (stored on board filesystem):
```json
{
    "ssid": "XRP-{robot_id}",
    "robot_id": 1,
    "password": "remote1234"
}
```

### Button Registration

These methods register callables for the directional arrow buttons on the web UI. Calling any of them enables arrow display on the page.

| Method | Signature | Description |
|--------|-----------|-------------|
| `registerForwardButton` | `(function)` | Callback for forward arrow. |
| `registerBackwardButton` | `(function)` | Callback for backward arrow. |
| `registerLeftButton` | `(function)` | Callback for left arrow. |
| `registerRightButton` | `(function)` | Callback for right arrow. |
| `registerStopButton` | `(function)` | Callback for stop button. |
| `add_button` | `(button_name: str, function)` | Add a custom named button. |

### Data Logging

```python
webserver.log_data(label: str, data) -> None
```

Registers a key-value pair displayed on the web UI. `data` is converted to string. Updated values appear on the next page refresh/poll.

---

## Gamepad (Bluetooth)

Bluetooth gamepad input via the RM2 radio module. Not included in `defaults` — import separately.

### Factory

```python
from XRPLib.gamepad import Gamepad
gp = Gamepad.get_default_gamepad()
```

### Axis Constants

| Constant | Index | Range |
|----------|-------|-------|
| `X1` | 0 | -1.0 to 1.0 (left stick X) |
| `Y1` | 1 | -1.0 to 1.0 (left stick Y) |
| `X2` | 2 | -1.0 to 1.0 (right stick X) |
| `Y2` | 3 | -1.0 to 1.0 (right stick Y) |

### Button Constants

`BUTTON_A` (4), `BUTTON_B` (5), `BUTTON_X` (6), `BUTTON_Y` (7), `BUMPER_L` (8), `BUMPER_R` (9), `TRIGGER_L` (10), `TRIGGER_R` (11), `BACK` (12), `START` (13), `DPAD_UP` (14), `DPAD_DN` (15), `DPAD_L` (16), `DPAD_R` (17).

### Methods

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `start` | `()` | `None` | Signal remote to begin sending data. |
| `stop` | `()` | `None` | Signal remote to stop sending data. |
| `get_value` | `(index: int)` | `float` | Axis value, -1.0 to 1.0. |
| `is_button_pressed` | `(index: int)` | `bool` | Button state. |

---

## resetbot Module

Emergency reset utilities. Importing `XRPLib.resetbot` auto-resets any already-loaded modules.

```python
from XRPLib.resetbot import reset_hard  # resets everything
from XRPLib.resetbot import reset_motors, reset_led, reset_servos, reset_webserver, reset_gamepad
```

| Function | Description |
|----------|-------------|
| `reset_motors()` | Stops all 4 motors, resets encoders. |
| `reset_led()` | Turns off LED and RGB LED. |
| `reset_servos()` | Frees both servos. |
| `reset_webserver()` | Stops server and network. |
| `reset_gamepad()` | Stops gamepad data. |
| `reset_hard()` | Calls all of the above. |

---

## Code Examples

### 1. Press-button-to-start pattern

```python
from XRPLib.defaults import *

board.led_blink(2)         # blink at 2 Hz while waiting
board.wait_for_button()    # blocks until press+release
board.led_on()             # solid LED = running
drivetrain.straight(30)
board.led_off()
```

### 2. Safety check before motor use

```python
from XRPLib.defaults import *

if not board.are_motors_powered():
    print("Connect battery!")
    board.led_blink(5)  # fast blink = error
    while not board.are_motors_powered():
        pass
board.led_on()
drivetrain.set_effort(0.5, 0.5)
```

### 3. WiFi remote control (access point)

```python
from XRPLib.defaults import *

def forward(): drivetrain.set_effort(0.5, 0.5)
def backward(): drivetrain.set_effort(-0.5, -0.5)
def left(): drivetrain.set_effort(-0.3, 0.3)
def right(): drivetrain.set_effort(0.3, -0.3)
def stop(): drivetrain.stop()

webserver.registerForwardButton(forward)
webserver.registerBackwardButton(backward)
webserver.registerLeftButton(left)
webserver.registerRightButton(right)
webserver.registerStopButton(stop)

webserver.start_network(ssid="MyXRP", password="12345678")
webserver.start_server()
```

### 4. Data logging to web UI

```python
from XRPLib.defaults import *
import time

webserver.start_network()
webserver.start_server()

while True:
    webserver.log_data("Heading", f"{imu.get_heading():.1f}")
    webserver.log_data("Range", f"{rangefinder.distance():.1f} cm")
    webserver.log_data("Battery", "OK" if board.are_motors_powered() else "LOW")
    time.sleep(0.5)
```

### 5. Gamepad tank drive

```python
from XRPLib.defaults import *
from XRPLib.gamepad import Gamepad

gp = Gamepad.get_default_gamepad()
gp.start()

while True:
    left = gp.get_value(Gamepad.Y1)
    right = gp.get_value(Gamepad.Y2)
    drivetrain.set_effort(left, right)
```

---

## Platform Notes

- **Board** and **Webserver** are identical between standard XRP and AgXRP — same control board, same WiFi radio, same LED/button/power hardware.
- The `Gamepad` class works on both platforms.
- The `resetbot` module works on both platforms.
- The `defaults` import creates a `DifferentialDrive` instance even on AgXRP — it just won't be useful for pump-based applications. Importing `defaults` is still the correct pattern for accessing motors, servos, and board on AgXRP.
