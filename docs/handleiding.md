# Handleiding: ESPHome Room Presence

*English version: [manual.md](manual.md)*

Deze handleiding brengt je van een kaal ESP32-bordje naar "Home Assistant weet wie in welke
ruimte is".

1. [Wat het doet](#1-wat-het-doet)
2. [Hardware en plaatsing](#2-hardware-en-plaatsing)
3. [Firmware installeren](#3-firmware-installeren)
4. [Module toevoegen aan Home Assistant en de ruimte kiezen](#4-module-toevoegen-aan-home-assistant-en-de-ruimte-kiezen)
5. [De webpagina van de module](#5-de-webpagina-van-de-module)
6. [Personen volgen met Bermuda](#6-personen-volgen-met-bermuda)
7. [Bijwerken](#7-bijwerken)
8. [Problemen oplossen](#8-problemen-oplossen)
9. [Naslag: entities en instellingen](#9-naslag-entities-en-instellingen)

---

## 1. Wat het doet

Elke ruimte krijgt één klein ESP32-bordje, een *room node* (ruimtemodule). Zo'n module doet
twee dingen:

- **Bluetooth-aanwezigheid.** De module luistert naar de Bluetooth-signalen die telefoons,
  smartwatches en tags steeds uitzenden, en stuurt die (met signaalsterkte) door naar Home
  Assistant. De integratie [Bermuda](https://github.com/agittins/bermuda) vergelijkt het
  signaal van een apparaat bij alle modules en zet het apparaat in de ruimte van de
  dichtstbijzijnde module.
- **Wifi-monitoring.** De module meldt de wifi-signaalsterkte, met welk access point en kanaal
  hij verbonden is en hoe vaak de verbinding wegvalt. Zo krijg je een wifi-kaart van je huis.

Alle gegevens blijven in je eigen netwerk.

**Goed om te weten**

- Bermuda volgt *apparaten*, geen mensen. Een telefoon die op de bank ligt, "is in de woonkamer".
- Bluetooth gaat door muren heen. Het werkt goed met een module in (bijna) elke ruimte; met
  één of twee modules weet je vooral "bij deze module of niet".
- Een wisseling van ruimte wordt binnen een paar seconden tot ongeveer een halve minuut opgemerkt.

## 2. Hardware en plaatsing

| Geschikt | Niet geschikt |
| --- | --- |
| ESP32 (bijv. ESP32-WROOM-32 DevKit, ca. €5), ESP32-S3, ESP32-C3 | ESP8266 / NodeMCU / Wemos D1 (geen Bluetooth) |

Je hebt ook een **USB-datakabel** nodig voor de eerste installatie, en per module een
**USB-telefoonlader**.

Plaatsing:

- één module per ruimte, op ongeveer 1–2 m hoogte (een plank of kast is prima);
- niet direct tegen een muur die je deelt met een andere ruimte;
- uit de buurt van grote metalen voorwerpen (koelkast, radiator, metalen kast);
- gebruik overal hetzelfde type bordje, dan zijn de signaalsterktes vergelijkbaar.

## 3. Firmware installeren

Er zijn twee manieren. Kies **A**, tenzij je al je eigen ESPHome-configuraties beheert.

### A. Webinstaller (aanbevolen, geen software nodig)

1. Open de installatiepagina: **https://schmeitz.github.io/esphome-room-presence/** in
   **Chrome of Edge op een computer**.
2. Steek het bordje in je computer en klik op **Installeren op ESP32**. Kies de seriële poort
   van het bordje (meestal *USB-SERIAL CH340*, *CP210x* of *USB JTAG*).
3. Bevestig de installatie. De juiste firmware voor je chip wordt automatisch gekozen. Het
   flashen duurt ongeveer een minuut.
4. Vul daarna de **naam en het wachtwoord van je wifi** in (2,4 GHz). Het bordje maakt verbinding
   en verschijnt in Home Assistant (zie hoofdstuk 4).

De kant-en-klare firmware bevat geen wachtwoorden, zodat iedereen hem kan installeren. Wil je
de module beveiligen (API-sleutel, OTA-wachtwoord, wachtwoord op de webpagina), neem hem dan
over in je ESPHome Device Builder: hij staat daar als *ontdekt*; klik op **Take control**.

### B. Je eigen ESPHome Device Builder (gevorderd)

Gebruik dit als je alles (wifi, sleutel, wachtwoorden, ruimte) in je eigen configuratie wilt.

1. Kopieer de map `packages/` uit deze repository naar
   `<esphome-config>/packages/room-presence/` (bij de Home Assistant-add-on is dat
   `/config/esphome/packages/room-presence/`).
2. Zet deze secrets in je ESPHome-`secrets.yaml` (zie
   [`examples/secrets.example.yaml`](../examples/secrets.example.yaml)):
   `wifi_ssid`, `wifi_password`, `room_node_api_key`, `room_node_ota_password`,
   `room_node_ap_password`, `room_node_web_password`.
3. Maak per module een bestand, bijvoorbeeld `room-woonkamer.yaml`:

   ```yaml
   substitutions:
     name: room-woonkamer            # hostname: a-z, 0-9 en '-'
     friendly_name: Ruimtesensor Woonkamer  # apparaatnaam in Home Assistant
     room: Woonkamer                 # gebied in Home Assistant, wordt aangemaakt als het niet bestaat

   packages:
     room_node: !include packages/room-presence/room-node.yaml
   ```

4. In de Device Builder: open de module, **Install → Plug into this computer** voor de eerste
   installatie. Daarna gaan updates via wifi.

Op Windows doet `tools/Deploy-ToHomeAssistant.ps1` stap 1–3 voor je: het kopieert de
packages, maakt de secrets aan en vraagt in welke ruimte de module komt.

```powershell
.\tools\Deploy-ToHomeAssistant.ps1 -EsphomeDir \\homeassistant.local\config\esphome
```

## 4. Module toevoegen aan Home Assistant en de ruimte kiezen

1. Home Assistant meldt **Nieuwe apparaten ontdekt**. Ga naar
   *Instellingen → Apparaten & diensten*, zoek *Room node …* (webinstaller) of je eigen naam
   (optie B) en klik op **Toevoegen**.
2. Bij optie A: kies in de laatste stap het **gebied**, dat is de ruimte waar de module komt.
   Je kunt ook een nieuw gebied typen. Bij optie B is het gebied al ingesteld via `room:`.
3. De module is nu een Bluetooth-scanner voor Home Assistant. Controleer dat onder
   *Instellingen → Apparaten & diensten → Bluetooth*.

**De ruimte later wijzigen**

- Wijzig het **gebied** in Home Assistant. Het gebied is wat telt voor Bermuda. Wijzig het
  bij **twee** apparaten, want Home Assistant toont de module twee keer:
  1. het ESPHome-apparaat (*Instellingen → Apparaten & diensten → ESPHome*, module openen,
     potloodicoon, gebied);
  2. het Bluetooth-scannerapparaat, met de naam van de module en het Bluetooth-MAC-adres
     erachter (*Instellingen → Apparaten & diensten → Bluetooth*). Bermuda leest het gebied
     van dit apparaat.

  Herlaad daarna Bermuda (*Instellingen → Apparaten & diensten → Bermuda*, ⋮,
  **Herladen**).
- Bij optie B pas je ook `room:` (en `friendly_name:`) aan in het YAML-bestand van de module
  en installeer je opnieuw, zodat het bestand leidend blijft. Dat alleen verplaatst een
  bestaand apparaat in Home Assistant **niet**: ESPHome zet het gebied alleen bij het
  toevoegen.
- Laat de hostname (`name:`) met rust. Dat is alleen een technische naam. Wil je hem toch
  wijzigen, gebruik dan **Rename** in het menu van de module in de Device Builder. Pas je
  `name:` aan in de YAML en klik je op Install, dan mislukt de upload met *device is
  offline*: de Device Builder zoekt de nieuwe naam, maar de module heeft nog de oude.
  Oplossing: installeer nog een keer en kies het IP-adres van de module als doel (of
  gebruik `esphome run <bestand> --device <ip-adres>`).

## 5. De webpagina van de module

Elke module heeft een eigen webpagina: open `http://<ip-adres-van-de-module>/` in een browser.
Het IP-adres vind je op de apparaatpagina in Home Assistant (sensor *IP address*) of in je router.

- **Optie B (eigen configuratie)**: gebruikersnaam `admin`, wachtwoord =
  `room_node_web_password` uit je ESPHome-secrets (Device Builder → *Secrets*).
- **Optie A (webinstaller)**: geen wachtwoord. Neem de module over (hoofdstuk 3A) als je er
  wel een wilt.

De pagina toont alle sensoren en de **live log**. Om te zien wat de module ontvangt:

- **Bluetooth devices in range**: hoeveel verschillende Bluetooth-apparaten de module de
  afgelopen 60 seconden heeft gehoord;
- **Nearest Bluetooth devices**: de vijf sterkste signalen, met naam, `iBeacon major/minor`
  of adres en de signaalsterkte in dBm (dichter bij 0 = dichterbij). Een telefoon met de
  BLE-zender van Home Assistant verschijnt bijvoorbeeld als `iBeacon 100/40004`.

Vuistregel: `-40` tot `-60` dBm is heel dichtbij (zelfde ruimte), onder `-85` dBm is ver weg.

## 6. Personen volgen met Bermuda

### 6.1 Bermuda installeren

Installeer **Bermuda BLE Trilateration** via HACS, herstart Home Assistant en voeg de
integratie toe (*Instellingen → Apparaten & diensten → Integratie toevoegen → Bermuda*). In de
opties zie je hoeveel scanners (modules) en apparaten hij heeft gevonden.

### 6.2 Apparaten herkenbaar maken

Telefoons wisselen voor de privacy steeds van Bluetooth-adres. Ze moeten dus een vast signaal
uitzenden, of Home Assistant moet hun sleutel kennen.

**Android-telefoons en Wear OS-horloges** (Home Assistant Companion-app):

1. *Instellingen → Companion app → Sensoren beheren → BLE-zender*: zet hem **aan**.
2. Zet **Zendvermogen** op *Hoog* en **Advertentiemodus** op *Lage latentie* (of
   *Gebalanceerd* om accu te sparen). De standaardwaarden (*Ultra laag*, *Laag
   energieverbruik*) zijn te zwak voor ruimtedetectie.
3. *Beacon Monitor* is **niet** nodig; die doet het omgekeerde (de telefoon zoekt naar beacons).

**iPhone en Apple Watch**

iOS kan geen vast beacon uitzenden. In plaats daarvan geef je Home Assistant de *IRK*
(identity resolving key) van de telefoon, zodat hij de wisselende adressen herkent:

1. Achterhaal de IRK van de iPhone. De documentatie van de integratie
   [Private BLE Device](https://www.home-assistant.io/integrations/private_ble_device/)
   beschrijft de manieren om dat te doen.
2. Voeg de integratie **Private BLE Device** toe en plak de IRK.

**Bluetooth-tags** met een vast adres kun je direct selecteren.

### 6.3 Apparaten selecteren in Bermuda

1. Bermuda → **Configureren → Select Devices**. Kies de telefoons/horloges/tags die je wilt
   volgen (bij een telefoon met BLE-zender: de regel die begint met `iBeacon:`; bij een
   iPhone: het Private BLE Device).
2. Bermuda maakt per keuze een apparaat met onder andere:
   - `sensor.<apparaat>_area`: de ruimte waar het apparaat is;
   - `sensor.<apparaat>_distance`: geschatte afstand tot de dichtstbijzijnde module;
   - `device_tracker.<apparaat>_bermuda_tracker`: thuis / weg (te koppelen aan een *persoon*).
3. Geef het apparaat een duidelijke naam (apparaatpagina → potloodicoon), bijvoorbeeld
   *Telefoon Anna*.

### 6.4 Bijstellen

- **Max radius** (globale opties): apparaten die verder weg zijn, zijn "in geen enkele ruimte".
- **Calibration 2: scanner RSSI offsets**: als één module steeds apparaten uit de kamer
  ernaast "trekt", geef hem dan een negatieve offset.
- Zet extra modules in ruimtes waar mensen vaak in de verkeerde ruimte worden geplaatst.

### 6.5 Toepassen

Voorbeelden: licht uit als er 10 minuten geen apparaat meer in een ruimte is, verwarming in de
werkkamer aan als een bepaalde telefoon daar is, "wie is waar" op een dashboard (een
entities-kaart met alle `sensor.*_area`-sensoren).

## 7. Bijwerken

- **Optie A**: nieuwe firmwareversies verschijnen als **update-entity** (*Firmware*) op de
  apparaatpagina van de module in Home Assistant. Klik op *Installeren*.
- **Optie B**: de Device Builder toont *Update* als ESPHome of de packages zijn veranderd.
  Kopieer eerst de nieuwe packages (of draai het Deploy-script opnieuw).

## 8. Problemen oplossen

| Probleem | Oplossing |
| --- | --- |
| Geen seriële poort in de installer | Installeer het USB-stuurprogramma (CH340 of CP210x), probeer een andere (data)kabel of USB-poort. |
| "Failed to connect" bij het flashen | Houd de **BOOT**-knop op het bordje ingedrukt terwijl je op Verbinden klikt; laat los als het flashen begint. |
| Module wordt niet ontdekt door Home Assistant | Home Assistant en de module moeten in hetzelfde netwerk zitten (mDNS). Anders handmatig toevoegen: *Integratie toevoegen → ESPHome* → IP-adres van de module. |
| Module komt niet op de wifi | Hij opent een hotspot met de naam van de module. Maak daar met je telefoon verbinding mee en vul de wifi-gegevens in (captive portal). Gebruik 2,4 GHz. |
| Telefoon wordt niet gezien | Controleer de instellingen van de BLE-zender (6.2) en kijk naar *Nearest Bluetooth devices* op de webpagina van de module. |
| Apparaat springt tussen ruimtes | Zet meer modules neer, gebruik de kalibratie van Bermuda (6.4), plaats modules niet tegen gedeelde muren. |
| Module herstart / *Free heap* erg laag | Gebruik de losse packages zonder `web-server.yaml` (optie B). |

## 9. Naslag: entities en instellingen

**Entities per module**

| Entity | Omschrijving |
| --- | --- |
| Wi-Fi signal / Wi-Fi signal quality | RSSI in dBm en als 0–100 % |
| Access point (BSSID), Wi-Fi channel, Wi-Fi SSID, IP address, MAC address | Verbindingsgegevens |
| Wi-Fi disconnects | Aantal onderbrekingen sinds opstarten |
| Bluetooth devices in range, Nearest Bluetooth devices* | Wat de module hoort |
| Bluetooth scanning (schakelaar) | BLE-scannen pauzeren |
| Room (optie B) | De ingestelde ruimte |
| Last restart, Reset reason, Chip temperature, Free heap*, Loop time* | Gezondheid van de module |
| Restart, Restart (safe mode) | Onderhoudsknoppen |
| Firmware (optie A) | Firmware-update |

\* Standaard uitgeschakeld in Home Assistant om de database klein te houden (ze veranderen voortdurend). Ze staan altijd op de webpagina van de module; zet ze aan op de apparaatpagina in Home Assistant als je historie wilt.

**Substitutions (optie B)**

| Substitution | Standaard | Doel |
| --- | --- | --- |
| `name` | `room-node` | Hostname |
| `friendly_name` | `Room node` | Apparaatnaam |
| `room` | `Room` | Gebied in Home Assistant |
| `board` | `esp32dev` | Board-id (`esp32-s3-devkitc-1`, `esp32-c3-devkitm-1`, …) |
| `log_level` | `INFO` | Logniveau |
| `wifi_update_interval` | `30s` | Interval van de wifi-sensoren |
| `wifi_signal_delta` | `2` | Minimale verandering (dB) voordat het wifi-signaal wordt doorgestuurd |
| `ble_scan_interval` / `ble_scan_window` | `1100ms` | BLE-scantiming (continu) |
| `ble_monitor_window_ms` | `60000` | Hoe lang een apparaat als "in bereik" telt |
| `ble_monitor_update_interval` | `30s` | Interval van de Bluetooth-monitor |

**Packages**

| Bestand | Inhoud |
| --- | --- |
| `core.yaml` | Board, wifi met noodhotspot, API, knoppen (geen secrets) |
| `credentials.yaml` | Jouw wifi, sleutel en wachtwoorden (uit secrets) |
| `wifi-monitor.yaml` | Wifi- en gezondheidssensoren |
| `ble-presence.yaml` | Bluetooth-proxy + Bluetooth-monitor |
| `web-server.yaml` | Webpagina |
| `room-node.yaml` | Alles hierboven + ruimte (`area`) |
