# ⚙️ Embedded System Design: Arduino/ESP32 Control System

---

## 🎯 Overview

The embedded system is the "brain" of the Smart Bus. It monitors all sensors, makes real-time decisions, and controls all actuators. This document explains the hardware architecture and control principles.

**Key Role**: Real-time monitoring, data processing, and automated control

---

## 🔧 Microcontroller Selection

### ESP32 vs Arduino: Comparison

| Feature | ESP32 | Arduino Mega 2560 | Verdict |
|---------|-------|------------------|--------|
| **Processor** | Dual-core 240MHz | Single 16MHz | ESP32 (40x faster) |
| **RAM** | 8MB PSRAM | 8KB | ESP32 (1000x more) |
| **Storage** | 4MB Flash | 256KB | ESP32 (16x more) |
| **WiFi/BLE** | Built-in | None (need module) | ESP32 |
| **I/O Pins** | 34 GPIO | 54 pins | Arduino (more, but unnecessary) |
| **Power Draw** | 80-200mA active | 50mA active | Arduino (lower, but ESP32 manages) |
| **Cost** | $8-15 | $12-20 | Similar |
| **Heat** | Moderate | Very low | Arduino (advantage) |
| **Community** | Growing | Mature | Arduino, but ESP32 catching up |

### Recommended Choice: **ESP32**

**Reasons**:
1. ✅ Sufficient computation power for control logic
2. ✅ Built-in WiFi for telemetry (future expansion)
3. ✅ Enough RAM for data buffering
4. ✅ Multiple UARTs for sensor communication
5. ✅ Good for real-time applications
6. ✅ Debugging via USB (development friendly)

**Specific Module**: **ESP32-DevKit-C v4**
- Readily available
- Good through-hole headers for prototyping
- Integrated USB programming
- 30 GPIO pins (enough for our needs)

---

## 📡 Sensor Integration

### Sensor List & Specifications

#### 1. **GPS Module - NEO-6M**

```
PURPOSE: Traffic localization and route tracking

SPECIFICATIONS:
├── Position Accuracy: ±2.5m (typical)
├── Update Rate: 1 Hz (1 fix per second)
├── Satellites Tracked: 22 channels, 12 in parallel
├── Cold Start: ~27 seconds
├── Warm Start: ~3 seconds
├── Power Draw: 45mA @ 3.3V
└── Output: NMEA protocol via UART

CONNECTION:
├── VCC → 3.3V (ESP32)
├── GND → GND
├── TX → RX pin (ESP32 UART2)
└── RX → TX pin (ESP32 UART2)

DATA OUTPUT (every 1 second):
{
  "latitude": 13.0827,
  "longitude": 80.2707,
  "altitude": 6.5,
  "speed_kmh": 45.2,
  "heading": 287,
  "satellite_count": 12,
  "fix_quality": "3D"
}

USAGE:
├── Route verification
├── Distance tracking
├── Speed monitoring
└── Real-time map display
```

#### 2. **Voltage/Current Sensor - INA219**

```
PURPOSE: Battery and solar power monitoring

SPECIFICATIONS:
├── Voltage Range: 0-26V
├── Shunt Voltage: ±320mV (programmable)
├── Current Range: ±3.2A (programmable)
├── ADC Resolution: 16-bit
├── Accuracy: ±0.8%
├── Power Draw: <1mA @ 5V
├── Address: I2C (0x40, 0x41, etc.)
├── Sample Rate: 10-32k samples/sec
└── Communication: I2C @ 100-400kHz

TYPICAL SETUP:
├── Shunt Resistor: 0.1Ω (3.2A full-scale)
├── Bus voltage divider: 1:1 (for 48V measurement)
└── Filter: 10µF capacitor for noise

CONNECTION:
├── SDA → SDA (ESP32 pin 21)
├── SCL → SCL (ESP32 pin 22)
├── GND → GND
└── VCC → 3.3V

MULTIPLE UNITS NEEDED:
1. Battery Bus Monitor (0-50A, 0-60V)
2. Solar Input Monitor (0-15A, 0-60V)
3. Load Rail Monitor (0-20A, 12V)

MEASUREMENT CYCLE (every 200ms):
{
  "battery_voltage": 48.2,
  "battery_current": 12.5,
  "battery_power": 603,
  "solar_voltage": 53.1,
  "solar_current": 8.2,
  "solar_power": 435
}

CALCULATIONS:
├── Energy consumed: Power × Time
├── Battery SoC: Calculated from voltage curve
├── Charging rate: Current over time
└── Efficiency: Output/Input power ratio
```

