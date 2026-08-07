# ESP32 Display Consumi Casa

Display OLED integrato su ESP32-C3 che mostra in tempo reale il consumo elettrico generale della casa, letto da Home Assistant.

## Hardware

- Scheda **01space ESP32-C3 0.42 OLED** (o clone equivalente) — display OLED SSD1306 già integrato sul PCB, nessun collegamento esterno necessario
- Risoluzione display: 72×40 pixel
- I2C: SDA=GPIO5, SCL=GPIO6 (pin fissi di fabbrica)

## Come funziona

Il firmware ESPHome si collega al WiFi di casa ed espone l'API nativa verso Home Assistant. Legge il valore di `sensor.shellyem_c7f7a1_channel_1_power` (misuratore Shelly EM già configurato in HA) e lo mostra sul display OLED integrato, aggiornato ogni 2 secondi.

## Quick start

1. Copia `src/secrets.yaml.example` in `src/secrets.yaml` e compila con le tue credenziali
2. Valida la configurazione: `esphome config src/display-consumi-casa.yaml`
3. Flash: `esphome run src/display-consumi-casa.yaml`

Guida completa passo-passo in `docs/INSTALLAZIONE.md`.

## Struttura progetto

Vedi `docs/ARCHITETTURA.md`.

## Problemi noti

Vedi `docs/TROUBLESHOOTING.md`.

## Changelog

Vedi `CHANGELOG.md`.
