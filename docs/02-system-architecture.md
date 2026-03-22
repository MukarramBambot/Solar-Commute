# 🏗️ System Architecture: Complete Overview

---

## 🎯 Architecture Overview

The Solar-Commute system operates on a **layered architecture** model:

```
       ┌─────────────────────────────────────────────────┐
       │          USER INTERFACE LAYER                   │
       │   ┌───────────────────────────────────────┐    │
       │   │ Driver Dashboard | Passenger Display │    │
       │   │ Real-time Status | Alert Notifications│   │
       │   └───────────────────────────────────────┘    │
       └────────────────┬────────────────────────────────┘
                        │
       ┌────────────────▼────────────────────────────────┐
       │      CONTROL & LOGIC LAYER                      │
       │   ┌───────────────────────────────────────┐    │
       │   │ Microcontroller (Arduino/ESP32)       │    │
       │   │ - Data Processing                     │    │
       │   │ - Decision Making                     │    │
       │   │ - System Control                      │    │
       │   │ - Communication Protocol              │    │
       │   └───────────────────────────────────────┘    │
       └────────────────┬────────────────────────────────┘
                        │
       ┌────────────────▼────────────────────────────────┐
       │      SENSOR & ACTUATOR LAYER                    │
       │   ┌──────────────┬──────────────┐             │
       │   │   SENSORS    │   ACTUATORS  │             │
       │   ├──────────────┼──────────────┤             │
       │   │ • GPS Module │ • LED Lights │             │
       │   │ • Volt/Curr  │ • Door Motor │             │
       │   │ • Temp Sensor│ • Displays   │             │
       │   │ • Pass Count │ • Buzzers    │             │
       │   └──────────────┴──────────────┘             │
       └────────────────┬────────────────────────────────┘
                        │
       ┌────────────────▼────────────────────────────────┐
       │       POWER & HARDWARE LAYER                    │
       │   ┌───────────────────────────────────────┐    │
       │   │ • Solar Panels (600W capacity)        │    │
       │   │ • Battery Pack (200Ah, 48V)           │    │
       │   │ • DC Bus (switching & distribution)   │    │
       │   │ • Motor Drive (if propulsion)         │    │
       │   │ • Thermal Management System           │    │
       │   └───────────────────────────────────────┘    │
       └─────────────────────────────────────────────────┘
```

---

## 📊 System Components Breakdown

### 1. **3D Bus Model** (Visual Component)

**Purpose**: Represents the physical bus with all integrated systems

**Key Elements**:
```
EXTERIOR:
├── Solar Panel Array
│   ├── Roof-mounted panels (400W)
│   ├── Side-panel arrays (optional, 200W)
│   └── Mounting structure
├── Body Structure
│   ├── Frame (aluminum/steel)
│   ├── Windows and doors
│   └── Safety bars
└── Wheels & Suspension
    ├── 4 wheels (standard bus size)
    └── Shock absorption

INTERIOR:
├── Seats
│   ├── Driver seat
│   ├── Passenger seats (45-50 capacity)
│   └── Wheelchair accessibility
├── Dashboard
│   ├── Display screen for driver
│   ├── Control buttons
│   └── Status indicators
└── Sensor Placement
    ├── Passenger counters (door frame)
    ├── Temperature sensors (roof)
    ├── GPS antenna (roof)
    └── Camera mounts (corners)
```

### 2. **Embedded System** (Control Component)

**Purpose**: Real-time monitoring, decision-making, and control

**Key Subsystems**:

#### A. Microcontroller Unit (MCU)
- **Primary**: ESP32 (32-bit dual-core processor, WiFi/Bluetooth)
- **Secondary**: Arduino Mega 2560 (as backup/redundancy)
- **Operating System**: Real-time kernel (FreeRTOS)
- **Memory**: 8MB PSRAM for data logging

#### B. Sensor Acquisition
```
Input Devices:
├── GPS Module (NEO-6M)
│   └── Updates: 1Hz, accuracy: ±2.5m
├── Voltage/Current Sensor (INA219)
│   └── Measures: Battery and solar input
├── Temperature Sensor (DHT22)
│   └── Monitors: Battery, motor, ambient
├── Passenger Counter
│   ├── Door pressure sensors
│   ├── Thermal camera
│   └── Ultrasonic sensors
└── Inertial Measurement Unit (MPU-6050)
    └── Detects: Acceleration, tilt, collisions
```

#### C. Actuator Control
```
Output Devices:
├── LED Indicators (WS2812B programmable)
│   ├── Status lights
│   └── Warning indicators
├── Door Control Motor
│   ├── Servo-based lock
│   └── Solenoid emergency release
├── Display Units
│   ├── Driver dashboard (7" touchscreen)
│   └── Passenger display (destination/next stop)
└── Audio System
    ├── Speaker (announcements)
    └── Buzzer (alerts)
```

#### D. Power Management
- **Input**: 48V DC from solar/battery
- **Output**: 
  - 5V for sensors and microcontroller
  - 12V for actuators and displays
  - 24V for door systems
