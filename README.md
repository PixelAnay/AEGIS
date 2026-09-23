# AEGIS — Autonomous Emergency Grid & Incident System

> **AI-Powered Autonomous Drone Response for Urban Safety**  
> A simulation-based command-center platform for real-time incident detection, drone dispatch, and live tactical visualization.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Features](#features)
3. [System Architecture](#system-architecture)
4. [Dashboard Panels](#dashboard-panels)
5. [Simulation Engine](#simulation-engine)
6. [File Structure](#file-structure)
7. [How to Run](#how-to-run)
8. [Controls & Usage](#controls--usage)
9. [Incident Types](#incident-types)
10. [Drone Fleet](#drone-fleet)
11. [Tech Stack](#tech-stack)
12. [Upgrade Roadmap](#upgrade-roadmap)
13. [License](#license)

---

## Project Overview

AEGIS is a single-file browser-based **Urban Drone Response Command Dashboard** that simulates the full pipeline of AI-driven emergency response:

```
Detection → Alert → Dispatch → Navigation → Arrival → Resolution
```

The platform is built entirely in vanilla HTML/CSS/JavaScript with no external dependencies. It uses a canvas-based animated map, simulated video camera feeds with bounding-box AI detections, a live drone fleet panel, incident queue, and a scrolling system log — all rendered in real time at ~60fps.

---

## Features

| Feature | Description |
|---|---|
| AI Incident Detection | Simulated YOLOv8-style bounding boxes on 4 live camera feeds |
| Real-time Alert Generation | Incidents appear instantly with severity, location, and confidence score |
| Geolocation Mapping | Canvas-based tactical city map (Pune urban sector) |
| Nearest-Drone Dispatch | Euclidean distance algorithm selects closest available drone |
| Live Drone Tracking | Animated drone icons navigate to incident sites with trail rendering |
| ETA Display | Per-drone estimated time of arrival updated each frame |
| Incident Queue | Right-panel incident list with severity classification |
| System Log | Timestamped rolling log of all system events |
| Auto Simulation | Timer-driven random incident generation |
| Battery Simulation | Drones drain battery during dispatch; visual bar indicator |

---

## System Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    AEGIS Dashboard                      │
├──────────────┬──────────────────────┬───────────────────┤
│  Drone Fleet │    Tactical Map      │  Incident Queue   │
│  Panel       │    (Canvas 2D)       │                   │
│  (6 drones)  │    City grid         │  Alert cards      │
│              │    Drone icons       │  Severity tags    │
│  Status tags │    Incident markers  │  Drone assigned   │
│  Battery bar │    Path trails       │  ETA countdown    │
│  ETA display │    Patrol zones      │                   │
├──────────────┴──────┬───────────────┴───────────────────┤
│  Video Feeds (4x)   │  System Log                       │
│  Canvas animation   │  Timestamped events               │
│  Bounding boxes     │  Detection / Dispatch / Return     │
│  Scan line effect   │  Rolling 60-entry history         │
└─────────────────────┴───────────────────────────────────┘
```

---

## Dashboard Panels

### 1. Header Bar
- **Brand**: AEGIS logo + full system name
- **Status indicators**: System nominal, active incident count, sector ID, coverage %
- **Live clock**: Real-time `HH:MM:SS` display

### 2. Left Panel — Drone Fleet
Displays all 6 drones with:
- Drone ID (e.g. `KITE-01`, `HAWK-03`, `RAVEN-05`)
- Model name (DJI M30T / Parrot Anafi / SkyScout X)
- Status badge: `STANDBY` / `DISPATCHED` / `RETURNING` / `CHARGING`
- Battery percentage + color-coded bar (green → amber → red)
- Altitude reading (wobbles realistically)
- ETA in seconds when dispatched
- Bottom strip: live counts of Active / Dispatched / Charging

### 3. Center Panel — Tactical Map
A `<canvas>` rendering an animated top-down city map:
- **Background**: City grid with road network, building blocks, patrol zones
- **Incident markers**: Pulsing red danger circles with incident type icons
- **Drone icons**: Cross-shaped symbols with rotation animation when active
- **Path trails**: Purple dotted trail following drone movement
- **Dispatch lines**: Dashed lines from drone to incident target
- **Legend**: Color key for all map elements
- **Coordinates display**: Static GPS reference (Pune sector)

### 4. Right Panel — Incident Queue
Each incident card shows:
- Incident type with icon (e.g. `🔥 FIRE DETECTED`)
- Severity badge: `CRITICAL` / `HIGH` / `MEDIUM`
- Location name and camera number
- Detection description and confidence score (%)
- Assigned drone ID and ETA

### 5. Bottom Left — Video Feeds (4 cameras)
Four canvas-based simulated CCTV feeds showing:
- Animated city scene: buildings, pedestrians, vehicles
- Scan-line and timestamp overlay for realism
- **AI detection bounding boxes** with corner bracket style on incident trigger
- Red border pulse animation when camera detects an incident
- Camera label, zone name, and `● LIVE` status

### 6. Bottom Right — System Log
Rolling timestamped log with color-coded entry types:

| Color | Type | Example |
|---|---|---|
| Red | Alert | `DETECTION: FIRE DETECTED @ FC Road [CONF: 91.2%]` |
| Purple | Dispatch | `DISPATCH: KITE-01 → INCIDENT #1002 \| ETA 14s` |
| Green | OK | `ARRIVED: HAWK-03 @ INCIDENT #1003` |
| Amber | Warning | `WARNING: No drones available for INCIDENT #1004` |
| Dim | System | `SYSTEM: All subsystems nominal` |

---

## Simulation Engine

### Drone Movement
Each drone has:
- `(x, y)` normalized position (0.0–1.0 of canvas)
- `(bx, by)` base position (home station)
- `(targetX, targetY)` current navigation target
- `speed` value (~0.002 units/frame)
- `trail[]` array (last 30 positions) for path rendering

Movement is computed each frame:
```
direction = normalize(target - current)
position += direction * speed
```

When a dispatched drone reaches its target (distance < 0.005), it transitions to `on-scene`, waits 5–10 seconds (simulated response time), then returns to base.

### Dispatch Algorithm
When an incident is created, `findNearestDrone()` scans all `standby` drones and picks the one with minimum Euclidean distance to the incident coordinates. This is a greedy nearest-neighbor approach — a placeholder for A* pathfinding in production.

### Battery Drain
Drones in `dispatched` or `on-scene` status lose `0.02%` battery per frame (~1.2% per second at 60fps). Battery affects the visual indicator but does not currently ground drones (upgrade opportunity).

### Auto Simulation
When enabled, a `setInterval` fires every 4 seconds with a 35% probability of triggering a random incident, creating a realistic sparse-event pattern.

---

## File Structure

```
drone_response_platform.html     ← entire platform, single file
```

All code (HTML structure, CSS variables, JavaScript engine) is contained in one file for zero-dependency portability. No build step, no npm, no server required.

---

## How to Run

1. Download `drone_response_platform.html`
2. Open in any modern browser (Chrome, Firefox, Edge, Arc)
3. No internet connection required — fully offline

```bash
# Or serve locally
npx serve .
# Then open http://localhost:3000/drone_response_platform.html
```

**Browser compatibility**: Requires Canvas 2D API and `requestAnimationFrame`. Works in all Chromium and Firefox versions from 2020+.

---

## Controls & Usage

| Control | Action |
|---|---|
| `⚡ SIMULATE` button | Triggers a random incident immediately |
| `AUTO: OFF` button | Toggles auto-simulation (incident every ~4s) |
| `✕ CLEAR` button | Clears all active incidents, returns all drones to base |
| Click drone card | (UI hook — extend to select/highlight drone on map) |

### Simulation Flow
1. Click **⚡ SIMULATE** — an incident spawns at a random map location
2. The camera feed for that zone flashes red with a bounding box detection
3. An incident card appears in the right queue with severity and confidence
4. The nearest drone is selected and dispatched within 400ms
5. On the map, a dashed line appears from drone to incident; drone begins moving
6. System log records the full event chain
7. Drone arrives, waits on-scene, then auto-returns to base

---

## Incident Types

| Type | Severity | Description |
|---|---|---|
| FIRE DETECTED | Critical | Smoke/flame pixel detection |
| CROWD STAMPEDE | Critical | Abnormal crowd density model |
| ARMED THREAT | Critical | Weapon signature classification |
| VEHICLE ACCIDENT | High | Collision detection |
| STRUCTURAL COLLAPSE | High | Motion anomaly in building pixels |
| MEDICAL EMERGENCY | High | Person-down pose estimation |
| FLOOD RISK | Medium | Water level rise sensor |
| TRESPASSING | Medium | Perimeter breach detection |

All incidents carry a **confidence score** (78–98%) representing AI model certainty.

---

## Drone Fleet

| ID | Model | Home Zone | Speed |
|---|---|---|---|
| KITE-01 | DJI M30T | Top-left sector | 0.002–0.003 |
| KITE-02 | DJI M30T | Top-center sector | 0.002–0.003 |
| HAWK-03 | Parrot Anafi | Top-right sector | 0.002–0.003 |
| HAWK-04 | Parrot Anafi | Bottom-left sector | 0.002–0.003 |
| RAVEN-05 | SkyScout X | Bottom-center sector | 0.002–0.003 |
| RAVEN-06 | SkyScout X | Bottom-right sector | 0.002–0.003 |

Base positions are distributed evenly across the map for optimal sector coverage. Each drone spawns with 75–100% battery at startup.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Structure | HTML5 |
| Styling | CSS3 with custom properties (CSS variables) |
| Map rendering | Canvas 2D API |
| Video feeds | Canvas 2D API (procedural animation) |
| Animation loop | `requestAnimationFrame` (~60fps) |
| Fonts | Google Fonts (Orbitron, Barlow Condensed, Share Tech Mono) |
| Dependencies | **None** — fully self-contained |

---

## Upgrade Roadmap

### Near-term (add to existing HTML)
- [ ] **Leaflet.js real map** — swap canvas map for OpenStreetMap tiles
- [ ] **TensorFlow.js detection** — real COCO-SSD on webcam input
- [ ] **A\* route optimizer** — pathfinding around no-fly zones
- [ ] **Battery management** — ground low-battery drones, charge stations
- [ ] **Weather system** — wind drift affecting drone paths

### Medium-term (requires backend)
- [ ] **Node.js + WebSocket server** — multi-operator real-time sync
- [ ] **MongoDB incident history** — persistent logs and analytics
- [ ] **REST API** — external alert injection endpoint
- [ ] **JWT authentication** — operator login with roles

### Long-term (major features)
- [ ] **Three.js 3D map** — full 3D city with Blender `.glb` drone model
- [ ] **Drone swarm logic** — multi-drone coordinated response
- [ ] **FPV camera mode** — first-person view following selected drone
- [ ] **Analytics dashboard** — incident heatmap, response time charts
- [ ] **Voice commands** — Web Speech API dispatch control
- [ ] **Mobile app** — React Native operator companion

---

## Design System

AEGIS uses a dark tactical aesthetic with a custom CSS variable palette:

| Variable | Value | Usage |
|---|---|---|
| `--bg-base` | `#030810` | Page background |
| `--accent-primary` | `#00e5ff` | Cyan — primary UI accent |
| `--accent-danger` | `#ff3b3b` | Red — incidents and alerts |
| `--accent-warn` | `#ffb800` | Amber — warnings, returning drones |
| `--accent-ok` | `#00ff88` | Green — standby drones, success |
| `--accent-drone` | `#7b5fff` | Purple — dispatched drone highlights |

Fonts: **Orbitron** (IDs, clock), **Barlow Condensed** (body), **Share Tech Mono** (labels, logs).

---

## License

Built for the MIT-ADT University hackathon project: **AI-Powered Autonomous Drone Response for Urban Safety**.  
Single-file prototype — free to extend, modify, and deploy.

---

*AEGIS — Autonomous Emergency Grid & Incident System*  
*Developed with HTML5 Canvas + Vanilla JS | Zero dependencies | Offline-capable*
