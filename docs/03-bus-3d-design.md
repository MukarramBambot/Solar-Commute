# 🎨 3D Bus Design Guide: Blender & Unreal Engine

---

## 📐 Overview

This document guides interns through designing a realistic, physically accurate 3D model of a solar-powered smart bus using either **Blender** or **Unreal Engine**.

**Key Goals**:
- Create realistic MTC bus proportions
- Visualize solar panel placement
- Show sensor mounting points
- Demonstrate interior layout
- Maintain optimization for real-time rendering

---

## 🎯 Bus Design Specifications (MTC Context)

### Realistic MTC Bus Dimensions

```
BUS SPECIFICATIONS (Based on MTC Standard Buses):

LENGTH:         11.5 meters
WIDTH:          2.5 meters
HEIGHT:         3.5 meters (with AC)
WHEELBASE:      6.5 meters
CAPACITY:       45-50 passengers
DOOR WIDTH:     0.9 meters
WINDOW COUNT:   8-10 per side
GROUND CLEAR:   0.4 meters

WEIGHT:
├── Chassis: ≈ 6,000 kg
├── Body: ≈ 3,000 kg
├── Solar panels + batteries: ≈ 2,000 kg
└── Total: ≈ 11,000 kg (empty)
    
CAPACITY WITH PAYLOAD:
├── 80 passengers (crush capacity): ≈ 17,000 kg
└── Safe working: 12,500 kg
```

---

## 🚌 EXTERIOR DESIGN

### 1. Bus Body Structure

#### Main Frame
- **Geometry**: Rectangular prism with rounded edges (in Blender: use Bevel modifier)
- **Proportions**: Length:Width:Height = 11.5:2.5:3.5
- **Curves**: Windshield angles (15° forward tilt), roof arch
- **Material**: Aluminum frame (lightweight)

#### Design Steps in Blender:
```
1. Create Base Cube
   → Scale to bus dimensions (11.5 x 2.5 x 3.5)
   → Add loop cuts for detail (windows, panels)

2. Model Windshield
   → Create front face (angled 15°)
   → Use Subdivision Surface for smoothness
   → Add depth for frame

3. Add Roof
   → Create arch using Bezier curve
   → Extrude along length
   → Add gutters

4. Create Side Panels
   → Flat surface for main body
   → Curved sections for wheel wells
   → Mounting points for solar panels
```

### 2. Windows and Doors

#### Window Design
```
WINDOW ARRANGEMENT:
Front:       Windshield (1 large) + Driver side (2)
Sides:       8-10 windows per side, evenly spaced
Rear:        Exit window (2 small) + blind spot monitor
Top:         Optional roof vents

WINDOW SPECIFICATIONS:
├── Type: Tinted glass (dark blue/grey)
├── Reflectivity: 0.4 (for realistic reflection)
├── Transmissivity: 0.7 (allows vision through)
└── Frame: 5cm aluminum trim
```

**Blender Modeling**:
- Create rectangular shapes for windows
- Use Array modifier for repetition
- Apply glass shader with reflectivity
- Add minimal geometry for aluminum frames

#### Door Design
```
DOORS:
Front (Driver):
├── Hinge: Left side
├── Width: 0.5m
├── Height: 2.2m
└── Type: Push-button automated

Passenger Doors (2):
├── Position: Center and rear-center
├── Width: 0.9m each
├── Height: 2.1m
├── Type: Hydraulic sliding doors
└── Safety: Door interlock sensors shown

Emergency Exit:
├── Position: Rear roof
├── Type: Pneumatic roof hatch
└── Visibility: Clear marker on roof
```

**Components to Add**:
- Door handles (cylinder primitives with bevels)
- Lock mechanisms (small mechanical details)
- Hinges (small box shapes)
- Pressure release handles

### 3. Wheels and Suspension

#### Wheel Specification
```
WHEEL CONFIGURATION:

Front Axle:
├── 2 wheels (dual-wheel setup)
├── Diameter: 0.9m (900mm)
├── Width: 0.3m (300mm)
└── Type: Radial tires with tread

Rear Axle:
├── 4 wheels (dual rear axle)
├── Same diameter and width
├── Pneumatic suspension
└── Load bearing: Higher capacity

Tire Specification:
├── Radial: 10R22.5 (Indian standard)
├── Load Index: 148L (high capacity)
├── Speed Rating: L (120 km/h max)
└── Sidewall: Reinforced for weight
```

