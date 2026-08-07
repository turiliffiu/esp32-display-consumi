# Architettura

## Panoramica

┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│ Shelly EM │ HA │ Home Assistant │ API │ ESP32 + OLED │
│ (misura potenza) │ ─────► │ (sensor.shellyem_ │ ─────► │ (ESPHome) │
│ │ │ c7f7a1_channel_1_ │ │ mostra Watt │
│ │ │ power) │ │ in tempo reale │
└──────────────────┘ └──────────────────┘ └──────────────────┘


## Componenti

### Home Assistant
- Nessuna configurazione aggiuntiva richiesta: il sensore `sensor.shellyem_c7f7a1_channel_1_power` esiste già ed è usato anche dalle automazioni di soglia 3500W.
- L'ESP32 si registra come dispositivo ESPHome nativo in HA (auto-discovery via mDNS), non serve MQTT.

### ESP32 (firmware ESPHome)
- Piattaforma: `homeassistant` sensor — legge il valore direttamente dall'API nativa di HA, con aggiornamento push (non polling).
- Display: `ssd1306_i2c`, refresh ogni 2s via `update_interval`.
- Font: Google Fonts (Roboto) scaricati automaticamente in fase di build.

### Sicurezza / segreti
- Credenziali WiFi e chiave di cifratura API in `secrets.yaml`, escluso da git.
- Comunicazione ESP32 ↔ HA cifrata tramite `api.encryption.key`.

## Flusso dati

1. Shelly EM misura la potenza istantanea → HA la espone come `sensor.shellyem_c7f7a1_channel_1_power`.
2. L'ESP32, tramite la piattaforma `homeassistant` di ESPHome, riceve aggiornamenti push da HA ogni volta che il sensore cambia stato.
3. Il display OLED viene ridisegnato ogni 2 secondi con l'ultimo valore disponibile.

## Estensioni future (non ancora implementate)

- Seconda schermata con costo stimato (tariffa fissa già in uso altrove: 0,21 €/kWh)
- Icona/allarme visivo quando si supera la soglia 3500W (coerente con le automazioni Alexa/Telegram esistenti)
- Consumo giornaliero cumulato
