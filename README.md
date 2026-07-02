# Arduino Learning Journal

A personal record of Arduino experiments and projects, progressing from basic GPIO control through WiFi-enabled web interfaces, digital signal processing (FFT), and multi-sensor integration on the Arduino Uno R4.

![Arduino](https://img.shields.io/badge/Platform-Arduino-00979D?logo=arduino&logoColor=white)
![C++](https://img.shields.io/badge/Language-C%2B%2B-00599C?logo=c%2B%2B&logoColor=white)
![ESP8266](https://img.shields.io/badge/Board-ESP8266-E7352C)
![Uno R4](https://img.shields.io/badge/Board-Arduino%20Uno%20R4-00979D)

---

## Visual Proof

> _Add a photo or GIF of the hardware setup here._

---

## Overview

This repository documents a hands-on learning progression through Arduino development. It is split into three main areas:

- **Arduino Guides** — 34 structured lessons from the Plusivo kit curriculum (ESP8266/NodeMCU target), covering hardware control from blinking an LED to hosting a web server that plays a piano over WiFi.
- **FFT** — Fast Fourier Transform experiments that sample an analog signal via pin A0 and identify the dominant frequency; includes a MATLAB script for offline spectral verification, multiple embedded FFT implementations, and a parallel version contributed by a collaborator (`FFT_Paul`).
- **Arduino_Sensor_Read** — An Arduino Uno R4 project that reads temperature and humidity from a DHT11 sensor and scrolls the readings across the board's built-in LED matrix, with optional LCD I2C output.

Reference datasheets for the Arduino Nano ESP32 (ABX00087) are included at the repo root.

---

## Key Features

**Arduino Guides (Lessons 1–34)**
- LED control: digital on/off (Lesson 1), PWM dimming (Lesson 2), RGB mixing (Lesson 3)
- Actuators: DC motor speed and direction control (Lesson 4), servo position via potentiometer (Lesson 14)
- Sensors: HC-SR04 ultrasonic distance (Lesson 5), DHT11 temperature/humidity (Lesson 13)
- Audio: buzzer tones (Lesson 9), song playback via note arrays (Lesson 12), web-controlled piano with one octave (Lesson 24) and seven octaves (Lesson 25)
- Displays: 4-digit 7-segment display driven through two chained 74HC595 shift registers with multiplexing (Lessons 26–30)
- Signal processing: Exponential Moving Average applied to ultrasonic distance readings to reduce noise before display (Lesson 31)
- WiFi networking: STA mode web server (Lesson 15), HTML control panels served inline from flash (Lessons 16–23), Soft Access Point mode without a router (Lesson 33), SPIFFS file system to serve HTML from flash separately from the sketch (Lesson 34)
- OTA firmware update: wirelessly push a new sketch over WiFi using `ArduinoOTA` (Lesson 32)

**FFT Projects**
- Reads 256 analog samples from pin A0 at a configurable sampling rate
- Runs `ZeroFFT` (Adafruit) and prints each frequency bin with its amplitude to Serial
- Identifies and prints the dominant frequency
- Standalone FFT implementation without external libraries (bit-reversal permutation + Cooley–Tukey butterfly, returns top 5 frequency peaks in `f_peaks[]`)
- MATLAB script for single-sided amplitude spectrum verification

**Arduino_Sensor_Read (Uno R4)**
- Reads float temperature and humidity from DHT11 via the `DHT` library
- Scrolls readings as text across the built-in 12×8 LED matrix using `Arduino_LED_Matrix` + `ArduinoGraphics`
- Optional I2C LCD output via `LiquidCrystal_I2C` (enabled by flipping the `LCD` and `DHT_DATA` compile-time flags)

---

## System Architecture & Data Flow

This is a collection of independent Arduino sketches, not a single application. Each project follows the standard Arduino model:

```
setup() — runs once on power-on or reset
  ↓ initialise pins, Serial, WiFi, libraries
loop() — runs repeatedly forever
  ↓ read sensors → process → output (GPIO / Serial / HTTP response)
```

**WiFi lessons flow** (Lessons 15–25, 33–34):

```
Board boots
  → connectToWiFi() joins network as STA  (or softAP() creates its own AP)
  → setupServer() registers URL handlers with ESP8266WebServer
  → loop() calls server.handleClient()
       → browser GET "/" → serve inline HTML page
       → browser POST "/led" → digitalWrite HIGH/LOW
       → browser GET "/c_note" → tone(buzzer, 1047 Hz)
```

**FFT flow**:

```
loop()
  → read_signal_data(): analogRead A0 × 256 samples → q15_t array
  → run_fft(): ZeroFFT(array, 256)
  → print each bin: "X Hz: Y"
  → scan bins for max amplitude → print dominant frequency
  → delay 5 s, repeat
```

**Sensor/display flow** (Arduino_Sensor_Read):

```
setup()
  → matrix.begin() → scroll "UNO" on LED matrix
  → dht.begin()
loop()
  → dht.readTemperature() / readHumidity()
  → matrix_print_text() scrolls "Temp: XX.XXC" then "Humidity: XX.XX%"
```

> _Add a hardware wiring diagram here._

---

## Project Structure

```
Arduino/
├── ABX00087-datasheet.pdf          # Arduino Nano ESP32 datasheet
├── ABX00087-full-pinout.pdf        # Arduino Nano ESP32 pinout
├── ABX00087-schematics.pdf         # Arduino Nano ESP32 schematics
│
├── Arduino Guides/
│   ├── Guide.pdf                   # Plusivo printed guide (companion to lessons)
│   └── lessons/
│       ├── Lesson 1_ Blink an LED/
│       │   └── Blink_an_LED/
│       │       └── Blink_an_LED.ino
│       ├── Lesson 2_ Dim an LED/ … Lesson 14_ … (hardware control)
│       ├── Lesson 15_ Wireless Connectivity/ … Lesson 25_ … (WiFi + web UI)
│       ├── Lesson 26_ Shift Register/ … Lesson 31_ … (display drivers + DSP)
│       ├── Lesson 32_ OTA Upload/      # Over-the-air sketch upload
│       ├── Lesson 33_ Soft Access Point/
│       ├── Lesson 34_ SPIFFS/          # HTML served from flash filesystem
│       └── HTML/                       # Standalone HTML examples (Bootstrap, jQuery)
│
├── FFT/
│   ├── FFT.ino                     # Main FFT sketch (Adafruit ZeroFFT)
│   ├── signal.h                    # Hard-coded 200 Hz + 800 Hz test signal
│   ├── FFT_tests.ino/              # Alternate implementations under test
│   │   ├── FFT_Function_upload_020521.ino   # Custom Cooley–Tukey, no library
│   │   ├── EasyFFT/
│   │   ├── Quick_FFT/
│   │   └── Test_nou/ApproxFFT/
│   └── Matlab_code/
│       ├── Arduino_FFT_Spectral.m  # MATLAB script for spectral verification
│       └── Input_Signal.mat        # Sample signal data
│
├── FFT_Paul/
│   └── AdafruitFFT_test/           # Collaborator's FFT variant (1000 Hz sample rate)
│       └── AdafruitFFT_test.ino
│
├── Arduino_Sensor_Read-Temperature-Humidity-/
│   ├── Temp_and_Humidity_Readings_via_DHT_Sensor.ino   # Main sketch (Uno R4)
│   └── ArduinoProject/libraries/
│       └── Adafruit_Sensor-master/
│
└── ArduinoProject/
    └── libraries/                  # Vendored Arduino libraries
        ├── DHT_sensor_library/
        ├── DHT11-2.1.0/
        ├── Adafruit_Sensor-master/
        ├── Adafruit_Unified_Sensor/
        ├── ArduinoGraphics/
        ├── ArduinoJson/
        ├── Arduino_JSON/
        ├── Arduino_ESP32_OTA/
        ├── LiquidCrystal_I2C/
        ├── AceWire/
        └── … (others)
```

---

## Tech Stack

### Arduino Guides lessons
| Component | Role |
|---|---|
| **ESP8266 board** (NodeMCU / Wemos D1 Mini) | Target hardware for all 34 lessons |
| **ESP8266WebServer** | HTTP server running on the board; handles GET/POST routes |
| **ESP8266WiFi** | STA mode WiFi connection and Soft AP mode |
| **ArduinoOTA** | Receives firmware over WiFi, used in Lesson 32 |
| **SPIFFS (`FS.h`)** | Flash filesystem; stores HTML files separately from the sketch |
| **SimpleDHT** | Reads DHT11 temperature and humidity sensor |
| **Servo.h** | PWM-based servo control |
| **Bootstrap 4 + jQuery 3** | UI framework loaded from CDN; served inline in flash strings |

### FFT
| Component | Role |
|---|---|
| **Adafruit ZeroFFT** | ARM CMSIS FFT on 256-sample q15_t arrays |
| **Custom Cooley–Tukey** | Library-free FFT using a 91-element sine lookup table |
| **MATLAB** | Offline spectral analysis and amplitude plot for verification |

### Arduino_Sensor_Read (Uno R4)
| Component | Role |
|---|---|
| **Arduino Uno R4** | Target board (requires built-in LED matrix hardware) |
| **Arduino_LED_Matrix** | Controls the Uno R4's 12×8 built-in LED grid |
| **ArduinoGraphics** | Scrolling text and font rendering on the LED matrix |
| **DHT (Adafruit)** | Reads float temperature and humidity from DHT11 |
| **LiquidCrystal_I2C** | Optional 16×2 I2C LCD output |
| **Wire** | I2C bus communication for LCD |

---

## Getting Started

### Prerequisites

- [Arduino IDE](https://www.arduino.cc/en/software) 1.8.x or 2.x
- Board support packages:
  - **ESP8266 lessons**: install "ESP8266 Community" board package via Boards Manager
    (URL: `http://arduino.esp8266.com/stable/package_esp8266com_index.json`)
  - **Arduino_Sensor_Read**: install "Arduino UNO R4 Boards" via Boards Manager
- Libraries: all required libraries are vendored in `ArduinoProject/libraries/`. Add this folder as your Arduino sketchbook `libraries` path, or install each library individually via the Library Manager.

### Opening a lesson

1. Navigate to the lesson folder, e.g. `Arduino Guides/lessons/Lesson 16_ Control an LED from web/Control_led_web/`
2. Open `Control_led_web.ino` in Arduino IDE
3. Set your WiFi credentials — every lesson that uses WiFi has these two lines near the top:
   ```cpp
   const char* ssid = "............";
   const char* password = "........";
   ```
   Replace the dots with your network name and password.
4. Select the correct board and port under **Tools**, then click **Upload**.

### Running the FFT sketch

1. Open `FFT/FFT.ino`
2. To test with hard-coded signal data, set `USE_HARD_CODED_DATA 1`; to sample from hardware, set it to `0`
3. Adjust `SAMPLE_SIZE` (must be a power of 2, max 2048) and `SAMPLING_RATE` (Hz) to match your signal
4. Upload and open **Serial Monitor** at 9600 baud to read frequency bins and the dominant frequency

### Running the Uno R4 temperature sketch

1. Open `Arduino_Sensor_Read-Temperature-Humidity-/Temp_and_Humidity_Readings_via_DHT_Sensor.ino`
2. Wire DHT11 data pin to digital pin 2
3. To enable LCD output, change `#define LCD 0` to `#define LCD 1` and set `LCD_ADDRESS`, `LCD_COLLUMNS`, `LCD_ROWS` for your display
4. To enable DHT data printing and scrolling, change `#define DHT_DATA 0` to `#define DHT_DATA 1`
5. Upload to Arduino Uno R4; temperature and humidity scroll across the built-in LED matrix

---

## Usage / Code Examples

### Connecting to WiFi and serving a page (pattern used in Lessons 15–25)

```cpp
#include <ESP8266WebServer.h>

const char* ssid     = "your-network";
const char* password = "your-password";

ESP8266WebServer server(80);

void setup() {
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) delay(500);
  Serial.println(WiFi.localIP());        // open this IP in a browser

  server.on("/", []() {
    server.send(200, "text/html", "<h1>Hello</h1>");
  });
  server.begin();
}

void loop() {
  server.handleClient();
}
```

### Controlling a GPIO from an HTTP POST (Lesson 16 pattern)

```cpp
server.on("/led", []() {
  String state = server.arg("state");   // reads POST body field "state"
  digitalWrite(LED, state == "On" ? HIGH : LOW);
  server.send(200, "text/html", "ok");
});
```

### Reading dominant frequency from FFT

```cpp
#define SAMPLE_SIZE   256
#define SAMPLING_RATE 80

q15_t data[SAMPLE_SIZE];
// ... fill data[] with analogRead() samples ...
ZeroFFT(data, SAMPLE_SIZE);

float max_amp = 0, dominant = 0;
for (int i = 0; i < SAMPLE_SIZE / 2; i++) {
  if (data[i] > max_amp) {
    max_amp  = data[i];
    dominant = FFT_BIN(i, SAMPLING_RATE, SAMPLE_SIZE);
  }
}
Serial.print("Dominant frequency: ");
Serial.println(dominant);   // Hz
```

### Scrolling sensor data on Uno R4 LED matrix

```cpp
void matrix_print_text(String text, unsigned long speed, unsigned long delay_ms) {
  matrix.beginDraw();
  matrix.stroke(0xFFFFFFFF);
  matrix.textScrollSpeed(speed);
  matrix.textFont(Font_5x7);
  matrix.beginText(0, 1, 0xFFFFFF);
  matrix.println(text);
  matrix.endText(SCROLL_LEFT);
  matrix.endDraw();
  delay(delay_ms);
}
// Usage:
matrix_print_text("Temp: 23.50C", 50, 0);
```

---

## License & Contributing

No license is specified in this repository. The lesson sketches carry credit notices pointing to [www.plusivo.com](https://www.plusivo.com); the Plusivo curriculum is their intellectual property. Personal experiment code (FFT, sensor reading) is original work.

This is a personal learning journal rather than a maintained open-source project. Pull requests are not expected.

---

## Contact

- **GitHub**: [darknick131](https://github.com/darknick131)

---

## Assumptions to Verify

- The Plusivo lesson sketches explicitly use `ESP8266WebServer.h` and NodeMCU-style `D1`/`D6`/`D7` pin names, indicating an **ESP8266 board** target — not the Arduino Uno R4. The Uno R4 is the primary personal board, but the guided lessons appear to run on a separate ESP8266 kit board.
- `FFT_Paul/` uses `SAMPLING_RATE 1000` vs the main `FFT/` sketch's `SAMPLING_RATE 80`; the reason for this difference is not documented in either sketch.
- The ABX00087 datasheets at the repo root suggest an Arduino Nano ESP32 is also in use, but no sketch in this repo explicitly targets that board — it may be a reference document for future work.
- The `Arduino_Sensor_Read` sketch references `LCD_ADDRESS`, `LCD_COLLUMNS`, and `LCD_ROWS` as macros but does not define them in the visible file; they must be defined elsewhere (a separate header not committed, or in the library itself) before the LCD path can compile.
