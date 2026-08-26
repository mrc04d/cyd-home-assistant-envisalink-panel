# CYD Alarm Panel for Home Assistant

A modern, touchscreen alarm panel for Home Assistant using cheap ESP32 displays.

## Overview

This project turns an inexpensive display (~$15–$30) into a fully functional alarm panel for Home Assistant. It features a modern dark UI, PIN code entry, grace-period cancel, and seamless integration with Home Assistant alarm control panel entities.

## Architecture

The configuration uses ESPHome's `packages` feature to share logic between the two supported panels:

- **`common/alarm-core.yaml`** — All shared logic: WiFi, API, OTA, time, globals, sensors, scripts, intervals, and the LVGL event handlers. Edit logic once here.
- **Device shells** (`cyd-alarm-panel.yaml`, `ESP32-4848S040C_I/cyd-alarm-panel-4-inch.yaml`) — Thin hardware + LVGL layout files that import the core.

### Widget-ID Contract

Every device layout **MUST** define LVGL widgets with these IDs:

| ID | Purpose |
|----|---------|
| `time_label` | Clock display |
| `wifi_bar1..4` | WiFi signal bars |
| `status_card` | Status card background |
| `status_label` | Status text |
| `stay_btn` | STAY arm button |
| `away_btn` | AWAY arm button |
| `cancel_btn` | CANCEL button |
| `disarm_btn` | DISARM button |
| `pin_display` | PIN prompt / CODE label |
| `pin_mask` | Masked PIN display |
| `clr_back_label` | CLR/BACK button label |
| `clr_back_btn` | CLR/BACK button |
| `api_badge` | HA connection status badge |

### Hook-Script Contract

Every device layout **MUST** define these scripts:

| Script | Purpose |
|--------|---------|
| `fx_buzzer(duration_ms: int)` | Audible feedback (no-op OK) |
| `fx_alarm_visual(active: bool)` | Triggered-state indication |

## Compatibility

This project was developed and tested with:
- **Envisalink integration**
- **Single partition** configuration (Partition 1)

However, it can be easily adapted for:
- Any Home Assistant `alarm_control_panel` entity
- **Alarmo**, **Manual Alarm**, or other alarm integrations
- **Multi-partition** setups (duplicate buttons/logic for additional partitions)

---

## Features

