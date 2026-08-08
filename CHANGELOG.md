# Changelog

Tutte le modifiche rilevanti di questo progetto sono documentate qui.

Il formato segue [Keep a Changelog](https://keepachangelog.com/it/1.0.0/),
e il progetto aderisce a [Semantic Versioning](https://semver.org/lang/it/).

## [1.0.0] - 2026-08-07

### Added
- Struttura iniziale del progetto (README, .gitignore, cartelle docs/src/scripts/backup)
- Configurazione ESPHome per display consumi casa, con lettura di `sensor.shellyem_c7f7a1_channel_1_power` via API nativa Home Assistant
- Primo flash e validazione funzionante su hardware reale

### Fixed
- Identificata scheda reale come **01space ESP32-C3 0.42 OLED** (non ESP32 classico, non modulo OLED esterno): display integrato sul PCB, chip ESP32-C3, interfaccia USB nativa JTAG/serial
- Corretto `esp32.board` da `esp32dev` a `esp32-c3-devkitm-1`
- Corretti pin I2C da GPIO21/22 a **GPIO5 (SDA) / GPIO6 (SCL)**, specifici di questo modello
- Risolto rumore grafico sul display: sostituito l'approccio con offset manuale (`offset_x: 30, offset_y: 12` su buffer 128x64) con il modello nativo ESPHome **"SSD1306 72x40"**, supportato direttamente dal componente `ssd1306_i2c` senza bisogno di offset

### Known issues
- Alcuni cloni di questa scheda montano in realtà un chip SH1107 anziché un vero SSD1306, che con il driver sbagliato può causare rendering parzialmente corrotto — se il problema dovesse ripresentarsi dopo un cambio scheda, verificare il datasheet del controller reale

## [1.1.0] - 2026-08-08

### Added
- Secondo display: scheda **ESP8266 HW-364A** (OLED 128x64 integrato), stesso dato `sensor.shellyem_c7f7a1_channel_1_power`
- File `src/display-consumi-hw364a.yaml` — nodo separato (`display-consumi-hw364a`), stesso `secrets.yaml` condiviso (stessa rete WiFi)
- I2C su GPIO14 (SDA) / GPIO12 (SCL), board ESPHome `nodemcuv2`, layout con titolo "Consumo Casa" + valore in Watt
- Flash funzionante al primo tentativo, nessun problema di rumore grafico riscontrato su questo modello