**Blender Modeling**:
```
1. Create Wheel & Rim
   → Cylinder (radius 45cm)
   → Array for 6 wheels (2 front, 4 rear)
   → Add tread using displacement map

2. Add Suspension
   → Springs: Coil geometry
   → Shock absorbers: Cylinder + piston rod
   → Control arms: Geometry for multi-link suspension

3. Materials
   → Tire: Black rubber (metallic: 0.1, roughness: 0.9)
   → Rim: Chrome (metallic: 1.0, roughness: 0.2)
   → Suspension: Powdercoat grey (metallic: 0.3)
```

---

## ☀️ SOLAR PANEL INTEGRATION

### Solar Panel Arrangement

#### Roof-Mounted Array (Primary)
```
ROOF LAYOUT:
┌─────────────────────────────────────────────┐
│  Panel 1 (200W)  │  Panel 2 (200W)          │
│  Monocrystalline │  Monocrystalline         │
├──────────────────┼──────────────────────────┤
│           Panel 3 (200W Flexible)           │
│           Curved to roof arch               │
└─────────────────────────────────────────────┘

SPECIFICATIONS:
├── Total: 600W capacity
├── Coverage: ~70% of roof
├── Angle: 25° tilt (optimized for Chennai latitude 13°N)
└── Spacing: 5cm between panels (thermal gap)
```

#### Side-Panel Array (Optional)
```
OPTIONAL SIDE PANELS (200W):
├── Mounted on side walls
├── Angle: Perpendicular to main surfaces
└── For afternoon/evening capture
```

### Blender/UE Implementation

#### Creating Solar Panels

**Modeling Approach**:
```
1. Create Panel Geometry
   → Rectangular plane: 1.5m x 1.0m x 0.05m
   → Scale: Multiple times for all panels

2. Add Panel Details
   → Cells: Array of small squares (using planes or geometry)
   → Frame: Border aluminum (thin extrusion)
   → Glass surface: Top face with transparency

3. Material/Shader Setup
   ├── Base: Dark blue (RGB: 25, 35, 65)
   ├── Metallic: 0.1
   ├── Roughness: 0.3
   └── Glass layer overlay (transparency: 0.3)

4. Placement
   ├── Parse position: Roof center (x: 0, y: -0.5, z: 1.8)
   ├── Rotation: 25° tilt along Y-axis
   └── Array modifier: Clone 3 identical panels

5. Lighting Impact
   ├── Add emission shader (slight glow when sunlight hits)
   ├── Adjust based on HDR lighting
   ├── Show reflection of sky on panel surface
```

#### Mounting Structure
```
MOUNTING COMPONENTS:
├── Aluminum frame (L-profile extrusion)
├── Bolts (small cylinders with threads - geometry simplification)
├── Bushings (rubber isolators)
├── Weatherproof wire conduits
└── Junction box (small cube, rear panel)

THERMAL MANAGEMENT:
├── Air gap: 10cm below panels
├── Fin placement: Under panels for airflow
└── Ventilation ducts: Shown leading to cooling system
```

---

## 🏛️ INTERIOR DESIGN

### Driver Area (Cockpit)

#### Specifications
```
DRIVER COMPARTMENT:
├── Length: 1.5m (from front bumper)
├── Width: 2.5m (full bus width)
├── Height: 2.3m (to ceiling)
└── Entry: Via driver-side door

SEATING:
├── Driver seat: Ergonomic, elevated (+0.2m)
├── Position: Offset to left side
├── Type: Suspension seat (shows bounce geometry)
└── Controls: Within arm's reach
```

#### Dashboard Components
```
DASHBOARD LAYOUT:

┌─────────────────────────────────────────────┐
│  7" TOUCHSCREEN DISPLAY (Center)            │
│  ┌─────────────────────────────────────┐   │
│  │ GPS Map | Battery Status | Alerts  │   │
│  │ Route | Passenger Count | Temp     │   │
│  └─────────────────────────────────────┘   │
├─────────────────────────────────────────────┤
│ Steering Wheel    │  Status LED Array      │
│ (3D steering      │  (Green/Yellow/Red     │
│  wheel model)     │   indicators)          │
├─────────────────────────────────────────────┤
│ Speed Display │ Door Control │ AC Control  │
│ (Digital)     │ Buttons      │ Panel       │
└─────────────────────────────────────────────┘

MODELS TO CREATE:
├── Steering wheel: Circular with grip details
├── 7" Display: Thin bezel monitor (optional screen texture)
├── Buttons: Small cylindrical objects with press indication
├── Switches: Toggle switches with labels
└── Indicator lights: Spheres with emissive material
```

