# Manual: ESPHome Room Presence

*Nederlandse versie: [handleiding.md](handleiding.md)*

This manual takes you from a bare ESP32 board to "Home Assistant knows who is in which room".

1. [What it does](#1-what-it-does)
2. [Hardware and placement](#2-hardware-and-placement)
3. [Install the firmware](#3-install-the-firmware)
4. [Add the node to Home Assistant and choose the room](#4-add-the-node-to-home-assistant-and-choose-the-room)
5. [The node's web page](#5-the-nodes-web-page)
6. [Track people with Bermuda](#6-track-people-with-bermuda)
7. [Updating](#7-updating)
8. [Troubleshooting](#8-troubleshooting)
9. [Reference: entities and settings](#9-reference-entities-and-settings)

---

## 1. What it does

Each room gets one small ESP32 board, a *room node*. A room node does two things:

- **Bluetooth presence.** It listens to the Bluetooth signals that phones, smartwatches and
  tags send out all the time, and forwards them (with signal strength) to Home Assistant.
  The [Bermuda](https://github.com/agittins/bermuda) integration compares the signal of a
  device across all room nodes and places the device in the room of the closest node.
- **Wi-Fi monitoring.** It reports the Wi-Fi signal strength, the access point and channel it
  is connected to and how often the connection drops, so you get a Wi-Fi map of your house.

Everything stays inside your own network.

**Good to know**

- Bermuda tracks *devices*, not people. A phone left on the couch is "in the living room".
- Bluetooth goes through walls. Results are good with a node in (almost) every room; with
  only one or two nodes you mostly learn "near this node or not".
- Room changes are detected within a few seconds to about half a minute.

## 2. Hardware and placement

| Supported | Not supported |
| --- | --- |
| ESP32 (e.g. ESP32-WROOM-32 DevKit, ~€5), ESP32-S3, ESP32-C3 | ESP8266 / NodeMCU / Wemos D1 (no Bluetooth) |

You also need a **USB data cable** for the first installation and a **USB phone charger**
per node.

Placement:

- one node per room, roughly 1–2 m high (a shelf or cupboard top is fine);
- not directly against a wall you share with another room;
- away from large metal objects (fridge, radiator, metal cabinet);
- use the same type of board everywhere, so signal levels are comparable.

## 3. Install the firmware

There are two ways. Pick **A** unless you already manage your own ESPHome configurations.

### A. Web installer (recommended, no software needed)

1. Open the installer page: **https://schmeitz.github.io/esphome-room-presence/** in
   **Chrome or Edge on a computer**.
2. Plug the board in and click **Install on ESP32**. Choose the serial port of the board
   (it is usually called *USB-SERIAL CH340*, *CP210x* or *USB JTAG*).
3. Confirm the installation. The right firmware for your chip is chosen automatically.
   Flashing takes about a minute.
4. When asked, enter the **name and password of your Wi-Fi** (2.4 GHz). The board connects
   and shows up in Home Assistant (see chapter 4).

The pre-built firmware has no passwords built in, so anyone can install it. When you want to
secure the node (API encryption key, OTA password, password on the web page), adopt it in
your ESPHome Device Builder: it shows up there as *discovered*; click **Take control**.

### B. Your own ESPHome Device Builder (advanced)

Use this when you want everything (Wi-Fi, encryption key, passwords, room) in your own
configuration.

1. Copy the `packages/` folder of this repository to
   `<esphome config>/packages/room-presence/` (in the Home Assistant add-on that is
   `/config/esphome/packages/room-presence/`).
2. Add these secrets to your ESPHome `secrets.yaml` (see
   [`examples/secrets.example.yaml`](../examples/secrets.example.yaml)):
   `wifi_ssid`, `wifi_password`, `room_node_api_key`, `room_node_ota_password`,
   `room_node_ap_password`, `room_node_web_password`.
3. Create one file per node, for example `room-living-room.yaml`:

   ```yaml
   substitutions:
     name: room-living-room          # hostname: a-z, 0-9 and '-'
     friendly_name: Room Living room # device name in Home Assistant
     room: Living room               # Home Assistant area, created if it does not exist

   packages:
     room_node: !include packages/room-presence/room-node.yaml
   ```

4. In the Device Builder: open the node, **Install → Plug into this computer** for the first
   installation. After that, updates go over Wi-Fi.

On Windows, `tools/Deploy-ToHomeAssistant.ps1` does steps 1–3 for you: it copies the
packages, generates the secrets and asks in which room the node goes.

```powershell
.\tools\Deploy-ToHomeAssistant.ps1 -EsphomeDir \\homeassistant.local\config\esphome
```

## 4. Add the node to Home Assistant and choose the room

1. Home Assistant shows **New devices discovered**. Go to
   *Settings → Devices & services*, find *Room node …* (web installer) or your own name
   (option B) and click **Add**.
2. With option A: in the last step, choose the **area**, which is the room where the node
   will be. You can type a new area. With option B the area is already set from `room:`.
3. The node is now a Bluetooth scanner for Home Assistant. You can check this under
   *Settings → Devices & services → Bluetooth*.

**Changing the room later**

- Easiest: open the node's device page in Home Assistant, click the **pencil icon** and
  change the **area**. Bermuda uses the new area right away.
- With option B you can also change `room:` (and `friendly_name:`) in the node's YAML file
  and install again, so the file stays the source of truth.
- The hostname (`name:`) can be changed with **Rename** in the Device Builder's menu for
  that node. This is optional; the hostname is only a technical name.

## 5. The node's web page

Every node has its own web page: open `http://<ip-address-of-the-node>/` in a browser. You
find the IP address on the device page in Home Assistant (sensor *IP address*) or in your
router.

- **Option B (own configuration)**: username `admin`, password = `room_node_web_password`
  from your ESPHome secrets (Device Builder → *Secrets*).
- **Option A (web installer)**: no password. Adopt the node (chapter 3A) if you want one.

The page shows all sensors and the **live log**. To see what the node receives:

- **Bluetooth devices in range**: how many different Bluetooth devices the node heard in the
  last 60 seconds;
- **Nearest Bluetooth devices**: the five strongest signals, with their name, `iBeacon
  major/minor` or address and the signal strength in dBm (closer to 0 = closer by). A phone
  with the Home Assistant BLE transmitter shows up as `iBeacon 100/40004` or similar.

Tip: `-40` to `-60` dBm is very close (same room), below `-85` dBm is far away.

## 6. Track people with Bermuda

### 6.1 Install Bermuda

Install **Bermuda BLE Trilateration** from HACS, restart Home Assistant and add the
integration (*Settings → Devices & services → Add integration → Bermuda*). In its options
you see how many scanners (room nodes) and devices it found.

### 6.2 Make the devices recognisable

Phones change their Bluetooth address all the time for privacy. They have to send a fixed
signal, or Home Assistant has to know their key.

**Android phones and Wear OS watches** (Home Assistant Companion app):

1. *Settings → Companion app → Manage sensors → BLE Transmitter*: switch it **on**.
2. Set **Transmit power** to *High* and **Advertise mode** to *Low latency* (or *Balanced*
   to save battery). The defaults (*Ultra low*, *Low power*) are too weak for room detection.
3. *Beacon Monitor* is **not** needed; it does the opposite (the phone looks for beacons).

**iPhone and Apple Watch**

iOS cannot send a fixed beacon. Instead you give Home Assistant the phone's *IRK* (identity
resolving key), so it can recognise the changing addresses:

1. Obtain the IRK of the iPhone. The documentation of the
   [Private BLE Device](https://www.home-assistant.io/integrations/private_ble_device/)
   integration lists the ways to do this.
2. Add the **Private BLE Device** integration and paste the IRK.

**Bluetooth tags** with a fixed address can be selected directly.

### 6.3 Select the devices in Bermuda

1. Bermuda → **Configure → Select Devices**. Pick the phones/watches/tags to track (for a
   phone with the BLE transmitter: the entry that starts with `iBeacon:`; for an iPhone: the
   Private BLE Device).
2. Bermuda creates a device per selection with, among others:
   - `sensor.<device>_area`: the room the device is in;
   - `sensor.<device>_distance`: estimated distance to the nearest node;
   - `device_tracker.<device>_bermuda_tracker`: home / away (can be added to a *person*).
3. Give the device a clear name (device page → pencil icon), for example *Phone Anna*.

### 6.4 Fine-tuning

- **Max radius** (global options): devices further away than this are "not in a room".
- **Calibration 2: scanner RSSI offsets**: if one node consistently "pulls" devices from the
  room next door, give it a negative offset.
- Add nodes to rooms where people are often placed in the wrong room.

### 6.5 Use it

Examples: turn off the lights when nobody's device has been in a room for 10 minutes, start the
heating in the office when a specific phone is there, show "who is where" on a dashboard
(an entities card with all `sensor.*_area` sensors).

## 7. Updating

- **Option A**: new firmware versions appear as an **update entity** (*Firmware*) on the
  node's device page in Home Assistant. Click *Install*.
- **Option B**: the Device Builder shows *Update* when ESPHome or the packages changed.
  Copy new package versions first (or run the Deploy script again).

## 8. Troubleshooting

| Problem | Solution |
| --- | --- |
| No serial port in the installer | Install the USB driver (CH340 or CP210x), try another (data) cable or USB port. |
| "Failed to connect" while flashing | Hold the **BOOT** button on the board while clicking Connect, release when flashing starts. |
| Node not discovered by Home Assistant | Home Assistant and the node must be on the same network (mDNS). Otherwise add it manually: *Add integration → ESPHome* → IP address of the node. |
| Node cannot connect to Wi-Fi | It opens a hotspot named after the node. Connect to it with your phone and enter the Wi-Fi details (captive portal). Use 2.4 GHz. |
| Phone is not seen | Check the BLE transmitter settings (6.2) and look at *Nearest Bluetooth devices* on the node's web page. |
| Device jumps between rooms | Add nodes, use Bermuda's calibration (6.4), place nodes away from shared walls. |
| Node restarts / *Free heap* very low | Use the individual packages without `web-server.yaml` (option B). |

## 9. Reference: entities and settings

**Entities per node**

| Entity | Description |
| --- | --- |
| Wi-Fi signal / Wi-Fi signal quality | RSSI in dBm and as 0–100 % |
| Access point (BSSID), Wi-Fi channel, Wi-Fi SSID, IP address, MAC address | Connection details |
| Wi-Fi disconnects | Number of drops since boot |
| Bluetooth devices in range, Nearest Bluetooth devices* | What the node hears |
| Bluetooth scanning (switch) | Pause BLE scanning |
| Room (option B) | The configured room |
| Last restart, Reset reason, Chip temperature, Free heap*, Loop time* | Device health |
| Restart, Restart (safe mode) | Maintenance buttons |
| Firmware (option A) | Firmware update |

\* Disabled by default in Home Assistant to keep the database small (they change all the time). They are always visible on the node's web page; enable them on the device page in Home Assistant if you want history.

**Substitutions (option B)**

| Substitution | Default | Purpose |
| --- | --- | --- |
| `name` | `room-node` | Hostname |
| `friendly_name` | `Room node` | Device name |
| `room` | `Room` | Home Assistant area |
| `board` | `esp32dev` | Board id (`esp32-s3-devkitc-1`, `esp32-c3-devkitm-1`, …) |
| `log_level` | `INFO` | Log level |
| `wifi_update_interval` | `30s` | Update interval of the Wi-Fi sensors |
| `wifi_signal_delta` | `2` | Minimum change (dB) before the Wi-Fi signal is sent |
| `ble_scan_interval` / `ble_scan_window` | `1100ms` | BLE scan timing (continuous) |
| `ble_monitor_window_ms` | `60000` | How long a device counts as "in range" |
| `ble_monitor_update_interval` | `30s` | Update interval of the Bluetooth monitor |

**Packages**

| File | Contents |
| --- | --- |
| `core.yaml` | Board, Wi-Fi with fallback hotspot, API, buttons (no secrets) |
| `credentials.yaml` | Your Wi-Fi, encryption key and passwords (from secrets) |
| `wifi-monitor.yaml` | Wi-Fi and health sensors |
| `ble-presence.yaml` | Bluetooth proxy + Bluetooth monitor |
| `web-server.yaml` | Web page |
| `room-node.yaml` | All of the above + room (`area`) |
