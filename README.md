# Hardware & Firmware Skeleton

**Author**: Minseok Lim (임민석)  
**Role**: Hardware Design & Firmware Foundation

---

## Goal
Make the project run on **ESP32 + Wokwi** with a clean skeleton that others can plug their modules into.

---

## Responsibilities

### 1. Hardware / Wokwi / PlatformIO
- Design & maintain **`diagram.json`** (ESP32, HX711, RTC, LCD, servo, buttons)
- Configure **`platformio.ini`** (board, libraries, SPIFFS/partitions)
- Ensure the basic sketch uploads and runs in Wokwi

### 2. Firmware Skeleton in `main.cpp`

#### Includes & Pin Definitions
```cpp
#define HX711_DT_PIN   4
#define HX711_SCK_PIN  5
#define SERVO_PIN      18
#define BUTTON_DISPLAY 12
#define BUTTON_SETTING 13
#define BUTTON_UP      14
#define BUTTON_DOWN    15
```

#### Global Objects
```cpp
RTC_DS1307 rtc;
LiquidCrystal_I2C lcd(0x27, 20, 4);
Servo feedServo;
HX711 scale;
```

#### Basic `setup()`
- Initialize I2C, LCD, RTC, servo, HX711
- Show simple "Pet Feeder v2.0 – Initializing" on LCD

#### Basic `loop()` Skeleton
```cpp
void loop() {
  DateTime now = rtc_ok ? rtc.now() : ...;
  currentWeight = readWeight(feedingActive, feederOpen);
  handleButtons();     // Person A
  coreLogicLoop();     // Person B
  handleApiLoop();     // Person C
  maybeUpdateDisplay(); // Person A
}
```

### 3. Hardware Helper Functions

| Function | Description |
|----------|-------------|
| `readWeight(...)` | Real HX711 + fake `simWeight` for Wokwi |
| `openFeeder()` | Servo control - open position |
| `closeFeeder()` | Servo control - close position |
| `updateDisplay()` | Basic version (Person B can extend) |
| `handleButtons()` | Read pins, call manual feed, slot setting |
| `resetSystemState()` | Hardware part: reset simWeight/tare, close servo, refresh LCD |

### 4. State & Data Structures

#### Feeding Slot Structure
```cpp
struct FeedingSlot {
  bool active;
  int hour, minute;
  float weight;
};
FeedingSlot slots[3];
```

#### Flags & Variables
```cpp
// State flags
bool feedingActive;
bool feederOpen;
bool manualMode;

// Weight tracking
float currentWeight;
float currentTargetWeight;

// Timing
unsigned long feedingStartMs;
unsigned long lastWeightChangeMs;
```

#### Trigger Guard
```cpp
int lastTriggerYear;
int lastTriggerMonth;
int lastTriggerDay;
int lastTriggerHour;
int lastTriggerMinute;
```

#### Safety Constants
```cpp
const unsigned long FEED_TIMEOUT_MS;
const unsigned long STUCK_WINDOW_MS;
const float MIN_INCREASE_G;
```

---

## Hardware Configuration

### Components Used

| Component | Type | Quantity |
|-----------|------|----------|
| ESP32 DevKit C V4 | Microcontroller | 1 |
| HX711 | Load Cell Amplifier | 1 |
| DS1307 | RTC Module | 1 |
| LCD 2004 | I2C Display | 1 |
| Servo Motor | Actuator | 1 |
| Push Button (Red) | Input | 1 |
| Push Button (Blue) | Input | 2 |
| Push Button (Green) | Input | 1 |
| Breadboard Half | Prototyping | 1 |

### Pin Mapping

| Component | Pin | ESP32 GPIO | Wire Color |
|-----------|-----|------------|------------|
| HX711 DT | Data | GPIO 4 | Blue |
| HX711 SCK | Clock | GPIO 5 | Green |
| Servo PWM | Control | GPIO 18 | Orange |
| I2C SDA | Data | GPIO 21 | Blue |
| I2C SCL | Clock | GPIO 22 | Green |
| Button Display | Input | GPIO 12 | Red |
| Button Up | Input | GPIO 14 | Blue |
| Button Down | Input | GPIO 15 | Blue |
| Button Setting | Input | GPIO 13 | Green |

### Power Distribution
- **3.3V Rail**: ESP32, HX711
- **5V Rail**: RTC, LCD, Servo
- **Common GND**: All components share common ground

---

## Circuit Diagram (`diagram.json`)

### Overview
The `diagram.json` file contains the complete Wokwi circuit schematic with precise component positioning and wiring connections.

### Key Sections

