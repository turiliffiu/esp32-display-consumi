# Architettura

## Panoramica

┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│ Shelly EM │ HA │ Home Assistant │ API │ 01space ESP32-C3 │
│ (misura potenza) │ ─────► │ (sensor.shellyem_ │ ─────► │ 0.42 OLED │
│ │ │ c7f7a1_channel_1_ │ │ (ESPHome) │
│ │ │ power) │ │ mostra Watt │
└──────────────────┘ └──────────────────┘ └──────────────────┘


## Hardware reale

- **Scheda:** 01space ESP32-C3 0.42 OLED (PCB viola, chip ESP32-C3-FH4, display OLED SSD1306 integrato saldato sul PCB)
- **Interfaccia USB:** USB-C nativa (USB JTAG/serial integrato nel chip, non chip esterno CP2102/CH340)
- **Display:** OLED 0.42", risoluzione effettiva **72×40 pixel** (il controller SSD1306 gestisce internamente un buffer 128×64, ma solo una finestra centrale 72×40 è fisicamente visibile)
- **I2C:** SDA su GPIO5, SCL su GPIO6 (pin fissi di fabbrica per il display integrato — non riassegnabili senza modifiche hardware)

## Componenti

### Home Assistant
- Nessuna configurazione aggiuntiva richiesta: il sensore `sensor.shellyem_c7f7a1_channel_1_power` esiste già ed è usato anche dalle automazioni di soglia 3500W.
- L'ESP32 si registra come dispositivo ESPHome nativo in HA (auto-discovery via mDNS), non serve MQTT.

### ESP32-C3 (firmware ESPHome)
- Board ESPHome: `esp32-c3-devkitm-1`
- Piattaforma sensore: `homeassistant` — legge il valore direttamente dall'API nativa di HA, con aggiornamento push (non polling).
- Display: `ssd1306_i2c`, modello **"SSD1306 72x40"** (supporto nativo ESPHome per questa finestra display, nessun offset manuale necessario), refresh ogni 2s via `update_interval`.
- Font: Google Fonts (Roboto, 14px) scaricato automaticamente in fase di build.

### Sicurezza / segreti
- Credenziali WiFi e chiave di cifratura API in `secrets.yaml`, escluso da git.
- Comunicazione ESP32 ↔ HA cifrata tramite `api.encryption.key`.

## Flusso dati

1. Shelly EM misura la potenza istantanea → HA la espone come `sensor.shellyem_c7f7a1_channel_1_power`.
2. L'ESP32-C3, tramite la piattaforma `homeassistant` di ESPHome, riceve aggiornamenti push da HA ogni volta che il sensore cambia stato.
3. Il display OLED viene ridisegnato ogni 2 secondi con l'ultimo valore disponibile, centrato nell'area visibile 72×40.

## Lezioni apprese (bug reali riscontrati)

- **Identificazione hardware errata inizialmente:** la config di partenza assumeva un ESP32 classico con OLED esterno collegato via jumper su GPIO21/22. L'hardware reale è invece una scheda All-in-One con OLED integrato su GPIO5/GPIO6 — verificare sempre fisicamente la scheda (foto/serigrafia) prima di scrivere la config, specialmente per cloni economici con pinout non standard.
- **Rumore grafico su display:** usare un offset manuale (`offset_x`/`offset_y`) su un modello dichiarato 128×64 per un pannello fisico 72×40 ha causato rendering corrotto. Il componente ESPHome `ssd1306_i2c` supporta nativamente il modello `"SSD1306 72x40"`, che va preferito quando disponibile.
- **Possibile variante SH1107:** alcuni cloni di questa scheda montano un controller SH1107 invece di un vero SSD1306. Se il rumore grafico si ripresenta con schede diverse dello stesso modello nominale, verificare il chip driver reale.

## Estensioni future (non ancora implementate)

- Seconda schermata con costo stimato (tariffa fissa già in uso altrove: 0,21 €/kWh)
- Icona/allarme visivo quando si supera la soglia 3500W (coerente con le automazioni Alexa/Telegram esistenti)
- Consumo giornaliero cumulato
- UI più ricca sfruttando meglio lo spazio 72×40 (font dinamico, doppia riga)
