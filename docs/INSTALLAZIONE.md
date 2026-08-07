# Installazione

## Prerequisiti

- Scheda 01space ESP32-C3 0.42 OLED (display integrato, non serve hardware aggiuntivo)
- Cavo USB-C dati (non solo-carica — verificare prima di procedere, causa comune di mancato riconoscimento)
- ESPHome CLI installato sull'ambiente di deploy (Server #2), in un virtualenv Python dedicato
- Home Assistant raggiungibile in rete dall'ESP32

## 1. Hardware

Nessun collegamento manuale necessario: il display OLED è già saldato sul PCB, collegato internamente su:
- SDA: GPIO5
- SCL: GPIO6

Questi pin sono di fabbrica e non vanno modificati nella configurazione.

## 2. Setup ambiente ESPHome

```bash
python3 -m venv ~/venv-esphome
source ~/venv-esphome/bin/activate
pip install esphome
esphome version
```

## 3. Configurazione segreti

```bash
cp src/secrets.yaml.example src/secrets.yaml
```

Compila `src/secrets.yaml` con:
- SSID e password WiFi di casa
- Chiave di cifratura API: genera con
```bash
  python3 << 'PYEOF'
  import base64, os
  print(base64.b64encode(os.urandom(32)).decode())
  PYEOF
```
- Password OTA a tua scelta

## 4. Validazione configurazione

```bash
esphome config src/display-consumi-casa.yaml
```

Deve terminare con `INFO Configuration is valid!`.

## 5. Identificazione porta seriale

```bash
lsusb
ls /dev/ttyACM* /dev/ttyUSB* 2>/dev/null
```

La scheda espone un'interfaccia USB JTAG/serial nativa Espressif (vendor ID `303a:1001`), tipicamente su `/dev/ttyACM0` (non `/dev/ttyUSB0`).

Verifica di essere nel gruppo `dialout`:
```bash
groups $USER
```

## 6. Flash iniziale

```bash
esphome run src/display-consumi-casa.yaml
```

Seleziona la porta seriale identificata al passo 5 quando richiesto.

## 7. Aggiunta integrazione in Home Assistant

1. *Impostazioni → Dispositivi e servizi* — il device dovrebbe comparire in auto-discovery (mDNS) come "display-consumi-casa"
2. Se non compare, aggiungi manualmente l'integrazione ESPHome inserendo `display-consumi-casa.local` o l'IP diretto
3. Inserisci la chiave di cifratura API (`api_encryption_key` da `secrets.yaml`) quando richiesta

## 8. Verifica finale

Il display dovrebbe mostrare il valore reale in Watt entro pochi secondi dall'autenticazione con HA. Se resta su `-- W`, vedi `TROUBLESHOOTING.md`.

## 9. Aggiornamenti successivi (OTA)

```bash
esphome run src/display-consumi-casa.yaml --device display-consumi-casa.local
```