- **Protection**: Fuses, relays, diode arrays
- **Monitoring**: Voltage/current on each rail

### 3. **Power System** (Energy Component)

**Purpose**: Generate, store, and distribute electrical energy

```
ENERGY FLOW:
┌──────────────┐
│ Solar Panels │ (600W max output)
│   (48V DC)   │
└──────┬───────┘
       │ (During Day)
       ▼
┌──────────────────┐     ┌──────────────────┐
│ Battery Charger  │────▶│ Li-ion Battery   │
│ (MPPT Controller)│     │ Pack (48V, 200Ah)│
└──────────────────┘     └────────┬─────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    │ (Discharges when needed) │
                    ▼
            ┌──────────────────┐
            │ Power Distribution│
            │     Bus (48V)     │
            └────┬────────┬────┬┘
                 │        │    │
        ┌────────▼┐  ┌───▼──┐ │
      48V→12V   48V→5V   │    │
        DC/DC   DC/DC    │    │
      Converter Converter │  Load
        │        │       │   Management
       12V       5V      │
       Bus       Bus     │
        │        │       │
  Actuators   Sensors  Direct 48V
  & Displays  & MCU    Loads
```

---

## 🔄 Data Flow Architecture

### System Data Flow Diagram

```
        ┌─────────────────────────────────────┐
        │      SENSOR DATA ACQUISITION        │
        └──────┬──────────┬──────────┬────────┘
               │          │          │
        ┌──────▼──┐  ┌────▼────┐  ┌─▼──────┐
        │   GPS   │  │ Sensors │  │ Power  │
        │ (1Hz)   │  │(Various)│  │Monitor │
        └──────┬──┘  └────┬────┘  └─┬──────┘
               │          │         │
               └──────────┼─────────┘
                    ┌─────▼────────────┐
                    │   Microcontroller │
                    │  (Data Processing)│
                    │                  │
                    │  • Aggregate     │
                    │  • Validate      │
                    │  • Filter        │
                    │  • Analyze       │
                    └──┬────────────┬──┘
                       │            │
          ┌────────────┘            └─────────────┐
     ┌────▼────┬────────────┬──────────┬──────────┐
     │ Display │ Log Storage│ Alert    │ Optimize │
     │ Update  │ (SD Card)  │ Generate │ Controls │
     │ (Driver)│            │          │          │
     └────┬────┴────────────┴──────────┴──┬───────┘
          │                               │
     ┌────▼──────────────────────┬────────▼────┐
     │     ACTUATOR CONTROL      │  OPTIMIZATION │
     │                          │   ALGORITHMS   │
     │ • LED Status            │               │
     │ • Door Control          │ • Route Plan  │
     │ • Alerts & Sound        │ • Speed Plan  │
     │ • Display Update        │ • Power Alloc │
     └────────────────────────┴───────────────┘
```

---

## 🔌 Electrical System Specification

### Main Subsystems

#### 1. **Solar Power Generation**
```
Solar Array:
├── Panel 1: 200W Monocrystalline
├── Panel 2: 200W Monocrystalline
└── Panel 3: 200W Flexible (curved roof)
   │
   └─→ MPPT Charge Controller
       ├── Input: 0-65V (adjusts to 48V DC)
       ├── Output: 48V DC @ 12.5A max
       └── Features: Efficiency tracking, overcharge protection
```

#### 2. **Energy Storage**
```
Lithium-ion Battery Pack:
├── Configuration: 48V (16S4P arrangement)
├── Capacity: 200Ah (9.6kWh total energy)
├── Chemistry: LiFePO₄ (safe, long-lived)
├── BMS: Integrated battery management system
├── Features:
│   ├── Cell balancing
│   ├── Temperature monitoring
│   ├── Overcharge/discharge protection
│   └── Fault detection
└── lifespan: 10+ years (3000+ cycles)
```

#### 3. **Power Distribution**
```
Main 48V Bus:
├── Fuses/Breakers for protection
├── Diode OR gate (Solar | Battery selection)
├── Contactor for load shedding (emergency mode)
└── Splits to:
    ├── Motor Drive (if EV propulsion) - 30A
    ├── DC/DC Converter to 12V - 20A
    ├── DC/DC Converter to 5V - 10A
    └── Reserve capacity - 10A
```

---

## 🎮 Control Logic Architecture

### Decision-Making Process

```
REAL-TIME MONITORING LOOP (Every 100ms):

START
  │
  ├─→ Read all sensors
  │   ├── Battery voltage/current
  │   ├── Solar panel output
  │   ├── Temperature
  │   ├── GPS location
  │   └── Passenger count
  │
  ├─→ Validate sensor data
  │   ├── Check data ranges
  │   ├── Detect sensor faults
  │   └── Mark unreliable data
  │
  ├─→ Update state variables
  │   ├── Bus location
  │   ├── Energy available
  │   ├── Occupancy level
  │   └── System health
  │
  ├─→ Evaluate conditions
  │   ├── Is battery low? (trigger charge mode)
  │   ├── Is temperature high? (cooling needed)
  │   ├── Is passenger count unusual? (alert driver)
  │   └── Has route changed? (update plan)
  │
  ├─→ Execute control actions
  │   ├── Adjust power distribution
  │   ├── Update displays
  │   ├── Trigger alerts
  │   └── Log data
  │
  └─→ END (repeat every 100ms)
```

