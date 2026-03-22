# 🔧 Hardware Components: Detailed List & Specifications

---

## 📋 Overview

This document provides a comprehensive list of all hardware components required for the Solar-Commute system, including specifications, sourcing information, and usage context.

---

## 🎛️ CATEGORY 1: MICROCONTROLLER & COMPUTING

### 1.1 Main Microcontroller - ESP32 DevKit-C v4

| Specification | Value |
|---------------|-------|
| **Processor** | Dual-core 240 MHz (Xtensa LX6) |
| **RAM** | 8MB SRAM + 8MB PSRAM |
| **Flash** | 4MB SPI Flash (QSPI) |
| **GPIO Pins** | 30 pins broken out |
| **PWM Channels** | 16 (flexible timing) |
| **ADC Channels** | 12 (0-4095 counts) |
| **UART** | 3 (UART0, UART1, UART2) |
| **I2C** | 2 (I2C0, I2C1) |
| **SPI** | 3 (plus dedicated SPI for memory) |
| **WiFi** | 802.11 b/g/n, 2.4 GHz |
| **Bluetooth** | Classic + BLE |
| **Power Supply** | 5V USB or external 3.3V |
| **Current Draw** | 80-200 mA active, <5 µA sleep |
| **Operating Temp** | 0-40°C recommended, -40-80°C rated |
| **Size** | 50mm × 25mm |
| **Cost** | $10-15 USD |

**Usage in System**:
- Central processing unit for all control logic
- Reads all sensors via UART/I2C/ADC
- Writes to all actuators via GPIO/PWM/SPI
- Logs data to SD card
- WiFi telemetry (future)
- Real-time scheduling via FreeRTOS

**Language/Tools**:
- Arduino IDE (easy for beginners)
- PlatformIO (professional development)
- ESP-IDF (native SDK, low-level control)

---

## 📡 CATEGORY 2: SENSORS

### 2.1 GPS Module - NEO-6M

| Specification | Value |
|---------------|-------|
| **Type** | u-blox NEO-6M (civilian) |
| **Update Rate** | 1 Hz (5-10 Hz can be configured) |
| **Accuracy** | ±2.5m (95% CE) typical |
| **Cold Start Time** | 27 seconds |
| **Warm Start** | 3 seconds |
| **Satellites** | 22 channels, 12 parallel |
| **Protocol** | NMEA 0183 @ 9600 baud (default) |
| **Operating Temp** | -40 to +85°C |
| **Supply Voltage** | 2.7 - 4.3V (typically 3.3V) |
| **Current Draw** | 45-50 mA active |
| **Antenna** | Ceramic patch (25mm × 25mm) |
| **Antenna Gain** | +3 dBi |
| **Antenna Connector** | SMA or MMCX (check module) |
| **Operating Frequency** | L1, 1575.42 MHz |
| **Size** | 16mm × 12mm × 2mm (module only) |
| **Cost** | $25-40 USD |

**Sample NMEA Output**:
```
$GPRMC,123456,A,1308.567,N,08027.345,E,45.2,287,200326,,,D*7C
       ↑      ↑ ↑   ↑     ↑    ↑    ↑↑ ↑ ↑↑↑↑↑↑↑↑↑↑↑
       |      | |   |     |    |    || | ||||||||||||
    Time Status Lat Lat Dir Lon Lon Dir Speed Course Date
    (000000)  (DD.ddd)        (DD.ddd)  (deg) (deg) (DDMMYY)
```

**Connection**:
```
NEO-6M    →  ESP32
VCC       →  3.3V
GND       →  GND
TX        →  GPIO 16 (UART2 RX)
RX        →  GPIO 17 (UART2 TX)
```

**Usage**: Real-time location tracking, route verification, speed logging

---

### 2.2 Voltage & Current Sensor - INA219

| Specification | Value |
|---------------|-------|
| **Type** | Texas Instruments INA219 |
| **Voltage Range** | 0-26V DC (can range via resistor divisor) |
| **Shunt Voltage** | ±320mV (full scale) |
| **Shunt Resistor** | 0.1Ω (for 3.2A range) |
| **Current Range** | ±3.2A (depends on shunt) |
| **ADC Resolution** | 16-bit |
| **Accuracy** | ±0.8% |
| **Interface** | I2C (400 kHz typical) |
| **Address** | 0x40-0x47 (8 different addresses via pins) |
| **Operating Temp** | -40 to +125°C |
| **Supply** | 3.3V or 5V |
| **Current Draw** | <1 mA @ 5V |
| **Size** | 8-pin SOIC or 3mm × 3mm BGA |
| **Cost** | $2-5 USD per chip |

