# Architettura

## Panoramica
                                ┌──────────────────┐
                                │  01space ESP32-C3 │
                          ┌───► │  0.42 OLED         │

┌──────────────────┐ HA │ │ (display-consumi- │
│ Shelly EM │ ─────► │ │ casa) │
│ (misura potenza) │ Home │ └──────────────────┘
│ │ Assist│
│ │ ant │ ┌──────────────────┐
└──────────────────┘ │ │ ESP8266 HW-364A │
└───► │ OLED 128x64 │
│ (display-consumi- │
│ hw364a) │
└──────────────────┘


Entrambi i dispositivi leggono lo stesso sensore Home Assistant (`sensor.shellyem_c7f7a1_channel_1_power`) via API nativa ESPHome, in due punti fisici diversi della casa.

## Dispositivi

### 1. display-consumi-casa (01space ESP32-C3 0.42 OLED)

- **Chip:** ESP32-C3, interfaccia USB nativa JTAG/serial (porta tipicamente `/dev/ttyACM0`)
- **Display:** OLED 0.42" integrato sul PCB, risoluzione effettiva **72×40 pixel**
- **I2C:** SDA = GPIO5, SCL = GPIO6 (pin fissi di fabbrica)
- **Board ESPHome:** `esp32-c3-devkitm-1`
- **Modello display:** `"SSD1306 72x40"` (supporto nativo, nessun offset manuale)
- **File:** `src/display-consumi-casa.yaml`

### 2. display-consumi-hw364a (ESP8266 HW-364A)

- **Chip:** ESP8266MOD, chip USB-seriale esterno CH340 (porta tipicamente `/dev/ttyUSB0`)
- **Display:** OLED 0.96" integrato via flat cable, risoluzione **128×64 pixel** (bicolore: banda superiore gialla, resto blu — caratteristica hardware del pannello)
- **I2C:** SDA = GPIO14 (D6), SCL = GPIO12 (D5)
- **Board ESPHome:** `nodemcuv2`
- **Modello display:** `"SSD1306 128x64"`
- **File:** `src/display-consumi-hw364a.yaml`

## Componenti condivisi

### Home Assistant
- Nessuna configurazione aggiuntiva richiesta: il sensore `sensor.shellyem_c7f7a1_channel_1_power` esiste già ed è usato anche dalle automazioni di soglia 3500W.
- Ogni ESP si registra come dispositivo ESPHome nativo separato in HA (auto-discovery via mDNS, nomi nodo distinti), non serve MQTT.

### Sicurezza / segreti
- **Un solo `secrets.yaml` condiviso** tra i due dispositivi (stessa rete WiFi di casa) — `wifi_ssid`/`wifi_password` in comune; `api_encryption_key` e `ota_password` sono anch'essi condivisi in questo setup (stesso file), ma HA li identifica come dispositivi diversi tramite il `name:` univoco di ciascun nodo YAML.
- Escluso da git in entrambi i casi.

## Flusso dati

1. Shelly EM misura la potenza istantanea → HA la espone come `sensor.shellyem_c7f7a1_channel_1_power`.
2. Ciascun ESP, tramite la piattaforma `homeassistant` di ESPHome, riceve aggiornamenti push da HA ogni volta che il sensore cambia stato.
3. Ogni display si aggiorna autonomamente ogni 2 secondi con l'ultimo valore disponibile, nel formato adatto alla propria risoluzione.

## Lezioni apprese (bug reali riscontrati)

- **Identificazione hardware errata inizialmente (ESP32-C3):** la config di partenza assumeva un ESP32 classico con OLED esterno su GPIO21/22. L'hardware reale è una scheda All-in-One con OLED integrato su GPIO5/GPIO6.
- **Rumore grafico su display 72×40 (ESP32-C3):** risolto passando dal modello generico `"SSD1306 128x64"` con offset manuale al modello nativo `"SSD1306 72x40"`.
- **Possibile variante SH1107 (ESP32-C3):** alcuni cloni montano un controller SH1107 invece di un vero SSD1306 — verificare il datasheet se il rumore si ripresenta su schede diverse.
- **HW-364A:** nessun problema riscontrato al primo flash — pin I2C (GPIO14/GPIO12) e board (`nodemcuv2`) verificati correttamente da fonti multiple prima della scrittura del firmware, evitando l'errore fatto con la prima scheda.

## Estensioni future (non ancora implementate)

- Seconda schermata con costo stimato (tariffa fissa già in uso altrove: 0,21 €/kWh)
- Icona/allarme visivo quando si supera la soglia 3500W (coerente con le automazioni Alexa/Telegram esistenti)
- Consumo giornaliero cumulato
- UI più ricca sfruttando meglio lo spazio disponibile su ciascun display
- Eventuale terzo/quarto display in altre stanze, riusando lo stesso pattern
