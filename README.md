# ESPHome Room Presence

Turn cheap ESP32 boards into **room nodes** for Home Assistant: one per room, each one
a **Bluetooth presence scanner** and a **Wi-Fi monitor**. Together they tell you *who is
in which room* (via the phones, watches or BLE tags people carry) and how good your Wi-Fi
is everywhere in the house.

- **Room presence**: every node is an active ESPHome Bluetooth proxy. Home Assistant +
  [Bermuda BLE Trilateration](https://github.com/agittins/bermuda) compares the signal of
  each tracked device across all nodes and places it in the area of the closest node.
- **Wi-Fi monitoring**: signal strength (dBm and %), connected access point (BSSID), channel,
  disconnect counter, uptime, chip temperature, free memory and reset reason, per room.
- **Free room choice at install time**: the room is a single setting that becomes the node's
  Home Assistant area. Nothing is hard-coded, so you can use it in any house.
- **Plain ESPHome packages**: no custom components, easy to review and extend.

## Hardware

Any ESP32 with Bluetooth, for example an ESP32-WROOM-32 DevKit (about €5) powered by a
USB phone charger. Classic ESP32, ESP32-S3 and ESP32-C3 work; the **ESP8266 does not**
(it has no Bluetooth).

Placement tips: one node per room, about 1–2 m high, away from large metal objects and
not right next to a wall you share with another room.

## Installation

### 1. Add the packages to ESPHome

**Option A: Windows helper script** (copies the packages, creates the shared secrets and
asks for the room):

```powershell
.\tools\Deploy-ToHomeAssistant.ps1 -EsphomeDir \\homeassistant.local\config\esphome
# or non-interactive:
.\tools\Deploy-ToHomeAssistant.ps1 -EsphomeDir \\homeassistant.local\config\esphome -RoomName "Living room"
```

**Option B: manual**
1. Copy `packages/` to `<esphome config>/packages/room-presence/`.
2. Add the keys from [`examples/secrets.example.yaml`](examples/secrets.example.yaml) to
   your ESPHome `secrets.yaml`.
3. Create one device file per room, see [`examples/room-living-room.yaml`](examples/room-living-room.yaml):

```yaml
substitutions:
  name: room-living-room          # hostname: a-z, 0-9 and '-'
  friendly_name: Room Living room # device name in Home Assistant
  room: Living room               # Home Assistant area (created if it does not exist)

packages:
  room_node: !include packages/room-presence/room-node.yaml
```

**Option C: straight from GitHub**: see [`examples/room-remote-package.yaml`](examples/room-remote-package.yaml).

### 2. Flash the node

In the ESPHome Device Builder: open the device, **Install**, then **Plug into this computer**
for the first flash. Later updates install over Wi-Fi.

### 3. Add it to Home Assistant

Home Assistant discovers the node; confirm it under *Settings → Devices & services → ESPHome*.
It lands in the area you chose as `room`. To move a node to another room, change its area in
Home Assistant (or change `room` and re-install).

### 4. Room presence with Bermuda

1. Install **Bermuda BLE Trilateration** (HACS) and add the integration.
2. Make the people's devices visible over Bluetooth:
   - **Android**: Home Assistant Companion app → *Settings → Companion app → Manage sensors →
     BLE Transmitter*, enable it. Bermuda picks up the iBeacon.
   - **iPhone / Apple Watch**: iOS randomises Bluetooth addresses. Use the core
     **Private BLE Device** integration with the phone's IRK, then select it in Bermuda.
   - **BLE tags** (Tile, NutTag, etc.) with a fixed address can be selected directly.
3. In Bermuda's options, select the devices to track. You get, per device,
   `sensor.<device>_area` (the room it is in) and a distance per node.
4. Optional: put the area sensor in a `person` or use it in automations
   ("turn off the lights when nobody is left in the room").

Tuning: Bermuda options *max radius* and *smoothing* trade speed for stability. Room nodes
should all run the same firmware, so the signal levels are comparable.

## Entities per node

| Entity | Description |
| --- | --- |
| Wi-Fi signal / Wi-Fi signal quality | RSSI in dBm and as 0–100 % |
| Access point (BSSID), Wi-Fi channel, Wi-Fi SSID | Which AP the node is connected to |
| Wi-Fi disconnects | Number of drops since boot |
| Uptime, Reset reason, Chip temperature, Free heap, Loop time | Device health |
| Bluetooth scanning (switch) | Pause BLE scanning, for example for diagnosis |
| Restart, Restart (safe mode) | Maintenance buttons |
| Status | Online / offline |

## Configuration options (substitutions)

| Substitution | Default | Purpose |
| --- | --- | --- |
| `name` | `room-node` | Hostname |
| `friendly_name` | `Room node` | Device name |
| `room` | *(empty)* | Home Assistant area |
| `board` | `esp32dev` | PlatformIO board id (`esp32-s3-devkitc-1`, `esp32-c3-devkitm-1`, ...) |
| `log_level` | `INFO` | ESPHome log level |
| `wifi_update_interval` | `30s` | Update interval of the Wi-Fi sensors |
| `ble_scan_interval` / `ble_scan_window` | `1100ms` | BLE scan timing (continuous by default) |

You can also include the packages separately (`core.yaml`, `wifi-monitor.yaml`,
`ble-presence.yaml`) if you only want one of the two functions.

## Privacy

All data stays in your own network: the nodes only listen to Bluetooth advertisements and
send them to your own Home Assistant. Only track people who know about it and agree to it.

## License

[MIT](LICENSE)