**Measurements Provided**:
- Shunt voltage (Vshunt)
- Bus voltage (Vbus)
- Computed current (Ibus)
- Computed power (P)

**Typical Setup for Solar Monitoring**:
```
INA219 #1: Battery voltage/current monitor
├─ Shunt: 0.1Ω in series with battery
├─ Input voltage divider: Use to measure 48V (scale to 26V max)
└─ I2C Address: 0x40 (default)

Voltage scaling example (48V → 16V input):
├─ Resistor divider: 100kΩ + 33kΩ
├─ Ratio: 33/(100+33) = 0.247
├─ 48V input → 11.9V to INA219 ✓ (safe)
└─ Calibration: Read 12V → actual 48V

INA219 #2: Solar array monitor
├─ Shunt: 0.05Ω (for higher current range ~6.4A)
├─ Direct voltage: ~50V output from panels
├─ I2C Address: 0x41 (configure A0 pin high)
└─ Measures: Solar panel output
```

**Usage**: Monitors solar generation, battery charge/discharge, calculates energy

---

### 2.3 Temperature & Humidity Sensor - DHT22

| Specification | Value |
|---------------|tandwidth |
| **Type** | DHT22 (AM2302) |
| **Measurement Range** | Temp: -40 to +80°C; Humidity: 0-100% RH |
| **Accuracy** | ±0.5°C, ±2% RH |
| **Resolution** | 0.1°C, 0.1% RH |
| **Update Rate** | 0.5 Hz (1 reading every 2 seconds) |
| **Response Time** | 1-2 seconds |
| **Operating Voltage** | 3.3-6V DC |
| **Current Draw** | Average 0.5-1mA (50mA peak during measurement) |
| **Protocol** | Proprietary DHT (single-wire digital) |
| **Operating Temp** | 0-50°C (for sensor itself) |
| **Size** | 4-pin DIP, 15mm × 25mm |
| **Cost** | $3-6 USD each |

**Connection** (single-data line):
```
DHT22  →  ESP32
VCC    →  5V (with 10kΩ pull-up resistor to data line)
GND    →  GND
Data   →  GPIO 14 (configurable)
```

**Communication Protocol**:
```
1. MCU sends START signal (pull data low 18ms)
2. DHT22 responds (pulls low ~80µs, then high ~80µs)
3. DHT22 transmits 40 bits:
   ├─ 16 bits humidity (integer + 1 decimal)
   ├─ 16 bits temperature (integer + 1 decimal)
   └─ 8 bits CRC checksum
4. Decode: 1 = high pulse >50µs, 0 = high pulse <50µs
5. Checksum validates before accepting data
```

**Placement in Bus** (multiple units):
- **Sensor 1**: Inside battery enclosure (critical for thermal monitoring)
- **Sensor 2**: Near motor/drive system (if EV)
- **Sensor 3**: Inside control electronics box (ambient at electronics)
- **Sensor 4**: External mounting (ambient air reference)

**Usage**: Thermal management, predictive battery SoC (temperature affects capacity)

---

### 2.4 Inertial Measurement Unit - MPU-6050

| Specification | Value |
|---------------|-------|
| **Type** | Invensense MPU-6050 |
| **Accelerometer** | ±16g, 16-bit resolution |
| **Gyroscope** | ±2000°/sec, 16-bit resolution |
| **Temperature Sensor** | Internal, ±3°C accuracy |
| **Update Rate** | Configurable 1 kHz - 8 kHz |
| **DMP (Digital Motion Processor)** | Yes (built-in sensor fusion) |
| **Interface** | I2C (400 kHz standard) |
| **I2C Address** | 0x68 (default) or 0x69 (configurable) |
| **Operating Temp** | -40 to +85°C |
| **Supply Voltage** | 3.3V (or 5V with level shifter) |
| **Current Draw** | 3.8 mA typical |
| **Size** | 4mm × 4mm × 0.9mm BGA package (requires breakout board) |
| **Cost** | $3-8 USD (breakout boards more affordable) |

