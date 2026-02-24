# ESP32 + DHT22 + MQTT + Home Assistant — Quick Start Guide

## Prerequisites
- ESP32 already flashed with ESPHome firmware
- Docker installed on your machine
- ESP32 connected to the same WiFi network as your machine

---

## 1. Start All Containers
```bash
docker start homeassistant esphome mosquitto
```

---

## 2. Verify Containers Are Running
```bash
docker ps
```
You should see `homeassistant`, `esphome`, and `mosquitto` all with status `Up`.

---

## 3. Check Your Machine's IP
```bash
hostname -I
```
Make sure it matches the broker IP in your ESPHome YAML (`~/esphome/esp32-dht22.yaml`).
If it changed, update the YAML and reflash:
```bash
nano ~/esphome/esp32-dht22.yaml
# change mqtt.broker to your new IP

docker exec -it esphome esphome run /config/esp32-dht22.yaml
# choose option 1 (OTA)
```

---

## 4. Power On the ESP32
Just plug it in — it will automatically:
- Connect to WiFi
- Connect to Mosquitto broker
- Start publishing temperature and humidity every 10 seconds

---

## 5. See Live Sensor Data in Terminal
```bash
docker exec -it mosquitto mosquitto_sub -h localhost -p 1884 -t "#" -v
```
You should see output like:
```
esp32-dht22/sensor/temperature/state 23.5
esp32-dht22/sensor/humidity/state 61.0
esp32-dht22/status online
```
Every 10 seconds new readings will appear.

---

## 6. Open Home Assistant
```
http://localhost:8123
```
Go to **Settings → Devices & Services → MQTT** to see your sensors.

---

## Useful Commands

| Action | Command |
|--------|---------|
| Start all containers | `docker start homeassistant esphome mosquitto` |
| Stop all containers | `docker stop homeassistant esphome mosquitto` |
| View MQTT data live | `docker exec -it mosquitto mosquitto_sub -h localhost -p 1884 -t "#" -v` |
| View ESP32 logs | `docker exec -it esphome esphome logs /config/esp32-dht22.yaml` |
| Reflash ESP32 (OTA) | `docker exec -it esphome esphome run /config/esp32-dht22.yaml` |
| Check containers status | `docker ps` |

---

## Architecture
```
DHT22 Sensor → ESP32 → WiFi → Mosquitto (port 1884) → Home Assistant
```
