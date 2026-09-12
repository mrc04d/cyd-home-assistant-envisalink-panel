# Spec: Global Secrets + Dual 4" Panel Setup

## Goal

Two Guition 4" alarm panels sharing a single `secrets.yaml` with DHCP (router-reserved IPs, no static IP config).

| Panel | device_name | friendly_name | IP (reserved in router) |
|-------|------------|---------------|------------------------|
| 1     | "107"      | "Garage Alarm Panel"  | 192.168.1.107 |
| 2     | "156"      | "Upstairs Alarm Panel" | 192.168.1.156 |

## ESPHome Secret Resolution

ESPHome's `!secret` directive resolves secrets from a `secrets.yaml` file in the directory of the configuration file being compiled. It does **not** automatically traverse parent directories. To share one global `secrets.yaml` across device subdirectories, each directory containing `!secret` references needs a `secrets.yaml` shim or symlink pointing to the root file.

**How it works:**
- `common/alarm-core.yaml` uses `!secret api_encryption_key` and `!secret ota_password`
- `common/secrets.yaml` → symlink to `../secrets.yaml` (root global secrets)
- Device shells in `guition-4inch/` include `common/alarm-core.yaml` via `packages`
- `guition-4inch/secrets.yaml` → symlink to `../secrets.yaml` (root global secrets)

**Verification:**
```bash
# ESPHome resolves secrets via the symlinks above
# No special configuration needed in each shell beyond the symlink
```

## Shared vs Unique Secrets

Per ESPHome security best practices, WiFi credentials can be shared across devices. API encryption keys and OTA passwords **should** be unique per device, but the user has explicitly decided to share all secrets. Document this decision **and its risks**:

| Secret | Scope | Notes |
|--------|-------|-------|
| `wifi_ssid` | Shared | Same WiFi for both panels |
| `wifi_password` | Shared | Same WiFi for both panels |
| `api_encryption_key` | Shared | **Risk:** Compromise of one panel allows API access to the other; physical loss of one panel requires rotating the key on both |
| `ota_password` | Shared | **Risk:** Same OTA credential controls firmware updates for both devices; leak allows reflashing both |
| `ap_password` | Shared | Same fallback AP password |

### Secrets naming convention (if unique credentials adopted later):
`api_encryption_key_107`, `api_encryption_key_156`, `ota_password_107`, `ota_password_156`. Current spec uses shared values per user decision.

## Directory Structure (Target)

```
cyd-alarm-v2/
├── secrets.yaml              # Global secrets (gitignored, real values)
├── secrets.yaml.example      # Template (committed, keys list)
├── .gitignore                # Must exclude secrets.yaml
├── common/
│   ├── alarm-core.yaml       # Shared logic (uses !secret)
│   └── secrets.yaml          # Symlink -> ../secrets.yaml
├── guition-4inch/
│   ├── secrets.yaml          # Symlink -> ../secrets.yaml
│   ├── cyd-alarm-panel-4-inch-wifi-overlay-and-status.yaml        # 107 Garage Alarm Panel
│   └── cyd-alarm-panel-4-inch-wifi-overlay-and-status-156.yaml    # 156 Upstairs Alarm Panel
├── cyd-2.8inch/
│   ├── secrets.yaml          # Symlink -> ../secrets.yaml
│   └── cyd-alarm-panel.yaml    # CYD 2.8" (uses same secrets.yaml)
└── SPEC-global-secrets.md      # This file
```

**Note:** Symlinks in `common/`, `guition-4inch/`, and `cyd-2.8inch/` are required for ESPHome to find `secrets.yaml` when resolving `!secret` tags in `alarm-core.yaml` and device shells.

## Device Shell Template

Each 4" panel device shell will have this structure:

```yaml
substitutions:
  device_name: "107"
  friendly_name: "Garage Alarm Panel"
  name_add_mac_suffix: false
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

**Requirements:**
- Every device shell must have a unique `device_name` and `friendly_name`
- `name_add_mac_suffix: false` ensures exact device names `107` and `156` are used
- `packages:` recursively merges `common/alarm-core.yaml` which defines `wifi:`, `api:`, `ota:` — do **not** duplicate these sections in the shell
- Numeric-only device names (`"107"`, `"156"`) are valid in ESPHome 2024+ (quoted as strings)

## Key Design Decisions

1. **No static IP** in config — DHCP with router reservation only
2. **Single `secrets.yaml`** — both panels read from the same root file via symlinks in `common/` and device directories
3. **No duplicate wifi/api/ota** — all in `common/alarm-core.yaml`; `packages:` recursively merges these sections
4. **Symlinks required** — `common/secrets.yaml`, `guition-4inch/secrets.yaml`, `cyd-2.8inch/secrets.yaml` each point to `../secrets.yaml`
5. **Both panels share secrets** — user decision; normally API key and OTA password would differ
6. **`secrets.yaml.example` sync** — template keys must stay in sync with `common/alarm-core.yaml` when new secrets are added

## Verification

Run from project root `cyd-alarm-v2/`:

```bash
# From project root - symlinks ensure secrets resolve
esphome config guition-4inch/cyd-alarm-panel-4-inch-wifi-overlay-and-status.yaml
esphome config guition-4inch/cyd-alarm-panel-4-inch-wifi-overlay-and-status-156.yaml
esphome config cyd-2.8inch/cyd-alarm-panel.yaml
```

All should output "Configuration is valid!" and resolve required secret keys without exposing values.

**Recovery requirements (DHCP with reservations):**
- Fallback AP and captive portal enabled in `alarm-core.yaml`
- Valid `wifi:` block with `use_address` only when needed for recovery
- Documented recovery procedure if panel moved to another network