### Passenger Area

#### Seating Arrangement
```
INTERIOR LAYOUT (Top View):

Front
  ▲
  │           Driver  │Passenger
  │           ─────────────────
  │
  │  ┌─┬─┐ ┌─┬─┐  ┌─┬─┐ ┌─┬─┐
  │  │ │ │ │ │ │  │ │ │ │ │ │  (Single-row: 4 sets per side)
  │  └─┴─┘ └─┴─┘  └─┴─┘ └─┴─┘
  │
  │  ┌─┬─┐ ┌─┬─┐  ┌─┬─┐ ┌─┬─┐  (Double-row: 8 sets per row)
  │  │ │ │ │ │ │  │ │ │ │ │ │
  │  └─┴─┘ └─┴─┘  └─┴─┘ └─┴─┘
  │
  │  Wheelchair Space (1.5m x 1.0m, near door)
  │
  │  ┌─────────────────────────┐
  │  │  Emergency Exit & Sound │
  │  └─────────────────────────┘
  │
Back

CAPACITY:
├── Seated: 45-50 passengers
├── Standing (peak): 80+ passengers
└── Wheelchair accessible: 1-2 spaces
```

#### Seat Specifications
```
SEAT MODELS:
├── Seat cushion: Padded rectangle (blue fabric)
├── Backrest: 90° angle
├── Armrest: Between seats (optional model)
├── Handrails: Overhead running rails
│   └── Grip points at regular intervals
└── Accessibility: Lower grab rails near doors

MATERIALS:
├── Upholstery: Blue fabric texture (standard MTC color)
├── Frame: Aluminum (silver)
├── Floor: Anti-slip tile pattern (grey)
└── Handrails: Stainless steel (reflective shader)
```

### Sensor Placement Visualization

#### Where Sensors are Located

```
EXTERIOR SENSORS:

┌────────────────────────────────────────────┐
│    Roof: GPS Antenna (center top)          │
│    Sides: Door frame - Passenger counters  │
│    Front: Speed radar, emergency beacon    │
│    Rear: Reverse camera, brake lights      │
└────────────────────────────────────────────┘

INTERIOR SENSORS:

   Door Frame          Ceiling
      ▼                  ▼
   Motion       Temperature & 
   Sensors      Humidity Sensors
      │                 │
      ▼                 ▼
   Passenger ◄──────────┘
   Counter
   (Uses both)

Added as simple geometry:
├── Door sensor: Small box on frame
├── Temp sensor: Small sphere on ceiling
├── Camera: Cube with lens
└── GPS antenna: Simple stick (cylinder) on roof top

COLOR CODING:
├── GPS: Orange
├── Temp: Red
├── Passenger: Green
├── Camera: Black
```

---

## 🎨 MATERIALS & TEXTURES

### Material Definitions

| Component | Material Type | Suggested Properties |
|-----------|---------------|----------------------|
| **Bus Body** | Metallic Paint | Color: #001a4d (dark blue), Metallic: 0.7, Rough: 0.4 |
| **Windows** | Glass | Color: #1a3a52, Transmission: 0.8, IOR: 1.5 |
| **Wheels** | Rubber + Chrome | Tire: Black, smooth=0.9; Rim: Chrome, smooth=0.2 |
| **Solar Panels** | Silicon + Glass | Color: #1a2632, Metallic: 0.1, Rough: 0.3 |
| **Seats** | Fabric | Color: #0066cc (MTC blue), Rough: 0.8, Metallic: 0.0 |
| **Floor** | Anti-slip Tile | Color: Grey, Rough: 0.9, Pattern: Checkerboard |
| **Rails** | Stainless Steel | Metallic: 0.95, Roughness: 0.15 |

### Texture Creation Tips

