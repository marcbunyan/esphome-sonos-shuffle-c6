# Tiny Sonos Display & Favourite Controller (ESP32-C6)

A pocket-sized "iPod Shuffle"-style Sonos controller and Now Playing display powered by **ESPHome**, **LVGL**, and **Home Assistant**. Built for the **Waveshare ESP32-C6-LCD-1.47** development board.

## Features
* **Live Album Art & Track Info:** Displays a 128×128 album cover alongside scrolling track title, artist, and playback state on the 320×172 IPS display.
* **Progressive JPEG Artwork Support:** Uses Home Assistant's built-in `ffmpeg` to automatically convert progressive JPEGs from streaming services (Spotify, Apple Music, etc.) into lightweight 128×128 PNGs for the ESP32-C6's internal SRAM.
* **Single-Button "Shuffle" Navigation:** Uses the onboard `BOOT` button (`GPIO9`) to browse and launch your Sonos Favourites:
  * **Single Click:** Cycle through your saved Sonos Favourites (`FAV 1/X`).
  * **Long Press (Hold >= 0.5s):** Starts playing the highlighted Favourite on your Sonos speaker, or toggles **Play/Pause** when in Now Playing mode.
* **Ambient RGB Status Diffuser:** Drives the onboard WS2812 RGB LED (`GPIO8`) sandwiched in the clear acrylic layer:
  * **Green:** Playing
  * **Amber:** Paused
  * **Off:** Idle / Stopped

---

## Hardware Required
* **Board:** Waveshare ESP32-C6-LCD-1.47 (Non-touch version with ST7789V3 172×320 SPI LCD and onboard WS2812 RGB LED)
* **Pinout Reference:**
  * **SPI Display (ST7789V):** `MOSI: GPIO6`, `SCLK: GPIO7`, `CS: GPIO14`, `DC: GPIO15`, `RST: GPIO21`, `BL: GPIO22` (capped at 50% PWM for thermal safety)
  * **Acrylic RGB LED:** `GPIO8`
  * **Control Button (`BOOT`):** `GPIO9` (located on the corner opposite the `RST` button)

---

## Home Assistant Prerequisites

### 1. Enable the `sensor.sonos_favorites` Entity
The Sonos integration in Home Assistant disables the favourites sensor by default.
1. Go to **Settings → Devices & Services → Sonos → Entities**.
2. Search for `sonos_favorites`.
3. Click the disabled `sensor.sonos_favorites` entity, click the **Gear icon (`⚙️`)**, toggle **Enabled**, and click **Update**.
4. Reload the Sonos integration so the `items` attribute populates with your saved Sonos Favourites.

### 2. Add the Artwork Conversion `shell_command`
Because the ESP32-C6 does not have external PSRAM and ESPHome's embedded JPEG decoder does not support Progressive JPEGs, Home Assistant converts the current cover art to a 128×128 PNG in `/config/www/`.

1. Ensure you have a `/config/www` folder created in your Home Assistant config directory.
2. Add the following to your Home Assistant `configuration.yaml`:
   ```yaml
   shell_command:
     convert_sonos_art: >
       ffmpeg -y -i "[http://127.0.0.1:8123](http://127.0.0.1:8123){{ url }}" -vf "scale=128:128" /config/www/sonos_cover.png
   ```
3. Restart Home Assistant (or reload Shell Commands in **Developer Tools → YAML**).

---

## ESPHome Installation

1. Copy `waveshare-esp32-c6.yaml` into your ESPHome directory.
2. Update the `substitutions:` block at the top of the YAML to match your environment:
   ```yaml
   substitutions:
     sonos_entity: "media_player.era_100s"           # Your Sonos media_player entity ID
     sonos_favs_entity: "sensor.sonos_favorites"     # Your enabled Sonos favorites sensor
     ha_base_url: "[http://homeassistant.local:8123](http://homeassistant.local:8123)"  # Your local HTTP Home Assistant URL/IP
   ```
3. Compile and flash the firmware to your Waveshare ESP32-C6-LCD-1.47 via USB-C or OTA.

### 4. Grant Home Assistant Action Permission
Once the device connects to Home Assistant:
1. Go to **Settings → Devices & Services → ESPHome**.
2. Click **Configure** (the gear icon) next to `tiny-sonos-display`.
3. Check **Allow the device to perform Home Assistant actions** and click **Submit**.
