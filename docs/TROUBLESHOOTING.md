# Troubleshooting

## Il display resta spento / non si accende

1. Verifica alimentazione: l'ESP32 riceve corrente via USB-C? LED rosso "PWR" acceso?
2. Il display è integrato sul PCB (SDA=GPIO5, SCL=GPIO6) — non ci sono cavi esterni da controllare su questo modello.
3. Controlla il log seriale per errori I2C:
```bash
   esphome logs src/display-consumi-casa.yaml
```

## Il display mostra rumore grafico / simboli strani

Sintomo riscontrato e risolto in questo progetto. Causa: uso del modello generico `"SSD1306 128x64"` con offset manuale (`offset_x`/`offset_y`) invece del modello nativo per pannelli 72×40.

**Fix:** nella sezione `display:` di `src/display-consumi-casa.yaml`, usa:
```yaml
display:
  - platform: ssd1306_i2c
    model: "SSD1306 72x40"
    address: 0x3C
```
senza `offset_x`/`offset_y` — il componente ESPHome gestisce internamente la finestra corretta.

Se il rumore persiste anche con questo modello, il chip driver reale potrebbe essere un SH1107 invece di un SSD1306 (variante nota su alcuni cloni economici) — verificare il datasheet specifico della scheda.

## Il display si accende ma mostra solo "-- W"

Significa che il sensore `homeassistant` non ha ancora ricevuto uno stato valido da HA — comportamento normale finché l'integrazione ESPHome non è stata aggiunta/autenticata in Home Assistant.

1. Verifica che l'ESP32 sia effettivamente connesso a HA: *Impostazioni → Dispositivi e servizi → ESPHome* → il dispositivo deve risultare "Connesso".
2. Controlla che `sensor.shellyem_c7f7a1_channel_1_power` esista e abbia un valore in HA: *Strumenti per sviluppatori → Stati*.
3. Controlla i log ESP32 per errori di autenticazione API — un sintomo tipico è:

[W][api.connection]: Socket operation failed HANDSHAKESTATE_READ_FAILED

   Questo indica che qualcosa (di solito HA) sta tentando la connessione con una chiave `api_encryption_key` errata o mancante. Verifica che la chiave in `secrets.yaml` coincida esattamente con quella inserita in HA al momento dell'aggiunta dell'integrazione.

## L'ESP32 non compare in Home Assistant dopo il flash

1. Verifica che ESP32 e HA siano sulla stessa subnet/VLAN (mDNS non attraversa subnet diverse senza reflector).
2. Prova ad aggiungere manualmente l'integrazione ESPHome in HA inserendo l'IP dell'ESP32 invece di aspettare l'auto-discovery.
3. Controlla il log seriale per errori di connessione WiFi:
```bash
   esphome logs src/display-consumi-casa.yaml
```

## La scheda non risulta collegata (nessuna porta seriale trovata)

Sintomo riscontrato in questo progetto: `ls /dev/ttyUSB* /dev/ttyACM*` non restituisce nulla.

1. Verifica con `lsusb` se il dispositivo è visto a livello USB, indipendentemente dalla porta seriale:
```bash
   lsusb
```
   Cerca una riga tipo `ID 303a:1001 Espressif USB JTAG/serial debug unit`.
2. Se non compare affatto: il cavo USB potrebbe essere **solo-carica** (senza linee dati) — causa più comune in assoluto. Prova un altro cavo, idealmente uno che sai già funzionare per trasferimento dati.
3. Su questa scheda (ESP32-C3 con USB nativo), la porta appare tipicamente come `/dev/ttyACM0`, non `/dev/ttyUSB0`.

## WiFi non si connette

1. Verifica SSID/password in `secrets.yaml` (attenzione a caratteri speciali).
2. Se il router usa WiFi 5GHz-only su quella rete, ricorda che l'ESP32 supporta solo 2.4GHz.
3. Come fallback, il device espone un AP temporaneo ("Display-Consumi Fallback") per riconfigurazione — connettiti e segui il captive portal.

## Aggiornamento OTA fallisce

1. Verifica che il device sia raggiungibile: `ping display-consumi-casa.local`
2. Se il flash OTA si blocca a metà, riprova con USB per evitare un device in stato inconsistente.
3. Prima di un OTA rischioso, valuta un tag/backup del firmware funzionante (vedi golden version in `backup/`).
