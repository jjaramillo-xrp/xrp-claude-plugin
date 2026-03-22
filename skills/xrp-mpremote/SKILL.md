---
name: xrp-mpremote
description: >
  mpremote CLI reference for MicroPython robotics development over USB. Use this
  skill whenever the user asks about uploading code to the board, running scripts
  on a MicroPython device, using the REPL, copying files to/from the board,
  mounting a local directory, checking firmware version, diagnosing board
  connectivity, or any mpremote command. Also trigger when the user asks about
  MicroPython development workflow, deploying code to the XRP or AgXRP, or
  debugging a board over USB serial.
---

# XRP mpremote — USB Development Workflow Reference

## Installation

```bash
pip install mpremote
```

Or with pipx: `pipx install mpremote`.

---

## Connection

`mpremote` auto-connects to the first available USB serial device. No configuration needed for single-board setups.

```bash
mpremote                        # auto-connect + REPL
mpremote connect list           # list available devices
mpremote connect port:/dev/ttyACM0 repl   # explicit port
```

Shortcuts: `a0`–`a3` for `/dev/ttyACM0`–`3` (Linux), `c0`–`c3` for `COM0`–`3` (Windows).

---

## Core Commands

### REPL — Interactive Console

```bash
mpremote repl
```

- `Ctrl-C` — interrupt running program
- `Ctrl-D` — soft reset (restart interpreter)
- `Ctrl-]` or `Ctrl-x` — exit REPL

Options: `--capture <file>` saves session, `--inject-code <string>` auto-runs on Ctrl-J, `--inject-file <file>` auto-runs on Ctrl-K.

### run — Execute Local File on Board (from RAM)

```bash
mpremote run script.py
```

Sends the local file to the board and executes it **in RAM** without copying to the board filesystem. Preferred for rapid iteration — no wear on flash storage.

`--no-follow` returns immediately without waiting for output.

### exec — Execute Code String

```bash
mpremote exec "<code>"
```

Runs a Python code string on the board. Useful for one-liners and diagnostics.

`--no-follow` for background execution.

### eval — Evaluate Expression

```bash
mpremote eval "<expression>"
```

Evaluates and prints the result of a single Python expression.

---

## Filesystem Commands

All filesystem commands use `:` prefix for board paths. Local paths have no prefix.

```bash
mpremote fs ls                        # list board root
mpremote fs ls :/lib                  # list /lib on board
mpremote fs cp main.py :main.py      # copy local → board
mpremote fs cp :main.py ./backup.py  # copy board → local
mpremote fs cp -r ./lib :lib         # recursive copy
mpremote fs rm :old_file.py          # delete from board
mpremote fs mkdir :data              # create directory on board
mpremote fs cat :boot.py             # print file contents
mpremote fs tree :                   # directory tree
mpremote df                          # filesystem space
```

Shorthand aliases work without the `fs` prefix: `mpremote cp`, `mpremote ls`, etc.

---

## mount — Live Local Directory

```bash
mpremote mount .
```

Mounts the current local directory as `/remote` on the board. The board sees and imports local files directly — **no copy step needed**. Best for active development.

- Changes to local files are immediately visible to the board
- Automatically enters REPL after mounting (unless followed by another command)
- `--unsafe-links` (`-l`) follows symlinks outside the mount directory

### mount + run pattern

```bash
mpremote mount . exec "import my_script"
```

---

## Auto-Reset Behavior

**Critical to understand:** `run`, `exec`, `eval`, and `fs` commands trigger an automatic **soft reset** before executing. This clears the Python heap, resets all variables, and re-runs `boot.py`.

Use `resume` to suppress the auto-reset:

```bash
mpremote resume exec "<code>"
mpremote resume run script.py
```

This is important when you need to inspect state from a previously running program, or when you want to run multiple commands without resetting between them.

### Command chaining

Multiple commands run sequentially with `+` separator:

```bash
mpremote exec "import setup" + run main.py
```

---

## Package Installation

```bash
mpremote mip install <package>            # from micropython-lib
mpremote mip install github:org/repo      # from GitHub
mpremote mip install --target /lib <pkg>  # custom install path
```

---

## Device Management

```bash
mpremote reset          # hard reset (machine.reset())
mpremote soft-reset     # clear heap, restart interpreter
mpremote bootloader     # enter UF2 bootloader mode
mpremote rtc --set      # sync board RTC to host clock
```

---

## XRP Diagnostic One-Liners

### Check firmware version

```bash
mpremote exec "import sys; print(sys.version)"
```

### Check motor power

```bash
mpremote exec "from XRPLib.defaults import *; print(board.are_motors_powered())"
```

### Read IMU heading

```bash
mpremote exec "from XRPLib.defaults import *; print(imu.get_heading())"
```

### Read all sensors

```bash
mpremote exec "from XRPLib.defaults import *; print(f'Heading: {imu.get_heading():.1f}, Range: {rangefinder.distance():.1f}, Left: {reflectance.get_left():.2f}, Right: {reflectance.get_right():.2f}')"
```

### Test a single motor

```bash
mpremote exec "from XRPLib.defaults import *; left_motor.set_effort(0.3); import time; time.sleep(2); left_motor.set_effort(0)"
```

### List files on board

```bash
mpremote fs ls :/
mpremote fs ls :/lib
```

### Check board identity

```bash
mpremote exec "import sys; print(sys.implementation._machine)"
```

---

## Recommended Development Workflows

### Active development (fastest iteration)

```bash
mpremote mount .
```

Then in the REPL: `import my_script` to run. Edit locally, `Ctrl-D` to soft reset, re-import. No file copying needed.

### Script testing (no permanent changes)

```bash
mpremote run my_script.py
```

Executes from RAM. Nothing written to board flash. Good for testing before deployment.

### Deployment (permanent)

```bash
mpremote fs cp main.py :main.py
mpremote fs cp -r lib :lib
mpremote reset
```

Copies files to board filesystem. They persist across resets and power cycles. The board runs `main.py` automatically on boot.

### Multi-file project deployment

```bash
mpremote fs cp main.py :main.py + fs cp -r XRPLib :XRPLib + reset
```

Chain commands with `+` to avoid repeated soft resets between operations.

---

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| `no device found` | Board not connected or in bootloader mode | Check USB cable, replug, check `mpremote connect list` |
| `OSError: [Errno 16] EBUSY` | Another program (Thonny, serial monitor) has the port open | Close the other program |
| Import errors after `mount` | Board can't find modules in mounted dir | Ensure you're mounting the correct directory; check `sys.path` |
| Script works with `run` but fails after `fs cp` | Different working directory or missing dependencies | Ensure all dependencies are also copied to the board |
| `KeyboardInterrupt` on connect | Previous script still running | `Ctrl-C` to interrupt, then `Ctrl-D` to soft reset |

---

## Platform Notes

- **mpremote workflow is identical for XRP and AgXRP** — same RP2350 board, same USB serial interface, same MicroPython firmware.
- The diagnostic one-liners above use `XRPLib.defaults` which works on both platforms (though sensor readings like reflectance/rangefinder may not be meaningful on AgXRP if those sensors aren't connected).
- File deployment paths and boot behavior are the same on both platforms.
