# TRMNL ESPHome: Home Assistant Weather and Calendar Dashboard

A fully local Home Assistant dashboard for the [TRMNL 7.5" OG DIY Kit](https://www.seeedstudio.com/TRMNL-7-5-Inch-OG-DIY-Kit-p-6481.html) from Seeed Studio, built with ESPHome. No TRMNL account required. No cloud dependency. Driven entirely by your own Home Assistant instance.

Full build guide and writeup: [Smart Home Secrets](https://smarthomesecrets.ca)

---

## What This Does

The display shows a weather panel on the left and a calendar panel on the right, refreshed every 15 minutes via a Home Assistant automation.

**Left panel:** Current temperature, weather condition and icon, feels-like temperature, 5-slot hourly forecast strip, and an 8-day daily forecast in two columns.

**Right panel:** Two layout options. Choose one:

| Layout | Description |
|--------|-------------|
| **Box-list** | Three labelled sections (Person 1 / Person 2 / Family), Today and Tomorrow columns side by side. Best for mixed household schedules. |
| **Agenda** | Time-blocked grid from 8 AM to 8 PM with lane assignment for overlapping events and colour coding by calendar owner. Best for dense timed schedules. |

---

## Hardware

- [TRMNL 7.5" OG DIY Kit](https://www.seeedstudio.com/TRMNL-7-5-Inch-OG-DIY-Kit-p-6481.html): includes XIAO ESP32-S3 Plus driver board and Waveshare 7.5" V2 e-paper panel (800×480, 125 DPI)
- USB-C data cable for initial flash (charge-only cables will not work)
- 3D printed enclosure. Design files available from [Seeed Studio](https://www.seeedstudio.com)

**Confirmed ESPHome pinout:**

| Signal | GPIO |
|--------|------|
| CLK    | GPIO7 |
| MOSI   | GPIO9 |
| CS     | GPIO44 |
| DC     | GPIO10 |
| Reset  | GPIO38 |
| Busy   | GPIO4 (inverted) |

**Buttons:**

| Button | GPIO | Default config |
|--------|------|----------------|
| KEY1   | GPIO2 | Toggles `light.YOUR_CLOSET_LIGHT` |
| KEY2   | GPIO3 | Toggles `light.YOUR_CLOSET_LIGHT` |
| KEY3   | GPIO5 | Toggles `light.YOUR_CLOSET_LIGHT` |
| RESET  | n/a | Hardware reset, not reprogrammable. Triggers display refresh on reconnect. |

---

## Files

| File | Purpose |
|------|---------|
| `trmnl_epaper_boxlist.yaml` | ESPHome config: Box-list layout |
| `trmnl_epaper_agenda.yaml` | ESPHome config: Agenda layout |
| `ha_trmnl_sensors.yaml` | Home Assistant template sensor package (shared by both layouts) |

---

## Prerequisites

### ESPHome
Install the ESPHome add-on in Home Assistant (Settings > Add-ons > ESPHome). The device is adopted through the ESPHome dashboard after the initial flash.

### Fonts
Download these font files and place them in `config/esphome/fonts/` in your Home Assistant config directory before compiling:

| Font | File | Source |
|------|------|--------|
| Inter | `Inter-Regular.ttf`, `Inter-SemiBold.ttf`, `Inter-Bold.ttf` | [Google Fonts](https://fonts.google.com/specimen/Inter) |
| Noto Sans | `NotoSans-Bold.ttf` | [Google Fonts](https://fonts.google.com/specimen/Noto+Sans) |
| Weather Icons | `weathericons-regular-webfont.ttf` | [Erik Flowers on GitHub](https://github.com/erikflowers/weather-icons) |

### Home Assistant integrations required
- **OpenWeatherMap** (or any weather integration that supports `weather.get_forecasts` with hourly and daily types)
- **Google Calendar** (or any calendar integration that provides events through `calendar.get_events`)

---

## Setup

### Step 1: Flash ESPHome

The TRMNL ships with TRMNL cloud firmware. To flash ESPHome:

1. Open [web.esphome.io](https://web.esphome.io) in Chrome or Edge
2. Hold **B** and press **R** on the TRMNL to enter bootloader mode
3. Immediately click **Connect** in the browser. You have roughly 9 seconds before the window closes
4. Flash the minimal ESPHome base firmware
5. Once adopted in the ESPHome add-on, all future updates go over the air

If the device does not appear after the B + R sequence, try a different USB cable. Most USB-C cables are charge-only.

### Step 2: Place font files

Download the font files listed above and copy them to `config/esphome/fonts/`. The ESPHome YAML references them by filename. Do not rename them.

### Step 3: Add the HA sensors package

Copy `ha_trmnl_sensors.yaml` to `config/packages/` in your Home Assistant config directory.

Add this to your `configuration.yaml` if not already using packages:

```yaml
homeassistant:
  packages:
    trmnl: !include packages/ha_trmnl_sensors.yaml
```

Reload Home Assistant configuration after adding the file.

### Step 4: Replace placeholders

All personal entity IDs use `ALL_CAPS` placeholder names. Search and replace each one before deploying.

**ESPHome YAML placeholders:**

| Placeholder | Replace with |
|-------------|-------------|
| `YOUR_DEVICE_NAME` | Your ESPHome device name (e.g. `closet-display`) |
| `192.168.1.XXX` | Your desired static IP for the device |
| `192.168.1.1` | Your router gateway IP |
| `YOUR_DNS1` | Primary DNS (e.g. your router IP or `8.8.8.8`) |
| `YOUR_DNS2` | Secondary DNS (e.g. `8.8.4.4`) |
| `weather.YOUR_WEATHER_ENTITY` | Your weather integration entity ID |
| `light.YOUR_CLOSET_LIGHT` | The light entity you want the buttons to toggle |

**HA sensors YAML placeholders:**

| Placeholder | Replace with |
|-------------|-------------|
| `weather.YOUR_WEATHER_ENTITY` | Your weather integration entity ID |
| `calendar.YOUR_WORK_CALENDAR` | Your work calendar entity |
| `calendar.YOUR_PERSONAL_CALENDAR` | Your personal calendar entity |
| `calendar.YOUR_SPORTS_CALENDAR` | Any third calendar for person 1 (optional, remove the block if not needed) |
| `calendar.YOUR_FAMILY_CALENDAR` | Shared family calendar entity |
| `calendar.YOUR_PARTNER_CALENDAR` | Your partner's calendar entity |
| `button.YOUR_DEVICE_NAME_refresh_display` | The ESPHome button entity, auto-generated from your device name |

### Step 5: Compile and flash

Choose your layout file (`trmnl_epaper_boxlist.yaml` or `trmnl_epaper_agenda.yaml`), install it in the ESPHome add-on, and compile. The first compile takes a few minutes. Updates after that flash over the air.

### Step 6: Add the refresh automation

The `ha_trmnl_sensors.yaml` package includes a refresh automation that presses the display button every 15 minutes. It also fires on Home Assistant startup. The 30-second delay gives the template sensors time to fetch fresh data before the display renders.

Verify the automation appears in Home Assistant under Settings > Automations after reloading the configuration.

---

## Customising the Box-list Layout

The box-list layout assigns rows to each calendar section based on the maximum event count across today and tomorrow. Default maximums per section:

| Section | Max rows shown |
|---------|---------------|
| Person 1 (Brad) | 5 (overflow shows `+N more`) |
| Person 2 (Kelly) | 2 |
| Family | 3 |

Adjust `brad_rows`, `kelly_rows`, and `fam_rows` in the `draw_box_col` lambda if your calendars are busier or lighter than these defaults.

---

## Customising the Agenda Layout

The agenda view covers 8 AM to 8 PM by default. To change the window, adjust `CAL_H0` and `CAL_H1` at the top of the display lambda.

Events are colour-coded by owner:
- **Person 1:** White fill, black outline, black text
- **Person 2:** Double outline, white fill, black text
- **Family:** Black fill, white text

Overlapping events are placed in side-by-side lanes automatically. Events shorter than 45 minutes are padded to a minimum visible height.

---

## Troubleshooting

**Display shows blank after first flash**
The `update_interval: never` setting means the display will not render until the first refresh is triggered. Press the Refresh Display button entity in Home Assistant or wait for the 15-minute automation to fire.

**White rectangles instead of white text**
ESPHome's `bpp: 4` anti-aliasing blends glyph pixels against the background colour. On e-paper, `Color(0,0,0)` is white. Using bpp:4 with a white colour floods the glyph bounding box. The YAML uses separate `bpp: 1` font declarations for any white-on-dark text rendering.

**Calendar events not appearing**
Check that the `ha_trmnl_sensors.yaml` package loaded correctly. In Developer Tools > Template, test one of the sensor state expressions directly to confirm calendar events are being returned.

**Flash window missed**
If the browser does not detect the device after B + R, the TRMNL firmware went back to sleep. Try again: hold B, tap R, and click Connect immediately. A USB hub or slower computer can help by keeping the port visible longer.

---

## Related

- Full build article: [smarthomesecrets.ca/trmnl-esphome-home-assistant](https://smarthomesecrets.ca/trmnl-esphome-home-assistant)
- More Home Assistant dashboards: [Smart Home Secrets GitHub](https://github.com/smarthomesecrets/My-Home-Assistant-Dashboards)

---

*Built and maintained by [Brad Andrews](https://smarthomesecrets.ca) | Smart Home Secrets*
*Licensed under CC BY-SA 4.0*