#### 3. **Temperature Sensor - DHT22**

```
PURPOSE: Thermal monitoring (battery, motor, ambient)

SPECIFICATIONS:
├── Temperature Range: -40 to +80°C
├── Humidity Range: 0-100% RH
├── Accuracy: ±0.5°C
├── Resolution: 0.1°C
├── Power Draw: 4mA (peak), 0.5µA (idle)
├── Response Time: 1-2 seconds
├── Output: DHT Protocol (proprietary)
└── Update Rate: 0.5 Hz (1 reading every 2 seconds)

PLACEMENT (multiple units):
├── Sensor 1: Inside battery enclosure
├── Sensor 2: Near motor (if EV propulsion)
├── Sensor 3: Inside electronic box (for component temp)
└── Sensor 4: Ambient air (external mount)

CONNECTION (each):
├── VCC → 5V (with 10kΩ resistor to GND)
├── GND → GND
└── DATA → GPIO pin (ESP32)

READING PROTOCOL:
1. MCU sends trigger to sensor
2. Wait 1-2ms for response
3. Read 40-bit serial data
4. Parse humidity (16-bit) and temp (16-bit)
5. Calculate actual values (apply calibration offset)

DATA OUTPUT (every 2 seconds):
{
  "battery_temp_c": 42.3,
  "motor_temp_c": 45.1,
  "box_temp_c": 38.5,
  "ambient_temp_c": 35.2,
  "humidity_percent": 62
}

THRESHOLDS:
├── Warning: > 50°C (reduce load)
├── Critical: > 60°C (activate emergency cooling)
├── Low temp: < 0°C (battery disconnected for safety)
└── Nominal: 25-45°C (optimal range)
```

#### 4. **Passenger Counter Sensor - Multiple Methods**

```
PURPOSE: Detect number of passengers on bus

CONFIGURATION (Redundant for accuracy):

METHOD 1: Door Frame Pressure Sensors
├── Type: Pressure-sensitive mats
├── Count: 2 (one at entry, one at exit door)
├── Output: 1 if pressed, 0 if not
├── Logic: Entry passengers += 1 (on entry press)
│          Exit passengers += 1 (on exit press)

METHOD 2: Thermal Camera
├── Type: MLX90640 (32×24 pixel thermal camera)
├── FOV: 55°H x 35°V
├── Temperature Range: -40 to +85°C
├── Power: 14mA @ 3.3V
├── Output: I2C thermal array
├── Logic: Count warm blobs (human bodies)

METHOD 3: Ultrasonic Sensor
├── Type: HC-SR04 (budget option)
├── Range: 2cm - 4m
├── Accuracy: ±3cm
├── Power: 15mA peak @ 5V
├── Output: Pulse width (time to echo)
├── Logic: Measure seat occupancy density

FUSION ALGORITHM:
Combined_Count = (Method1 × 0.5) + (Method2 × 0.3) + (Method3 × 0.2)

CONNECTION (Example: Pressure sensors):
├── Entry mat: GPIO 32
├── Exit mat: GPIO 33
└── Logic uses rising edge interrupt
```

#### 5. **Inertial Measurement Unit (IMU) - MPU-6050**

```
PURPOSE: Detect collisions, acceleration, orientation

SPECIFICATIONS:
├── Accelerometer: ±16g range, 16-bit resolution
├── Gyroscope: ±2000°/sec, 16-bit resolution
├── Temperature Sensor: Built-in (calibration)
├── Power Draw: 3.8mA @ 3.3V
├── I2C Address: 0x68 (default)
├── Sample Rate: 1-8000 Hz (configurable)
└── Sensitivity: 16,384 LSB/g

MEASUREMENTS PROVIDED:
{
  "accel_x": -0.05,    # g (gravity units)
  "accel_y": 0.02,
  "accel_z": 1.00,
  "gyro_x": 5.2,       # °/sec
  "gyro_y": -2.1,
  "gyro_z": 8.4,
  "temp": 38.5         # °C
}

CONNECTION:
├── SDA → SDA (ESP32 pin 21)
├── SCL → SCL (ESP32 pin 22)
├── GND → GND
├── VCC → 3.3V
└── INT → GPIO 35 (interrupt pin, optional)

FEATURES:
├── Digital Motion Processor (DMP): On-chip fusion
├── Activity recognition: Detects motion start/stop
├── Collision detection: Sudden force changes
└── Calibration: Auto-calibrates on startup

USAGE SCENARIOS:
1. Collision Detection:
   IF |accel_x| > 5 && |accel_y| > 5 THEN
     → Trigger emergency alert
     → Log event with timestamp
     → Notify driver

2. Hard braking detection:
   IF accel_x < -8 THEN
     → Activate stability control
     → Reduce solar power draw (engine assist)

3. Orientation tracking:
   Quaternion = DMP_Fusion(accel, gyro)
   Angle = Extract_Pitch_Roll_Yaw(Quaternion)
   → Detect if bus is tilted (suspension failure)
```