**Orientation Reference** (how to interpret readings):
```
          Y (forward)
          ↑
          │  
    X ←───┼───→ X
       (right)
          │
          ↓
       -Y (backward)
          
Z: Up/Down (perpendicular to plane)
  +Z: Up (out of bus roof)
  -Z: Down (into bus floor)

Accelerometer readings:
  Normal stationary: accel_z ≈ 1.0g (gravity pointing down)
  Hard braking: accel_x < -5g (deceleration forward)
  Turning left: accel_y < -2g (centripetal acceleration)
  Collision front: accel_x > 8g + accel_z < -3g
```

**DMP Features**:
- Quaternion calculation (orientation)
- Gravity vector estimation
- Activity recognition
- 6-axis sensor fusion (more accurate than raw sensors)

**Usage**: Collision detection, hard braking detection, orientation monitoring, tilt detection (suspension failure indicator)

---

### 2.5 Passenger Counter Sensors (Multiple Types)

#### 2.5.1 Door Frame Pressure Mats
- **Type**: Resistive pressure-sensitive mat
- **Size**: 1.0m × 0.3m (fits door frame)
- **Resistance**: ~100kΩ inactive, <10kΩ when pressed
- **Connection**: Analog input to GPIO ADC or digital interrupt
- **Cost**: $30-50 each (need 2: entry + exit)

#### 2.5.2 Thermal Camera - MLX90640
- **Type**: Melexis MLX90640 thermal camera
- **Resolution**: 32×24 pixels
- **Field of View**: 55°H × 35°V
- **Temperature Range**: -40 to +85°C
- **Accuracy**: ±3°C typical
- **Output**: I2C (frame every 33ms)
- **Power**: 14mA @ 3.3V
- **Cost**: $50-70 USD

**Thermal imaging logic**:
```
Algorithm: Blob Detection
1. Read 32×24 thermal frame (~768 pixels)
2. Threshold: Pixels > 30°C = potential person
3. Morphology: Connect nearby pixels → blobs
4. Count blobs, estimate passenger count
5. Filter noise: Require blob size > X pixels (person-sized)
6. Output: Passenger count (0-80)
```

#### 2.5.3 Weight Distribution Sensors (Seat Pressure)
- **Type**: Thin pressure sensors embedded in seats
- **Principle**: Capacitive or resistive change under pressure
- **Array**: One per seat (45-50 sensors total) or per row (10-15)
- **Connection**: Multiplexed ADC or digital presence detect
- **Cost**: $200-400 USD for full seat array retrofit

---

## 🎬 CATEGORY 3: ACTUATORS

### 3.1 Door Control Servo Motor

| Specification | Value |
|---------------|-------|
| **Type** | 10kg-cm digital servo |
| **Voltage** | 5-6V DC |
| **Torque** | 10 kg-cm @ 5V (8.2 kg-cm @ 4.8V) |
| **Speed** | 0.12 sec/60° |
| **Weight** | 40-60 grams |
| **Rotation** | 180° (typical) |
| **Control** | PWM signal (1-2ms pulse) |
| **PWM Frequency** | 50 Hz (20ms period) |
| **Envelope** | Aluminum or metal gearbox |
| **Bearings** | Ball bearings (smoother than bushing) |
| **Cost** | $5-15 USD |

**PWM Encoding**:
```
Signal timing: 20ms period (50 Hz)
├─ 1.0 ms pulse: -90° (full counterclockwise)
├─ 1.5 ms pulse: 0° (center)
└─ 2.0 ms pulse: +90° (full clockwise)

For door lock (0° = locked, 90° = unlocked):
├─ Lock:   1.0-1.2 ms pulse
├─ Middle: 1.4-1.6 ms pulse
└─ Unlock: 1.8-2.0 ms pulse

ESP32 PWM setup:
├─ GPIO 12: PWM Pin
├─ Frequency: 50 Hz (servo frequency)
├─ Resolution: 10-bit (0-1023)
└─ Duty cycle: 25-125 (out of 1023) for 1-2ms
```

**Usage**: Automatic door lock/unlock control (backup to main door system)

---

### 3.2 Addressable RGB LED Strip - WS2812B

