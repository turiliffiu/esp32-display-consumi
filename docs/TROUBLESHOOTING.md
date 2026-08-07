# Troubleshooting

## Il display resta spento / non si accende

1. Verifica alimentazione: l'ESP32 riceve corrente via USB? LED di power acceso?
2. Controlla i collegamenti I2C (GPIO21=SDA, GPIO22=SCL) con un tester o riflash del cavo.
3. Verifica l'indirizzo I2C: il default è `0x3C`, ma alcuni moduli usano `0x3D`. In caso di dubbio, aggiungi temporaneamente `i2c: scan: true` (già presente nel config) e controlla il log seriale:
```bash
   esphome logs src/display-consumi-casa.yaml
```

## Il display si accende ma mostra solo "-- W"

Significa che il sensore `homeassistant` non ha ancora ricevuto uno stato valido da HA.

1. Verifica che l'ESP32 sia effettivamente connesso a HA: *Impostazioni → Dispositivi e servizi → ESPHome* → il dispositivo deve risultare "Connesso".
2. Controlla che `sensor.shellyem_c7f7a1_channel_1_power` esista e abbia un valore in HA: *Strumenti per sviluppatori → Stati*.
3. Controlla i log ESP32 per errori di autenticazione API (`api.encryption.key` non coincidente tra ESP32 e HA).

## L'ESP32 non compare in Home Assistant dopo il flash

1. Verifica che ESP32 e HA siano sulla stessa subnet/VLAN (mDNS non attraversa subnet diverse senza reflector).
2. Prova ad aggiungere manualmente l'integrazione ESPHome in HA inserendo l'IP dell'ESP32 invece di aspettare l'auto-discovery.
3. Controlla il log seriale per errori di connessione WiFi:
```bash
   esphome logs src/display-consumi-casa.yaml
```

## WiFi non si connette

1. Verifica SSID/password in `secrets.yaml` (attenzione a caratteri speciali).
2. Se il router usa WiFi 5GHz-only su quella rete, ricorda che l'ESP32 supporta solo 2.4GHz.
3. Come fallback, il device espone un AP temporaneo ("Display-Consumi Fallback") per riconfigurazione — connettiti e segui il captive portal.

## Aggiornamento OTA fallisce

1. Verifica che il device sia raggiungibile: `ping display-consumi-casa.local`
2. Se il flash OTA si blocca a metà, riprova con USB per evitare un device in stato inconsistente.
3. Prima di un OTA rischioso, valuta un tag/backup del firmware funzionante (vedi golden version in `backup/`).
