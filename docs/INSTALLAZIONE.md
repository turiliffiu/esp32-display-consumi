# Installazione

## Prerequisiti

- ESP32 DevKit
- Display OLED SSD1306 128x64 (I2C)
- ESPHome CLI installato sull'ambiente di deploy (Server #2)
- Home Assistant raggiungibile in rete dall'ESP32

## 1. Collegamento hardware

| ESP32   | SSD1306 |
|---------|---------|
| 3.3V    | VCC     |
| GND     | GND     |
| GPIO21  | SDA     |
| GPIO22  | SCL     |

## 2. Configurazione segreti

```bash
cp src/secrets.yaml.example src/secrets.yaml
```

Compila `src/secrets.yaml` con:
- SSID e password WiFi di casa
- Chiave di cifratura API (generabile con `esphome secrets` o un generatore base64 a 32 byte)
- Password OTA a tua scelta

## 3. Validazione configurazione

```bash
esphome compile src/display-consumi-casa.yaml
```

Verifica che la compilazione termini senza errori prima di procedere.

## 4. Flash iniziale

Collega l'ESP32 via USB al Server #2, poi:

```bash
esphome run src/display-consumi-casa.yaml
```

Alla prima installazione ESPHome chiederà la porta seriale (es. `/dev/ttyUSB0`).

## 5. Verifica in Home Assistant

Dopo il primo flash, l'ESP32 dovrebbe comparire automaticamente in *Impostazioni → Dispositivi e servizi* come nuova integrazione ESPHome (via mDNS). Se non compare entro un minuto, controlla che HA e l'ESP32 siano sulla stessa subnet.

## 6. Aggiornamenti successivi (OTA)

Una volta flashato via USB la prima volta, gli aggiornamenti successivi possono avvenire via WiFi:

```bash
esphome run src/display-consumi-casa.yaml --device display-consumi-casa.local
```