---

## 🎮 Actuator Control

### Actuator Specifications

#### 1. **LED Status Indicators - WS2812B (Addressable)**

```
PURPOSE: Visual status indication to driver and passengers

SPECIFICATIONS:
├── Type: RGB LED (individually addressable)
├── Count: 12-24 (distributed around interior)
├── Protocol: SPI-like (1-wire)
├── Power: 60mA per full-brightness LED @ 5V
├── Brightness Levels: 0-255
├── Update Rate: >100 Hz
└── Total Power: ~1.5A (all on full white)

PLACEMENT:
├── Dashboard: 4 indicators (Battery, Solar, Temp, Status)
├── Door Frame: 2 indicators (Entry/Exit activity)
├── Roof: 3 position lights (front, back, emergency)
└── Interior zones: 3-4 for passenger information

CONNECTION:
├── Data → GPIO 5 (ESP32)
├── VCC → 5V (via capacitor: 100µF)
├── GND → GND
└── Note: Use level shifter (3.3V → 5V) for signal

COLOR SCHEME:
├── Green: System OK, operation normal
├── Blue: Charging mode, solar available
├── Yellow: Warning (temperature high, cargo shifting)
├── Red: Error/Emergency (battery low, collision)
├── Purple: Blink on door open/close
└── Off: System idle/disabled

CONTROL LOGIC:
Status_LED[0] = GREEN        # Battery status
Status_LED[1] = BLUE         # Solar status
Status_LED[2] = YELLOW if temp > 50 else GREEN
Status_LED[3] = RED if battery_low else GREEN

Update every 100ms (10 Hz blink rate for warnings)
```

#### 2. **Door Control Motor - Servo/Solenoid**

```
PURPOSE: Automatic door locking/unlocking

SPECIFICATIONS:
Type A: Servo Motor (smooth operation)
├── Torque: 10 kg-cm
├── Operating Speed: 0.12 sec/60°
├── Power: 5-6V DC, 500mA peak
├── Control: PWM signal (1-2ms pulse)
├── Range: 0° (locked) to 90° (unlocked)

Type B: Solenoid (simple locking)
├── Voltage: 12V DC
├── Current: 1A peak
├── Force: 100N (push)
├── Response Time: 10-20ms
└── Hold power: 0.2W

TYPICAL SETUP:
├── Primary: Servo for smooth control
├── Backup: Solenoid for emergency unlock
└── Manual override: Mechanical key access

CONNECTION (Servo):
├── Signal → GPIO 12 (PWM capable)
├── VCC → 5V (power supply)
└── GND → GND

DOOR CONTROL LOGIC:
```
Event: Passenger press exit button
→ Check: Is door lock engaged? 
  Yes: Unlock via servo
  No: Already unlocked
→ Activate door motor (different hardware)
→ Wait: 3 seconds
→ Lock door again
→ Update LED indicator
```

#### 3. **Display Systems**

```
PURPOSE: Show information to driver and passengers

DRIVER DASHBOARD - 7" Touchscreen (SPI connection)
├── Resolution: 1024×600 pixels
├── Voltage: 5V
├── Current: 400mA (typical)
├── Connection: SPI bus (MOSI, MISO, CLK, CS)
├── Update Rate: 30 FPS (sufficient)

INFORMATION DISPLAYED:
┌────────────────────────────────┐
│  GPS Route │ Passenger Count   │
│  34.5 km   │ 38 of 50          │
├────────────────────────────────┤
│ Battery: 72% ▓▓▓▓░            │
│ Solar: 285W                    │
├────────────────────────────────┤
│ Temp: 42°C │ Speed: 45 km/h   │
│ Status: ✓ Normal               │
└────────────────────────────────┘

