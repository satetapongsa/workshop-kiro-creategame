# Flappy Kiro

A zero-dependency, high-performance HTML5 Canvas 2D action game implemented in vanilla JavaScript, built with clean architecture, deterministic state machine management, and zero external runtime dependencies.

---

## Architecture Overview

Flappy Kiro is designed around a decoupled, single-threaded Game Loop architecture driven by `requestAnimationFrame`. State transitions, physics calculations, collision detection, and rendering layers are strictly segregated to ensure low latency and deterministic frame processing.

### Architecture Diagram

```
+--------------------------------------------------------------------------+
|                               Browser DOM                                |
|                                                                          |
|  +--------------------+     +-------------------+     +---------------+  |
|  | Keyboard / Pointer |     |  LocalStorage     |     | HTML5 Canvas  |  |
|  | Event Listeners    |     |  Persistence      |     | 2D Context    |  |
|  +---------+----------+     +---------+---------+     +-------+-------+  |
+------------|--------------------------|-----------------------|----------+
             |                          |                       |
             v                          v                       |
+---------------------------------------------------------------+----------+
| Game Core Engine                                              |          |
|                                                               |          |
|  +--------------------+                                       |          |
|  | State Machine      | <---> [ IDLE | PLAYING | GAME_OVER ]  |          |
|  +---------+----------+                                       |          |
|            |                                                  |          |
|            v                                                  |          |
|  +---------------------------------------------------------+  |          |
|  | Game Loop (requestAnimationFrame @ ~60 FPS)             |  |          |
|  |                                                         |  |          |
|  |  +---------------------------------------------------+  |  |          |
|  |  | Update Phase                                      |  |  |          |
|  |  |  - Physics Engine (Euler Integration, Terminal V) |  |  |          |
|  |  |  - Pipe Subsystem (Spawn, Linear Scroll, Cull)    |  |  |          |
|  |  |  - Collision Detection (Axis-Aligned Bounding Box)|  |  |          |
|  |  |  - Scoring & High Score Engine                    |  |  |          |
|  |  +---------------------------+-----------------------+  |  |          |
|  |                              |                          |  |          |
|  |                              v                          |  |          |
|  |  +---------------------------------------------------+  |  |          |
|  |  | Render Phase (Layered Compositing Pipeline)       |  |  |          |
|  |  |  1. Sky & Procedural Background Canvas Clearing   |  |  |          |
|  |  |  2. Atmospheric Cloud Vector Pass                 |  |  |          |
|  |  |  3. Pipe Geometry Rasterization                   |  |  |          |
|  |  |  4. Ghost Sprite Blit / Procedural Fallback Box   |  |  |          |
|  |  |  5. Heads-Up Display (Score, Best Record)         |  |  |          |
|  |  |  6. State Overlay Layer (Idle / Game Over Modal)  |  |  |          |
|  |  +---------------------------------------------------+  |  |          |
|  +---------------------------------------------------------+  |          |
|                                                               |          |
|  +--------------------+                                       |          |
|  | Audio Engine (SFX) | ---> [ HTML5 Audio API / Fail-Safe ]  |          |
|  +--------------------+                                       |          |
+---------------------------------------------------------------+----------+
```

---

## State Machine Model

The application lifecycle follows a strict finite state machine (FSM) to avoid invalid game states and race conditions:

```
                  +-----------------------------------+
                  |                                   |
                  v                                   |
           +--------------+   Click / Space           |
           |     IDLE     | -----------------+        |
           +--------------+                  |        |
                                             v        |
           +--------------+   Collision    +----------------+
           |  GAME_OVER   | <------------- |    PLAYING     |
           +-------+------+  / Boundary    +----------------+
                   |
                   | Restart Button Click
                   +----------------------------------+
```

| State | Allowed Inputs | Transitions | Subsystem Behavior |
| :--- | :--- | :--- | :--- |
| `IDLE` | Spacebar, Canvas Click | `-> PLAYING` | Static physics, clouds rendered, title screen overlay active. |
| `PLAYING` | Spacebar, Canvas Click (Flap) | `-> GAME_OVER` | Continuous physics integration, dynamic pipe spawning, real-time collision checks, audio cues. |
| `GAME_OVER` | Restart Button Click | `-> PLAYING` | Physics suspended, game over HUD overlay shown, high score committed to persistent storage. |

