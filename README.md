# Hardware & Firmware Skeleton

* **Lead:** Minseok (@YourGitHubID)
* **Target Platform:** ESP32 + Wokwi Simulator
* **Core Responsibility:** Hardware circuit design, firmware skeleton development, and sensor/actuator driver interfacing.

---

## 1. Hardware Configuration (Pin Map)

This pin map follows the design defined in `diagram.json`. Please ensure all connections match the table below.

| Component | ESP32 Pin | Interface | Power | Note |
| :--- | :--- | :--- | :--- | :--- |
| **HX711 (Load Cell)** | DT: 4, SCK: 5 | Serial | 3.3V | Weight measurement |
| **Micro Servo** | GPIO 18 | PWM | 5V | Feeding gate control |
| **RTC DS1307** | SDA: 21, SCL: 22 | I2C | 5V | Time keeping |
| **LCD 2004** | SDA: 21, SCL: 22 | I2C | 5V | Status display (20x4) |
| **Button (Red)** | GPIO 12 | Digital | GND | Display mode toggle |
| **Button (Green)** | GPIO 13 | Digital | GND | Setting / Manual feed |
| **Button (Blue UP)** | GPIO 14 | Digital | GND | Increment value |
| **Button (Blue DOWN)**| GPIO 15 | Digital | GND | Decrement value |

---

## 2. Firmware Architecture

The firmware utilizes a modular skeleton structure to facilitate collaboration between team members.

### Weight Data Wrapper
A wrapper function supports both Wokwi simulation and real-world hardware.
* **SIM_FAKE_WEIGHT 1:** Simulates weight increase during feeding in Wokwi.
* **SIM_FAKE_WEIGHT 0:** Reads actual units from the HX711 sensor.

### Main Control Loop
The `loop()` function synchronizes hardware data and calls hooks for specific logic handlers.

```cpp
void loop() {
  server.handleClient(); // Web API Handling (Person C)

  // 1. Hardware Data Synchronization (Minseok)
  DateTime now = rtc_ok ? rtc.now() : ...;
  currentWeight = readWeight(feedingActive, feederOpen);

  // 2. Button & Menu Logic (Person A)
  // Input detection and state management for settingState/manualState

  // 3. Scheduling & Feeding Algorithm (Person B)
  if (settingState == NOT_SETTING && manualState == MANUAL_IDLE) {
    checkScheduledFeeding();
  }
  if (feedingActive) {
    monitorFeeding();
  }

  // 4. Display Update (Person A/B)
  updateDisplay();
}
