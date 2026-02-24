# ESP32 + ESPHome + Home Assistant Setup Summary

## 1. Network Fix — Move Home Assistant to Host Network
```bash
docker stop homeassistant && docker rm homeassistant

docker run -d \
  --name homeassistant \
  --network host \
  --restart unless-stopped \
  -v ~/homeassistant:/config \
  --privileged \
  ghcr.io/home-assistant/home-assistant:stable
```

## 2. Recreate ESPHome with USB Device Access
```bash
docker stop esphome && docker rm esphome

docker run -d \
  --name esphome \
  --network host \
  --restart unless-stopped \
  -v ~/esphome:/config \
  --device /dev/ttyUSB1:/dev/ttyUSB1 \
  ghcr.io/esphome/esphome:latest
```

## 3. Find ESP32 USB Port
```bash
dmesg | tail -20
# or
ls /dev/ttyUSB*
# Result: /dev/ttyUSB1
```

## 4. Create ESPHome YAML Config
```bash
nano ~/esphome/esp32-dht22.yaml
```
Content:
```yaml
esphome:
  name: esp32-dht22

esp32:
  board: esp32dev

wifi:
  ssid: "YourWiFiName"
  password: "YourWiFiPassword"

api:

ota:
  - platform: esphome

logger:

sensor:
  - platform: dht
    pin: GPIO27
    model: DHT22
    temperature:
      name: "Temperature"
    humidity:
      name: "Humidity"
    update_interval: 10s
```

## 5. Stop ModemManager (was blocking the USB port)
```bash
sudo systemctl stop ModemManager
sudo systemctl disable ModemManager
```

## 6. Flash Firmware to ESP32 (First Time via USB)
```bash
docker exec -it esphome esphome run /config/esp32-dht22.yaml --device /dev/ttyUSB1
```

## 7. Fix USB Port Busy (Chromium was holding it)
```bash
sudo lsof /dev/ttyUSB1
# Found: Chromium was holding the port
# Solution: Close Chromium browser, then re-run step 6
```

## 8. Update Config and Flash OTA (Wireless)
After first flash, all updates are wireless:
```bash
docker exec -it esphome esphome run /config/esp32-dht22.yaml
```

## Result
- ESP32 connected to WiFi (IP: 10.86.179.4)
- DHT22 sensor reading Temperature and Humidity on GPIO27
- ESPHome running at http://localhost:6052
- Home Assistant running at http://localhost:8123
- Both on host network — auto-discovery works