| Specification | Value |
|---------------|-------|
| **Type** | Individually addressable LED |
| **Voltage** | 5V DC |
| **Current per LED** | 60 mA @ full white, 20 mA @ full red/green/blue |
| **Color Options** | 24-bit: 256 Red × 256 Green × 256 Blue (16M colors) |
| **Brightness Levels** | 0-255 |
| **Protocol** | Single-wire PWM (GRB timing) |
| **Update Rate** | >100 Hz refresh capable |
| **Daisy-chain** | Yes (series connection) |
| **Typical Strip Size** | 5m with 60 LEDs/m (300 LEDs total) |
| **Individual LED Size** | 5mm × 5mm or surface-mount |
| **Operating Temp** | -20 to +60°C |
| **Cost** | $10-20 USD for 5m strip |

**Color Encoding**:
```
Each LED requires 24 bits (3 bytes):
Byte 1: Green (0-255)
Byte 2: Red   (0-255)
Byte 3: Blue  (0-255)

Examples:
├─ Red:    0xFF, 0x00, 0x00
├─ Green:  0x00, 0xFF, 0x00
├─ Blue:   0x00, 0x00, 0xFF
├─ Yellow: 0xFF, 0xFF, 0x00
├─ White:  0xFF, 0xFF, 0xFF
└─ Off:    0x00, 0x00, 0x00
```

**Placement in Bus**:
- **4 LEDs** on driver dashboard (battery, solar, temp, status)
- **2 LEDs** on door frame (passenger indicator)
- **3 LEDs** on roof (position lights: front center, rear center, side markers)
- **4 LEDs** interior zones (passenger notifications)

**Usage**: Status indication, warnings, ambient lighting

---

### 3.3 Speaker/Buzzer System

#### 3.3.1 Piezo Buzzer (Alerts)
- **Type**: Piezoelectric buzzer
- **Frequency**: ~4 kHz (fixed tone for simple model)
- **Voltage**: 5-12V DC
- **Current**: 50-100 mA
- **Sound Level**: 85 dB
- **Connection**: GPIO digital output
- **Cost**: $1-3 USD

#### 3.3.2 Audio Amplifier & Speaker (Announcements)
- **Amplifier**: PAM8403 (2×3W stereo)
- **Voltage**: 5V USB power
- **Speaker**: 4Ω or 8Ω, 2-3W power handling
- **Frequency Response**: 20 Hz - 20 kHz
- **Connection**: Audio amplifier → GPIO/I2S (digital audio)
- **Cost**: $5-10 USD (amplifier), $5-15 USD (speaker)

**Audio Output Scenarios**:
```
Startup:    Beep tone (0.5s, 1 kHz)
Warning:    Double beep (0.3s each, 2 kHz)
Error:      Siren pattern (variable 1-3 kHz, 2 sec)
Door open:  Chime sound (melodic)
Announce:   Pre-recorded voice files (USB storage)
```

---

## 🔋 CATEGORY 4: POWER SYSTEMS

### 4.1 Lithium-ion Battery Pack (LiFePO₄)

| Specification | Value |
|---------------|-------|
| **Chemistry** | LiFePO₄ (Lithium Iron Phosphate) |
| **Nominal Voltage** | 48V DC (16S arrangement) |
| **Capacity** | 200Ah (9.6 kWh total energy) |
| **Cell Configuration** | 16S4P (16 series, 4 parallel) |
| **Max. Voltage** | 54.4V (full charge) |
| **Min. Voltage** | 40V (safe cutoff) |
| **Usable Window** | 48V to 40V = 40% practical depth of discharge (DoD) |
| **Max Current** | 200A (1C rate, full discharge in 1 hour) |
| **Continuous Current** | 100A (safe for extended periods) |
| **Peak Current** | 250A (emergency, <5 seconds) |
| **Cycle Life** | 3000-5000 cycles (~10+ years) |
| **Temperature Range** | -20 to +60°C (operating range for Li-ion) |
| **Self-discharge** | <0.3%/month (very low - can sit for months) |
| **Weight** | ~120 kg (heavy, requires sturdy mounting) |
| **Dimensions** | ~1.2m × 0.8m × 0.4m (example: varies by design) |
| **BMS** | Integrated battery management system |
| **Cost** | $4,000-6,000 USD |

**BMS Functions**:
- Voltage monitoring (per cell)
- Current monitoring (charge/discharge)
- Temperature monitoring
- Cell balancing (active)
- Short circuit protection
- Over/under voltage cutoff
- State of Charge (SoC) calculation

