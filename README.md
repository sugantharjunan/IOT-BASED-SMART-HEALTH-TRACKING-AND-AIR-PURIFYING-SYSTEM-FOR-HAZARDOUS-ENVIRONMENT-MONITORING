   IOT BASED SMART HEALTH TRACKING AND AIR PURIFYING SYSTEM
             FOR HAZARDOUS ENVIRONMENT MONITORING
==============================================================================
  ESP32 | MAX30100 | MQ Sensors | OLED | Firebase | Air Purifier Control
  Real-time Health + Environment Monitoring with Automated Air Purification
==============================================================================


------------------------------------------------------------------------------
1. PROJECT OVERVIEW

This project implements a complete IoT-based health and environment monitoring
system designed for hazardous work environments such as factories, chemical
labs, mining sites, and industrial areas.

It continuously monitors:
  - Worker vital signs (Heart Rate, SpO2)
  - Environmental air quality (CO, LPG, Smoke, Dust)

It automatically triggers an air purifier when dangerous thresholds are
exceeded, and pushes all data to Firebase for real-time cloud monitoring.


------------------------------------------------------------------------------
2. FEATURES

HEALTH MONITORING:
  - Heart Rate (BPM)         -- via MAX30100 pulse oximeter sensor
  - Blood Oxygen (SpO2 %)    -- via MAX30100
  - Real-time finger detection with auto-reset
  - Adaptive beat detection with rolling average BPM

AIR QUALITY MONITORING:
  - CO / LPG / Smoke         -- via MQ-2 or MQ-7 sensor
  - Fine dust / PM2.5        -- via GP2Y1010AU0F or PMS5003
  - Temperature & Humidity   -- via DHT11 / DHT22
  - Air Quality Index (AQI) calculation

AUTOMATED AIR PURIFICATION:
  - Relay-controlled air purifier / exhaust fan
  - Auto-ON when gas/dust exceeds safe threshold
  - Auto-OFF when environment returns to safe levels
  - Manual override via web dashboard

CLOUD & DISPLAY:
  - Firebase Realtime Database -- live data sync
  - OLED display (SSD1306)     -- local status readout
  - Web dashboard / mobile app compatible
  - Threshold-based alerts and notifications


------------------------------------------------------------------------------
3. HARDWARE REQUIREMENTS

  Component              Model / Part              Purpose
  ---------------------  ------------------------  --------------------------
  Microcontroller        ESP32 (38-pin)            Main controller + WiFi
  Pulse Oximeter         MAX30100                  Heart Rate + SpO2
  Gas Sensor             MQ-2 / MQ-7              CO, LPG, Smoke detection
  Dust Sensor            GP2Y1010AU0F              Particulate matter (PM2.5)
  Temp & Humidity        DHT22                     Ambient temp & humidity
  Display                SSD1306 OLED 0.96"        Local data display
  Relay Module           5V 1-channel relay        Air purifier control
  Air Purifier / Fan     12V DC exhaust fan        Environment purification
  Power Supply           5V 2A USB / Li-Po         System power
  Resistors / Caps       150 Ohm, 220uF            Dust sensor LED circuit


------------------------------------------------------------------------------
4. PIN CONNECTIONS

  MAX30100 -->      ESP32:
  MAX30100 Pin    ESP32 Pin
  ----------    ---------
  VIN             3.3V
  GND             GND
  SDA             GPIO 21
  SCL             GPIO 22
  INT             Not connected

OTHER SENSORS --> ESP32:
  Component          ESP32 Pin        Notes
  -----------------  ---------------  -------------------------------
  MQ-2 AOUT          GPIO 34 (ADC)    Analog gas reading
  MQ-2 DOUT          GPIO 35          Digital threshold trigger
  DHT22 DATA         GPIO 4           Single-wire protocol
  Dust Sensor LED    GPIO 16          LED pulse control (via 150 Ohm)
  Dust Sensor AOUT   GPIO 36 (ADC)    Dust analog output
  Relay IN           GPIO 26          Air purifier control
  OLED SDA           GPIO 21          Shared I2C bus
  OLED SCL           GPIO 22          Shared I2C bus


------------------------------------------------------------------------------
5. SOFTWARE & LIBRARIES

ARDUINO IDE SETUP:
  - Board           : ESP32 Dev Module
  - Upload Speed    : 115200
  - Flash Size      : 4MB
  - Partition Scheme: Default

REQUIRED LIBRARIES:
  Library                  Install Via         Purpose
  -----------------------  ------------------  ----------------------------
  Wire.h                   Built-in            I2C communication
  Adafruit SSD1306         Library Manager     OLED display
  Adafruit GFX             Library Manager     Graphics for OLED
  DHT sensor library       Library Manager     DHT11/22 reading
  Firebase ESP Client      Library Manager     Firebase cloud sync
  WiFi.h                   Built-in (ESP32)    WiFi connection

NOTE: MAX30100 uses custom direct register code (no external library needed).