### Feature Control Logic

#### Energy Optimization Loop
```
IF solar_output > required_power THEN
  → Charge battery at available rate
  → Divert excess to non-critical loads
ELSE IF battery_level > 80% THEN
  → Discharge to support loads
ELSE IF battery_level < 20% THEN
  → Activate eco-mode (reduce non-critical power)
  → Alert driver
ELSE
  → Normal operation
END IF
```

---

## 📡 Communication Architecture

### Internal Communication

**Protocol**: CAN Bus (Controller Area Network)
- **Speed**: 250 kbps
- **Reliability**: Error detection and correction
- **Topology**: Linear bus with terminators

**Connected Devices**:
- MCU (ESP32)
- Battery Management System
- Motor Controller (if EV)
- Remote IO modules
- Future: Solar charge controller

### External Communication

**WiFi (ESP32 built-in)**:
- Real-time telemetry upload
- Over-the-air (OTA) firmware updates
- Remote monitoring (optional)

**GPS/GNSS**:
- Location tracking (1Hz update)
- Route verification
- Speed measurement

---

## 🛡️ Safety & Redundancy Systems

### Critical System Safeguards

```
SAFETY LAYER:
│
├─→ Sensor Redundancy
│   ├── Multiple temperature sensors (cross-check)
│   ├── Dual voltage monitoring
│   └── Passenger count validation (3 methods)
│
├─→ Fault Detection
│   ├── Out-of-range sensor checks
│   ├── Rate-of-change monitors
│   └── Watchdog timers
│
├─→ Graceful Degradation
│   ├── Single sensor failure → use redundant
│   ├── Partial system loss → limp-home mode
│   ├── Critical loss → safe stop + alert
│   └── Always maintain safety-critical functions
│
├─→ Manual Override
│   ├── Driver can override door control
│   ├── Emergency stop button
│   ├── Manual brake engagement
│   └── Direct access to main contactor
│
└─→ Logging & Alerts
    ├── All critical events logged
    ├── Automatic alerts on faults
    ├── Data retention (30 days minimum)
    └── Remote monitoring capability
```

---

## 📊 System Specifications Summary

| Aspect | Specification |
|--------|----------------|
| **Operating Voltage** | 48V DC (main bus) |
| **Peak Power Draw** | 12 kW (all systems) |
| **Continuous Power** | 3-5 kW (normal operation) |
| **Solar Generation** | 600W peak (3 hour average: 300W) |
| **Battery Capacity** | 9.6 kWh (200Ah @ 48V) |
| **Operating Range** | 150-200 km (single charge) |
| **GPS Update Rate** | 1 Hz (1 fix per second) |
| **Data Logging** | 10MB/hour |
| **Temperature Range** | -10°C to +60°C operating |
| **Precision** | ±5% for power measurements |

---

## 🔍 Component Interactions

### Example Scenario: Evening Operation (Low Solar)

```
TIME: 5:00 PM (solar generation decreasing)

1. GPS detects 15km remaining in route
2. Solar panel output drops below 100W
3. Battery voltage still 48V (good charge)
4. System calculates energy required: 750Wh
5. Available energy (battery) will allow completion
6. Door opens 3 times (passenger load factor: 80%)
7. Temperature rises to 45°C
8. Cooling fan activates (draws 200W temporarily)
9. Battery discharge increases to 3000W
10. Energy optimization: Extends journey by 5 mins to reduce power
11. Route completion with 15% battery remaining
12. Alert: Next charge opportunity recommended
```

---

## 🎓 Understanding System Integration

### How All Parts Work Together

1. **Monitoring**: Sensors continuously collect real-time data
2. **Processing**: MCU analyzes data and makes decisions
3. **Actuation**: Based on logic, actuators execute commands
4. **Feedback**: Results feed back to monitoring, creating control loop
5. **Logging**: All important events recorded for analysis
6. **Optimization**: Historical data improves future decisions

This creates a **closed-loop control system** that continuously improves efficiency and safety.

---

## 📚 Next Steps

1. Review [Bus 3D Design](03-bus-3d-design.md) to see how components are physically arranged
2. Study [Embedded System](04-embedded-system.md) for detailed hardware specifications
3. Read [Hardware Components](07-hardware-components.md) for technical details on each device
4. Explore [Physics Concepts](06-physics-concepts.md) to understand underlying principles

---

**This architecture represents a modern, scalable system that prioritizes reliability, efficiency, and safeguards while remaining maintainable and updatable.**
