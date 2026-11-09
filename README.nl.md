# ESPHome Room Presence

*English: [README.md](README.md)*

Maak van goedkope ESP32-bordjes **ruimtemodules** voor Home Assistant: één per ruimte, elk een
**Bluetooth-aanwezigheidsscanner** en een **wifi-monitor**. Samen vertellen ze *wie in welke
ruimte is* (via de telefoons, horloges of tags die mensen bij zich hebben) en hoe goed je wifi
overal in huis is.

**[→ Installeren vanuit je browser](https://schmeitz.github.io/esphome-room-presence/)** ·
**[Handleiding](docs/handleiding.md)** · [Manual (EN)](docs/manual.md)

## Functies

- **Aanwezigheid per ruimte**: elke module is een actieve ESPHome-Bluetooth-proxy. Home
  Assistant + [Bermuda BLE Trilateration](https://github.com/agittins/bermuda) zetten elke
  gevolgde telefoon, horloge of tag in het gebied van de dichtstbijzijnde module.
- **Wifi-monitoring**: signaalsterkte (dBm en %), access point (BSSID), kanaal, aantal
  onderbrekingen, uptime, chiptemperatuur en vrij geheugen, per ruimte.
- **Bluetooth-monitor**: zie hoeveel apparaten een module hoort en de vijf sterkste signalen.
- **Ingebouwde webpagina** op elke module met alle sensoren en de live log.
- **Vrije keuze van de ruimte**: één firmware voor elke module; je kiest de ruimte (gebied in
  Home Assistant) bij het toevoegen, en kunt die altijd wijzigen.
- **Twee manieren van installeren**: een browserinstaller voor iedereen, of gewone
  ESPHome-packages voor je eigen Device Builder.

## Snel aan de slag

1. Neem een ESP32-, ESP32-S3- of ESP32-C3-bordje (een ESP8266 heeft geen Bluetooth).
2. Open de [webinstaller](https://schmeitz.github.io/esphome-room-presence/) in Chrome of
   Edge, klik op **Installeren** en vul je wifi in.
3. Voeg de ontdekte module toe in Home Assistant en kies de ruimte.
4. Installeer Bermuda en maak je telefoons herkenbaar (Android: BLE-zender in de
   Companion-app; iPhone: Private BLE Device).

Alles staat stap voor stap in de [handleiding](docs/handleiding.md).

## Je eigen ESPHome-configuratie

Beheer je de modules liever zelf (eigen wifi-secrets, sleutel, wachtwoorden)? Kopieer
`packages/` naar `<esphome-config>/packages/room-presence/` en maak per ruimte een klein bestand:

```yaml
substitutions:
  name: room-woonkamer
  friendly_name: Ruimtesensor Woonkamer
  room: Woonkamer            # gebied in Home Assistant

packages:
  room_node: !include packages/room-presence/room-node.yaml
```

Zie [hoofdstuk 3B van de handleiding](docs/handleiding.md#b-je-eigen-esphome-device-builder-gevorderd)
en het Windows-hulpscript `tools/Deploy-ToHomeAssistant.ps1`.

## Privacy

Alle gegevens blijven in je eigen netwerk: de modules luisteren alleen naar
Bluetooth-signalen en sturen die naar je eigen Home Assistant. Volg alleen mensen die
daarvan weten en ermee instemmen.

## Licentie

[MIT](LICENSE)