**Blender**:
```
1. UV Mapping
   → Unwrap each component properly
   → Mark seams at logical places
   → Minimize distortion

2. Texture Painting
   → Use built-in painter for wear/tear
   → Add dust/dirt details on lower body
   → Rust spots on metal components

3. Procedural Textures
   → Use noise texture for fabric weave
   → Bump map for tire tread
   → Weathering effect using mix tools
```

**Unreal Engine**:
```
1. Texture Import
   → Import PBR textures (BaseColor, Normal, Roughness, Metallic)
   → Set compression settings for optimal quality

2. Material Blueprint
   → Connect textures to material nodes
   → Set up normal maps for detail
   → Add parallax occlusion for depth

3. Instance Materials
   → Create material instances for variation
   → Adjust colors for different bus liveries
   → Create wear variation without new textures
```

---

## 🌅 SCENE SETUP & LIGHTING

### Environment Setup

#### Scene Context
```
SCENE CONFIGURATION:

Location: Chennai, India
├── Latitude: 13°N
├── Typical conditions: Hot, humid, monsoon season
└── Lighting: Intense tropical sunlight (6am-6pm)

Time Scenarios to Model:
1. Morning (8:00 AM)
   ├── Sun angle: 30° elevation
   ├── Direction: East (shadows to west)
   └── Intensity: Moderate
   
2. Noon (12:00 PM)
   ├── Sun angle: 70° elevation
   ├── Direction: Overhead
   └── Intensity: Intense + heavy shadows
   
3. Evening (4:00 PM)
   ├── Sun angle: 35° elevation
   ├── Direction: West (shadows to east)
   └── Intensity: Moderate + golden tone

4. Night (8:00 PM)
   ├── No direct sunlight
   ├── Street lights only
   └── Opacity: Lights active
```

### Lighting Setup

#### Blender Lighting
```
BLENDER LIGHTS:

1. Sun Light (Primary)
   ├── Type: Sun lamp
   ├── Energy: 1.5 (for tropical intensity)
   ├── Angle: Varies by time scenario
   ├── Color: #fff5dc (warm, 5500K)
   └── Shadow: Ray shadow, 2048px resolution

2. Sky Texture (HDRI)
   ├── Type: World shader (HDRI map)
   ├── Rotation: Adjustable for sun direction
   ├── Strength: 1.5
   └── File: Use real sky HDRI for authenticity

3. Fill Lights (Secondary)
   ├── Type: Area lights
   ├── Purpose: Soften harsh shadows
   ├── Energy: 0.5
   └── Color: Slightly blue (#e8f4ff)

4. Emission Surfaces
   ├── Bus lights: Emissive material (yellow)
   ├── Dashboard: Soft glow (blue)
   └── Sensor indicators: Red/green glow
```

#### Unreal Engine Lighting
```
UNREAL LIGHTS:

1. Directional Light
   ├── Intensity: 1.5
   ├── Color: Morning (5500K), Noon (6500K), Evening (4500K)
   ├── Shadow quality: High
   └── Cascade: 4 levels for optimal shadow detail

2. Sky Sphere
   ├── Material: Custom sky material (blue gradient)
   ├── Time multiplier: Adjustable for sun movement
   └── Clouds: Optional cloud layer

3. Point Lights
   ├── Street light simulation
   ├── Bus headlights + taillights
   ├── Dashboard glow
   └── Energy: Varies by scenario

4. Exposure Settings
   ├── Exposure compensation: +0.5 (bright tropical)
   ├── Tone curve: Slight S-curve for contrast
   └── Color grading: Warm temperature
```

---

## 📤 EXPORT & OPTIMIZATION

### Export Formats & Settings

#### For Visualization (High Quality)
```
FORMAT: glTF 2.0 (.glb)

SETTINGS:
├── Triangulation: Enable all faces
├── Smoothing: Smooth Shading with normals
├── Compression: ON (cuts file size 50%)
├── Materials: Embedded
├── Textures: Embedded (or separate reference)
└── Animations: Include if any

FILE SIZE TARGET: 50-100 MB per scene
```

#### For Real-time Rendering (Optimized)
```
FORMAT: FBX (.fbx) or Alembic (.abc)

POLYGON REDUCTION:
├── Original: 250K polygons
├── Target: 50K-100K polygons (optimized)
└── Method: Decimation modifier in Blender

LOD (Level of Detail):
├── LOD0: Full detail (close view)
├── LOD1: 60% reduction (medium distance)
├── LOD2: 20% detail (far view)
└── LOD3: 10% detail (extreme distance)

SETTINGS:
├── Bake lighting: Pre-baked lightmaps
├── Texture resolution: 2048px max
├── Normal maps: Included
└── Use 16-bit vertex positions if available
```

