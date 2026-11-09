# 🌱 ESP32 Soil Moisture Monitoring System

**Live web dashboard + Telegram bot alerts, running entirely on an ESP32.**

An IoT soil moisture monitor built with an ESP32 and a two-prong probe with an LM393 module. The board reads the soil, converts it to a calibrated percentage, hosts its own web dashboard, and messages you on Telegram when the soil is dry and when it has recovered. No cloud server or subscription needed.

![Project overview](images/overview.jpg)

## ✨ Features

- **Real-time moisture (%)** from a 12-bit ADC, averaged over 20 samples and calibrated with two points
- **Self-hosted dashboard** (no external server): live gauge, device stats, 20-minute history chart, alert log, dry-warning banner, light/dark mode, phone and desktop layout
- **Telegram alerts:** dry alert, recovery message and an "online" message on every start-up
- **Telegram commands:** `/start`, `/moisture`, `/status`
- **Spam-free alerts:** hysteresis (30 % / 40 %) plus a 30-minute cooldown
- **Authorised chat only:** other users receive "Unauthorized user"
- **Auto-reconnect** if Wi-Fi drops

## 🧰 Hardware

| Component | Purpose |
|---|---|
| ESP32 development board | Reads the sensor, runs the web server and Telegram client |
| Soil moisture probe + LM393 module | Gives an analog voltage (AO) that falls as soil gets wetter |
| Jumper wires, breadboard | Connections |
| USB cable / 5 V supply | Power and programming |
| Wi-Fi with internet access | Local dashboard and Telegram |

### Wiring

![Wiring](images/wiring.png)

| Sensor pin | ESP32 pin |
|---|---|
| VCC | 3V3 |
| GND | GND |
| AOUT | GPIO34 |

> GPIO34 is an input-only **ADC1** pin, which keeps working while Wi-Fi is on. ADC2 pins are unreliable with Wi-Fi active.

## 💻 Software and libraries

- Arduino IDE with the **ESP32 board package**
- `WiFi.h`, `WebServer.h`, `WiFiClientSecure.h` (included with the ESP32 core)
- [UniversalTelegramBot](https://github.com/witnessmenow/Universal-Arduino-Telegram-Bot) by Brian Lough
- [ArduinoJson](https://arduinojson.org) **6.21.x** by Benoit Blanchon

## 🏗️ How it works

![Architecture](images/architecture.png)

1. The probe's resistance drops as soil gets wetter, so the LM393 AO voltage drops and the **ADC value falls** (lower = wetter).
2. The ESP32 averages 20 ADC samples (5 ms apart) and converts them to a percentage:

   ```
   Moisture (%) = (Raw - 3200) x 100 / (1400 - 3200)   // clipped to 0-100
   ```

   ![Calibration](images/calibration.png)

3. Condition levels:

   | Moisture | Condition |
   |---|---|
   | < 30 % | Dry – needs water (alert) |
   | 30 – 59 % | Moist (good) |
   | ≥ 60 % | Wet |

4. **Alert logic (hysteresis):** a dry alert is sent below 30 %; the system returns to normal only above 40 %, when a recovery message is sent. Between 30 % and 40 % nothing is sent. While still dry, the alert repeats only after 30 minutes.

   ![Hysteresis](images/hysteresis.png)

### Timing

| Parameter | Value |
|---|---|
| Sensor check | every 10 s |
| Telegram polling | every 1 s |
| Dashboard refresh | every 5 s |
| History buffer | 120 readings (about 20 min, kept in RAM) |
| Alert cooldown | 30 min |
| Samples per reading | 20 (5 ms apart) |

### Web endpoints

| Path | Returns |
|---|---|
| `/` | The dashboard page (HTML, CSS, JS stored in flash) |
| `/data` | JSON: raw value, Wi-Fi RSSI, uptime, thresholds, sampling step and stored history |

### Telegram commands

| Command | Reply |
|---|---|
| `/start` | Welcome message, command list and dashboard address |
| `/moisture` | Moisture %, condition and raw ADC value |
| `/status` | Moisture details, Wi-Fi signal, uptime and dashboard address |

## 🚀 Setup

1. **Create the bot:** in Telegram, talk to **@BotFather**, create a bot and copy the **bot token**. Get your **chat ID** from **@myidbot**.
2. **Install libraries:** UniversalTelegramBot and ArduinoJson (6.21.x) via the Arduino Library Manager.
3. **Wire the sensor:** VCC → 3V3, GND → GND, AOUT → GPIO34.
4. **Configure the sketch:** set your Wi-Fi name and password, bot token and chat ID in the user-settings section.
5. **Calibrate:** note the raw value in open air (dry) and in a glass of water (wet) and enter them as the calibration values.
6. **Upload and run:** upload to the ESP32, open the Serial Monitor at **115200 baud** to see the IP address, then open it in a browser on the same Wi-Fi.

> 🔒 Never commit your Wi-Fi password, bot token or chat ID. Keep them in a separate, git-ignored config file.

## 📸 Results

| Dashboard – moist (58 %) | Dashboard – dry + alert | Dashboard – wet (68 %) |
|---|---|---|
| ![Moist](images/dashboard-moist.png) | ![Dry](images/dashboard-dry.png) | ![Wet](images/dashboard-wet.png) |

![Telegram bot](images/telegram-bot.png)

| Situation | Raw ADC | Moisture | Condition |
|---|---|---|---|
| Sensor dry / out of moisture | 4095 | 0 % | Dry – alert sent |
| Dry soil | 2855 | 19 % | Dry |
| Moist soil | 2141 | 58 % | Moist (good) |
| Wet soil | 1969 | 68 % | Wet – recovery message |

The Telegram dry alert and dashboard banner appeared together, and recovery messages were sent only after moisture crossed 40 %, confirming the two-threshold logic.

### Hardware prototype

| In test container | In open air |
|---|---|
| ![Prototype](images/prototype-in-container.jpg) | ![Prototype in air](images/prototype-open-air.jpg) |

## ⚠️ Limitations

- Dashboard is reachable only on the same Wi-Fi network (unless you add port forwarding or a VPN)
- History lives in RAM and resets when the board restarts
- Resistive probes corrode over time and readings may drift
- Calibration values are specific to each sensor and soil type
- Monitors only; it does not water the plant

## 🔭 Future scope

- Relay + water pump for automatic watering
- LED / buzzer for local alerts
- SD card or cloud logging for long-term trends
- DHT11 temperature and humidity sensor
- Battery + solar power with deep sleep
- Change thresholds from the dashboard or Telegram

## 📚 References

- Espressif Systems, ESP32 Series Datasheet and Arduino-ESP32 documentation
- Brian Lough, UniversalTelegramBot library
- Benoit Blanchon, ArduinoJson library
- Telegram Bot API: <https://core.telegram.org/bots/api>

## 👤 Author

**Pathan Mohammad Shoaib Khan**
[GitHub](https://github.com/shoaib725) · [LinkedIn](https://linkedin.com/in/shoaibkhan725)
