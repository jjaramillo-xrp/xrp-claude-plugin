# xrp-plugin

A Claude Code plugin providing comprehensive MicroPython API reference and development skills for the [XRP (eXtensible Robotics Platform)](https://xrpusers.org) robot.

## Overview

This plugin gives Claude structured knowledge about XRP robotics development, covering motor control, sensors, board management, hardware pin mappings, and the `mpremote` development workflow. It supports both the standard wheeled XRP and the agricultural **AgXRP** variant.

## Skills

| Skill | Description |
|---|---|
| `xrp-board` | Board class, LED/button control, WiFi webserver, Bluetooth gamepad input |
| `xrp-drive` | Motors, encoders, differential drive, servos, motor groups |
| `xrp-hardware` | GPIO pin assignments, I2C buses, power system, encoder specs, physical dimensions |
| `xrp-mpremote` | USB development workflow — REPL, file transfer, mounting, diagnostics |
| `xrp-sensors` | IMU (LSM6DSO), rangefinder, reflectance sensors, PID controller |

## Installation

Register the plugin with Claude Code by adding the plugin directory to your configuration, or install it via the Claude Code plugin registry if available.

Once installed, invoke skills by name in any Claude Code conversation:

```
/xrp-drive
/xrp-sensors
```

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- An XRP robot running [XRPLib](https://github.com/Open-STEM/XRPLib) MicroPython firmware
- [`mpremote`](https://docs.micropython.org/en/latest/reference/mpremote.html) for board communication (`pip install mpremote`)

## XRP Hardware Support

- **Standard XRP**: Wheeled robot with two drive motors, IMU, ultrasonic rangefinder, and dual line-following sensors
- **AgXRP**: Agricultural variant with pump motors, actuators, and agricultural sensors

## Plugin Structure

```
xrp-plugin/
├── .claude-plugin/
│   └── plugin.json        # Plugin manifest
└── skills/
    ├── xrp-board/SKILL.md
    ├── xrp-drive/SKILL.md
    ├── xrp-hardware/SKILL.md
    ├── xrp-mpremote/SKILL.md
    └── xrp-sensors/SKILL.md
```

## License

See repository for license details.
