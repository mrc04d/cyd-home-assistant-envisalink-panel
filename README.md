# CYD Alarm Panel for Home Assistant

A modern, touchscreen alarm panel for Home Assistant using cheap ESP32 displays.

## Overview

This project turns an inexpensive display (~$15–$30) into a fully functional alarm panel for Home Assistant. It features a modern dark UI, PIN code entry, and seamless integration with Home Assistant alarm control panel entities.

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

- 🎨 **Modern Dark UI** – Clean, professional appearance with status card
- 📶 **WiFi Signal Indicator** – Visual signal strength bars
- 🕐 **Time Display** – NTP synced with configurable timezone
- 🔐 **PIN Code Entry** – Secure numpad for disarming
- ⏱️ **Grace Period** – Cancel arming within 10 seconds without PIN
- 💡 **Auto Dim** – Screen dims after 30 seconds, wakes on touch
- 📡 **Captive Portal** – Easy WiFi setup without hardcoding credentials

## Alarm States

| State | Display | Color |
|-------|---------|-------|
| Disarmed | READY | Green |
| Armed Home | STAY ARMED | Yellow |
| Armed Away | AWAY ARMED | Red |
| Arming | EXIT DELAY | Orange |
| Entry Delay | ENTRY DELAY | Orange |
| Triggered | ALARM! | Red |

## Supported Hardware

### 1. CYD 2.8" (320x240) ESP32-2432S028R

| Specification | Details |
|---------------|---------|
| Display | ILI9341 SPI LCD (320x240) |
| Touch | XPT2046 Resistive |
| MCU | ESP32-WROOM-32 |
| Config | `cyd-alarm-panel.yaml` |
| Test Config | `cyd-basic-test.yaml` |
| Touch Screen Calibration | `touch-screen-test-and-calibration.yaml` |
| Purchase | Search "ESP32 2.8 inch CYD" on AliExpress |

### 2. Guition ESP32-4848S040C_I (480x480) - NEW ✨

| Specification | Details |
|---------------|---------|
| Display | ST7701S RGB LCD (480x480, 4.0") |
| Touch | GT911 Capacitive (I2C, polling mode) |
| MCU | ESP32-S3 with 16MB Flash, 8MB PSRAM (Octal) |
| Config | `ESP32-4848S040C_I/cyd-alarm-panel-4-inch.yaml` |
| Test Config | `ESP32-4848S040C_I/basic-config-test.yaml` |
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

### Step 3: Install Alarm Panel
1. Install `cyd-alarm-panel.yaml` or `ESP32-4848S040C_I/cyd-alarm-panel-4-inch.yaml` to your device.
2. Edit the `substitutions` at the top of the YAML file.
3. Configure your WiFi credentials using your preferred method.
4. Flash the firmware and perform future updates via OTA.

### Step 4: Enable Home Assistant Actions
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
| `grace_period_ms` | Cancel without PIN time | `10000` |
| `timezone` | Your timezone | `Australia/Sydney` |

---

## Usage

### Main Screen
- **STAY**: Arm in home mode.
- **AWAY**: Arm in away mode.
- **DISARM**: Opens PIN pad (when armed).

### PIN Pad
- **CLR**: Clear entry.
- **BACK**: Return to main screen.
- **OK**: Submit PIN.
- *Wrong PIN shows "WRONG PIN" in red.*

---

## Credits
- [CYD for Beginners](https://github.com/witnessmenow/ESP32-Cheap-Yellow-Display)
- [ESPHome](https://esphome.io/)
- [LVGL](https://lvgl.io/)

**Disclaimer**: This project is not affiliated with Home Assistant or ESPHome. Use at your own risk.