**Safety Features Built-In**:
- Fuses on each cell group
- Thermal sensors
- Disconnect relay
- Pre-charge circuit (soft start)
- CAN bus communication interface

**Configuration Details**:
```
16 Series = 16 cells × 3.2V nominal = 51.2V nominal ≈ 48V
4 Parallel = 4 string runs in parallel: 200Ah ÷ 4 = 50Ah per cell needed
Each cell: 50Ah LiFePO₄ prismatic cells

Voltage at different SoC:
├─ 100% SoC: 54.4V (full charge)
├─ 80% SoC: 50.2V (optimal operating point)
├─ 50% SoC: 48.0V (mid-point)
├─ 20% SoC: 43.2V (start limiting power)
└─ 0% SoC: 40.0V (absolute minimum, stop discharge)
```

---

### 4.2 MPPT Charge Controller

| Specification | Value |
|---------------|-------|
| **Type** | Maximum Power Point Tracking (MPPT) |
| **PV Input Range** | 20-120V DC (typical for this power range) |
| **Output Voltage** | 48V DC (fixed for LiFePO₄ nominal) |
| **Max. Input Current** | 50A (limited by controller) |
| **Max. Output Current** | 50A (charging current) |
| **Max. Power Handling** | 2500W (50V × 50A) at input = ~2400W at output (96% efficient) |
| **Efficiency** | >98% (excellent) |
| **MPPT Tracking Method** | Incremental conductance or perturb-and-observe |
| **Tracking Speed** | Adjusts every 10-100ms (very responsive) |
| **Temperature Compensation** | Automatic (temperature sensor input) |
| **Over-charge Protection** | Yes (disconnects at max voltage) |
| **Over-current Protection** | Yes (fuse rated for 50A) |
| **Operating Temp** | -20 to +60°C |
| **Available Models** | EPEVER MPPT series (60/100, 80/60, etc.) |
| **Cost** | $200-400 USD |

**MPPT Algorithm Operation**:
```
MAXIMUM POWER POINT TRACKING:

Solar panel has characteristic curve:
        Power (W)
        600 ┤      ╱╲  (optimal point here)
            │     ╱  ╲
        400 ├───╱────╲───
            │  ╱        ╲
        200 ├╱──────────╲
            │                ╲
          0 ├────────────────╲___
            └──┴──┴──┴──┴──┴──┴──
            0  10 20 30 40 50  Voltage (V)

MPPT controller:
1. Measures: PV voltage, PV current
2. Calculates: Power = V × I
3. Compares: Current power vs. last power
4. Adjusts: V and I to find peak
5. Repeats: Every 10-100ms

Result: Continuous optimization to maximum power point
  → Increase solar harvest by 15-25% vs. fixed regulation
  → Smart charging prevents overcharge
  → Temperature adjusts for thermal losses
```

---

### 4.3 DC-DC Converters (12V and 5V)

#### 4.3.1 48V to 12V Converter

| Specification | Value |
|---------------|-------|
| **Type** | Isolated DC-DC converter (buck) |
| **Input** | 48V nominally (40-60V operational) |
| **Output** | 12V DC, stable ±5% |
| **Max Output Current** | 20A (240W) |
| **Isolation** | 1500V DC isolation (high safety) |
| **Efficiency** | >92% at full load |
| **Operating Freq** | 100 kHz (typical) |
| **Protection** | Over-current, thermal shutdown |
| **Operating Temp** | -20 to +70°C |
| **Size** | 10cm × 8cm × 5cm (typical module) |
| **Cost** | $15-25 USD |

**Usage**: Power for door motors, backup systems

#### 4.3.2 48V or 12V to 5V Converter

| Specification | Value |
|---------------|-------|
| **Type** | Isolated DC-DC or USB converter |
| **Input** | 48V or 12V DC (wide range: 10-60V models available) |
| **Output** | 5V DC, ±5% regulation |
| **Max Output Current** | 10A (50W) |
| **Isolation** | 1000V DC isolation |
| **Efficiency** | >90% |
| **Protection** | Over-current foldback, thermal |
| **USB Outputs** | Optional (2-4 USB ports for convenience) |
| **Cost** | $5-15 USD |

