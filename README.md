# Tiny Sonos Display & Favourite Controller (ESP32-C6)

A pocket-sized "iPod Shuffle"-style Sonos controller and Now Playing display powered by **ESPHome**, **LVGL**, and **Home Assistant**[cite: 14]. Built for the **Waveshare ESP32-C6-LCD-1.47** development board[cite: 2, 14].

## Features
* **Live Album Art & Track Info:** Displays a 128×128 album cover alongside scrolling track title, artist, and playback state on the 320×172 IPS display[cite: 11, 14].
* **Progressive JPEG Artwork Support:** Uses Home Assistant's built-in `ffmpeg` to automatically convert progressive JPEGs from streaming services (Spotify, Apple Music, etc.) into lightweight 128×128 PNGs for the ESP32-C6's internal SRAM[cite: 14].
* **Single-Button "Shuffle" Navigation:** Uses the onboard `BOOT` button (`GPIO9`) to browse and launch your Sonos Favourites:
  * **Single Click:** Cycle through your saved Sonos Favourites (`FAV 1/X`).
  * **Long Press (Hold >= 0.5s):** Starts playing the highlighted Favourite on your Sonos speaker, or toggles **Play/Pause** when in Now Playing mode.
* **Ambient RGB Status Diffuser:** Drives the onboard WS2812 RGB LED (`GPIO8`) sandwiched in the clear acrylic layer[cite: 1, 2]:
  * **Green:** Playing
  * **Amber:** Paused
  * **Off:** Idle / Stopped

---

## Hardware Required
* **Board:** Waveshare ESP32-C6-LCD-1.47 (Non-touch version with ST7789V3 172×320 SPI LCD and onboard WS2812 RGB LED)[cite: 2]
* **Pinout Reference:**
  * **SPI Display (ST7789V):** `MOSI: GPIO6`, `SCLK: GPIO7`, `CS: GPIO14`, `DC: GPIO15`, `RST: GPIO21`, `BL: GPIO22` (capped at 50% PWM for thermal safety)[cite: 14]
  * **Acrylic RGB LED:** `GPIO8`[cite: 14]
  * **Control Button (`BOOT`):** `GPIO9` (located on the corner opposite the `RST` button)[cite: 2]

---

## Home Assistant Prerequisites

### 1. Enable the `sensor.sonos_favorites` Entity
The Sonos integration in Home Assistant disables the favourites sensor by default.
1. Go to **Settings → Devices & Services → Sonos → Entities**[cite: 4].
2. Search for `sonos_favorites`[cite: 4, 5].
3. Click the disabled `sensor.sonos_favorites` entity, click the **Gear icon (`⚙️`)**, toggle **Enabled**, and click **Update**[cite: 4, 5].
4. Reload the Sonos integration so the `items` attribute populates with your saved Sonos Favourites.

### 2. Add the Artwork Conversion `shell_command`
Because the ESP32-C6 does not have external PSRAM[cite: 2] and ESPHome's embedded JPEG decoder does not support Progressive JPEGs, Home Assistant converts the current cover art to a 128×128 PNG in `/config/www/`[cite: 14].

1. Ensure you have a `/config/www` folder created in your Home Assistant config directory[cite: 14].
2. Add the following to your Home Assistant `configuration.yaml`:
   ```yaml
   shell_command:
     convert_sonos_art: >
       ffmpeg -y -i "[http://127.0.0.1:8123](http://127.0.0.1:8123){{ url }}" -vf "scale=128:128" /config/www/sonos_cover.png
   ```
3. Restart Home Assistant (or reload Shell Commands in **Developer Tools → YAML**).

---

## ESPHome Installation

1. Copy `waveshare-esp32-c6.yaml` into your ESPHome directory[cite: 14].
2. Update the `substitutions:` block at the top of the YAML to match your environment[cite: 14]:
   ```yaml
   substitutions:
     sonos_entity: "media_player.era_100s"           # Your Sonos media_player entity ID
     sonos_favs_entity: "sensor.sonos_favorites"     # Your enabled Sonos favorites sensor
     ha_base_url: "[http://homeassistant.local:8123](http://homeassistant.local:8123)"  # Your local HTTP Home Assistant URL/IP
   ```
3. Compile and flash the firmware to your Waveshare ESP32-C6-LCD-1.47 via USB-C or OTA[cite: 9, 10, 14].

### 4. Grant Home Assistant Action Permission
Once the device connects to Home Assistant:
1. Go to **Settings → Devices & Services → ESPHome**.
2. Click **Configure** (the gear icon) next to `tiny-sonos-display`[cite: 14].
3. Check **Allow the device to perform Home Assistant actions** and click **Submit**.
