# ESPHome E-Ink Weather Display

A beautiful, battery-powered weather display based on ESP32 and 7.5" Waveshare E-Paper display. Shows current weather, 5-day forecast, and room sensors from Home Assistant with intelligent deep sleep for long battery life.

![E-Ink Weather Display](images/display-photo.jpg)

## Features

- 🌤️ **Real-time Weather Display** - Current temperature, weather description, and "feels like" temperature
- 📅 **5-Day Weather Forecast** - Visual forecast with icons and high/low temperatures
- 🏠 **6 Room Sensors** - Temperature and humidity for multiple rooms with dynamic icons
- 🔋 **Battery Powered** - Intelligent deep sleep for weeks of battery life
- 🎨 **Beautiful UI** - Clean, modern design with multiple fonts and Material Design icons
- 🌙 **Night Mode** - Automatic update schedule with reduced activity at night
- 📱 **Home Assistant Integration** - All data from Home Assistant sensors
- ⚡ **Energy Efficient** - Updates display then sleeps for 30 minutes

## Hardware

### Required Components

- **ESP32 Development Board** (ESP32dev or similar)
- **Waveshare 7.5" E-Paper Display V2** (Model: 7.50inV2p)
  - Resolution: 800x480 pixels
  - Interface: SPI
- **Battery** (3.7V Li-Ion/Li-Po, recommended 2000mAh+)
- **TP4056 or similar** battery charging module (optional but recommended)
- **Voltage divider** for battery monitoring (optional)

### Wiring

| ESP32 Pin | Display Pin | Function |
|-----------|-------------|----------|
| GPIO13    | CLK         | SPI Clock |
| GPIO14    | DIN         | SPI MOSI |
| GPIO15    | CS          | Chip Select |
| GPIO25    | BUSY        | Busy pin (inverted) |
| GPIO26    | RST         | Reset |
| GPIO27    | DC          | Data/Command |
| A0        | -           | Battery voltage (optional) |
| 3.3V      | VCC         | Power |
| GND       | GND         | Ground |

### Power Consumption

- **Active (updating):** ~150-200mA for ~2-3 seconds
- **Deep Sleep:** ~10-20µA
- **Battery Life:** ~2-4 weeks with 2000mAh battery (depending on update frequency)

## Installation

### Prerequisites

- **Home Assistant** with ESPHome add-on
- **ESPHome Dashboard** installed and configured
- Required sensors in Home Assistant (see below)

### Step 1: Prepare the Configuration

1. Copy `esphome-weather-display.yaml` to your ESPHome configuration directory
2. Rename it to match your device name (e.g., `eink-weather-display.yaml`)
3. Update the `name` in the `esphome:` section

### Step 2: Configure WiFi

Replace the WiFi credentials:

```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
```

Or hardcode them (not recommended):

```yaml
wifi:
  ssid: "YOUR_WIFI_SSID"
  password: "YOUR_WIFI_PASSWORD"
```

### Step 3: Create Required Home Assistant Sensors

You need to create the following entities in Home Assistant:

#### Input Text Entities
- `input_text.weather_display_greeting` - Greeting message (e.g., "Good morning")
- `input_text.weather_display_dates` - Holiday dates (optional)

#### Input Boolean Entities
- `input_boolean.weather_display_deep_sleep` - Enable/disable deep sleep
- `input_boolean.weather_display_holiday_mode` - Show/hide holiday dates

#### Sensor Entities
**Weather Sensors:**
- `sensor.outdoor_temperature` - Current outdoor temperature
- `sensor.weather_description` - Weather condition text
- `sensor.feels_like_temperature` - Feels like temperature

**Forecast Sensors (Day 1-5):**
- `sensor.weather_forecast_day1_day` - Day of week (e.g., "Mon")
- `sensor.weather_forecast_day1_condition` - Weather icon (Material Design icon code)
- `sensor.weather_forecast_day1_high` - High temperature
- `sensor.weather_forecast_day1_low` - Low temperature

**Room Sensors (Room 1-6):**
- `sensor.bedroom_temperature` - Room 1 temperature
- `sensor.bedroom_humidity` - Room 1 humidity
- `sensor.bedroom2_temperature` - Room 2 temperature
- `sensor.bedroom2_humidity` - Room 2 humidity
- (Repeat for rooms 3-6)

#### Example Template Sensors

Here's an example for weather forecast sensors using the default weather integration:

```yaml
# configuration.yaml
sensor:
  - platform: template
    sensors:
      weather_forecast_day1_day:
        friendly_name: "Day 1 Day"
        value_template: >
          {% set forecast = state_attr('weather.home', 'forecast')[0] %}
          {{ forecast.datetime | timestamp_custom('%a') }}

      weather_forecast_day1_condition:
        friendly_name: "Day 1 Condition"
        value_template: >
          {% set forecast = state_attr('weather.home', 'forecast')[0] %}
          {{ forecast.condition | replace('sunny', '\U000F0590') | replace('cloudy', '\U000F0F29') | replace('rainy', '\U000F058A') | replace('snowy', '\U000F0E4C') }}

      weather_forecast_day1_high:
        friendly_name: "Day 1 High"
        value_template: >
          {% set forecast = state_attr('weather.home', 'forecast')[0] %}
          {{ forecast.temperature }}

      weather_forecast_day1_low:
        friendly_name: "Day 1 Low"
        value_template: >
          {% set forecast = state_attr('weather.home', 'forecast')[0] %}
          {{ forecast.templow }}
```

