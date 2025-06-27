# BMV080 Particle Sensor with OLED Display, SD Logging & MQTT

A comprehensive air quality monitoring system using the SparkFun ESP32-S3 Thing Plus, BMV080 particulate matter sensor, OLED display, SD card logging, and MQTT connectivity for real-time remote monitoring.

## Features

### 📊 Real-time Monitoring
- **PM1, PM2.5, and PM10** particulate matter readings
- **OLED display** with pollution level indicators
- **RGB LED** status indication (Green/Orange/Red/Purple)
- **Automatic display modes** based on air quality levels

### 💾 Data Logging
- **SD card logging** with mean value calculations
- **CSV format** with timestamps
- **Daily file rotation** (new file every 24 hours)
- **Hot-swap SD card support**
- **Automatic error recovery**

### 🌐 MQTT Connectivity
- **WiFi connectivity** with auto-reconnection
- **MQTT data transmission** every 30 seconds
- **Location-based topic structure** for multi-sensor deployments
- **Real-time status reporting**

### 🔧 System Management
- **Auto-reset** after 48+ days to prevent millis() overflow
- **Sensor obstruction detection**
- **Connection status monitoring**
- **Visual feedback** on display and LED

## 🛠️ Hardware Requirements

### Primary Components
- **SparkFun ESP32-S3 Thing Plus** (DEV-24408)
- **SparkFun BMV080 Particulate Matter Sensor** (SEN-22859)
- **OLED Display** (SSD1327, 128x128, I2C)
- **MicroSD card** (FAT32 formatted)
- **LiPo Battery 2000mAh (need to be unplugged, to upload the code to the ESP32-S3 Thing Plus**
- **SD Card 16/32GB ---> if oyur SD Card fails: Make sure the SD Card is fast enough and formatted correctly!

### Connections
```
ESP32-S3 Thing Plus ↔ BMV080 Sensor
     I2C/Qwiic connector

ESP32-S3 Thing Plus ↔ OLED Display  
     I2C (SDA: GPIO8, SCL: GPIO9)

RGB LED: GPIO46 (onboard)
SD Card: Built-in slot
```

### Pin Definitions
| Component | Pin | Description |
|-----------|-----|-------------|
| RGB LED | GPIO46 | Onboard WS2812 LED |
| SD CS | GPIO33 | SD card chip select |
| SD SCK | GPIO38 | SD card SPI clock |
| SD MISO | GPIO39 | SD card SPI data out |
| SD MOSI | GPIO34 | SD card SPI data in |
| SD Detect | GPIO48 | SD card detection |
| I2C SDA | GPIO8 | I2C data (Qwiic) |
| I2C SCL | GPIO9 | I2C clock (Qwiic) |

## 📚 Required Libraries

Install these libraries through the Arduino Library Manager:

```
- SparkFun_BMV080_Arduino_Library
- U8g2lib (for OLED display)
- SdFat (by Bill Greiman)
- FastLED
- WiFi (ESP32 core)
- ArduinoMqttClient
- Wire (I2C communication)
```

## ⚙️ Configuration

### 1. WiFi Settings
```cpp
const char ssid[] = "YOUR_WIFI_SSID";
const char pass[] = "YOUR_WIFI_PASSWORD";
```

### 2. MQTT Broker Settings
```cpp
const char broker[] = "192.168.1.100";  // Your MQTT broker IP
int port = 1883;                        // MQTT port
const char mqtt_username[] = "username"; // MQTT credentials
const char mqtt_password[] = "password";
```

### 3. Location Configuration
**🏢 For multiple sensors, change the location for each device:**

```cpp
// Office sensor
const char location[] = "Office";

// Workshop sensor  
const char location[] = "Workshop";

// Kitchen sensor
const char location[] = "Kitchen";
```

## 📡 MQTT Topic Structure

The system uses a **location/device/measurement** structure:

```
Office/airsensor/pm1          # PM1 readings from Office
Office/airsensor/pm25         # PM2.5 readings from Office  
Office/airsensor/pm10         # PM10 readings from Office
Office/airsensor/status       # Air quality status from Office
Office/airsensor/obstruction  # Sensor obstruction status

Workshop/airsensor/pm1        # PM1 readings from Workshop
Workshop/airsensor/pm25       # PM2.5 readings from Workshop
...
```

### MQTT Data Format
- **PM Values**: Float with 2 decimal places (e.g., `15.42`)
- **Status**: String values: `good_air`, `medium_pollution`, `high_pollution`, `obstructed`
- **Obstruction**: Boolean as string: `true` or `false`

### Subscription Examples
```bash
# All data from Office
Office/+/+

# PM2.5 from all locations
+/airsensor/pm25

# All air sensor data
+/airsensor/+

# Specific measurement from specific location
Office/airsensor/pm25
```

## 🚀 Installation & Setup

### 1. Hardware Assembly
1. Connect BMV080 sensor to ESP32-S3 via Qwiic connector
2. Connect OLED display to I2C via Qwiic connector
3. Insert formatted microSD card 
4. Power via USB-C 
5. And or Connect the Battery - Unplug the Battery to upload the code! 

### 2. Arduino IDE Setup
1. Install ESP32 board package
2. Select **"ESP32S3 Dev Module"**
3. Configure settings:
   - USB Mode: **Hardware CDC and JTAG**
   - USB CDC on Boot: **Enabled**
   - Upload Mode: **UART0 / Hardware CDC**
   - PSRAM: **QSPI PSRAM**

### 3. Code Configuration
1. Update WiFi credentials
2. Set MQTT broker details
3. **Set unique location** for each sensor
4. Upload to ESP32-S3

### 4. Multi-Sensor Deployment
For multiple sensors:
1. Use the same code base
2. **Only change the `location` variable** for each sensor
3. Each sensor will automatically use different MQTT topics

## 📊 Air Quality Thresholds

| Level | PM Reading | Display | LED Color | Status |
|-------|------------|---------|-----------|---------|
| Good | < 15 μg/m³ | "Good Air!" | Green | `good_air` |
| Medium | 15-30 μg/m³ | "Some Particles" | Orange | `medium_pollution` |
| High | > 30 μg/m³ | "Open a Window!" | Red (blinking) | `high_pollution` |
| Error | Sensor blocked | Warning display | Purple | `obstructed` |
| Connecting | WiFi connecting | Connection status | Blue | N/A |

## 💾 Data Logging

### SD Card Format
- **File Format**: CSV with headers
- **Filename Pattern**: `airdata_[boot]_[day].csv`
- **Example**: `airdata_1_0.csv`, `airdata_1_1.csv`

### CSV Structure
```csv
Timestamp,PM1_Mean,PM2.5_Mean,PM10_Mean,ObstructionPercent,SampleCount
60000,12.45,15.32,18.67,0.0,60
120000,11.98,14.89,17.23,0.0,60
```

### Logging Intervals
- **Sensor sampling**: Every 1 second
- **SD card logging**: Every 1 minute (mean values)
- **MQTT transmission**: Every 30 seconds
- **File rotation**: Every 24 hours

## 🔧 Troubleshooting

### LED Status Indicators
- **White blink (2x)**: System startup
- **Blue**: Connecting to WiFi
- **Green**: Good air quality + connected
- **Orange**: Medium pollution
- **Red**: High pollution
- **Purple**: Sensor error or obstruction

### Common Issues

**1. RGB LED not working**
- Ensure you're using the correct ESP32-S3 Thing Plus board
- Pin 46 is correct for this board (not pin 35)

**2. SD Card not detected**
- Ensure card is FAT32 formatted
- Check card insertion and connections
- Try different SD card

**3. WiFi connection fails**
- Verify SSID and password
- Check 2.4GHz network (ESP32 doesn't support 5GHz)
- Ensure network allows new device connections

**4. MQTT not connecting**
- Verify broker IP and port
- Check username/password if broker requires authentication
- Ensure broker is accessible from your network

**5. Sensor readings show obstruction**
- Check if sensor air intake is blocked
- Ensure proper airflow around sensor
- Clean sensor if necessary

### Serial Monitor Output
Enable serial monitor (115200 baud) to see:
- Initialization status
- WiFi/MQTT connection status
- Real-time sensor readings
- SD card operations
- Error messages

## 📈 Integration Examples

### Home Assistant
```yaml
sensor:
  - platform: mqtt
    name: "Office Air Quality PM2.5"
    state_topic: "Office/airsensor/pm25"
    unit_of_measurement: "μg/m³"
    
  - platform: mqtt
    name: "Office Air Quality Status"
    state_topic: "Office/airsensor/status"
```

### Node-RED
```json
[{"id":"mqtt-in","type":"mqtt in","topic":"Office/airsensor/+"}]
```

### Python Subscriber
```python
import paho.mqtt.client as mqtt

def on_message(client, userdata, message):
    topic = message.topic
    value = message.payload.decode()
    print(f"{topic}: {value}")

client = mqtt.Client()
client.on_message = on_message
client.connect("192.168.1.100", 1883, 60)
client.subscribe("Office/airsensor/+")
client.loop_forever()
```

## 🔄 System Behavior

### Startup Sequence
1. **Hardware initialization** (I2C, SD, LED)
2. **WiFi connection** (with visual feedback)
3. **MQTT broker connection**
4. **Sensor initialization and calibration**
5. **Begin monitoring loop**

### Normal Operation
1. **Sample sensor** every 1 second
2. **Update display** with current readings
3. **Log mean values** to SD every minute
4. **Transmit via MQTT** every 30 seconds
5. **Monitor connections** and auto-reconnect if needed

### Error Recovery
- **WiFi disconnection**: Automatic reconnection attempts
- **MQTT disconnection**: Reconnection with exponential backoff
- **SD card removal**: Hot-swap detection and reinitialization
- **Sensor errors**: Continue operation, report error status

## 📝 License

This project is open source. Feel free to modify and distribute according to your needs.
