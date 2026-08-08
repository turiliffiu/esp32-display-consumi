# ESP32/ESP8266 Display Consumi Casa

Display OLED che mostrano in tempo reale il consumo elettrico generale della casa, letto da Home Assistant. Due dispositivi indipendenti, stesso dato, punti diversi della casa.

## Dispositivi

| Nome nodo | Hardware | Display | File |
|---|---|---|---|
| `display-consumi-casa` | 01space ESP32-C3 0.42 OLED | 72×40 px, integrato | `src/display-consumi-casa.yaml` |
| `display-consumi-hw364a` | ESP8266 HW-364A | 128×64 px, integrato (bicolore) | `src/display-consumi-hw364a.yaml` |

Dettagli hardware, pin I2C e note tecniche complete in `docs/ARCHITETTURA.md`.

## Come funziona

Ogni firmware ESPHome si collega al WiFi di casa ed espone l'API nativa verso Home Assistant. Legge il valore di `sensor.shellyem_c7f7a1_channel_1_power` (misuratore Shelly EM già configurato in HA) e lo mostra sul proprio display OLED integrato, aggiornato ogni 2 secondi.

## Quick start

1. Copia `src/secrets.yaml.example` in `src/secrets.yaml` e compila con le tue credenziali (condiviso tra entrambi i dispositivi, stessa rete WiFi)
2. Valida la configurazione:
```bash
   esphome config src/display-consumi-casa.yaml
   esphome config src/display-consumi-hw364a.yaml
```
3. Flash del dispositivo desiderato:
```bash
   esphome run src/display-consumi-casa.yaml
   # oppure
   esphome run src/display-consumi-hw364a.yaml
```

Guida completa passo-passo in `docs/INSTALLAZIONE.md`.

## Struttura progetto

Vedi `docs/ARCHITETTURA.md`.

## Problemi noti

Vedi `docs/TROUBLESHOOTING.md`.

## Changelog

Vedi `CHANGELOG.md`.