------------------------------------------------------------------------------
6. PROJECT STRUCTURE

  IoT-Health-AirPurifier/
  |-- src/
  |   |-- main.ino              # Main Arduino sketch
  |   |-- max30100.h            # MAX30100 direct register driver
  |   |-- air_quality.h         # MQ sensor + dust sensor logic
  |   |-- firebase_sync.h       # Firebase push functions
  |   `-- display.h             # OLED display functions
  |-- schematics/
  |   |-- circuit_diagram.png   # Full wiring diagram
  |   `-- pcb_layout.png        # PCB layout (optional)
  |-- dashboard/
  |   `-- index.html            # Web dashboard (Firebase hosted)
  |-- docs/
  |   `-- README.txt            # This file
  `-- LICENSE


------------------------------------------------------------------------------
7. FIREBASE SETUP

STEP 1 - CREATE FIREBASE PROJECT:
  - Go to https://console.firebase.google.com
  - Create new project
  - Enable Realtime Database
  - Set database rules to test mode

STEP 2 - GET CREDENTIALS:
  - Project Settings --> General --> Your apps --> Add Web App
  - Copy: API Key, Database URL, Project ID

STEP 3 - CONFIGURE IN CODE:
  Edit firebase_sync.h with your credentials:

    #define FIREBASE_HOST  "your-project.firebaseio.com"
    #define FIREBASE_AUTH  "your-database-secret"
    #define WIFI_SSID      "your-wifi-ssid"
    #define WIFI_PASS      "your-wifi-password"

FIREBASE DATABASE STRUCTURE:
  {
    "health": {
      "heartRate": 74.5,
      "spo2": 98.2,
      "fingerDetected": true
    },
    "environment": {
      "gasLevel": 320,
      "dustDensity": 0.045,
      "temperature": 28.5,
      "humidity": 65.2,
      "airQuality": "Good"
    },
    "control": {
      "purifierActive": false,
      "manualOverride": false
    }
  }


------------------------------------------------------------------------------
8. SAFETY THRESHOLD CONFIGURATION

  Parameter        Safe Level       Warning           Danger (Purifier ON)
  ---------------  ---------------  ----------------  --------------------
  CO (MQ-7)        < 9 ppm          9 - 35 ppm        > 35 ppm
  LPG/Smoke (MQ-2) < 300 ppm        300 - 500 ppm     > 500 ppm
  Dust PM2.5       < 0.05 mg/m3     0.05-0.15 mg/m3   > 0.15 mg/m3
  SpO2             > 95%            90 - 95%           < 90% (Alert)
  Heart Rate       60 - 100 BPM     50-60 / 100-120   < 50 or > 120
  Temperature      18 - 30 C        30 - 40 C         > 40 C

All thresholds are configurable in config.h.
The relay triggers automatically when any danger threshold is crossed.


------------------------------------------------------------------------------
9. SYSTEM WORKING FLOW

1. STARTUP:
   - ESP32 boots and connects to WiFi
   - Initializes MAX30100, MQ sensor, DHT22, OLED, Relay
   - Confirms Firebase connection

2. MAIN LOOP (every 10ms):
   - Reads FIFO from MAX30100 --> calculates Heart Rate + SpO2
   - Reads MQ-2 analog --> calculates gas PPM
   - Reads dust sensor with LED pulse timing
   - Reads DHT22 --> temperature + humidity

3. DECISION ENGINE (every 1 second):
   - Checks all readings against thresholds
   - If danger threshold exceeded --> activates relay (purifier ON)
   - If all readings safe for 30s  --> deactivates relay (purifier OFF)
   - Pushes all data to Firebase
   - Updates OLED display



------------------------------------------------------------------------------
10. EXPECTED SERIAL MONITOR OUTPUT

  ==============================
    IoT Health + Air Monitor
  ==============================
  WiFi Connected. IP: 192.168.1.105
  Firebase Connected.
  MAX30100 Ready.
  ------------------------------
  Heart Rate : 74.2 BPM
  SpO2       : 98.1 %
  Gas Level  : 312 ppm  [SAFE]
  Dust       : 0.032 mg/m3  [SAFE]
  Temp       : 27.4 C
  Humidity   : 63.2 %
  Purifier   : OFF
  Firebase   : Synced


------------------------------------------------------------------------------
11. TROUBLESHOOTING


  Issue                    Cause                    Fix
  -----------------------  -----------------------  --------------------------
  MAX30100 not found       Wrong wiring / 5V VIN    Use 3.3V. Check GPIO 21/22
  Heart Rate = 0           Weak finger contact      Press firmly, wait 10-15s
  Gas always 0             MQ sensor cold           Wait 30s warm-up after ON
  Firebase not syncing     Wrong credentials        Check SSID, password, key
  OLED not displaying      I2C address mismatch     Try 0x3C or 0x3D in init
  Relay not triggering     Wrong GPIO / logic       Check active LOW relay
  ESP32 keeps restarting   Power issue              Use 5V 2A, add 100uF cap


------------------------------------------------------------------------------
12. CONTRIBUTING

Pull requests are welcome!
For major changes, please open an issue first to discuss.

  1. Fork the repository
  2. Create your feature branch:  git checkout -b feature/YourFeature
  3. Commit your changes:         git commit -m "Add YourFeature"
  4. Push to the branch:          git push origin feature/YourFeature
  5. Open a Pull Request


------------------------------------------------------------------------------
13. LICENSE


This project is licensed under the MIT License.
See the LICENSE file for full details.


------------------------------------------------------------------------------