#### For Physics Simulation
```
FORMAT: FBX with collision meshes

COLLISION SHAPES:
├── Bus body: Compound shape (simplified)
├── Wheels: Spheres (physics approximation)
├── Doors: Individual boxes
└── Sensors: Trigger spheres (non-physical)

REQUIRES DOCUMENTATION:
├── Mass: 11,000 kg (full load: 17,000 kg)
├── Center of mass: 1.2m above ground
├── Inertia tensors: Pre-calculated
└── Friction: High (rubber tires)
```

### Optimization Checklist

```
✓ GEOMETRY:
  ├── Removed duplicate vertices
  ├── Merged adjacent faces
  ├── Used modifiers sparingly
  └── Applied all transformations

✓ TEXTURES:
  ├── Compressed to DXT5 format
  ├── Mipmaps generated
  ├── Atlas used where possible
  └── 2K or 1K resolution max

✓ MATERIALS:
  ├── Minimal material count (< 20 unique)
  ├── Used instancing for variation
  ├── Removed unused material slots
  └── Simple procedural where possible

✓ HIERARCHY:
  ├── Organized parent-child structure
  ├── Named objects logically
  ├── Hidden detail during render (optional)
  └── Grouped related components

✓ RENDERING:
  ├── Baked textures for common objects
  ├── LOD generated
  ├── Culled off-screen objects
  └── Reduced shadow casting objects
```

---

## 🎬 Rendering Scenarios

### Scenario 1: Daytime Operation (Noon)

**Camera Setup**:
- Position: 45° angle, ~8m from bus
- Focus: Full bus profile
- Lighting: Direct tropical sunlight

**Output**:
- Bus clearly visible with sharp shadows
- Solar panels highlighted (angled for sun)
- Interior visible through windows (faint)
- Reflections on windows visible

### Scenario 2: Interior Walkthrough

**Camera Setup**:
- Start: At bus entrance
- Path: Through bus from front to rear
- Speed: Slow, 2-3 seconds per section

**Focus**:
- Seat layout and arrangement
- Dashboard details
- Sensor placements
- Emergency features

### Scenario 3: Solar Panel Detail

**Camera Setup**:
- Macro view of solar panel section
- 60° top-down angle
- Close enough to see cell details

**Focus**:
- Panel mounting structure
- Thermal gap spacing
- Connection points
- Tilt angle visualization

---

## 🎓 Best Practices for Bus Design

### Dos ✓
- ✅ Use real-world dimensions from MTC specifications
- ✅ Add details gradually (start simple, add complexity)
- ✅ Use modifiers for efficiency (Array, Bevel, Subdivision)
- ✅ Test multiple lighting scenarios
- ✅ Optimize before final export
- ✅ Document all custom settings
- ✅ Version control your blend files

### Don'ts ✗
- ❌ Over-model details (focus on visual quality, not perfection)
- ❌ Use non-optimized textures (keep under 2K resolution)
- ❌ Create single giant mesh (use logical grouping)
- ❌ Forget about UV mapping (plan before modeling)
- ❌ Ignore scale (always maintain real-world proportions)
- ❌ Export without testing (verify file integrity)

---

## 📚 Next Steps

1. Start with basic bus body shape in Blender/UE
2. Add wheels and suspension
3. Create and place solar panels
4. Model interior basics (seats, dashboard)
5. Add materials and textures
6. Set up lighting scenarios
7. Optimize and export
8. Test in target application

---

## 🔗 References & Resources

**Blender Resources**:
- Blender 3.6+ (free, open-source)
- Modifier documentation for optimization
- Principled BSDF shader for realistic materials

**Unreal Engine Resources**:
- UE 5.1+ for best performance
- Nanite system for geometry detail
- Lumen for real-time lighting

**3D Models Reference**:
- City Bus reference images (Google Images)
- MTC official bus specifications
- Real solar panel dimensions
- Indian road vehicle standards (IS codes)

---

**The 3D model should serve as both a visualization tool and a design reference document. Each component placement should be justified by the underlying system architecture.**
