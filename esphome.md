# Esphome configuration
- If you are going to flash the esp32, add the line ```--device /dev/ttyUSB0:/dev/ttyUSB0 \ ```
- If you are just going to connect and receive data, just remove it.


```bash
docker stop esphome && docker rm esphome

docker run -d \
  --name esphome \
  --network host \
  --restart unless-stopped \
  -v ~/esphome:/config \
  --device /dev/ttyUSB0:/dev/ttyUSB0 \
  ghcr.io/esphome/esphome:latest
```