PASSENGER DISPLAY - Small LED/LCD (optional)
├── Type: 16×2 LCD or small OLED
├── Display: "Next Stop: Chepauk"
├── Update: Every 500ms (smooth updates)

DRIVER INTERACTION:
├── Touchscreen: For navigation and settings
├── Physical buttons: Emergency stop, door control
├── Voice feedback: Audio announcements (speaker)
```

#### 4. **Audio System - Buzzer & Speaker**

```
PURPOSE: Alerts and passenger announcements

BUZZER (Simple alerts):
├── Type: Piezo buzzer
├── Voltage: 5V DC
├── Current: 50-100mA
├── Sound Level: 85dB
├── Connection: GPIO 13 (can PWM for tones)

SPEAKER (Announcements):
├── Type: 4-8Ω speaker
├── Power: 1W typical
├── Amplifier: Small audio amp (PAM8403)
├── Frequency: 80Hz - 20kHz

AUDIO SCENARIOS:
├── Startup: Beep (0.5s, 1kHz)
├── Warning: Double beep (0.3s, 2kHz each)
├── Error: Siren (varies frequency, 2 sec)
├── Door open: Chime (melodic)
└── Announcement: Pre-recorded voice files (USB storage)

CONNECTION:
├── Buzzer → GPIO 13
├── Speaker → PAM8403 audio amp
├── Audio source → I2S peripheral (digital audio)
└── Amplifier power → 5V
```

---

## 🔌 Power Management System

### Power Distribution Architecture

```
MAIN POWER FLOW:

┌─────────────────────────────────┐
│  Solar Array (600W) + Battery    │
│  Main DC Bus: 48V nominal        │
└────────────┬────────────────────┘
             │
    ┌────────▼────────┐
    │ Main Contactor  │
    │ (Connect/Disc)  │
    └────────┬────────┘
             │
    ┌────────▼──────────────┐
    │ Distribution Board    │
    │ with Fused Outputs    │
    └┬─────┬──────────┬─────┤
     │     │          │     │
    15A   10A        5A    20A
   12V   5V DC/DC   Sensors Actuators
   Rail  Rail
     │     │          │      │
     ▼     ▼          ▼      ▼
  Lights SAC  GPS/   Motor
  Motor  Unit Sensors Controllers
```

### Power Rail Specifications

| Rail | Voltage | Max Current | Protected By | Load |
|------|---------|-------------|--------------|------|
| **Main** | 48V | 30A | Main breaker | Solar/Battery switch |
| **Motor** | 48V | 25A | 25A relay | Propulsion (if EV) |
| **Actuators** | 24V | 15A | 15A fuse | Door, motor drive |
| **12V** | 12V | 15A | 15A fuse | Lights, fans, displays |
| **5V Logic** | 5V | 10A | 10A fuse | MCU, sensors, LEDs |

### Battery Management System (BMS) Integration

```
BMS FUNCTIONS:
1. Voltage monitoring (per cell)
2. Temperature monitoring
3. Current limits (overcharge/discharge protection)
4. Cell balancing (active or passive)
5. Fault detection (short circuit, overtemperature)

BMS INTERFACE:
├── Communication: CAN bus or I2C
├── Signals to MCU:
│   ├── Battery voltage
│   ├── Current (direction: charge/discharge)
│   ├── State of Charge (0-100%)
│   ├── State of Health (% capacity remaining)
│   ├── Temperature
│   └── Fault flags (if any)
│
├── Signals from MCU:
│   └── Enable discharge (if fault condition clears)

BMS THRESHOLDS:
├── Overcharge: > 55 V (disconnect solar)
├── Over-discharge: < 40 V (stop all loads except essentials)
├── Overtemp: > 60°C (reduce power)
├── Over-current: > 50A (emergency stop)
```

---

## 🧠 Control Logic Framework

### Main Control Loop

```
INITIALIZATION (Startup):
1. Configure GPIO pins (inputs/outputs)
2. Initialize I2C bus (for sensors)
3. Initialize UARTs (for GPS and data)
4. Initialize SPI (for displays and mem)
5. Start real-time kernel (FreeRTOS)
6. Create tasks for each subsystem
7. Calibrate sensors
8. Report status

