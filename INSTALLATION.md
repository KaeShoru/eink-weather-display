# Installation Guide - ESPHome E-Ink Weather Display

This guide will walk you through the complete setup of your E-Ink Weather Display.

## Table of Contents

1. [Hardware Assembly](#hardware-assembly)
2. [ESPHome Configuration](#esphome-configuration)
3. [Home Assistant Setup](#home-assistant-setup)
4. [First Upload](#first-upload)
5. [Testing](#testing)
6. [Battery Calibration](#battery-calibration)

---

## Hardware Assembly

### Step 1: Gather Components

Before starting, ensure you have:

- [ ] ESP32 development board
- [ ] 7.5" Waveshare E-Paper Display V2
- [ ] Li-Ion/Li-Po battery (2000mAh+ recommended)
- [ ] TP4056 charging module (optional)
- [ ] Jumper wires (male-to-female)
- [ ] Breadboard or perfboard for permanent mounting

### Step 2: Wire the Display

Connect the display to ESP32 as follows:

```
Waveshare Display    ESP32
================    ======
VCC (3.3V)    ----> 3.3V
GND           ----> GND
DIN           ----> GPIO14 (MOSI)
CLK           ----> GPIO13 (SCK)
CS            ----> GPIO15
DC            ----> GPIO27
RST           ----> GPIO26
BUSY          ----> GPIO25
```

**Important:**
- Use 3.3V, NOT 5V (E-Paper displays are 3.3V only!)
- Keep wires as short as possible
- Double-check all connections before powering on

### Step 3: Battery Connection (Optional)

For battery monitoring, connect the battery voltage divider to A0:

```
Battery (+) ----[100kΩ]---- A0 ----[100kΩ]---- GND
                               |
                              10µF (to GND)
```

**Simple option without voltage divider:**
- Connect battery positive directly to ESP32 VIN (if board supports)
- Battery negative to GND
- Note: Battery monitoring will need adjustment

### Step 4: Mounting

- Mount ESP32 and display on a board or 3D printed case
- Ensure display is secure (use the mounting holes)
- Position for good WiFi reception
- Add a power switch for easy on/off

---

## ESPHome Configuration

### Step 1: Install ESPHome Add-on

1. Open Home Assistant
2. Go to **Settings** → **Add-ons** → **Add-on Store**
3. Search for "ESPHome"
4. Click **Install**
5. After installation, click **Start** and **Open Web UI**

### Step 2: Create New Device

1. In ESPHome Dashboard, click **"New Device"**
2. Name it: `eink-weather-display` (or your preferred name)
3. Click **Next**

### Step 3: Copy Configuration

1. Open `esphome-weather-display.yaml` from this repository
2. Copy the entire content
3. Paste it into the ESPHome editor
4. Click **"Save"**

### Step 4: Configure WiFi

Replace the WiFi section:

```yaml
wifi:
  ssid: "YOUR_WIFI_SSID"
  password: "YOUR_WIFI_PASSWORD"
```

Or use secrets (recommended):

1. In ESPHome, go to **Secrets** tab
2. Add:
   ```
   wifi_ssid: "YOUR_WIFI_SSID"
   wifi_password: "YOUR_WIFI_PASSWORD"
   ```
3. In the configuration, use:
   ```yaml
   wifi:
     ssid: !secret wifi_ssid
     password: !secret wifi_password
   ```

### Step 5: Adjust Device Name

Change the device name in the first line:

```yaml
esphome:
  name: eink-weather-display  # Change to your preferred name
```

---

## Home Assistant Setup

### Step 1: Create Input Text Entities

Add to your `configuration.yaml` or create `input_texts.yaml`:

```yaml
input_text:
  weather_display_greeting:
    name: "Weather Display Greeting"
    initial: "Good morning"
    mode: text

  weather_display_dates:
    name: "Weather Display Dates"
    initial: ""
    mode: text
```

### Step 2: Create Input Boolean Entities

Add to your `configuration.yaml` or create `input_booleans.yaml`:

```yaml
input_boolean:
  weather_display_deep_sleep:
    name: "Weather Display Deep Sleep"
    initial: true

  weather_display_holiday_mode:
    name: "Weather Display Holiday Mode"
    initial: false
```

### Step 3: Create Weather Sensors

#### Option 1: Using Default Weather Integration

If you use the default weather integration, add these template sensors:

```yaml
sensor:
  # Current weather
  - platform: template
    sensors:
      outdoor_temperature:
        friendly_name: "Outdoor Temperature"
        value_template: "{{ state_attr('weather.home', 'temperature') }}"
        unit_of_measurement: "°C"

      weather_description:
        friendly_name: "Weather Description"
        value_template: "{{ state_attr('weather.home', 'condition') }}"

      feels_like_temperature:
        friendly_name: "Feels Like Temperature"
        value_template: "{{ state_attr('weather.home', 'temperature') }}"

  # Forecast - Day 1
  - platform: template
    sensors:
      weather_forecast_day1_day:
        friendly_name: "Forecast Day 1 - Day"
        value_template: >
          {% set forecast = state_attr('weather.home', 'forecast')[0] %}
          {{ forecast.datetime | timestamp_custom('%a') }}

      weather_forecast_day1_condition:
        friendly_name: "Forecast Day 1 - Condition"
        value_template: >
          {% set forecast = state_attr('weather.home', 'forecast')[0] %}
          {{ forecast.condition | replace('sunny', '\U000F0590')
                                | replace('cloudy', '\U000F0F29')
                                | replace('rainy', '\U000F058A')
                                | replace('snowy', '\U000F0E4C')
                                | replace('partlycloudy', '\U000F0F2C') }}

      weather_forecast_day1_high:
        friendly_name: "Forecast Day 1 - High"
        value_template: >
          {% set forecast = state_attr('weather.home', 'forecast')[0] %}
          {{ forecast.temperature }}
        unit_of_measurement: "°C"

      weather_forecast_day1_low:
        friendly_name: "Forecast Day 1 - Low"
        value_template: >
          {% set forecast = state_attr('weather.home', 'forecast')[0] %}
          {{ forecast.templow }}
        unit_of_measurement: "°C"
```

Repeat for days 2-5, changing `[0]` to `[1]`, `[2]`, etc.

#### Option 2: Using OpenWeatherMap

If you use OpenWeatherMap, you'll need to map the forecast attributes similarly.

### Step 4: Create Room Sensors

For each room, you need temperature and humidity sensors. Example:

```yaml
sensor:
  # Bedroom
  - platform: template
    sensors:
      bedroom_temperature:
        friendly_name: "Bedroom Temperature"
        value_template: "{{ states('sensor.bedroom_temp_sensor') }}"
        unit_of_measurement: "°C"

      bedroom_humidity:
        friendly_name: "Bedroom Humidity"
        value_template: "{{ states('sensor.bedroom_humidity_sensor') }}"
        unit_of_measurement: "%"
```

Repeat for all 6 rooms, updating entity IDs.

### Step 5: Restart Home Assistant

After adding all entities, restart Home Assistant:

**Settings** → **System** → **Restart**

---

## First Upload

### Step 1: Connect ESP32 to Computer

1. Connect ESP32 to your computer via USB
2. Wait for drivers to install (Windows only)
3. Note the COM port (Windows) or /dev/ttyUSBx (Linux/Mac)

### Step 2: Upload via USB

1. In ESPHome Dashboard, click your device
2. Click **"Install"**
3. Click **"Plug into this computer"**
4. Select the correct serial port
5. Click **"Connect"**
6. Wait for compilation and upload (~5-10 minutes)

### Step 3: Verify Installation

After upload completes:

1. Check ESPHome logs for errors
2. Verify device appears in Home Assistant **Settings** → **Devices & Services**
3. Device should show as "Connected"

---

## Testing

### Step 1: Test Display Update

1. Enable the device in Home Assistant
2. Wait for the display to update (should happen within 10 seconds)
3. Verify:
   - [ ] Display shows greeting
   - [ ] Time is correct
   - [ ] WiFi signal shows
   - [ ] Battery percentage shows
   - [ ] Weather data appears

### Step 2: Test Deep Sleep

1. Ensure `input_boolean.weather_display_deep_sleep` is ON
2. Wait for update to complete (2-5 seconds)
3. Device should enter deep sleep
4. Check logs: should see "Going to deep sleep"
5. Wait 30 minutes (or manually wake by pressing RESET)

### Step 3: Test Manual Refresh

1. Turn OFF `input_boolean.weather_display_deep_sleep`
2. Device should wake up and update every 60 seconds
3. Turn it back ON to enable deep sleep again

### Step 4: Test Holiday Mode

1. Set `input_boolean.weather_display_holiday_mode` to ON
2. Update greeting or dates in input texts
3. Wait for display update
4. Verify dates appear below greeting

---

## Battery Calibration

### Step 1: Measure Full Battery

1. Charge battery to 100%
2. Measure voltage at ADC pin (or ESP32 VIN)
3. Note the voltage (e.g., 4.2V)

### Step 2: Measure Empty Battery

1. Use device until battery is low (display stops working)
2. Measure voltage at ADC pin
3. Note the voltage (e.g., 3.0V)

### Step 3: Update Calibration

Edit the configuration:

```yaml
sensor:
  - platform: adc
    pin: A0
    filters:
      - multiply: 100.0
      - calibrate_linear:
          - 3.0 -> 0   # Your measured empty voltage
          - 4.2 -> 100 # Your measured full voltage
```

### Step 4: Upload Updated Config

1. Update the configuration
2. Upload via OTA (wireless)
3. Verify battery percentage is accurate

---

## Troubleshooting

### Display Not Working

**Problem:** Display stays blank or shows garbage

**Solutions:**
1. Check all SPI connections
2. Verify 3.3V power supply (not 5V!)
3. Try reducing SPI speed (add `frequency: 1MHz` to SPI config)
4. Check display model in configuration

### WiFi Not Connecting

**Problem:** Device can't connect to WiFi

**Solutions:**
1. Verify SSID and password
2. Check 2.4GHz WiFi only (ESP32 doesn't support 5GHz)
3. Check signal strength at device location
4. Try moving device closer to router

### Deep Sleep Not Working

**Problem:** Device doesn't enter deep sleep

**Solutions:**
1. Verify `input_boolean.weather_display_deep_sleep` is ON
2. Check logs for error messages
3. Ensure all sensors have valid data
4. Try disabling some sensors to isolate issue

### Battery Percentage Wrong

**Problem:** Battery shows incorrect percentage

**Solutions:**
1. Recalibrate battery (see [Battery Calibration](#battery-calibration))
2. Check voltage divider resistors (if using)
3. Verify ADC pin is not connected to anything else

### Home Assistant Sensors Not Appearing

**Problem:** Display shows "--" for sensor values

**Solutions:**
1. Verify all required sensors exist in Home Assistant
2. Check entity IDs match configuration
3. Ensure sensors have valid data (not "unknown" or "unavailable")
4. Check Home Assistant logs for errors

---

## Next Steps

After successful installation:

1. **Customize greeting:** Edit `input_text.weather_display_greeting`
2. **Adjust room names:** Edit the configuration and room sensor entity IDs
3. **Tune deep sleep:** Adjust `sleep_duration` based on battery life
4. **Create a case:** 3D print or build a custom enclosure
5. **Add automation:** Create Home Assistant automations for special events

---

**Congratulations!** Your E-Ink Weather Display is now ready to use! 🎉

For additional help, check the main [README.md](README.md) or open a GitHub issue.