### Step 4: Upload to Device

1. Open ESPHome Dashboard in Home Assistant
2. Click **"New Device"** or open existing device
3. Paste the configuration
4. Click **"Install"** → **"Plug into this computer"** (first time) or **"Wirelessly"** (OTA update)

## Configuration Options

### Display Settings

```yaml
display:
  - platform: waveshare_epaper
    model: 7.50inV2p  # Display model
    rotation: 90°    # Rotation angle
    update_interval: 10s  # Update interval (non-sleep mode)
```

### Deep Sleep Settings

```yaml
deep_sleep:
  sleep_duration: 30min  # How long to sleep between updates
  # wake_up_pin: GPIO0   # Optional: wake on button press
```

### Battery Calibration

Adjust the calibration values based on your battery:

```yaml
sensor:
  - platform: adc
    pin: A0
    filters:
      - multiply: 100.0
      - calibrate_linear:
          - 3.0 -> 0   # Empty voltage
          - 4.2 -> 100 # Full voltage
```

### Night Mode Schedule

```yaml
time:
  - platform: homeassistant
    on_time:
      - hours: 2  # Update at 2 AM
        then:
          - script.execute: update_screen
          - deep_sleep.enter:
              id: deep_sleep_control
      - hours: 5  # Update at 5:30 AM
        minutes: 30
        then:
          - script.execute: update_screen
          - deep_sleep.enter:
              id: deep_sleep_control
```

## Customization

### Changing Room Names

Edit the room name in the display lambda:

```cpp
it.printf(100 - 20, 446, id(font_roboto_500_15), color_on, TextAlign::CENTER, "YOUR_ROOM_NAME");
```

### Adding More Forecast Days

Copy the forecast day 1 code and adjust the positions:

```cpp
// Day 2
it.printf(140, 358, id(font_material_design_icons_400_32), color_on, TextAlign::CENTER, "%s", id(sensor_forecast_day2_condition).state.c_str());
// ... adjust positions for Day 2
```

### Changing Fonts

To use custom font files instead of Google Fonts:

```yaml
font:
  - file: 'fonts/your-font.ttf'
    id: custom_font
    size: 40
    glyphs: "!\"%()+=,-_.:°0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnopqrstuvwxyz"
```

Place font files in `/config/esphome/fonts/` directory.

### Customizing Icons

The configuration uses Material Design Icons. To change an icon:

1. Find icon code at [Material Design Icons](https://pictogrammers.com/library/mdi/)
2. Add to font glyphs list
3. Use in display lambda with `\U` followed by the unicode code

Example:
```cpp
it.printf(x, y, id(font_mdi), color_on, TextAlign::CENTER, "\U000F0590"); // Sunny icon
```

## Troubleshooting

### Display Not Updating

1. Check WiFi connection in ESPHome logs
2. Verify Home Assistant API connection
3. Ensure all required sensors exist in Home Assistant
4. Check battery voltage (should be > 3.0V)

### Battery Draining Fast

1. Verify deep sleep is working (check logs for "Going to deep sleep")
2. Reduce `update_interval` in non-sleep mode
3. Increase `sleep_duration`
4. Check for sensor updates triggering wake-ups

### WiFi Connection Issues

1. Verify SSID and password
2. Check signal strength at device location
3. Try 2.4GHz WiFi only (ESP32 doesn't support 5GHz)
4. Check antenna connection

### Display Shows Garbage

1. Verify SPI wiring
2. Check display model in configuration
3. Try reducing SPI clock speed
4. Ensure display receives adequate power (3.3V, at least 200mA during updates)

### Calibration Issues

If battery percentage is incorrect:

1. Measure actual voltage at ADC pin
2. Adjust calibration values:
   ```yaml
   calibrate_linear:
     - 3.0 -> 0   # Measured empty voltage
     - 4.2 -> 100 # Measured full voltage
   ```

## Credits

- **Inspired by:** [ESPHomeDesigner](https://github.com/koosoli/ESPHomeDesigner) by Koosoli
- **Design:** Most of the UI design and lambda rendering was manually customized
- **Display:** Waveshare 7.5" E-Paper Display
- **Platform:** ESPHome

## License

This project is open source and available under the MIT License.

## Contributing

Contributions are welcome! Feel free to:

- Report bugs
- Suggest new features
- Submit pull requests
- Improve documentation

## Support

For issues or questions:

1. Check the [Troubleshooting](#troubleshooting) section
2. Search existing [GitHub Issues](../../issues)
3. Create a new issue with:
   - Hardware details
   - ESPHome logs
   - Configuration (sanitized)
   - Description of the problem

## Roadmap

- [ ] Add touch support for manual refresh
- [ ] Integrate calendar events
- [ ] Add sunrise/sunset times
- [ ] Support for air quality sensors
- [ ] Custom theme configuration
- [ ] Web interface for quick configuration

---

**Enjoy your weather display!** 🌤️
