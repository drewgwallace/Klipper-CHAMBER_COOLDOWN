# 🔧 Klipper CHAMBER_COOLDOWN Macros

This set of Klipper macros enables a controlled **chamber cooldown sequence** at the end of a print. It monitors temperature sensors and manages exhaust fans to help streamline cooling your chamber.

⚠️ Note: These macros are designed as a reference implementation. They are not standalone and must be integrated into your existing printer configuration to function properly.
---

## 🪟 Overview

When a print ends or is canceled, the `CHAMBER_COOLDOWN` macro is triggered. It:
- Checks temperatures of the **chamber** and **bed**.
- Controls the **exhaust fan** speed.
- Loops until safe cooldown targets are reached or a time limit is hit.

You can configure target chamber temps, exhaust speeds, and custom delays.

---

## 🔧 Parameters (Used in `print_start`)

| Parameter      | Description                                                                 |
|----------------|-----------------------------------------------------------------------------|
| `CHAMBER`      | Target temperature (°C) for chamber preheat. <br> `0`: Off + full exhaust. <br> `1–10`: Passive cool only, no exhaust. <br> `11+`: Actively heat chamber. |
| `EXHAUST_END`  | Final exhaust fan speed during cooldown. Value between `0.0` and `1.0` (0–100%). |

> ⚠️ The `print_start` and `print_end` entries below are **only the relevant lines** you need to insert into your existing macros. Do not replace your full `print_start` or `print_end` definitions — instead, **merge these lines** into the appropriate place in your configuration.

---

## 🌐 Global Configuration

In your `printer.cfg`, define or update the following in a macro named `global_vars`:

```ini
[gcode_macro global_vars]
variable_max_chamber_cooldown_run_count: 1442     # Max loops (each = 5s, 1442 ≈ 2 hours)
variable_exhaust_off_chamber: 35                  # Chamber temp (°C) target to stop fan
variable_exhaust_off_bed: 40                      # Bed temp (°C) target to stop fan
variable_exhaust_delay: 120                       # Delay (s) before cooldown starts
gcode:
```

---

## ✅ Usage Steps

1. In your `print_start` macro:
   - Set the `CHAMBER` parameter to your desired target.
   - Set the `EXHAUST_END` parameter to control the exhaust fan during cooldown.

2. At the end of a print (`print_end` or `print_cancel`), the `CHAMBER_COOLDOWN` macro will:
   - Wait for `exhaust_delay` seconds.
   - Start looping every **5 seconds** to check:
     - Current bed and chamber temps.
     - Exhaust fan speed consistency.
   - Report chamber and bed temps **every 30 seconds** via `M118`.
   - Exit when **any** of the following conditions are met:
     - Chamber temp is below `exhaust_off_chamber`, **AND**
     - Bed temp is below `exhaust_off_bed`, **OR**
     - Exhaust fan speed has changed from the initially configured value, **OR**
     - Max loop count (`max_chamber_cooldown_run_count`) is reached.

---

## 🛑 Canceling the Cooldown Manually

You can stop the cooldown sequence manually by running:

```gcode
CHAMBER_RESET
```

Alternatively, modifying the exhaust fan speed will also interrupt the loop.

---

## ⚠️ Requirements

- Your **exhaust fan** must be named exactly: `exhaust_fan`.
- Your **chamber temperature** must be available through **either**:
  - A `temperature_sensor` named `Chamber`, **OR**
  - A `heater` (e.g. `heater_generic`) named `Chamber`.