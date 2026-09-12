# Spec: Global Secrets + Dual 4" Panel Setup

## Goal

Two Guition 4" alarm panels sharing a single `secrets.yaml` with DHCP (router-reserved IPs, no static IP config).

| Panel | device_name | friendly_name | IP (reserved in router) |
|-------|------------|---------------|------------------------|
| 1     | "107"      | "Garage Alarm Panel"  | 192.168.1.107 |
| 2     | "156"      | "Upstairs Alarm Panel" | 192.168.1.156 |

## ESPHome Secret Resolution

ESPHome's `!secret` directive automatically resolves from a global `secrets.yaml` in the configuration root directory (traversing parent directories automatically). No explicit file inclusion syntax is required for `!secret` tags.

**How it works:**
- When a device shell in `guition-4inch/` uses `!secret api_encryption_key`, ESPHome looks for `secrets.yaml` starting from the current file and traversing upward
- Root `secrets.yaml` is automatically inherited by all device configs
- No symlink is needed in device directories

**Verification:**
```bash
# ESPHome resolves secrets from root secrets.yaml automatically
# No special configuration needed in each shell
```

## Shared vs Unique Secrets

Per ESPHome security best practices, WiFi credentials can be shared across devices. API encryption keys and OTA passwords **should** be unique per device, but the user has explicitly decided to share all secrets. Document this decision.

| Secret | Scope | Notes |
|--------|-------|-------|
| `wifi_ssid` | Shared | Same WiFi for both panels |
| `wifi_password` | Shared | Same WiFi for both panels |
| `api_encryption_key` | Shared | User decision (same key for both) |
| `ota_password` | Shared | User decision (same password for both) |
| `ap_password` | Shared | Same fallback AP password |

## Directory Structure (Target)

```
cyd-alarm-v2/
├── secrets.yaml              # Global secrets (gitignored, real values)
├── secrets.yaml.example      # Template (committed)
├── common/
│   └── alarm-core.yaml       # Shared logic (uses !secret)
├── guition-4inch/
│   ├── cyd-alarm-panel-4-inch-wifi-overlay-and-status.yaml        # 107 Garage Alarm Panel
│   └── cyd-alarm-panel-4-inch-wifi-overlay-and-status-156.yaml    # 156 Upstairs Alarm Panel
├── cyd-2.8inch/
│   └── cyd-alarm-panel.yaml    # CYD 2.8" (uses same secrets.yaml)
└── SPEC-global-secrets.md      # This file
```

**Note:** No `secrets.yaml` symlinks needed in device directories. ESPHome automatically inherits secrets from the root `secrets.yaml`.

## Device Shell Template

Each 4" panel device shell will have this structure:

```yaml
substitutions:
  device_name: "107"
  friendly_name: "Garage Alarm Panel"
  alarm_entity: "alarm_control_panel.home_alarm_partition_1"
  timezone: "Etc/UTC"
  idle_brightness: "0.15"

packages:
  core: !include ../common/alarm-core.yaml

esphome:
  name: ${device_name}
  friendly_name: ${friendly_name}

esp32:
  board: esp32-s3-devkitc-1
  variant: esp32s3
  flash_size: 16MB
  framework:
    type: esp-idf
    version: recommended

psram:
  mode: octal
  size: 8MB

display:
  - platform: ld056uc
    ...

lvgl:
  ...
```

## Key Design Decisions

1. **No static IP** in config — DHCP with router reservation only
2. **Single `secrets.yaml`** — both panels read from the same file via ESPHome's automatic secret resolution
3. **No duplicate wifi/api/ota** — all in `common/alarm-core.yaml`
4. **No `<<: !include` needed** — ESPHome automatically inherits secrets from root `secrets.yaml`
5. **Both panels share secrets** — user decision; normally API key and OTA password would differ

## Verification

After implementation:
```bash
esphome config guition-4inch/cyd-alarm-panel-4-inch-wifi-overlay-and-status.yaml
esphome config guition-4inch/cyd-alarm-panel-4-inch-wifi-overlay-and-status-156.yaml
```
Both should output "Configuration is valid!" and show resolved secrets from `secrets.yaml`.
