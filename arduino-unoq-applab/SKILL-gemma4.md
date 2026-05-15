# Skill: arduino-unoq-applab

## Description
Guide for developing applications for the Arduino UNO Q, integrating them into the Arduino App Lab ecosystem as apps and bricks, deploying Edge Impulse ML models, and building Flask-based web UIs. Use this skill when users ask about Arduino UNO Q development, App Lab apps/bricks, ML model deployment, or converting existing projects into App Lab-compatible applications.

## Core Knowledge

### 1. Hardware Architecture
The Arduino UNO Q is a dual-processor board:
- **MPU (Microprocessor)**: Qualcomm® QRB2210 running **Debian Linux**. Handles high-performance tasks, Python, and Linux apps.
- **MCU (Microtemplate)**: STM32U585 running **Zephyr OS**. Handles real-time tasks and Arduino sketches.
- **Communication**: The `Arduino_RouterBridge` library enables **RPC (Remote Procedure Call)** between the MPU and MCU via a Unix Domain Socket (`/var/run/arduino-router.sock`).

### 2. Development Environments
#### **Arduino App Lab**
The primary environment for unified development (Apps, Bricks, AI models).
- **App Structure**:
    - `python/main.py`: Mandatory. The entry point for high-level logic running on Linux.
    - `sketch/sketch.ino`: Optional. The C++ sketch that runs on the board's microcontroller.
       - `app.yaml`: Mandatory manifest for App metadata and Bricks.
    - `sketch.yaml`: Mandatory only if a sketch is present (manages sketch libraries).
    - `python/requirements.txt`: For managing Python dependencies (e.g., `numpy`).
    - `README.md`: Optional project documentation displayed in the App Lab UI.
- **Modes**:
    - **SBC Mode**: Board runs standalone.
    - **PC-Hosted Mode**: Board connected via USB-C to a PC.
    - **Network Mode**: Remote access via local network using mDNS.
- **Python Development**:
    - **`App.run()`**: Must be called at the end of `main.py`.
    - **`user_loop`**: Pass a custom function to `App.run(user_loop=loop)` for repetitive tasks.
    - **Bricks**: Modules like Web UI that run as separate processes. Use `Add Brick` in the UI.
- **Sketch Development**:
    - **Libraries**: Use "Add sketch library" in the App Lab sidebar.
    - **Debugging**: Use `Monitor.print()` instead of `Serial.print()` to see logs in the App Lab Console.
- **Workflow**: Combines Arduino sketches (MCU), Python scripts (MPU), and Linux containers.
- **Startup**: Apps can be set to run at boot using `arduino-app-cli properties set default user:<NAME>`.

#### **Arduino IDE 2+**
Used specifically for programming the **MCU (STM32)** side.
- **Core**: Requires `Arduino UNO Q Zephyr Core`.
- **Additional URLs**: `https://downloads.arduino.cc/packages/package_zephyr_index.json`
- **Required Library**: `Arduino_RouterBridge`.

### 3. Setup & Configuration
#### **Linux Host (udev rules)**
Required for USB communication on Linux hosts.
```bash
echo '# Operating mode
SUBSYSTEMS=="usb", ATTRS{idVendor}=="2031", ATTRS{idProduct}=="0078", MODE="0660", TAG+="uaccess"
# EDL mode
SUBSYSTEMS=="usb", ATTRS{idVendor}=="05c6", ATTRS{idProduct}=="9008", MODE="0660", TAG+="uaccess"' | sudo tee "/etc/udev/rules.d/60-Arduino-UNO-Q.rules" && sudo udevadm control --reload-rules && sudo udevadm trigger
```
*(Note: Verify PID/VID with `lsusb` as they may vary).*

### 4. Coding Patterns

#### **LED Matrix (MCU - Arduino)**
Uses `Arduino_LED_Matrix.h`.
```cpp
#include <Arduino_LED_Matrix.h>
Arduino_LED_Matrix matrix;
void setup() {
  matrix.begin();
  matrix.setGrayscaleBits(1); 
}
void loop() {}
```

#### **RGB LEDs (MPU - Python)**
Controllable via the Linux sysfs interface.
```python
import time
from arduino.app_utils import App

def loop():
    with open("/sys/class/leds/red:user/brightness", "w") as f:
        f.write("1\n")
    time.sleep(1)
    with open("/sys/class/leds/red:user/brightness", "w") as f:
        f.write("0\n")
    time.sleep(1)

App.run(user_loop=loop)
```

#### **RGB LEDs (MCU - Arduino)**
Active Low logic.
```cpp
void setup() {
  pinMode(LED3_R, OUTPUT);
  digitalWrite(LED3_R, HIGH); // OFF
}
void loop() {
  digitalWrite(LED3_R, LOW);  // ON
  delay(1000);
}
```

### 5. Key Peripherals
- **Connectivity**: Wi-Fi 5, Bluetooth 5.1 (via WCBN3536A).
- **Sensness**: I2C (Standard `Wire` and 3.3V `Wire1` via Qwiic).
- **Analog**: 14-bit ADC on JANALOG pins.
- **Debug**: 1.8V UART interface for low-level SoC debugging.
