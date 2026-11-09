# Roadmap

## v1.0 (current)
- [x] ESPHome packages: core, Wi-Fi monitor, Bluetooth presence (active proxy)
- [x] Room = free setting, becomes the Home Assistant area
- [x] Windows helper `tools/Deploy-ToHomeAssistant.ps1` (packages, secrets, device file)
- [x] First node live (werkkamer, 2026-11-09): flashed, adopted in HA, registered as Bluetooth scanner
- [ ] Bermuda configured + phones tracked
- [ ] Tag v1.0.0

## Next
- [ ] **Web installer**: GitHub Action builds a generic firmware, GitHub Pages hosts an
      ESP Web Tools page (flash from the browser, Wi-Fi via Improv), room chosen in the
      Home Assistant "new device" dialog. Requires a public repository.
- [ ] `dashboard_import` so web-installed nodes can be adopted in the Device Builder
- [ ] Example Home Assistant dashboard: who is where + Wi-Fi heat map per room
- [ ] Example automations / blueprint (lights off when a room is empty, ...)
- [ ] Optional on-device presence per person (`ble_rssi` with IRK) as an alternative to Bermuda
- [ ] Optional mmWave radar (LD2410/LD2450) package for presence of people without a phone
- [ ] Board presets for ESP32-S3 / ESP32-C3 and common enclosures