**Usage**: Power microcontroller, sensors, communication modules

---

### 4.4 Solar Panels (Photovoltaic Array)

| Specification | Value |
|---------------|-------|
| **Type** | Monocrystalline Silicon |
| **Efficiency** | 18-20% (typical) |
| **Panel Size** | 1.5m × 1.0m × 40mm (200W typical) |
| **Weight per panel** | 20-25 kg |
| **Rated Power** | 200W (under STC: 1000W/m², 25°C cell temp) |
| **Voltage @ STC** | 37V open circuit, 30V @ max power |
| **Current @ STC** | 8.5A max power |
| **Temperature Coeff.** | -0.4%/°C (efficiency drops in heat) |
| **Operating Temp** | -10 to +70°C (module surface) |
| **Degradation** | ~0.7% annually (UV, thermal cycling) |
| **Warranty** | 25 years (90% power at 25 years) |
| **Cost per panel** | $150-250 USD |

**Array Configuration** (for our bus):
```
SOLAR ARRAY:
├─ Panel 1: 200W (roof front)
├─ Panel 2: 200W (roof center)
├─ Panel 3: 200W (flexible, roof curve)
└─ Total: 600W rated capacity

Actual output in Chennai:
├─ Peak sun hours (equivalent): 3.5-4.0 per day
├─ Peak output: 600W × 0.80 (angle factor) = 480W typical
├─ Average: 300W (accounting for angle + time variation)
├─ Daily energy: 600W × 3.5 hrs ÷ efficiency = ~1.0-1.2 kWh usable
```

**Mounting Consideration**:
```
OPTIMAL ANGLE FOR CHENNAI:
Latitude: 13°N
Optimal tilt in winter: 25-30° (southern exposure)
Optimal tilt in summer: 10-15° (less angle, sun very high)
Compromise: Fixed at 25° year-round
├─ Captures less in summer
├─ Captures more in winter
└─ Averaged: ~85-90% of theoretical maximum
```

---

## 🔌 CATEGORY 5: POWER DISTRIBUTION & PROTECTION

### 5.1 Main Distribution Board / Breaker Panel

**Components**:
- **Main circuit breaker**: 30A (rated for max system current)
- **Sub-breakers**: 15A, 20A, 5A (for individual rails)
- **Fuses**: Backup protection on each rail
- **Disconnect switches**: Manual on/off capability
- **Diodes**: OR gate (solar auto-select or manual switch)
- **Contactor relay**: For emergency power shutdown

**Wiring Gauge** (American Wire Gauge, AWG):
```
Main 48V from battery (50A continuous):
├─ Wire: 6 AWG copper (13mm² area)
├─ Voltage drop: 50A × wire resistance @ 5m = ~0.3V acceptable
└─ Cost: ~$2-3 per meter

12V secondary circuit (20A):
├─ Wire: 10 AWG copper
├─ Length: typically 2m
└─ Acceptable: Minimal voltage drop

5V logic circuits (10A):
├─ Wire: 14-16 AWG copper
└─ Typically short runs: <1m
```

---

### 5.2 Fuses & Breakers

```
PROTECTION HIERARCHY:

LEVEL 1: Main Breaker
├─ 30A Double-pole breaker
├─ Rating: 48V DC (not AC!)
└─ Purpose: Disconnect entire system

LEVEL 2: Sub-breakers
├─ Motor circuit: 25A breaker
├─ 12V rail: 15A breaker
├─ 5V logic: 10A breaker
└─ Redundancy: If sub-breaker fails, main protects

LEVEL 3: Component Fuses
├─ LED circuit: 5A fuse
├─ Door solenoid: 10A fuse
├─ Sensor arrays: 2A fuse per group
└─ Fine-grain protection

CRITICAL: Use DC-rated breakers/fuses (not AC)
├─ AC breakers: Can't extinguish DC arcs (fires!)
├─ DC breakers: Explosion-proof design prevents arcing
└─ Cost: Higher, but essential for safety
```

---

## 📊 CATEGORY 6: DATA LOGGING & COMMUNICATION

### 6.1 Micro SD Card & Module