- Modern Dark UI with status card
- WiFi Signal Indicator (visual bars)
- HA Connection Badge (green = connected, red = offline)
- Time Display (NTP synced)
- PIN Code Entry with auto-submit at configured length
- Grace Period Cancel (no-PIN cancel during exit delay)
- Event-driven Disarm Verification (8s bounded wait)
- Exit/Entry Countdown Timers
- Arm-failure Watchdog (CHECK PANEL alert)
- Keypad Tones (2.8" buzzer)
- Triggered-state Blink (both displays, 1Hz)
- Dimmed-input Lockout (LVGL top-layer shield)
- Wake-on-alarm and Wake-on-touch
- Auto Dim after 30s (first tap wakes without pressing buttons)
- Captive Portal for WiFi setup
- API Encryption + OTA Password
- Safe Mode (4 attempts)
- Static IP opt-in (device-local only)

## Alarm States

| State | Display | Color |
|-------|---------|-------|
| Disarmed | READY | Green |
| Armed Home | STAY ARMED | Cyan |
| Armed Away | AWAY ARMED | Blue |
| Arming | EXIT DELAY - Ns | Blue |
| Entry Delay | ENTRY DELAY - Ns | Blue |
| Triggered | ALARM! | Red (blinking) |

## Supported Hardware

### 1. CYD 2.8" (320x240) ESP32-2432S028R

| Specification | Details |
|---------------|---------|
| Display | ILI9341 SPI LCD (320x240) |
| Touch | XPT2046 Resistive |
| MCU | ESP32-WROOM-32 |
| Config | `cyd-alarm-panel.yaml` |
| Buzzer | GPIO 26 (active) |
| Purchase | Search "ESP32 2.8 inch CYD" on AliExpress |

### 2. Guition ESP32-4848S040C_I (480x480)

| Specification | Details |
|---------------|---------|
| Display | ST7701S RGB LCD (480x480, 4.0") |
| Touch | GT911 Capacitive (I2C, polling mode) |
| MCU | ESP32-S3 with 16MB Flash, 8MB PSRAM (Octal) |
| Config | `ESP32-4848S040C_I/cyd-alarm-panel-4-inch.yaml` |
| Purchase | [AliExpress](https://www.aliexpress.com/item/1005008797813823.html) |

---

## Hardware Pinouts

### ESP32-4848S040C_I

| Function | GPIO | Notes |
|----------|------|-------|
| **Display** | | |
| CS | 39 | SPI chip select |
| DE | 18 | Data enable |
| HSYNC | 16 | Horizontal sync |
| VSYNC | 17 | Vertical sync |
| PCLK | 21 | Pixel clock (12MHz) |
| SPI CLK | 48 | Init commands |
| SPI MOSI | 47 | Init commands |
| Red Data | 11, 12, 13, 14, 0 | 5-bit red |
| Green Data | 8, 20, 3, 46, 9, 10 | 6-bit green |
| Blue Data | 4, 5, 6, 7, 15 | 5-bit blue |
| **Touch** | | |
| SDA | 19 | I2C data |
| SCL | 45 | I2C clock (strapping pin) |
| **Other** | | |
| Backlight | 38 | PWM controlled |

> **Important Notes:**
> - Touch uses **polling mode** (no interrupt pin) for maximum compatibility.
> - GPIO45 (SCL) is a strapping pin; the warning is suppressed in config.
> - Display requires ST7701S initialization via SPI before RGB works.

### ESP32-2432S028R

#### Display (ILI9341)
| Function | GPIO |
|----------|------|
| CLK | 14 |
| MOSI | 13 |
| MISO | 12 |
| CS | 15 |
| DC | 2 |
| Backlight | 21 |

#### Touchscreen (XPT2046)
| Function | GPIO |
|----------|------|
| CLK | 25 |
| MOSI | 32 |
| MISO | 39 |
| CS | 33 |
| IRQ | 36 |

#### Other
| Function | GPIO |
|----------|------|
| Buzzer | 26 |

---

## Installation

### Step 1: Basic Display Test
First, verify your CYD is working with the basic test config:
- **For CYD 2.8":** Use `cyd-basic-test.yaml`.
- **For Guition 4":** Use `ESP32-4848S040C_I/basic-config-test.yaml`.

### Step 2: Touchscreen Calibration
Follow the [CYD for Beginners](https://github.com/witnessmenow/ESP32-Cheap-Yellow-Display) guide to calibrate the touchscreen if needed.

### Step 3: Install Secrets
Copy `secrets.yaml.example` to `secrets.yaml` and fill in your credentials:

```bash
cp secrets.yaml.example secrets.yaml
```

Then edit `secrets.yaml` with your values. **NEVER commit `secrets.yaml` to git.**

### Step 4: Install Alarm Panel
1. Install `cyd-alarm-panel.yaml` or `ESP32-4848S040C_I/cyd-alarm-panel-4-inch.yaml` to your device.
2. Edit the `substitutions` at the top of the YAML file.
3. Flash the firmware via USB once (required after changing API encryption key).
4. Perform future updates via OTA.

### Step 5: Enable Home Assistant Actions
1. Go to **Settings → Devices & Services**.
2. Find the **ESPHome** integration.
3. Click **Configure** on your CYD device.
4. Enable **"Allow the device to perform Home Assistant actions."**

---

## Configuration

### Substitutions

| Variable | Description | Default |
|----------|-------------|---------|
| `device_name` | ESPHome device name | `cyd-alarm` |
| `friendly_name` | Display name | `CYD Alarm Panel` |
| `alarm_entity` | Your HA alarm entity | `alarm_control_panel.home_alarm_partition_1` |
| `grace_period_ms` | Cancel without PIN time (0 to disable) | `10000` |
| `pin_length` | PIN length for auto-submit | `4` |
| `exit_delay_s` | Exit delay in seconds (0 to disable countdown) | `30` |
| `entry_delay_s` | Entry delay in seconds (0 to disable countdown) | `15` |
| `arm_timeout_s` | Arm-failure watchdog timeout | `20` |
| `timezone` | Your timezone | `Etc/UTC` |

### Tuning Notes

- `arm_timeout_s` must exceed your panel's exit delay plus state propagation time, or the CHECK PANEL banner will fire during normal arming.
- `grace_period_ms: "0"` disables grace-period cancel entirely (use if your Envisalink integration doesn't support code-less disarm).

---

## Secrets & Security

The following secrets are stored in `secrets.yaml` (gitignored):

| Key | Purpose | Generation |
|-----|---------|------------|
| `api_encryption_key` | Encrypts API traffic | `openssl rand -base64 32` |
| `ota_password` | Protects OTA updates | Any strong string |
| `wifi_ssid` | WiFi network name | Your WiFi SSID |
| `wifi_password` | WiFi password | Your WiFi password |
| `ap_password` | Fallback AP password | Any strong string (8+ chars) |

### What Encryption Protects
- **API encryption** prevents local network attackers from intercepting alarm commands or injecting fake states.
- **OTA password** prevents unauthorized firmware updates.
- **WiFi credentials** are never committed to git.

---

## Recovery

### Safe Mode
Safe mode triggers after **4 consecutive failed boot attempts** (e.g., bad WiFi password). In safe mode:
- The device starts a fallback AP for reconfiguration.
- The API and OTA are disabled.
- Connect to the AP and use the captive portal to fix the config.

### Anti-Bootloop Settings
`api.reboot_timeout: 0s` and `wifi.reboot_timeout: 0s` are **deliberate** settings that prevent the device from rebooting when WiFi or API is unreachable. This ensures the panel stays responsive during network outages. **Do not change these during future cleanups.**

### If You Lose the Device
1. Wait for safe mode (4 failed boots).
2. Connect to the fallback AP (`${friendly_name} Setup`).
3. Use the captive portal to reconfigure.
4. If safe mode doesn't engage, flash via USB.

### Static IP — Risks and Recovery
Static IPs are intentionally **NOT supported in the shared core**. A wrong `manual_ip` associates WiFi but leaves the device unroutable — fallback AP and safe mode will NOT engage because the link looks "up".

**If you must use a static IP:**
- Uncomment the template in your **device shell only** (never in core).
- Use a temp-subnet procedure: configure on an isolated network first.
- **Recommendation:** Use DHCP reservation on your router instead.

---

## Usage

### Main Screen
- **STAY**: Arm in home mode (starts exit delay countdown).
- **AWAY**: Arm in away mode (starts exit delay countdown).
- **CANCEL**: During exit delay, cancels arming without PIN (within grace period).
- **DISARM**: Opens PIN pad (when armed).

### PIN Pad
- **Digits 0-9**: Enter PIN (auto-submits at configured length).
- **CLR**: Clear entry.
- **BACK**: Return to main screen.
- **OK**: Submit PIN manually.

### Grace-Period Cancel
Press **CANCEL** during the exit delay to disarm without entering a PIN. Works only within the configured grace period (default 10s).

### Dimmed Screen
After 30s of inactivity, the screen dims. The **first tap wakes the screen WITHOUT pressing any underlying button** (LVGL top-layer shield absorbs the tap).

### HA Connection Badge
- **Green**: API connected.
- **Red**: API disconnected (check HA/network).

---

## On-Device Test Checklist

After installation, verify the following:

- [ ] Flash via USB once post-auth-change (OTA password invalidates old pairing assumptions).
- [ ] Arm STAY/AWAY → countdown matches panel programming ±1s.
- [ ] CANCEL within grace window aborts with no PIN.
- [ ] CANCEL outside window opens PIN pad.
- [ ] Grace-cancel with HA stopped → "CANCEL FAILED" after ~6s.
- [ ] Commissioning: confirm your Envisalink integration accepts code-less disarm (can you disarm from the HA UI without typing a code?). If not, grace cancel will always report CANCEL FAILED and should be disabled via `grace_period_ms: "0"`.
- [ ] Wrong PIN → WRONG CODE + long buzz (2.8").
- [ ] Disarm with HA stopped → "HA OFFLINE" (not "WRONG CODE") after ~8s.
- [ ] Correct PIN → DISARMED confirmation arrives as fast as HA responds (no fixed lag).
- [ ] Auto-submit fires at configured length.
- [ ] Dim after 30s → first tap wakes WITHOUT pressing underlying button.
- [ ] Entry-delay wake lights screen.
- [ ] Unplug HA/network → badge goes red within 2s.
- [ ] AP-fallback rehearsal: force WiFi failure, confirm `${friendly_name} Setup` AP appears and captive portal loads on a phone.
- [ ] Safe-mode rehearsal: temporarily bad WiFi password, observe 4-attempt safe-mode boot, OTA in, restore.

### Maintenance Rules
- Re-run the widget-ID audit (grep for each required ID) after ANY LVGL layout refactor.
- Do not remove the dim-lockout top-layer flag when tweaking UI pages — it is the mis-touch guard, not decoration.

---

## Credits
- [CYD for Beginners](https://github.com/witnessmenow/ESP32-Cheap-Yellow-Display)
- [ESPHome](https://esphome.io/)
- [LVGL](https://lvgl.io/)

**Disclaimer**: This project is not affiliated with Home Assistant or ESPHome. Use at your own risk.
