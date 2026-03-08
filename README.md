# CYD Alarm Panel for Home Assistant

A modern, touchscreen alarm panel for Home Assistant using the ESP32-2432S028R "Cheap Yellow Display" (CYD).

## Overview

This project turns the inexpensive ESP32-2432S028R display (~$15) into a fully functional alarm panel for Home Assistant. It features a modern dark UI, PIN code entry, and seamless integration with Home Assistant alarm control panel entities.

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

## Hardware

### Required
**ESP32-2432S028R** (Cheap Yellow Display / CYD)
- 2.8" ILI9341 320x240 TFT display
- XPT2046 resistive touchscreen
- ESP32-WROOM-32 module
- USB-C and Micro USB ports

### Where to Buy
- **AliExpress**: Search for "ESP32-2432S028R"
- **Amazon**: Search for "Cheap Yellow Display ESP32"
- Typical price: $10-15 USD

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

```yaml
esphome:
  name: cyd-display-test

esp32:
  board: esp32dev
  framework:
    type: esp-idf

logger:

api:
  reboot_timeout: 0s

ota:
  - platform: esphome

# =============================================================================
# WiFi Configuration
# =============================================================================
# Option 1: Use the captive portal (default)
#   - On first boot, the device creates a WiFi access point
#   - Connect to "CYD-Alarm Setup" with password "configme123"
#   - A portal will open to configure your WiFi credentials
#
# Option 2: Hardcode your WiFi credentials
#   - Uncomment the ssid/password lines below
#   - Add your credentials to secrets.yaml (ESPHome Dashboard → Secrets):
#       wifi_ssid: "Your_WiFi_Name"
#       wifi_password: "Your_WiFi_Password"
#   - Or replace !secret references with your actual credentials
#
# Option 3: Static IP (optional, use with Option 2)
#   - Uncomment the manual_ip section below
# =============================================================================
wifi:
  # Uncomment below to hardcode WiFi credentials
  # ssid: !secret wifi_ssid
  # password: !secret wifi_password
  
  # Optional: Uncomment below for static IP
  # manual_ip:
  #   static_ip: 192.168.1.70
  #   gateway: 192.168.1.1
  #   subnet: 255.255.255.0
  #   dns1: 192.168.1.1
  
  # Fallback/Setup AP - connects to this if no WiFi configured or connection fails
  ap:
    ssid: "CYD-Alarm Setup"
    password: "configme123"
  
  reboot_timeout: 0s

captive_portal:

output:
  - platform: ledc
    pin: GPIO21
    id: backlight_pwm

light:
  - platform: monochromatic
    output: backlight_pwm
    name: Display Backlight
    id: backlight
    restore_mode: ALWAYS_ON

spi:
  - id: tft
    clk_pin: GPIO14
    mosi_pin: GPIO13
    miso_pin: GPIO12

display:
  - platform: ili9xxx
    model: ILI9341
    spi_id: tft
    cs_pin: GPIO15
    dc_pin: GPIO2
    auto_clear_enabled: true
    invert_colors: false
    color_palette: 8BIT
    show_test_card: true
    rotation: 0
    dimensions:
      width: 320
      height: 240
```

If you see a blue screen with "CYD TEST" text, proceed to the next step.

### Step 2: Touchscreen Calibration

Add touchscreen support and calibrate:

```yaml
esphome:
  name: cyd-alarm

esp32:
  board: esp32dev
  framework:
    type: esp-idf

logger:

api:
  reboot_timeout: 0s

ota:
  - platform: esphome

# =============================================================================
# WiFi Configuration
# =============================================================================
# Option 1: Use the captive portal (default)
#   - On first boot, the device creates a WiFi access point
#   - Connect to "CYD-Alarm Setup" with password "configme123"
#   - A portal will open to configure your WiFi credentials
#
# Option 2: Hardcode your WiFi credentials
#   - Uncomment the ssid/password lines below
#   - Add your credentials to secrets.yaml (ESPHome Dashboard → Secrets):
#       wifi_ssid: "Your_WiFi_Name"
#       wifi_password: "Your_WiFi_Password"
#   - Or replace !secret references with your actual credentials
#
# Option 3: Static IP (optional, use with Option 2)
#   - Uncomment the manual_ip section below
# =============================================================================
wifi:
  # Uncomment below to hardcode WiFi credentials
  # ssid: !secret wifi_ssid
  # password: !secret wifi_password
  
  # Optional: Uncomment below for static IP
  # manual_ip:
  #   static_ip: 192.168.1.70
  #   gateway: 192.168.1.1
  #   subnet: 255.255.255.0
  #   dns1: 192.168.1.1
  
  # Fallback/Setup AP - connects to this if no WiFi configured or connection fails
  ap:
    ssid: "CYD-Alarm Setup"
    password: "configme123"
  
  reboot_timeout: 0s

captive_portal:

output:
  - platform: ledc
    pin: GPIO21
    id: backlight_pwm

light:
  - platform: monochromatic
    output: backlight_pwm
    name: Display Backlight
    id: backlight
    restore_mode: ALWAYS_ON

spi:
  - id: tft
    clk_pin: GPIO14
    mosi_pin: GPIO13
    miso_pin: GPIO12

  - id: touch
    clk_pin: GPIO25
    mosi_pin: GPIO32
    miso_pin: GPIO39

display:
  - platform: ili9xxx
    model: ILI9341
    id: my_display
    spi_id: tft
    cs_pin: GPIO15
    dc_pin: GPIO2
    auto_clear_enabled: true
    invert_colors: false
    color_palette: 8BIT
    rotation: 0
    dimensions:
      width: 320
      height: 240
    lambda: |-
      it.fill(Color(32, 32, 32));
      it.print(160, 120, id(roboto), Color(255, 255, 255), TextAlign::CENTER, "TOUCH TEST");

touchscreen:
  - platform: xpt2046
    id: my_touchscreen
    spi_id: touch
    cs_pin: GPIO33
    interrupt_pin: GPIO36
    calibration:
      x_min: 280
      x_max: 3860
      y_min: 340
      y_max: 3860
    on_touch:
      - logger.log: "Touch detected!"
      - light.turn_on:
          id: backlight
          brightness: 1.0

font:
  - file: "gfonts://Roboto"
    id: roboto
    size: 24
```

Touch each corner of the screen and note the values in the ESPHome logs. Use these values to set your calibration.

### Step 3: Install Alarm Panel

1. Install `cyd-alarm-panel.yaml` to your ESP device using the the web or other supported method

2. Edit the substitutions at the top:

```yaml
substitutions:
  device_name: cyd-alarm
  friendly_name: "CYD Alarm Panel"
  # Change this to your alarm entity ID
  alarm_entity: "alarm_control_panel.home_alarm_partition_1"
  # Grace period in milliseconds (10 seconds = 10000)
  grace_period_ms: "10000"
  # Change this to your timezone - examples:
  # Australia/Sydney, America/New_York, Europe/London, Asia/Tokyo
  # Full list: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
  timezone: "Australia/Sydney"
```

3. Configure your WIFI credentials using which ever method you prefer
   
4. Flash to your CYD via USB for initial install

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

```yaml
touchscreen:
  - platform: xpt2046
    calibration:
      x_min: 435
      x_max: 3835
      y_min: 225
      y_max: 3635
    transform:
      swap_xy: true
      mirror_x: true
      mirror_y: true
```

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