#### 1. Parts Definition
```json
"parts": [
  { "type": "wokwi-breadboard-half", "id": "bb1" },
  { "type": "board-esp32-devkit-c-v4", "id": "esp" },
  { "type": "wokwi-hx711", "id": "cell1", "attrs": { "type": "5kg" } },
  { "type": "wokwi-ds1307", "id": "rtc1" },
  { "type": "wokwi-servo", "id": "servo1" },
  { "type": "wokwi-lcd2004", "id": "lcd2", "attrs": { "pins": "i2c" } },
  { "type": "wokwi-pushbutton", "id": "btn1", "attrs": { "color": "red" } },
  { "type": "wokwi-pushbutton", "id": "btn2", "attrs": { "color": "blue" } },
  { "type": "wokwi-pushbutton", "id": "btn3", "attrs": { "color": "blue" } },
  { "type": "wokwi-pushbutton", "id": "btn4", "attrs": { "color": "green" } }
]
```

#### 2. Connection Groups

**Power Connections**
- ESP32 3.3V → Breadboard positive rail
- ESP32 5V → Breadboard positive rail (separate)
- ESP32 GND → Breadboard negative rails (multiple)

**HX711 Load Cell**
- VCC → 3.3V rail
- GND → Ground rail
- DT → ESP32 GPIO 4
- SCK → ESP32 GPIO 5

**I2C Bus (RTC + LCD)**
- SDA line → ESP32 GPIO 21
- SCL line → ESP32 GPIO 22
- Both devices share the I2C bus

**Servo Motor**
- PWM → ESP32 GPIO 18
- V+ → 5V rail
- GND → Ground rail

**Buttons (Pull-up Configuration)**
- All buttons connected between GPIO pins and GND
- Internal pull-up resistors enabled in firmware

#### 3. Layout Positions
Components are positioned for optimal wiring and visualization:
- ESP32: Top-left reference point
- Breadboard: Center-right
- HX711: Top-center
- RTC: Top-right
- LCD: Upper-right
- Servo: Middle-right
- Buttons: Bottom-right (arranged in a grid)

### Wiring Color Scheme
- **Red**: Power (3.3V, 5V)
- **Black**: Ground
- **Blue**: Data signals (I2C SDA, HX711 DT)
- **Green**: Clock signals (I2C SCL, HX711 SCK)
- **Orange**: PWM (Servo control)

---

## PlatformIO Configuration

### `platformio.ini`
```ini
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino

lib_deps = 
    adafruit/RTClib @ ^2.1.1
    marcoschwartz/LiquidCrystal_I2C @ ^1.1.4
    madhephaestus/ESP32Servo @ ^0.13.0
    bogde/HX711 @ ^0.7.5

monitor_speed = 115200
```

### Libraries Used
- **RTClib**: Real-time clock management
- **LiquidCrystal_I2C**: LCD display control
- **ESP32Servo**: Servo motor control
- **HX711**: Load cell amplifier interface

---

## Deliverables

### Files Provided
1. **`diagram.json`** - Complete Wokwi circuit schematic
   - Component definitions
   - Precise positioning coordinates
   - Wire routing paths
   - Color-coded connections

2. **`platformio.ini`** - Project configuration
   - Board settings
   - Library dependencies
   - Serial monitor configuration

3. **`main.cpp`** - Firmware skeleton
   - Hardware initialization
   - Helper functions
   - Integration hooks for team members

---

## Setup Instructions

### 1. Open in Wokwi
- Load `diagram.json` in Wokwi simulator
- All components will be automatically positioned
- Wiring connections are pre-configured

### 2. PlatformIO Setup
```bash
# Initialize project
pio init

# Install dependencies
pio lib install

# Build and upload
pio run --target upload
```

### 3. Verify Hardware
- Check LCD shows "Pet Feeder v2.0 – Initializing"
- Test buttons (should register in serial monitor)
- Verify servo moves to 0° on startup
- Confirm RTC communication (check for error messages)

---

## Testing

### Simulation Mode (Wokwi)
```cpp
#define SIM_FAKE_WEIGHT 1
```
- Uses simulated weight values
- No real sensor required
- Perfect for testing logic

### Hardware Mode
```cpp
#define SIM_FAKE_WEIGHT 0
```
- Reads actual HX711 data
- Requires proper calibration
- Real-world deployment

---

## Integration Points

This skeleton provides clean hooks for team collaboration:

All modules share access to:
- Global state variables
- Hardware control functions
- Sensor readings

---

**Contact**: Minseok Lim (임민석) | Sejong University | Student ID: 20011935
