# Roadmap

## v1.2 (current)
- [x] ESPHome packages: core, credentials, Wi-Fi monitor, Bluetooth presence (active proxy), web page
- [x] Room = free setting, becomes the Home Assistant area
- [x] Bluetooth monitor sensors (devices in range, nearest devices)
- [x] Windows helper `tools/Deploy-ToHomeAssistant.ps1` (packages, secrets, device file)
- [x] Pre-built firmware for ESP32 / ESP32-S3 / ESP32-C3 built by GitHub Actions
- [x] Web installer page (ESP Web Tools, Improv Wi-Fi), EN/NL
- [x] Manual EN/NL
- [x] First node live (werkkamer, 2026-11-09), Bermuda tracking one phone
- [x] Repository public + GitHub Pages: https://schmeitz.github.io/esphome-room-presence/
- [x] Database-friendly sensors (averaging, delta filters, restart timestamp)
- [ ] Test the web installer with a fresh board
- [ ] Tag v1.2.0

## Next
- [ ] Example Home Assistant dashboard: who is where + Wi-Fi heat map per room
- [ ] Example automations / blueprint (lights off when a room is empty, ...)
- [ ] Optional IRK capture mode to make adding iPhones easier
- [ ] Optional mmWave radar (LD2410/LD2450) package for presence of people without a phone
- [ ] Optional on-device presence per person (`ble_rssi` with IRK) as an alternative to Bermuda
- [ ] Board presets and enclosures