---

## Key Subsystems and Specifications

### 1. Physics Engine
- **Gravity Acceleration:** Constant `0.4 px/frame^2` downward impulse applied continuously during `PLAYING` state.
- **Flap Impulse:** Instantaneous upward velocity vector reset to `-8 px/frame`.
- **Terminal Velocity:** Clamped downward velocity cap at `12 px/frame` to ensure fair reaction windows and prevent tunneling.

### 2. Collision Detection (AABB)
- Precise Axis-Aligned Bounding Box (AABB) computation with inset padding (`4px` perimeter offset) on the player entity to provide forgiving, arcade-standard hitboxes.
- Separate boundary evaluations for ceiling (`y < 0`) and ground floor baseline (`y + height > CANVAS_HEIGHT - HUD_HEIGHT`).
- Real-time overlapping test against both top and bottom pipe structures.

### 3. Rendering Pipeline
- **Target Resolution:** Fixed logical resolution of `480 x 640` pixels with uniform CSS aspect ratio maintenance.
- **Double Buffering:** Managed natively via the browser's 2D context drawing commands.
- **Fault-Tolerant Assets:** Sprite asset pipeline includes automatic fallback rendering to vector geometries if image assets fail to load or are blocked over strict networks.

### 4. Audio Architecture
- Non-blocking asynchronous playback using HTML5 Audio elements.
- Defensive error trapping on promise rejections (handling browser autoplay policies without runtime crashes).

### 5. Persistence
- Client-side storage abstraction utilizing Web Storage API (`localStorage`).
- Graceful degradation in environments where third-party cookies or storage APIs are restricted.

---

## Project Structure

```
.
|-- .kiro/                       # System specification and design documentation
|   `-- specs/
|       `-- flappy-kiro/
|           |-- design.md        # Technical design document
|           |-- requirements.md  # Formal requirements specification
|           `-- tasks.md         # Task breakdown and execution tracking
|-- assets/                      # Binary media assets
|   |-- game_over.wav            # Game over audio effect
|   |-- ghosty.png               # Ghost player character sprite
|   `-- jump.wav                 # Jump/flap audio effect
|-- img/                         # Project preview documentation
|   `-- example-ui.png           # Gameplay interface snapshot
|-- index.html                   # Monolithic entrypoint (HTML5, CSS3, JS ES6+)
|-- LICENCE.md                   # Project license terms
`-- README.md                    # Technical documentation and architecture
```

---

## Getting Started

### Prerequisites

Any modern web browser supporting HTML5 Canvas, ECMAScript 2015 (ES6), and Web Audio / HTML5 Audio:
- Google Chrome 60+
- Mozilla Firefox 55+
- Apple Safari 11+
- Microsoft Edge 79+

### Execution

Because the application is built without complex external bundlers or node runtime dependencies, it can be launched directly:

1. Clone or download the repository to your local environment.
2. Open `index.html` directly in your browser:
   ```bash
   # Windows (PowerShell)
   Start-Process index.html

   # Linux
   xdg-open index.html

   # macOS
   open index.html
   ```
3. Alternatively, serve via any lightweight static HTTP server:
   ```bash
   # Python 3.x
   python -m http.server 8080

   # Node.js npx
   npx serve .
   ```

---

## Controls and Keybindings

| Input | Key / Action | Context |
| :--- | :--- | :--- |
| **Start Game** | Spacebar or Primary Mouse Click | IDLE State |
| **Flap / Jump** | Spacebar or Primary Mouse Click | PLAYING State |
| **Restart Game** | Click Restart Button | GAME_OVER State |

---

## Engineering Standards

- **Zero Runtime Dependencies:** No external CDNs, runtime frameworks, or bloated bundler outputs.
- **Strict Mode:** Executed entirely under `'use strict'` semantics for performance optimization and error safety.
- **Cross-Platform Compatibility:** Responsive layout styling centered within any viewport dimensions.