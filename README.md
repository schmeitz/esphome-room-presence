# ESPHome Room Presence

*Nederlands: [README.nl.md](README.nl.md)*

Turn cheap ESP32 boards into **room nodes** for Home Assistant: one per room, each one a
**Bluetooth presence scanner** and a **Wi-Fi monitor**. Together they tell you *who is in
which room* (via the phones, watches or tags people carry) and how good your Wi-Fi is
everywhere in the house.

**[→ Install from your browser](https://schmeitz.github.io/esphome-room-presence/)** ·
**[Manual](docs/manual.md)** · [Handleiding (NL)](docs/handleiding.md)

## Features

- **Room presence**: every node is an active ESPHome Bluetooth proxy. Home Assistant +
  [Bermuda BLE Trilateration](https://github.com/agittins/bermuda) place each tracked phone,
  watch or tag in the area of the closest node.
- **Wi-Fi monitoring**: signal strength (dBm and %), access point (BSSID), channel, number
  of drop-outs, uptime, chip temperature and free memory, per room.
- **Bluetooth monitor**: see how many devices a node hears and the five strongest signals.
- **Built-in web page** on every node with all sensors and the live log.
- **Free room choice**: one firmware for every node; you choose the room (Home Assistant
  area) when you add the node, and can change it at any time.
- **Two ways to install**: a browser installer for everyone, or plain ESPHome packages for
  your own Device Builder.

## Quick start

1. Get an ESP32, ESP32-S3 or ESP32-C3 board (an ESP8266 has no Bluetooth).
2. Open the [web installer](https://schmeitz.github.io/esphome-room-presence/) in Chrome or
   Edge, click **Install** and enter your Wi-Fi.
3. Add the discovered node in Home Assistant and choose its room.
4. Install Bermuda and make your phones recognisable (Android: BLE transmitter in the
   Companion app; iPhone: Private BLE Device).

Everything is explained step by step in the [manual](docs/manual.md).

## Your own ESPHome configuration

Prefer to manage the nodes yourself (own Wi-Fi secrets, encryption key, passwords)? Copy
`packages/` to `<esphome config>/packages/room-presence/` and create one small file per room:

```yaml
substitutions:
  name: room-living-room
  friendly_name: Room Living room
  room: Living room          # Home Assistant area

packages:
  room_node: !include packages/room-presence/room-node.yaml
```

See [chapter 3B of the manual](docs/manual.md#b-your-own-esphome-device-builder-advanced)
and the Windows helper `tools/Deploy-ToHomeAssistant.ps1`.

## Repository layout

| Path | Contents |
| --- | --- |
| `packages/` | ESPHome packages (core, credentials, Wi-Fi monitor, Bluetooth presence, web page) |
| `firmware/` | Configurations of the pre-built firmware per chip (used by the web installer) |
| `site/` | The web installer page (published on GitHub Pages) |
| `examples/` | Example device files and secrets |
| `tools/` | Windows helper to deploy to Home Assistant |
| `docs/` | Manual (EN/NL) and roadmap |
| `.github/workflows/` | Builds the firmware and publishes the installer |

## Privacy

All data stays in your own network: the nodes only listen to Bluetooth advertisements and
send them to your own Home Assistant. Only track people who know about it and agree to it.

## License

[MIT](LICENSE)
