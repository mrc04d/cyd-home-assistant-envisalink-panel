# CYD Alarm Panel for Home Assistant

A modern, touchscreen alarm panel for Home Assistant using the cheap esp32 displays.

## Overview

This project turns the inexpensive display (~$15-$30) into a fully functional alarm panel for Home Assistant. It features a modern dark UI, PIN code entry, and seamless integration with Home Assistant alarm control panel entities.

## Compatibility

This project was developed and tested with:
- **Envisalink integration**
- **Single partition** configuration (Partition 1)

However, it can be easily adapted for:
- Any Home Assistant `alarm_control_panel` entity
- **Alarmo**, **Manual Alarm**, or other alarm integrations
- **Multi-partition** setups (duplicate buttons/logic for additional partitions)

## Features

- 🎨 **Modern Dark UI** - Clean, professional appearance with status card
- 📶 **WiFi Signal Indicator** - Visual signal strength bars
- 🕐 **Time Display** - NTP synced with configurable timezone
- 🔐 **PIN Code Entry** - Secure numpad for disarming
- ⏱️ **Grace Period** - Cancel arming within 10 seconds without PIN
- 💡 **Auto Dim** - Screen dims after 30 seconds, wakes on touch
- 📡 **Captive Portal** - Easy WiFi setup without hardcoding credentials

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

#### Verified Pinout for ESP32-4848S040C_I

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

**Important Notes:**
- Touch uses **polling mode** (no interrupt pin) for maximum compatibility
- GPIO45 (SCL) is a strapping pin - warning suppressed in config
- Display requires ST7701S initialization via SPI before RGB works

# Verified Pinout for ESP32-2432S028R

### Display (ILI9341)
| Function | GPIO |
|----------|------|
| CLK | 14 |
| MOSI | 13 |
| MISO | 12 |
| CS | 15 |
| DC | 2 |
| Backlight | 21 |

### Touchscreen (XPT2046)
| Function | GPIO |
|----------|------|
| CLK | 25 |
| MOSI | 32 |
| MISO | 39 |
| CS | 33 |
| IRQ | 36 |

### Other
| Function | GPIO |
|----------|------|
| Buzzer | 26 |

## Getting Started

### Prerequisites
- Home Assistant with an alarm control panel entity
- ESPHome addon installed in Home Assistant
- USB cable for initial flashing

### Reference Documentation
This project builds upon the excellent CYD setup guide:
- [CYD for Beginners by Decryption](https://github.com/witnessmenow/ESP32-Cheap-Yellow-Display)

## Installation

### Step 1: Basic Display Test

First, verify your CYD is working with this basic test config:

If you see a blue screen with "CYD TEST" text, proceed to the next step.

### Step 2: If needed, Touchscreen Calibration

Add touchscreen support and calibrate:

Touch each corner of the screen and note the values in the ESPHome logs. Use these values to set your calibration.

### Step 3: Install Alarm Panel

1. Install `cyd-alarm-panel.yaml` to your ESP device using the the web or other supported method

2. Edit the substitutions at the top:

3. Configure your WIFI credentials using which ever method you prefer
   
4. Flash to your device

5. Future updates can be done OTA

### Step 4: Enable Home Assistant Actions

After the device connects to Home Assistant:

1. Go to **Settings → Devices & Services**
2. Find **ESPHome** integration
3. Click **Configure** on your CYD device
4. Enable **"Allow the device to perform Home Assistant actions"**

## Configuration

### Substitutions

| Variable | Description | Default |
|----------|-------------|---------|
| `device_name` | ESPHome device name | `cyd-alarm` |
| `friendly_name` | Display name | `CYD Alarm Panel` |
| `alarm_entity` | Your HA alarm entity | `alarm_control_panel.home_alarm_partition_1` |
| `grace_period_ms` | Cancel without PIN time | `10000` (10 seconds) |
| `timezone` | Your timezone | `Australia/Sydney` |

### Timezone Examples
- `Australia/Sydney`
- `America/New_York`
- `America/Los_Angeles`
- `Europe/London`
- `Europe/Paris`
- `Asia/Tokyo`

Full list: [Wikipedia Timezone Database](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)

### Touch Calibration

If touch is inaccurate, adjust the calibration values:

## Usage

### Main Screen
- **STAY** - Arm in home mode
- **AWAY** - Arm in away mode

### During Exit Delay
- **CANCEL** - Cancel arming
  - Within 10 seconds: No PIN required (grace period)
  - After 10 seconds: PIN required

### When Armed
- **DISARM** - Opens PIN pad to disarm

### PIN Pad
- Enter your alarm PIN
- **CLR** - Clear entry
- **BACK** - Return to main screen
- **OK** - Submit PIN

Wrong PIN shows "WRONG PIN" in red and clears for retry.

## Troubleshooting

### Display shows white/blank
- Verify SPI pins match your CYD variant
- Check `invert_colors` setting
- Try `color_order: bgr` or `color_order: rgb`

### Touch not working
- Verify touch SPI pins (separate from display SPI)
- Check calibration values in logs
- Try different transform combinations

### Time is wrong
- Verify timezone in substitutions
- Check internet connectivity for NTP

### Actions not working
- Enable "Allow device to perform Home Assistant actions" in ESPHome integration settings
- Verify `alarm_entity` matches your HA entity ID

### Build errors
- Clean build files: Three dots menu → Clean Build Files
- Delete `.esphome/build/cyd-alarm` folder

## Hardware Pinout

### Display (ILI9341)
| Function | GPIO |
|----------|------|
| CLK | 14 |
| MOSI | 13 |
| MISO | 12 |
| CS | 15 |
| DC | 2 |
| Backlight | 21 |

### Touchscreen (XPT2046)
| Function | GPIO |
|----------|------|
| CLK | 25 |
| MOSI | 32 |
| MISO | 39 |
| CS | 33 |
| IRQ | 36 |

### Other
| Function | GPIO |
|----------|------|
| Buzzer | 26 |

## Tested Configuration

This project was developed and tested with:

| Component | Details |
|-----------|---------|
| Hardware | ESP32-2432S028R (USB-C + Micro USB variant) |
| Alarm Panel | DSC PowerSeries |
| Integration | Envisalink EVL-4 |
| Home Assistant | 2024.x+ |
| ESPHome | 2024.x+ |
| Partitions | Single partition (Partition 1) |

## License

MIT License - See LICENSE file

## Credits

- [CYD for Beginners](https://github.com/witnessmenow/ESP32-Cheap-Yellow-Display) - Hardware setup guide
- [ESPHome](https://esphome.io/) - Firmware framework
- [LVGL](https://lvgl.io/) - Graphics library
- [Home Assistant](https://www.home-assistant.io/) - Home automation platform

## Contributing

Contributions welcome! Please open an issue or pull request.

Ideas for contributions:
- Multi-partition support
- Additional alarm integrations
- Alternative color themes
- Zone status display
- Panic button

---

**Disclaimer**: This project is not affiliated with Home Assistant, ESPHome, Envisalink, DSC, or Honeywell. Use at your own risk. This is a supplementary control panel and should not be relied upon as your primary alarm interface.