MAIN LOOP (Every 100ms):

┌─────────────────────────────═
│  READ SENSORS (10ms)        │
├─────────────────────────────┤
│ • GPS: Get position, speed  │
│ • Voltage/Current: Power    │
│ • Temperature: All zones    │
│ • Passenger count: Objects  │
│ • IMU: Acceleration, tilt   │
│ • Door: Status signals      │
└─────────────────────────────┘
           ▼
┌─────────────────────────────┐
│  VALIDATE DATA (10ms)       │
├─────────────────────────────┤
│ • Check ranges              │
│ • Detect sensor failures    │
│ • Apply noise filters       │
│ • Fuse redundant values     │
└─────────────────────────────┘
           ▼
┌─────────────────────────────┐
│  UPDATE STATE (20ms)        │
├─────────────────────────────┤
│ • Position history          │
│ • Energy consumption        │
│ • Temperature trends        │
│ • System diagnostics        │
└─────────────────────────────┘
           ▼
┌─────────────────────────────┐
│  EVALUATE CONDITIONS (20ms) │
├─────────────────────────────┤
│ Is battery low?             │
│ Is temperature high?        │
│ Has collision occurred?     │
│ Is door sensor active?      │
│ Are passengers unusual?     │
└─────────────────────────────┘
           ▼
┌─────────────────────────────┐
│  EXECUTE CONTROL (20ms)     │
├─────────────────────────────┤
│ • Update LEDs               │
│ • Control doors             │
│ • Adjust power distribution │
│ • Update displays           │
│ • Trigger alerts            │
└─────────────────────────────┘
           ▼
┌─────────────────────────────┐
│  LOG DATA (20ms)            │
├─────────────────────────────┤
│ • Write to event buffer     │
│ • Send telemetry (WiFi)     │
│ • Update statistics         │
└─────────────────────────────┘
```

### Real-Time Operating System: FreeRTOS

```
TASK STRUCTURE:

Task 1: SENSOR_READ (Priority: 3)
├── Period: 100ms
├── Function: Read all sensors
└── Output: Updated sensor variables

Task 2: CONTROL_LOGIC (Priority: 2)
├── Period: 100ms
├── Function: Make decisions
└── Output: Control commands

Task 3: ACTUATOR_CMD (Priority: 2)
├── Period: 100ms
├── Function: Send commands to hardware
└── Output: GPIO/PWM updates

Task 4: DISPLAY_UPDATE (Priority: 1)
├── Period: 200ms
├── Function: Update screens
└── Output: SPI commands to display

Task 5: DATA_LOG (Priority: 0)
├── Period: 500ms
├── Function: Log to SD or memory
└── Output: Stored data

PRIORITY RULES:
- Higher number = higher priority
- Sensor reading: HIGH (time-critical)
- Control logic: MEDIUM
- Display updates: LOW (can wait)
- Logging: LOWEST (background task)
```

---

## ⚠️ Safety & Fault Handling

### Fault Detection Mechanism

```
CONTINUOUS MONITORING:

1. Sensor Health Check (every 100ms)
   IF sensor_no_update > 2s THEN
     → Mark sensor as FAULTY
     → Use backup method or last known value
     → Log fault event
     → Alert driver if critical
   END IF

2. Range Check
   IF temperature > 70°C THEN
     → Activate emergency cooling
     → Reduce system load
     → Disable non-essential systems
   END IF

3. Consistency Check
   IF |power_in - power_out| > 500W for 5 sec THEN
     → Check for short circuit
     → Verify sensor calibration
     → Reduce solar input if issue persists
   END IF

4. Watchdog Timer
   IF main_loop_miss > 200ms THEN
     → Force system reset
     → Log reset event
     → Activate safe mode
   END IF
```

### Graceful Degradation

```
FAULT SCENARIO: GPS Module Failure

Stage 1: Detection (< 5 sec)
→ No fix updates for 5 seconds
→ Mark GPS as FAULTY

Stage 2: Fallback (Immediate)
→ Use last known position
→ Estimate movement using IMU
→ Display "Position Estimate - Accuracy Unknown"

Stage 3: Limited Operation
→ Continue route using stored waypoints
→ Manual driver confirmation needed
→ Driver can override route

Stage 4: Recovery
→ Once GPS fix restored -> Resume normal operation
→ Log total downtime

