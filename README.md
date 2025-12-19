# 🛠️ Hardware & Firmware Skeleton
> **Lead:** Minseok (@YourGitHubID)  
> **Core Stack:** ESP32, Wokwi, PlatformIO, C++ (Arduino Framework)

---

## 🎯 Goal
**ESP32**와 **Wokwi 시뮬레이터**를 기반으로 프로젝트의 뼈대를 구축합니다. 다른 팀원들이 자신의 로직(UI, 알고리즘, 통신)을 즉시 결합할 수 있도록 유연하고 안정적인 펌웨어 스켈레톤을 제공하는 것이 목표입니다.

---

## 🏗️ Hardware Configuration (Wokwi)
시뮬레이션 환경인 `diagram.json`을 통해 설계된 하드웨어 구성입니다. **I2C 라인 공유**를 통해 핀 효율성을 극대화했습니다.

| Component | Interface | Pin Assignment | Description |
| :--- | :--- | :--- | :--- |
| **ESP32 DevKit V1** | MCU | - | Main Controller |
| **LCD 1602** | I2C | SDA(21), SCL(22) | Status Display |
| **RTC DS1307** | I2C | SDA(21), SCL(22) | Real-Time Clock |
| **HX711** | Serial | DT(18), SCK(19) | Load Cell (Weight Sensor) |
| **Micro Servo** | PWM | GPIO 13 | Feeder Gate Control |
| **Buttons** | Digital | GPIO 14, 27... | Manual Feed & Settings |

---

## 📂 Development Environment
**PlatformIO**를 사용하여 라이브러리 의존성과 하드웨어 설정을 관리합니다.

```ini
; platformio.ini 핵심 설정
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
lib_deps =
    bogde/HX711
    adafruit/RTClib
    marcoschwartz/LiquidCrystal_I2C
    madhephaestus/ESP32Servo
board_build.partitions = huge_app.csv
