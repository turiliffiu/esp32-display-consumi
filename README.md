# ESP32 Display Consumi Casa

Display OLED collegato a ESP32 che mostra in tempo reale il consumo elettrico generale della casa, letto da Home Assistant.

## Hardware

- ESP32 DevKit
- Display OLED SSD1306 128x64 (I2C)

## Collegamento

| ESP32   | SSD1306 |
|---------|---------|
| 3.3V    | VCC     |
| GND     | GND     |
| GPIO21  | SDA     |
| GPIO22  | SCL     |

## Come funziona

Il firmware ESPHome si collega al WiFi di casa ed espone l'API nativa verso Home Assistant. Legge il valore di `sensor.shellyem_c7f7a1_channel_1_power` (misuratore Shelly EM già configurato in HA) e lo mostra sul display OLED, aggiornato ogni 2 secondi.

## Quick start

1. Copia `src/secrets.yaml.example` in `src/secrets.yaml` e compila con le tue credenziali
2. Valida la configurazione: `esphome compile src/display-consumi-casa.yaml`
3. Flash: `esphome run src/display-consumi-casa.yaml`

## Struttura progetto

Vedi `docs/ARCHITETTURA.md`.

## Changelog

Vedi `CHANGELOG.md`.