System remains SAFE throughout
```

---

## 📊 Data Logging Strategy

### Event Types to Log

```
CRITICAL EVENTS (Timestamp, Duration, Status):
├── System startup/shutdown
├── Sensor failures and recoveries
├── Battery low/high alerts
├── Temperature threshold breaches
├── Collision detection
├── Door open/close cycles
├── Emergency stops
└── Faults and errors

OPERATIONAL DATA (Logged every 1 minute):
├── GPS position (lat, lon, altitude)
├── Battery: voltage, current, capacity
├── Solar: voltage, current, power
├── Temperature: all zones
├── Passenger count
├── Route position (route ID, stop, progress)
└── System status flags

PERFORMANCE METRICS (End of trip):
├── Total energy consumed
├── Total energy generated
├── Distance traveled
├── Average speed
├── Number of stops
├── Passenger count statistics
└── Route efficiency (energy/km)

STORAGE LOCATION:
├── Primary: SD card (32GB) via SPI
├── Backup: Internal flash memory
├── Cloud (optional): WiFi telemetry upload
└── Retention: 30 days rolling buffer
```

---

## 🔗 Communication Protocols

### Inter-Component Communication

```
UART (Serial Communication)
Used for: GPS module
├── Baud Rate: 9600 bps
├── Data Frames: NMEA sentences
├── Structure: $PREFIX,field1,field2,...*checksum
└── Example: $GPRMC,123456,A,1308,N,08027,E,45.2,287,220326*4F

I2C (TWO-wire Interface)
Used for: Temperature, voltage/current sensors, IMU
├── Speed: 400kHz (standard)
├── Address scheme: 7-bit addressing
├── Multi-master capable (but single master here)
└── Example addresses:
    ├── INA219 #1: 0x40
    ├── INA219 #2: 0x41
    ├── DHT22 address: GPIO-based (not I2C)
    ├── MPU-6050: 0x68

CAN Bus (Future expansion)
Used for: Battery BMS, motor controller
├── Baud Rate: 250 kbps
├── Message ID: 11bit
├── Data Length: 0-8 bytes
└── Advantages: Scalable, robust, industrial standard

SPI (Serial Peripheral Interface)
Used for: Displays, SD card, flash memory
├── Clock Speed: 10-50 MHz
├── Mode: SPI Mode 0 (typical)
├── Slave Select: Per device
└── Signals: MOSI, MISO, CLK, CS (Chip Select)

WiFi (Optional, future)
Used for: Remote telemetry and updates
├── Protocol: TCP/IP
├── Frequency: 2.4 GHz (802.11b/g/n)
├── Range: 50-100m (typical)
└── Updates: Over-the-air (OTA) firmware

1-WIRE (Proprietary)
Used for: Addressable LEDs (WS2812B)
├── Protocol: Single-wire PWM
├── Data format: RGB values (8+8+8 bits)
└── Daisy-chainable
```

---

## Summary: System Interaction

```
DATAFLOW OVERVIEW:

Sensors (S) → Microcontroller (C) → Actuators (A)

GPS(S) ─┐
        │
Voltage(S) ┼─→ ESP32 ─┬─→ LEDs(A)
           │    MCU   │
Temp(S) ──┼─→ (48 kB) ├─→ Motors(A)
           │    RAM   │
IMU(S) ───┤      ├─→ Displays(A)
           │      │
Pass(S) ───┤      ├─→ Buzzer(A)
           │      │
Door(S) ───┘      └─→ SD Card(storage)

TIMING DIAGRAM (not to scale):
0ms:    Read sensors ▓▓▓▓▓▓▓ (10ms)
10ms:   Validate data ░░░░░░░ (10ms)
20ms:   Update state ▓▓▓▓▓▓▓ (20ms)
40ms:   Decision logic ░░░░  (20ms)
60ms:   Control output ▓▓▓▓▓ (20ms)
80ms:   Log data ░░░░ (20ms)
100ms:  LOOP REPEAT
```

---

## 📚 Next Steps

1. Review [Hardware Components](07-hardware-components.md) for detailed specs
2. Study [Physics Concepts](06-physics-concepts.md) for mathematical models
3. Read [System Architecture](02-system-architecture.md) for integration details
4. Check [Project Phases](08-project-phases.md) for development timeline

---

**The embedded system is the integrative layer that brings all hardware components together. Understanding its architecture is critical for system success.**