| Specification | Value |
|---------------|-------|
| **Card** | MicroSD/MicroSDHC (32GB typical) |
| **Module** | SPI SD card reader |
| **Protocol** | SPI (not I2C) |
| **CS Pin** | Any GPIO (e.g., GPIO 5) |
| **Data Rate** | 25 MHz SPI clock |
| **Read Speed** | ~5-10 MB/s |
| **Write Speed** | ~2-5 MB/s |
| **Operating Voltage** | 3.3V (level shifting needed from 5V if applicable) |
| **Cost** | Card: $5-10; Module: $2-3 USD |

**Data Logging Example**:
```
FILE STRUCTURE: /logs/data.csv
timestamp,gps_lat,gps_lon,battery_v,battery_a,solar_w,temp_c,passengers
2026-03-22 08:00:00,13.0827,80.2707,48.1,+12.5,285,-0.05,8
2026-03-22 08:00:10,13.0828,80.2708,48.1,+12.3,290,-0.03,8
2026-03-22 08:00:20,13.0829,80.2710,48.1,+12.1,295,+0.02,12
...

Write rate: Every 10 seconds
Data per line: ~100 bytes
Throughput: 10 lines/sec × 100 bytes = 1 kB/sec = 1 MB/minute
Storage: 32 GB × 60 min/hour = 32,000 MB / 1 MB per minute = 32,000 minutes ≈ 22 days

Typical usage: Keep 30-day rolling buffer
```

---

### 6.2 LCD/OLED Display for Driver Dashboard

#### 7-inch Touch Screen Display
- **Resolution**: 1024 × 600 pixels
- **Interface**: SPI bus
- **Voltage**: 5V
- **Current**: 300-400 mA
- **Response time**: <50ms (touch)
- **Viewing angle**: 160° (good visibility from driver seat)
- **Cost**: $40-70 USD

#### 16×2 Character LCD (Backup Display)
- **Type**: I2C interface (simpler than parallel)
- **Voltage**: 5V
- **Current**: 50 mA
- **Characters**: 16×2 display
- **Cost**: $5-10 USD

---

## 📐 CATEGORY 7: MISCELLANEOUS COMPONENTS

### 7.1 Wiring & Connectors

| Item | Specification | Cost |
|------|---------------|------|
| **Main power cable** | 6 AWG, 48V rated, 5m | $10-15 |
| **Secondary wiring** | 10-16 AWG (mixed), 5m each | $15-20 |
| **Anderson Power Connector** | 50A rated (battery interface) | $5-10 |
| **XT90 Connectors** | 90A rated (solar to controller) | $3-5 |
| **M4 bolts & terminals** | 100 pack (for connections) | $3-5 |
| **Busbars** | 48V distribution (optional but recommended) | $20-30 |

---

### 7.2 Monitoring & Debugging

**USB-TTL Serial Adapter**:
- Type: CP2102 or CH340 chip
- Purpose: Program & debug ESP32 via USB
- Cost: $2-5 USD

**Terminal block connectors**:
- Purpose: Easy wire-in/wire-out for testing
- Cost: $0.50-1 each

---

## 💰 COMPONENT COST SUMMARY

| Category | Items | Cost (USD) |
|----------|-------|----------|
| **Microcontroller** | 1 ESP32 | $15 |
| **Sensors** | GPS, INA219 (×2), DHT22 (×4), MPU6050, counters | $200 |
| **Actuators** | Servos, LEDs, displays, speaker | $150 |
| **Power (high-cost)** | Battery 9.6kWh + MPPT + Solar panels | $8,000 |
| **Distribution** | Breakers, fuses, wiring, connectors | $150 |
| **Data Logging** | SD card + module | $15 |
| **Misc** | Enclosures, mounting, cables, etc. | $200 |
| **TOTAL ESTIMATED** | **~$8,730 USD** | |

**Break-down by major cost drivers**:
- Battery pack: 60-65% of total
- Solar panels: 10-12% of total
- MPPT controller: 3-4% of total
- Sensors & actuators: 8-10% of total
- Everything else: 12-15% of total

---

## 🔗 Next Steps

1. Review [System Architecture](02-system-architecture.md) to understand how components integrate
2. Study [Embedded System](04-embedded-system.md) for connection details
3. Read [Physics Concepts](06-physics-concepts.md) for performance calculations
4. Check [Project Phases](08-project-phases.md) for procurement planning

---

**Component selection is critical to system reliability and performance. Each component was chosen for a specific reason—understanding the "why" helps troubleshoot issues when they arise.**
